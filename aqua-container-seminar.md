010

先日、日比谷公園にほど近い会場で<a href="https://www.creationline.com/" target="_blank">クリエーションライン株式会社</a>様主催のAqua Securityコンテナセミナーに参加してきました。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://www.creationline.com/lab/30335" target="_blank">2019年11月20日 Aqua Securityコンテナセミナー を開催します  - クリエーションライン株式会社</a>
</p>

基本は、弊社でも扱っているAqua Container Securityについての話が主だったのですが、製品仕様以外にも価格コム様でのKubernetes運用のGitOpsの話であったり、決済サービスPaidyにおけるコンテナ運用の話であったり、最近話題のOSSな脆弱性スキャンツールのTrivyの作者、福田さんがイスラエルからオンラインセミナーやったり、コンテナ界隈の幅広い話題が盛りだくさんで、楽しい一日でした。

ここでは、そんな一日の内容を紹介します。

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     ｺﾝﾃﾅｾｷｭﾘﾃｨｰﾏﾂﾘﾀﾞﾜｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>

## セミナーアジェンダ

- Aqua Securityの最新アップデート
- 事例講演：株式会社カカクコム様
- 事例講演：株式会社Paidy様
- 特別講演：Aqua Security社
- その他デモブース、ネットワーキング有り

## Aqua Security最新アップデート（+α）

020

講演者は、クリエーションライン株式会社　マグルーダ健人様

### Aqua Security 概要

今日初っ端の登壇ですが、最初っからAquaは皆さんご存知という前提でお話させていただきます！と思いましたが、それはあまりにもあまりなので、最初にAqua Securityの概要をお伝えさせていただきます。

Aqua Securityはコンテナのライフサイクルを一貫して見ることができるツールとなってます。一般的にBuild→Ship→Runとなる流れですね。

030

ここで一つ、シフトレフトというキーワードが出てくるのですが、これはセキュリティに問題があるシステムは大半が設計構築時に埋め込まれているため、後期工程での対応はリスクが高くなります。そのために、開発のより早い段階でセキュリティ対策を施すことをシフトレフトと表現しています。

040

こちらが、Aqua Securityのコンポーネント構成イメージです。

050

右側にコンテナ実行環境があり、ホストOSには「Aqua Enforcer」、コンテナのサイドカーとして「Micro Enforcer」をエージェントとして設置します。そのエージェントと通信しながら、Aquaサーバーでその状態を検知し、各種情報をDBに蓄積します。最新の脆弱性情報データベースなどはAqua社の管理サーバーより取得します。

大きな特徴は、Aqua Gatewayコンテナ。エージェントの数が増えた場合に、通信を受けるGatewayがボトルネックにならないように複数コンテナでスケールさせるために独立しています。素敵ですよ〜。これ。


### Aqua Security v4.5 アップデート紹介

それでは、v4.5における新機能を紹介していきます。

#### Scanning Improvements

一番おもしろいのは、イメージのレイヤー毎に脆弱性情報を取得できるようになっているところです。こんな感じで、Layesタブをクリックすると、そのイメージで発見された脆弱性がどのレイヤーに存在しているかを可視化できます。

055

CVSSv3に新しく導入された<strong>Critical</strong>にも対応しています。

060

#### Risk Explorer

コンテナワークロードを運用中にリスクを発見したときに、そのコンテナやPodを把握することが容易になり、運用が簡単になります。このようなグラフィカルな表示により、どのNamespaces上のどのPodに脆弱性が潜んでいるか、ひと目で分かるようになっています。

070

#### vShield

仮想的なパッチ。運用中コンテナの脆弱性に即座に対応するのは、一定時間必要になる。それを仮想的にパッチを当てることができるのが、このvShield。面白い機能なので是非お試しあれ。

その他、以下のアップデートがあります。

- VM Security　→　コンテナだけではなく、VMに対するセキュリティ機能も追加
- Serverless Runtime Protection　→　機能追加
- Pivotal Aplication Service　→　追加
- Integrations　→　SIEM/SSOが追加
- サポート範囲がさらに充実　→　PKS/PAS/Cloud Foundry 

