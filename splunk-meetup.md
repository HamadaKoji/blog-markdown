# SplunkミートアップでAWS連携部分の基礎について喋ってきた #GOJAS_JP

先日、FargateのログドライバーにSplunkが対応し、「えらいこっちゃやで！」とブログ書きました。

 -<a href="https://dev.classmethod.jp/cloud/aws/fargate-splunk-logdriver/" target="_blank">【祝】FargateでログドライバーにSplunkが利用可能になりました！ ｜ DevelopersIO</a>

 気づいたら、その1ヶ月後、Splunkミートアップで喋っている自分がいました。

<strong> Splunkド初心者が、いろいろ苦労しながらAWS連携部分の基礎をまとめたので、普段AWS触っていてSplunkにも興味があるかたは、是非見ていただければと思います。</strong>

 <pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     Splunk ﾏﾂﾘﾀﾞﾜｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>

## 超綺麗で豪華なミートアップ会場の様子

イベントページはこちら。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://gojas.doorkeeper.jp/events/92430" target="_blank">【GOJAS Meetup-11】 GOJAS Meetup Summer Special - Go Japan Splunk User Group | Doorkeeper</a>
</p>

会場は、<a href="https://www.segasammy.co.jp/japanese/newoffice/" target="_blank">セガサミーホールディングス・TUNNEL TOKYO </a>。ものすごい広くて綺麗で、ディスプレイがめちゃくちゃでかくて驚きました。

会場全体、超広々としており。

S01


何故か、ワープする入り口があったり。

S02

発表時の様子。このディスプレイの大きさですよ。

S04

というわけで、張り切って登壇させていただきました。

## 登壇資料

<script async class="speakerdeck-embed" data-id="75f7724eee87417a8331f1e2e9790746" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

## 発表内容

### 登壇のきっかけ

AWS FargateがSplunkに対応したというニュースがきっかけです。Splunkをそれまで全く触ったことがなかったので、いろいろ調べながら書いてみたブログがこの２つ。

- <a href="https://dev.classmethod.jp/etc/about-splunk/" target="_blank">統合ログ分析ソリューションSplunkの凄さを公式チュートリアルで体感する ｜ DevelopersIO</a>
- <a href="https://dev.classmethod.jp/cloud/aws/splunk-app-for-aws/" target="_blank">SplunkのAWS統合サービス「Splunk App for AWS」を試してみた ｜ DevelopersIO</a>

そうすると、全く面識がなかった山田さんから、こんなメッセージが飛んできました。

slide12.png

この男気！というわけで、今自分はここにいるわけです。宜しくおねがいします。

今日のAgendaです。

- AWS環境へのSplunk構築方法
- SplunkのAWS連携サービスの概説
- AWSデータ連携パターンとその解説

### AWS環境へのSplunk構築方法

AWS環境へSplunk構築する方法は、大きく分けて3つです。

1. EC2を構築し、自分でインストールする
2. AWS Maketplaceから、Splunk Enterpriseを起動する
3. AWS クイックスタートから、Splunk Enterpriseを起動する

#### 「1. EC2を構築し、自分でインストールする」

普通に、EC2インスタンスを立ててインストールする方法です。そんなに詰まる要素は無いと思います。

- <a href="https://docs.splunk.com/Documentation/Splunk/7.3.0/Installation/Chooseyourplatform" target="_blank">Installation instructions - Splunk Documentation</a>

#### 「2. AWS Maketplaceから、Splunk Enterpriseを起動する」

Marketplaceから起動する方法です。

- <a href="https://aws.amazon.com/marketplace/pp/B00PUXWXNE" target="_blank">AWS Marketplace: Splunk Enterprise</a>

予めSplunk Enterpriseインストール済みのAMIが用意されているので、セットアップがめっちゃ楽です。とりあえずHA構成など不要で1台で良いのであれば、有力な選択肢ですね。

#### 「3. AWS クイックスタートから、Splunk Enterpriseを起動する」

AWS クイックスタートもあります。

- <a href="https://aws.amazon.com/jp/quickstart/architecture/splunk-enterprise/" target="_blank">AWS での Splunk Enterprise - クイックスタート</a>

CloudFormationを駆使してパラメータを指定して、AWSのベストプラクティスに基づいたHA構成を実現できます。こんな雰囲気です。すばらしい。

s24

ガイドも30ページぐらいあって非常に充実しているので、本格的なプロダクション環境で利用するための構成もすぐに作ることができるので、一度試してみることをおすすめします。


