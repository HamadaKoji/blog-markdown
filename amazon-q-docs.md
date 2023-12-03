# 新サービス Amazon Q（ビルダー向け）のドキュメントを総なめして、できることの全体像をざっくり把握する

<strong>「Amazon Q、なんかすげぇ…　アルファベット一文字のサービスってAWSでは初めてなのでは？」</strong>

最近、TwitterがXに変わったばかりですが、こういうアルファベット一文字のサービスって流行っているんですかね？AWSには今までこういうサービス名なかったので、新鮮かつAWSのこのQにかける意気込みをひしひしと感じたハマコーでした。

というわけで、re:Invent 2023、AWSのCEOであるAdam SelipskyのKeynoteにおいて、Amazon Qが発表されました。新規サービスとしては、異例の膨大なドキュメントかつ利用用途が多岐にわたるため、その全体像を掴むのが難しいんじゃないでしょうか。

便利そうな機能は山のようにあり、delvelopers.IOにおいても多数のブログ記事が上がっていますが、この記事ではAmazon Qの公式ドキュメントを総ざらいし、できることの全体像を抑えることを目的としています。基本的には、公式ドキュメントの要約という位置づけで、全体像を把握するのにこの記事を役立ててもらえれば幸いです。

このブログでは、Amazon Qの機能のうち、主にビルダー向けドキュメントの内容を解説しています。ビジネスユーザー向けのAmazon Qについては、この記事に含まれていないことご了承ください。

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     ｺﾄｼ ﾊ Amazon Q ﾏﾂﾘﾀﾞﾜｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>

それぐらい勢いのあるKeynoteでの紹介でしたね。

コンソールURL
https://us-east-1.console.aws.amazon.com/amazonq/home?region=us-east-1#welcome


## 公式情報と全体のドキュメント体系

Amazon Qの公式情報トップページはこちら。