### Cleationine 会社紹介

CL LABというエンジニア向けブログやってます。AquaSecurityのアップデート情報などもこちらで確認可能。

- <a href="https://www.creationline.com/lab" target="_blank">CL LAB - クラウド・サーバー・インフラエンジニアのためのトピック集</a>

また、先日開催されたCNDT2019（Cloud Native Days Tokyo）にてLinux Foundationsよりご紹介いただきました。

- KCSP
- KTP
- Member

また、日本で唯一のLinux Foundation認定Kubernetesトレーニングを提供しています。

- <a href="https://www.creationline.com/kubernetes-training" target="_blank">Kubernetesトレーニング - クリエーションライン株式会社</a>


### 濱田感想

最近、詳細なアップデートを追ってなかったのですが、vShieldとか面白そうな機能が満載でごっつ興味を惹かれた内容でした。ものすごい勢いで進化しているAqua container Security。また、機能検証していこうと思います。

- <a href="https://dev.classmethod.jp/referencecat/aqua/" target="_blank">Aqua ｜ 特集カテゴリー ｜ Developers.IO</a>


## gitopsとAqua Security

080

講演者は、株式会社カカクコム　プラットフォーム技術本部システムプラットフォーム部　橋本和樹様

### Agenda

1. Aqua導入の経緯
2. 弊社Kubernetesの構成例
3. GitOpsについて
4. GitOps + Aqua
5. デモ
6. まとめ

### Aqua導入の経緯

090

コンテナセキュリティの課題として、一番良く知られているのが脆弱なイメージのスキャンもあります。最近ここを機能提供しているパブリッククラウドも複数ありますが、この脆弱性スキャンをすり抜けて運用環境にデプロイされるコンテナもゼロとは言えません。Aquaは、この起動中コンテナをセキュリティ担保できる非常に貴重な製品で、これが導入の決め手となりました。

### 弊社Kubernetesの構成例

100

オンプレ上の環境に、Master NodeとWorker Nodeを構築。監視ツールにPrometheus、検索エンジンとしてElasticsearchを利用しています。

### GitOpsについて

110

CI部分に、GitLab CI、CD部分にはGitOpsとして、ArgoCDを採用。マニフェストファイルのコンテナバージョンの書き換えには、sedを利用しています。マニフェストファイルはリポジトリ管理され、それがSSoT（Single Source of Truth）としてArgoCDにより運用環境に反映されます。


### GitOps + Aqua

120

ここにAquaを導入することで、イメージの脆弱性スキャンとK8S Cluster、k8sNodeに対してランタイムスキャンが可能になります。

### まとめ

- 脆弱なイメージのスキャン
  - GitOpsの中にAquaのイメージスキャンを組み込むことで脆弱なイメージの強制的な排除
- 起動中コンテナのセキュリティの担保
  - AquaEnforcerによるポリシーの実施で不正なプロセスの監視・制御

### 濱田感想

k8s環境での実践的な運用とAqua導入の話が聞けて非常に参考になりました。橋本さんとは、講演終わりにいろいろ話が聞けたのですが、環境ごとのマニフェストファイルの差分吸収にはKustomize、またCDツール検討時には、Spinnakerも評価したとのこと。そのうえで、ArgoCDの便利さと簡便さと信頼さに満足されている様子でした。

k8s環境のCDとして、GitOpsの考え方、そしてArgoCDの存在感は日増しに高まっているのを感じました。




## Aqua Security in Slow Motion

130

講演者は、株式会社 PaidyのJoseph Leroy様と、Jeremy Turner様。

### 講演内容

Paidyはオンライン決済サービスPaidy（ペイディー）を運営。複数の投資家からこれまで総額1億6300万米ドル（約177億円）を超える出資を受けている。ただ、そんな多額の出資を受けてサービスを運営していると、いろいろ問題がでてきた。特に決済サービスを運営しているため、サイバー犯罪のターゲットになるかもしれない。

