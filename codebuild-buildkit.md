（仮）AWS Code Buildが次世代高速コンテナビルドのBuildKitをサポートしました！

年明け、正月休みモードでのんべんたらりとFacebookを眺めていたら、驚きのニュースが！

<a href="https://medium.com/sakajunlabs/aws-codebuild-buildkit-cf8596e1b43e" target="_blank">AWS CodeBuild で Docker 18.09 が使えるようになったので BuildKit をためしてみた</a>

<strong>BuildKitは次世代高速コンテナビルドとも言えるもので、ちょっとした設定変更で皆さんのDockerビルドも高速化できる可能性があるので、是非導入を検討してみましょう。</strong>

<pre style="line-height:120%;">
BuildKit きたか…!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>


## AWS Code BuildのDockerランタイムバージョン18.09.0対応

今回のアップデートで、AWS Code Buildで利用できるマネージド型イメージのDockerランタイムに、バージョン18.09.0が新たに利用できるようになりました。リリースノートはこちら。

<a href="https://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref-available.html" target="_blank">Docker Images Provided by AWS CodeBuild - AWS CodeBuild</a>

従来、AWS Code **Buildで提供されていたバージョンは17**.09.0で、Docker本家でリリースされたのは2017年9月26日と、結構古いものでした。

今回新しく提供されたバージョン18.09.0は、2018年11月8日リリースのもので、2019年1月4日現在の最新Dockerバージョンとなります。

AWS Code Buildでは、利用するイメージに自分でDockerbuildしたイメージを使うこともできますが、今回マネージドとして提供されたことにより、イメージの管理を自分で実施する必要なく、AWS公式のセキュアなビルドイメージを利用することができます。

## バージョン18.09.0対応でなにがうれしいの？

<strong>一番インパクトがあるのが、BuildKitへの対応です。</strong>BuildKitのメンテナであるNTT須田さんのスライドが異常にわかりやすいので、まずは一読をオススメします。従来のDocker buildの不満点とBuildKitによる改善点がまとまっています。

<a href="https://www.slideshare.net/AkihiroSuda/buildkit" target="_blank">BuildKitによる高速でセキュアなイメージビルド</a>

010.png

DAG構造を採用した中間言語であるLLBを用いることで、各レイヤーの依存関係を正確に表現し、キャッシュが良く効くようになっています。これにより、従来はマルチステージビルドにおいて依存関係がないビルドも直列で実行されていたのが、BuildKitにより並列実行が可能になっています。

BuildKitの使い方ですが、環境変数<code>DOCKER_BUILDKIT</code>に<code>1</code>を設定するだけで有効にできます。

また、新しいDockefile構文<code>RUN --mount=type=cache</code>や、機密情報を安全にマウント可能な<code>RUN --mount=type=secret</code>による、S3やSSH鍵のコンテナ内利用もサポートされています。

### BuildKitまとめ

その他BuildKitのまとめを須田さん資料より引用します。

* Dockerfileの各ステージを並列実行できる
* コンパイラやパッケージマネージャのキャッシュを使える
* AWSやSSHなどの鍵を安全に扱える
* Dockerfile以外の言語も使える（Buildpackesなど）
* root権限無しで実行できる
* 最新のDockerなら、<code>export DOCKER_BUILDKIT=1</code>するだけですぐに使える

## Code BuildでmobyプロジェクトをBuildKitでBuildしてみる

AWS Code Buildを利用して、BuildKitの有無によるビルド時間の比較を実施してみます。

### 利用するDockerfile

今回は本家mobyプロジェクトのDockerfileを利用します（mobyプロジェクトは、OSSとしてのDocker）。

<a href="https://github.com/moby/moby" target="_blank">moby/moby: Moby Project - a collaborative project for the container ecosystem to assemble container-based systems</a>

### 利用するbuildspec.yml

今回、BuildKitのある無しでビルド時間の差分を把握するため、Code Buildで利用するbuildspec.ymlを2種類用意します。環境変数のセットとDockerビルドするだけの非常にシンプルなものです。

BuildKitを利用するときは、環境変数<code>DOCKER_BUILDKIT</code>に<code>1</code>をセットする必要があるため、それぞれの環境変数を0と1を用意しておきました。また、ビルド時のキャッシュを無効にするために、両方で<code>--rm --no-cache</code>を指定します。