[Amazon Q Documentation](https://docs.aws.amazon.com/amazonq/)

まず一番最初にこちらアクセスした時に戸惑う点ですが、ドキュメントは主に以下の2つに別れます。

- [What is Amazon Q \(For Business Use\)? \- Amazon Q](https://docs.aws.amazon.com/amazonq/latest/business-use-dg/what-is.html)
- [What is Amazon Q \(For AWS Builder Use\)? \- Amazon Q](https://docs.aws.amazon.com/amazonq/latest/aws-builder-use-ug/what-is.html)

Amazon Qのユーザー別にドキュメントを分けています。このビジネスユーザーとビルダーユーザーの違い、最初見るとイマイチピンとこないんですが、基本的にはAWSを利用するユーザーと、AWSを使ってアプリケーションを構築するユーザーという分類で問題なさそうです。

以下、ビルダー向けユーザードキュメント別にドキュメントの中身を総ざらいしていきます。


## Amazon Q (For AWS Builder Use) User Guide

[What is Amazon Q \(For AWS Builder Use\)? \- Amazon Q](https://docs.aws.amazon.com/amazonq/latest/aws-builder-use-ug/what-is.html)

Amazon QはAmazon Bedrock上に構築されている。Amazon Qは、人工知能（AI）を搭載した会話アシスタントで、必要に応じて人間とのインターフェースを持ち顧客の質問の文脈に寄り添った回答が可能。

 Amazon Q（For AWS Builder Use）は、AWSコンソールとIDEの2つの環境で利用可能です。このドキュメントでは、AWS コンソールは AWS Management Console、AWS Marketing Website、AWS Documentation pages、AWS Console Mobile Application を指します。IDEはAmazon Qが利用可能な統合開発環境を指します。


### 主な機能

[Amazon Q Features \- Amazon Q](https://docs.aws.amazon.com/amazonq/latest/aws-builder-use-ug/features.html)

- 文脈を保った複数応答の会話
- ソフトウェアの構築、更新、進化
- ソフトウェアコードの変換
- マネジメントコンソールにおけるトラブルシューティングの補佐
- AWSサポートへの問い合わせ
- チャットボット連携
- AWS マネジメントコンソールモバイルアプリとの統合

（ハマコー所感）こうやってざっと眺めただけでも、その機能は多岐にわたるのがわかります。おおよそ、人間がAWSを利用する時に困る場面で、それこそ秘書のようにAmazon Qはたちまわってくれそうです。個人的には、普段全然使ったことがないAWS マネジメントコンソールモバイルアプリとの統合が気になりました。このためだけでも、モバイルアプリ使ってみるのもありかもしれませんね。

### Setting up Amazon Q

[Setting up Amazon Q \- Amazon Q](https://docs.aws.amazon.com/amazonq/latest/aws-builder-use-ug/setting-up.html)

AWS Management Console、AWS Marketing Website、AWS Documentation、AWS Console Mobile ApplicationでAmazon Qを使用するには、<code>AmazonQFullAccess</code>の権限が必要。

### Getting started with Amazon Q

[Getting started with Amazon Q](https://docs.aws.amazon.com/amazonq/latest/aws-builder-use-ug/getting-started.html)

コンソールで利用する場合、上述した<code>AmazonQFullAccess</code>が必要となる。そのIAMポリシーを持っていれば、AWS Management Console、AWS Marketing Website、AWS Documentation、AWS Console Mobile ApplicationでAmazon Qが利用可能。

マネジメントコンソールにログインすると、このようにQのチャット欄が表示されています。めちゃくちゃ目立ちますね、これ。

010

AWSのドキュメントを見ているときでも、このようにQのアイコンが出現していて、クリックすると、チャット欄が出現します。かなり存在感強めですね。このチャット欄から、ドキュメントについての相談ができます。

020


030

試しに、Amazon Qのセキュリティ上の考慮点について聞いてみた様子。ドキュメント内を検索するのも良いですが、こういう会話形式で重要なポイントを抜き出してくれるのはありがたいですね。

040

IDE で Amazon Q の権限を設定するには、AWS Toolkit をインストールし、IAM Identity Center または AWS Builder ID で認証します。詳細はこちら。

[Set up Amazon Q in your IDE \- Amazon Q](https://docs.aws.amazon.com/amazonq/latest/aws-builder-use-ug/q-in-IDE-setup.html)

### Amazon Q in the console

[Amazon Q in the console \- Amazon Q](https://docs.aws.amazon.com/amazonq/latest/aws-builder-use-ug/q-in-console.html)

コンソールにおけるAmazon Qの使い方です。

#### Amazon Qとの連携

Amazon Qに質問するには、Amazon Qパネルのテキストバーに質問を入力すると、参考文献のリンクを含む回答を生成します。回答に対して、サムアップとサムダウンでフィードバックし、回答精度の向上に寄与できます。

Amazon Qは、与えられたセッション内の会話を、今後の回答に役立つコンテキストとして記録しているため、セッションの期間中、フォローアップの質問をしたり、以前の質問や回答を参照したりすることができます。コンソール内の別の場所に移動したり、別のブラウザ、タブ、デバイスでAmazon Qを開いたりすると、以前の会話からのコンテキストなしで新しい会話が始まります。

会話を再開し、以前の質問や回答によって提供されたコンテキストを消去したい場合は、「新しい会話」を選択します。以前の会話はAmazon Qからの回答に使用されなくなります。ブラウザを更新してコンテキストをクリアすることもできます。

質問例はこんな感じ。ドキュメントの項目を探すより、先にAmazon Qに問い合わせする体験がこれから中心になっていきそうですね。

- What’s the maximum runtime for a Lambda function?
- When should I put my resources in a VPC?
- What’s the best container service to use to run my workload if I need to keep my costs low?
- How do I list my Amazon S3 buckets?
- How do I create and host a website on AWS?

#### Troubleshoot console errors with Amazon Q

[Troubleshoot console errors with Amazon Q \- Amazon Q](https://docs.aws.amazon.com/amazonq/latest/aws-builder-use-ug/troubleshoot-console-errors.html)

AWSのマネジメントコンソールでエラーが発生したときに、発生したエラーに対するトラブルシュートをAmazon Qで実施できます。ほとんどの一般的なコンソールエラーのトラブルシュートが可能とのこと。

2023年11月30日現在、オレゴンリージョンでのみ利用できるらしいので、別途試してみようと思います。想像しただけでも便利そうですよね。追加のIAMポリシーも必要になる場合があるとのこと。詳細は、こちらに記載があります。

[Manage access to Amazon Q with IAM policies \- Amazon Q](https://docs.aws.amazon.com/amazonq/latest/aws-builder-use-ug/security_iam_manage-access-with-policies.html)

#### Chat with AWS Support

[Chat with AWS Support \- Amazon Q](https://docs.aws.amazon.com/amazonq/latest/aws-builder-use-ug/support-chat.html)

Amazon Qを使用して、AWS Support Centerコンソールを含むAWS Management Consoleのどこからでもサポートケースを作成し、AWSサポートに連絡することができます。Amazon Qとの会話コンテキストから自動的にサポートケースを作成可能。利用できるAWSサポートは、アカウントのサポート階層によって異なる。

前提条件として、Amazon Qの利用権限と、AWS Supportケースの作成権限が必要です。

実際のサポートケースの作成方法ですが、こんな感じで、サポートに問い合わせたい、といった質問を入力することで、サポートケース入力のフォーマットをAmazon Q上で表示できます。この内容を埋め込んでいくと、サポートケースが起票される仕組みになっていると。事前にAmazon Qと話していた場合は、そのコンテキストから自動で入力項目が埋まっていることもありそうです。これ、めちゃくちゃ便利ですよね。

050

### Amazon Q in IDEs

[Amazon Q in IDEs \- Amazon Q](https://docs.aws.amazon.com/amazonq/latest/aws-builder-use-ug/q-in-IDE.html)

いくつかのIDEでは、Amazon QはAmazon CodeWhispererを介してAWS Toolkitの一部として利用可能です。この環境では、Amazon Qは、AWS上でのビルド、ソフトウェア開発とコードに関する一般的な質問への回答、コードの生成、コード言語バージョンの更新、特定のコードスニペットの説明、リファクタリング、最適化など、ソフトウェア開発プロセスを支援する機能を含んでいます。

対応するIDEは、現在、Visual Studio CodeおよびJetBrains。

Visual Studio Codeの場合、このようにAWS Toolkitプラグインをインストール。

060

認証方法な、Builder IDとIAMを使う場合の2種類あり、それぞれで利用に必要な権限と機能が異なるとのこと。Builder ID で認証した場合、Amazon Q の機能開発と Code Transformation 機能は使用できません。

070

### Amazon Q and other AWS services

[Amazon Q and other AWS services \- Amazon Q](https://docs.aws.amazon.com/amazonq/latest/aws-builder-use-ug/q-and-aws-services.html)

以下のサービスと連携できます。多方面でAmazon Qが活用可能になっているので、他の機能でのインテグレーションをさらに深ぼってみたい方はこちらを参考にしてみてください。

- Amazon Q (For Business Use)
  - Amazon Q (For Business Use)は、完全に管理された、ジェネレーティブAIを搭載したエンタープライズチャットアシスタント
  - 詳細：[What is Amazon Q \(For Business Use\)? \- Amazon Q](https://docs.aws.amazon.com/amazonq/latest/business-use-dg/what-is.html)
- Amazon Q in AWS Chatbot
  - AWS Chatbotで設定したSlackやMicrosoft TeamsのチャンネルでAmazon Qを有効にして、AWSやAWSサービス、AWSでの構築に関する質問をすることができます
  - 詳細：[Asking AWS Chatbot questions from chat channels \- AWS Chatbot](https://docs.aws.amazon.com/chatbot/latest/adminguide/asking-questions.html)
- Amazon Q in Amazon CodeCatalyst
  - Amazon CodeCatalystのAmazon Q機能開発機能は、課題を割り当てることができる生成的なAIアシスタント。Amazon Qに課題が割り当てられると、そのタイトルと説明に基づいて課題を分析し、指定されたリポジトリ内のコードをレビューする。アプローチを作成することができれば、プルリクエストでユーザーが評価できるようなドラフトソリューションを作成。
  - 詳細：[Tutorial: Using CodeCatalyst GenAI features to speed up your development work \- Amazon CodeCatalyst](https://docs.aws.amazon.com/codecatalyst/latest/userguide/getting-started-project-assistance.html)
- Amazon Q in Amazon Connect
  - Amazon ConnectのAmazon Qは、生成的なAIカスタマーサービスアシスタント。これは、コンタクトセンターのエージェントが迅速かつ正確に顧客の問題を解決するのに役立つリアルタイムの推奨事項を提供するAmazon Connect WisdomのLLM強化された進化版。
  - 詳細：[Use Amazon Q in Connect for generative AI–powered agent assistance in real\-time \- Amazon Connect](https://docs.aws.amazon.com/connect/latest/adminguide/amazon-q-connect.html)
- mazon Q in Amazon QuickSight
  - Amazon QuickSightのAmazon Qは、販売、マーケティング、小売に関するフレーズを含む、あなたが仕事の一部として毎日使用するビジネス用語を理解するように最適化されており、あなたのビジネス上の質問に素早く答えることができます。
  - 詳細：[Answering business questions with Amazon QuickSight Q \- Amazon QuickSight](https://docs.aws.amazon.com/quicksight/latest/user/working-with-quicksight-q.html)
- Amazon Q in Reachability Analyzer
  - Amazon Qにネットワーク接続の問題の解決を依頼すると、Amazon VPC Reachability Analyzerと連携して接続をチェックし、ネットワーク構成を検査し、潜在的な問題を特定、その後、Amazon Qは問題の解決方法やさらなる診断方法についてガイダンスを提供します。Amazon QとReachability Analyzerの会話機能を一緒に使用することで、ネットワーク接続の問題を素早く修正する直感的な方法が生まれます。
  - 詳細；[Amazon Q network troubleshooting \- Amazon Virtual Private Cloud](https://docs.aws.amazon.com/vpc/latest/reachability/amazon-q-network-troubleshooting.html)

### Security in Amazon Q

[Security in Amazon Q \- Amazon Q](https://docs.aws.amazon.com/amazonq/latest/aws-builder-use-ug/security.html)

このドキュメントは、Amazon Q を使用する際に責任共有モデルを適用する方法を理解するのに役立ちます。また、Amazon Q リソースの監視とセキュリティ確保に役立つ他の AWS サービスの使用方法についても説明します。

### Monitoring Amazon Q

[Monitoring Amazon Q \- Amazon Q](https://docs.aws.amazon.com/amazonq/latest/aws-builder-use-ug/monitoring-overview.html)

CloudTrailによる、Amazon Qの利用状況を監視できます。CloudTrailのログ監視内容の詳細はこちら。

[Logging Amazon Q API calls using AWS CloudTrail \- Amazon Q](https://docs.aws.amazon.com/amazonq/latest/aws-builder-use-ug/logging-using-cloudtrail.html)


### Supported Regions for Amazon Q

[Supported Regions for Amazon Q \- Amazon Q](https://docs.aws.amazon.com/amazonq/latest/aws-builder-use-ug/regions.html)

AWS Management Console、AWS Marketing Website、AWS Documentation、AWS Console Mobile Application、AWS ChatbotのAmazon Qが利用できるリージョン。こうやって眺めてみると、通常の商用リージョンの殆どで利用可能なようです。

- US East (Ohio)
- US East (N. Virginia)
- US West (N. California)
- US West (Oregon)
- Asia Pacific (Mumbai)
- Asia Pacific (Osaka)
- Asia Pacific (Seoul)
- Asia Pacific (Singapore)
- Asia Pacific (Sydney)
- Asia Pacific (Tokyo)
- Canada (Central)
- Europe (Frankfurt)
- Europe (Ireland)
- Europe (London)
- Europe (Paris)
- Europe (Stockholm)
- South America (São Paulo)



## その他参考資料

- developers.IOのAmazon Qに関するブログ一覧。
  - [Amazon Q の記事一覧 \| DevelopersIO](https://dev.classmethod.jp/tags/amazon-q/)

## AWSの他のサービスとも密接に連携して、Amazon Qが利用可能。

以上、ざっくりとしたビルダー向けドキュメントの内容紹介でした。他のサービスとの連携もかなり深いレベルで統合されているので、リリースされたばかりなのに非常に多方面で利用できることが確認できるかと思います。

今後も、Amazon Qを触りまくって、業務でのうまい利用方法を模索できていければ！

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。