そんな不安から、Aquaを導入した。実際にスキャンすると、心臓発作がでるぐらい脆弱性が発見され、開発者に見せた時非常にびっくりされたのも良い思い出です。

Aqua導入のおかげでDevSecOpsが捗るようになった。開発差と運用者の両方が同じダッシュボードみることでコミュニケーションが活発化した。最初は脆弱性のリストを見たときに「忙しい！」と言っていた開発者も、徐々に協力的になってくれた。

ISO27001などのレギュレーションやPCI-DSSなどもAquaのおかげで対応が楽になった。DockerHubは基本的に使ってないが、ベースイメージとしては使うので、そこに脆弱性があるイメージが紛れ込む可能性はある。ここに対しても定期的、およびCI/CDに組み込んで利用できるAquaは非常に便利と感じている。

Aquaは導入するのに半年ほどの時間がかかった。

### なぜAqua CSPでスローモーションなのか？

システムに対して考えておくべき点は複数ある。

1. セキュリティフレームワーク
2. 資産管理
3. セキュリティー規則

これらが最初に無いのであれば、Aquaセキュリティを導入する前に、このあたりを考慮したほうがよい。

### Aqua CSPを導入するときの3つの課題

1. Aqua CSPのインフラをどうするか？
2. 複数のAWSアカウントをどうやって扱うか？
3. 役割と責任

paydiでは、まだDevOpsの文化が成熟していなかった。管理するインフラは非常に沢山あったため、IaCは必須であった。CloudFormationを利用しているが、Terraformを使うのもよい。

AWSアカウントが20個以上あるような場合、Aqua CSPはどこに保持するべきか？

150

paidyでは、各アカウントをVPC Peeringで接続し、一つのAWSアカウントに接続してている。

160

Paidyは絶賛採用中なので、よろしく！

- <a href="https://careers.paidy.com/" target="_blank">Careers at Paidy</a>

### 濱田感想

なかなか、AWS環境における実際のAqua導入事例を聞くことがなかったので、非常に参考になりました。やはりAquaサーバーを複数たてるのは管理が無駄に複雑になるので、VPC Peeringを利用した構成は非常に明解で頭に入れておきたい構成でしたね。

実際に、Aquaを導入する前後での現場のやりとりが生々しく聞くことができたセッションで、面白かったです。

## Container security with Aauq OSS

200

講演者は、Aqua Open Source Team所属の福田鉄平さん。イスラエルからリモートでの講演となりました。

### 自己紹介

<a href="https://twitter.com/knqyf263" target="_blank">スッキリごん！（@knqyf263）/ Twitter</a>

Aqua社で唯一の日本人。Trivyの開発者。

### イスラエルに来るまでの話

大体の経緯はブログに書いた（はてぶ1900超の伝説級エントリ）。

<a href="http://knqyf263.hatenablog.com/entry/2019/08/20/120713" target="_blank">趣味で作ったソフトウェアが海外企業に買われるまでの話 - knqyf263&apos;s blog</a>

OSSの脆弱性スキャンツールTrivyを開発していた。

<a href="https://github.com/aquasecurity/trivy" target="_blank">aquasecurity/trivy: A Simple and Comprehensive Vulnerability Scanner for Containers, Suitable for CI</a>

- コンテナの脆弱性スキャナTrivyをGW中に開発
- 自分のこだわりを詰め込みまくった
- OSSとしてGitHubにて公開
- 既存の脆弱性スキャナとの比較グラフも作成

すると、ある日突然、Aqua社からメールが来た。Aquaが提供しているOSSのコンテナ脆弱性スキャナ、Microscannerとの比較をのせていたので、「や、やばいことになったのか！！」と怖くなったが、中味は非常によいものだった。 

- Trivyを買収したい
- OSSチームに入ってメンテナンスを続けて欲しい
- クラウドネイティブのセキュリティOSSに関する仕事
- どこで働いても良い（ビザもサポートする）
- 給料も多く出す（ストック・オプションもある）

