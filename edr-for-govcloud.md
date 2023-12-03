# (仮)Elastic Disaster RecoveryがGovCloud(US)で利用可能になりました！

AWS Elastic Disaster Recovery(AWS DRS)、みなさんご存知でしょうか。今まですべての商用リージョンで提供されていたこのサービスが、GovCloud（米国）リージョンで利用可能になりました。

まぁ正直、このブログを読んでいる人で普段GovCloudを利用している人は皆無！？だと思いますが、DRのマネージドサービスとしてのAWS DRSを全く知らない人も多かったと思うので、そのあたりの復習も兼ねて、皆さん見ていただければ。

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     ﾃﾞｨｻﾞｽﾀﾘｶﾊﾞﾘﾏﾂﾘﾀﾞﾜｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>

祭りってのはちょっと…

## アップデートの公式情報

AWSのWhat's Newはこちら。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="Announcing AWS Elastic Disaster Recovery in the AWS GovCloud (US) Regions" src="https://hatenablog-parts.com/embed?url=https://aws.amazon.com/jp/about-aws/whats-new/2023/11/aws-elastic-disaster-recovery-govcloud-regions/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>


## そもそも、GovCooud（US）とは？

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="AWS GovCloud (米国) - アマゾン ウェブ サービス" src="https://hatenablog-parts.com/embed?url=https://aws.amazon.com/jp/govcloud-us/?whats-new-ess.sort-by=item.additionalFields.postDateTime&whats-new-ess.sort-order=desc" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

<blockquote>AWS GovCloud (米国) は、政府機関のお客様とそのパートナーに、DOJ の刑事司法情報システム (CJIS) セキュリティポリシー、米国国際武器取引規則 (ITAR)、輸出管理規則 (EAR)、国防総省 (DoD) クラウドコンピューティングセキュリティ要件ガイド (SRG) の影響レベル2、4、5、FIPS 140-2、IRS-1075、およびその他のコンプライアンス体制など、FedRAMP High ベースラインに準拠して安全なクラウドソリューションを設計できる柔軟性を提供します。</blockquote>

なんだか物々しいですが、米国の他の商用リージョン（米国東部や西部）などに比べて、運用ルールが根本的に異なります。AWS利用者なら誰でも利用できるわけではなく、事前のスクリーニングプロセスを経た法人と利用者のみに利用を制限されています。

端的に、超重要機密情報を扱う政府機関などの利用を想定していると言えるでしょう。

## AWS Elastic Disaster Recovery(AWS DRS)とは？

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="災害対策 | AWS" src="https://hatenablog-parts.com/embed?url=https://aws.amazon.com/jp/disaster-recovery/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

<blockquote>AWS Elastic Disaster Recovery (AWS DRS) は、手頃な料金のストレージ、最小限のコンピューティング、ポイントインタイムリカバリを使用して、オンプレミスおよびクラウドベースのアプリケーションを迅速かつ確実に復旧することで、ダウンタイムやデータ損失を最小限に抑えます。</blockquote>

いわゆる一つのDRに特化したAWSのマネージドサービスです。ソースサーバーにAWS Elastic Disaster Recoveryを設定しレプリケーション設定することで、ソースサーバーがダウンした時に可能な限り最短の復旧ポイントと復旧時間を実現します。

このレプリケーションは常時稼働するためランニングコストがかかりがちな部分ですが、この部分のプロビジョニングを手頃な料金と最小限のコンピューティングリソースを利用して実現するのが、AWS DRSです。

010-dr
引用：https://aws.amazon.com/jp/disaster-recovery/

より詳細は、公式ドキュメントから参照できます。

[What is Elastic Disaster Recovery? \- AWS Elastic Disaster Recovery](https://docs.aws.amazon.com/drs/latest/userguide/what-is-drs.html)

## 公共性の高いGovCloud（米国）でのAWS DRS提供でDRの実装の幅が広がる

今回のアップデートで、公共性が非常に高いGovCloud（米国）でのDRS利用が可能になりました。そもそも利用用途として限界までの可用性が求められるGovCloudにおいて、DRS提供は非常に相性が良いので、これからも利用が広がっていくのではないでしょうか。

また、AWS DRSはまだまだ知名度が低いサービスだと個人的に思っているので、これをきっかけに国内商用リージョンでDRを実装する際の手段として、AWS DRSを改めて検討してみるのも良さそうですね。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。
