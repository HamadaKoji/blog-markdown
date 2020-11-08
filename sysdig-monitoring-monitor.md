# EKS環境をsysdigでモニタリングしてみた（モニタリング編）

最近、巷で噂のsysdigをいろいろ触ってみてます。機能はシンプルでいながらダッシュボードやパネルの種類やメトリクスの種類が非常に豊富で、切れ味鋭いモニタリングツールという感触です。

まだまだ深堀りには足らないのですが、ドキュメントをみながらざっくりした機能をためしてみたのでその様子をお届けします。

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     Sysdig ﾏﾂﾘ ﾏﾀﾞﾏﾀﾞﾜｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>

## この記事の前提条件

この記事では、すでにEKS環境におけるSysdigのセットアップが完了している前提で書いています。この記事単体でも、Sysdig Monitorの概要を知ることはできますが、自分で手を動かしたほうが必ず面白いので、ぜひ、こちらの記事を参考にEKS環境でセットアップを完了しておいてください。

<a href="https://dev.classmethod.jp/cloud/aws/sysdig-monitoring-setup/" target="_blank">EKS環境をsysdigでモニタリングしてみた（セットアップ編） ｜ Developers.IO</a>

## Sysdig Monitor概説

公式マニュアルの入り口はこちら

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://docs.sysdig.com/en/sysdig-monitor.html" target="_blank">Sysdig Monitor</a>
</p>

<blockquote>Sysdig Monitorは、Sysdigプラットフォームの一部で、コンテナやマイクロサービスに最適化されたアーキテクチャで、セキュリティや監視やフォレンジックを提供します。Sysdig Monitorをつかうことで、動的な分散環境に対するプロセスレベルの詳細な可視性を提供し、監視やトラブルシューティングに役立つ情報を提供します。<br />
引用元：<a href="https://docs.sysdig.com/en/sysdig-monitor.html" target="_blank">Sysdig Monitor</a>
</blockquote>

モニタリングアーキテクチャとしては、下記の図がわかりやすいです。すべてのアプリケーション、コンテナ、ホスト、ネットワークシステムコールを把握することを可能にしています。

010
（引用元：<a href="https://www.scsk.jp/sp/sysdig/sysdig_presentation.html" target="_blank">Sysdigの概要 | Sysdig - コンテナ・Kubernetes環境向けセキュリティ・モニタリング プラットフォーム</a>）

## Sysdig Monitorの始め方

Sysdig Monitor（https://app.sysdigcloud.com/）にログインすると、初期画面が表示されます。

020

画面の各TOPメニューの概要。

|  **Module** | **Description** |
| --- | --- |
|  **Explore** | メインメニュー。インフラ全体の深いレベルでのメトリクスを収集し表示し、トラブルシュートに役立てます |
|  **Dashboards** | いわゆるひとつのダッシュボード機能。収集した各種メトリクスを、システム要件に沿った形でカスタマイズして整理し表示することができます |
|  **Alerts** | モニターした各種メトリクスに対してアラートを設定し、一覧する画面。システム全体のアラート状況を一括して確認するには便利 |
|  **Events** | アラートによってトリガーされたイベント一覧 |
|  **Captures** | Sysdig Monitorによってキャプチャされたファイルの一覧 |  

また、Sysdig Spotlightを使うと、特定領域ミドルウェアのサマリーを自動的に抽出できます。左下にあるメニューの[Sysdig Spotlight]をクリック。

030

すると、[Sysdig Monitor Spotlight]画面が表示されます。

040

すでにインテグレートされているものもありますが、[Manage Your Integrations]をクリックすると、追加できるインテグレーションが複数表示されるので、アプリ特性に合わせて選択すると、それようのビューを取得することができます。

050

## Explore（インフラ全体のメトリクスの表示）

060

Exploreはメトリクス収集のメイン画面です。かなりいろんな角度から分析、収集ができるっぽいのですが、機能がかなり豊富なため最初はちょっととっつきにくいかもしれません。順を追って説明していきます。

### Data Sourceの選択

左上のここをクリックすると、Exploreに表示するデータの抽出元を選択できます。抽出元の分類として最初に大雑把に指定できるのは楽ですね。ここには、AWSインテグレーションを実施していない場合AWS表示はないです。

070

### Groupings

Grupingsを指定することでタグの階層構造を規定します。ここで、インフラ構造を可視化する際のビューを整理可能。

080

おそらく、上の説明ではなんだかよくわかんないと思いますが、ここの詳細は下記マニュアルを参照してみてください。

<a href="https://docs.sysdig.com/en/grouping,-scoping,-and-segmenting-metrics.html" target="_blank">Grouping, Scoping, and Segmenting Metrics</a>

直感としては、最初の階層構造のセットが左側のExploreビューに反映されているという理解でOKです。例えば、上部に<code>kubernetes.cluster.name</code>→<code>kubernetes.namespace.name</code>→<code>kubernetes.deployment.name</code>→<code>kubernetes.pood.name</code>→<code>container.id</code>と設定されていれば、その階層構造が左側のExploreビューに表示されています。

090

で、この階層構造のデフォルトが先述したGroupingsに設定されているわけです。

### Table Search

現在表示されている表の一覧を部分一致でフィルタしたい場合は、検索窓に文字列を設定します。

110

### Table Columnes

表のヘッダー部分へのマウスオーバーで、各カラムの数値による背景色の色付け、カラムの表示非表示、表示順などを詳細に接k亭可能です。

120

