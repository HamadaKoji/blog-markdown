# Gitlab RunnerのKubernetes Executorを完全に理解してカスタマイズの基礎を身につける

<strong>「このExecutorの動き、どないなってるんや…」</strong>

今支援させていただいてる顧客環境では、CI/CD基盤として、GitLab RunnerをセルフホスティングのEKS上で動作させています。設定自体は、マニュアルを読みながら実施することでそれほど苦労することはないんですが、さらに踏み込んでちょっとしたカスタマイズやインフラの最適化を実施しようとすると、内部構造を事前に理解しておく必要に迫られるわけです。

GitLab Runnerのセルフホスティングと一口で言ってもその方式はぎょーさんあるのですが、このブログでは、GitLab RunnerのKubernetes Executorの動作原理を、公式マニュアルを紐解きながら解説していきます。

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     GitLab Runnerﾏﾂﾘﾀﾞﾜｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>


## GitLab Runnerとは？

公式マニュアルのトップはこちら。

<p style="background-color: #e8e8e8;padding: 12px;">
[GitLab Runner \| GitLab](https://docs.gitlab.com/runner/)
</p>

GitLab Runnerは、GitLab社が提供するCI/CD基盤。位置づけとしてはGitHubにおけるGitHub Actionsと同等です。SaaSとしてGitLab.comの中でも利用できますし、セルフホスティングして自分で管理するインフラ上で動作させることも可能です。

### GitLab RunnerのExecutorとは？

<p style="background-color: #e8e8e8;padding: 12px;">
[Executors \| GitLab](https://docs.gitlab.com/runner/executors/)
</p>

GitLab Runnerをセルフホスティングする際に、Runnerを動作させる利用方式で、2022年5月現在、以下のExecutorが用意されています。

- SSH
- Shell
- Parallels
- VirtualBox
- Docker
- Docker Machine (auto-scaling)
- Kubernetes
- Custom

各Executorによる機能差はこちらに記載のとおり。

[Selecting the Executor](https://docs.gitlab.com/runner/executors/#selecting-the-executor)

今回支援している顧客環境では、アプリケーションワークロードにEKSを利用していることも有り、機能が豊富でノウハウもあるKubernetes Executorを採用することになった次第です。

## Kubernetes Executorの内部構造を理解する

<p style="background-color: #e8e8e8;padding: 12px;">
[The Kubernetes executor for GitLab Runner \| GitLab](https://docs.gitlab.com/runner/executors/kubernetes.html)
</p>

最小にKubernetes Executorがどんなステップでビルドを実行しているかの概要を押さえておきます。

1. Prepare：ジョブ用のPodが生成される
2. Pre-build：全ステージのアーティファクトクローン、キャッシュ復元、ダウンロードを行う。これは、Podの一部である専用コンテナを利用
3. Build：自身のジョブのビルドを実行
4. Post-build：キャッシュ作成、アーティファクトのGitLabへのアップロード。これも、Podの一部である専用コンテナを利用

もう少し、詳細にシーケンス図で把握するとこうなります。画像は公式マニュアルからの引用なので、先程のステップと合わせて動作をイメージしてみましょう。

010

## GitLab RunnerをKubernetesにインストールし、内部構造を調べてみる

ここまでで、基本的なKubernetes Executorの内部構造のイメージができたかと思います。

GitLabからは、KubernetesへのGitLab Runnerインストール用途でHelm Chartが公開されています。ここからは、このHelmChartを利用して、もう少し細かくKubernetes Executorの中身を確認していきます。

[GitLab Runner Helm Chart \| GitLab](https://docs.gitlab.com/runner/install/kubernetes.html)

事前に最小構成の設定ファイル<code>config-tmp.yaml</code>を用意しておきます。

[yaml title="config-tmp.yaml"]
gitlabUrl: https://gitlab.com/
runnerRegistrationToken: "XXXXXXXXXX1234567890"
rbac:
  create: true
[/yaml]

- <code>gitlabUrl</code>：GitLabサーバーのURL。SaaS版のGitLabからGitLab Runnerを利用する場合は、<code>https://gitlab.com/</code>
- <code>runnerRegistrationToken</code>：GitLabからこのGtiLab Runnerを利用するために必要なトークン。事前にGitLab側で取得しておく必要あり
- <code>rbac</code>：Role Base Accecc Controleを有効にするために設定しておきます

helm installを<code>dry-run</code>し、適用されるマニフェストファイルを確認します。

[bash]
helm install hamada-gitlab-runner gitlab/gitlab-runner --dry-run -f config-tmp.yaml --namespace gitlab-runner
[/bash]

上記を実行すると適用予定のマニフェストファイル一式が出力されます。作成されるリソースは、以下の通り

- ServiceAccount
- Secret
- ConfigMap
- Role
- RoleBinding
- Deployment

以下に、作成されるリソースについて解説していきます。

### ServiceAccount

GitLab Runnerで利用するService Accountの定義です。

[yaml]
# Source: gitlab-runner/templates/service-account.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
  name: hamada-gitlab-runner-gitlab-runner
  labels:
    app: hamada-gitlab-runner-gitlab-runner
    chart: gitlab-runner-0.37.2
    release: "hamada-gitlab-runner"
    heritage: "Helm"
[/yaml]


### Secret

GitLabからGitLab Runnerを起動する際に利用するトークンを格納するためのSecretを作成します。

[yaml]
# Source: gitlab-runner/templates/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: "hamada-gitlab-runner-gitlab-runner"
  labels:
    app: hamada-gitlab-runner-gitlab-runner
    chart: gitlab-runner-0.37.2
    release: "hamada-gitlab-runner"
    heritage: "Helm"
type: Opaque
data:
  runner-registration-token: "R1IxMzQ4OTQxSGlMZnVFcmVGR2pHdno2WVhQOFM="
  runner-token: ""
[/yaml]

### ConfigMap

GitLab RunnerのJobが実行されるとき専用のPodが起動しますが、そのPodで利用する設定情報が、このConfigMapで定義されます。<code>data</code>以降に、PodのEntryポイントが記載されているので、環境編集や設定ファイルがどのように展開されているか、ここで確認できます。

data以降は長いのでここには全文は掲載していません。

[yaml]
# Source: gitlab-runner/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: hamada-gitlab-runner-gitlab-runner
  labels:
    app: hamada-gitlab-runner-gitlab-runner
    chart: gitlab-runner-0.37.2
    release: "hamada-gitlab-runner"
    heritage: "Helm"
data:
  entrypoint: |
    #!/bin/bash
    set -e

    mkdir -p /home/gitlab-runner/.gitlab-runner/

    cp /configmaps/config.toml /home/gitlab-runner/.gitlab-runner/

    # Set up environment variables for cache
    if [[ -f /secrets/accesskey && -f /secrets/secretkey ]]; then
      export CACHE_S3_ACCESS_KEY=$(cat /secrets/accesskey)
      export CACHE_S3_SECRET_KEY=$(cat /secrets/secretkey)
    fi

〜〜後、省略〜〜

[/yaml]

### Role

ジョブ実行のPodで利用するServiceAccountで利用するRoleの設定です。権限は、<code>rules:</code>配下に記載があるとおり、クラスター内の全リソースに対する全ての処理が許可されています。

[yaml]
# Source: gitlab-runner/templates/role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: "Role"
metadata:
  name: hamada-gitlab-runner-gitlab-runner
  labels:
    app: hamada-gitlab-runner-gitlab-runner
    chart: gitlab-runner-0.37.2
    release: "hamada-gitlab-runner"
    heritage: "Helm"
  namespace: "gitlab-runner"
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["*"]
[/yaml]


### RoleBinding

前述で作成したRoleを、このRoleBindingでServiceAccountに紐付けます。<code>roleRef:</code>で紐付けるRoleを、<code>subjects:</code>で紐付けるServiceAccountの名前を定義しています。

[yaml]
# Source: gitlab-runner/templates/role-binding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: "RoleBinding"
metadata:
  name: hamada-gitlab-runner-gitlab-runner
  labels:
    app: hamada-gitlab-runner-gitlab-runner
    chart: gitlab-runner-0.37.2
    release: "hamada-gitlab-runner"
    heritage: "Helm"
  namespace: "gitlab-runner"
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: "Role"
  name: hamada-gitlab-runner-gitlab-runner
subjects:
- kind: ServiceAccount
  name: hamada-gitlab-runner-gitlab-runner
  namespace: "gitlab-runner"
[/yaml]


### Deployment

ジョブ実行時に起動するPodの初期情報を設定するDeploymentです。ここで、Podに設定されるImage、Volume、紐づくConfigmap、利用するServiceAccount、環境変数などが全て指定されます。

実際にPodが起動される際に利用させる全ての設定情報がここに記載されているので、細かい情報はここで確認可能です。

[yaml]
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hamada-gitlab-runner-gitlab-runner
  labels:
    app: hamada-gitlab-runner-gitlab-runner
    chart: gitlab-runner-0.37.2
    release: "hamada-gitlab-runner"
    heritage: "Helm"
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: hamada-gitlab-runner-gitlab-runner
  template:
    metadata:
      labels:
        app: hamada-gitlab-runner-gitlab-runner
        chart: gitlab-runner-0.37.2
        release: "hamada-gitlab-runner"
        heritage: "Helm"
      annotations:
        checksum/configmap: 9f5e01ff935a315851d42292060f67bc1387319f7ca2814f43546e9dd0daab7d
        checksum/secrets: 6cb1d605cac5eac1bca1526e9761f649a33f0ca2d647bc7ba6b121c62b20850c
        prometheus.io/scrape: 'true'
        prometheus.io/port: "9252"
    spec:
      securityContext:
        runAsUser: 100
        fsGroup: 65533
      terminationGracePeriodSeconds: 3600
      initContainers:
      - name: configure
        command: ['sh', '/configmaps/configure']
        image: gitlab/gitlab-runner:alpine-v14.7.0
        imagePullPolicy: "IfNotPresent"
        securityContext:
          allowPrivilegeEscalation: false
        env:
                
        - name: CI_SERVER_URL
          value: "https://gitlab.com/"
        - name: CLONE_URL
          value: ""
        - name: RUNNER_EXECUTOR
          value: "kubernetes"
        - name: REGISTER_LOCKED
          value: "true"
        - name: RUNNER_TAG_LIST
          value: ""
        volumeMounts:
        - name: runner-secrets
          mountPath: /secrets
          readOnly: false
        - name: configmaps
          mountPath: /configmaps
          readOnly: true
        - name: init-runner-secrets
          mountPath: /init-secrets
          readOnly: true
        resources:
          {}
      serviceAccountName: hamada-gitlab-runner-gitlab-runner
      containers:
      - name: hamada-gitlab-runner-gitlab-runner
        image: gitlab/gitlab-runner:alpine-v14.7.0
        imagePullPolicy: "IfNotPresent"
        securityContext:
          allowPrivilegeEscalation: false
        lifecycle:
          preStop:
            exec:
              command: ["/entrypoint", "unregister", "--all-runners"]
        command: ["/usr/bin/dumb-init", "--", "/bin/bash", "/configmaps/entrypoint"]
        env:
                
        - name: CI_SERVER_URL
          value: "https://gitlab.com/"
        - name: CLONE_URL
          value: ""
        - name: RUNNER_EXECUTOR
          value: "kubernetes"
        - name: REGISTER_LOCKED
          value: "true"
        - name: RUNNER_TAG_LIST
          value: ""
        livenessProbe:
          exec:
            command: ["/bin/bash", "/configmaps/check-live"]
          initialDelaySeconds: 60
          timeoutSeconds: 1
          periodSeconds: 10
          successThreshold: 1
          failureThreshold: 3
        readinessProbe:
          exec:
            command: ["/usr/bin/pgrep","gitlab.*runner"]
          initialDelaySeconds: 10
          timeoutSeconds: 1
          periodSeconds: 10
          successThreshold: 1
          failureThreshold: 3
        ports:
        - name: "metrics"
          containerPort: 9252
        volumeMounts:
        - name: runner-secrets
          mountPath: /secrets
        - name: etc-gitlab-runner
          mountPath: /home/gitlab-runner/.gitlab-runner
        - name: configmaps
          mountPath: /configmaps
        resources:
          {}
      volumes:
      - name: runner-secrets
        emptyDir:
          medium: "Memory"
      - name: etc-gitlab-runner
        emptyDir:
          medium: "Memory"
      - name: init-runner-secrets
        projected:
          sources:
            - secret:
                name: "hamada-gitlab-runner-gitlab-runner"
                items:
                  - key: runner-registration-token
                    path: runner-registration-token
                  - key: runner-token
                    path: runner-token
      - name: configmaps
        configMap:
          name: hamada-gitlab-runner-gitlab-runner
[/yaml]

### GitLab Runnerのインストール

ここまでで、実際に作成されるリソースの一覧の概要を把握できたので、実際にhelm installすることで、Kubernetes環境上にGitLab Runnerをインストールします。

[bash]
helm install hamada-gitlab-runner gitlab/gitlab-runner -f config-tmp.yaml --namespace gitlab-runner
[/bash]

無事インストールされ、GitLab.comからこのようにGitLab Runnerが認識されていれば、インストール成功です。お疲れさまでした！

[Settings] -> [CI/CD Settings] -> [Runners]

### 実際にデプロイされたPodを確認

Namespaceに<code>gitlab-runner</code>を指定しているため、該当Namespaceのに以下のPodが常駐しています。

[bash]
$ kubectl get pods -n gitlab-runner
NAME                                           READY   STATUS    RESTARTS   AGE
gitlab-runner-gitlab-runner-XXXXXXXXXX-YYYYYYYY   1/1     Running   0          3d17h
[/bash]

Describeすると、GitLab Runnerインストール時に設定された各種関連リソースを確認できます。

[yaml]
$ kubectl describe pod gitlab-runner-gitlab-runner-XXXXXXXXXX-YYYYYYYY -n gitlab-runne
Name:         gitlab-runner-gitlab-runner-XXXXXXXXXX-YYYYYYYY
Namespace:    gitlab-runner
Priority:     0
Node:         ip-10-0-2-230.ec2.internal/10.0.2.230
Start Time:   Mon, 02 May 2022 14:53:22 +0900
Labels:       app=gitlab-runner-gitlab-runner
              chart=gitlab-runner-0.28.0
              heritage=Helm
              pod-template-hash=5bf78645f6
              release=gitlab-runner
Annotations:  checksum/configmap: f38a8b6828475f9a23923abbc5cfefc71b8e77dd1711eb4a643ee33adc2f2035
              checksum/secrets: b854aa3a7279d6415ddd6de04c79674630640ea1753a0dfee201d6c55d24003a
              kubernetes.io/psp: eks.privileged
              prometheus.io/port: 9252
              prometheus.io/scrape: true
Status:       Running
IP:           10.0.2.105
IPs:
  IP:           10.0.2.105
Controlled By:  ReplicaSet/gitlab-runner-gitlab-runner-5bf78645f6
Init Containers:
  configure:
    Container ID:  docker://9b929e7b9b418a782e18c62b8c571339ce71b7fdb41a7557d1d21e7d6cbfcf6e
    Image:         gitlab/gitlab-runner:alpine-v13.11.0
    Image ID:      docker-pullable://gitlab/gitlab-runner@sha256:d01c7fdfe5b85d55b66bb0ba01d1e52ac31747811b522f05dac2d514219fbd9f
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      /configmaps/configure
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Mon, 02 May 2022 14:53:23 +0900
      Finished:     Mon, 02 May 2022 14:53:23 +0900
    Ready:          True
    Restart Count:  0
    Environment:
      CI_SERVER_URL:                                    https://gitlab.com/
      CLONE_URL:                                        
      RUNNER_REQUEST_CONCURRENCY:                       1
      RUNNER_EXECUTOR:                                  kubernetes
      REGISTER_LOCKED:                                  false
      RUNNER_TAG_LIST:                                  hamada-common-eks-gitlab-runner
      KUBERNETES_IMAGE:                                 
      KUBERNETES_NAMESPACE:                             gitlab-runner
      KUBERNETES_CPU_LIMIT:                             
      KUBERNETES_CPU_LIMIT_OVERWRITE_MAX_ALLOWED:       
      KUBERNETES_MEMORY_LIMIT:                          
      KUBERNETES_MEMORY_LIMIT_OVERWRITE_MAX_ALLOWED:    
      KUBERNETES_CPU_REQUEST:                           
      KUBERNETES_CPU_REQUEST_OVERWRITE_MAX_ALLOWED:     
      KUBERNETES_MEMORY_REQUEST:                        
      KUBERNETES_MEMORY_REQUEST_OVERWRITE_MAX_ALLOWED:  
      KUBERNETES_SERVICE_ACCOUNT:                       
      KUBERNETES_SERVICE_CPU_LIMIT:                     
      KUBERNETES_SERVICE_MEMORY_LIMIT:                  
      KUBERNETES_SERVICE_CPU_REQUEST:                   
      KUBERNETES_SERVICE_MEMORY_REQUEST:                
      KUBERNETES_HELPER_CPU_LIMIT:                      
      KUBERNETES_HELPER_MEMORY_LIMIT:                   
      KUBERNETES_HELPER_CPU_REQUEST:                    
      KUBERNETES_HELPER_MEMORY_REQUEST:                 
      KUBERNETES_HELPER_IMAGE:                          
      KUBERNETES_PULL_POLICY:                           
    Mounts:
      /configmaps from configmaps (ro)
      /init-secrets from init-runner-secrets (ro)
      /secrets from runner-secrets (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from gitlab-runner-gitlab-runner-token-6cqb4 (ro)
Containers:
  gitlab-runner-gitlab-runner:
    Container ID:  docker://fcb071087e27e0a73fc33f095baa6c32cc60dc5644bd48b16b07f766ac09a01a
    Image:         gitlab/gitlab-runner:alpine-v13.11.0
    Image ID:      docker-pullable://gitlab/gitlab-runner@sha256:d01c7fdfe5b85d55b66bb0ba01d1e52ac31747811b522f05dac2d514219fbd9f
    Port:          9252/TCP
    Host Port:     0/TCP
    Command:
      /bin/bash
      /configmaps/entrypoint
    State:          Running
      Started:      Mon, 02 May 2022 14:53:24 +0900
    Ready:          True
    Restart Count:  0
    Liveness:       exec [/bin/bash /configmaps/check-live] delay=60s timeout=1s period=10s #success=1 #failure=3
    Readiness:      exec [/usr/bin/pgrep gitlab.*runner] delay=10s timeout=1s period=10s #success=1 #failure=3
    Environment:
      CI_SERVER_URL:                                    https://gitlab.com/
      CLONE_URL:                                        
      RUNNER_REQUEST_CONCURRENCY:                       1
      RUNNER_EXECUTOR:                                  kubernetes
      REGISTER_LOCKED:                                  false
      RUNNER_TAG_LIST:                                  hamada-common-eks-gitlab-runner
      KUBERNETES_IMAGE:                                 
      KUBERNETES_NAMESPACE:                             gitlab-runner
      KUBERNETES_CPU_LIMIT:                             
      KUBERNETES_CPU_LIMIT_OVERWRITE_MAX_ALLOWED:       
      KUBERNETES_MEMORY_LIMIT:                          
      KUBERNETES_MEMORY_LIMIT_OVERWRITE_MAX_ALLOWED:    
      KUBERNETES_CPU_REQUEST:                           
      KUBERNETES_CPU_REQUEST_OVERWRITE_MAX_ALLOWED:     
      KUBERNETES_MEMORY_REQUEST:                        
      KUBERNETES_MEMORY_REQUEST_OVERWRITE_MAX_ALLOWED:  
      KUBERNETES_SERVICE_ACCOUNT:                       
      KUBERNETES_SERVICE_CPU_LIMIT:                     
      KUBERNETES_SERVICE_MEMORY_LIMIT:                  
      KUBERNETES_SERVICE_CPU_REQUEST:                   
      KUBERNETES_SERVICE_MEMORY_REQUEST:                
      KUBERNETES_HELPER_CPU_LIMIT:                      
      KUBERNETES_HELPER_MEMORY_LIMIT:                   
      KUBERNETES_HELPER_CPU_REQUEST:                    
      KUBERNETES_HELPER_MEMORY_REQUEST:                 
      KUBERNETES_HELPER_IMAGE:                          
      KUBERNETES_PULL_POLICY:                           
    Mounts:
      /configmaps from configmaps (rw)
      /home/gitlab-runner/.gitlab-runner from etc-gitlab-runner (rw)
      /secrets from runner-secrets (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from gitlab-runner-gitlab-runner-token-6cqb4 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  runner-secrets:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     Memory
    SizeLimit:  <unset>
  etc-gitlab-runner:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     Memory
    SizeLimit:  <unset>
  init-runner-secrets:
    Type:                Projected (a volume that contains injected data from multiple sources)
    SecretName:          gitlab-runner-gitlab-runner
    SecretOptionalName:  <nil>
  configmaps:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      gitlab-runner-gitlab-runner
    Optional:  false
  gitlab-runner-gitlab-runner-token-6cqb4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  gitlab-runner-gitlab-runner-token-6cqb4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:          <none>
[/yaml]


## 参考：GitLab Runner構築時の公式マニュアルとそのあるき方

最後に、GitLab Runner関連の情報を得るときの公式マニュアルの参照方法を参考までにお届け。

こちらで、GitLab Runnerの構築手順が記載されています。GitLab Runnerは様々な環境でセルフホストする手段があるので、それぞれの環境で最適なものを選択し、利用してみてください。

[Install GitLab Runner \| GitLab](https://docs.gitlab.com/runner/install/)

最初わかりにくかったのですが、公式マニュアルには、別の階層にGitLab Runnerの解説マニュアルがあります。

[Use GitLab] -> [Build your application] -> [Runners]

[GitLab Runner \| GitLab](https://docs.gitlab.com/runner/)

1つ目に記した[Install GitLab Runner]配下にも、GitLab Runnerのカスタマイズに関する情報が含まれていますが、こちらでは、より詳細な設定方法や運用上のベストプラクティスも含まれているため、本格的にGitLab Runnerをセルフホスティングで運用するのであれば、こちらも一通り目を通しておくことをオススメします。


## セルフホスティングのGitLab Runnerを細かくカスタマイズするためには、内部構造の理解は必須

インストールして動かすだけであれば、マニュアルに基づきそのままHelm installでEKS上にGitLab Runnerをインストールするだけで事足ります。が、その後急遽プロジェクトの要件で、GitLab Runnerの各ジョブに個別にIAMロールを紐付ける必要性がでてきました。

最初とりあえずインストールしただけでは、GitLab Runnerの設定ファイルである<code>values.yaml</code>の内容がイマイチ理解できていなかったのですが、今回、実際にGitLab Runnerインストール時に適用される各種Kubernetesリソースのマニフェストファイルを把握することで、起動されるジョブに紐づくServiceAccountや、関連するRole、各種設定ファイルの関連がわかり、より深くGitLab RunnerのKubernetes Executorの構造が理解できたのが収穫でした。

セルフホストのGitLab Runnerには膨大な設定項目があり、それらを利用することでよりプロジェクトの状況に合わせたCI/CD基盤が構築できます。カスタマイズを進めていくに当たり、基本的な構造を抑えておくことは必須なので、このブログを参考にしていただきながら、紹介した公式マニュアルと合わせて理解を深めてもらえれば幸いです。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

