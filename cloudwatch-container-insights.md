# k8s環境のメトリクスやログを取得するマネージド・サービス「CloudWatch Container Insights」が発表されました！

現在開催されている<a href="https://events.linuxfoundation.org/events/kubecon-cloudnativecon-europe-2019/" target="_blank">KubeCon + CloudNativeCon Europe 2019 - Linux Foundation Events</a>において、Kubernetes環境のコンテナ環境のメトリクスを取得する「CloudWatch Container Insights」が発表されました！

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">コンテナワークロードのためのメトリクス・ログモニタリングサービス、CloudWatch Container Insights の Public Preview を発表しました！！ <a href="https://twitter.com/hashtag/KubeCon?src=hash&amp;ref_src=twsrc%5Etfw">#KubeCon</a><br><br>続) <a href="https://t.co/pRCZtHexcp">pic.twitter.com/pRCZtHexcp</a></p>&mdash; ポジティブな Tori (@toricls) <a href="https://twitter.com/toricls/status/1130505697651765250?ref_src=twsrc%5Etfw">May 20, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<strong>パブリックプレビューなので、今すぐ動作確認できます。もちろん、東京リージョンもあるよ！！</strong>

* Canada (Central)
* US East (N. Virginia)
* US East (Ohio)
* US West (N. California)
* US West (Oregon)
* EU (Frankfurt)
* EU (Paris)
* Asia Pacific (Mumbai)
* Asia Pacific (Singapore)
* Asia Pacific (Sydney)
* <strong>Asia Pacific (Tokyo)</strong>
* Asia Pacific (Seoul)
* South America (São Paulo)


<p class="alert">CloudWatch Container Insightsは、2019年５月21日現在、まだ正式リリースされていないパブリックプレビューの状態で、サービス内容はいつでも変更される可能性があるので、あくまで開発〜検証環境で試してみてください</p>

<pre style="line-height:120%;">
k8sコンテナ丸見えきたか…!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>

## CloudWatch Container Insightsとは？

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
公式ドキュメント：<a href="https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/ContainerInsights.html" target="_blank">Using Container Insights - Amazon CloudWatch</a>
</p>

従来、AWSにおけるk8s環境（EKS含む）において、そのNode（EC2）や各コンテナ、ネットワークのメトリクスやログを収集するためのマネージドの仕組みは存在せず、別途FluentdやDatadogなどのオープンソースやSaaSなどを利用する必要がありました。

<strong>今回のCloudWatch Container Insightsの登場により、CPU、メモリー、ディスク、ネットワークを収集し、かつコンテナ環境の問題の特定と解決に重要な、コンテナの再起動の失敗などの診断情報も取得できます。さらに、CloudWatchアラームとの連携ももちろんできます！
</strong>

さらに、運用データはログイベントとして収集可能で、それらを組み合わせたダッシュボードやメトリクスを提供します。

## EKS環境のセットアップ

もし、手元にEKSの環境がない場合は、クイックスタートを使うのが一番はやいです。こちらの記事を参考にしてみてください。

- <a href="https://dev.classmethod.jp/cloud/aws/eks-quickstart/" target="_blank">3個のパラメータでEKSクラスタをサクッと構築するクイックスタートの紹介 ｜ DevelopersIO</a>

## Container Insightsのセットアップ方法

- <a href="https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/deploy-container-insights.html" target="_blank">Setting Up Container Insights - Amazon CloudWatch</a>

まず最初に、導入の前提条件を確認します。

### 1. 導入前提条件の確認

- <a href="https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/Container-Insights-prerequisites.html" target="_blank">Verify Prerequisites - Amazon CloudWatch</a>

改めて、現在利用できるリージョンを確認します。

- <a href="https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/ContainerInsights.html" target="_blank">Using Container Insights - Amazon CloudWatch</a>

導入には<code>kubectl</code>が必要となります。こちら（<a href="https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/install-kubectl.html" target="_blank">kubectl のインストール - Amazon EKS</a>）を確認してください。

