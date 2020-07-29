# （仮）組織最適化されたECSデプロイを考える（最新機能とともに）

URL https://dev.classmethod.jp/articles/ecs-deploy-all/



<strong>「ECSデプロイの話だけで45分喋った男がいた…」</strong>

というわけで、先日、<a href="https://classmethod.jp/m/devio_2020_connect/" target="_blank">Developers.IO 2020 CONNECT</a>において、以下のタイトルで喋りました。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
「あなたの組織に最適なコンテナデプロイ方法とは？〜ECSにおけるデプロイ最新機能てんこ盛り〜」
</p>

オンラインセッションは何度か登壇経験あったのですが、今回は45分。<strong>正直めっちゃくちゃ疲れました。</strong>いやぁ、登壇ってもしかしたら、リアルよりもオンラインのほうがつかれるかもしれません。

そんな登壇だったわけですが、内容あれこれ詰め込んでECSのデプロイだけに内容を絞ったのですが、その甲斐あってかいろんな方に参考にしていただける内容になったのではと考えています。

ぜひ、この記事を、皆さんの現場のECSデプロイをパワーアップする参考にしていただければと思います。

<pre style="line-height:120%;">
ホンマにECSデプロイだけで45分喋ったの…!?

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>

## はじめに「この記事について」

このブログ記事では、セッション内容をスライドを交えながら改めて文章ベースで再構成したものです。スライド単体でみていただくよりは、補足事項があったり関連リファレンスへの遷移も簡単なので、このブログ記事を見ていただくことをおすすめします。

当日の喋り含めて見たい方は、ぜひYoutubeのアーカイブも御覧ください！

## 登壇資料（動画）

（公開されたら追記します）


## 登壇資料（スライド）

<script async class="speakerdeck-embed" data-id="76f65005222b4c648b338255860ba85c" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>



## このセッションを聞いていただく意味について

みなさん、こんにちは。金曜日のお昼の貴重な時間をさいて、こちらのオンラインセッション参加いただきありがとうございます。最初に、みなさんと、本セッションを聞いていただくことの意味について共有させてください。

AWS上にコンテナワークロードをこれから展開し、開発していく時、検討が必要な項目は多岐にわたります。ざっくりあげてもこれだけあります。

- ネットワーク設計
- コンテナ設計
- 監視設計
- ロギング設計
- セキュリティ設計
- デプロイ設計

この中で、一番最初に検討いただきたいのが、<strong>デプロイ設計</strong>です。なぜか。

s08

アプリケーション開発〜保守運用へのライフサイクルに置いて、そのベースとして一番長きに渡り聞いてくるからです。ここが不安定だったり手間がかかるようだと、それをずっと引きずることになります。端的に言うと、デプロイを最初に設計することで、お金を節約できます。

S09

このお金というのは、単純にインフラに関わるランニングコストだけではなく、デプロイ待ち時間やトラブルシュートに関わるエンジニアの人件費、本番リリース時のトラブルによる機会損失、機能リリース頻度が低下することによるユーザーエクスペリエンスの低下からのMAU低下、など、ありとあらゆるところに、お金として効いてくるわけです。

ので、是非ともみなさんにはこの45分間のセッションで、ECSにおけるデプロイ機能について学んで頂き、将来に渡るTCO削減のヒントを掴んで頂ければと思います。

## 本日お話すること

本日お話することと、その順番は以下の通りです。最初に基礎を抑えていただいた後、開発環境と本番環境をモデルケースに、ECSにおけるデプロイパイプラインの代表パターンを解説します。最後に、GitHub ActionsやIaC管理リソースとの矛盾への対応方法などの発展的な内容もお話できればと思います。

s11

それでは、行きますか１

## ECSデプロイの構造を理解する

s13

AWSのコンテナ関連サービスは主に、ECSとEKSがありますが、今回はECSの話です。また、データプレーンにはEC2とFargateの2修理がありますが、今日のセッションは、どちらも適用可能です。

S16

最初に、みなさんとECSの基本構造について振返りましょう。ECSのタスク定義は以下の構造になっています。

S17

これをサービス定義に拡張した図がこちら。

S18

このECSに対してコンテナをデプロイするのですが、実際にECS上でコンテナを実行する方法は、大きく２つあります。

1. タスク実行
   - 主にFargateなどで定期バッチ処理などで利用
2. ECSサービスの利用
   - 常駐サービス（WebやAPIサーバなどで利用）

