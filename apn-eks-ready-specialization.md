この度、AWS Partner Networkにおいて、新しくAmazon EKS Ready Specializationが発表されたので、その内容をお届けします。従来EKSのService Delivery Programは提供されていたのですが、今回あたらしく、サービスレディプログラムにEKSが追加された、という内容になります。

## 公式情報

公式情報：[AWS Partner Network launches new Amazon EKS Ready Specialization](https://aws.amazon.com/jp/about-aws/whats-new/2023/11/aws-partner-network-eks-ready-specialization/)

ざっくりした和訳は以下の通り。

<blockquote>Amazon EKS Ready Specilazationは、Amazon EKS、Amaozn EKS-Anywhere(EKS-A)を利用して、AWS、オンプレミス、エッジ環境でKubernetesを実行するソリューションを提供するAPNパートナーにフォーカスするために設計されています。<br />
<br />
Amazon EKS Ready パートナーは、コンテナのセキュリティ、ネットワーク、コスト管理、自動化などの機能を補助し、環境（オンプレミス、AWS）に関わらず顧客を支援します。ただ、EKS Ready パートナーは、AWSのソリューションアーキテクトによりその能力を検証されることが必要になります。</blockquote>

ざっくり言うと、EKSを中心としたKubernetesを中心とした専門知識をもっているAWSのパートナー会社は、事前にAWSのソリューションアーキテクトにより認定された場合に、EKS Ready Partnerとなりその能力を外部から見える形で公開できるとのことです。

## Amazon EKS パートナーページでのサービスレディプログラムリンクの追加

010

Amazon EKS パートナーページには、従来は恐らく無かったAWS サービスレディプログラムへのリンクが追加されています。

[パートナー \- Amazon EKS \| AWS](https://aws.amazon.com/jp/eks/partners/)

[Amazon EKS Partners \- Amazon Web Services \(AWS\)](https://aws.amazon.com/eks/partners/)

020

AWS サービスデリバリープログラムは、主に特定のAWSサービスの提供においての専門知識を持つパートナー向け、AWS サービスレディプログラムは、主にAWSのサービスを利用したソフトウェア・ソリューション（製品フォーカス）に関する経験とリア機があるパートナー向け、となっています。

今回は、EKSについて、このサービスレディプログラムが提供されたということになります。

AWS Service Ready Programのページ([AWS Service Ready Program](https://aws.amazon.com/partners/programs/service-ready/?nc1=h_ls))にアクセスすると、Amazon EKS Ready Partnersのページが追加されています。

030

上記画面から、実際のクライテリアに必要となるRequirementsを確認できます。パートナー向けの内容になっているので、公開はできない情報です。ざっと見てみた感想としては、EKSやEKS Anywhereに対する技術的な裏付けだけではなく製品運用やサポート面など、EKSを軸としたサービスを提供するために必要な要件が広く提示されている印象でした。しっかりした準備が必要と思われる内容になっており、一朝一夕では突破できない雰囲気です。

逆に、このプログラムを取得しているパートナーの製品は、一定以上の信頼があると判断して良さそうです。気になる方は、一度目を通しておくとよいと思います。

## AWSのパートナー向けのRezuirementsを参照することで、能力評価の一つの指標として活用

今回のアップデートをきっかけに、関連するサービスデリバリープログラムやサービスレディプログラムのRequirementsをなんこか見ていたのですが、AWSが特定サービスにおいてパートナーに求める技術水準が記載されていて、ベンダー側としてどういった能力が必要なのかという点を第三者的に見ることができ有意義でした。

これらの認定は、対外的にAWSのお墨付きをもらっていることを証明できるものなので、まだRequirementsを見たことがないパートナーの方は、今一度、このあたりのドキュメントを参照しつつ、自社のケイパビリティを振り返るきっかけにしてもらえればと思います。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。
