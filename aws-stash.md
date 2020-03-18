# AWSの膨大な公式情報を一括して爆速検索可能なサイト「AWS Stash」

<strong>「AWSの公式情報ってめっちゃあるやん。これ、どこから検索すりゃいいの？」</strong>

2006年にサービスを開始したAWS。その公式情報は膨大かつイベントも数限りなく開催されてきました。

最近のイベントはYoutubeなどにも公開されており、公式情報だけにしぼってそれらを一括で検索するのは、各サイトのRSSフィードを一括購読とかしないかぎり困難でした。

そんな悩みを一発でふっとばすのが、この<a href="https://awsstash.com/" target="_blank">
AWS Stash</a>というサイト。

<strong>re:Inventのセッション動画だけではなく、公式ブログやスライド、QuickstartやホワイトペーパーやGitHubまで横断的に爆速で検索可能な素晴らしく便利なサイト</strong>なので、まだ未体験の方は是非一度試してみてください。

<pre style="line-height:120%;">
何でも検索できちゃうの…!?

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>


## 「AWS Stash」とは？

010

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://awsstash.com/" target="_blank">AWS Stash</a>
</p>

一言、<strong>AWSのあらゆる公式情報を一括して爆速で検索可能なサイト</strong>です。基本的にはサイト上段のキーワードを入力し検索するのですが、各種条件指定内容が非常に使いやすくコンテンツ検索が捗ります。

### 検索可能なコンテンツは10種類

020

検索可能なコンテンツが幅広いのが最大の特徴。ブログだけでなく、ポッドキャストやホワイトペーパー、クイックスタート、各種動画も幅広くコンテンツ指定が可能です。

- AWS Blog
- AWS Podcast
- AWS Quick Start
- AWS Solution
- AWS Tech Talk
- AWS Whitepaper
- GitHub
- New Release
- Slides
- Video

### フィルタリング条件

030

4種類の条件でフィルタリングが可能。

#### Level

コンテンツの難易度。re:Inventに行ったことある人ならピンとくるかもですが、AWSのコンテンツのレベルは概ね以下の分類となっています。

- 200 - Intermediate
- 300 - Advanced
- 400 - Expert

最初の導入や初心者向けなのが200番台、中級者なら300番台、その道のエキスパートなら400番台というのが一つの目安です。各サービスの応用的であったりアーキテクチャの一段踏み込んだ内容を学ぶなら400番台を選ぶのがオススメです。

#### Event

これが使いやすい。AWSでは世界中で様々なイベントを開催していますが、それをフィルタリングできます。re:Inventでのフィルタリングはもちろん、セキュリティ系イベントのre:Inforceや、世界中のAWS Summit、パブリックセクター向けのイベントも選択可能です。

#### Published Year

各コンテンツの発表年度。旬なものに限定したいときは重要なフィルタリング条件です。

#### Speakers

これは、需要あるのかな？特定のスピーカーに限定したいときに利用できます。かなり大量の人が登録されているので、誰かお気に入りの人がいる場合、使いやすいかもしれません。

### 以前はどんなサイトだったのか？

実は、以前こちらの記事でこのサイトの前進を紹介していました。

<a href="https://dev.classmethod.jp/cloud/aws/all-reinvent-breakoutsessions/" target="_blank">7年間に渡る歴代re:Inventのセッション動画2340個を爆速で検索するサイト #reinvent ｜ Developers.IO</a>

このときは、re:Inventの動画検索専用サイトだったんですが、ドメインを変えて新しくサイトをリニューアルしており、<code>https://reinventvideos.com</code>から<code>https://awsstash.com</code>にリダイレクトされるようになっています。

[bash]
$ curl -I https://reinventvideos.com/
HTTP/1.1 301 Moved Permanently
Content-Length: 0
Connection: keep-alive
Server: CloudFront
Date: Sun, 23 Feb 2020 14:45:37 GMT
Location: https://awsstash.com
X-Cache: LambdaGeneratedResponse from cloudfront
Via: 1.1 d4ec4fe8ac7dc1717cdfe6977662568f.cloudfront.net (CloudFront)
X-Amz-Cf-Pop: NRT20-C2
X-Amz-Cf-Id: Wze18VIfQ2lcBd0qawc8FR5RNBZDCyS9C6A8FZlPmO9S9mSBKQo5pw==
[/bash]


## 実際に使ってみる

というわけで、実際につかってみた様子をお届けします。最近個人的に気になってるECSのデプロイを調べたくて、「ecs deploy」というキーワードで検索してみます。

040

検索ボタンは特になく、条件入力後即座に検索が実施されます。結果の一覧はこんな感じ。

050

検索結果はカード形式で表示されます。特に検索結果をソートする方法がありません。もう少し最近の情報がほしいので、[Published Year]を2019にしてみます。すると、最近のイベントでのYoutube動画がでてきます。「Best practices for CI/CD using AWS Fargate and Amazon ECS」とかはど真ん中の資料で良いですね。

060

動画以外にも絞りたければ、SlideやQuick Startや、Blogでもコンテンツ選択ができるので、そのあたり試してみるとお目当てのコンテンツに行き当たることができるかと思います。

## このサイトのGitHub

サイトのアーキテクチャは、GitHubに公開されています。

<a href="https://github.com/nragusa/reinventvideos" target="_blank">nragusa/reinventvideos: The code behind reinventvideos.com</a>

このリポジトリでは、サイトのIssueも受け付けているので、サービス利用上なにか気になった点があれば、Issueあげてみるのも良いと思います。

## 気軽にAWSの公式コンテンツを横断的に検索できる最高のサイト

いろいろ試してみたのですが、UIが直感的かつ動作も早く、コンテンツのフィルタリングが非常に便利で、re:Inventの動画を調べたりQuickStartを検索するのが非常に捗ります。公式ブログの情報も指定して検索できるのもありがたい。

AWSの公式サイトではないので未来永劫このサイトが使えるかどうかはわかりませんが、登録されているコンテンツは膨大なので、日々のAWS生活のオプションとして使いこなしてみると良いでしょう。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。