### SplunkのAWS連携サービスの概説

Splunkに対して、AWS側からデータ連携する方法は複数あります。

- <a href="https://splunkbase.splunk.com/app/1876/" target="_blank">Splunk Add-on for Amazon Web Services</a>
  - 様々な種類のAWSからのデータインターフェースを提供する
- <a href="https://splunkbase.splunk.com/app/3719/" target="_blank">Splunk Add-on for Amazon Kinesis Firehose</a>
  - Firehose経由でSplunkにデータを取り込む
- <a href="https://splunkbase.splunk.com/app/3790/" target="_blank">Amazon GuardDuty Add-on for Splunk</a>
  - GuardDutyのデータをインプット
  - イベント検知後、アラームを起こすなど
- <a href="https://splunkbase.splunk.com/app/1274/" target="_blank">Splunk App for AWS</a>
  - AWSから取得したログのダッシュボードを提供

で、本日メインでお話するのは、「Splunk Add-on for Amazon Web Services」です。


### AWSデータ連携パターンとその解説

Source Typeは複数あり、連携設定パターンも複数あります。

- CloudWatch
- CloudWatch Logs
- Generic S3 inputs
- Incremental S3 inputs
- SQS-based S3 inputs


#### CloudWatch

S36

CloudWatch Metricsの内容をSplunkに転送します。転送するメトリクスのフィルタ条件やディメンションも設定可能。

#### CloudWatch Logs

S38

CloudWatch Logsの内容をSplunkに転送します。設定は転送対象のLogGroupを指定するだけで非常に簡単。ただし、SourceTypeが全てCloudWatchLogsになるので、アプリケーションログの種類ごとに区別が必要であれば、利用には注意が必要。

#### Generic S3 inputs

S41

バケット内の全てのオブジェクトを都度リストし各ファイルの変更日を調査し、未収集のデータを取得。バケット内のオブジェクトの数が多い場合、めちゃくちゃ遅くなる。


#### Incremental S3 inputs

S44

ファイル名に含まれる日時情報をチェックポイントレコードと比較して、バケットから取得されていないオブジェクトのみを取得。Generic S3 inputsに比べてだいぶ高速

#### SQS-based S3 inputs（推奨）

S47

現在の推奨方式。ほぼリアルタイムで、S3に新しく作成されたレコードをSQSに送信し、Splunk側から取得。

SNSのサブスクリプションはクロスアカウント指定できるため、マルチアカウント（開発〜検証〜本番）の各ログを統一ログ分析基盤アカウントに一括集約することも可能。

S48

SNS→SQSのクロスアカウント接続や、SNSとSQSの連携に関わる情報は、以下のページを参照。

- <a href="https://dev.classmethod.jp/cloud/aws/cross-account-sns-sqs-integration/" target="_blank">AWSアカウントでSNSとSQSを連携させるには ｜ DevelopersIO</a>
- <a href="https://dev.classmethod.jp/cloud/aws/sns-topic-should-be-placed-behind-sqs-queue/" target="_blank">【AWS】SQSキューの前には難しいこと考えずにSNSトピックを挟むと良いよ、という話 ｜ DevelopersIO</a>

また、大規模構成では、以下のようにCloudWatch Alarmに<code>ApproximateNumberOfMessagesVisible</code>のしきい値を設定したAuto Scalingでheavy fowaderのインスタンスを指定する方法もあり。

S52

### まとめ

- AWS環境へのSplunk構築は、Market Placeかクイックスタートを使うのが早い
- AWSの各種データからSplunkへデータ転送する方法は複数ある
  - 各サービスネイティブなものはそのまま転送
  - S3格納の時系列データは、Incremental S3 inputs、または、SQS-based S3 inputsから選択

## つぶやき（twitter）の数々

