**「Grafana Meetupももう3回目なんすね、早いもんだ…」**

感慨にふけりながら、オラクル青山センターに向かうハマコー。前回のタイミー様といい、素敵な会場を提供いただける会社様に感謝しながら、青山のハイソなビルに入っていきました。

今回のMeetupも、各登壇者非常に熱のこもった発表内容が多く、運営でツイートばっかりしてた自分もめっちゃ楽しかったです。資料を公開されているスピーカーの方も多いので、少しでも気になった方は、登壇資料や動画も見ていただけると、Grafana捗るんじゃないでしょうか！

## Grafana Meetup #3 イベント概要

https://grafana-meetup-japan.connpass.com/event/330796/

<blockquote>お待ちかねの第3回は、株式会社AbemaTVの山本氏による**「Grafana エコシステムの活用事例 on ABEMA」**や株式会社ZTVの脇田氏による「地域DXにおけるGrafana活用事例」に始まり、恒例のLT(1本5分 x 4)など他では聞けない**Grafana Meetup Japanならではのコンテンツ**が盛り沢山です。現地参加者には懇親会もありますよ！</blockquote>

| 時間 | タイトル  | 発表者 |
|:-------------- |:-------------- |:------------ |
| 18:30 - 19:00 | 開場・受付開始 | - |
| 19:00 - 19:10 | **オープニング・会場説明** | Grafana Meetup Japan |
| 19:10 - 19:40 | **「Grafana エコシステムの活用事例 on ABEMA」** | 株式会社AbemaTV 山本氏|
| 19:40 - 20:10 | **「地域DXにおけるGrafana活用事例」** | 株式会社ZTV 脇田氏 |
| 20:10 - 20:20 | 休憩 | - |
| 20:20 - 20:25 | **「LT1：GrafanaのHTTP APIを眺めてみよう」** | eForce株式会社 林氏 |
| 20:25 - 20:30 | **「LT2：Amazon Managed Grafana で IoT TwinMaker によるデジタルツインアプリケーションを動かしてみた」** | クラスメソッド株式会社 若槻氏 |
| 20:30 - 20:35 | **「LT3：Grafanaプラグイン探訪」** | サイオステクノロジー株式会社 久保氏 |
| 20:35 - 20:40 | **「LT4：OpenTelemetryのSignalsをGrafana Cloudに送る」** | FRAIM株式会社 Yamaguchi氏 |
| 20:40 - 20:50 | **クロージング・会場スポンサーからの告知**| Grafana Meetup Japan |
| 20:50 - 21:30 | **懇親会** | (現地参加者のみ) |

いつも通り、メインセッション2つとLT4つの構成です。19時開始のイベントにしては欲張り過ぎなのかもしれません。ただ、どのセッションも熱量が高いので、コンテンツ減らしたくないなぁというのも悩みどころだったりします。

会場の様子。メインディスプレイが大きくて視認性抜群。天井からでるサブディスプレイも複数設置されているセミナー専用ルームで、イベント利用としては最高の環境でした。改めてオラクル様に感謝ですね。

005