もともとイスラエルに興味があった。第二のシリコンバレーといわれたり、サイバーセキュリティの中心地だったりする。楽しそうと思って飛び込んだ。

210

今イスラエルに来て3ヶ月。海が家から8分ぐらい。市場もある。環境も楽しい。超楽しい。仕事が楽しい。所属しているオープンソースチームがキャラが濃くて面白い。

220

### Introduction to Aqua Security

Aquaでは、オープンソースツールを複数開発している。

- trivy
  - Image vulnerability scanner
  - <a href="https://github.com/aquasecurity/trivy" target="_blank">https://github.com/aquasecurity/trivy</a>
- kube-bench
  - CIS benchmark for k8s
  - <a href="https://github.com/aquasecurity/kube-bench" target="_blank">https://github.com/aquasecurity/kube-bench</a>
- kube-hunter
  - K8S penetration-testing
  - <a href="https://github.com/aquasecurity/kube-hunter" target="_blank">https://github.com/aquasecurity/kube-hunter</a>

Aqua Securityのコンテナワークロードにおける位置付けとしては、このようになる。

230

### trivyの紹介

機能紹介。いろいろ書いたけれど、<strong>なんせ早くなりました！</strong>

- Detect comprehensive vulnerabilities
- Easy installation
- Simple
- High accuracy
- DevSecOps
- <strong>Fast(v0.2.0)</strong>

v0.2.0以前は、脆弱性情報をGitHubに保存してあり、起動時にGit CloneしてローカルでDB構築していた。これのため初回が重く、キャッシュを基本持たないCIサービスでは非常に時間がかかるものであった。

これが、v0.2.0以降は、BoltDBをzip化してGitHub Releaseにうつし、データサイズを削減。全体で80%の削減に成功。<strong>これにより、初回実行でも10秒以内に完了。AWS系のCIサービスなら1秒を切ることも可能となっている。</strong>

また、CNCF Cloud Native Interactive Landscapeにも無事入りました。

240

### trivy vs clair

300

ここで仁義なき戦いを繰り広げます。




そんな感じです！まじです。

### イスラエルのいいところ

最後に、イスラエルのいいところまとめます。めっちゃいいところです。みんな来たらええのに。

400

### 濱田感想

昔からTrivyとTwitterはよく知っていたのですが、初めて肉声を聞いてざっくばらんに喋っているのを聞くことができてめっちゃ楽しかったです。まぁ全部書くのは憚れるぐらいのぶっちゃけトークでしたが、今のtrivyとAquaとイスラエルのすごい生々しいリアルな現実がよく見えて非常に面白かったです。

個人的には、trivyとclairとの比較とともに、trivy vs aquaコンテナセキュリティの現状と、将来的にAquaにTrivyや福田さんのノウハウがどうやって活かされていくにか、非常に興味を持ちました。質問タイムで聞けばよかったなぁ。

なんせ、trivyがAquaに買収されたのは個人的に非常に衝撃でしたし、今後の両者の進化から目が離せません。

## コンテナセキュリティ周辺の盛り上がりを感じた充実の一日

コンテナを本番運用すること自体は非常に一般的になってきてはいますが、その運用におけるセキュリティ対策については、手法もノウハウも運用もまだまだ過渡期にあるというのが現状だと思います。最近はAWSのコンテナレジストリであるECRで脆弱性スキャン機能が実行されたり、パブリッククラウドの機能として提供されるものも増えてきましたが、ランタイム環境での防御などは、まだまだ対策ができていない現場が多いと思います。

懇親会タイムでは、色んな人とこの界隈のざっくばらんな話が聞けたので非常に楽しかったです。なかなか、こういった属性で開催されるイベントも無いですからね。クリエーションライン様、貴重なお時間をいただきありがとうございました。

クラスメソッドでも、aquaコンテナセキュリティ扱っておりますので、気になる方はお気軽にお問い合わせくださいませ。

<a href="https://classmethod.jp/news/aqua/" target="_blank">コンテナセキュリティソリューション「Aqua」を提供するクリエーションラインとパートナー契約｜プレスリリース｜</a>

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

