# Lambdaのコールドスタートを解決するLambda SnapStartのセッションに参加してきた(SVS320)　#reinvent

先日のre:Invent2022のキーノートにおいて、Lambdaのコールドスタートを大幅に改善する新機能[Lambda SnapStart](https://aws.amazon.com/jp/blogs/aws/new-accelerate-your-lambda-functions-with-lambda-snapstart/)」が発表されました。

すでにAWSの公式ブログや、[かずえ](https://dev.classmethod.jp/author/kazue-masaki/)による[Lambda SnapStartの解説ブログ](https://dev.classmethod.jp/articles/lambda-snapshot-reinvent/)も出ていますが、この記事では、その新機能のセッションを聴講する機会があったので、内容をお届けします。


<pre style="line-height:120%;">
Lambda SnapStartってすごいの？…!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>

（セッション内容はだいぶ難しかったです…後述してますが、オンデマンド視聴できるようになったら追記予定）

## セッション情報

010.png

### セッション概要

[NEW LAUNCH!] AWS Lambda SnapStart: Fast cold starts for your Java functions SVS320

<blockquote>In this session, learn about the newly released AWS Lambda SnapStart service. With new innovations, AWS SnapStart makes your Java-based function cold starts up to 10x faster, typically with no changes to your function code.<br /><br />
本セッションでは、新しくリリースされたAWS Lambda SnapStartサービスについて学びます。新しいイノベーションにより、AWS SnapStartはJavaベースのFunctionのコールドスタートを最大10倍高速化します。通常、Functionのコードに変更を加えることなく利用することができます。
</blockquote>

### セッションスピーカー

020.png

Mark Sailes, Specialist Solutions Architect, Serverless, AWS


## セッション内容

010.png

皆さん、こんにちは。来てくださって本当にありがとうございます。こんなにたくさんの方にお会いできるなんて、なんだか圧倒されますね。

私の専門は、API Gateway, Lambda, SNS, SQSなどです。また、私は本当に長い間、Java、プログラミングを楽しむ。大学時代からずっと、AWS lambdaを導入しようとしている、Javaアプリケーションをたくさん持っているお客さんと一緒に仕事をしています。

もしあなたがLambdaでJava Runtimeを使ったことがあるのであれば、これはそんなあなたのためのセッションです。今回リリースされた、SnapStartは本当にクールで最高でファンタスティックです。

そして願わくば、私たちはその旅をより簡単にしたい。ちょっとだけ、簡単に。私はもう20年くらいJavaを使っていると言いましたが、それは信じられないくらい長い時間のように思えます。

ここ数年、Javaの人気が再燃しているように感じます。その理由のひとつは、ここ数年でJavaに導入されたリリース・ケイデンス・チームにあると思います。リリースとリリースの間に長い期間があったのが、今では毎年2回リリースされるようになりました。Javaに新しいイノベーションの波が押し寄せています。

030.png

しかし、JavaでLambdaを動作させる時、このようなコールドスタートに苦労した人は多くいるのではないでしょうか。

040.png

これは、話をわかりやすくするため誇張した表現にしていますが、傾向としては正しいものです。大多数のリクエストはとても速く動作していますが、少数のリクエストは遅くなります。顧客向けにAPIを実装したり、応答性の高いインタラクションなアプリケーションが必要な場合、これは本当にイライラしますね。わかります！


そこで、SnapStartの新機能を実証するために、非常にシンプルな Spring Boot アプリケーションを作成しました。APIゲートウェイからリクエストを受け取り、Sprint Boot Functionを起動します。そして、DynamoDBに情報を保存し、取得します。

050.png

多くのSpring Bootアプリケーションのコントローラと同じように、このサンプルは多くのMVCアクションを備えています。サービスにはいくつかのビジネスオブジェクトがあります。そして、MongoDBという別のクラスを持っています。


ほとんどの場合、起動は非常に速く、30ミリ秒前後で終わります。でもたまに、非常にレスポンスが遅くなる場合があります。この例では6.5秒かかっています。これがコールドスタートです。

060.png

こういったコールドスタートをできるだけ減らし、応答性の高いアプリケーションを実現したいものです。

一旦、ここで、なぜこのようなコールドスタートがおこるのか、その理由を把握しておきましょう。Javaにはどのような特徴があるのでしょうか。

070.png

Lambdaの実行環境を作るとき、Firecrackerのインスタンスを作り、そのコードをダウンロードしてJavaランタイムを再起動させています。もしあなたがDIを使用するフレームワークを使っているなら、それはかなり多くのリフレクションを実行するため時間がかかります。また、Javaはバイトコードを受け取ったのち、マシン上で動作するようにコンパイルしています。これは実行環境を作るときに起こります。

この問題に対処するため、様々なアプローチが必要となっていました。こちらのスライドに現在のアプローチを記載しています。

080.png

基本的なアプローチは全てを、あらかじめ完了させておくことです。Javaコードのコンパイルを先行して行い、ネイティブな実行ファイルを生成します。最適化は、コールドスタートを改善するための努力にほかなりません。

この領域でイノベーションを起こすことで、お客様のお役に立てるのだと思います。全く違うものを提供するために。どのようにすれば、より高速なコードが作成できるのか、

090.png

そのため、私たちは、お客様が簡単に高いスケーラビリティと応答性を構築できるようにしたいと思います。

Lambda SnapStartの登場です。

095.png

動作原理を見ていきましょう。

100.png

Functionをプロビジョニングすると、コードを初期化し、仮想マシン全体のスナップショットを取得します。そのスナップショットの中にFunctionがあり、そのスナップショットをロードして、低レイテンシーで検索できるようにします。

このプロセスの実行中、Functionは保留状態になっています。この状態では、Funcitonの実行はできません。一度起動されるとアクティブな状態に遷移し実行できるようになります。

あなたが投票に来たとき、それはスナップショットから実行環境を復元し、その後、ローカルパブリックになる関数。そして、今日と同じように、本当に素早くモトスピーチ環境をスコアリングできること。あるものについても同じことが言えます。そのラムダバージョンのすべての将来の実行環境は、複数のレビューを見るために作成されます。

レーダー下で、それでは少し深掘りしてみましょう。統一されたので、呼び出しレベルが少し変わったという話をします。この講義は同じようなステップを行うことになっていましたが、スナップショットが取られる前に実行したい特定のコードを実行する余分な機会も与えるつもりです。そして、なぜそのようなことが起こったのかについてお話します。

しかし、私たちがテストしたところ、この方法では、特定の時間にその特定のコードを読む機会をお客様に与えることができず、何が本当に有用なのかがわかりません。そこで、アフター・スナップショットのための特定のルック

スナップショット後 これはスナップショット前とスナップショット後のフックです。私のラムダ関数では、私の指はHTTPでリクエストを受け、DynamoDBに書き込んでいます。

私は7秒ほどのタイムスタンプから、私のルータをホストしている患者のタイムスタンプに移動しました。

これは本当に、本当にクールな改善なのですが、試してみると本当に面白いと思うのです。それでは、どのように機能を注文してみるかを発見してみましょう。一般的に、設定は新しい項目となり、公開バージョンと呼ばれる1つのオプションがあります。

もしユーザーバージョンを使っていないなら、これは新しいことではありません。あなたが2つのサービスを統合している場合、例えば、任意のプレスロンドンへのAPIゲートウェイは、私たちは私の関数バージョンでサービスに接続するために使用されている場合があります。

140.png

あなたは、そのプロセスの手順は、バージョンまたはそのバージョンのエイリアスを有効にしていることを指定する必要があります。それらのいずれかを指定しない場合は、スナップ開始機能を呼び出すことはありません、それはあなたの通常の関数を呼び出します。もし、初めてこの機能を試してみて、何の変化も見られなかった場合は、次のことを再確認してください。フォーチュン・バージョンであることを確認してください。さて、フックについて説明しました。

では、実際にどのように実現するのかを説明します。

141

スナップショット処理の前後にコードを実行する機会があります。これはJavaで書かれたハンドラの例です。そして、ここにインターフェースを追加しました。これはオープンJDKで利用可能なインターフェイスで、cracというプロジェクトの一部です。少なくとも、OpenJDKプロジェクトは外部依存です。

その結果、2つのメソッドが追加されていることがわかります。これはすべてのハンドラに追加する方法です。そして、いくつかのメッセージを出力しています。

ではなぜステップの前後にコードを走らせたいのでしょうか？それは、考慮すべき事柄のいくつかをお伝えしたいからです。

143-1

Network connections

ラムダ関数のタイムアウトがデータベースの中で発生する可能性があります。理由はいくつもあり得ます。

143

Ephemeral data

145

この例では、機密情報について説明します。DB接続パスワード、あるいは同様のアプリケーションの固有設定を作成するかもしれません。これらは秘密にしておきたいのですが、もし機密情報がスナップショットから取得できるような場合も有りえます。ので、エラーになったり、無限に不正な認証情報を取得される可能性があることを気をつけておく必要があります。


Uniqueness

146

私たちはスナップショットから新しい実行環境を作っています。これが問題を起こすことはよくありえます。例えば、乱数を生成するとき同じシードを持ちたくはありません。

AWSはAmazon LinuxをオープンSSLに更新して、スナップショット処理に弾力性を持たせるようにしました。乱数を生成するためにホストのベストプラクティスを使用している場合、これはそれを行うための方法です。

具体的には、こちらで紹介している、<code>java.security.SecureRandom</code>の利用を検討してください。

以下が、上記の結果になります。

147

平均的なレスポンスタイムは全て8ミリ秒です。これは全ての結果で同じです。SnapStartがない場合、p99.9において、5000ミリ秒を超えていて、これは非常に長いコールドスタートが発生していることを表しています。

SnapStartを利用することで、p99.9の値でも500ミリ秒を維持しており、ほぼすべてのケースにおいて、コールドスタートを回避できていることが示されています。

### Java利用において活用してもらいたいツール郡

148

149


普段Javaを利用していて、Lambdaに興味をお持ちの方は、是非こちらのツールを活用してみてください。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="Homepage - AWS Lambda Powertools for Java" src="https://hatenablog-parts.com/embed?url=https://awslabs.github.io/aws-lambda-powertools-java/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

### 最後に

私たちは、コンテナには決してフォーカスしていません。私たちは、Lambdaのためのベストプラクティスを作っているだけなのです。私たちはコミュニティの声に耳を傾けてきました。

## セッションを聞いた所感

以上、セッション会場で聞いた内容を手元メモと画像を参考に起こしてみました。

Javaの起動方式の説明から、SnapStartがどう動いて何を解決しようとしているのかを丁寧に説明してもらい理解が深まりました。ただ、セッションを聞いていて正直理解ができていない部分や、聞きそこねた部分、特に最後にJavaの内部構造についての話があったのですが、私の実力不足でその部分を大きく省略しています。

恐らく、このセッションも追ってオンデマンドで視聴できるようになると思うので、そのときは不足部分を追記させていただきます。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

## Lambda SnapStart関連情報

以下、Lambda SnapStartについての、公式ブログです。以下も合わせて参考にしてもらえれば理解が深まるかと思います！

AWSの公式ブログ。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="New – Accelerate Your Lambda Functions with Lambda SnapStart | AWS News Blog" src="https://hatenablog-parts.com/embed?url=https://aws.amazon.com/jp/blogs/aws/new-accelerate-your-lambda-functions-with-lambda-snapstart/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

AWSの公式ドキュメント。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="Lambda execution environment - AWS Lambda" src="https://hatenablog-parts.com/embed?url=https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html#runtimes-lifecycle" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

かずえさんが書いた、アップデートブログ。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="[アップデート] JavaのLambda関数の実行を高速化するLambda SnapStartがリリースされました #reinvent | DevelopersIO" src="https://hatenablog-parts.com/embed?url=https://dev.classmethod.jp/articles/lambda-snapshot-reinvent/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>