当日のアーカイブ動画。配信は安定の[EMTEC \- IT勉強会特化の配信チーム](https://emtec.tv/)です。いつも本当にありがとうございます！

https://www.youtube.com/watch?v=u5nFZsZfzlU

実施協力は、こちらの4団体。

010

アンケートやツイートの投稿などを司会から誘導した後に、ビッグニュースが会場でお知らせされました。

## 2025年2月25日に、Grafana Lbas公式イベントが日本で開催されます

冒頭お知らせですが、Grafana Labsが東京で初イベントの開催が決定しました！

020

会場や形式などのアナウンスはこれからなのですが、日本で行われる初のGrafana Labs公式イベントなので、Grafana興味あるよってかたは、是非この日程を空けておいてもらえればと思います！

## 第1セッション「Grafana エコシステムの活用事例 on ABEMA」

030

> ABEMA では可視化基盤として Grafana を利用しています。サービスのモニタリングだけではなく社内のあらゆるデータを Grafana で可視化することで様々な場面で活用しています。サービスのモニタリングだけではなく改善にも活用している Grafana エコシステムの全体像を ABEMA での実績・展望・悩みを含めて紹介させていただきます。

@[speakerdeck](ededb5f9d44e4f9c8e1c4cb58b6992e7)

登壇者は、[tetsuya28（@\_tetsuya28）さん / X](https://x.com/_tetsuya28)。

### セッション感想

ABEMAの膨大なトラフィックを支えるGrafanaエコシステムの活用に関する発表。多数あるGrafana関連ツールをフル活用している、まさにGrafana総合格闘技という内容でした。いろんなユースケースが存在するGrafanaですが、実際にGrafanaで何ができるのかの主流なユースケースを把握するのに非常に役に立つ発表だったと思います。まじでてんこ盛り。

ABEMAで利用しているツールの一覧はこちら。

032

それぞれのツールに対する推しポイントが複数あり、主要な関連ツールの外観を把握できます。個人的には、Grafana Pyroscopeのユースケースででていたプロファイルの可視化のイメージ（UIでの比較）、Grafana Tempo周辺のインフラ構成、Terraform providerあたりの話が興味深かったです。

最後に今後の理想像のところで、オブザーバビリティ4本柱に言及されていたのが印象的でした。

033

Grafanaのエコシステムで、ABEMA TVという巨大プラットフォームのオブザーバビリティを追求していく姿勢は、全Grafanaユーザー参考になると思います。

## 第2セッション「地域DXにおけるGrafana活用事例」

040

> ZTVでは、昨今の異常気象による水害増加を受け、氾濫しやすい河川や用水路、ため池に水位・雨量・冠水センサを設置し、モニタリングサービスを提供しています。加えて、気象計、温湿度CO2、傾斜、積雪、罠などのIoTセンサも活用し、データの可視化範囲を広げています。本セッションでは、地域DXの一環としてIoTセンサと視覚的に分かりやすいダッシュボードの活用事例についてご紹介させていただきます。

@[speakerdeck](effe96fe77744fe68b53e18f959ba07a)

登壇者は、株式会社ZTV / 新事業推進部 次長、脇田（Wacky）氏。

### セッション感想

ZTV社の地域（主に関西〜中部地方）DX/IoTサービスでのGrafana活用事例の紹介。

Grafana導入経緯。このようにパッケージの融通の効かなさとフルスクラッチ開発のコストの丁度よいバランスを求めてGrafanaを活用しようとする場合も多いんじゃないでしょうか。自分自身もこのあたりの検討過程でGrafanaが採用される過程をよく見るので、非常に共感しました。

041

システムイメージ。AWS側はよくある構成かなと思うのですが、通信回線で、[ELTRES](https://eltres-iot.jp/)や[Sigfox](https://www.kccs.co.jp/sigfox/service/)などのIoT用ネットワークは初耳だったのでめっちゃ参考になりました。

042

スマート農業ユースケース。こういった、現場での悩み事をプラットフォームとして提供し課題解決につなげているのはすごく良いなと思いました。

043

前側のABEMAさんのユースケースは完全にシステム管理者向けだったのですが、こちらは直接Grafanaのダッシュボードをエンドユーザーがみるユースケースで、本当にGrafanaのユースケースって広いんだなぁと改めて感じる素敵な事例でした。

## 「LT1：GrafanaのHTTP APIを眺めてみよう」

050

> Grafanaに公開されているAPIの存在を知ってましたか？このAPIはGrafana上で行う操作と同等の機能を扱えます。つまり、外部からGrafanaの操作がある程度できるということです。一緒にAPIを種類を見て、さらなるGrafanaの可能性を見つけていければと思います。

@[speakerdeck](b047662d5b19435ab2b3d0f03b965312)

登壇者は、[林 直哉（@stupid\_owl）](https://x.com/stupid_owl)さん。

### セッション感想

ダッシュボードがメインのユースケースのため、Grafanaの操作ってGUIが中心になりますが、改めてGrafana　HTTP APIについての紹介セッション。ざっと並べただけでもこれぐらいのHTTP APIがあるとのこと。

051

発表中面白かったのは、Annotation APIの内容で、通常GUIでは設定できないパネルやダッシュボードに依存しないAnnotationが追加できる部分。このあたりGUI前提ではないAPIを利用することで、運用上の手間を省略する方法がいろいろあるなと、新しい発見がありました。

HTTP API referenceはこちらから確認できるので、気になる方は、一旦ざっとどんなAPIがあるか眺めてみるだけでもヒントになる部分多いと思います。

https://grafana.com/docs/grafana/latest/developers/http_api/

## 「LT2：Amazon Managed Grafana で IoT TwinMaker によるデジタルツインアプリケーションを動かしてみた」

060

> AWS IoT TwinMaker は、工場や製造ラインなどの現実世界のシステムをデジタルツイン化するアプリケーションを構築することができるマネージドサービスです。今回は IoT TwinMaker を Amazon Managed Grafana で動かす方法をご紹介します。

https://dev.classmethod.jp/articles/grafana-meetup-japan-3-amazon-managed-grafana-aws-iot-twinmaker/

登壇者は、[若槻龍太](https://dev.classmethod.jp/author/wakatsuki-ryuta/)さん。

### セッション感想

デジタルツイン、みなさん聞いたことありますでしょうか？

> インターネットに接続（せつぞく）した機器などを活用して現実（げんじつ）空間の情報（じょうほう）を取得し、サイバー空間内に現実空間の環境（きょう）を再現（さいげん）することを、デジタルツインと呼（よ）びます。<br />
> 引用：[デジタルツインって何？](https://www.soumu.go.jp/hakusho-kids/use/economy/economy_11.html)

AWSにはAWS IoT TwinMakerというデジタルツインを実現するサービスがあるのですが、それをAmazon Managed Grafanaで動かしてみたという内容です。AWSからは、デジタルツインのデモとして、クッキーを作る工場のサンプルコードがGitHubで公開されています。

061

LTでは、これが実際に動くデモとして紹介されていました。実際にデモでみるとビビるんですが、工場設備で発生したアラームの一覧がリスト出力されていて、そのアラームを選択すると、実際に工場のどこで発生したのか、3Dモデルにフォーカスがあたる動きをします。これどうなってるんですかね？Grafanaダッシュボードにこんな機能あったっけ？

062

逆もできて、工場モデルの中をクリックすると、それに紐づくアラームやトレース情報を確認することができます。

専用のプラグイン導入が必須ですが、Grafanaダッシュボードの拡張性の高さ、自由度の高さを垣間見るインパクトのあるデモでした。作り込みの手間はもちろんありますが、うまく構築することで現実世界だけでは実現できない生産ラインの設計、可視化、運用ができそうで、製造ビジネステクノロジー部のハマコーはがっつり興奮しております。

実際に動かしてみよ！って方は、上で紹介した若槻執筆のブログを参考にしてみてください。GitHubリポジトリはこちら。

https://github.com/aws-samples/aws-iot-twinmaker-samples

## 「LT3：Grafanaプラグイン探訪」

070

> Grafanaでは標準で様々なパネルやデータソースプラグインがあります。しかし世の中には個人から企業まで数え切れないほどのコミュニティプラグインがあります。今回はその中からいくつかピックアップして使い方などをご紹介いたします。

登壇者は、[久保 雄平（@kubo\_engineer）](https://x.com/kubo_engineer)さん。

### セッション感想

Grafanaはエコシステムが発達していて、プラグインの種類も膨大なのですが、その中でオススメのプラグインを3つ紹介するセッション。自分が一番気になったのは、このInfinityプラグイン。

071

REST APIで提供されているシステムはそれこそ無数にありますが、そのデータを受け取りダッシュボードに表示するのに便利なプラグイン。こんな感じで、Query部分でJSONの各項目をマッピングしてデータを取得してダッシュボードに表示するのを、全て完結させることができるとのこと。使い方次第でいろいろ応用がききそう。

072

Infinity Pluginはもともとコミュニティプラグインだったのが、現在は正式にGrafana Labsに移管されたとのこと。

https://grafana.com/blog/2024/02/05/infinity-plugin-for-grafana-grafana-labs-will-now-maintain-the-versatile-data-source-plugin/

その他、以下のプラグインも紹介されていました。

- [Dynamic image panel plugin for Grafana \| Grafana Labs](https://grafana.com/grafana/plugins/dalvany-image-panel/)
- [Business Charts plugin for Grafana \| Grafana Labs](https://grafana.com/grafana/plugins/volkovlabs-echarts-panel/)

Grafanaのプラグインって膨大な数リリースされています。うまく使えば、ダッシュボード構築に手間が一気に省略できて捗るものも多い印象。推しプラグインを紹介するだけのLT大会があっても良いのでは？と思うぐらい、そのきっかけをもらえるLTでした。


## 「LT4：OpenTelemetryのSignalsをGrafana Cloudに送る」

080

> アプリケーションからOpenTelemetryを利用して計測したTraces,Metrics,LogsをGrafana Cloudで可視化する方法について。 具体的には、OpenTelemetry Collectorの設定、OpenTelemetryのTraces, Metrics,Logsがそれぞれ、Tempo, Prometheus(Mimir), Lokiのデータにどのように変換されるか。

https://www.docswell.com/s/ymgyt/Z9VGDW-export-opentelemetry-signals-to-grafana-cloud

登壇者は、[Yuta Yamaguchi （@YAmaguchixt）](https://x.com/YAmaguchixt)さん。

### セッション感想

オブザーバビリティの民主化の流れの中でOpenTelemetryも非常に注目を浴びている昨今ですが、そのデータをOpenTlemetry Collectorを利用してGrafanaCloudに送信する方法についてのLT．

https://opentelemetry.io/ja/

OpenTelemetry自体は、ベンダーニュートラルなオープンな実装でCNDFプロジェクトにも採択されており、それを利用したGrafana Cloudへの接続方法について、ハマりポイントを中心に紹介されていました。

Grafana Cloudに接続する際は、以下3種が必要。専用のエンドポイントが用意されているので、あとはCollectorを設定するだけで、データのエクスポートが可能。

- OTLP Endpoint
- Grafana Cloud instancd ID
- API Token

https://grafana.com/oss/opentelemetry/

ただ、logs、metrics、tracesでそれぞれOpenTelemtryのデータ構造がどのようにマッピングされているのかは意識する必要があるとのことでした。

普段は自分、OpenTelemetryを意識したことがなくなんとなく雰囲気でしか理解していなかったのですが、改めて上のGrafanaの記事を参考に、手を動かして理解してみようと思いました。

## クロージングでのお知らせ

クロージングでは、3つお知らせが有りました。

### Oracle Cloud Hangout Cafe

100

通称「OCHa Cafe」ですね。


オラクルさん主催だけれど、テーマはOracleだけではなく非常に幅広く扱っているとのこと。気になる方は、こちらに参加するとよいかと思います！

https://ochacafe.connpass.com/

### 「LLMアプリケーションのオブザーバビリティ」

105

「LLMアプリケーションのオブザーバビリティ」というタイトルで、技術書店17で新刊がでるとのこと！

「俺達のxQL完全ガイド」が非常によい本だったので、こちらも非常に楽しみです。物理本買ってしまおう。

### クラメソふくおか Grafanaハンズオン 〜手を動かして学ぶはじめての可視化とオンコール〜

https://connpass.com/event/332669/

Grafana初心者向けに、ふくおかでGrafanaのハンズオンを実施します。ダッシュボード構築だけではなく、Grafana OnCallを利用したアラート通知も体験できるので、興味あるかたはお気軽に登録お願いします！

## クロージングから懇親会〜そして今後に向けて

今回も懇親会実施しました。参加者の8割以上は参加いただいたんじゃないでしょうか。すこし運営の都合上時間が短くなってしまった点は反省点ですが、Grafana関連の話題でいろいろ盛り上がることができたので、自分も非常に有意義な時間でした。

Grafana Meetup Japanもこれで3回目なのですが、どの回もGrafanaユーザーの声と熱気に溢れていて自分自身運営していてめっちゃ楽しいMeetupです。まだまだネタは尽きない界隈だと思うので、いろんなGrafana事例をこれからも聞いていきたいし、自分もどんどこネタをアウトプットしていきたいと思います。

以下URLより、発表者も募集しているので、我こそは！という方は、お気軽にフォームに記入してもらえればと思います。

https://forms.gle/akZWraLnync5YzTs9

それでは、今日はこのへんで。[濱田孝治（ハマコー）（@hamako9999）](https://twitter.com/hamako9999)でした。

