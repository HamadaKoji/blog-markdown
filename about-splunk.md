# 公式チュートリアルでSplunkの凄さを体感してみた

<strong>「Splunk！有名だけど、一切さわったこと無いんやんなぁ、どないしよ」</strong>

先日、FargateのログドライバーにSplunkが対応したというニュースが飛び込んで来ました。

- <a href="https://aws.amazon.com/jp/about-aws/whats-new/2019/05/aws-fargate-pv1-3-now-supports-the-splunk-log-driver/" target="_blank">AWS Fargate PV1.3 now supports the Splunk log driver</a>

一瞬全ハマコーが歓喜したんですが、Splunk全く触ったことない自分は「え、でもためそうにも環境ないよ？」と手が止まってしまったわけです。

公式サイトにアクセスすると、セットアップ方法やチュートリアルなどのドキュメントがすごい充実してそうだったので、これを機会にチュートリアルを進めながらSplunk触ってみたんですが、<strong>3時間ほど経過した今、すっかりSplunkファンになってしまいました</strong>

というわけで、この記事ではSplunkを全く知らない人向けに、公式チュートリアルをやってみた様子をお届けします。自分の感激が伝わればこれ幸い。

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     Splunk ﾏﾂﾘﾀﾞﾜｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>




## Splunkとは

010

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://docs.splunk.com/Documentation/Splunk/7.2.4/SearchTutorial/WelcometotheSearchTutorial" target="_blank">About the Search Tutorial - Splunk Documentation</a>
</p>

一言で表すと、「あらゆる機器を対象にした統合ログプラットフォーム」。

トップページにいきなり耳慣れない<strong>「ダークデータ」</strong>という言葉が現れますが、これは
「未活用データ」の意味で、この未活用データをログ分析により活用することで、導入企業にさらなる競争力を導く、というわけです。

メインの製品は、主に以下の3種類。

- Splunk Enterprise
	- Splunkのメイン製品
	- ホストインストール型で、インフラは自前で用意
- Splunk Cloud
	- Splunkのクラウド・プラットフォーム
	- インフラの管理不要でログ分析ソリューションを利用可能
- Splunk light
	- Splunk Enterpriseの簡易版の位置付け
	- 利用規模が大きくなってきた時、シームレスにSplunk Enterpriseに移行可能


## Search Tutorialの実施

s020


実際なにができるのかは手を動かしたほうが圧倒的に早いです。公式でSearch Tutorialが用意されていたので、これを元にSplunkで何ができるか試してみました。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://docs.splunk.com/Documentation/Splunk/7.2.4/SearchTutorial/WelcometotheSearchTutorial" target="_blank">About the Search Tutorial - Splunk Documentation</a>
</p>

チュートリアルは、以下の7パートで構成されています。

* Part 1: Getting started
* Part 2: Uploading the tutorial data
* Part 3: Using the Splunk Search app
* Part 4: Searching the tutorial data
* Part 5: Enriching events with lookups
* Part 6: Creating reports and charts
* Part 7: Creating dashboards

この記事では、Splunkを使う上での超絶基礎となるPart1〜Part4までと、検索結果をレポート保存するまでを、チュートリアルをやってみながらその様子をお届けします。ざっと眺めていただくだけでも、Splunkのその柔軟性と凄さはなんとなく把握できるんじゃないでしょうか。

<strong>気が向いたら、是非実際に環境セットアップして、あれこれ試してみることをオススメします。こいつは楽しい！！</strong>


## Part 0「環境セットアップ」

Search Tutorial実施のためには、何はなくともSplunkの動作環境が必要です。いろんな方法が提供されていますが、AWS環境で簡単にSplunkをセットアップする方法を紹介します。

- <a href="https://www.splunk.com/ja_jp/cloud/aws-solutions.html" target="_blank">AWS ソリューション | Splunk</a>

こちらのページにSplunkのAWSにおける各種ソリューションがまとまっているんですが、この中でAMIが用意されています。