今日のセッションは、この２つ目。ECSサービスに対するデプロイの話です。このECSサービスに実際にコンテナがデプロイされるまでに、どのリソースのどのプロパティが関連しているかを示した図がこちら。

S23

デプロイだけに絞って考えると、関連しているプロパティは非常に少なく、以下の３点です。

- ECSタスク定義のコンテナのイメージURI
- ECSサービスのタスク定義バージョン
- クラスターへのタスクのデプロイ

ここまで、基本的なECSへのデプロイについて共有させて頂きました。


## ECSデプロイのアンチパターンを理解する

S25

次に、ECSへのデプロイを設計するにあたり、最初に絶対さけたほうがよいアンチパターンについて言及します。一言こちら。

S26

latest運用は、イメージの最新版にタグをlates固定でつけてしまい、それをタスク定義で常に参照する方法です。メリット・デメリットをまとめます。

### latest運用のメリット

- デプロイにおいてタスク定義、サービス定義の更新が不要
- サービスの「新バージョンの強制デプロイ」で全て完結

今回のセッションの大きな意味は、「以下に効率よくタスク定義とサービス定義を更新してデプロイパイプラインを回すか」ということに焦点をおいています。もし、イメージをlatest運用している場合は、この運用自体が不要となります。しかし、<strong>latest運用には安定的にコンテナワークロードを開発〜本番運用するときに致命的なデメリットがあります。</strong>

### latest運用のデメリット

- コンテナでエラー発生時、ソースコードとのトレーサビリティがとれない
- どのバージョンが今本番環境で動いているのか確証がとれない
- 適切なロールバック対処ができない

特に、本番環境において「現在動いているコンテナイメージとソースコードとのトレーサビリティがとれない」部分は致命傷になりえます。原因追求やエラーの再現に不確かな手順を持ち込むことで、対応が必然的に難しくなり、それはそのままコンテナワークロード安定運用の根幹を揺るがします。

ので、<strong>皆さんには、最初デプロイ設計をするとき「latest運用を排除する」を前提として考えていただきたいと思います。</strong>

ECRには、仕組みとしてイメージタグの上書きを禁止する設定がサポートされています。仕組み上の抑制策として、こちらを設定することを推奨します。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="latestタグ絶対禁止！？ECRでコンテナイメージタグの変更禁止設定がサポートされました！ | Developers.IO" src="https://hatenablog-parts.com/embed?url=https://dev.classmethod.jp/articles/ecr-immutable-image-tags/" width="300" height="150" frameborder="0" scrolling="no"></iframe>

## 開発環境にデプロイしてみる

s30

それでは、ここから実践ということで開発環境へデプロイしていきます。今回設定する開発環境の前提条件はこちら。

- AWSアカウントは一つ
- デプロイ先のECSは同じ環境
- デプロイ方法はローリングアップデート
- 承認ステージは無し

Codeシリーズを利用したデプロイパイプラインの構成図。

S33

ここから、ビルドとデプロイの詳細を説明します。

### ビルドの詳細

S35

ビルドでは主に、CodeBuildで利用する<code>Buildspec.yml</code>が中心となります。

[yaml title="Buildspec.yml"]
build:
  commands:
    - REPOSITORY_URI=${ECR_URI}
    - IMAGE_TAG=${CODEBUILD_RESOLVED_SOURCE_VERSION}
    - docker build -t $REPOSITORY_URI:$IMAGE_TAG .
post_build:
  commands:
    - docker push $REPOSITORY_URI:$IMAGE_TAG
    - echo "[{\"name\":\"${CONTAINER_NAME}\",\"imageUri\":\"${REPOSITORY_URI}:${IMAGE_TAG}\"}]" > imagedefinitions.json
artifacts:
  files: imagedefinitions.json
[/yaml]

何点か解説します。

- <code>${ECR_URI}</code>
  - プッシュ先ECRのURI。CodeBuildの環境変数で自分で設定
- <code>${CODEBUILD_RESOLVED_SOURCE_VERSION}</code>
  - CodeBuildのソースがGitリポジトリの場合に自動的に設定される環境変数
  - ソースコードリポジトリのコミットIDが格納される
- <code>imagedefinitions.json</code>
  - イメージ定義ファイルを作成
  - この中にECSのタスク定義中のコンテナとECRのURIをマッピングする情報をうめこむ

imagedefinitions.jsonの値の例はこちら。

