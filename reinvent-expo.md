# re:Invent2022のExpo出典パートナーを開発目線で8つ紹介

「来るたびに思うけど、なんやねん、この規模は…」

re:Invent2022のEXPO会場についた時の感想です。

いやほんと、re:Inventの規模感を身を持って体感できるイベントが、このEXPOです。いわゆるひとつの展示会場。re:Inventは各種セッションやアクティビティなどの数も膨大で桁外れなのですが、このEXPOもそれに負けず劣らず規模がでかい。

そんなブースをうろついているだけでも楽しいのですが、今回は、主にアプリケーション開発の文脈で使えそうなサービスを中心に8種類ほどまとめてみました。どれも、珠玉の面白そうなサービスなので、気になった方は、是非トライしてみましょう。

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     ﾏﾂﾘﾀﾞﾜｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>


## re:Invent 2022のExpoとは？

公式サイトの説明はこちら。

[AWS re:Invent 2022 Expo \| Amazon Web Services](https://reinvent.awsevents.com/learn/expo/)

公式サイトでは、いろんなフィーチャーが紹介されていますが、一言で言うと<strong>「祭り」</strong>です。AWSの公式コーナーもかなりの規模ありますが、それ以上にre:Invent2022に出典しているスポンサーのブースが所狭しと、そして、異常に広大なスペースに出典されています。

expo-100.jpg

会場全体で、300メートル四方あるのでは？と思うぐらいの広大さで、ブースを歩きながらいろんなお土産をもらうだけでも、丸一日は潰せるだろうぐらい盛況でした。

この中でいろんな企業のブースを回って、担当の方と話してみて、ハマコーなりに琴線にふれた面白そうなものを中心にまとめてみました。資料としての網羅性などは一切考慮していない独自のセレクションですよ。

## 紹介するサービス一覧

| 公式サイト | キーフレーズ | 特徴 | フリートライアル有無 |
| --- | --- | --- | --- |
| [Solace](https://solace.com/) | Powering real-time event-driven enterprises | オンプレ含めたマルチクラウドにおけるイベント管理ツール | 有り、60日間 |
| [Quali](https://www.quali.com/) | Simplify & Accelerate Youre Infrastructure Processes | インフラ環境プロビジョニングの横断的なフルマネージドサービス | Torque：30日間<br>CloudShell：申請必要 |
| [Unqork](https://www.unqork.com/) | Introducing Codeless as a Service (CaaS) | モダンノーコードプラットフォーム | 無。代わりにガイドツアーへの申込みフォーム有 |
| [Console Connect](https://www.consoleconnect.com/) | THE SIMPLER, SMARTER WAY TO CONNECT | コンソールから即利用できる、グローバル最高水準のNetwork as a Service | 無。無料のコンソールアカウントを作成後、各リソース作成でコストが発生 |
| [Hasura](https://hasura.io/) | Instant GraphQL on all your data | DBスキーマから一分でGraphQLサーバ構築 | セルフホステッド：無料<br>SaaS版：Tireにより異なる |
| [Appian](https://appian.com/) | Low-Code Platform | ローコードプラットフォーム（業務プロセス、ワークフローの自動化） | 申請による無料アカウントの申込み |
| [Lens](https://k8slens.dev/) | The Kubernetes IDE | Kubernetes環境の統合運用管理を目的としたデスクトップ製品を中心としたサービス群 | PERSONAL：無料<br>PRO：30日間試用期間有り |
| [LaunchDarkly](https://launchdarkly.com/) | Fundamentally change how you deliver software | 統合プラットフォームによるデプロイ特化サービス | Starter：14日間試用期間有り<br>Pro：14日間試用期間有り |


## Solace「オンプレ含めたマルチクラウドにおけるイベント管理ツール」

expo-101.jpg

[Advanced Event Broker\. An event mesh for connected enterprises \| Solace](https://solace.com/)

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
Powering real-time event-driven enterprises
</p>

リアルタイムイベントドリブンを軸としたサービスを展開。イベント・ストリーミングのフルマネージドなプラットフォームで、企業がイベント駆動アプリケーションの活用を支援。

- イベントメッシュを構築し、あらゆる環境でイベントをストリーミング
- イベントの設計、発見、共有
- 企業全体のイベントフローを可視化し、管理

expo-101-1.jpg

上記の画像イメージで説明されているように、マルチクラウド、さらにオンプレミスも含めたイベント連携が可能となるサービスのようです。各クラウド環境の中では、近年AWSのEvent Bridgeのような、イベント関連系を専門とするマネージドサービスが非常に充実してきていますが、それを、マルチクラウドにまで広げることで、企業全体の分散アプリケーションの構築と一貫した管理ができるとのことです。

### 試すことはできるか？

以下のページから、60日間のフリートライアルの申込みが可能。

[Try It Now \- Solace](https://solace.com/try-it-now/)

## Quali「インフラ環境プロビジョニングの横断的なフルマネージドサービス」

expo-103.png

[Simplify & Accelerate Your Infrastructure](https://www.quali.com/)


<blockquote>Simplify & Accelerate Your Infrastructure Processes<br /><br />
Build, deliver, and manage your complex infrastructure and environments faster and more efficiently.</blockquote>

複雑なインフラ環境の構築をより加速させ、より効率的に管理する製品。インフラを再現可能なブループリントに変換し、継続的に環境を管理し利用方法を標準化します。

- 自動化
  - インフラをすぐに稼働できる環境に自動的にプロビジョニング
- デリバリー
  - CI/CDのツール内で、セルフサービスと自動化により、オンデマンドで環境をデプロイ
- メンテナンス
  - コードと環境の差分を検出（ドリフト検出）し、チームで展開すべき全てのインフラに適用されるアップデートをプッシュ
- 最適化
  - クラウドの設定を管理するためのポリシーを設定し、不要なコストを削減し、セキュリティリスクを軽減

ユースケースとして、適用領域が6種類紹介されています。Terraform以外にも、HelmやKubernetesに対応しているのは、適用範囲が広くて良いですね。

- Terraform
- Helm
- Kubernetes
- Self Service Deployment
- CI/CD Deployment
- Cloud Cost Management

### Teffaromのユースケース

[Terraform Automation \| Quali](https://www.quali.com/terraform-automation/)

expo-103-1

Terraformでプロビジョニングされたインフラを一元的に管理し、開発者がセルフサービスとCI/CD自動化によって、オンデマンドでデプロイできるとのこと。Terraformアセットを一元管理し、そこからのデプロイとドリフトの検出、環境の構築まで全て、Torqueというツールで対応が可能なようです。

### 試すことはできるか？

以下のページから、トライアルの申込みが可能。製品により扱いが異なります。

[Free Trial for Quali Torque and CloudShell](https://www.quali.com/start-trial/)

- Torque
  - AWSおよびAzureインフラのデリバリの簡素化、高速化が必要なDevOpsチーム向け
  - 30日間の無料トライアル有り
- CloudShell
  - オンプレとクラウドのハイブリッド環境向けのプロビジョニングツール
  - トライアルは申込みが必要

## Unqork「モダンノーコードプラットフォーム」

expo-104-title.png

[Codeless as a Service \| Enterprise Apps \| Unqork](https://www.unqork.com/)

<blockquote>Introducing Codeless as a Service (CaaS)<br /><br />
“CaaS isn't just another tool in a tool shed. It's a category that hasn't existed before.”</blockquote>

いわゆるひとつのコードレスサービス。既存のテクノロジーは限界に達していて、Unqorkを使うことでモダナイゼーションをイノベーションを加速し、よりビジネスに密着した形でスマートに実装することを目的とした製品です。

- マーケット投入までの時間を短縮
  - 従来の方法に比べて3倍早く投入
- レガシーコードの排除
  - アプリケーションロジックの動作をコードから除外し、技術的負債の増大を排除
- 低コスト
  - コード/ローコードベースのアプローチと比較し、TCOを3分の1まで削減

expo-104-1.png

製品概要の動画をみてましたが、非常に実現できることが多く、UIも直感的で使いやすそうでした。ノーコードツールは、DBアクセスが必要な業務ロジックなどを組み込もうとすると途端に扱いが難しくなる印象ですが、そのあたりのジレンマをどうやって解決するのか、興味深いです。サイトを見ていると事例も多く、今勢いのあるのーコードツールの一つと言えるんじゃないでしょうか。

### 試すことはできるか？

サイトからフリートライアルについての記述を見つけることができませんでした。導線として基本はデモをリクエストして商談を行うことが基本のようです。

また別の導線として、ガイドツアーへの申込みフォームが用意されており、ここからより詳細に製品内容を把握することが可能なようです。

[Register: Guided Tour of Unqork's Codeless Platform](https://www.unqork.com/guided-tour/)


## Console Connect「コンソールから即利用できる、グローバル最高水準のNetwork as a Service」

expo-108-1.png

[Software\-Defined Interconnection Provider \- Console Connect](https://www.consoleconnect.com/)

<blockquote>THE SIMPLER, SMARTER WAY TO CONNECT<br /><br/>
Manage and grow all your mission-critical network connections in real-time through one easy-to-use platform.</blockquote>

世界最高水準のグローバルネットワークを利用した、クラウド間高速接続サービス。

- 簡単サインアップ
  - データセンター、アプリケーション、各パートナーとの接続を即時サインアップ
- グローバル仕様
  - グローバルで最高クラスのプライベートネットワークインフラ
- 信頼性とパフォーマンス
  - インターネットではなく、Network-as-a-Serviceを通じて、ミッションクリティカルなワークロードのセキュリティ、スピード、パフォーマンスを向上させることが可能
- オンデマンド
  - ビジネスの状況に応じて、柔軟に対応
  - 複雑でコストの掛かるネットワーク接続とはおさらば

最初理解が難しかったのですが、NaaS（Network as a Service）と呼ばれるサービスのようで、グローバル最高クラスのプライベートネットワークを、サービスとして提供する形態のようです。コンソールアカウントの作成は即時にできるので、試してみたのですが、DataCenterの一覧がこのように表示されており、これらデータセンター間の接続をオンデマンドでコンソール上から実施できるとのこと。

expo-108-2.png

帯域の大きさもコンソールから設定。設定内容で、どの程度料金がかかるか、即時計算してくれます。

expo-108-3.png
expo-108-4.png
expo-108-5.png

もちろんマルチクラウドにも対応していて、CloudRouterを利用して、およそ主要なパブリッククラウド間のプライベートネットワーク構築を、統一されたコンソールから実現可能とのこと。なんかすごいな…

expo-108-9.png

### 試すことはできるか？

コンソールのアカウント作成は、非常に簡単な手続きで即可能です。後は、実際に利用するネットワークの帯域や拠点により従量課金される仕組み。

## Hasura「DBスキーマから一分でGraphQLサーバ構築」

expo-110-1.jpg

[Instant GraphQL APIs on your data \| Built\-in Authz & Caching](https://hasura.io/)

<blockquote>Instant GraphQL on all your data<br />
- TickBuild modern apps & APIs 10x faster<br />
- TickBuilt in Authorization & Caching<br />
- TickBlazing fast GraphQL & REST APIs<br />
- TickOpen source</blockquote>

データベースから、GraphQL用のAPIを1分間で作成するプロダクト。ローカルでもクラウドでも動作し、既存のデータベースに接続設定することで、プロダクションで利用できるGraphQLサーバを実装可能とのこと。メインのDBエンジンは、OSSとして公開されています。

[hasura/graphql\-engine](https://github.com/hasura/graphql-engine)

### Hasuraが対応しているデータベース

サポートしているデータベースの数はなかなか多く、以下のデータベースが対応しています（2022年12月時点）。

expo-110-2.jpg

上の一覧にはAmazon Auroraもありますが、MySQLには現在対応していないので注意です。

- サポート済み
  - PostgreSQL
  - SQL Server
  - Big Query
- カミングスーン
  - Oracle
  - MongoDB
  - MySQL
  - Elastic

詳細な一覧は、対応予定のものも含めて以下で確認できます。

[Instant GraphQL APIs & REST APIs on Databases](https://hasura.io/graphql/database/)

### Hasuraの使い方

expo-110-3.jpg

主に3種類の利用方法があり、それぞれで値段が異なります。オープンソースベースなだけありセルフホステッドだと無料で使えるのも良いですね。SaaS版も個人利用は無料なので、サクッと試すことができます。

- Hasura Community Edition（CE）
  - オープンソースで提供されているものをセルフホスティング
  - [Hasura GraphQL Engine Documentation \| Hasura GraphQL Docs](https://hasura.io/docs/latest/index/)
- Hasura Community Edition（EE）
  - CE版の機能肉分けて、SSOやサポートが着いてくる
  - 価格は要問合せ
  - [Get in touch with us \| Hasura](https://hasura.io/contact-us/)
- Hasura Cloud
  - SaaS利用版
  - Free版有り（個人利用のみ）
  - Standardは、$99/mo/project。無料お試し期間有り
  - Enterpriseは、さらにサポート月。価格は要問い合わせ


### その他参考情報

DevelopersIOには、弊社[森](https://dev.classmethod.jp/author/mori-ryosuke/)が執筆しているHasuraの記事一覧もあります。

[Hasura の記事一覧 \| DevelopersIO](https://dev.classmethod.jp/tags/hasura/)

## appian「ローコードプラットフォーム（業務プロセス、ワークフローの自動化）」

expo-111-1.png

[Appian: Low\-Code Platform for Workflow, Automation & Process Mining](https://appian.com/)

<blockquote>Build enterprise apps and workflows rapidly.<br /><br />
Our unified low-code platform and solutions maximize your resources and dramatically improve your business results. Solve your business process challenges.</blockquote>

ローコードプラットフォームサービス。ワークフローの構築を統一プラットフォームでデータを統合し、そこにアプリケーションを統合することで、ビジネス全体を自動化し、効率化するサービスです。

ざーっと公式サイトを見た限り、非常に広範な領域をカバーしているようで、Visualによるモデルの作成、専用IDEの提供、オートコンプリート、共通業務ルールの作成などに対応してます。

expo-111-2.png

expo-111-3.png


### 試すことはできるか？

こちらから、メールアドレスを入力することで、無料アカウントの作成ができます。

[Start Free with Appian Community Edition \- Trial \| Appian](https://appian.com/landing/community-edition/get-started.html)

## LENS「Kubernetes環境の統合運用管理を目的としたデスクトップ製品を中心としたサービス群」

expo-114-1.png

[Lens \| The Kubernetes IDE](https://k8slens.dev/)

<blockquote>The way the world runs Kubernetes<br /><br />
Kubernetes is the OS for the cloud. Thousands of businesses and people develop and operate their Kubernetes on Lens — The largest and most advanced Kubernetes platform in the world.</blockquote>

主要製品は、Lens Desktopで、Kubernetesのコンソールを提供するサービスです。基本的にはクライアントのデスクトップで動作させる前提で、主要なプラットフォームに対応しています。オープンソース上に構築されていて、Kubernetesとクラウドネイティブエコシステムと深く連携しています。

- Mac OS Intel Chip (.dmg)
- Mac OS Apple Silicon (.dmg)
- Windows x64 (.exe)
- Linux x64 (.rpm)
- Linux x64 (.deb)
- Linux x64 (AppImage)
- Linux x64 (.snap)

あらゆるKubernetesで動作し、複雑さを解消し生産性を向上させる。開発者から運用担当者、スタートアップから大企業まですべての人が利用可能。Youtubeでデモ動画も公開されています。

<iframe width="560" height="315" src="https://www.youtube.com/embed/eeDwdVXattc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

また、Lens Kubernetesというものも提供されており、パブリッククラウドでもオンプレミスでも構築できるKubernetesプラットフォームとして利用可能とのことです。

[Lens Kubernetes](https://k8slens.dev/kubernetes.html)


### 試すことはできるか？

Pricingページに記載があります。所属組織の売上（もしくは資金調達額）の制限はありますが、PERSONALプランであれば無料で利用できます。

[Pricing](https://app.k8slens.dev/subscribe)

- PERSONAL
  - 無料
  - 制限として、会社の所得が年間1000万ドル未満であること
- PRO
  - $19.90/Month
  - 30日間の無料試用期間有り
  - PERSONALプランの機能に加えて、チームコラボレーションや、24*5のサポート有り

## LaunchDarkly「統合プラットフォームによるデプロイ特化サービス」

expo-119-1.png

[LaunchDarkly: Feature Flag & Toggle Management](https://launchdarkly.com/)

<blockquote>Fundamentally change how you deliver software.<br /><br />
Innovate faster, deploy fearlessly, and make each release a masterpiece.
</blockquote>

デプロイ〜リリースに特化した統合プラットフォームサービスです。本番リリースはどんな時代も緊張するセンシティブな作業ですが、LaunchDarklyを利用することで、リスクを最小化し、安全なシステム移行や本番環境での事前の検証などを、統合プラットフォーム上で管理運用できるとのこと。

[PRODUCT OVERVIEW \| LaunchDarkly](https://launchdarkly.com/product/)

機能フラグの設定によるデプロイ管理

expo-119-2.png

あらゆるプラットフォームへの接続対応（クラウドだけではなく、AndroidやiOSにも対応）

expo-119-3.png

チームへのデータを元にした知見の提供

expo-119-4.png

### 試すことはできるか？

Pricingページはこちら。

[LaunchDarkly: Pricing \| LaunchDarkly](https://launchdarkly.com/pricing/)

プランは3つに分かれていて、StarterとProにはそれぞれ14日間のトライアル期間が設定されています。とりあえず触ってみるのであれば、Proの14日間トライアルから初めて見るのが良さそう。

- Starter
  - $8.33/Month Build Annually
- Pro
  - $16.67/Month Build Annually
- Enterprise
  - 要問い合わせ


## re:InventのEXPOは宝の山

以上、自分がブースをうろつきながら、おもに開発目線で気になったサービスをざーっとまとめてみました。今回紹介したものはどれも面白そうなのがてんこ盛り、かつ無料試用期間が設定されているものも結構あるので、自分でもいくつかこの後、試してみようと思います。

個人的には、気軽に使えそうなHASURAとLENS Desktopあたりから触ってみる予定です。皆さんも、この中から興味深いものがあったら、ぜひ触ってみてください。今のアプリケーション運用を解決するサービスに出会えるかもしれませんよ。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。