また、すでにk8s環境を動作させている場合は、いかが必要となります。

- k8sクラスターがrole-based access control（RBAC）で動作していること
	- 参考情報（<a href="https://kubernetes.io/docs/reference/access-authn-authz/rbac/" target="_blank">Using RBAC Authorization - Kubernetes</a>）
- kubeletがWebhook認証モードで動作していること
	- 参考情報（<a href="https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-authentication-authorization/" target="_blank">Kubelet authentication/authorization - Kubernetes</a>）

### 2. ワーカーノードへのIAMロールの割当

k8sのワーカーノードとして動作しているEC2インスタンスのIAMロールに<code>CloudWatchAgentServerPolicy</code>をアタッチします。

### 3. クラスターメトリクス収集のためのCloudWatch Agentの設定

- <a href="https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-metrics.html" target="_blank">Set Up the CloudWatch Agent to Collect Cluster Metrics - Amazon CloudWatch</a>

#### CloudWatch用のネームスペースの作成

CloudWatch用のネームスペースをk8sクラスターに作成します。

[bash]
$curl -O https://s3.amazonaws.com/cloudwatch-agent-k8s-yamls/kubernetes-monitoring/cloudwatch-namespace.yaml
[/bash]

yamlの中はこんな感じです。

[bash title="cloudwatch-namespace.yaml"]
$ cat cloudwatch-namespace.yaml 
apiVersion: v1
kind: Namespace
metadata:
  name: amazon-cloudwatch
  labels:
    name: amazon-cloudwatch
[/bash]

<code>kubectl apply</code>で、名前空間を適用します。

[bash]
$kubectl apply -f cloudwatch-namespace.yaml
[/bash]

こんな感じでネームスペースが作成されていればOkです。

[bash]
$ kubectl get namespace
NAME                STATUS    AGE
amazon-cloudwatch   Active    16s
default             Active    33m
kube-public         Active    33m
kube-system         Active    33m
[/bash]

#### クラスター内にサービスアカウントを追加

まだ設定していない状態であれば、下記手順で、CloudWatch　Agentのサービスアカウントを作成します。

[bash]
$curl -O https://s3.amazonaws.com/cloudwatch-agent-k8s-yamls/kubernetes-monitoring/cwagent-serviceaccount.yaml
[/bash]

[bash]
$kubectl apply -f cwagent-serviceaccount.yaml
[/bash]

#### CloudWatch Agent用のConfigMapを追加

CloudWatch agent用のConfigMapを追加します。CloudWatch agentの設定情報は、ConfigMapを利用して設定するのでkubernetesのバージョンは<code>1.12</code>以上が必要となります。

[bash]
$ curl -O https://s3.amazonaws.com/cloudwatch-agent-k8s-yamls/kubernetes-monitoring/cwagent-configmap.yaml
[/bash]

ファイルの中身はこんな感じになっています。

[bash title="cwagent-configmap.yaml"]
apiVersion: v1
data:
  # Configuration is in Json format. No matter what configure change you make,
  # please keep the Json blob valid.
  cwagentconfig.json: |
    {
      "structuredlogs": {
        "metrics_collected": {
          "kubernetes": {
            "cluster_name": "{{cluster-name}}",
            "metrics_collection_interval": 60
          }
        },
        "force_flush_interval": 5
      }
    }
kind: ConfigMap
metadata:
  name: cwagentconfig
  namespace: amazon-cloudwatch
[/bash]

<code>{{cluster-name}}</code>の部分を、適用するk8sクラスターのクラスター名に変更します。これにより、CloudWatch agentがEC2タグからクラスタ名をEC2タグから取得可能となります。

以下のコマンドで、ConfigMapを適用します。

[bash]
$kubectl apply -f cwagent-configmap.yaml
[/bash]

#### CloudWatch Agentをデーモンセットとして配置