詳細や列の追加は、歯車アイコンクリックで表示される、[Configure Table Columnes]でも設定可能です。

130

140

### Drill-Down Menu

Sysdig Monitorは大量にあるダッシュボードやメトリクスから利用できるビューをドリルダウン形式で選択することが可能です。以下の手順を試してみてください。

Exploreから対象の階層を選択し、ドリルダウンを開いてピン留めします。

150

そうすると、ドリルダウンメニューがこのようにExploreの横に固定されるので、各種ビューを比較するのに非常に便利です。

160

もとに戻すときは、ドリルダウンメニュー下方の[Unpin Menu]をクリックします。

また、この状態で右上のドット3つアイコンをクリックするとダッシュボードのコピーや細かいレイアウトの修正が可能です。

170

ダッシュボードではなく、個別のメトリクスを確認したい場合は、ドリルダウンメニューの中から[Metrics]を展開し、該当するメトリクスを選択。タイムナビゲーションの設定や、メトリクス表示内容の設定、追加オプションの表示が可能です。


180


190

## Dashborads（いわゆるひとつのダッシュボード）

ダッシュボードは、Sysdig Monitorで表示した各種メトリクスやグラフを一つにまとめて、システム仕様に最適化されたダッシュボードを表示する機能です。こちらも操作が最初難しいと思うので、段階的にその絞り込み方法を紹介します。

左側のModule一覧から[DASHBOARDS]をクリックし、プリセットされた任意のダッシュボードを開きます。個々では[Network Overview]を開いてみます。

200

そうすると、Network関連の各種メトリクスが表示されたダッシュボードが表示されます。これは、第一レベルのドロップダウンメニューです。この上部でダッシュボードに表示するスコープを編集できます。

210

下の状態だと、<code>kubernetes.namespace.name</code>の<code>kube-system</code>が部分一致でヒットするメトリクスが表示されます。

220

このようにラベルをセットしておくと、上部のメニューでドロップダウンで、<code>kubernetes.namespace.menu</code>を選択可能です。便利でござんすな。

230

このラベルは複数設定することができるので、ドリルダウンしたい属性をあらかじめ指定しておくことで、分析がはかどります。

240

また、各パネルにマウスオーバーすることで、各パネルの編集が可能です。[In vx Out Network Bytes]をTopology表示して、kubernetes.daemonsetでグルーピングすると、このようなネットワーク・トポロジーマップも表示可能。ここらへん、Web上ですべて完結する柔軟性は素晴らしい！

250

その他詳細なDasyuboardsの設定については、下記マニュアルが参考になります。

<a href="https://docs.sysdig.com/en/configure-dashboards.html" target="_blank">Configure Dashboards</a>

## Alerts（いわゆるひとつのアラート設定）

260

アラートは、ここで設定します。もともとプリセットされたアラートが多数あるのでこれを編集、設定するのもありですが、右上の[Add Alert]ボタンをクリックすると、任意のAlertを追加できます。

270

ここにアラートタイプが一括してまとめられているのがわかりやすい。[Downtime]や[Metric]、[Event]はだいたいそのままの意味ですが、注目ポイントは[Anomaly Detection]ですね。最近、AWSのCloudWatchでも類似の機能が出てきていますが、過去のメトリクスの状態から判断して異常なメトリクスになったときにそれを検知する機能となります。

また、[Group Outlier]は、ホストOSをグループ単位で監視し、一つだけ他のホストとは違った挙動をしたときにそれを検知する機能です。

試しに[Downtime]アラートの設定画面はこのようになっています。

280

上部でアラートを定義してNotify先に各種通知先を設定します。こちらは、事前に[Settings]画面でも設定できますが、アラート先には各種サードパーティ製品も使えます。もちろんSlackやメールもあります。

290

アラートですが、ダッシュボードの各パネルにそのまま[Create Alert]メニューがあるので、こちらを使うのが実際は効率が良いです。こちらも必ず抑えておきましょう。

300

## Captures（条件指定しておいたファイルのキャプチャー）

Capturesでは予めホストに対して設定しておいた、Capture条件に一致するファイルを一覧化できます。このように、Exploreモジュールのホスト関連パネルからSettingsをクリックすると、[Sysdig Capture]のメニューがあるので、これをクリックすることでキャプチャー条件を設定できます。

310

ここで指定した条件に合致したファイルは、事前に[Settings]画面で[Sysdig Storage]として指定したS3バケットに対して自動的に格納されます。ここらへんも、インテグレーションされているのでAWSを普段遣いしている人にとっては非常に便利ですね。


## Sysdig Monitorの一般的な使い所を最初に把握しよう

以上、駆け足でしたが、Sysdig Monitorのモニタリング機能の概略をお伝えしました。通り一遍の紹介でしたが、Sysdig Monitorがどういったものか。どういったメトリクスを収集できるのかなど、雰囲気を掴んで頂いたでしょうか？

Sysdigには、Monitor以外にも、コンテナ環境のセキュリティ対策に役立つSysdig Securityもあります。こちらも面白い機能で、すでに<a href="https://dev.classmethod.jp/author/yoshie-kento/" target="_blank">吉江</a>による検証記事もあります。

<a href="https://dev.classmethod.jp/etc/sysdig_secure_ecr_scan/" target="_blank">Sysdig Secureを使ってECRイメージスキャン試してみた ｜ Developers.IO</a>

SecurityにもCI/CDへのインテグレーションであったり様々な機能がありますので、ぜひそちらも試してみようと思います。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