| **キー** |   **値**   |                                   **例**                                   |
| :------: | :--------: | :------------------------------------------------------------------------: |
|   name   | コンテナ名 |                               php-container                                |
| ImageURI |  imageUri  | 123456789012.dkr.ecr.ap-northeast-1.amazonaws.com/php-container:b5649704b0 |

### デプロイ部分

S40

デプロイ部分では、前段で作成した<code>imagedefinitions.json</code>を利用しつつ、デプロイ対象の情報を入力していきます。

- Action provider：ECS
- Input artifacts：BuildArtifact
- デプロイ対象のECSクラスター、ECSサービス
- イメージ定義ファイル名

### 全体の挙動

これら、ビルドとデプロイのパイプラインを組み合わせることで、全体として以下が実現されます。

s42

### CodebuildのTIPS1：Buildkitの有効化

<code>Buildspec.yml</code>内で、環境変数の<code>DOCKER_BUILDKIT</code>を1に設定することで、BuildKitが有効化されます。マルチステージビルドやその他機能で、ビルド時間の短縮化が見込めますので是非とも有効にしましょう。

[yaml]
env:
  variables:
    DOCKER_BUILDKIT: "1"
[/yaml]

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="【ビルド高速化！】AWS CodeBuildが次世代高速コンテナビルドのBuildKitをサポートしました | Developers.IO" src="https://hatenablog-parts.com/embed?url=https://dev.classmethod.jp/articles/codebuild-buildkit/" width="300" height="150" frameborder="0" scrolling="no"></iframe>

### CodebuildのTIPS2：ローカルキャッシュ利用

CodeBuildでDockerレイヤーのローカルキャッシュを有効化することでも、ビルド時間の短縮化が見込めます。

S46

<a href="https://dev.classmethod.jp/articles/codebuild-local-cache/" target="_blank">docker buildを高速化！CodeBuildのローカルキャッシュ機能を試してみる | Developers.IO</a>

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="docker buildを高速化！CodeBuildのローカルキャッシュ機能を試してみる | Developers.IO" src="https://hatenablog-parts.com/embed?url=https://dev.classmethod.jp/articles/codebuild-local-cache/" width="300" height="150" frameborder="0" scrolling="no"></iframe>

### その他、参考リファレンスURL

- <a href="https://docs.aws.amazon.com/ja_jp/codepipeline/latest/userguide/file-reference.html#pipelines-create-image-definitions" target="_blank">イメージ定義ファイルのリファレンス - CodePipeline</a>
- <a href="https://docs.aws.amazon.com/ja_jp/codebuild/latest/userguide/build-spec-ref.html" target="_blank">CodeBuild のビルド仕様に関するリファレンス - AWS CodeBuild</a>


## 本番環境にデプロイしてみる

s50

続いて、本番環境へデプロイします。前提条件はこちら。

- AWSアカウントは開発環境のアカウントとわける
- テスト済みのコンテナイメージを利用する
- デプロイ方法としてBlue/Greenデプロイを採用する
- 承認ステージを設置する

構成図はこちら。

53

### 本番環境で使うイメージの用意手法

ここで、本番環境で使うイメージをどう用意するか、大きく分けて2つの手法があります。

#### 手段1. コードベースを同じにして再度ビルドする方法

この場合、各環境用のデプロイ用ブランチを用意しておき、それぞれのブランチにマージしてプッシュしたのをトリガーに、それぞれのブランチからイメージをビルドしてデプロイします。ただ、この場合、ビルドするタイミングにより、ソースコードが完全に同一でもイメージが同じになるとは限りません。

Dockerfileのベースイメージはたとえタグが同じでも、中身が完全に同一という保証はありません。タグの運用は各ベースイメージを提供している組織のポリシーに依存します。また、緊急性が高いセキュリティパッチなどは、タグを変えずにイメージの内容が変わることはあります。

そのため、開発環境でテストしたイメージをそのまま使いたい場合、次の方式を推奨しています。

#### 手段2. テスト済みイメージを使い回す

この場合は、開発環境でテストしたイメージを使ってステージング環境、本番環境にデプロイします。この方式にすることで、イメージの同一性を完全に保証した状態で、他のステージング環境や本番環境にデプロイし展開することができます。

### ECSのBlue/Greenデプロイの素晴らしいところ

本番環境では、利用できるのであればBlue/Greenデプロイを推奨します。

#### 理由①：テストリスナーにより本番環境でリリース対象タスクの動作確認が可能

- テストリスナーに任意のポート（例：8080）を設定
- ALBのセキュリティグループでソースIPを限定
- 本番トラフィックはオリジナル・バージョンに流しつつ、同じデータと環境で、新バージョンの動作確認が可能