最後に、CloudWatch agentを各ワーカーノードにデーモンセットとして配置します。

[bash]
$curl -O https://s3.amazonaws.com/cloudwatch-agent-k8s-yamls/kubernetes-monitoring/cwagent-daemonset.yaml
[/bash]

ファイルの内容はこんな感じ。

[bash title="cwagent-daemonset.yaml"]
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: cloudwatch-agent
  namespace: amazon-cloudwatch
spec:
  selector:
    matchLabels:
      name: cloudwatch-agent
  template:
    metadata:
      labels:
        name: cloudwatch-agent
    spec:
      containers:
        - name: cloudwatch-agent
          image: amazon/cloudwatch-agent:latest
          imagePullPolicy: Always
          #ports:
          #  - containerPort: 8125
          #    hostPort: 8125
          #    protocol: UDP
          resources:
            limits:
              cpu:  200m
              memory: 100Mi
            requests:
              cpu: 200m
              memory: 100Mi
          # Please don't change below envs
          env:
            - name: HOST_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.hostIP
            - name: HOST_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            - name: K8S_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          # Please don't change the mountPath
          volumeMounts:
            - name: cwagentconfig
              mountPath: /etc/cwagentconfig
      volumes:
        - name: cwagentconfig
          configMap:
            name: cwagentconfig
      terminationGracePeriodSeconds: 60
      serviceAccountName: cloudwatch-agent
[/bash]

適用します。

[bash]
$kubectl apply -f cwagent-daemonset.yaml
[/bash]

配置結果を確認します。

[bash]
$kubectl get pods -n amazon-cloudwatch
NAME                     READY     STATUS    RESTARTS   AGE
cloudwatch-agent-6dwzv   1/1       Running   0          14s
cloudwatch-agent-khmkx   1/1       Running   0          14s
cloudwatch-agent-kq57t   1/1       Running   0          14s
[/bash]

ひとまずここまでで、最低限のインストールは完了です。

## CloudWatchを確認してメトリクスとログを確認する

早速、CloudWatchを確認してみましょう。

ロググループに、以下のようなロググループが作成されていればOKです。

030.png

ドリルダウンすると、各ワーカーノード毎のログストリームが作成されています。

040.png

さらにドリルダウンすると、ワーカーノードで出力されたログが確認できます。

050.png

もちろん、CloudWatch Logs Insightsと統合されているため、クエリによるログの分析も可能です。

060.png

メトリクスももちろん取得可能。メトリクス直下にContainerInsightsのメトリクスが自動的に取得されています。

070.png

メトリクス名の「自動ダッシュボードの表示」から、各メトリクスの一覧をグラフ形式で表示可能です。

080.png

左上のドロップダウンからPodを選択すると…

090.png

<strong>Pod単位でのCPU、メモリ、ネットワーク関連のメトリクスを一括ダッシュボード表示可能です。</strong>いやぁこれは感動するわ。もちろん、このメトリクスからCloudWatch Alarmも設定が可能。

100.png

取得できるメトリクスやLogイベントは、下記ページに纏められています。合わせて参照ください。

- <a href="https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/Container-Insights-metrics.html" target="_blank">Metrics Collected by Container Insights - Amazon CloudWatch</a>
- <a href="https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/Container-Insights-reference-performance-entries.html" target="_blank">Relevant Fields in Performance Log Events - Amazon CloudWatch</a>

## FluentDを利用したコンテナログのCloudWatch Logsへの転送

ここまでで、基本的なメトリクスやインフラ関連のログがCloudWatch Logsに転送されているところまで確認しました。

ここからは、肝心要のコンテナログのCloudWatch Logsへの転送設定をやります。ここでは、FluentDを利用します。

- <a href="https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-logs.html" target="_blank">Set Up FluentD as a DaemonSet to Send Logs to CloudWatch Logs - Amazon CloudWatch</a>

この設定により、以下のロググループがCloudWatch Logsに作成されます。

