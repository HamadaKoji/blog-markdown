# 【狂宴再び】変態ミートアップ、Container build meetup #2に参加してきた #container_build

コンテナ大好きな皆さん、こんなマニアックなイベントがあったのをご存知でしょうか？

<a href="https://dev.classmethod.jp/tool/docker/docker-build-meetup-1/" target="_blank">【docker buildのマニアックすぎる狂宴】Container Build Meetup #1に参加してきた #container_build ｜ DevelopersIO</a>

一種独特の世界観がただよっていたこのイベント、「この人達頭おかしいやろ」と恐れおののいていた自分ですが、ヤバさにひきよせられてしまったのか先日第2回目に参加してしまいました。

<a href="https://build.connpass.com/event/120830/" target="_blank">Container Build Meetup #2 - connpass</a>

2回めはなぜか神楽坂のど真ん中、閑静な住宅街の古民家での開催となったのですが、そのマニアックさは前回に負けず劣らずヤバイものでした。

マニアックな話題にウキウキする面々を目の当たりにしながら、<strong>「あぁ、これがその筋の人たちか…」</strong>とある種の諦観の境地に達した様子をお届けいたします。

くコ:彡


## 神楽坂の古民家が会場

<a href="https://www.spacemarket.com/spaces/koori/rooms/wahoeMuDezWEuOre" target="_blank">【古民家Kori/神楽坂駅1分】ライブイベントやパーティーに最適な和洋室！～20名規模の会議や研修/セミナー/落語会/子連れＯＫ | スペースマーケット</a>

会場は、あまりエンジニアの勉強会で使われることはないだろう神楽坂の古民家。主催のtoriさんよりツイートで雑に案内があります。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">入り口こんなところです <a href="https://twitter.com/hashtag/container_build?src=hash&amp;ref_src=twsrc%5Etfw">#container_build</a> <a href="https://t.co/pgNdKhbzup">pic.twitter.com/pgNdKhbzup</a></p>&mdash; ポジティブな Tori (@toricls) <a href="https://twitter.com/toricls/status/1127875106405937152?ref_src=twsrc%5Etfw">May 13, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">入ったら靴を靴棚に置いて、スリッパを履いて奥の部屋までまっーすぐお越しください <a href="https://twitter.com/hashtag/container_build?src=hash&amp;ref_src=twsrc%5Etfw">#container_build</a> <a href="https://t.co/DhxyoOGqHJ">pic.twitter.com/DhxyoOGqHJ</a></p>&mdash; ポジティブな Tori (@toricls) <a href="https://twitter.com/toricls/status/1127876834551422976?ref_src=twsrc%5Etfw">May 13, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">古民家だ、すごい <a href="https://twitter.com/hashtag/container_build?src=hash&amp;ref_src=twsrc%5Etfw">#container_build</a> <a href="https://t.co/fA5hcA3Gbv">pic.twitter.com/fA5hcA3Gbv</a></p>&mdash; y-ohgi (@_y_ohgi) <a href="https://twitter.com/_y_ohgi/status/1127881508755132416?ref_src=twsrc%5Etfw">May 13, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

この会場、すごくないすか？

## 当日イベントのハッシュタグ

当日のハッシュタグはこちらにまとめられています。発表内容への参加者のツッコミや、補完情報もつぶやかれているすげぇ充実したまとめなので、是非こちらもご覧あれ。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">変態ミートアップまとめました / Container Build Meetup #2 - Togetter <a href="https://t.co/6N5rm3xV2L">https://t.co/6N5rm3xV2L</a> <a href="https://twitter.com/hashtag/container_build?src=hash&amp;ref_src=twsrc%5Etfw">#container_build</a></p>&mdash; stormcat24 (@stormcat24) <a href="https://twitter.com/stormcat24/status/1128123808395579392?ref_src=twsrc%5Etfw">May 14, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>



## イベントアジェンダ

|  **時間** | **タイトル** | **スピーカー** |
| :--- | :--- | :--- |
|  19:30 - 19:45 | 開場 & 誰かの大事なﾄﾞｯｶｰｲﾒｰｼﾞかもって思ったら、私が嫌いだからって捨てることはできないよ | @toricls |
|  19:45 - 20:15 | 環境の一致について考えてみる | @Spring_MT |
|  20:15 - 20:45 | Container Build; Kaniko and Friends | @sakajunquality |
|  20:45 - 20:50 | 5-min break | --- |
|  20:50 - 20:55 | Dockerfile AST | @po3rin |
|  20:55 - 21:00 | 役に立たなそうなLT2 dockerignoreの話 | @orisano |
|  21:00 - 21:05 | そうでもないLT 「軽く」コンテナで色々試す | @kaedemalu |
|  21:05 - 21:35 | Japan Container Daysで作った公式デモアプリのDockerfileについて（仮） | @_inductor_ |


