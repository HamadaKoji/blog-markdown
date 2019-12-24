EKSは本当にECSより難しいのか？

個人的な感触ですが、ECSとEKSを比べてみた場合、ものすごい単純に言って以下の感想でした。

- 難易度：ECS > EKS
- 機能性：ECS < EKS

よく考えるとこの難易度という単語、簡単に使ってしまいがちなんですが使い方を間違えると非常に危険な言葉だとも思ってます。それは「誰にとっての難易度か？」という観点が抜け落ちがちだから。

AWSにおいてコンテナワークロードを展開しようと検討する人は様々いると思いますし、その組織や担当者の技術スタックも千差万別でしょう。kubernetesの経験は豊富だけれどAWSの経験はあんまりない、またはその逆、もしくはDockerは散々さわっているけれど、オーケストレーションツール自体が初めての人など。

自分の現場経験はECSのほうがEKSより圧倒的に多く、今でも「EKSってやっぱりなんか敷居高いなぁ」と感じてます。でも、よくよく思い出すと最初ECSを触り始めたときもこんな愚痴を言ってたんですね。

- 「え？なにこのECSのコンソールわかりにくくね？」
- 「タスク定義、なにこれパラメータ多くてなんやねんこれ」
- 「サービスってなに？タスク実行じゃあかんの？てか、設定項目大杉」

<strong>ぶっちゃけ、ECSだって敷居は低くないだろうと。</strong>というところで、今一度、ECSとEKSをいくらかの観点で比較しながら、EKSが本当にECSに比べて難しいのか？難易度が高いのか？という点を<strong>主観丸出し</strong>で語っていくのが、「<a href="https://qiita.com/advent-calendar/2019/amazon-eks" target="_blank">Amazon EKS Advent Calendar 2019</a>」15日目のこの記事です。

<pre style="line-height:120%;">
ﾊﾏｺｰﾎﾟｴﾑﾀｲﾑきたか…!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>


## クラスターの運用管理の違い

ECSとEKSのクラスターそのものの特性を比べてみます。どうやって比べたもんかと思いましたが、とりあえずAWS CLIの<code>create-cluster</code>コマンドを比較して、どのようなオプションがあるか調べてみました。

<code>aws ecs create-cluster</code>のドキュメントはこちら。

<a href="https://docs.aws.amazon.com/cli/latest/reference/ecs/create-cluster.html" target="_blank">aws ecs create-cluster</a>

主なプロパティはたった3種類。改めてめっちゃシンプル。クラスター名さえ必須ではないという潔さ。

- （必須）
  - なし
- （任意）
  - クラスター名
  - Container Insightsの有効化
  - Capacity Provider設定

Capacity Providerは先日のre:Invent 2019において発表されたホットな機能で、クラスター内タスクが起動するインフラ環境を定義できます。

- <a href="https://dev.classmethod.jp/cloud/aws/regrwoth-capacity-provider/" target="_blank">Capacity Providerとは？ECSの次世代スケーリング戦略を解説する</a>

対して、<code>aws eks create-cluster</code>のドキュメントはこちら。

<a href="https://docs.aws.amazon.com/cli/latest/reference/eks/create-cluster.html" target="_blank">create-cluster</a>

主要なプロパティは以下の通り。ECSに比べて必須項目が多く、またEKSを運用する上で宿命のkubernetesバージョンも指定可能です。

- （必須）
  - クラスター名
  - ロールARN
  - リソースVPC設定
- （任意）
  - クラスターのロギング設定
  - クライアントリクエストトークン
  - <strong>kubernetesバージョン</strong>

以下、クラスター管理の点から、ECSとEKSを比較していきます。

### クラスター作成速度の違い

ECSクラスターの作成は爆速で、だいたい2秒ぐらいで完了します。

[bash]
$ time aws ecs create-cluster
{
    "cluster": {
        "clusterArn": "arn:aws:ecs:ap-northeast-1:629895769338:cluster/default",
        "clusterName": "default",
        "status": "ACTIVE",
        "registeredContainerInstancesCount": 0,
        "runningTasksCount": 0,
        "pendingTasksCount": 0,
        "activeServicesCount": 1,
        "statistics": [],
        "tags": [],
        "capacityProviders": [],
        "defaultCapacityProviderStrategy": []
    }
}