|  **Log Group Name** | **Log Source** |
| :--- | :--- |
|  /aws/containerinsights/Cluster_Name/application | All log files in /var/log/containers |
|  /aws/containerinsights/Cluster_Name/host | Logs from /var/log/dmesg, /var/log/secure, and /var/log/messages |
|  /aws/containerinsights/Cluster_Name/dataplane | Logs from /var/log/journal |

### CloudWatch用のネームスペース作成

これはCloudWatchエージェント導入したときと同じです。既に導入してたら、不要です。

[bash]
$curl -O https://s3.amazonaws.com/cloudwatch-agent-k8s-yamls/kubernetes-monitoring/cloudwatch-namespace.yaml
[/bash]

[bash]
$kubectl apply -f cloudwatch-namespace.yaml
[/bash]

### FluentDのインストール

FluentDデプロイ用の設定ファイルをダウンロードします。

[bash]
$curl -O https://s3.amazonaws.com/cloudwatch-agent-k8s-yamls/fluentd/fluentd.yml
[/bash]

ログを送信するクラスターとリージョン名を設定して、configmapを作成します。<code>cluster_name</code>と<code>region_name</code>は、それぞれの環境に合わせて変更してください。

[bash]
$kubectl create configmap cluster-info \
--from-literal=cluster.name=cluster_name \
--from-literal=logs.region=region_name -n amazon-cloudwatch
[/bash]

上記コマンドでConfigmapの作成が完了したら、FluentDをデーモンセットとしてクラスターに設定します。

[bash]
$kubectl apply -f fluentd.yml
[/bash]

しばらく待って、各ノードにデーモンセットとして、以下のpodが起動しているのが確認できればOKです。

[bash]
$kubectl get pods -n amazon-cloudwatch
NAME                       READY     STATUS    RESTARTS   AGE
cloudwatch-agent-6dwzv     1/1       Running   0          4h21m
cloudwatch-agent-khmkx     1/1       Running   0          4h21m
cloudwatch-agent-kq57t     1/1       Running   0          4h21m
fluentd-cloudwatch-6v6cv   1/1       Running   0          9m31s
fluentd-cloudwatch-ccqzs   1/1       Running   0          9m31s
fluentd-cloudwatch-j7hzx   1/1       Running   0          9m31s
[/bash]

### CloudWatch Logsへのログ転送の確認

CloudWatch Logsコンソールを開いて、以下のロググループが作成されていればOKです。

110.png

ロググループの中に、アプリケーションログが転送されていることを確認できました。素晴らしい！

120.pngt




## まとめ「Container InsigtesでEKSのメトリクス監視〜ログ管理が超絶進化」

AWSの代表的なコンテナワークロードのECSやFargateは、既にCloudWatchと密接に統合されているため、ログの管理やメトリクスの取得は、他のAWSサービスと同じ手法で設定することができていました。はっきり言って設定はめっちゃ簡単です。

ただ、EKSについてはそのあたりFluentdを活用したり、別途監視用のSaaSを契約したり、PrometheusやGrafanaの導入が必要でした。

<strong>今回のContainer Insightsの発表により、サードパーティーのツールを組み合わせる面倒くささが解消され、慣れ親しんだCloudWathを使うことで、ログの管理、メトリクスの管理、アラームの設定が全て一括してマネージドサービスで完結します！
</strong>

設定も特段難しいことはなく、k8sに適用するマニフェストファイルがAWSより提供されるので、それを順番に<code>kubect apply</code>していくだけです。後は、ワーカーノードのEC2のIAMロールにポリシーを追加するぐらいですね。

2019年5月21日時点では、まだパブリックベータという位置付けですが、従来のEKS運用の面倒くささを大きな部分で解消するでかいアップデートなので、まずは手元の検証環境にいれて動作確認してみてください。

やっぱり、AWSのサービスはマネージド統合されてこそですから。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。
