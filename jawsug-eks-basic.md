# EKS入門者に必須の「EKSの基礎」について登壇しました #jawsug_ct

先日、<strong>「EKS祭り」</strong>をテーマにJAWS-UGコンテナ支部 #16を開催しました。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://jawsug-container.connpass.com/event/160835/" target="_blank">JAWS-UGコンテナ支部 #16〜EKS on Fargateローンチ記念！EKS祭りだワッショイ - connpass</a>
</p>

EKS縛りというだいぶ濃いイベントの中で、自分はトップバッターで<strong>「今こそ振り返るEKSの基礎」</strong>と第して喋ってきたのでその内容をまとめます。基礎といえども普段隠れがちなEKSに関連するAWSリソースについてフォーカスを当てたある意味マニアックな内容だと思うので、EKS気になる方は是非ご覧ください。

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     EKS ﾀﾉｼｲﾖ ｺﾚﾏｼﾞﾃﾞ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>

## この記事の主旨

この記事では登壇当日に口頭で補足した部分や、各種資料へのアクセスをリンクにしてまとめています。<strong>スライドだけをみるよりこちらの記事のほうが情報量が多く内容としてもまとまっているため、この記事をベースに見ていただければと思います。</strong>

## 登壇概要：「今こそ振り返るEKSの基礎」

<script async class="speakerdeck-embed" data-id="71057eeab45e4cda9b06c7606ce6fff0" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

<blockquote>皆さん、EKS始めるときeksctl使ってますか？コマンド一発でAWS関連リソースやインフラ一式できあがる超便利ツールですが、逆に個別のAWSリソースがどのように作成されているか、クラスターに紐づくVPCやNodeにはどのような制約があるのか、IAM周辺がどのように処理されているのか、そのあたり隠蔽されて分かりづらい部分あるんじゃないでしょうか。<br /><br />
このセッションでは、改めてEKS周辺のAWSリソースがどのように関連して動作しているのかを振り返りつつ、今日のEKS祭りのためのベース知識を皆さんと共有できればと思います。</blockquote>

## EKSの基礎とは？

まず一番最初にみなさんと<strong>EKSの基礎</strong>という単語について共有させてください。EKSはざっくりいうとkubernetesのAWSマネージド・サービスですが、EKS=kubernetesではありません。じゃ、EKSの基礎とは何でしょうか？

ここで、自分がKubernetesを始めようと思ったときの流れを紹介します。こんな流れでした。

1. kubernetes触りたい
2. どこでやろ。やっぱり慣れてるからAWSつかうか
3. eksctl。便利。マジで神！
4. <code>kubectl apply</code>でコンテナできたで！環境できたのでkubernetes本の内容試してみるで！
5. kubernetesおもろいやん！ﾜｯｼｮｲﾜｯｼｮｲ

こんなイメージです。

09

でも、よくよく考えてみてください。eksctlって何やってると思います？

10

eksctl、みなさんご存知でしょうか。AWS公式のeksクラスター構築コマンドラインツールです。現在は<a href="https://www.weave.works/" target="_blank">Weaveworks</a>社がホストしています。eksctlの公式ページはこちら。マニュアルなども一式あります。

- <a href="https://eksctl.io/" target="_blank">Introduction | eksctl</a>

11

eksctlの強力さを一番現しているコマンドがこちら。

[bash]
$ eksctl create cluster
[/bash]

これにより、以下のAWSリソースが作成され、さらに<code>kubeconfig</code>まで自動的に更新されます。

- EKS ControlPlane
- VPC, InternetGateway, route table, subnet, EIP, NAT Gateway, security group
- IAM Role, Policynode group, Worker node（EC2）
- 〜/.kube/config

これだけのコマンドが、コマンド一発で即kubernetesの世界に足を踏み入れることができるんですね。ただ、これはこれで不安点はあります。

13

例えばこの状態で、EKS運用ででてくる以下のシチュエーションに対応できるでしょうか？