[yaml title="normal_buildspec.yml"]
version: 0.2

env:
  variables:
    DOCKER_BUILDKIT: "0"

phases:
  build:
    commands:
      - docker build --rm --no-cache -t moby:normal .
[/yaml]

[yaml title="buildkit_buildspec.yml"]
version: 0.2

env:
  variables:
    DOCKER_BUILDKIT: "1"

phases:
  build:
    commands:
      - docker build --rm --no-cache -t moby:buildkit .
[/yaml]

これらのファイルを、Forkしたmobyプロジェクトのリポジトリ直下に追加し、Pushしておきます。

### CodeBuildの設定

CodeBuildの設定は非常にシンプル。今回は、BuildKit用とノーマル用の2つのビルドプロジェクトを用意します。

* プロジェクト名
  * ノーマル用：moby-normal-build
  * BuidKit用：moby-buildkit-build

* ソース：上で用意したリポジトリを選択。

* 環境：マネージド型イメージのDockerランタイムで、バージョン18.09.0が選択可能になっているのでそちらを指定。

020.png

* Buildspec
  * ノーマル用：normal_buildspec.yml
  * BuidKit用：buildkit_buildspec.yml

その他は全てデフォルトにしておきます。

## ビルド時間が約1.36倍向上

それぞれのビルドプロジェクトを起動し、ビルドにかかった時間を測定しました。

* ノーマル用でのビルド時間：7分12秒
* BuildKit用でのビルド時間：5分16秒

ビルド時間で<strong>約1.36倍</strong>の速度向上が確認できました。もちろんこの値は、Dockerfileの構造やCode Buildで利用するコンピューティング環境に依存するので一概には言えませんが、ランタイムバージョンを挙げて、環境変数を設定しただけの結果としてはかなりインパクトがある結果と言えるのでは無いでしょうか。

BuildKitを利用すると、ビルドログの出力内容も変わるので面白いです。

ノーマル用ビルドのログ。

030.png

BuildKit用ビルドのログ。

040png

## 前バージョンCode BuildからBuildKitへの移行設定内容

現在、Code BuildでDocker buildしている環境も非常に多いかと思いますが、BuildKitへ移行するときの変更点は基本的に以下の2つのみです。

* ランタイムバージョンの変更（17.09.0　→　18.09.0）
* ビルド環境への環境変数の設定（export DOCKER_BUILDKIT=1）

Code Buildに対して環境変数を設定する方法として、上の例では<code>buildSpec.yml</code>に指定しましたが、他にも複数方法は複数あります。

* buildspec.ymlのvariablesで指定
* buildspec.ymlのparameter-storeで指定
* CodeBuildプロジェクトの環境変数で指定
* ビルド実施時の環境変数上書き

このあたりは下記記事が非常に詳しいので、是非皆さんの環境に合わせて一番運用がフィットしている方法を選択いただければと思います。

<a href="https://dev.classmethod.jp/cloud/codebuild-env/" target="_blank">[CodeBuild]buildspec.ymlでの環境変数指定方法あれこれまとめ ｜ DevelopersIO</a>

## まとめ「簡単な設定変更で、コンテナ運用のライフサイクル全般が高速化される可能性有り」

<strong>Dockerビルドの高速化は、コンテナ開発〜運用のライフサイクルにおいて非常に重要な要素です。</strong>

Dockerを普段遣いされている環境では、CI/CDも整備されていてコードプッシュからの自動ビルド→環境適用を自動化している人も多いんじゃないでしょうか。特に、すでにマルチステージビルドを採用していてそれらのステージに依存関係がない場合は、確実に早くなります。

もちろん、事前検証による評価は必須ですが、前述したように設定変更はそれほど手間ではないため、まずは皆さんの環境で気軽に試してみていただければと思います。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

## Dockerビルドの参考記事

先日参加したこちらのイベント、Dockerビルドに命をかけた人たちの超マニアックで有用な話が満載だったので、Dockerfileの書き方に悩んでいる全ての人にオススメです。

<a href="https://dev.classmethod.jp/tool/docker/docker-build-meetup-1/" target="_blank">【docker buildのマニアックすぎる狂宴】Container Build Meetup #1に参加してきた #container_build ｜ DevelopersIO</a>