real    0m1.285s
user    0m0.396s
sys     0m0.129s
[/bash]

対してEKSクラスターの作成は、既にネットワーク周辺のリソースが出来上がっている状態で、単独でクラスター作成しても、おおよそ5分程度はかかります。

### 料金面の違い

ECSクラスター自体にはお金かかりません。無料です。対して、EKSは、0.20USD/時間。1ヶ月で144USD。

プロダクション運用で使う場合どってことない金額のように思えますが、個人検証で使うには正直つらい金額です…　Workerノード（EC2インスタンス）にお金がかかるのは当然までも、どうしても「ECSに比べてなぁ…」と感じてしまうポイントだったりします。

個人でちょっと試そうとしたときのハードルの低さは、そのままその技術にふれるエンジニアの裾野を広げる効果があり、採用事例や情報発信の増加に間接的に結びついていくと思うので、これまで数々の値下げを断行してきたAWSさん、近いうちに無料になることを願っております。

### ネットワーク面の違い

ECSでは、Fargateなどのawsvpcネットワークモードを使えばタスクにそのままENIをアタッチできるため、<code>run-task</code>や、<code>create-service</code>など、タスクの起動時に任意のネットワーク（VPC）で起動することができます。

対して、EKSのクラスター作成においては、そのクラスターで利用するVPCやネットワークなどの初期設定が必須となっており、あとからの変更はできません。

また、EKSで利用するVPCは、基本的にNAT GATEWAYを利用したプライベートサブネットの構成がほぼ必須であったり、各種リソースへのタグ付が必要であったり制約が多いです。

このあたり、クラスター以外で利用するリソース（VPCやEC2インスタンス）を自前で構築するのは結構面倒くさく、EKSリリース当初は専用のCloudFormationを自分で実行する手順になっていて「まじかよ…」とひいた思い出があるんですが、今は<a href="https://github.com/weaveworks/eksctl/" target="_blank">eksctl</a>というツールが、AWS公式として広く使われています。

eksctl、非常に便利なツールなのですが、Workderノードなどの全ての設定をこれでまかなえるわけではないため、eksctl管理できないところは、Webコンソールから修正するなどの工夫が必要になったりします。

### クラスターバージョンの概念の違い

ECSはそのクラスターのバージョンをユーザーが知ることはできません。<a href="https://docs.aws.amazon.com/cli/latest/reference/ecs/describe-clusters.html" target="_blank">describe-clusters</a>の出力結果にバージョンの記載はありません。

コントロールプレーンとしてのECSの歴史は非常に長く膨大な機能追加が繰り返されてますが、ECSのメンテナンスによるダウンタイムなどは皆無で、ユーザーが気にせず使えるのは素晴らしい点ですね。

対して、EKSはkubernetesを扱っている宿命上、kubernetesのバージョンアップデートに必ず追随する必要があります。具体的にはEKSでサポートされないkubernetesバージョンをもつクラスターは、強制的にクラスターバージョンがアップデートされます。

<a href="https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/kubernetes-versions.html" target="_blank">Amazon EKS Kubernetes バージョン - Amazon EKS</a>

AWSは後方互換がないアップデートをすることは非常に稀で、それはAWSのフルマネージドサービスとして提供されているECSにもあてはまり、基本的にECSにおいて内部的なクラスターや機能のアップデートによって使えなくなる機能は存在しません。

対して、EKSはkubernetes互換であることが宿命付けられており、kubernetesは普通に既存機能に影響があるアップデートが行われます。<strong>大雑把に言えば、同一のバージョンを1年以上使うとサポートが切れます。</strong>

EKSクラスターをバージョンアップするAPIもありますが、検証環境での事前確認は必須ですし、本番のバージョンアップ作業も失敗したときの切り戻しを考えると、ブルーグリーンでクラスターを別途用意しておいて、DNSレベルで切り替えるぐらいの対応は必要になるかと思います。

もちろん、エンドツーエンドの検証に必要なリソースはEKSやそのWorkderノードだけではなく、大抵はRDS（DB）やElastiCacheやSQSなど周辺リソースもあるのが普通で、それらインフラを構築〜運用〜管理するため、EKSの運用には現実的にIaCの導入が必須と言えます。