当日のハッシュタグ、<a href="https://twitter.com/search?q=%23GOJAS_JP&src=recent_search_click&f=live" target="_blank">#GOJAS_JP</a>で、流れていたツイートをいくつか紹介します。ツイ廃としては、イベントのハッシュタグが盛り上がっているのを見るのは、すごい楽しい。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">かっちょいー会場ができました！<a href="https://twitter.com/hashtag/GOJAS_JP?src=hash&amp;ref_src=twsrc%5Etfw">#GOJAS_JP</a> <a href="https://t.co/yC6lKKZiis">pic.twitter.com/yC6lKKZiis</a></p>&mdash; GOJAS (@GOJAS_JP) <a href="https://twitter.com/GOJAS_JP/status/1141989828906508288?ref_src=twsrc%5Etfw">June 21, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">株式会社サイバーエージェント リードエンジニア 牧垣様による「ハイブリッドクラウドに対応したログ集約・解析プラットフォームのためのHEC活用法」セッション始まりました‼︎ <a href="https://twitter.com/hashtag/GOJAS_JP?src=hash&amp;ref_src=twsrc%5Etfw">#GOJAS_JP</a> <a href="https://twitter.com/hashtag/splunk?src=hash&amp;ref_src=twsrc%5Etfw">#splunk</a> <a href="https://t.co/cLeWR4Z7zo">pic.twitter.com/cLeWR4Z7zo</a></p>&mdash; きつねうどん (@Ki2neudon) <a href="https://twitter.com/Ki2neudon/status/1142003052628471809?ref_src=twsrc%5Etfw">June 21, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">クラスメソッド濱田様のセッションが始まりました！<a href="https://twitter.com/hashtag/splunk?src=hash&amp;ref_src=twsrc%5Etfw">#splunk</a> <a href="https://twitter.com/hashtag/GOJAS_JP?src=hash&amp;ref_src=twsrc%5Etfw">#GOJAS_JP</a> <a href="https://t.co/6KyWJiN3aI">pic.twitter.com/6KyWJiN3aI</a></p>&mdash; GOJAS (@GOJAS_JP) <a href="https://twitter.com/GOJAS_JP/status/1142009769168789504?ref_src=twsrc%5Etfw">June 21, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">クラスメソッド株式会社 ソリューションアーキテクト 濱田様による「AWSのい〜〜〜ろんなログをSplunkで一括分析しよう！」が始まりました！ <a href="https://twitter.com/hashtag/GOJAS_JP?src=hash&amp;ref_src=twsrc%5Etfw">#GOJAS_JP</a> <a href="https://twitter.com/hashtag/splunk?src=hash&amp;ref_src=twsrc%5Etfw">#splunk</a> <a href="https://t.co/uz1SeGzoik">pic.twitter.com/uz1SeGzoik</a></p>&mdash; きつねうどん (@Ki2neudon) <a href="https://twitter.com/Ki2neudon/status/1142009840681680896?ref_src=twsrc%5Etfw">June 21, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">オフィス強すぎるw<a href="https://twitter.com/hashtag/gojas?src=hash&amp;ref_src=twsrc%5Etfw">#gojas</a> <a href="https://twitter.com/hashtag/GOJAS_JP?src=hash&amp;ref_src=twsrc%5Etfw">#GOJAS_JP</a> <a href="https://t.co/4FvOJqPDa4">pic.twitter.com/4FvOJqPDa4</a></p>&mdash; マシュー(Matthew) (@matthew_sec) <a href="https://twitter.com/matthew_sec/status/1142014393011470336?ref_src=twsrc%5Etfw">June 21, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">やりたい放題Ninjaの時間がやってまいりました<a href="https://twitter.com/hashtag/splunk?src=hash&amp;ref_src=twsrc%5Etfw">#splunk</a> <a href="https://twitter.com/hashtag/GOJAS_JP?src=hash&amp;ref_src=twsrc%5Etfw">#GOJAS_JP</a> <a href="https://t.co/dh4ebXPhnx">pic.twitter.com/dh4ebXPhnx</a></p>&mdash; GOJAS (@GOJAS_JP) <a href="https://twitter.com/GOJAS_JP/status/1142024650152538112?ref_src=twsrc%5Etfw">June 21, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">セガサミーホールディングス株式会社 佐藤様、株式会社ブロードバンドセキュリティ 宮城様による「UF を一括バージョンアップしてみた（仮）」が始まりました！こちらは残念ながら写真撮影、内容のSNS公開が感じなのです。内容凄い私よりなのに悔しい！ <a href="https://twitter.com/hashtag/GOJAS_JP?src=hash&amp;ref_src=twsrc%5Etfw">#GOJAS_JP</a> <a href="https://twitter.com/hashtag/splunk?src=hash&amp;ref_src=twsrc%5Etfw">#splunk</a></p>&mdash; きつねうどん (@Ki2neudon) <a href="https://twitter.com/Ki2neudon/status/1142018018597171200?ref_src=twsrc%5Etfw">June 21, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">「来たぜ Splunk 7.3！！！ どんなモンなのか教えておくんなまし、旦那！！！」の発表の様子。<br><br>さすが小松原さん、発表前から役作りしてた。意識高い。<br><br>（´-`）.｡oO (マスクごしで聞き取りづらかったけどw<a href="https://twitter.com/hashtag/GOJAS_JP?src=hash&amp;ref_src=twsrc%5Etfw">#GOJAS_JP</a> <a href="https://t.co/Zwi8EKZcEQ">pic.twitter.com/Zwi8EKZcEQ</a></p>&mdash; かつさんど (@Ca23d) <a href="https://twitter.com/Ca23d/status/1142042139817201664?ref_src=twsrc%5Etfw">June 21, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">「来たぜ Splunk 7.3！！！ どんなモンなのか教えておくんなまし、旦那！！！」の発表の様子。<br><br>さすが小松原さん、発表前から役作りしてた。意識高い。<br><br>（´-`）.｡oO (マスクごしで聞き取りづらかったけどw<a href="https://twitter.com/hashtag/GOJAS_JP?src=hash&amp;ref_src=twsrc%5Etfw">#GOJAS_JP</a> <a href="https://t.co/Zwi8EKZcEQ">pic.twitter.com/Zwi8EKZcEQ</a></p>&mdash; かつさんど (@Ca23d) <a href="https://twitter.com/Ca23d/status/1142042139817201664?ref_src=twsrc%5Etfw">June 21, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">今日のGOJAS 今まで参加してた時よりすごい収穫あったなー。やはり積極的にコミュ取るの大事だし、スペシャリストに敵う技術を身につけないとな。改めて色々情報交換するの大事だと思った。感&amp;謝。技術ネタ提供できるよう頑張ろ！ <a href="https://twitter.com/hashtag/GOJAS_JP?src=hash&amp;ref_src=twsrc%5Etfw">#GOJAS_JP</a></p>&mdash; sprunner (@sprunner55) <a href="https://twitter.com/sprunner55/status/1142089574203580416?ref_src=twsrc%5Etfw">June 21, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


