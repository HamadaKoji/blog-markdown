# ECSの新ネットワーク機能「Service Connect」がリリースされました！

マイクロサービスを実装する時、アプリケーションコード以外にも、他のサービスの健全性把握や名前の管理、それらをどのようにコンテナで運用するか、課題は多いものです。

今回新しくリリースされた「[Service Connect](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-connect.html)」はECSのネットワークに関する新機能です。

[Amazon ECS introduces Service Connect](https://aws.amazon.com/jp/about-aws/whats-new/2022/11/amazon-ecs-service-connect/)

- 任意の名前をサービスに付与して接続
- エンドポイントのヘルスチェック対応
- コンソールやCloudWatchによる豊富なメトリクスの提供
- 自動接続ドレインのサポート

など、ECSでマイクロサービスを運用する場合に必要なネットワーク機能が揃っています。

この記事では、まずは速報で、みんな大好きECSの新機能を紐解いていければと思います！

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     ECSﾏﾂﾘﾀﾞﾜｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>

## Service Connetの機能概要

AWS公式ブログでの紹介はこちら。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="New – Amazon ECS Service Connect Enabling Easy Communication Between Microservices | AWS News Blog" src="https://hatenablog-parts.com/embed?url=https://aws.amazon.com/jp/blogs/aws/new-amazon-ecs-service-connect-enabling-easy-communication-between-microservices/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

今まで、ECS上でマイクロサービスを実現するとき、代表的な3つの選択肢がありましたが、それぞれデメリットもありました。

- ELB
  - 利用はわかりやすいが、追加の設定とインフラコストが発生
- Service Discovery
  - トラフィックメトリクスの収集や、エンドポイントの正当性把握など、追加のアプリケーションコードが必要な場合が多い
- App Mesh
  - 高度な機能を有するが、ECSの外で実行される

このあたりを一気に解決するのが、[Service Connect](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-connect.html)。

動作イメージは、公式ブログから引用したこちらがわかりやすいですね。Cloud Mapは存在しつつ、複数クラスターやVPCに展開されたサービスを、ECS Service Connectを挟んでつなぐことで、アプリケーションコードに影響を与えずに、サービス通信に弾力性を付与しネットワークトラフィックを観測できるとのこと。

010.png

従来のService Discoveryで利用していたCloud Mapが名前空間を提供するので、ロードバランサーの設定は不要。サービス毎に、ヘルスチェック、503エラーの自動思考、コネクションドレインなど、ネットワークに関する一通りの機能も提供されます。また、コンソールで、ネットワークトラフィックに関するメトリクスを提供し、可観測性を向上させています。

## Service Connectのドキュメント

ECSの公式ドキュメントに、新しく、Service Connectが追加されています。

[Service Connect](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-connect.html)


## Service Connectを利用できるリージョンや、関連するサービス

全ての商用リージョンですぐに利用可能！かつ、AWS CloudFormation、AWS CDK、AWS Copilot、AWS Protonで完全にサポートされているため、従来のIaCの流儀に則った運用が可能とのこと。

個人的には、すでにCopilotが対応しているのが素晴らしいですね。ますます便利になりそうです。

ここではｍ，主な制約事項をあげておきます。

### Service Connet利用時の考慮事項

- Windowsコンテナは未サポート
- コンテナインスタンスは最新のもの<code>2.0.20221115</code>以降の利用が必須
- ECS Anywhereの外部コンテナはサポートしない
- タスク定義に追加設定が必要
- サービスに所属するタスクにのみ設定が可能（タスク実行などで起動するタスクは、接続不可）

## Service Connectの料金

特にService Connectの利用にあたって追加のコストは不要で、関連するインフラリソースやトラフィックにかかる費用が従来どおりかかるとのことです。

## ECSにおけるサービス接続の新しい実装として期待

以上、ざーっと、ドキュメントベースで、ECSのService Connectについて紹介してきました。今までECSのサービス間接続においては、設定の手間やわかりやすさから、ALBやService Discoveryを利用していた人も多いと思います。ただ、それぞれの方法にはデメリットもありました。

今回新しく、このService Connectの機能をりようすることで、ネットワークの可観測性と各種ネットワーク健全性の把握、インフラコストの最適化などがそれぞれバランス良く実装されている新機能だと思うので、ECSでマイクロサービスを運用している人は、まずは一度触ってみることをおすすめします。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。
