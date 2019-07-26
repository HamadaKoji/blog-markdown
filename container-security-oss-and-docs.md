# コンテナセキュリティを始めるのに有用なドキュメント3つと無料ツール5つの紹介

<strong>「コンテナセキュリティってなんか必要そうやねんけど、実際なにすんの？」</strong>

先日、我らがDevelopers.IO Cafeにおいて、<a href="https://www.creationline.com/" target="_blank">クリエーションライン株式会社 (CREATIONLINE, INC.)</a>様と共催で、以下のイベントを開催しました。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://connpass.com/event/136344/" target="_blank">あなたのコンテナ運用大丈夫？コンテナセキュリティの考え方と対応策 - connpass</a>
</p>

全部で3セッションで構成されているのですが、私の方では、「コンテナセキュリティ関連OSSの紹介」と題して、コンテナセキュリティこれから検討始めようという方に向けて、そのとっかかりに有用なドキュメントと無料ツールを紹介させていただきました。

<strong>ドキュメントもツールもどれも有用なものなので、コンテナセキュリティについて不安や必要性を感じている人は、これらの中から実際に読んだり試してみたりしてもらえればと思います。</strong>

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     ｺﾝﾃﾅｾｷｭﾘﾃｨ ﾏﾂﾘﾀﾞﾜｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>


## セッション資料

Speakerdeckに挙げたのはこちら。

<script async class="speakerdeck-embed" data-id="b9fce659fc474fe9aac0e9c583e1e32a" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

この記事は、上記セッション資料を元にブログ記事として最構成したものです。<strong>関連情報の補足、およびリンク先の利用にはスライドよりこっちのほうが確実に便利なので、この記事中心に見てください。</strong>

## このセッションで伝えたいこと

皆さんコンテナを運用する上で、セキュリティ上の不安を感じていることと思います。コンテナセキュリティについて具体的な対策を立てている方はおられますかね。

具体的な解決策としては、本日、先程説明いただいたaqua Container Securityなども有力な選択肢となります。

<a href="https://www.aquasec.com/" target="_blank">Aqua - Container Security, Serverless Security &amp; Cloud Native Security</a>

s10.png

ただ、「実際に試してみるにしても、なんか大変そう…」という気持ちになる人も多いかもですね。そこで、<strong>「まずはOSSで気軽に試してもらいながら、コンテナセキュリティの必要性を肌で感じてもらいたい」</strong>、というのが本セッションの主旨となります。

本日紹介するものの一覧はこちら。

- ドキュメント
  - Google Cloud コンテナセキュリティ
  - NIST Application Container Security Guide
  - CIS Docker Benchmarks
- OSS
  - aqua MicroScanner
  - Clair
  - Trivy
  - Dockle
  - aqua kube-hunter

ここで、突然ドキュメントを紹介している理由がこちら。

S14

一度、コンテナセキュリティについてのドキュメントでコンテナセキュリティの全体像を把握しておくことで、商用製品含めた各製品やツールの位置付けを捉えることができます。

ほな、いってみよ！

## 「Google Cloud コンテナセキュリティ」

010

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://cloud.google.com/containers/security/?hl=ja" target="_blank">Kubernetes Engine のコンテナ セキュリティ  |  Google Cloud</a>
</p>

Google Cloudが提唱するコンテナ環境を保護するための考え方をまとめたドキュメントで、以下の3つの分野で、それぞれどういった対処が必要かが、紹介されています。

- インフラストラクチャのセキュリティ
- ソフトウェアのサプライチェーン
- ラインタイムセキュリティ

コンテナの実行により、根本的に異なるセキュリティモデルが必要になることが解説されています。

020

このドキュメントの素晴らしさは、<strong>コンテナ環境におけるセキュリティの考慮事項が非常にコンパクトにまとめられていること</strong>につきます。10分ぐらいで読めつつ、コンテナセキュリティの概観を即座に掴むことができるので、コンテナセキュリティ気になる方は、まずはこれを読むことをオススメします。