<a href="https://aws.amazon.com/marketplace/pp/B00PUXWXNE/ref=srh_res_product_title?ie=UTF8&sr=0-2&qid=1420671234493" target="_blank">AWS Marketplace: Splunk Enterprise</a>

起動に関してはこれが一番簡単なので、ドキュメントを参考にセットアップしてみてください。

セットアップが完了し、ログインして初期画面が表示されれば準備OKです。

sp030

## Part 1: Getting started

<p style="background-color: #e8e8e8;padding: 12px;">
<a href="https://docs.splunk.com/Documentation/Splunk/7.2.4/SearchTutorial/Systemrequirements" target="_blank">What you need for this tutorial - Splunk Documentation</a>
</p>

最初にチュートリアル用のデータファイルをダウンロード。

[bash]
$ curl -O http://docs.splunk.com/images/Tutorial/tutorialdata.zip
$ curl -O http://docs.splunk.com/images/d/db/Prices.csv.zip
[/bash]

もし、環境変数<code>$SPLUNK_HOME</code>が未設定ならそれを追加。

[bash]
$ export SPLUNK_HOME=/opt/splunk
$ export PATH=$SPLUNK_HOME/bin:$PATH
[/bash]


## Part2: Uploading the tutorial data

<p style="background-color: #e8e8e8;padding: 12px;">
<a href="https://docs.splunk.com/Documentation/Splunk/7.2.4/SearchTutorial/AboutgettingdataintoSplunk" target="_blank">About uploading data - Splunk Documentation</a>
</p>

このチュートリアルでは、チュートリアル用のデータが予め用意されていてそれを利用しながら進めることができます。すげぇ便利ですね。こういうログ分析プラットフォームって、最初にみるためのものがないと寂しですからね…

最初に、「設定」→「データの追加」をクリックすると、こんな感じでツアーが始まります。このあたりすべて日本語になっていて非常に親切で驚きます。

sp040

### チュートリアルデータの種類

- access.log

[bash]
75.44.24.82 - - [22/Feb/2019:18:44:40] "POST /product.screen?productId=WC-SH-A01&JSESSIONID=SD7SL9FF5ADFF5066 HTTP 1.1" 200 3067 "http://www.buttercupgames.com/product.screen?productId=WC-SH-A01" "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0; BOIE9;ENUS)" 307
142.233.200.21 - - [22/Feb/2019:19:20:13] "GET show.do?productId=SF-BVS-01&JSESSIONID=SD6SL8FF4ADFF5218 HTTP 1.1" 404 1329 "http://www.buttercupgames.com/cart.do?action=purchase&itemId=EST-13" "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" 674
[/bash]

- secure.log

[bash]
Thu Mar 22 2019 00:15:06 mailsv1 sshd[60445]: pam_unix(sshd:session): session opened for user mdubios by (uid=0)
Thu Mar 22 2019 00:15:06 mailsv1 sshd[3759]: Failed password for djohnson from 194.8.74.23 port 3769 ssh2
Thu Mar 22 2019 00:15:08 mailsv1 sshd[5276]: Failed password for invalid user appserver from 194.8.74.23 port 3351 
[/bash]

- vendor_sales.log

[bash]
[13/Apr/2019:18:23:07] VendorID=5037 Code=C AcctID=5317605039838520
[13/Apr/2019:18:23:22] VendorID=9108 Code=A AcctID=2194850084423218
[13/Apr/2019:18:23:49] VendorID=1285 Code=F AcctID=8560077531775179
[13/Apr/2019:18:23:59] VendorID=1153 Code=D AcctID=4433276107716482
[/bash]

### チュートリアルデータのアップロード

予め<a href="https://docs.splunk.com/Documentation/Splunk/7.2.4/SearchTutorial/Systemrequirements#Download_the_tutorial_data_files" target="_blank">Download the tutorial data files</a>で、ダウンロードしたチュートリアルデータをWebUIからアップロードします。

「設定」→「データの追加」→「コンピュータのファイルからアップロード」をクリック。

s050

「ソースの選択」画面で、<code>tutorialdata.zip</code>をアップロードし、「次へ」ボタンをクリック。

