絶賛re:Invent2023の開催中ですが、re:Invent2021で発表されたコミュニティサービスであるre:Postのプライベート版が発表されたということで、その概要をお届けします。

re:Post自体は、完全にパブリックで世界でAWSの知識を共有しよう！という思想で始まったもサービスですが、大規模にAWSを利用している組織向けに、組織の中での情報共有を促すPrivate版が発表されたというものになります。

<pre style="line-height:120%;">
AWS企業利用の公式福利厚生サービスってこと…!!??

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>

まぁ、そんな雰囲気もなくはないかな。


## re:Postの概要

公式のアップデートブログはこちら。

[Announcing the general availability of AWS re:Post Private](https://aws.amazon.com/jp/about-aws/whats-new/2023/11/general-availability-aws-re-post-private/)

re:Post自体は2021年12月2日という、2021年のre:Invent期間中にリリースされたサービスで、AWSが管理するQ&Aサービスです。以前あったAWSフォーラムに変わり、専門家監修の回答を提供しユーザーはその回答を確認し、また、専門家は評判ポイントを獲得してコミュニティエキスパートとして昇格していきます。

- 参考
  - [AWS re:Post – AWS コミュニティのために再考された Q&A サービス \| Amazon Web Services ブログ](https://aws.amazon.com/jp/blogs/news/aws-repost-a-reimagined-qa-experience-for-the-aws-community/)
  - [コミュニティに貢献してre:Postエキスパートになろう！今日から始めるre:Postキホンの「キ」 \| DevelopersIO](https://dev.classmethod.jp/articles/how-to-start-re-post/)

今回のアップデートで、このrre:Postの仕組みを企業単位でプライベートで提供することができるようになっています。パブリックな場所ではできなかった、よりセキュアでかつ企業のドメインに特化したコミュニケーションとナレッジを共有し、それを再利用可能な形で提供します。

また、AWSが提供するトレーニングや技術コンテンツ、一元化されたナレッジリソースでスキルを向上させ、新しいチームメンバーのオンボーディングにも利用可能です。

また、AWSサポートとも統合されていて、re:Post内でのディスカッションをサポートケースに変換することもできます。

サービスページはこちら。

[AWS re:Post Private](https://aws.amazon.com/jp/repost-private/)

010.png

公式ドキュメントももちろん提供されています。

[AWS re:Post Private](https://docs.aws.amazon.com/repostprivate/)

### （重要）利用条件は？

Enterprise SupportおよびEnterprise On-Ramp Supportプランの利用が必須です。全てのAWSアカウントやOrganizationで利用できるものではないことは、最初に頭にいれておいたほうが良いでしょう。


### 利用できるリージョンは？

- US West(Oregon)
- Europe(Frankfurt)

のリージョンのみです。少なく感じますが、そもそもこのサービスがリージョナルサービスとして提供されている事自体なんか違和感ありますね。てっきりグローバルサービスかと思ってました。

## re:Post Privateの設定をやってみようと思ったけれど、Organizationの権限が必要で無理でした

結論から言うと、実際のre:Post Privateの設定には、AWS IAM アイデンティセンターの利用が必要なためOrganizationの強い権限が必要となります。自分は、クラスメソッドでそんな権限はもっていないため、ステップ1だけの手順となることご了承ください。

既にAWSのマネジメントコンソールにサービスメニューとして追加されています。

020.png

登録の全ステップは以下の通り。

030.pnt

「プライベート re:Postを作成」ボタンをクリックします。

040.png

料金プランは2種類から選択。

050.png

名前を決めます。re:Post用のドメインに利用するリージョンが影響するようですね。

060.png

re:Postを作成しようとすると、以下のエラーが表示されました。エラー内容的にはSSOがオレゴンリージョンにないと表示されてますが、Identity Centerに対する権限が無いということな気がします。

070.png

## ユーザーガイドの参照ポイント

ユーザーガイドも全てフルで提供されています。

[AWS re:Post Private](https://docs.aws.amazon.com/repostprivate/)

### IAM Identity Centerの利用は必須

[Onboard to re:Post Private through IAM Identity Center \- AWS re:Post Private](https://docs.aws.amazon.com/repostprivate/latest/caguide/onboard-iam-identity-center.html)

re:Post PrivateはAWS IAM Identity Centerと統合されていて、組織メンバーにIDフェデレーションを提供可能です。


### Security

[Data protection in AWS re:Post Private \- AWS re:Post Private](https://docs.aws.amazon.com/repostprivate/latest/caguide/data-protection.html)

企業情報がもろに含まれる想定のre:Post Privateなので、データ保護の観点とIAMでre:Post Privateをどのように制御するのか記載されています。

[How re:Post Private works with IAM \- AWS re:Post Private](https://docs.aws.amazon.com/repostprivate/latest/caguide/security_iam_service-with-iam.html)

しかし、こういうコミュニケーションベースのサービスに対しても、AWSのサービスとしてIAMで制御する姿勢は、思想が一貫してて面白い。

### re:Post Privateの使い方全般

- [Create, configure, and customize your private re:Post \- AWS re:Post Private](https://docs.aws.amazon.com/repostprivate/latest/caguide/create-configure-repost.html)
- [Manage your private re:Post in the re:Post Private console \- AWS re:Post Private](https://docs.aws.amazon.com/repostprivate/latest/caguide/manage-repost-console.html)

re:Post Privateの利用方法や、ユーザー追加・変更設定などの方法が解説されています。

### re:Post Privateの監視

[Monitoring AWS re:Post Private \- AWS re:Post Private](https://docs.aws.amazon.com/repostprivate/latest/caguide/monitoring-overview.html)

なんとまぁ、監視もできます。AWSで監視と言えば、CloudWatchなわけですが、re:Post Private用のメトリクスもしっかり提供されています。

[Monitoring AWS re:Post Private with Amazon CloudWatch \- AWS re:Post Private](https://docs.aws.amazon.com/repostprivate/latest/caguide/monitoring-cloudwatch.html)

提供されているメトリクスは3種類ということで、それほど多いわけではないですが、CloudWatch統合されていることで、既存の運用に載せやすい組織も多いのではないでしょうか。

- <code>NumberOfSpaces</code>
  - The number of private re:Posts in the current account.
  - Units: Count
- NumberOfUsers
  - The number of users in a private re:Post. This metric uses spaceId as a dimension.
  - Units: Count
- ContentSize
  - The amount of content in a private re:Post. This metric uses spaceId as a dimension.
  - Units: Bytes

## re:Post Privateの公開が、組織のAWS活用をさらに推進する！

以上、re:Post Privateのざっくりした紹介でした。

<blockquote>Enterprise SupportおよびEnterprise On-Ramp Supportプランの利用が必須です。
</blockquote>

実際に動かしてみないとどういうことができるのかわかりにくいサービスですが、大規模にAWSを利用している組織では、一定の恩恵を得られると思いますし、最初の6ヶ月は無料利用枠があるということなので、まずはミニマムに設定してはじめてみるのが良いのではないでしょうか。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。
