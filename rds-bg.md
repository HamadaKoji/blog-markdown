# 【衝撃】AWSのRDSが、データを失わないBlue/Greenデプロイに対応しました

<strong>「最近は、データベースもB/Gデプロイできるらしいよ？」</strong>

<strong>「そりゃそうやろ。B/Gデプロイなんて、最近当たり前………　へ？DBが？無理でしょ？ほぇ？どういうこと？」
</strong>

最初アップデートのタイトルを見たときの、ハマコーの率直な完走です。

Blue/Greenデプロイは、現行バージョンのトラフィックを活かしたまま新バージョンを動作確認し、問題なければ新バージョンをリリースするという、最近の安全なデプロイの概念において無くてはならないものです。

同時に新旧バージョンを稼働させるため、基本的にはステートレスなアプリケーション・サーバーにおいて利用するものという固定概念があったのですが、それが<strong>ステートフルの塊であるデータベースにおいて適用されるという謎技術がAWSからリリースされた</strong>ので、その内容を確認します。

この記事では、速報的に概要を把握するために、まずは公式情報をかいつまんでまとめることを目的にしています。実際の動作確認は、別途クラメソの誰かが書くことでしょう！

<pre style="line-height:120%;">
謎技術リリースきたか…!!
　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>

## アップデートのサマリー情報

What's Newで、紹介されています。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="Announcing Amazon RDS Blue/Green Deployments for safer, simpler, and faster updates" src="https://hatenablog-parts.com/embed?url=https://aws.amazon.com/jp/about-aws/whats-new/2022/11/amazon-rds-blue-green-deployments-safer-simpler-faster-updates/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

上記What's Newの抄訳。

<blockquote>Amazon RDSは、今回のアップデートでBlue/Greenデプロイをサポートし、安全かつシンプルなデータベースの更新に対応します。Blue/Greenデプロイを利用することで、管理されたステージング環境を作成し、現在の本番データベースを維持したまま、本番環境の変更をデプロイしてテストすることが可能です。ワンクリックで、ステージング環境を新しい本番システムに昇格させることができ、アプリケーションの変更も不要で、データの損失も有りません。

Amazon RDS Blue/Green Deployments を使用して、Amazon RDS Console から数クリックでデータベースを更新できます。</blockquote>

データの損失も無いということなので、新旧で更新データの同期がとれるということでしょうか。なかなかにアツいアプデのようです。

## 対象のリージョンとエンジンバージョン

対象のバージョンは、今はMySQLベースのものになっており、PostgreSQLは対象外のようです。すでに中国リージョン以外の全リージョンで一般提供開始とのことなので、即時利用できます。

- 対象バージョン
  - Amazon Aurora with MySQL compatibility 5.6 以上
  - Amazon RDS for MySQL 5.7 以上
  - Amazon RDS for MariaDB 10.2 以上
- 対象リージョン
  - すべてのAWSリージョン（AWS中国リージョンを除く）とAWS GovCloud（米国リージョン）

## RDSのBlue/Greenデプロイの動作概要について

AWSの公式ブログに、もうすこし詳細な内容が記載されています。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="New – Fully Managed Blue/Green Deployments in Amazon Aurora and Amazon RDS | AWS News Blog" src="https://hatenablog-parts.com/embed?url=https://aws.amazon.com/jp/blogs/aws/new-fully-managed-blue-green-deployments-in-amazon-aurora-and-amazon-rds/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

##　凄さを要約してみた

およそ信じられない技術ですが、データベース更新における長年人類が抱えていたジレンマを根本から解決する、素晴らしい仕組みになってます。

- 数ステップで、本番環境をミラーリングするフルマネージドなステージング環境を作成
- ステージング環境は、本番環境のプライマリデータベースとリージョン内リードレプリカのクローンを作成
- Blue/Greenデプロイメントでは、論理レプリケーションを使用して2つの環境の同期を維持
- ステージングの昇格にかかる時間はわずか1分
- 切替の間、Blue環境もGreen環境も両方、書き込みをブロックしデータ損失を防ぐ
- 最後に、本番環境のトラフィックを昇格したステージング環境にリダイレクト
- あらゆるシチュエーションで利用可能
  - エンジンのアップグレード
  - <strong>（重要）</strong>スキーマの変更
  - OSやメンテナンスのアップデート対応

