# EKS環境をsysdigでモニタリングしてみた（セットアップ編）

「みんな、sysdig初めてみましょうかね」

という軽いノリで始まりましたが、AWSのContainers Security Partnersにも名前を連ねるSysdigを触ってみたのでその様子をお届けいたします。今回はインストール編ということで、EKS環境の作成とSysdigエージェントの設定周辺をご紹介。その設定手順の様子を見ていただきながら、案外簡単に入るんだなぁというところ見てもらえればと思います。

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     Sysdig ﾏﾂﾘﾀﾞﾜｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>


## Sysdigとは？

005

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://sysdig.com/" target="_blank">Secure DevOps Platform for Cloud-Native | Sysdig</a>
</p>

sysdigのVisionについては、こちらの動画によく出てます。作りがすごいドラマチックでかっこいい！

<iframe width="560" height="315" src="https://www.youtube.com/embed/5vd2gGTUfAM" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

AWSのContaeinr Security Partnersにも名前を連ねています。

- <a href="https://aws.amazon.com/jp/containers/partner-solutions/" target="_blank">AWS Container Partner Solutions</a>

日本語だと、こちらの記事でSysdigが生まれてきた背景やアーキテクチャ上の特徴が端的に紹介されているので、非常に参考になります。ぜひこちらも御覧ください。

- <a href="https://thinkit.co.jp/article/15841" target="_blank">クラウドネイティブなセキュリティはどこに向かう？ Sysdigに訊いたモニタリングとセキュリティの融合 | Think IT（シンクイット）</a>

アーキテクチャ上の特徴としては、Kubernetesベースのシステムにおいて、Podにエージェントを入れるのではなく、ノードにエージェントを入れる方法をとっており、ここからホストのカーネルにアクセスしてシステムコールを採取する方式です。

007
（引用元：https://thinkit.co.jp/article/15841）

メイン製品ラインとしては、Sysdig MonitorとSysdig Securityがあり、今回はSysdig Monitorのセットアップについてのブログとなります。

また、sysdigにはSaaS版とオンプレミス版の2つがありますが、今回はSaaS版を前提とした手順になっています。

## EKS環境のモニタリングを完了させるまでの手順

今回、EKS環境をSysdigモニタリングするのですが、その大まかな手順をここでおさらいしておきましょう。といっても手順自体は全体的にシンプルで、Kubernetes環境が存在していれば、そんなに苦労するところは無いと思います。

1. EKS環境をワーカーノード含めて作成し、適当なPodを動作させる
2. Sysdigエージェントのアクセスキーを取得する
3. kubernetesクラスターにSysdigエージェント用の設定を追加する
4. AWSインテグレーション部分の設定を追加する

## 手順①：EKS環境をワーカーノード含めて作成し、適当なPodを動作させる

最初にKubernetes環境作っていきます。Kubernetes環境自体は別になにでも良いので、すでに適当な環境があるかたはそれをお使いください。

AWSでEKS環境をセットアップするのに一番簡単なのは、<code>eksctl</code>を使う方法です。以下手順にそって、EKSクラスターを作成してみてください。

- <a href="https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/getting-started-eksctl.html" target="_blank">eksctl の開始方法 - Amazon EKS</a>

<strong>一点注意事項として、今回SysdigエージェントはワーカーノードにDaemonsetとして展開するため、ワーカーノードを作成する[Cluster with Linux-only workload]を利用して進めてください（Fargateではなく）。</strong>

ワーカーノードまでできあがったら、モニタリング用のPodを配置しましょう。なんでも良いのですが、自分はEKS Workshopで紹介されていた、以下のサンプルアプリを利用しました。

- <a href="https://eksworkshop.com/beginner/050_deploy/" target="_blank">Deploy the Example Microservices :: Amazon EKS Workshop</a>

さて、無事、EKS環境出来上がりましたでしょうか？

## 手順②：Sysdigエージェントのアクセスキーを取得する

ここから、Sysdigの話になります。アクセスキーの発行にはライセンスが必要となるので、無料お試しからライセンスを取得してみましょう。日本語ページがわかりやすいと思います。

- <a href="https://sysdig.jp/" target="_blank">クラウドネイティブな Secure DevOps Platform</a>

