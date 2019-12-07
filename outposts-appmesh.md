# [速報]App MeshがOutpostsに対応しました！

先日、めでたくAWS Outpostsが一般公開されました。内容は、弊社豊崎のブログを御覧ください。

- <a href="https://dev.classmethod.jp/cloud/aws/aws_outpost_launch_partner_reinvent2019/" target="_blank">[速報]AWS Outpostsが一般公開されました！！ #reinvent ｜ Developers.IO</a>
- <a href="https://dev.classmethod.jp/cloud/aws/aws-outposts_faq_reinvent2019/" target="_blank">AWS Outpostsのよくある疑問についてまとめてみた #reinvent ｜ Developers.IO</a>

これにともない、AWS版サービスメッシュといえる、App MeshもOutpostsに対応したので、その内容をお届け致します。

## App MeshをOutpostsで使うとはどういうことなのか？

公式ドキュメントはこちら。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://docs.aws.amazon.com/app-mesh/latest/userguide/app-mesh-on-outposts.html" target="_blank">App Mesh on AWS Outposts - AWS App Mesh</a>
</p>

### 利用における前提条件

利用するためには、以下の条件を満たす必要があります。

- オンプレミスにOutpostsが既にインストールされていること
- OutpostsとAWSリージョンとの強固な相互通信が可能なこと
- 相互通信しているAWSリージョンがAWS App Meshに対応していること
  - 対応リージョンは<a href="https://docs.aws.amazon.com/general/latest/gr/rande.html#appmesh_region" target="_blank">AWS Service Endpoints - AWS General Reference</a>を確認

基本的にOutpostsは、全てのリージョンで利用可能になる予定です。また、App Meshも東京リージョンに対応しているので、日本のユーザーでもApp MeshをOutpostsで使うことが可能です。

### 利用上の制限

以下のサービスは、Outpostsでは提供されていないため、これらのサービスとコンテナの通信のレイテンシが大きくなる可能性があります。

- AWS Identity and Access Management
- Application Load Balancer
- Network Load Balancer
- Classic Load Balancer
- Amazon Route 53

### ネットワークの考慮事項

AWS OutpostsでEKSを使う場合、以下を考慮する必要があります。

- OutpostsとAWs リージョンの接続が切れても、App MeshのEnvoyプロキシは動作し続けます。しかし、接続が復帰するまでは、サービスメッシュを修復できません
- AWS リージョンとOutpostsとの間には、信頼性が高く低いレイテンシーの品質をもつ回線が望まれます

## OutpostへのApp Mesh Envoy プロキシの実装

010
（引用元：https://docs.aws.amazon.com/app-mesh/latest/userguide/app-mesh-on-outposts.html）

OutpostはAWSリージョンの拡張であり、VPCをマルチAZに拡張可能です。

- <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-on-outposts.html" target="_blank">Amazon Elastic Container Service on AWS Outposts - Amazon Elastic Container Service</a>
- <a href="https://docs.aws.amazon.com/eks/latest/userguide/eks-on-outposts.html" target="_blank">Amazon EKS on AWS Outposts - Amazon EKS</a>

で、上記ページを見てもらえればわかりますが、AWS App MeshをOutpostで使うための、特別な設定方法があるわけではありません。Outpostは物凄く単純に言うと、オンプレ側に独自の拡張AZを設置するサービスなのですが、AWS App　Meshの機能を使う上での特段の制限は無さそうです。

## 利用が一気に広がるApp MeshのOutpost対応

Outpostのユースケースとして、オンプレ環境のデータやリソースを最大限に活かしつつ、AWSのサービスを使うというものがあります。機械学習などの用途でEKSを使う場合、データのアップロードなしにオンプレミス側で処理をするニーズもあるきがします。

その環境でApp Meshが同時に使えるのは、構成を考える上での柔軟性があがって、非常に便利につかえるシチュエーションも多いのではないでしょうか。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