スキーマの変更にも対応して、データ同期してくれるって、ホンマかいな。

## 公式ドキュメント

RDSの公式ドキュメントに1セクション増えています。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="Using Amazon RDS Blue/Green Deployments for database updates - Amazon Relational Database Service" src="https://hatenablog-parts.com/embed?url=https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/blue-green-deployments.html" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

実際の動作イメージは、公式ドキュメント（以下は公式ドキュメントから引用）に画像つきで紹介されているので、まずはこちらをみると理解が捗るかと思います。

画像


## 利用上の考慮事項

公式ドキュメントには、利用上の考慮事項も記載されています。代表的なものをまとめてみました。

[Considerations for blue/green deployments](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/blue-green-deployments-overview.html#blue-green-deployments-considerations)

- Blue/Greenデプロイは、リソースに固有のIDにより、リソースを識別する
- Blue/Greenデプロイにより、Greenリソースが本番リソースに昇格した場合、リソースIDは変更される

そのため、リソースIDに依存した運用がある場合、そのメンテナンスが必要になります。

- RDS APIをリソースIDを利用してフィルタリングしている場合
- リソースの監視に、リソースIDに依存しているCloudTrailを利用している場合
- IAMポリシーでリソースIDを利用している場合
- AWS BackuupでリソースIDをしている場合

リソースIDが変わることによる影響は、だいぶ大きそうですね。IAMポリシーなども、ID指定しているシチュエーションは結構あると思うので、リソースIDが変わって、RDS操作できなくなった！なんてことが無いように、事前の検証はしっかりやっておく必要がありそうです。

## B/Gデプロイのベストプラクティス

実際にどのようなシチュエーションで使うのが良いのか、ベストプラクティスが紹介されています。

[Best practices for blue/green deployments](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/blue-green-deployments-overview.html#blue-green-deployments-best-practices)

自分が気になったのは、スキーマ変更に関する記述。<strong>全てのスキーマ変更が、データのレプリケーションに対応しているわけでは有りません。</strong>

- レプリケーションに対応しているスキーマ変更
  - テーブル末尾への列追加
  - インデックスの作成と削除
- レプリケーションに対応していないスキーマ変更
  - 列名の変更
  - テーブル名の変更

このあたり、レプリケーションの動作を考えると当たり前の話ですね。スキーマ変更がなんでもB/Gデプロイのデータレプリケーションに対応しているわけではないこと、注意しておきましょう。

## B/Gデプロイの主な制約

このあたりのサービス使っていると、利用できないようです。個人的にはCloudFormationが対応していないのは、結構きついかもですね。リソース定義を静的にコードで記載するのと、ある程度AWS側のマネージドな仕組みでリソースID含めた変更がはいるのは、運用が破綻しそうなのでしかたないかなとも思います。

[Limitations for blue/green deployments](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/blue-green-deployments-overview.html#blue-green-deployments-limitations)

- Amazon RDS Proxy
- Cascading read replicas
- Cross-Region read replicas
- Multi-AZ DB clusters
- AWS CloudFormation

## 制約はある程度あるが、マネージドな仕組みでDB変更に安全性をもたらす夢の技術

以上、ドキュメントベースに、主な動作内容は制限事項をまとめてみました。まだ詳細は把握できておらず、実際のプロダクション導入前には入念な検証が必須なのは間違いないですが、うまく使うと、本番環境のDB変更のリスクを飛躍的に抑える可能性があるアップデートだと思います。

RDSやAuroraを使っている人は、まずは一回手元で動かしてみて、動作検証してみても良いのではないでしょうか。皆さんの日々の運用がさらに進化する可能性を秘めていると思います。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。