「入力設定」画面が表示される。ここで、ソースタイプ、ホスト、インデックスを指定できる。この指定は非常に重要で、この設定によって、Splunkがこのログをどのように扱うかが決定されます。

ここでは、セグメント番号に1を入力。

s060

最終的に、以下の通りに設定できていれば、「実行」ボタンをクリック。

s070

ファイルがアップロードされると、以下の画面が表示されます。アップロードしたデータをすぐ確認するため、「サーチ開始」ボタンをクリック。

s080

すると、サーチ結果が一気に画面に表示されます。一気にテンションがあがってくるのがここからすね。

s090

## Part 3: Using the Splunk Search app

<p style="background-color: #e8e8e8;padding: 12px;">
<a href="https://docs.splunk.com/Documentation/Splunk/7.2.4/SearchTutorial/Aboutthesearchapp" target="_blank">Exploring the Search views - Splunk Documentation</a>
</p>

s100

サーチ画面の初期表示。ログを探すための一番基本の画面になります。

### データサマリーの探索

「データサマリー」をクリック。現在、Splunkに登録されているデータが、種類別に確認可能。

- ホスト
- ソース
- ソースタイプ

ここでは、「ソース」の<code> tutorialdata.zip:./www1/access.log </code>をクリックすると、該当ログの検索結果が表示されます。

s110

他の検索結果を見たければ、適宜最初のサーチ画面に戻り、別のフィルタリングでログを確認していくのが基本の流れとなります。

### データ取得範囲の指定

s120

検索ボックスには、様々な範囲検索を行うための機能が集約されているので、これらをつかって、取得したいログの日付や時刻の範囲を指定できます。

## Part 4: Searching the tutorial data

<p style="background-color: #e8e8e8;padding: 12px;">
<a href="https://docs.splunk.com/Documentation/Splunk/7.2.4/SearchTutorial/Startsearching" target="_blank">Basic searches and search results - Splunk Documentation</a>
</p>

検索入力欄はインクリメンタルサーチに対応していて、任意の文字列を入力すると自動的に一致する単語の候補を表示してくれます。<strong>これ、めっちゃくちゃ便利ですね。</strong>

下の画像は、検索窓に<code>category</code>を入力したときの様子。

s130

自動的にログの検索対象候補となるcategoryが複数表示されてます。ここで試しに、<code>category=sports</code>をクリックすると、スポーツカテゴリーに該当するログが検索されます。この利用感、ステキ。また、検索履歴を利用したサジェストも可能。

### インデックスからイベントを取得する

s140

例として、<code>Buttercup Games website</code>に、エラーが何個あるかを検索してみます。複数のキーワードを利用して検索するのであれば、Booleanオペレーション（AND,OR, and NOT）が利用できます。

- <code>buttercupgames error</code>は、<code>buttercupgames AND error</code>と同一
- <code>buttercupgames (error OR fail* OR severe)</code>のような複合検索も可能
- <code>fail*</code>で、<code>failed</code>、<code>failure</code>、<code>failed</code>、<code>failing</code>などを検索可能

### 検索結果について理解する

#### Search Result

検索結果を切り替えるには、ドロップダウンから「raw」、「リスト」、「テーブル」を選択可能。デフォルトは「リスト」。

s150

テーブルにすると、フィールド別に値が並ぶレイアウトに。

s160

#### Timeline of events

タイムラインでは、時系列でそのログの量を把握できます。ズームアウトや選択項目にズームの動きが超なめらかで興奮する。

s170

#### Fields sidebar

検索結果のフィールドに関する情報はこちらに集約されています。

s180

- Selected fields
	- 検索結果に含まれるフィールドの一覧
- Interesting fields
	- 検索結果から関連するフィールドとして抽出されたもの
	- クリックすることで関連フィールドの情報を表示

s190

というわけで、検索結果には関連情報を得るためのいろんな情報が表示されていて、なおかつすごい直感的に高性能で各種データを表示できるので、面白いです。是非、これはいろいろ触ってもらいたい。関連フィールドとかも面白いすね。

### 表示フィールドの追加

