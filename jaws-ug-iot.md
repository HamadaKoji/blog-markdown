# IoT熱わっしょいなJAWS-UG IoT専門支部「re:Inent振返りとAmazon Monitron」の現地参加レポート　#jawsug_iot #AWSreInvent

<strong>「IoT支部初参加や…　怖い人ばっかりやったらどないしよ…」</strong>

普段、JAWSはコンテナ界隈に生息している[ハマコー](https://twitter.com/hamako9999)ですが、来年からIoTや製造関連の顧客に携わることになったのがきっかけで、JAWSのIoT支部のイベントに現地参加してきました。

[JAWS\-UG IoT専門支部「re:Invent振り返りとAmazon Monitron」 \- connpass](https://jawsug-iot.connpass.com/event/301998/)

冒頭の心配は杞憂に終わり（当たり前だ）、re:Invent2023の振返りと最近のIoT事情をガッツリ聞くことができ、懇親会にも楽しく参加させていただきました。お世話になったということも有り、参加レポートをここに記させていただきます。

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     IoTﾏﾂﾘﾀﾞﾜｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>

これから、どんどん来ますYO


## IoT専門支部の紹介とイベント概要

### IoT専門支部とは

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="JAWS-UG IoT専門支部 - connpass" src="https://hatenablog-parts.com/embed?url=https://jawsug-iot.connpass.com/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

<blockquote>IoTに関連するシステムを構築する上でデバイス〜ビジネス価値を産み出す一連のシステム構築における課題の共有や技術的な学習を、楽しく、なるべく実機を使って理解を深めることを目的としたユーザ会です。<br />
<br />
デバイスで遊ぶだけでなく、ユースケースを念頭に、IoTを支えるAWSのアーキテクチャがどうであるべきかを議論、研究する場とします。</blockquote>

### 当日のイベント概要

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="JAWS-UG IoT専門支部「re:Invent振り返りとAmazon Monitron」 - connpass" src="https://hatenablog-parts.com/embed?url=https://jawsug-iot.connpass.com/event/301998/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

「re:Inventの振り返りをしつつ、大注目のAmazon Monitronについて学びましょう！」

| 時刻 | 内容 | 登壇・対応者 |
| --- | --- | --- |
| -19:00:00 | オンライン配信開始 |  |
| 19:00-19:05 | ご案内、説明、会場諸注意など | JAWS UG IoT 専門支部運営 |
| 19:05- | 第一部 re:Invent振り返り |  |
| 19:05-19:35 | AWS IoT 系サービス注目Updateまとめ | AWSJ 服部さん |
| 19:35-19:45 | 産業IoTで欠かせない防爆仕様とAWSアップデート | IoT専門支部 青木さん |
| 19:45-19:55 | AWSの生成AIの取り組みとIoTとの付き合い方 | ソラコム Maxさん |
| 19:55-20:05 | 休憩 ＆ バッファ |  |
| -20:55:00 | 第二部 Amazon Monirton |  |
| 20:05-20:25 | Amazon Monitronの紹介と活用 | AWSJ 吉川さん |
| 20:25-20:55 | AWS re:Inventで見聞きした Amazon Monitron と通信の課題 | ソラコム Maxさん |
| 20:55-21:00 | クロージング | JAWS UG IoT専門支部運営 |

目黒のAWSさんオフィス現地会場とオンライン配信のハイブリッド開催。自分も何度か経験あるんですが、オンライン配信と現地開催のハイブリッドって考慮事項多くてかなり大変なんですよね。今回は、JAWS配信部の皆さんのご協力の下、配信もスムーズに実施いただきました。感謝！

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">本日も配信部の安定した運用で、オンライン配信出来ました。ありがとうございました！ <a href="https://twitter.com/hashtag/jawsug_iot?src=hash&amp;ref_src=twsrc%5Etfw">#jawsug_iot</a> <a href="https://twitter.com/hashtag/jawsug?src=hash&amp;ref_src=twsrc%5Etfw">#jawsug</a></p>&mdash; Jun Ichikawa (@sparkgene) <a href="https://twitter.com/sparkgene/status/1735629464602894707?ref_src=twsrc%5Etfw">December 15, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

冒頭、IoT専門支部運営の[Hide（@Hide\_IoT）さん](https://twitter.com/Hide_IoT)より、本日のイベントの趣旨と支部の紹介をもらったのち、イベントが始まりました。自分は初参加ということも有り、ハッシュタグを埋めることに専念！？してました。

### 当日のアーカイブ動画

（アーカイブがまとまり次第、ここに記載させていただきます）


## 「　re:Inent 2023 IoT系サービスUpdateまとめ」

<script defer class="speakerdeck-embed" data-id="8587f8f0828144a5846f8d8316deffec" data-ratio="1.7777777777777777" src="//speakerdeck.com/assets/embed.js"></script>

最初にAWSJ 服部さんより、re:Inbent 2023で発表されたIoT系サービスのまとめがありました。ちなみに服部さん、この超絶おもしろブログの作者でもあり、濃い専門知識を持っているのが伺えます。

[カオスエンジニアリングをゲーム形式で体感できるデバイスを開発してみた \- builders\.flash☆ \- 変化を求めるデベロッパーを応援するウェブマガジン \| AWS](https://aws.amazon.com/jp/builders-flash/202310/chaos-kitty/?awsf.filter-name=*all)

昔の自分のつぶやき。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">めっちゃ面白い！今度コレ、家でもやってみよ。 / 1件のコメント <a href="https://t.co/kziVqAEEid">https://t.co/kziVqAEEid</a> “カオスエンジニアリングをゲーム形式で体感できるデバイスを開発してみた - builders.flash☆ - 変化を求めるデベロッパーを応援するウェブマガジン | AWS” (1 user) <a href="https://t.co/Y8H6A5ZBzJ">https://t.co/Y8H6A5ZBzJ</a></p>&mdash; 濱田孝治（ハマコー） (@hamako9999) <a href="https://twitter.com/hamako9999/status/1709376768438935765?ref_src=twsrc%5Etfw">October 4, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

最初にAWS提供のIoT向けサービス一覧。自分があんまりIoTに馴染みがないので、こうやって一覧にしてもらえるとありがたい。

010

このあと、それぞれの分野に分類してアップデートを話していただきました。各アップデートへのリンクも添えて。


### ① Analytics

#### AWS IoT SiteWise

AWS IoT SiteWise のUpdateがもりだくさん。今回のre:Inentでは、このあたりのアプデがものすごく充実していたと感じました。今の旬ってことなんでしょうね。多方面においてIoT SiteWiseの機能を拡充する

- [AWS IoT SiteWise、バッファリングされ、バッチ処理された測定データの取り込みをサポート](https://aws.amazon.com/jp/about-aws/whats-new/2023/11/aws-iot-sitewise-buffered-batched-measurement-data/)
- [AWS IoT SiteWise、産業データ向けの新しいストレージ階層を発表](https://aws.amazon.com/jp/about-aws/whats-new/2023/11/aws-iot-sitewise-storage-tier-industrial-data/)
- [AWS IoT SiteWise が Amazon Lookout for Equipment による多変量異常検出を新たにサポート](https://aws.amazon.com/jp/about-aws/whats-new/2023/11/aws-iot-sitewise-multi-variate-anomaly-detection-amazon-lookout-equipment/)
- [AWS IoT SiteWise がメタデータおよびテレメトリデータ取り出し用のクエリ API をリリース](https://aws.amazon.com/jp/about-aws/whats-new/2023/11/aws-iot-sitewise-query-api-metadata-telemetry-retrieval/)
- [AWS IoT SiteWise、メタデータの一括インポート、エクスポート、更新をサポート](https://aws.amazon.com/jp/about-aws/whats-new/2023/11/aws-iot-sitewise-support-bulk-import-export-update-metadata/)
- [AWS IoT SiteWise がアセットモデルコンポーネントを新たにサポート](https://aws.amazon.com/jp/about-aws/whats-new/2023/11/aws-iot-sitewise-supports-asset-model-components/)
- [AWS IoT SiteWise、ユーザー定義の一意の識別子をサポート](https://aws.amazon.com/jp/about-aws/whats-new/2023/11/aws-iot-sitewise-user-defined-unique-identifiers/)
- [EasyEdge による AWS IoT SiteWise Edge のプロトコルのサポートを拡張](https://aws.amazon.com/jp/about-aws/whats-new/2023/11/aws-iot-sitewise-edge-easy-edge-protocol-support/)
- [Siemens Industrial Edge Marketplace で AWS IoT SiteWise Edge が利用可能に \(プレビュー\)](https://aws.amazon.com/jp/about-aws/whats-new/2023/11/aws-iot-sitewise-edge-siemens-industrial-edge-marketplace-preview/)
- [AWS announces new no\-code dashboard application to visualize IoT data](https://aws.amazon.com/jp/about-aws/whats-new/2023/12/aws-no-code-dashboard-visualize-iot-data/)


#### AWS IoT TwinMaker

AWS IoT SiteWiseほどではないですが、Twinmakerの複数のUpdateが来てました。

TwinMakerってなんやねん？って方は、こちらのIoT専門支部の過去アーカイブをみれば良さそうですよ。

<iframe width="560" height="315" src="https://www.youtube.com/embed/pYjJIBCl_EA?si=vn6VEnAhdhQDhccN" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

- [AWS IoT TwinMaker、デジタルツインのエンティティモデリングエクスペリエンスを向上させる新しい機能をリリース](https://aws.amazon.com/jp/about-aws/whats-new/2023/11/aws-iot-twinmaker-features-improve-digital-twin-entity-modeling-experience/)

あんまり馴染みがなかったんですが、Amazon Kinesis Video Streamsって、IoT系サービスなんですね。

- [Amazon Kinesis Video Streams WebRTC Ingestion の一般提供を開始](https://aws.amazon.com/jp/about-aws/whats-new/2023/11/amazon-kinesis-video-streams-webrtc-ingestion/)

### ② Device

#### FreeRTOS

FreeRTOSについては、ロードマップが[FreeRTOS](https://freertos.org/)とGitHubで公開されました。

- [What is FreeRTOS? \- FreeRTOS](https://docs.aws.amazon.com/freertos/latest/userguide/what-is-freertos.html)
- [FreeRTOS のロードマップとコード投稿プロセスが freertos\.org で公開されました](https://aws.amazon.com/jp/about-aws/whats-new/2023/11/freertos-roadmap-code-contribution-process-published-freertos-org/)

ロードマップの公開とともに、機能追加のIssueによりリクエストや、ロードマップを踏まえた顧客とのPJ計画の立案が可能になっています。合わせてコントリビューションプロセスも明確化されているので、よりオープンな運用への道筋ができたのは大きな変更点ではないでしょうか（参考：[Contributions \- FreeRTOS](https://freertos.org/community/contributions.html)）。

#### AWS IoT ExpressLink

<iframe width="560" height="315" src="https://www.youtube.com/embed/tM9LC5gvuVE?si=ABqjEQJi_bTVqvDi" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

こちら、自分がまったく知らない内容でした。確かに、クラウド接続処理におけるセキュリティ考慮が必要になる場面多いのはわかります。

020

- [AWS IoT ExpressLink が BLE 接続をサポートした Technical Specification v1\.2 をリリース](https://aws.amazon.com/jp/about-aws/whats-new/2023/10/aws-iot-expresslink-technical-specification-v1-2-ble-connectivity/)


#### AWS IoT Greengrass

AWS IoT Greengrassが閉域のみで運用可能になりました。インターフェース型のVPCエンドポイントを作成することで、エッジ側からAWSへの閉域網接続が可能になっています。手順の詳細は日本語版には記載がないため、英語版のドキュメントを参照とのこと。

- [AWS IoT Greengrass and interface VPC endpoints \(AWS PrivateLink\) \- AWS IoT Greengrass](https://docs.aws.amazon.com/greengrass/v2/developerguide/vpc-interface-endpoints.html)

AWS IoT Greengrassは、3rdパーティーによって作成されたコンポーネントのカタログ提供が開始されています。

- [Publisher\-supported components \- AWS IoT Greengrass](https://docs.aws.amazon.com/ja_jp/greengrass/v2/developerguide/publisher-supported-components.html)

継続して、Coreのアップデートがされています。

- [Release: AWS IoT Greengrass Core v2\.12\.0 software update on November 7, 2023 \- AWS IoT Greengrass](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-release-2023-11-07.html)

### ③ Connectivity

#### AWS IoT Core

AWS IoT Coreにも複数アップデートが来ています。

- [Authorizing direct calls to AWS services using AWS IoT Core credential provider \- AWS IoT Core](https://docs.aws.amazon.com/iot/latest/developerguide/authorizing-direct-aws.html)
- [AWS IoT Core now supports private certificate authorities with fleet provisioning \| The Internet of Things on AWS – Official Blog](https://aws.amazon.com/jp/blogs/iot/aws-iot-core-private-certificate-authorities-with-fleet-provisioning/)

#### AWS IoT Fleetwise

AWS IoT FleetWise、みなさんご存知でしょうか？コネクテッドカーのために専用で用意されているサービスです。こちらにも複数アップデートが来ています。

- [aws\-iot\-fleetwise\-edge/docs/dev\-guide/vision\-system\-data/vision\-system\-data\-demo\.ipynb at main · aws/aws\-iot\-fleetwise\-edge · GitHub](https://github.com/aws/aws-iot-fleetwise-edge/blob/main/docs/dev-guide/vision-system-data/vision-system-data-demo.ipynb)
- [Processing and visualizing vehicle data \- AWS IoT FleetWise](https://docs.aws.amazon.com/iot-fleetwise/latest/developerguide/process-visualize-data.html)

### One more thing

最後に、「あらよ、もういっちょぅ！」という感じでいきなりアナウンスされたのが、このAWS IoT Sensors。

030.png

- [GitHub \- aws\-samples/aws\-iot\-sensors](https://github.com/aws-samples/aws-iot-sensors)
- [AWS IoT Sensors](https://apps.apple.com/jp/app/aws-iot-sensors/id6447531633)

AWS IoT系のサービスとスマホを使ったセンサーデータの収集、処理、可視化ができるとのこと。なんとも意外なのは、AWS アカウントが不要ということ。デバイスのデータをWebで確認したいときは、アプリ側でダッシュボードのURLを作成し、そこにアクセスするだけという簡単仕様。

早速、弊社の[よな](https://dev.classmethod.jp/author/yona/)が、やってみたブログをあげているのでこちらも合わせて参考にしてみてください！

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="AWS IoT Sensorsを触ってみた | DevelopersIO" src="https://hatenablog-parts.com/embed?url=https://dev.classmethod.jp/articles/aws-iot-sensors/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

### てんこ盛りのアップデート

普段自分が馴染みのないIoT界隈だったので全ての内容が新鮮でした。全般的なアプデの量や充実度で見た場合に、IoT SiteWiseのアップデートが明らかに多かったのが印象的でしたね。今までは、AWS IoT Coreあたりしか自分しらなかったのですが、SiteWiseもこれから追っていこうと思います。

突如アナウンスがあった、AWS IoT Sensorsも面白い。めちゃくちゃ簡単に、AWS IoT Coreのダッシュボードを体験できるので、IoT Coreの一番最初のとっかかりに紹介してみるのも有りなんじゃないでしょうか。

## 「産業IoTで欠かせない防爆仕様とAWSアップデート」

040.png

お次は、IoT専門支部運営の青木　秀康さん（[@Hide\_IoT](https://twitter.com/Hide_IoT)）から、防爆仕様とそれに関連するAWSアップデートの話。

- 防爆とは
- 防爆仕様の基礎知識
- 防爆仕様と法規制
- Amazon MonitronのEx-rated sensors発売

「防爆」という言葉自体全く初めて聞く言葉だったので、全てが新鮮でした。そもそもなんぞやという話から、基礎知識と法規制と絡めた後、ついに「我らが」Amazon Monitronに、関連する機能が備わったとのことです。

- [Amazon Monitron、危険な場所向けの Ex 定格センサーを発売](https://aws.amazon.com/jp/about-aws/whats-new/2023/11/amazon-monitron-ex-rated-sensors/)

<blockquote>お客様は、これらの Ex 定格のセンサーを設置することで、Monitron を使用して危険な場所 (ガス、クラス 1 / ゾーン 2) にある資産を監視できるようになりました。</blockquote>

- クラス1 → 可燃ガス
- ゾーン2 → 異常時に可燃性物質が滞留する可能性がある危険区域

ということなので、石油プラントや化学工場といった場所への投入が可能になり、さらにユースケースが広がったとのことでした。

このあたり、もっと細かい内容は、弊社[市田善久](https://dev.classmethod.jp/author/ichida-yoshihisa/)のブログも参考になります。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="【アップデート】Amazon Monitron で防爆対応センサー（TE1A195）の発売が発表されました | DevelopersIO" src="https://hatenablog-parts.com/embed?url=https://dev.classmethod.jp/articles/amazon-monitron-ex-rated-sensors/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

## 「AWSの生成AIへの取組とIoTとの付き合い方」

<script defer class="speakerdeck-embed" data-id="3cee4bd9b7da4eb695b3598fe51050c8" data-ratio="1.7777777777777777" src="//speakerdeck.com/assets/embed.js"></script>

お次は、ソラコム　[Max\(松下 Kohei\)（@ma2shita）さん](https://twitter.com/ma2shita)より、IoTと生成AIの話。MaxさんのIoT観点でのre:Invent2023総括ブログはこちら。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="現場ニーズに応えた、正統な進化 ― IoTの視点から振り返る AWS re:Invent 2023 - SORACOM公式ブログ" src="https://hatenablog-parts.com/embed?url=https://blog.soracom.com/ja-jp/2023/12/15/report-of-aws-reinvent-2023/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

今年のre:Inventは、猫も杓子も生成AIがベースになっていた印象でしたが、その生成AIを再発明のための手段として位置づけているのがキーポイントと話されていました。

話されている内容が一貫してエバンジェリスト目線で、メタ的にキーノートの意図やメッセージを解釈しようとされているのがめっちゃ新鮮で面白かったです。

更に面白かったのがここの部分。

050

実際にIoTを扱う我々が、生成AIをどう活用していくかの話なのですが、re:InventのExpoで展示されていたAWS IoT TwinMakerでの故障箇所の提示について、Amazon Qを組み込んだ事例が紹介されていました。

060

改めて、今の自分の業務や製品にどうやって生成AIを組み込んでいくのかという部分で、これからの想像力を掻き立てる刺激的なセッションでした。


## 「Amazon Monitronの紹介と活用」

<script defer class="speakerdeck-embed" data-id="22c28432ebd84a59bd9006b5e56df2d0" data-ratio="1.7777777777777777" src="//speakerdeck.com/assets/embed.js"></script>

AWSJ吉川さん[（Kohei Yoshikawa（@yoskoh8））](https://twitter.com/yoskoh8)の、Amazon Monitronの紹介セッション。あらためて、Amazon Monitronの基本知識と動作原理、最近のUpdateや、リファレンスとなるアーキテクチャまで紹介されていて、これからAmazonMonitron始める人や理解してみよって方には、めっちゃコンパクトに全ての情報がまとまった内容になってました。個人的にめっちゃ大助かりでしたよ！

- [Amazon Monitron Part 1（基本編）【AWS Black Belt】 \- YouTube](https://www.youtube.com/watch?v=w_fF73NXu2s)
- [Amazon Monitron Part 2（設定編）【AWS Black Belt】 \- YouTube](https://www.youtube.com/watch?v=7wzCNP8durg)

### AWSの他のサービスとのデータ連携

Amazon Monitronを他サービスと連携させるにあたり、基礎の部分の解説。常にRAWデータが取得されるわけではないことには注意が必要ですね。

070

Amazon Kinesis Data Streamとのデータ連携

080

こちらのページでは、Amazon Kinesisを利用した予防保守のための産業データレイクの詳細も紹介されています。

[Industrial Data Lake for Predictive Maintenance using Amazon Monitron and Amazon Kinesis](https://docs.aws.amazon.com/architecture-diagrams/latest/industrial-data-lake-for-predictive-maintenance-using-amazon-monitron-and-amazon-kinesis/industrial-data-lake-for-predictive-maintenance-using-amazon-monitron-and-amazon-kinesis.html)

090

さらに、ごっつ面白いかったのが、このMonitron Digital Twin Workshop。

[Digital Twin with Monitron and TwinMaker](https://catalog.us-east-1.prod.workshops.aws/workshops/a6d1b5c0-90c6-47a3-bfa7-ed816c6f60ac/en-US)

100

Monitronが取得した情報をGrafanaとAWS IoT TwinMakerで可視化するワークショップになっていて、実際に現地では画面を動かして、一つのダッシュボード上で各種センサー情報と異常状態の3Dモデルが表示されていました。<strong>特筆すべきなのは、なかなか個人では手を出しにくいMonitronデバイスの実機がなくても、このWorkshopは試すことができるとのこと。</strong>Amazon Monitron、まずは試してみよう！ってかたにうってつけのWorkshopかと思います。

勝手な主観ですが、Amazon Monitronは、今のAWSのIoT関連で一番脂が乗っているサービスだと思っていたので、その内容を一気通貫して知ることができる内容になっていて、参考になりました。最後に紹介したWorkshopも実際に動かして試してみようと思います。

## 「AWS re:Inventで見聞きしたAmazon Monitronと通信の課題」

<script defer class="speakerdeck-embed" data-id="7be2e4a75b754aacbbe93c031617aaeb" data-ratio="1.7777777777777777" src="//speakerdeck.com/assets/embed.js"></script>

最後は、ダブルヘッダーとして、本日2回目の登壇になったソラコムMAXさんの、Amazon Monitron利用時における通信の課題について。

AWS Monitronの導入時に以下の2点が障害になることがよくあるとのこと。

- 閉域ネットワークが必要
- AWS Iam Identity Center(IIC)の運用ノウハウがない

この点について、Maxさんがre:InventでAI/ML(AWS)のグローバルBDリーダーである[Dianne Eldridge](https://www.linkedin.com/in/dianneeldridge/)さんに話を聞いたところ、以下の返答があったとのことでした。

- 閉域ネットワークが必要
  - USでも要件はあるが、AWS Direct ConnectやAWS VPNを利用
- AWS Iam Identity Center(IIC)の運用ノウハウがない
  - ここの課題は聞いたことがない

IICに関するところは、USでは、あまり問題にならないとのことでした。ただ、IICをAWSアカウントのガバナンスに普段利用していない顧客もいるなかで、どうするかの案としてでてきたのがこちら。

110

こんな感じで、Amazon Monitronを利用するアカウントと、データ活用したいアカウントを分けて、IICを使うアカウントを極小化するというアイディアです。こうすることで、既存の運用を大きく変えずに、Amazon Monitron使う部分だけを、IICの管理下に置くことができます。

さらに、既存ネットワーク環境に影響を与えないポン付け（Maxさん談）ソリューションが、SORACOMさんから提供されています。

[SORACOM セルラーパック for Amazon Monitron \- IoTデバイス通販 \- SORACOM \(ソラコム\) IoTストア](https://soracom.jp/store/26949/)

既存のネットワーク環境に影響させずに即現場導入できるこのソリューション、良いですね。また、実際のAmazon Monitronの通信料とそれにかかるSORACOM利用料まで例示されていて、至れり尽くせりの内容になってました。

このあたり、AWSのサービスだけでは足りない部分をばっちり埋めに行くサービスの作り方、真似てみようと真剣に思いました！

## 懇親会まで続くあっついあっついIoT魂

懇親会にも参加したのですが、現地会場にいた人の大半は懇親会にも参加してましたｗ　普段はコンテナ界隈に生息している自分ですが、IoT専門支部にもいろんな方がおられて、普段聞けない界隈やAWSのIoT関連サービスに対するあれやこれやの視点をがっつり頂いたので、すっごい新鮮で楽しかったです。やっぱり技術の話を肴にして呑む酒はうまいわ、まじで。

次回は参加するだけではなくて登壇する側にもはいって、これからも継続的にIoT関連の情報発信していきたいと思います。

それでは、今日はこのへんで。[濱田孝治（ハマコー）（@hamako9999）](https://twitter.com/hamako9999)でした。