- クラスターのアップデート、実際どないしよ
- クラスター触れる人を増やしたい
- 既存のクラスターVPCに別のクラスターを接続させたい
- ノードグループを複数VPCに展開したい
- NAT Gateway高いからInternetGatewayだけにしたい
- コントロールプレーンとワーカーノードを別AWSアカウントで管理したい
- 2AZのVPCを3AZに拡張したい。もちろんノードも使いたい

はい、わからんですよね。ので、今日の趣旨はこちらです。こちらを皆さんに共有していければと思います。

18

## EKSの開始方法は２種類

公式ドキュメントに<a href="https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/getting-started.html" target="_blank">Amazon EKS の開始方法</a>があるんですが、開始方法は大きく2つあります。

1. <a href="https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/getting-started-eksctl.html" target="_blank">eksctl の開始方法</a>
2. <a href="https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/getting-started-console.html" target="_blank">AWS マネジメントコンソール の開始方法</a>

1は非常に簡単なのですが、今回はeksctlが実際に何をやっているのかを理解することが目的なので、「2 マネジメントコンソールの開始方法」を読みながら、ステップバイステップでポチポチと各AWSリソースを作ることをイメージしながら、解説していきます。

手順は大きく分けて5つ。

1. Amazon EKSサービスロール作成
2. EKSクラスターVPCの作成
3. EKSクラスターの作成
4. kubeconfigの設定
5. マネージド型ノードグループの起動

## ①Amazon EKS サービスロールの作成

一番最初にEKSクラスターが他のAWSリソースに対してアクセスするためのロールを作成する必要があります。必須のAWSマネージドポリシーは2つ。

### AmazonEKSServicePolicy

- <strong>EKSが</strong>Kubernetesクラスタを作成〜運用するためのポリシー
- 例）クラスターログの転送（logs:PutLogEvents）
- 例）VPCに作成するENI（ec2:CreateNetworkInterface）

### AmazonEKSClusterPolicy

- <strong>Kubernetesクラスタのコントロールプレーンが</strong>AWSリソースを操作するためのポリシー
- 例）autoscaling,ec2,elasticloadbalancing

両方とも同じロール内に適用するので、それぞれのポリシーの違いを意識することはあまりないですが、EKSというサービスとkubernetesクラスターをごっちゃにしないためにも、ここの違いを意識しておくことは重要です。イメージとしてはこんな感じです。

22

下記公式ドキュメントに、それぞれのポリシーの詳細が記載されているので、合わせて参照ください。

- 公式ドキュメント：<a href="https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/service_IAM_role.html" target="_blank">Amazon EKS サービス IAM ロール - Amazon EKS</a>

また、toriさんによるtwitterリプ合戦も参考になりますYO！

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">AmazonEKSClusterPolicy<br>AmazonEKSServicePolicy<br><br>これの違いがよーわからん。雰囲気勝負か。</p>&mdash; 濱田孝治（ハマコー） (@hamako9999) <a href="https://twitter.com/hamako9999/status/1219987231256924161?ref_src=twsrc%5Etfw">January 22, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


## ②EKSクラスターVPCの作成

EKSクラスターの作成前に、事前にクラスターで利用するVPCを作成しておく必要があります。<code>EKS CreateCluster API</code>の必須パラメータは下記3つ。VPCの情報が必要ですね。

- name：クラスターの名前
- role-arn：サービスロールのARN
- resources-vpc-config：クラスターが利用するVPCの情報

というわけで、クラスターVPCを作成する必要があるのですが、このEKSクラスターで利用するVPCにはEKS用の要件が必要です。

- 公式ドキュメント：<a href="https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/network_reqs.html" target="_blank">クラスター VPC に関する考慮事項</a>

この中に全て詳細に定義されているのですが、簡単に必要となる要件をまとめます。これが全て必須ではないですが、代表的なパターンです。

- 2つ以上のAZ
- パブリックサブネット：インターネットフェーシングのロードバランサーなどの設置
- プライベートサブネット：ワーカーノード用
  - アウトバウンドインターネットアクセスが必要→NAT Gateway
- VPC IP アドレス指定
- VPCのタグ付要件
- サブネットのタグ付要件
- 内部ロードバランサー用のプライベートサブネットタグ付け要件
- 外部ロードバランサー用のパブリックサブネットタグ付け要件

