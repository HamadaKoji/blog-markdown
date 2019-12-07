# EC2のイメージ作成を自動化するEC2 Image Builderが発表されました！

<strong>「イメージの定期更新、いつも俺がやってるねんけど、これめんどくさい…」</strong>

何らかのカスタムしたEC2イメージを利用してシステムを運用している場合、そのイメージのメンテナンスは非常に手間がかかるものです。<strong>イメージにはいろんなアプリケーションが含まれますが、それらの更新をせずに放置したイメージを使い続けると、システム全体のセキュリティリスクが増大してしまいます。</strong>

そんな手間を一気に解決させるサービスが、今般、AWSからリリースされた<strong>EC2 Image Builder</strong>です。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://aws.amazon.com/jp/blogs/aws/automate-os-image-build-pipelines-with-ec2-image-builder/" target="_blank">Automate OS Image Build Pipelines with EC2 Image Builder | AWS News Blog</a>
</p>

今まで手動で実行したり、Packerなどのサードパーティ製ツールを使って実装する必要があったイメージ更新の自動化〜テストが、このサービス一つで全部広範囲に対応することができます。

すぐに使えるサービスなので、まずはこの記事を見ながら簡単なレシピを作成し、この機能の素晴らしさを体感してみてください。

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     ｲﾒｰｼﾞﾏﾂﾘﾀﾞﾜｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>


## EC2 Image Bullderとは？

ドキュメントはこちら。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://aws.amazon.com/jp/image-builder/" target="_blank">EC2 Image Builder</a>
</p>


AWS上でアプリケーションを運用している場合、利用するEC2のイメージの最新化は非常に手間がかかるものでした。仮想マシンを手動で更新し都度手動でスナップショットを取得したり、自動化スクリプトを作成して動かしたり。これは、開発チームにとって大きな負荷でした。

このたびリリースされた、EC2 Image Builderが発表されたことで、Amazon Linux2やWindows Serverのイメージ作成をより簡単かつ迅速に保守、運用できるようになります。

HashiCorpのPackerでイメージの作成を自動化していた現場では、その代替になりうるマネージドサービスと言えます。

## 実際にパイプラインを作ってみた

ざっとドキュメントを眺めてみましたが、できる機能は非常に多いため、まずは手を動かして何ができるかを試してみます。

イメージビルダーのWebコンソールにアクセスします。

<a href="https://ap-northeast-1.console.aws.amazon.com/imagebuilder/home?region=ap-northeast-1#landingPage" target="_blank">EC2 Image Builder</a>

010

「Create image pipeline」をクリック。

### レシピの定義

「Define recipe」画面に遷移します。最初に作成するイメージの種類と元になるイメージを設定します。ここは一度作成すると修正できません。

020


ここでは、Amazon Linux2を選択し、「Select managed images」をクリック。その下のイメージ選択画面で、「Brows Images」ボタンを押すとイメージ選択画面が出てきます。イメージは下記3種類からフィルタリング可能。

- 「自分で作成したイメージ」
- 「自分で共有したイメージ」
- 「AWSが公開しているイメージ」

030

ここでは、<code>Amazon Linux2 x86</code>を選択。また「Initiate a new image build when there are updates to your selected image version.」にチェクをつけます。

### ビルドコンポーネントの選択および作成

次に、このベースイメージに導入するビルドコンポーネントを選択します。このビルドコンポーネント自分で作成もできますが、あらかじめAWSで用意されているものがあるので、それを今回は利用します。

040

「Browse build Components」をクリックすると、このイメージに導入するコンポーネントとして既に用意されているものが一覧で表示されます。

050

適当に導入したいコンポーネントをクリックしていきます。こんな感じで、amazon-correttoやphpやpythonを選択してみました。

060

### テストの定義

最後に、イメージをテストするためのコンポーネントを選択します。

070

「Browse tests」ボタンをクリックすると、テストコンポーネントの一覧が表示されます。

080

ここから、イメージにテストさせていコンポーネントを選択し、元の画面に戻ります。

090

### パイプライン詳細の設定

パイプラインの名前と説明、パイプラインで利用するIAMロール、ビルドのスケジュールを設定します。

100

次の画面の「Configure additional settings」は、OutputAMIに<code>test-imagebuilder-output-ami</code>だけ設定し、最後に設定内容を確認後、パイプラインを作成します。

元の画面に戻ると、無事イメージビルドのパイプラインが作成されました！

110

## パイプラインを実際に実行してみる

早速、作成したパイプラインを実行してみます。チェックボックスで対象のパイプラインを選択後、「Action」ドロップダウンから、「Run pipeline」をクリック。

しばらくすると、パイプラインの実行詳細画面に遷移します。

120

最初はStatusが<code>Building</code>の状態ですが、しばらくするとStatusが<code>Enabled</code>になり、Output imagesに作成されたAMIが表示されています！いやっほぅ！

EC2のAMIメニューを見ると、確かに作成されたAMIが表示されていました。

130

## イメージ作成の超強力な手段となる新機能

AutoScalig　Groupや、アプリケーション開発〜運用のベースとするイメージの更新、特に含まれている各コンポーネントのバージョンアップ、およびそれに伴うテストは、システムのセキュリティリスクを抑えるために非常に重要です。ですが、その運用を自動化するのは難しく、手作業でするには手間がかかるものでした。

今回このEC2 Image Builderのリリースで、そのあたりがまるっと全てAWSのマネージド・サービスとして提供されることで、そのあたりのイメージ作成の自動化の手間が一気に解消される可能性があります。

<strong>イメージに導入するコンポーネントも独自で作成可能ですし、イメージ作成後のテストも自前で実装可能です。</strong>独自のアプリケーションの導入や、アプリケーションレイヤーでのテストも含めて自動化し最新のイメージを常に作成することで、日々のシステム運用やセキュリティの脅威が一気に改善されるかもしれません。

<strong>現在、イメージの更新が手間だなぁと思っているすべての人が使うべきサービスです。</strong>まずは、一度触ってみてはいかがでしょうか。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。