## 「Dockerfile書くときによく聞くBGMと好きなワンライナーの発表」

何故かイベント参加フォームにこのアンケートがあり、それの結果発表が主催の<a href="https://twitter.com/toricls" target="_blank">ポジティブな Tori（@toricls）</a>さんよりありました。

- Dockerfile書くときに好きなBGMは？
- Docker関連で好きなワンライナーは？

「なんなんだ、この時間は…」と思いながら聞いていると、それはそれはマニアックなこだわりがたくさん。BGMは全体的にアニメやヘビメタが多かった印象。好きなワンライナーは、イメージ一発お掃除系（サブコマンドでイメージ洗い上げるやつとpruneつかうやつ）が主流でした。

<strong>「時代はやっぱりpruneだよね。(　･ิω･ิ)」</strong>

と謎の盛り上がりをみせ、会場の雰囲気も最高潮！


## 「環境の一致について考えてみる」

<script async class="speakerdeck-embed" data-id="f24506d56a424ddf819e71bf02999855" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

登壇者は、DeNAの春山誠さん（<a href="https://twitter.com/Spring_MT?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor" target="_blank">@Spring_MT）</a>。


REREPという新規サービスを開発している中ででてきた、「複数環境においてイメージを使い回すために、その環境祭をどこで吸収するのか？」という点を、真正面から取り扱ったセッションでした。

Dockerを使う上で、原理原則として意識しておくのは「同じイメージを極力複数環境で使い回すこと」。そうすることで、開発環境〜検証環境〜本番環境とシームレスに動作確認からリリースができるわけですが、じゃぁ環境依存する設定をどこにもつかは永遠の課題です。

Deep Environment Parityの名のもとに「コード」と「実行環境」と「データ」にわけて、それぞれの取り扱い方をものすごい実践的に解説されていたのが印象的でした。トピックとしては以下の内容です。

- 実行経路の同一化
- 設定ファイルの切り替え
- 時刻処理
- 設定ファイルと環境変数の使い分け
- 設定ファイルの管理
- ビルドにおけるDockerイメージの使い回し方法
- 環境間差分を吸収するkustomizeの使い方
- 各種データ（設定、マスター、ユーザー）の保存場所の扱い

内容が濃く刺激が強かったのか、QAも飛び交ってました。恐らく、世の中にあるコンテナワークロードにおいては、アプリケーション種別によって設定ファイルの管理方法のバリエーションは物凄く多いと思いますが、コンテナ運用している現場に参考にできる内容が多いと思います。
the Twelve-Factor App</a>




## 「Container Build; Kaniko and Friends」

<script async class="speakerdeck-embed" data-id="fd86f612be91420f970e59ef660b9f60" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>
登壇者は、Jun Skataさん<a href="https://twitter.com/sakajunquality?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor" target="_blank">（@sakajunquality）</a>。

一貫して、コンテナビルドについてのお話。基本はGoogleが開発したOSSであるKanikoとBuildKitのガチンコ対決でした。

030

<p style="background-color: #e8e8e8;padding: 12px;">
<a href="https://github.com/GoogleContainerTools/kaniko" target="_blank">GoogleContainerTools/kaniko: Build Container Images In Kubernetes</a>
</p>

自分は、Kanikoの詳細をちゃんと聞いたのは初めてだったので凄い楽しかったです。

- Docker Daemon不要
- Imageとして動作
- レイヤーを各RUNコマンドでキャッシュ

現時点でGoogle Cloud BuildがBuildKitに対応していなかったんですね。<a href="https://dev.classmethod.jp/cloud/aws/codebuild-buildkit/" target="_blank">AWS CodeBuildは対応</a>してますよ（宣伝）。

そんなGoogle Cloud BuildでもKanikoはサポートしているらしいです。逆にAWS CodeBuildでKanikoも動くは動くらしいです。だれか変態な人はチャレンジしてみてはいかがでしょうか。

また、BuildKit以外のビルドフレンズも紹介されています。

セッション中、何故かやたらBuildKit派をageたりsageたり、会場からもツッコミや合いの手がはいる、なんか殺伐としているのかワイワイしているのかよくわかんない不思議な時間でした。

最後、Dockerメンテなの須田さんから、こういうものがあることも共有されてます！

<a href="https://cloud.google.com/tekton/" target="_blank">Tekton  |  Google Cloud</a>

## 「Dockerfile AST」

<script async class="speakerdeck-embed" data-id="61f7228076844509b0f1cf0b78b1945b" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>

登壇者は<a href="https://twitter.com/po3rin" target="_blank">@po3rin</a>さん。