特にハマリポイントなのが、VPCの各種リソースに設定が必須なタグがあることです。これらのタグは、クラスターVPC作成用のCloudFormationを利用したときに勝手に付与されたりしますが、<strong>自分でVPCを作ったりあとからサブネットやAZを拡張したときにハマリポイントとなるので注意が必要です。</strong>

実際にVPCを作成するときには、公式のCloudFormationが提供されているので、構造が要件にあっていればそれを使うのも良いでしょう。

30


## ③EKSクラスターの作成

いよいよ、実際のEKSクラスターの作成です。ここは、マネジメントコンソールから指定するのがわかりやすいです。主な作成時のパラメーターは以下の通り。

- <strong>必須</strong>
  - Cluster name
  - Kubernetes version
  - Role：前で作成したEKSのサービスロール
  - VPC：前で作成したクラスターVPC
  - Subnets：上で指定したVPCのサブネットが全て表示されているので選択
- 非必須
  - Endpoint private access
  - Endpoint Public access
  - ログ記録

これにより、以下が実行されます。

33

作成されるセキュリティグループにも要件があります。基本的に上記手順によりEKSが自動的に必要なセキュリティグループを作成してくれますが、<strong>ネットワーク構成を変更したりクラスター管理しないリソース追加時に混乱しないように、以下のドキュメントも目を通しておくと良いです。</strong>

- 公式ドキュメント：<a href="https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/sec-group-reqs.html" target="_blank">Amazon EKS セキュリティグループの考慮事項 - Amazon EKS</a>

### EKSコントロールプレーンとVPCの関係

上記手順により、VPCとEKSコントロールプレーンが紐づくのですが、コントロールプレーンとVPCの関係は複数パターンあるので、それを整理するとこうなります。ｌ

34

真ん中の複数コントロールプレーンから同一のVPCに接続するパターンですが、VPCの中にRDBがいる状態で、クラスターのバージョンアップをB/G方式で実行するときなどに使います。

## ④kubeconfigの設定

kubeconfigファイルとは、kubernetesを扱うためのCLIであるkubectlコマンドの接続先クラスターを管理する設定ファイルです。これが設定されていないと、先ほど作ったEKSクラスターに接続できません。

37

ただ、それを一瞬でやってくれるコマンドが存在します。

[bash]
$ aws eks update-kubeconfig –name cluster-name
[/bash]

これにより、kubeconfigファイルにコンテキストが追加され、また認証情報取得用のコマンドも設定されます。<strong>本当に便利です。これ。</strong>

[yaml title="~/.kube/config"]
- context:
    cluster: arn:aws:eks:ap-northeast-1:629895769338:cluster/test-fargate
    user: arn:aws:eks:ap-northeast-1:629895769338:cluster/test-fargate
  name: arn:aws:eks:ap-northeast-1:629895769338:cluster/test-fargate
current-context: arn:aws:eks:ap-northeast-1:629895769338:cluster/handson-eks-cluster
kind: Config
preferences: {}
users:
- name: arn:aws:eks:ap-northeast-1:629895769338:cluster/test-fargate
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1alpha1
      args:
      - --region
      - ap-northeast-1
      - eks
      - get-token
      - --cluster-name
      - test-fargate
      command: aws
      env: null
[/yaml]

これにより、ようやく<code>kubectl</code>コマンドでクラスターに接続できます。

[bash]
$ kubectl get service
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.100.0.1   <none>        443/TCP   10h
[/bash]

ただ、まだワーカーノード（EC2）はありません。

[bash]
$ kubectl get nodes
No resources found.
[/bash]

## ⑤マネージド型ノードグループの起動

### ワーカーノード用IAMロールの作成

いよいよ最後の手順、コンテナを動かすためのワーカーノード（EC2）を作っていきます。予めワーカーノードに設定するIAMロールを作成しておきます。このIAMロールには以下のポリシーが含まれます。

- AmazonEC2ContainerRegistryReadOnly
  - ECR参照用