#### 理由②：ルーティング切替の方法を柔軟に設定可能

CodeDeployデプロイ設定で事前定義されたルーティングの切替設定を利用可能。またカスタム定義もOK。

| デプロイ設定                                      | 説明                                                                                            |
| :------------------------------------------------ | :---------------------------------------------------------------------------------------------- |
| CodeDeployDefault.ECSLinear10PercentEvery1Minutes | すべてのトラフィックを移行するまで、1 分ごとにトラフィックの 10% を移行します。                 |
| CodeDeployDefault.ECSLinear10PercentEvery3Minutes | すべてのトラフィックを移行するまで、3 分ごとにトラフィックの 10% を移行します。                 |
| CodeDeployDefault.ECSCanary10percent5Minutes      | 最初の増分でトラフィックの 10% を移行します。残りの 90% は 5 分後にデプロイします。             |
| CodeDeployDefault.ECSCanary10percent15Minutes     | 最初の増分でトラフィックの 10% を移行します。残りの 90 パーセントは 15 分後にデプロイされます。 |
| CodeDeployDefault.ECSAllAtOnce                    | すべてのトラフィックを同時に更新済み Amazon ECS コンテナに移行します。                          |

詳細は、こちらを参照。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="CodeDeploy を使用した Blue/Green デプロイ - Amazon ECS" src="https://hatenablog-parts.com/embed?url=https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/userguide/deployment-type-bluegreen.html" width="300" height="150" frameborder="0" scrolling="no"></iframe>

#### 理由③：事故ったときの切り戻しが楽

- ロールバックがCodeDeployのコンソールから1クリック
- ローリングデプロイ時のサービス定義更新、git revertしてからの再デプロイといった手間が不要

s65

どうでしょうか。ECSへのBlue/Greenデプロイの素敵な点、わかりいただけましたでしょうか？組織ポリシーとして必須ではないですが、メリットは非常に多くあるので、ぜひこちら取り入れてみてください。

以降で、実際の本番環境へのデプロイについて紹介していきます。

### ソース部分の詳細

S67

#### appspec.yml

S68

#### taskdef.json

S69

### 承認部分の詳細

S70

CodePipelineの承認ステージを利用して、デプロイパイプラインを止めるのは数々のメリットがあります。ぜひ取り入れてみましょう。

- 開発環境、ステージング環境、本番環境でアカウントをわける大きなメリットとして、権限管理が容易さがある
- 承認の履歴やメッセージも残る
- <strong>承認プロセスはこの本番アカウントに対してアクセスするポリシーを持った人のみが実施可能</strong>なため、自然とガバナンスが効く仕組みとなる

### デプロイ部分の詳細

S72

CodePipelineのデプロイステージで設定が必要な項目は多くありますが、一つ一つ設定していきましょう。

- Action provider：ECS（Blue/Green）
- 利用するCodeDeployのApplication Nameとdeployment group
- task def SourceArtifact：taskdef.json
- AppSpec SourceArtifact：appspec.yaml
- Input artifact：ソースステージのECRイメージ名
- Placeholder text in task definition：IMAGE1_NAME

ここまで設定が完了し、無事に動くと、パイプラインとBlue/Greenデプロイがつながり、もうこれを眺めているだけでご飯10杯ぐらいイケてしまいます。最高です！！

S74

このあたり、設定する項目は多岐にわたりますが、非常に丁寧なチュートリアルがAWS公式で提供されているので、ぜひ一度こちらを参考にチャレンジしてみてください。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="チュートリアル: Amazon ECR ソースと、ECS と CodeDeploy 間のデプロイでパイプラインを作成する - CodePipeline" src="https://hatenablog-parts.com/embed?url=https://docs.aws.amazon.com/ja_jp/codepipeline/latest/userguide/tutorials-ecs-ecr-codedeploy.html" width="300" height="150" frameborder="0" scrolling="no"></iframe>

### ECS Blue/Greenデプロイの制約

そんな便利なECS Blue/Greenデプロイですが、大きな制約もあるので注意が必要です。

一番大きな制約は、Blue/Greenデプロイを使用する場合、Auto Scalingはサポートされないことです。

- 回避策として、サービスのデプロイ前後で、Auto Scalingグループのスケーリングプロセスを停止〜再開させる方法あり
- パイプラインのデプロイ前後でCodeBuildなどでapplication-autoscalingのregister-scalable-target、APIを叩く