Dockerfileの抽象構文木（abstract syntax tree）の話です。もう、なんなんすかね。自分AWS環境におけるコンテナ環境インフラ屋としてコンテナと関わることが多いので、話はきいていたけど深く見たことはなかった分野の話だったので、すげぇ面白かったです。

040

そもそも、「抽象構文木ってなんやねん」って方は、同じく@po3rinさんが書いたこちらの記事が参考になります。

<a href="https://qiita.com/po3rin/items/a3934f47b5e390acfdfd" target="_blank">Goを読んでDockerの抽象構文木の構造をサクッと理解する - Qiita</a>


## 「dockerignoreの話」

<script async class="speakerdeck-embed" data-id="a68779cad7a84ba899f14ef23176ae5d" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>

登壇者は、<a href="https://twitter.com/orisano" target="_blank">おりさの（@orisano）</a>さん。

界隈では、このアカウントIDを見るだけで「また、やばい人がキタ」というオーラを漂わせているおりさのさんですが、非常にわかりやすく、ためになる内容でした。

docker buildがやっていることの解説から始まって、不要なファイルを含めたくない理由、含めてはだめなファイルの例、そこからつながるdockerignoreの話という流れだったので、コンテキストが整理されていて、凄くはらに落ちました。

この勉強会で一番気持ち悪かった（褒め言葉）瞬間がこちら。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">気持ち悪いｗｗｗ <a href="https://twitter.com/hashtag/container_build?src=hash&amp;ref_src=twsrc%5Etfw">#container_build</a> <a href="https://t.co/vKb8QXOQ1k">pic.twitter.com/vKb8QXOQ1k</a></p>&mdash; stormcat24 (@stormcat24) <a href="https://twitter.com/stormcat24/status/1127908062721798144?ref_src=twsrc%5Etfw">May 13, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

これの気持ち悪さと有用さはすべて登壇資料で解説されているので、気になる人はスライド見てみましょう。

## 「軽くいろいろ試す」

（登壇資料は近日掲載？）

登壇者は<a href="https://twitter.com/kaedemalu" target="_blank">Taisei Ito（@kaedemalu）</a>さん。

docker buildの基礎からいろいろ試された話でした。alpineのつらみをみんなで想像しあいながら「ﾜｶﾘﾐﾌｶｲ」と妙な連帯感を持ったのが今日のハイライト。


## 「ベストプラクティスに沿って、Dockerfileの可読性とキャッシュ、イメージのサイズの戦略について考えてみた」

<script async class="speakerdeck-embed" data-slide="4" data-id="46e7825077e04de2a15cb08ea4b81bdd" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

登壇者は、<a href="https://twitter.com/_inductor_" target="_blank">@_inductor_</a>さん。

ミートアップの趣旨に反して一番役に立つ話をしてしまったので、一部会場からブーイングが起こるという謎の展開。

Docker buildのお悩みポイント別に、それぞれコンパクトに知見を紹介していく様は、まさに王道そのもの。

- イメージサイズ
	- ベースイメージ毎の得手不得手紹介
	- multi-stage buildを使おう！
	- dockerignoreを使ったり、ソースコードを排して無駄を省略
- ビルドの速さ
	- BuildKitを使うがいいさ！
- 可読性とメンテナンス性
	- ええ感じでかけ！
- 再現性
	- ええ感じでかけ！
- セキュリティ
	- 機密情報はちゃんと<code>--mount=type=secret</code>を使おうね

途中、会の趣旨をきちんと振り返ってます。素晴らしい姿勢ですね。



「Dockerfileの最適化だけを追い求めていくと、可読性が下がったりキャッシュが効きづらくなったり運用がつらくなったりするので、きちんと用法用量を守りましょう！」

というわけで、非常に参考になってしまったセッションでした。改めてDockerfileの痛みや全体を俯瞰しての話だったので、トリを飾るのにふさわしい収まり具合だった思います。

## Docker buildに的を絞った濃密で独特な２時間

<strong>古民家での開催という非日常感と、参加者の前のめりに楽しんでやろう！という空気感がないまぜになった貴重な時間でした。</strong>

前回も思ったんですが、テーマをギュッと絞り込むと、参加者の熱もあがってQAも活発になって楽しいですね。この雰囲気大好きだし、ブログ書きながら講演内容振り返ることでめっちゃ勉強になります。

はてさて、第３回目が開催されるかどうかはわかりませんが、とりあえず気になった方は、commpassのページがありますのでメンバー登録しておきましょう！普段味わえない体験ができると思います。自分も都合がつけば、必ず参加してみようかと。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://build.connpass.com/" target="_blank">Container Build Meetup - connpass</a>
</p>

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。