### クラスター運用のECSとEKSのそれぞれの印象

自分の頭の棚卸しを兼ねて、ECSとEKSのクラスターをいくらかの面で比較してみました。まぁなんとなくはわかっていましたが、全体的にEKSの方が考慮事項が多く扱いが難しい面があること、おわかりいただけたかと思います。

特に、ECSはそのバージョンを全くユーザーに感じさせないのに対して、EKSではクラスターバージョンアップの運用が重くそれが必ず必要となるので、そこの検討は必ず必要となります。

## コンテナ環境の定義・プロビジョニング手法について

実際にクラスターを構築したあと、データプレーンであるコンテナ環境を定義していくわけですが、このプロビジョニング手法もECSとEKSで大きく違います。

### ECSにおけるコンテナ環境の定義方法

タスク定義とサービス定義で構築します。ECSのタスク定義では、複数コンテナを組み合わせたコンテナ実行の最小単位を指定し、kubernetesのPodに相当するものです。サービス定義は、そのタスク定義のネットワーク周辺やスケーリングポリシーなどを定義します。

AWSのフルマネージドサービスとしてのECSの特徴ですが、このタスク定義とサービス定義は、完全独自仕様で、共にJSON、もしくはClouFormationなどで定義します。

この時代にJSONかよ…とひく人も多いのですが、AWSのある程度構造化されたリソースはJSONで表現されることが普通でそれにそっていると言えばそれまでなんですが、この独自仕様のJSONの学習〜運用管理が煩わしく思う人も多いんじゃないでしょうか。

特にローカル環境での開発のために、Docker ComposeとECSタスク定義の2重管理をしている現場も多いと思います。

<a href="https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/ECS_CLI.html" target="_blank">ecscli</a>などを利用すると、Docker Compose構文でECS環境のプロビジョニングが可能だったりしますが、ECSの全ての機能に対応しているわけではありません。逆にecscliの機能で、ECSタスク定義をDocker Compose仕様に変更する機能がリリースされていたりします（参考：<a href="https://dev.classmethod.jp/cloud/aws/ecs-local/" target="_blank">ECSタスク定義を利用したローカル環境でのテスト実行が可能に！</a>）

#### ECSサービス定義の設定項目の多さ

ECSサービスは、実際にコンテナをクラスター上で管理〜展開するためにほぼ必須の機能なんですが、このECSの<a href="https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/service_definition_parameters.html" target="_blank">サービス定義パラメータ</a>が、往年の機能拡充に伴いめちゃくちゃ増えています。

- ネットワーク設定
- サービススケジューラ
- ロードバランサーの登録
  - ターゲットグループを複数登録できるようになってコンソールのUIがえらいこっちゃです
- オートスケーリングポリシー
- デプロイ設定
- サービスディスカバリ設定
  - 非常に便利だけれど、デフォルトでONになっている必要あるのか？

非常に多機能で明らかに便利にはなっているのですが、その分Webコンソールで最初定義するときの設定項目の多さは、初学者には敷居が高くなってるなぁとも感じます。

#### ECS、Webコンソールの複雑さ

コンテナ環境のプロビジョニングはコードベースでやりつつも、実際に稼働したあとのクラスターの状態を見るとき、Webのコンソールは強い味方です。

ECSのWebコンソール非常に、非常によく出来てると思います！自分も今以上のUIが考えられるかというと無理なんですが、その多機能さ故の「これ、今どこ見たらええの？」的な難しさを感じます。

ECSサービス内で動くタスク、それ以外も含めた全体のタスク一覧、クラスター上のホストインスタンス、スケジュール登録されたタスク、さらに最近華々しく登場したCapacity Providerも絡んできて、見れる箇所が多いんですね。ここに関しては、触って慣れる以外に無いですが、初学者にとっては辛くなっているなとも感じます。

### EKSにおけるコンテナ環境の定義方法

当たり前ですが、EKSはPodの作成もネットワークへの展開もスケジューラーの登録もkubernetesのマニフェストファイルが中心です。この一貫性、オープンなkubernetesの仕様をそのまま使える点は、kubernetesに慣れた人にとっては非常に親和性が高く、受け入れられやすい点かと思います。