## 「NIST Application Container Security Guide」

030

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://www.nist.gov/publications/application-container-security-guide" target="_blank">Application Container Security Guide | NIST</a>
</p>

National Institute of Standards and Technology（NIST）がまとめた、アプリケーションコンテナテクノロジーのセキュリティ上の問題に関する報告書です。全部で63ページの英語のドキュメント。

セキュリティ上の脅威を回避するためにセキュリティ対策を実施すべき領域として5つの分野を指摘し、それぞれの対応策も提示されています。

- イメージリスク
- レジストリリスク
- オーケストレーターリスク
- コンテナリスク
- ホストOSリスク

<strong>コンテナを扱う上での代表的なセキュリティリスクが、ベンダーニュートラルに幅広く全体網羅されているので、非常に汎用性が高く原理原則を俯瞰するのに最適なドキュメントです。</strong>

040

ボリュームがたっぷりな英語のドキュメントなので正直ちょっとひくけれど、それぞれの項目で記載されている内容は具体的かつ簡潔なので、ある程度コンテナについて経験があるかたはすらすら読んでしまえるんじゃないでしょうか。

<strong>今後、いろんなコンテナセキュリティ製品がでてきたときも、このドキュメントの知識をベースにすることで、その製品のできることできないことや特徴を把握しやすくなると思うので、一読をオススメします。</strong>



## 「CIS Docker Benchmarks」

050

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://www.cisecurity.org/benchmark/docker/" target="_blank">CIS Docker Benchmarks</a>
</p>

Dockerコンテナやそのホスト環境に対して、セキュリティ上注意しておくべき項目を説明したガイドライン。全部で220ページほどの分量で、診断項目の分類は以下の通り。

- Host Configuration
- Docker daemon configuration
- Docker daemon configuration files
- Container Images and Build File
- Container Runtime
- Docker Security Operations
- Docker Swarm Configuration

ドキュメント自体はかなり膨大で、Dockerそのものと、それをホストするホストOS側の設定についても細かく触れられているのが特徴です。

Docker社では、このガイドラインに則りDocker環境を診断するツールとして<a href="https://github.com/docker/docker-bench-security" target="_blank">docker/docker-bench-security</a>を提供しているので、<strong>ドキュメント読むのめんどくさい！というひとは、まずはこちらのツールを試してみるのが早いんじゃないでしょうか。</strong>

以下のページで詳細に解説されているので、合わせて参考にしていただければ。

- <a href="https://kakakakakku.hatenablog.com/entry/2019/01/21/080009" target="_blank">Docker 公式のセキュリティ診断ツール「Docker Bench for Security」を試した - kakakakakku blog</a>


## 「aqua MicroScanner」

100

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://github.com/aquasecurity/microscanner" target="_blank">aquasecurity/microscanner</a>
</p>

aqua社が提供するOSSのコンテナイメージ静的脆弱性診断ツール。利用までのステップは以下の通り。

- 利用に必要なトークンの取得
- Dockerfileに4行追加
  - 脆弱性スキャン用バイナリを取得して中で実行

便利なツールではあるんですが、Dockerfileを書き換える必要があるので、日常的なスキャン設定やCI/CDパイプラインへの組込にはあまり適さないです。

細かい利用方法は、こちらを参照ください。

- <a href="https://dev.classmethod.jp/tool/docker/microscanner/" target="_blank">無料で脆弱性検査！Dockerfileに4行追加で導入できるmicroscannerを試してみた ｜ DevelopersIO</a>

## 「Clair」

110.png

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://github.com/coreos/clair" target="_blank">coreos/clair: Vulnerability Static Analysis for Containers</a>
</p>

CoreOS社開発のコンテナイメージ脆弱性スキャンツール。ビルトインデータソースが豊富。

Debian Security Bug Tracker, Ubuntu CVE Tracker, Red Hat Security Data, Oracle Linux Security Data, Amazon Linux Security Advisories, SUSE OVAL Descriptions, Alpine SecDB, NIST NVD

