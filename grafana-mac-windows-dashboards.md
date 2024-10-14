# Grafana Integrationsを使って10分でMacOSのダッシュボードを作成し、Prometheusを学ぶ

「Grafanaのダッシュボードあれこれ触ってみたいけれど、実際やるなら自分の身近にある機器のメトリクスを使ってやってみたいよなぁ」

Grafanaには、学習目的で[TestData data source](https://grafana.com/docs/grafana/latest/datasources/testdata/)が公式から提供されています。ダッシュボードの表示形式を、あれこれ編集しながら学ぶには良いデータソースなのですが、いかんせんテスト用のデータなので、実際のユースケースからは遠かったり、クエリがシナリオとして隠蔽されているため、Grafanaの基本であるPrometheusやLokiを学ぶには不向きです。

どうしたもんかと考えていたところ、だれの環境にもあるクライアントPC（Mac, Windows, Linux）をデータソースとして使うのが、実際のデータに触れるには手っ取り早いんじゃないか？ということで、Grafana CloudのIntegrationsの機能を利用して試してみました。

設定自体もそれほど手間がかからず、ダッシュボードもさくっと構築できて、PrometheusやLokiのクエリの書き方、アラートルールなどを学ぶことができるオススメデータソースだったので、この記事で紹介したいと思います。

是非、この記事をきっかけに、Grafanaを利用したPrometheusのメトリクス取得の世界に足を踏み入れてください！

## （前提）この記事の動作検証環境

今回のこの記事は、Grafana Labs社提供のGrafana Cloudの`Grafana v11.2.0-74515 (b4126d3bce)`で動作確認しています。無料でさくっとアカウントは作れるので、まだアカウント無いよって人は、さくっと作ってしまうのが良いかと思います。

:::message
OSS版では、今回紹介するIntegrqationsのメニューが無いので気をつけてください。
:::

## Grafana Integrationsについて理解する

早速設定していきますが、実際の設定の前に、Grafana integrationsについて知っておきましょう。

https://grafana.com/docs/grafana-cloud/monitor-infrastructure/integrations/

> Integrations bundle Grafana Alloy and Grafana Agent configuration snippets, tailored Grafana dashboards, and alerting defaults for common observability targets like Linux hosts, Kubernetes clusters, and Nginx servers. With Grafana integrations, you can get a pre-configured Prometheus and Grafana-based observability stack up and running in minutes.

上記の日本語訳。

> Integationsは、Grafana AlloyとGrafana Agentの設定スニペット、カスタマイズされたGrafanaダッシュボード、およびLinuxホスト、Kubernetesクラスタ、Nginxサーバのような一般的な観測可能性ターゲットのためのアラートデフォルトをバンドルします。 Grafanaインテグレーションを使えば、設定済みのPrometheusとGrafanaベースのobservabilityスタックを数分で立ち上げて実行することができます。

雑に言ってしまえば、「データソースとダッシュボードとアラートをいい感じで一つにまとめたバンドルのセット」です。Grafanaは多数のデータソースに対応しているのは皆さんご存知のところだと思いますが、実際の可視化までそれなりにやることがあります。代表的な内容で一括設定してくれるのが、Grafana Integrationsというわけです。

- データソース設定
- データソースを元にしてクエリを書いてダッシュボードを設定
- ダッシュボードのメトリクスを元にアラートを設定

IntegrationsはGrafanaのあらゆるデータソースに用意されているわけではなく、Grafana Alloyを前提としているので、対応データソースもGrafana Alloyが対応しているものに限られるようです。まぁそれでも十分に対応範囲は広いわけですが。

[Grafana Alloy \| Grafana Alloy documentation](https://grafana.com/docs/alloy/latest/)

Grafana Alloyリリースの公式ブログです。日付が2024年4月9日になっていて、結構最近です。

[Introducing an OpenTelemetry Collector distribution with built\-in Prometheus pipelines: Grafana Alloy \| Grafana Labs](https://grafana.com/blog/2024/04/09/grafana-alloy-opentelemetry-collector-with-prometheus-pipelines/?pg=oss-alloy&plcmt=hero-btn-2)


### Grafana Agent方式は今後どうなるのか？

これまで、メトリクスの収集にはGrafana Agentをクライアントにインストール方式もありましたが、今後Grafanaとしては、メトリクスの収集方式はこのGrafana Alloyに統一され、Grafana Agentは、順次サポートが終了されるというアナウンスがでています。

[From Agent to Alloy: Why we transitioned to the Alloy collector and why you should, too \| Grafana Labs](https://grafana.com/blog/2024/04/09/grafana-agent-to-grafana-alloy-opentelemetry-collector-faq/)

### 現在対応しているIntegrationsの一覧

[Integrations reference \| Grafana Cloud documentation](https://grafana.com/docs/grafana-cloud/monitor-infrastructure/integrations/integration-reference/)

上記をざっと見るとわかりますが、196種（2024年8月16日現在）のIntegrationが提供されてます。ほぼメジャーどころのDataSourceは、この中にあるんじゃないでしょうか？

というわけで、Grafanaに新しいメトリクスを追加するときは、DataSourceではなく、このIntegrationsの一覧を先にみておくのが、手っ取り早く試すのに良さそうです。

## Grafana Integrationsを利用したMacメトリクスの連携方法

Integrationsを試してみるということで、今回はどこにでもあるMacOSのメトリクスを連携します。WindowsもLinuxもあるので、皆さんの環境の好きなクライアントで試してみると良いんじゃないでしょうか。ざっと見た限り、Integrationsの導入手順はどれも似たようなものでした。

### Integrationsの追加

- 左側メニュー、[Connections]から[Add new connections]をクリック。
- [Search connections]で`macOS`と入力し、Integrationsを選択。

そうすると、このようなmacOSについてのIntegration画面が表示されます。

010

上部タブで、概要や設定方法、取得できるメトリクスの一覧、アラートなどが確認できます。

### Integrationを設定してみる（Grafana Alloyのインストールと設定）

Integrationsのページで、[Configuration Details]をクリックすると設定方法が表示されるので、この通りに進めていきます。

プラットフォームを指定して、Grafana Alloyをインストールします。インストール用のコマンドはArchitecture(`Amd64` or `Arm64`)によって異なるので、皆さんのクライアント環境にあったものを利用してください。

参考までに、Macのアーキテクチャは以下のコマンドで確認できます。

```bash
$ uname -m
arm64
```

プラットフォームを選択すると、Grafana Alloyのインストールに進みます。[Run Grafana Alloy]ボタンをクリックすると、インストール手順が表示されます。

MacのクライアントとGrafanaクラウドとの接続のためにトークンを新規発行し、そのトークンを利用して、Grafana Alloyの設定ファイルを作成する流れになっているので、表示された手順通りに進めればOKです。

020

一通り設定が完了したら、[Test Alloy connection]ボタンをクリックして、こんな感じで疎通できれば、クライアントとGrafana Cloudの疎通は完了です。

030

このあと、収集するメトリクスの設定にはいります。自分はこんな感じに設定しました。

- Extended metrics: on
- Include logs: on
- Simple set-upを選択

040

最後に、Grafana Alloyの設定ファイルに、画面に表示されている内容を追記します。設定手順に記載されているとおりですが、HomebrewでGrafanaをインストールした場合、Grafana Alloyの設定ファイルは以下に格納されています。

```
$(brew --prefix)/etc/alloy/config.alloy
```

上記パスを適当なエディタで開き、画面下部に表示されているテキストを設定ファイルに追記（上書きではない！）します。

あとは、Grafana alloyを再起動（Homebrewインストールの場合は、`brew services restart alloy`）し、[Test connection]ボタンをクリック。このようなメッセージが表示されれば設定完了です。

050

### ダッシュボードのインストール

上記までの設定でも、MacOSのメトリクスをGrafana Cloud上でデータソースとして扱うことができますが、Integrationsにはあらかじめ設定されているダッシュボードもあるので、[Install dashboards and alerts]セクションで[Install]ボタンをクリックすることで、いい感じのダッシュボードをインストールできます。

そうすると、Dasyuboardsの一覧に`MacOS / logs`と`MacOS / overview`のダッシュボードが追加されているので、クリックして中身を見てみましょう。


## 追加されたMacOSダッシュボードを確認する

### MacOS / overviewダッシュボード

`MacOS / overview`のダッシュボードを見てみます。一番最初、「No data」と何も表示されていない場合は、おそらくダッシュボード左上の`Data source`に別のものが設定されているので、指定のData sourceを選択しましょう。私の環境では`grafanacloud-macのホスト名-prom`という名前になっていました。

このような感じでダッシュボード表示されていればOK。

060

デフォルトではこのダッシュボードはReadOnlyの状態なっています。もし編集がしたい場合は、ダッシュボード右上の[Make editable]ボタンのクリックで編集が可能となります。

各パネルで、実際にどのようなクエリが動いているかは、各パネルにマウスオーバーした時に表示される縦3つの点クリックから[Explore]をクリック。

070

クエリ内容と結果をExploreメニューで表示できます。

080

CPU使用率の取得だけだと簡単そうに見えますが、実際のPromQLのクエリ内容はそれなりに複雑であることがわかります。これを見ながら編集することで、PromQLに慣れてみるという使い方もできます。

### MacOS / logsダッシュボード

ログのダッシュボードも見てみます。アクセスするとこのように、Grafana Lokiで収集されたMacOSのsyslogが表示されます。

090

LogsのパネルもExploreを選択することで、Lokiのクエリ内容を知ることができます。

100

あとは、実際に既存のパネルをあれこれ

## 追加されたアラートを確認する

Grafana IntegrationsのMacOSをインストールすると、ダッシュボードとは別にアラートも設定されています。左側のメニューから[Alert & IRM] -> [Alerting] -> [Alert rules]に、このようにアラートが設定されています。

110

ここから各アラートの内容と、そこから連携するNotification policiesを確認できます。こちらも、実際にどのようなメトリクスを拾ってどのようなNotificationが設定されているかを見ることで、アラート設定方法も追加で学ぶことができます。

## PrometheusのクエリーやLoki、アラートの設定方法を学習する素材として

以上、MacOSのIntegrationsを元に、ダッシュボードの構築とアラート作成を実施してみました。今回の内容は、実際に設定して監視するというよりは、Prometheusのクエリーやログ取得のためのLokiを、Integrationsから作成されたダッシュボードを元に学習するのを主な目的としていました。

Grafanaの機能は本当に幅広く奥が深いのですが、学習ハードルが高いのも事実。誰の環境にもあるクライアントPCを題材にGrafana Cloudに取り込んでみて学ぶことで、実際にGrafana利用の肌感を得ることができるので、是非参考にしてみてください。

それでは、今日はこのへんで。[濱田孝治（ハマコー）（@hamako9999）](https://twitter.com/hamako9999)でした。