フィールドサイドバーには、表示フィールドとして3つのみが選択されていますが、他のフィールドの表示も一覧に表示可能です。

フィールドサイドバー部分に「すべてのフィールド」と記載されている部分をクリック。

s200

そうすると、追加で表示対象とするフィールドの一覧が表示されます。

s210

### Search languageについて学ぶ

今まで見てきたとおり、Splunkを使いこなす上で重要なのが、検索クエリに記述するSearch language。これについても、ごっつ丁寧なサーチアシスタントが用意されています。

最初に、メニュー上部のアカウント名から「基本設定」→「SPLエディタ」をクリックすることで、サーチアシスタントの挙動を選択できるので、慣れないうちはここを「完全」にしておくことをオススメします。

s220

この状態で、新規サーチを起動し、検索欄に<code>s</code>を入力すると、一致する検索結果と、簡単な検索方法を表示してくれます。<strong>これ一瞬です。ビビります。</strong>

s230

試しに、検索欄に<code>sourcetype=access_* status=200 action=purchase</code>と入力した後、パイプ<code>|</code>を入力することで、Linuxのパイプのように、前のコマンドの実行結果を受けて、後コマンドを実行できます。

s240

さらに後ろに続けて、<code>top categoryId</code>を入力すると、SQLの集計関数の用に、CategoryIdの数が多い順に集計結果を表示してくれます。すげぇな。

s250

これさらにチャートに一瞬で出来ちゃうんですよね。こんな感じで。

s260

s270

チュートリアルでは、さらに、SQLのサブクエリのようなSubsearchというものも紹介されているので、興味がある人は是非こちらも見てみましょう。ココらへんは、splunkを使いこなす上で必須でしょうね。

- <a href="https://docs.splunk.com/Documentation/Splunk/7.2.4/SearchTutorial/Useasubsearch" target="_blank">Use a subsearch - Splunk Documentation</a>


## Part 6: Creating reports and charts

検索結果はレポートとして保存できます。検索欄に以下のコマンドを入力。

[text]
sourcetype=access_* status=200 action=purchase [search sourcetype=access_* status=200 action=purchase | top limit=1 clientip | table clientip] | stats count AS "Total Purchased", dc(productId) AS "Total Products", values(productName) AS "Product Names" BY clientip | rename clientip AS "VIP Customer"
[/text]

右上の「名前をつけて保存」から、レポートをクリック。

S280

タイトルと詳細をつけて、「作成」ボタンをクリックすると、レポート一覧に追加されます。

s290

s300

最初作成したレポートはプライベート状態になっているので、ログインユーザーしか見れませんが、これを他のユーザーに参照権限を付与することも可能です。

作り方次第では、こんなチャートも作ったりできます。ここまでくれば、自由自在でしょ。

S310


## Splunkのチュートリアルで柔軟で強力なログプラットフォームを垣間見た

もともと、FargateのログドライバーがSplunk対応したのが、Splunkに興味をもったきっかけでした。恥ずかしながら今までSplunk自体を触ったことがなかったので、これを機にチュートリアルをやってみたんですが、やってみて楽しかったです。

- ドキュメントが非常に丁寧で、迷うところが一切ない
- バリエーションが豊富なサンプルデータが用意されているので、いろんな側面からログ分析を試すことができる
- シンプルで美しいUIと軽快な動作が、触ってて楽しくなる

ホストインストール型のSplunk Enterpriseであれば、60日間は無料で使える、かつAWSで利用するにはMarketplaceにAMIが用意されていてセットアップがめちゃくちゃ簡単なので、一度このチュートリアルを実施してもらいながら、Splunkの凄さを体験してみるのも良いんじゃないでしょうか。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://docs.splunk.com/Documentation/Splunk/7.2.4/SearchTutorial/WelcometotheSearchTutorial" target="_blank">About the Search Tutorial - Splunk Documentation</a>
</p>

また、AWS統合のアプリケーションも用意されており、CloudTrail、Config、CloudWatchなどのログも統合管理できるようなので、追ってそちらも試してみてブログ書こうと思います。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。