また、キャパシティプロバイダーはサポートされないであったりとか、Fargate起動タイプ、またはCODE_DEPLOYデプロイコントローラータイプのタスクは、DAEMONスケジューリング戦略をサポートしないという制約もあります。

これら制約についても、AWS公式ドキュメントにきちんと記載されているので、必ず事前に確認しておいてください。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="Blue/Greenデプロイの考慮事項" src="https://hatenablog-parts.com/embed?url=https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/userguide/deployment-type-bluegreen.html#deployment-type-bluegreen-considerations" width="300" height="150" frameborder="0" scrolling="no"></iframe>


## GitHub Actionsを使ってデプロイする

S81

ようやく応用編ということで、GitHub Actionsについてです。最近非常に利用が広がっていますね。みなさん使ってますか？<strong>はっきりってめっちゃ魅力的ですよ。</strong>

魅力的な理由ですが、AWS公式のActionsにECS関連のものが多く、それが非常に使いやすいからです。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://github.com/aws-actions" target="_blank">AWS for GitHub Actions</a>
</p>

ECS関連で利用頻度が高いものを紹介します。

- configure-aws-credentials
  - AWSへアクセスするときの権限管理が簡単。最高
- amazon-ecr-login
  - ECRにログイン
- amazon-ecs-render-task-definition
  - タスク定義にイメージURIを挿入
- amazon-ecs-deploy-task-definition
  - タスク定義をデプロイ

### amazon-ecs-render-task-definition

[yaml highlight="5-7"]
- name: Render Amazon ECS task definition
  id: render-web-container
  uses: aws-actions/amazon-ecs-render-task-definition@v1
  with:
    task-definition: task-definition.json
    container-name: web
    image: amazon/amazon-ecs-sample:latest
[/yaml]

このように、デプロイ時に頻出するタスク定義のイメージURIの書き換えをActionsでデフォルトで保持しているため、非常に使い勝手が良いです。

### amazon-ecs-deploy-task-definition

[yaml highlight="8-10"]
- name: Deploy to Amazon ECS
  uses: aws-actions/amazon-ecs-deploy-task-definition@v1
  with:
    task-definition: task-definition.json
    service: my-service
    cluster: my-cluster
    wait-for-service-stability: true
    codedeploy-appspec: appspec.json
    codedeploy-application: my-codedeploy-application
    codedeploy-deployment-group: my-codedeploy-deployment-group
[/yaml]

こちらでは、先程説明したCodeDeployを利用したECSへのBlue/Greenデプロイにも対応しています。素晴らしい！

承認フローがなかったりするけれど仕組みでカバーできる部分もあると思うので、十分にプロダクション環境でも導入する余地はあると思います。ぜひ一度、利用してみてください。

## IaC管理するリソースとの矛盾に立ち向かう

S87

ECSのタスク定義やサービスをIaC（Infrastructure as Code）で管理している方々、こんな悩みありますよね？例えばこれは、ECSタスク定義をCloudFormationで管理している場合です。

S90

IaCでの変更ライフサイクルと、アプリケーションデプロイによるタスク定義のイメージURIのライフサイクルは異なる場合が多く、IaCのコードと整合性がとれないといったことが起こりがちです。

対処方法ですがこちら。

S91

なのですが、何点か対応策考えてきたので、その方法をみなさんと共有させてください。

### 解決案①：IaCでの全自動管理を諦める

割り切った方法です。

- 最初のプロビジョニングでのみ利用
- 更新はGUIを使うか変更点を自前で都度吸収

### 解決案②：タスク定義はプレースホルダを利用して管理し、アプリケーションデプロイサイクルにのせる

- GitHub Actionsのamazon-ecs-render-task-definitionや、CodeDeployのtaskdef.jsonを使うと、リポジトリにあるタスク定義を元にして、デプロイサイクルのなかでイメージURIを埋めることが可能
- イメージURI以外のタスク定義プロパティ（CPU、メモリ、環境変数等）の変更も、アプリケーションデプロイと同じライフサイクルで本番環境にデプロイしていく

### 解決案③：Terraformのlifecycleブロックを使ってリソース差分を無視する

- タスク定義のイメージURIや、サービスのタスクリビジョンのリソース差分を無視する設定にする
- デプロイによりそれらプロパティが変わっても変更検知されないし、TerraformでApplyできる！