上から「無料お試し」からライセンスを取得してみてください。その後、流れの中でSysdig Agentのアクセスキーが表示されるのでそれをメモっておいてください。

もしくはセットアップ後、自分のアイコンをクリックし[Settings]→[Agent Installation]にアクセスでも、アクセスキーを確認できます。

## 手順③：kubernetesクラスターにSysdigエージェント用の設定を追加する

いよいよ、クラスターにSysdigエージェント用の設定を追加していきます。メインで<code>kubectl</code>を使います。

エージェントインストールにあたり前提条件は以下の通り。

- エージェントインストール対象ホスト制限
  - <a href="https://docs.sysdig.com/en/host-requirements-for-agent-installation.html" target="_blank">Host Requirements for Agent Installation</a>
  - EC2なら問題なし
- Kubernetesのバージョンが1.2以上
  - sysdigエージェントはDaemonSetsでデプロイされるため、1.2以上のバージョンが必要となります
- Sysdigのアカウントとアクセスキー
  - 事前にSysdingアカウントを作成し、ウェルカムウィザードでアクセスキーを取得しておきます

実際にインストールを実施していきます。公式マニュアルの手順は、以下を参照してみてください。

- 公式マニュアル：<a href="https://docs.sysdig.com/en/kubernetes-agent-installation-steps.html" target="_blank">Kubernetes Agent Installation Steps</a>

<a href="https://github.com/draios/sysdig-cloud-scripts/tree/master/agent_deploy/kubernetes" target="_blank">sysdig-cloud-scripts/agent_deploy/kubernetes at master · draios/sysdig-cloud-scripts</a>より、以下のファイルをダウンロードします。

- <code>sysdig-agent-clusterrole.yaml</code>
- <code>sysdig-agent-daemonset-v2.yaml</code>
- <code>sysdig-agent-configmap.yaml</code>

[bash]
curl -O https://raw.githubusercontent.com/draios/sysdig-cloud-scripts/master/agent_deploy/kubernetes/sysdig-agent-clusterrole.yaml
curl -O https://raw.githubusercontent.com/draios/sysdig-cloud-scripts/master/agent_deploy/kubernetes/sysdig-agent-daemonset-v2.yaml
curl -O https://raw.githubusercontent.com/draios/sysdig-cloud-scripts/master/agent_deploy/kubernetes/sysdig-agent-configmap.yaml
[/bash]

sysdigエージェント用の名前空間を設定します。名前空間の作成は任意ですが、sysdig関連のコンポーネントはそれなりに数があるため、管理上の手間も考えると名前空間（namespace）は作っておいたほうが良いです。今回は、Namespaceに<code>sysdig-agent</code>を使います。

[bash]
kubectl create ns sysdig-agent
[/bash]

sysdigエージェントが利用するシークレットを登録します。利用するAccess Keyは、「手順②：Sysdigエージェントのアクセスキーを取得する」を使います。下記の<code><your sysdig access key></code>を書き換えてください。

[bash]
kubectl create secret generic sysdig-agent --from-literal=access-key=<your sysdig access key> -n sysdig-agent
[/bash]

kubernetesクラスター用のロール、サービスアカウントなどを定義していきます。

[bash]
kubectl apply -f sysdig-agent-clusterrole.yaml -n sysdig-agent
kubectl create serviceaccount sysdig-agent -n sysdig-agent
kubectl create clusterrolebinding sysdig-agent --clusterrole=sysdig-agent --serviceaccount=sysdig-agent:sysdig-agent  
[/bash]

ConfigMapを書く環境に合わせて修正します。kubernetesのStateメトリクスとクラスター名を有効化します。<code>sysdig-agent-configmap.yaml</code>を開きます。以下の部分の、<code>new_k8s</code>、<code>k8s_cluster_name</code>の部分をコメントアウトして書き換えます。

修正前。

[bash title="sysdig-agent-configmap.yaml" highlight="2-3"]
    #######################################
    # new_k8s: true
    # k8s_cluster_name: production
    security:
      k8s_audit_server_url: 0.0.0.0
      k8s_audit_server_port: 7765
[/bash]

修正後はこのようになります。cluster_nameは、各環境のkubernetesクラスター名を指定してください。