- AmazonEKSWorkerNodePolicy
  - ワーカーノードからのEKSクラスターアクセス用
- AmazonEKS_CNI_Policy
  - VPC CNI Plugin用ポリシー
  - ワーカーノードによるネットワーク設定変更用

この3つ目、CNIプラグインですが、ワーカーノードのVPC IPアドレスをポッドのIPアドレスに割り当てるための機構です。このプラグインによりAWSのネットワークとkubernetesのネットワークが紐づくわけです。あまり意識することは無いかもですが、裏でこのプラグインが動いているということは頭の片隅にいれておいてください。

- 公式マニュアル：<a href="https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/pod-networking.html" target="_blank">ポッドネットワーキング (CNI) - Amazon EKS</a>

### マネージド型ノードグループの起動

これも、2019年11月のアップデートで非常に簡単になりました。Webコンソール上からポチポチすれば、ワーカーノードがグループとしてクラスターに認識され起動します。主なパラメータは以下の通り。

- IAMロール
- ワーカーノード展開対象のサブネット
- リモートアクセス可否
- コンピューティング構成設定（AMIタイプ、インスタンスタイプ、ディスクサイズ）
- スケーリングポリシー

しばらく待って、ノードグループがACTIVEになったら、<code>kubectl get node</code>を叩いてみましょう。無事、ノードが認識されたら、めくるめくkubernetesの世界にたどり着いた証です！

[bash]
$ kubectl get node
NAME                                                 STATUS    ROLES     AGE       VERSION
ip-192-168-148-252.ap-northeast-1.compute.internal   Ready     <none>    6m        v1.14.7-eks-1861c5
ip-192-168-213-27.ap-northeast-1.compute.internal    Ready     <none>    6m        v1.14.7-eks-1861c5
[/bash]

## ハマコーオススメのEKS学習マテリアル

最後に、皆さんにオススメの学習マテリアルを紹介します。それがこちら。

49

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://qiita.com/advent-calendar/2019/amazon-eks" target="_blank">Amazon EKS Advent Calendar 2019 - Qiita</a><br />
<a href="https://qiita.com/advent-calendar/2019/amazon-eks-2" target="_blank">Amazon EKS #2 Advent Calendar 2019 - Qiita</a>
</p>

2019年12月の奇跡と思ってます。EKSの最新事情や現場でのツラミ、一歩深い知見を得るには最高の記事が凝縮しているので、一度教科書的な部分で知識を蓄えた後は、この記事をみて学ぶのをオススメします。

## セッションまとめ

最後に、今日のセッションのまとめです。

- EKSを長きに渡って運用するには、kubernetesの知識以前に、各AWSリソースがどのように連携して動作しているのか知ることが不可欠
- 公式マニュアルの「コンソールでの開始」を元にじっくりポチポチ始めてみよう
- Amazon EKS Advent Calendar 2019は宝の山

今日お集まりのみなさんは、EKSの関わり方や経験は千差万別だと思います。既にガンガンにEKS使っていてその酸いも甘いも知り尽くしている人から、これからEKSを始めようとして情報収集しにこられた方など。ただ、一点共通するのは<strong>「EKSに何かしらの思い入れがある」</strong>というところですね。

これから、EKSに長く関わっていく方も多いと思いますが、是非そんな方々に向けて自分の今日の発表内容が活きれば幸いです。以上、ご清聴ありがとうございました。

## ハマコー登壇後日談

20分弱ぐらいの登壇時間でしたが、自分の伝えたいことは概ね伝えられたのかなと思います。正直自分の準備不足と消化不良の部分が残っている状態の登壇だったので、途中説明を中途半端に端折ったり喋りが上滑り感があったと自己反省しております。

ただ、今セッション内容を改めてまとめながら、このEKSの基礎を理解することのEKS運用における重要性やEKSとkubernetesの位置づけなど再度整理できたので良かったです。

EKSもkubernetesも学習しないといけないこと山のようにありますが、それだけエキサイティングで応用範囲も広い技術だと思うので、これからもこの界隈追っていこうと思います。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