この方法、CloudFormationでは使えないですが、TerraformでECSのサービスやタスク定義を管理している場合には、こういうやり方もあることをお伝えしておきます。一見力技ですが理にかなっていると思います。

## まとめ

ここまで長々と喋ってきましたが、最後にECSデプロイについての考え方をまとめておきます。

ECSのデプロイですが、各組織がもつポリシーやアプリケーション特性によりECSデプロイの理想形は様々です。それこそ、組織の数だけデプロイフローは存在するでしょう。ただ、そういった中でも運用していくときにさけたほうがよいアンチパターンもあります。

アンチパターンをさけつつ、効率良いECSデプロイのパイプラインを構築するために、今日のセッションを参考にしていただければと思います。ありがとうございました。


## セッション中にでたQAコーナー

セッションで上げられていたQAを紹介。

### Q「latest運用しない」ということでしたが、ベースイメージを作成する場合はlatestのほうが良いですか？ベースイメージを使うイメージの設定変更が大変になるような気がするのですが」

プロジェクトで共通で使うベースイメージを、独自に作成している現場ということで理解しました。ベースイメージの更新を完全にプロジェクト側でコントロールできるのであれば、それを利用する側では、Dockerfileでlatest指定するのも一つの方法だと思います。

ただ、その場合でも、ビルドタイミングによって、そのイメージに含まれるベースイメージが不定になるという問題はのこり、トレーサビリティが取れなくなる可能性が出てくるので、この場合でもlatest運用は避けたほうが良いと考えます。


### Q「事故ったときボタンで戻した直後に、何らかの原因でタスクが停止・起動した場合、タスク定義のImageURI:TAGが事故ったバージョンで起動してくることはないのでしょうか？」

ECSのBlue/Greenデプロイでのロールバック（事故ったときの戻し）時に起動するタスクは、もともと安定稼働していたタスク定義ですので、それが起動しないことは基本的にないはずです。元のバージョンでも起動しない場合は、なにか他の原因が考えられます。

### Q「最初にデプロイを設計するのは、どのあたりまで考えますか？実際に使いはじめる前に、最初から完成形を目指すと大変なのかなと思いました。」

そうですね。最低限、開発環境へのデプロイまでパイプラインが完成していれば大丈夫です。なんらかの開発環境と対応するブランチを定めておき、そのブランチのプッシュをトリガーに、開発環境にデプロイされるパイプラインを組んでおくだけで、開発スピードは非常に高速化すると思います。ステージング環境や本番環境へのデプロイは、アプリケーションの開発を進めながら並行して進めていっても大丈夫です。


### Q「GitHub Actionsで擬似的でも良いので承認ステージを追加する手法はありますでしょうか？」

現状、GitHub Actionsにはそれっぽい機能が無いので、GitHub Actionsの挙動をラップするようなパイプラインを別で構築し、そちらで承認ステージ的なものを導入するしか方法がなさそうです。

### Q「GitHub ActionsとCodeDeployを組み合わせてBlue/Greenデプロイする例のサンプルやブログ記事などはありますか？」

ちょっと探した限りなかったです。公式ページ（<a href="https://github.com/aws-actions/amazon-ecs-deploy-task-definition#aws-codedeploy-support" target="_blank">AWS CodeDeploy Support</a>）を参考にしてみてください。

また、ハマコーが近々、ブログを書きます！

### Q「タスク定義はIaCで管理するのが良いんでしょうか、GithubやCodecommitにのせて、パイプラインで更新するみたいなことは出来るんでしょうか？」

オススメとしては、TerraformのLifeCycleでイメージURIの変更を無視する方法です。さらに組み合わせて、本日のセッションで説明した、CodeDeployに含ませるtaskdef.jsonの雛形機能や、GitHub Actionsの機能を使えば、IaCで管理するECSタスク定義をデプロイとTerraformの両方で併用可能です。



## 登壇後日談

というわけで、45分間オンラインで喋ったのですが、集中力の入れ方が半端なかったです。ぶっちゃけリアルで喋るときの2倍ぐらい疲れました。一瞬たりとも気を抜けないというか…

直前まで資料に手を入れていたんですが、QAもたくさん頂き、ありがとうございました。ECSへのデプロイは、CodeDeployがECSへのBlue/Greenをサポートしたことにより一気に広がりを見せています。利用上の注意点や、実際に組み込むときの注意点もありますが、うまくみなさんのパイプラインに組みこめば、本番デプロイを異次元の体験にまで昇華できます。ぜひ、課題感を持っている方は、この記事を参考にしてみてください。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。