## まとめ「あったけぇ、来て良かった」

一言、そんな感想です。

<strong>セッションでも話しましたが、最初山田さんから登壇依頼があったときは正直ひよりました。「まじか、俺、なんかしゃべることあるのかな…」と</strong>。AWSのことなら、なんぼでもしゃべることはあるんですが、Splunkはド素人でしたからね。

率直に言って、Splunkの設定に関する情報、特にAWS連健周辺については、現状非常に情報が少なく感じます。そんな中、自分が試している途中で感じた詰まった点や苦労した点を初心者目線で解説して共有できれば、来てくれた人の役に立つのかなと思って、登壇を引き受けさせてもらいました。

会場がめちゃくちゃ豪華で喋っててごっつ楽しかったですね。また、ヤフー株式会社　山田さんをはじめ、Splunkの中の人やコミュニティの方々といろいろ話ができたのも良かったです。お誘いありがとうございました。

懇親会の食事も豪華だった…

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">期待してた！！！1kgチーズピザ！！！しかも3枚！！ <a href="https://twitter.com/hashtag/GOJAS_JP?src=hash&amp;ref_src=twsrc%5Etfw">#GOJAS_JP</a> <a href="https://twitter.com/hashtag/splunk?src=hash&amp;ref_src=twsrc%5Etfw">#splunk</a> <a href="https://t.co/SEIuQKRMFb">pic.twitter.com/SEIuQKRMFb</a></p>&mdash; きつねうどん (@Ki2neudon) <a href="https://twitter.com/Ki2neudon/status/1142030476917395456?ref_src=twsrc%5Etfw">June 21, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">今日も豪華なご飯ありがとうございます！ <a href="https://twitter.com/hashtag/GOJAS_JP?src=hash&amp;ref_src=twsrc%5Etfw">#GOJAS_JP</a> <a href="https://t.co/JLdSdfrKvu">pic.twitter.com/JLdSdfrKvu</a></p>&mdash; msnr (@MASA89434701) <a href="https://twitter.com/MASA89434701/status/1142030790164791297?ref_src=twsrc%5Etfw">June 21, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

まだまだ自分にはノウハウ不足していますが、AWS環境におけるログ分析のバリエーションとして、Splunkの今後の進化を追いつつけてみようと思います。

Splunkに興味を持たれた方は、<a href="https://gojas.doorkeeper.jp/" target="_blank">Go Japan Splunk User Group</a>に参加されてみてはいかがでしょうか。今後も継続的にイベントが開催されるようです。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