EKSを採用されるお客様から、その採用の理由を聞いたとき「もともとKubernetesは使っていて慣れていた」という声が返ってくることも非常に多いです。

WebコンソールもEKSについてはクラスターのベースの部分の情報が表示されているのみで、実際的にクラスターの状態を確認するには、kubernetesで標準提供されているダッシュボードを使うことが多いです。これはこれでなんとも潔い仕様ですね。

### コンテナ環境のプロビジョニングの難易度は？

ECSもkubernetesもどちらも知らない人が、じゃぁこれからコンテナ環境構築していく上での難易度を比較した場合、<strong>ぶっちゃけ「どっちもそれなりに大変」</strong>というのが率直な感想です。

やらないといけないことには基本的に変わりがなく、それを実現する主な手段がECSの独自仕様かkubernetesの仕様なのかの違いで、特にECSが簡単でEKSが難しいということも無いんじゃないでしょうか。現実的に複数環境を管理しようとした場合、EKSではKustomizeの導入などが必要になってきますが、そのあたりの考慮点の多さはECSでも特に変わらないです（考える必要あり）。

## CI/CDについて

コンテナがもつ可搬性、デプロイの速さのメリットを最大限活かすため、コンテナワークロードにおいてCI/CDの整備はほぼ必須事項でしょう。というか、やらなきゃもったいない！すよね。ECSとEKSにおいて、現状どんな手法がとり得るかか考えてみます。

### CI（コンテナビルド→レジストリ登録）

AWSのコンテナワークロードにおいては、ECRを使うことはほぼ必然でしょう。AWSの中で使う分にはアクセス制御もやりやすく可用性も高い（S3相当）ため、便利に使えます。

ビルド手段としてCodeBuild使ってもいいですし、CircleCIや、最近ならGitHub Actionsを使うのもありですね。ここについてはECSとEKSで差はありません。

### ECSのコンテナデプロイ

ECSの特徴は、CodePipelineからのECSサービスへのデプロイが、マネージド統合されている点です。とりあえずローリングアップデートでよければ、CodePipelineからECSサービスに直接デプロイでき、この場合、タスク定義の変更→サービス定義の変更も自動的に行われます。また、CodeDeployを使ったB/Gデプロイも可能で、このあたりマネージドで作成〜管理できるのはありがたいですね。

### EKSのコンテナデプロイ

EKSにはCodePipelineとの統合機能は全くありません。ので、Kubernetesの世界で必要となるデプロイツールを利用することになります。もちろん、<code>kubectl apply</code>を手動実行する方法もありますが、事故の元になりうるので何らかのツールの導入が必要になるでしょう。自分がしる範囲では、<a href="https://www.spinnaker.io/" target="_blank">Spinnaker</a>や<a href="https://argoproj.github.io/argo-cd/" target="_blank">ArgoCD</a>、<a href="https://github.com/fluxcd/flux" target="_blank">flux</a>などの名前をよく聞きます。

### CI/CDの難易度の違いは？

特にこの点でのECSとEKSの難易度の違い（習得する必要がある技術スタックの多さ）は、あまり変わらない気がします。マネージド統合されている時点で、ECSに分があるかと考えていましたが、そっちはそっちでCodePipelineやCodeDeployについて習熟する必要があるので、EKSに比べて特段楽ということも無いように感じます。ただ、運用上はECSクラスターの状態とCode系シリーズを同じUIで見ることができるECSの方が、一定のアドバンテージがありそうです。

## 監視について

これについては、ECSもEKSも、CloudWatch Container Insightsの登場で大きく変わりました。以前は、メトリクスをコンテナ単位の粒度で見ようとすると、サードパーティ製のSaaS（DatadogやMackerel）の導入がほぼ必須でしたが、それがCloudWatchだけでも一通りまかなえるようになっています。

<a href="https://dev.classmethod.jp/cloud/aws/container-insights-ga/" target="_blank">ECSやEKSのメトリクスを一括取得するContainer Insightsが一般公開！既存ECSクラスタも追加設定可能に！</a>