[bash title="sysdig-agent-configmap.yaml" highlight="2-3"]
    #######################################
    new_k8s: true
    k8s_cluster_name: <your cluster name>
    security:
      k8s_audit_server_url: 0.0.0.0
      k8s_audit_server_port: 7765
[/bash]

修正が完了したら、Configmapを展開します。

[bash]
kubectl apply -f sysdig-agent-configmap.yaml -n sysdig-agent
[/bash]


DaemonSetを展開します。

[bash]
kubectl apply -f sysdig-agent-daemonset-v2.yaml -n sysdig-agent
[/bash]

これで、実際のsysdigエージェントがPodとしてRunning状態になっていればOKです。daemonsetやpodの状態を確認してみましょう。

[bash]
$ kubectl get daemonset -n sysdig-agent
NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
sysdig-agent   2         2         2       2            2           <none>          2d
[/bash]

[bash]
$ kubectl get pod -n sysdig-agent
NAME                 READY   STATUS    RESTARTS   AGE
sysdig-agent-bxqpz   1/1     Running   0          2d
sysdig-agent-msmn4   1/1     Running   0          2d
[/bash]

こんな感じで、sysdigエージェントが起動していればOKです。ここまでOKでしょうか？後一息ですよ。


## 手順4：AWSインテグレーション部分の設定を追加する

最後に、Sysdig monitorからAWS環境に対して情報を取得するための認証情報を設定します。流れとしては、アクセス対象のAWSアカウントに権限をもつIAMユーザーを作成し、そのアクセスキーとシークレットキーをsysdig側に設定する流れとなります。詳細な手順は以下を参照してください。

- <a href="https://docs.sysdig.com/en/aws--integrate-aws-account-and-cloudwatch-metrics--optional-.html" target="_blank">AWS: Integrate AWS Account and CloudWatch Metrics (Optional)</a>

AWSに慣れている方であれば、下記ポリシーを保有したIAMユーザーを作成し、その情報を[Settings]→[AWS]で設定すればOKです。

[json]
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "autoscaling:Describe*",
                "cloudwatch:Describe*", 
                "cloudwatch:Get*", 
                "cloudwatch:List*", 
                "dynamodb:ListTables",
                "dynamodb:Describe*",
                "ec2:Describe*",
                "ecs:Describe*",
                "ecs:List*",
                "elasticache:DescribeCacheClusters", 
                "elasticache:ListTagsForResource",
                "elasticloadbalancing:Describe*",
                "rds:Describe*",
                "rds:ListTagsForResource",
                "sqs:ListQueues",
                "sqs:GetQueueAttributes",
                "sqs:ReceiveMessage"
            ],
            "Effect": "Allow",
            "Resource": "*"
        }
    ]
}
[/json]

090

上記画面でStatusがOKになっていたら大丈夫です。

## （重要）すべての設定が完了したらしばらく待つ

自分はここでしばらくハマりましたｗ　エージェントも動いていて、AWSインテグレーションも設定したののに、いつまでたっても、Sysdig Monitorのエクスプローラーに何も表示されない場合は、しばらく待ちましょう。15分ほどまって、さらにSysdig管理画面からログアウトしてログインするなどして見ましょう。

しばらくまって、以下のように[EXPLORE]において、GROUPINGSにAWSのアイコンやKubernetesのアイコンが選択可能になっていたら、無事データが連携されています。お疲れさまでした！

100

## セットアップ自体はシンプル。画面反映に時間がかかる

以上、Sysdig MonitorによるEKSクラスターへのセットアップ手順を紹介しました。丁寧にやっていけばそれほどハマリポイントも無いと思います。

sysdigのドキュメントはすべてこちらにまとめられているので、全体像を知るにはこちらをみてください。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://docs.sysdig.com/?lang=en" target="_blank">Sysdig Documentation</a>
</p>

セットアップが完了したので、この後は、EKSクラスターで展開したPodや各種リソースがどのように見えるのかを解説していきます。こちらの記事をご参照ください。

- <a href="https://dev.classmethod.jp/cloud/aws/sysdig-monitoring-monitor/" target="_blank">EKS環境をsysdigでモニタリングしてみた（画面編） ｜ Developers.IO</a>

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。