- https://github.com/coreos/clair/blob/master/Documentation/drivers-and-data-sources.md

GitHubのREADMEも非常に充実していて情報量が多い。実行には、PostgreSQLが必要。導入手順はこちらを参照。

- <a href="https://github.com/coreos/clair/blob/master/Documentation/running-clair.md" target="_blank">clair/running-clair.md at master · coreos/clair</a>

対象の脆弱性スキャン対象データベースも非常に多く機能が充実しているのですが、導入してスキャンするまでのハードルが、他のツールに比べて高く感じました。

### 参考資料

JAWS-UGコンテナ支部で用意されているハンズオンをやるのが、最初のとっかかりには良さそうなので、ぜひ一度こちら参考にしていただくのをオススメします。

- <a href="https://github.com/jawsug-container/scan-image-vulnerabilities" target="_blank">jawsug-container/scan-image-vulnerabilities</a>

また、AWSブログにも、terraformで、スキャン環境一式をterraformで作る記事もありますので、興味ある方はこちらもどうぞ。

- <a href="https://aws.amazon.com/jp/blogs/publicsector/detect-vulnerabilities-in-the-docker-images-in-your-applications/" target="_blank">Detect vulnerabilities in the Docker images in your applications | AWS Government, Education, &amp; Nonprofits Blog</a>

120

## 「Trivy」

130

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://github.com/knqyf263/trivy" target="_blank">knqyf263/trivy: A Simple and Comprehensive Vulnerability Scanner for Containers, Suitable for CI</a>
</p>

作者は<a href="https://twitter.com/knqyf263" target="_blank">スッキリごん！（@knqyf263）さん</a>。シンプルで実行が簡単なコンテナイメージの脆弱性スキャンツール。

インストール、脆弱性スキャン共にものすごい簡単です。例えば、Macの場合、インストールはこちら。

[bash]
$ brew install knqyf263/trivy/trivy
[/bash]

脆弱性スキャンはこちら。

[bash]
$ trivy <your_container_image_uri>
[/bash]

こんだけです。最初の実行時は、実行環境に脆弱性に関わるデータベースを構築するため時間がかかりますが、その後は早い。

README.mdも非常に充実してます。

- https://github.com/knqyf263/trivy

各オプションの解説だけじゃなく、メジャーどころのCI（Travis CI, CircleCI）への組込コード例や、プライベートリポジトリへの認証設定方法、他の脆弱性私見検知ツールとの比較まであるので、コレ眺めているだけでも楽しい。

140.png

<strong>ワンライナーで実行可能、かつ対応する脆弱性データベースが豊富で非常にオススメです。是非一度触ってみてください。</strong>

### 参考資料

試してみたブログはこちら。

- <a href="https://dev.classmethod.jp/etc/trivy_poc/" target="_blank">最近話題のコンテナ脆弱性ツール「Trivy」を試してみた ｜ DevelopersIO</a>

Clairに引き続き、TrivyもJAWS-UGのハンズオンがあります。こっちも要チェックやで！

- <a href="https://github.com/jawsug-container/scan-image-vulnerabilities" target="_blank">jawsug-container/scan-image-vulnerabilities</a>


## Dockle

150

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://github.com/goodwithtech/dockle" target="_blank">goodwithtech/dockle: Container Image Linter for Security, Helping build the Best-Practice Docker Image, Easy to start</a>
</p>

作者は、<a href="https://twitter.com/tomoyamachi" target="_blank">Tomoya AMACHI（@tomoyamachi）さん</a>。こちらもイメージ名指定だけで脆弱性検出が可能なツールです。

インストールと利用方法は、Trivy同様非常にシンプル。Macの場合、インストールはこちら。

[bash]
$ brew install goodwithtech/dockle/dockle
[/bash]

脆弱性検出方法はこちら。めっちゃ簡単。

[bash]
$ dockle [YOUR_IMAGE_NAME]
[/bash]