Container Insightsは2019年9月にGAとなった機能で、ECSの場合クラスターでこれを有効化しておくだけ、EKSではWorkerノードにエージェントコンテナをDaemonSetとして展開することで、ほぼ自動的にクラスター内の各コンテナやホストインスタンスのメトリクスを取得できます。

以前は、ECSでもコンテナ単位でのメトリクスの取得には<a href="https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/task-metadata-endpoint.html" target="_blank">タスクメタデータエンドポイント</a>から自前で、各種メトリクスデータを取得する必要がありました。

収集できるメトリクスはこちらを参考ください。

- <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-metrics-ECS.html" target="_blank">Amazon ECS Container Insights Metrics - Amazon CloudWatch</a>
- <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-metrics-EKS.html" target="_blank">Amazon EKS and Kubernetes Container Insights Metrics - Amazon CloudWatch</a>


## まとめ「EKSはECSより難しいのか？」

ここまで主観丸出しでだらだらと書いてきましたが、ざっくり自分の印象をまとめてみます。

|  比較項目 | ECS | EKS | 比較結果 |
| --- | --- | --- | --- |
|  クラスターの運用 | バージョンの概念無 | kubernetesバージョンアップへの追従が必須 | EKSが大変 |
|  コンテナ環境のプロビジョニング | ECS独自仕様への習熟が必要 | kubernetesの仕様準拠（一部AWS独自） | どっちもあんまり変わらない |
|  CI/CDの構築 | Codeシリーズで実装 | 自前実装 | CIは同じ。CDはECSが比較的楽 |
|  監視 | Container Insights | Container Insights | どっちもあんまり変わらない |

やっぱり総体的にみて「EKSのほうが大変」という印象は変わらないですね。こんだけ長々と書いて結局それかよ！って感じですが、EKS素人の正直な気持ちです。

どこまでいっても「クラスターの運用の大変さ」が、あらゆるところに効いてきます。EKSには、CNCFでホストされているkubernetes関連の様々なツールが使える大きな利点がありますが、それらはAWSマネージド統合されていないため、kubernetesのバージョンアップ時には必ず全て検証が必要になり、導入するツールが多ければ多いほど、その検証も大変になっていく印象です。ECSと比べて一番違うところじゃないでしょうか。

### 個人的なEKSに対する思い入れ

ただ、世の中の「kubernetesが難しい」という印象ですが、語られる文脈として「kubernetes（をがんがん活用して運用する大規模なマイクロサービスの運用が）難しい」という側面も多い気がするんですよね。そもそも構成要素が膨大だから難しいというか。このイメージが先行しすぎて「とてもじゃないけど、うちの組織にはいらないし学ぶ価値もない」と思う人が多いというか。

実際構築するときのコンテナ環境を作ったりCI/CDを整備するために必要な技術知識は、上の比較でも書いたとおりECSとEKSであんまり差が無い印象です。

ので、もっと気軽にEKS使ってみても良いとおもうんですよ。最初は1ノード、2Podとかで簡単なWebサービスを動かしてみるのもありかと思いますし、別に落ちても問題にならない社内利用オンリーのところで実験的に立ててみて、メンバーから「落ちとるで、どないなっとんねん」「まじか、なんでやろ」とワーワーやりながら感触をつかむといった使い方もありかと思います。スケジューラ登録してスクレイピングするとか、いろんなネタがありますね。ロードバランサーもCLBとかでええやん。

あと、やっぱりKubernetes触るの楽しいすよ。これはアプリエンジニアでもインフラエンジニアでもそうだと思います。

そういった中で、敷居が高そうなEKSにふれることで「お、案外こいつええやつやん。もうちょっと深堀りして、今のアプリケーションEKSで使えそうか検証してみよか」と思う人が増えてくると、もっとEKSに対する誤解や変な敷居の高さも減って、導入事例ももっと増えてくると思ってます。草の根活動的なね。

そういう意味では、しつこいけど「EKSクラスター料金のさらなる値下げ（もしくは無料化！）」は、そういった流れを後押しすると思っているし、是非実現して欲しいなと感じています。

最後クレクレ君みたいなったな。まぁええか。今日はポエムや。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

