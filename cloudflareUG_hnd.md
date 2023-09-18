# （仮）CloudFlareUGハンズオン開催報告 #CloudflareUG

<strong>「最近、Cloudflare、ごっつ勢い感じるで！」</strong>

そんな噂が巷で飛び交う昨今、Cloudflareのミートアップの東京キックオフが、クラスメソッド日比谷オフィスで開催されたので、1参加者としてのレポートをお届けします。2時間という短い時間でしたが、Cloudflareの基礎から、代表的な機能やユースケース、そして、圧倒的に簡単でびっくりするハンズオンからの懇親会など、その魅力が凝縮されたイベントになってました。

<pre style="line-height:120%;">
これがCloudflareの勢いなのか…!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>

そんな感じでしたYO

## イベント概要

[Cloudflare Meetup Tokyo Kick Off \!\! \- connpass](https://cfm-cts.connpass.com/event/275461/)

### タイムスケジュール

| 時間 | 内容 | 登壇者 |
| --- | --- | --- |
| 18:15〜 | 受付開始 | - |
| 19:00-19:05 | オープニング | 運営 山口正徳 |
| 19:05-19:10 | 会場説明 | クラスメソッド 大栗宗 |
| 19:10-19:30 | Cloudflareとは？ | Cloudflareエバンジェリスト 亀田治伸 |
| 19:30-19:45 | CloudflareのPages/Workersとは？ | 運営 森茂洋 |
| 19:45-19:55 | 事例：デジタル待合室利用の事例を紹介 | 運営 武田可帆里 |
| 20:05-20:50 | ハンズオン | 運営 新居田晃史 他 |
| 20:50-20:55 | クロージング＆アンケート回答 | 運営 新井雅也 |
| 21:00 | 解散 | - |

### 会場の様子

会場は、クラスメソッドの日比谷オフィスでの開催となりました。2023年4月から本格稼働開始ということで、自分自身もまだ慣れていないところがあり、いろいろてんやわんやでしたね。

今回のミートアップのスタッフの皆さんと。

010.png

メイン会場の様子。窓の奥には東京タワーが垣間見えます。よござんすねぇ。

020.png

いっつも元気なエバンジェリスト亀田さん。今日もその亀田節は健在でした！

030.png

オープニングは、運営の[山口さん(@kinunori)](https://twitter.com/kinunori)より。本日のミートアップの位置づけやタイムテーブルの説明がありました。

040.png

その後、クラメソ社員大栗より、会場のファシリティ面の案内があったあと、ミートアップの開始となりました。


## 「はじめてのCloudflare　高速化とセキュリティの両立を行うために」　

050.png

（資料は、亀田さんから共有された時点で、ここにあげます）

最初にCloudflareエバンジェリストの亀田治伸さんより、「はじめてのCloudflare〜」と題して、そもそもCloudflareについてあまり馴染みがない人向けの、基調講演の趣があるセッションで始まりました。

実は自分も、正直Cloudflareのことほとんど知らなかったので、新鮮な情報がつまっていてめっちゃ楽しかったです。普段からいわゆるエッジサービスであるAmazon CloudFrontはよく触っていたので、Cloudflareもそれに近いものかなぁと漠然と考えていたのですが、良い意味で裏切られました。

そもそもの一般的なパブリッククラウドとの設計思想の違いや、単なるオリジンのエッジキャッシュサービスには収まらない「インターネットをより安全にしたい」という思想や背景が伝わってきて、Cloudflareが世の中にどういった価値を提供しようとしているのかが、おぼろげながら見えてきました。

Cloudflareの個別の機能を追う前にこういった企業の存在意義や背景を知っておいたほうが、提供されている機能を使うとき、よりユースケースにあったものを選択できると思います。そう言った意味でも、これからCloudflareを触ろうとしている全人類が聞いておくべき内容だったんじゃないでしょうか。

というわけで、資料公開されたら、また、こちらに記載させていただきます！まじで良い資料でした。

## 「Cloudflare Pages/Workersとは」

060.png

続いて、[森茂洋（@\_himorishige）](https://twitter.com/_himorishige)（株式会社microCMS所属）さんより、Cloudflareのメイン機能と言って良いでしょう。Cloudflare Pages/Workersのセッションでした。


Workersは、これぞCloudflareという匂いがするサービスで、従来はオリジンのキャッシュを置く場所というイメージが強いエッジ環境に、がっつりアプリケーションを動作させようという思想のサービスです。

一番印象的だったのが、このWorkersのユースケース。

070.png

アイディア次第で、従来のオリジンサーバーで処理していた様々なビジネスロジックをエッジ環境で動かすことができる夢が広がるのが視野が広くなり良かったです。特に、Workersと連携するデータストレージが豊富に用意されていることもあり、ステートフルなアプリケーションもエッジ環境だけにデプロイすることが可能になっているのは驚きです。

080.png

従来のパブリッククラウドだけを使っているだけではでてこなかったユースケースが沢山紹介されており、まずは代表的な使い方を把握するのに、非常に有用なセッションでした。

登壇資料はこちらから、参照できます。

<script defer class="speakerdeck-embed" data-id="b621c0b4a694467ca05d3f499f705a3c" data-ratio="1.77725118483412" src="//speakerdeck.com/assets/embed.js"></script>

## 「あの頃数百自治体のコロナワクチン 予約フォームを救ったWaiting Roomの運用」

090.png

続いて、[武田可帆里（@taketakekaho）](https://twitter.com/taketakekaho)
さんより、CloudflareのWaiting Roomというサービスの紹介がありました。

コロナワクチンの摂取予約という、世間を代表するユースケースにおいて、Cloudflareがどのように役に立ったのか？を、その課題感から実際に設定してみた様子、使ってみた振返りがコンパクトにまとまっておりめっちゃ面白かったです。

Waiting Roomは、アクセス待機の順番管理を可能にするサービスで、コロナワクチン接種予約のようなアクセス集中するWebサイトの前段におくことで、オリジンサーバへのトラフィックを制御するものです。

100.png

一番おもしろいと思ったのが、Waiting Roomと、元のWebサイトの間の依存関係が非常にすくない点ですね。凄く割り切ったエッジが効いた設計になっていると感じました。

- Waiting Room側のユニークユーザ管理は、Waiting Room側独自管理で、オリジンサーバー側のユーザーとは連動しない
- Waiting Room側では、オリジンサーバの負荷状況は把握しない

という割り切った設計のため、導入までが非常に早いです。設定項目これだけって凄くないすか？

110.png

実際に運用してみて感じた課題も「それな！なるへそなぁ」と思うものばかりで、凄く参考になりました。この先、Waiting Roomを使おうと思う人にはマストな資料だと思います。

登壇資料はこちらから参照できます。

<script defer class="speakerdeck-embed" data-id="b621c0b4a694467ca05d3f499f705a3c" data-ratio="1.77725118483412" src="//speakerdeck.com/assets/embed.js"></script>

## 「ハンズオンでCloudflareにTODOアプリをデプロイしよう！！」

120.png

最後に、新居田晃史（JBアドバンスト・テクノロジー株式会社所属）さんより、全員参加のハンズオンの時間がありました。事前にConnpassで共有されているハンズオン資料を元に、各自がもくもくと、Cloudflare WorkersにTODOアプリをデプロイしていきます。

実際やってみて感じるんですが、めっちゃ簡単です。環境作るところでちょっと手こずるかもしれませんが、それが終われば、あとはシンプルなコマンドと手順で、Cloudflare上にアプリケーションを実装することができます。

こんなのがあっという間にデプロイ可能。

130.png

当日利用したハンズオン資料はこちら。だれでも試せるので、興味がある方はぜひやってみることをおすすめします。Workersのデプロイの簡単さに胸打たれることでしょう。

1. Cloudflareアカウント新規登録[【超入門】Cloudflareアカウントを新規登録する](https://zenn.dev/taketakekaho/articles/5f72f4c58ab0ba)
   - ※二要素認証設定は必須ではありません。※ハンズオンの時間を有効に使っていただくために、事前のアカウント作成を強く推奨させていただきます。 こちらのリンクを参考にしてアカウント作成をお願いします。
2. [Windows 環境でCloudflare 開発ツール Wranglerを設定する方法とHello World!の実行まで](https://zenn.dev/kameoncloud/articles/1fac9762aab4ec)
3. [Cloudflare Workers と KV でTodoListアプリを作る](https://zenn.dev/kameoncloud/articles/7236a2c6ad35c0)

## そして懇親会へ…

ハンズオン終了後、会場は締めて懇親会に流れ込んだわけですが、懇親会めっちゃ人多かったです。多分、40人以上は居たんじゃないかなぁ。オフィス周辺はあまりでかい箱の居酒屋はなかったので、新橋近くのHUBや居酒屋に分かれて、みんなでわいわいやってました。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">カーンパーイ！！！<a href="https://twitter.com/hashtag/CloudflareUG_hnd?src=hash&amp;ref_src=twsrc%5Etfw">#CloudflareUG_hnd</a> <a href="https://twitter.com/hashtag/CloudflareUG?src=hash&amp;ref_src=twsrc%5Etfw">#CloudflareUG</a> <a href="https://t.co/wYka2hwuHv">pic.twitter.com/wYka2hwuHv</a></p>&mdash; たけだ (@taketakekaho) <a href="https://twitter.com/taketakekaho/status/1650845809330753538?ref_src=twsrc%5Etfw">April 25, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

なんかこういうところにまじっていると、つくづく、コロナも過去のものになりつつあるなぁと感じました。色んな人達と喋る機会があったので、自分としても楽しかったです。当日、お話できた方々、ありがとうございました！


## Cloudflareコミュニティの熱量の高さを感じたミートアップ

全体通した感想は、一言これにつきます。なんというか、まぁみんな楽しそうでしたね。運営の方々も参加者の方々も、このCloudFlareというサービスに興味津々で、ちょうどいまここにムーブメントが起きつつある雰囲気を感じました。今回は1参加者という立場でしたが、これは運営もおもしろそうすね。えぇ。

運営面では、しばらくマイクが安定しない、机の数が足りないなど反省点もありましたので、今後の改善に努めていければと思います。

今後もCloudflare、積極的にイベントがあるようで、7月には勝浦で合宿なんかもあるらしいので、今回和っしょい的な気持ちになった人は、こっちも参加してみると良いんじゃないでしょうか。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">7-15にCampイベントを開催します。その名も「Cloud in the Camp 勝浦」<a href="https://t.co/aEboPvXeVs">https://t.co/aEboPvXeVs</a></p>&mdash; kame (@kameoncloud) <a href="https://twitter.com/kameoncloud/status/1650391381079367680?ref_src=twsrc%5Etfw">April 24, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

それでは、今日はこのへんで。濱田（ハマコー）（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。



## その他イベント資料やトゥギャッターなど

当日のtogetterです。凄く積極的にツイートがされていて、雰囲気がよく分かるtogetterになっているかと。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="Cloudflare Meetup Tokyo Kick Off !! - Togetter" src="https://hatenablog-parts.com/embed?url=https://togetter.com/li/2133435" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

[みのるん☁️（@minorun365）さん](https://twitter.com/minorun365)のイベントレポートです！ハンズオンおさらいのお絵かきが概念を理解するのにすごい有用なので、参考になります。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="勉強会レポ：Cloudflare Meetup Tokyo Kick Off !! - Qiita" src="https://hatenablog-parts.com/embed?url=https://qiita.com/minorun365/items/f94dcf46c60fd08bf5ea" width="300" height="150" frameborder="0" scrolling="no"> </iframe>








