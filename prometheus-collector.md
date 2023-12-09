# フルマネージドでエージェントレスなPrometheus用のCollectorが発表されました！　#AWSreInvent

皆さん、EKS環境のメトリクス可視化にManaged Service for Prometheus使ってますか？従来は、Prometheus用のメトリクス収集のためにエージェントの導入が必須だったのですが、今回のアップデートで、フルマネージドなエージェントレスコネクターが発表されました。アップデートの概要と、全体的な設定手順を実際にやってみた様子をお届けするので、このアップデート気になっていた方は要チェックやで！！

<pre style="line-height:120%;">
なんか、楽なのきたか…!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>

一部制約もあるので要注意やで。

## アップデートの概要

アップデートの公式情報はこちら。

[Amazon Managed Service for Prometheus launches an agentless collector for Prometheus metrics from Amazon EKS](https://aws.amazon.com/jp/about-aws/whats-new/2023/11/amazon-managed-service-prometheus-agentless-collector-metrics-eks/)


<blockquote>a fully-managed agentless collector customers can use to collect Prometheus metrics from their workloads running on Amazon EKS. Customers can now enable the discovery and collection of Prometheus metrics from their Amazon EKS applications and infrastructure through the EKS console or through an API call, without having to self-manage agents.<br />（以下、日本語訳）<br />Amazon Managed Service for Prometheus collectorは、Amazon EKS上で実行されているワークロードからPrometheusメトリクスを収集するためにお客様が使用できるフルマネージドエージェントレスコレクターです。お客様は、エージェントを自己管理することなく、EKSコンソールまたはAPIコールを通じて、Amazon EKSアプリケーションとインフラストラクチャからPrometheusメトリクスの検出と収集を可能にすることができます。</blockquote>

以前は、EKSワークロードからのPrometheusメトリクスの収集には、個別にPrometheusのエージェントをインストールする必要がありましたが、今回のアップデートで、このエージェントをユーザー側で管理することなく、マネージドな仕組みでエージェントレスでPrometheusのメトリクス収集ができるようになりました。

詳細は、以下の公式ドキュメントを参照。

[AWS managed collectors \- Amazon Managed Service for Prometheus](https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-collector.html)



### 利用可能なリージョン

Amazon Managed Service for Prometheusが利用可能な全てのリージョンで、利用可能です。

### 主な制約

agentless collectorには、以下の公式ドキュメントに記載の通りの制約があります。

[Scraper limitations](https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-collector-how-to.html#AMP-collector-limits)

- リージョン
  - EKSクラスタ、Amazon Managed Service for Prometheus スクレイパー、Amazon Managed Service for Prometheusワークスペースはすべて同じAWSリージョンにある必要があります
- アカウント
  - EKSクラスタ、Amazon Managed Service for Prometheus スクレイパー、Amazon Managed Service for Prometheusワークスペースはすべて同じAWSアカウントである必要があります
- コレクター
  - Amazon Managed Service for Prometheus スクレイパーは、アカウントごとにリージョンあたり最大 10 個まで持つことができます


## agentless collectorを使ってみる。

というわけで、実際にagentless collectorをセットアップして使ってみます。各ツールのバージョンは以下の通り。

[bash]
$ aws --version
aws-cli/2.14.5 Python/3.11.6 Darwin/22.5.0 exe/x86_64 prompt/off
$ kubectl version --client
Client Version: v1.28.3-eks-e71965b
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
$ eksctl version
0.165.0
[/bash]

### Prometheusワークスペースの作成

最初に、新規でPrometheusワークスペースを作成しておきます。以下では、ワークスペース名を<code>test-managed-collector-workspace</code>で作成します。

[bash]
$ aws amp create-workspace --alias=test-managed-collector-workspace
[/bash]

### EKSクラスターの作成

東京リージョンに、<code>test-managed-collector</code>という、EKSクラスターを作成します。AWSマネジメントコンソールのEKSクラスター作成ウィザードに、Managed Collectorの設定が既に追加されているので、ここでは、マネジメントコンソールからクラスターを作成します。

マネジメントコンソールからEKSにアクセスし、[Add Cluster] -> [Create]。

- Name:<code>test-managed-collector-cluster</code>
- Kubernetes version:<code>1.28</code>
- Cluster service role:既定のものを選択

[Next]をクリックすると、Specify networking画面が表示されます。ここはデフォルトのまま、[Next]をクリック。

Configure observabilityの画面が表示されます。ここで、メトリクスに関する設定を実施します。

- Prometheus: [Send Prometheus metrics to Amazon Managed Service for Prometheus]をクリック
- Scraper alias:　デフォルトのまま
- Destination: 作成するScraperの接続先を指定します。[Select existing workspace]をクリックし、前の手順で作成したPrometheus Workspaceを選択

その他設定はデフォルトのまま、[Next]ボタンをクリック。Select add-ons画面が表示されるので、デフォルトのまま（kube-proxy、Amazon VPC CNI、CoreDNS、Amazon EKS Pod Identity Agentが選択）で、[Next]ボタンをクリック。

Configure selected add-ons settings画面はそのままで、[Next]ボタンをクリック。

Review and create画面で、[Create Cluster]ボタンをクリック。クラスターが作成されるのを待ちます。無事、StatusがActiveになればOK。


### Prometheusのscraper設定ファイルの作成

Managed Collectorの設定前に、Prometheus用のscraperファイルを作成しておきます。ここでは、Amazon Managed Service for Prometheus API の GetDefaultScraperConfigurationで提供されている内容をbase64デコードして、<code>default.yml</code>に書き出しておきます。一旦、設定はこのデフォルトファイルをそのまま利用します。

<code>get-default-scraper-configuration</code>APIは、base64エンコードされた状態で出力されるので、base64からデコードして、ファイル保存します。

[bash]
$ aws amp get-default-scraper-configuration --output text | base64 -D > default.yml
[/bash]

ファイル<code>default.yml</code>の内容は、以下の通り。

[yaml title="default.yml"]
global:
  scrape_interval: 30s
  # external_labels:
    # clusterArn: <REPLACE_ME>
scrape_configs:
  # pod metrics
  - job_name: pod_exporter
    kubernetes_sd_configs:
      - role: pod
  # container metrics
  - job_name: cadvisor
    scheme: https
    authorization:
      credentials_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    kubernetes_sd_configs:
      - role: node
    relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - replacement: kubernetes.default.svc:443
        target_label: __address__
      - source_labels: [__meta_kubernetes_node_name]
        regex: (.+)
        target_label: __metrics_path__
        replacement: /api/v1/nodes/$1/proxy/metrics/cadvisor
  # apiserver metrics
  - bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    job_name: kubernetes-apiservers
    kubernetes_sd_configs:
    - role: endpoints
    relabel_configs:
    - action: keep
      regex: default;kubernetes;https
      source_labels:
      - __meta_kubernetes_namespace
      - __meta_kubernetes_service_name
      - __meta_kubernetes_endpoint_port_name
    scheme: https
  # kube proxy metrics
  - job_name: kube-proxy
    honor_labels: true
    kubernetes_sd_configs:
    - role: pod
    relabel_configs:
    - action: keep
      source_labels:
      - __meta_kubernetes_namespace
      - __meta_kubernetes_pod_name
      separator: '/'
      regex: 'kube-system/kube-proxy.+'
    - source_labels:
      - __address__
      action: replace
      target_label: __address__
      regex: (.+?)(\\:\\d+)?
      replacement: $1:10249
[/yaml]


### メトリクス収集用のscraperの作成

Amazon Managed Service for Prometheusのコレクターは、EKSクラスターからメトリクスを検出し収集するscraperで構成されます。scraper用の構成ファイルを作成します。前の段階で作成したscraper設定ファイルを利用して、クラスター用のscraperを設定します。

設定には、AWS CLIの<code>create-scraper</code>APIを利用します。

以下の変数を利用して、scraperを作成します。

- EKSクラスターのARN
- EKSクラスターのSecurityGroupID
- EKSクラスターのsubnetID
- Managed Prometheus WorkspaceのARN
- --scrape-configuration：前の手順で作成した<code>default.yml</code>をbase64エンコードして利用

EKSクラスターに設定されているSecurityGroupとSubnetIDとPrometheusのワークスペースARNを利用して、[aws amp create-scraper]コマンドでscraperを作成します。実際のコマンドは上記パラメーターで適宜書き換えて実行してください。

[bash]
aws amp create-scraper \
  --source eksConfiguration="{clusterArn='arn:aws:eks:us-west-2:account-id:cluster/cluster-name', securityGroupIds=['sg-security-group-id'],subnetIds=['subnet-subnet-id-1', 'subnet-subnet-id-2']}" \
  --destination ampConfiguration="{workspaceArn='arn:aws:aps:us-west-2:account-id:workspace/ws-workspace-id'}" \
  --scrape-configuration configurationBlob=$(cat default.yml | base64)"
[/bash]

うまく登録できると、このように作成されたScraperIdが表示されます。マネジメントコンソールでもクラスターの[Observability]タブに、作成したScraperが表示されています。現段階では、ロールの設定が完了していないため、StatusはCreatingとなっています。

010

### EKSクラスターの設定

公式ドキュメントの[Configuring your Amazon EKS cluster](https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-collector-how-to.html#AMP-collector-eks-setup)を参考に、EKSクラスターの設定を進めます。

以下の<code>clusterrole-binding.yml</code>ファイルを作成します。

[yaml title="clusterrole-binding.yml"]
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: aps-collector-role
rules:
  - apiGroups: [""]
    resources: ["nodes", "nodes/proxy", "nodes/metrics", "services", "endpoints", "pods", "ingresses", "configmaps"]
    verbs: ["describe", "get", "list", "watch"]
  - apiGroups: ["extensions", "networking.k8s.io"]
    resources: ["ingresses/status", "ingresses"]
    verbs: ["describe", "get", "list", "watch"]
  - nonResourceURLs: ["/metrics"]
    verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: aps-collector-user-role-binding
subjects:
- kind: User
  name: aps-collector-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: aps-collector-role
  apiGroup: rbac.authorization.k8s.io
[/yaml]

クラスターに適用します。

[bash]
$ kubectl apply -f clusterrole-binding.yml
[/bash]

前手順で作成したScraperのIDを利用して、Scraperの内容を確認します。

[bash]
$ aws amp describe-scraper --scraper-id <scraper-id>
[/bash]

出力内容から、<code>roleArn</code>を確認し、以下のコマンドで、ロールにIAMをマッピングします。

[bash]
$ eksctl create iamidentitymapping --cluster test-managed-collector-cluster --arn <roleArn> --username aps-collector-user
[/bash]

このコマンドで、scraperが、<code>clusterrole-binding.yml</code>で作成した、ロールとユーザーでクラスターにアクセスすることを許可します。この権限付与が正常に完了したら、このように先程<code>InActive</code>だったscraperのStatusが<code>Active</code>になります。

020.png

### メトリクス収集内容の確認

Managed Service for Prometheusでメトリクスが収集されているかどうか確認するのに、最も手っ取り早い方法は、<code>awscurl</code>を利用する方法です。

参考：[Query using Prometheus\-compatible APIs \- Amazon Managed Service for Prometheus](https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-onboard-query-APIs.html)

macの場合、以下のコマンドで、awscurlをインストール。

[bash]
$ brew install awscurl
[/bash]

先ほど作成したPrometheus WorkspaceのQUERYエンドポイントを取得します。<code><Workspace-id></code>を、皆さんの環境のPrometheus Workspaceの値に書き換えてください。このQUERYエンドポイントは、マネジメントコンソールからも確認できます。

[bash]
$ export AMP_QUERY_ENDPOINT=https://aps-workspaces.Region.amazonaws.com/workspaces/<Workspace-id>/api/v1/query
[/bash]

aws認証情報を保持したクライアントから、以下のコマンド<code>up</code>を実行し、何かしらのJSONが返却されればOKです！

[bash]
$ awscurl -X POST --service aps "$AMP_QUERY_ENDPOINT?query=up"

{
  "status": "success",
  "data": {
    "resultType": "vector",
    "result": [
      {
        "metric": {
          "__name__": "up",
          "instance": "localhost:9090",
          "job": "prometheus",
          "monitor": "monitor"
        },
        "value": [
          1652452637.636,
          "1"
        ]
      },
      
      
    ]
  }
}
[/bash]

Prometheusのクエリ式の詳細は、こちらの公式ドキュメントから確認。

[Querying basics \| Prometheus](https://prometheus.io/docs/prometheus/latest/querying/basics/)

もし、正常にメトリクスが収集できていない場合は、下記公式ドキュメントを参考にscraperの設定を見直してみてください。

[Troubleshooting scraper configuration](https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-collector-how-to.html#AMP-collector-troubleshoot)

### （オプション）Managed Grafanaを利用したPrometheusメトリクスの可視化

Prometheusのメトリクスは、実際にはGrafana上で可視化することが多いでしょう。まだGrafanaの環境を持っていない場合は、以下を参考に、Grafanaのセットアップと、上記Prometheus WorkspaceへのDataSource設定を実施してみてください。

- 公式：[Set up Amazon Managed Grafana for use with Amazon Managed Service for Prometheus \- Amazon Managed Service for Prometheus](https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-amg.html)
- ユーザー管理をIAM Identity Centerを利用する場合
  - 参考：[Amazon Managed Grafanaをセットアップして使ってみる \| DevelopersIO](https://dev.classmethod.jp/articles/amazon-managed-grafana-awssso/)
- ユーザー管理をSAML(Auth0)を利用する場合
  - 参考：[Amazon Managed Grafana（AMG）のワークスペースに管理者ユーザー/一般ユーザーがSAML（Auth0）でアクセスできるようにしてみた \| DevelopersIO](https://dev.classmethod.jp/articles/amazon-managed-grafana-with-auth0/)

## エージェントレスコネクターによる、さらなるEKS監視のPrometheusメトリクス利用の広がりに期待

以上、公式ドキュメントを紐解きながら、PrometheusのAWS managed collectorの設定方法を解説してきました。今までのhelmチャートを利用したNodeへのエージェント導入に比べて、作成するリソースがシンプルになり管理対象が減っていることが理解できたかと思います。

冒頭の公式情報の概要にも述べましたが、以下の制約がありますので、まずは皆さんの環境で導入の余地があるかどうか把握した後、開発環境で試してみていただければと思います。

[Scraper limitations](https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-collector-how-to.html#AMP-collector-limits)

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