Trivyと大きく違うところは脆弱性スキャンの対象。Trivyは、主にCVEなどの脆弱性データベースがスキャン対象になっていたのとは違って、DockleはDockerfileのベストプラクティスやCISベンチマークが検出対象になっています。

1. Build <a href="https://docs.docker.com/develop/develop-images/dockerfile_best-practices/" target="_blank">Best practices</a> Docker images
2. Build secure Docker images
   - Checkpoints includes <a href="https://www.cisecurity.org/cis-benchmarks/" target="_blank">CIS Benchmarks</a>

特に、Docker Benchでは対応していないCIS Benchmarkに対応しているのは素晴らしい。

160

また、オリジナルのチェックポイントも充実しています。

170

<strong>Dockleも非常に使いやすく、独自のポイントでイメージのセキュリティ検査やDockerfileのベストプラクティスも実施してくれるため、非常にオススメです。特にキャラが違うTrivyとの併用が良いですね。</strong>

### 参考資料

作者の@tomoyamachiさんが、関連記事を挙げています。

- <a href="https://qiita.com/tomoyamachi/items/bb6ac5788bb734c91282" target="_blank">CIで簡単につかえるコンテナのセキュリティ診断｢Dockle｣ - Qiita</a>
- <a href="https://qiita.com/tomoyamachi/items/2397457f5d516a1fbc85" target="_blank">3分でできる！最高のDockerfileを書いたあとにやるべき1つのこと - Qiita</a>

また、DockerHub上位800イメージに対する、DockleとTrivyの診断結果がまとめられたページもあります。これをみると、それぞれのツールで、どんな脆弱性試験結果が検出できるのか一覧できるため、こちらも一読することをオススメします。

- <a href="https://qiita.com/tomoyamachi/items/e0e69da521505e73237b" target="_blank">DockerHubで公開されているコンテナが安全か確かめてみた結果【人気のコンテナ上位800個】 - Qiita</a>

## aqua kube-hunter

180

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://github.com/aquasecurity/kube-hunter" target="_blank">aquasecurity/kube-hunter: Hunt for security weaknesses in Kubernetes clusters</a>
</p>

aqua社が開発した、kubernetes環境に対する脆弱性試験ツールです。kubernetesクラスターやワーカーノードに対して、ワンライナーで30項目以上の脆弱性試験が可能です。

<strong>これも、実行方法はめちゃくちゃ簡単。自前でKubernetes環境を構築したり、ワーカーノードを作成している場合に、その脆弱性を検査するのに非常に有用です。</strong>

### 参考資料

基本的な使い方とツールの概要は、こちらのブログにまとめているので、合わせて参照ください。

- <a href="https://dev.classmethod.jp/cloud/aws/kube-hunter/" target="_blank">【Kube-hunter】Dockerワンライナーで30項目のkubernetes環境脆弱性テストができるOSSを試してみた ｜ DevelopersIO</a>

## まとめ「まずは、コンテナセキュリティを身近に感じてもらうためのきっかけに」

以上、3種のドキュメントと5種のツールを紹介しました。

コンテナセキュリティって一言でいっても、対策には従来のアプリケーションとは根本的に違う観点が必要です。コンテナのイメージに含まれるパッケージの脆弱性ももちろん脅威の対象になりえますが、それをホストするサーバー自体の管理、イメージに含まれてしまう機密情報、外部ネットワークの適切な遮断と制御、不適切なイメージの混入に対する防御など、実施項目は多岐にわたります。

結局は、それら一つ一つに対して適切な対策をとっていく必要があるんですが、正直全体像がわかりにくくて、なにか始めるにしても最初の一歩が億劫だと思います。

そういうときに、まずは今日紹介したドキュメントを参考にしてもらいつつ、<strong>最初に手元にあるイメージに対して脆弱性検査をし、そのスキャン結果を確認することで、「おうふ、こんなん出てるで。さて、これええのかな？」と必要性を検討するための材料としてみてください。</strong>

なにかえげつないのがでてきたら、それはチャンスです。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。














