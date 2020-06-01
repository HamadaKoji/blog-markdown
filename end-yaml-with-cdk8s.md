# （仮）汎用プログラミング言語でKubernetesのYAMLを生成するcdk8sが発表されました！

<strong>「CDK（Cloud Developmet Kit）でKubernetesのYAMLを生成できるよ！そうcdk8sならね」
</strong>

というわけで、先日AWSより突然、<strong>cdk8s</strong>なるものが発表されました。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://aws.amazon.com/jp/about-aws/whats-new/2020/05/introducing-the-cdk-for-kubernetes-a-new-software-development-framework-and-open-source-project-for-defining-kubernetes-applications-using-code/" target="_blank">Introducing the CDK for Kubernetes, a New Software Development Framework and Open Source Project for Defining Kubernetes Applications Using Code</a>
</p>

正直最初これを見たときの第一印象は「なんて安易なネーミングなんや…」でしたが、見れば見るほど、KubernetesのYAMLをそのまま管理する辛さに対しての別次元のアプローチとして、すごい斬新で期待感が持てたので、意気揚々とブログを書いております。

<pre style="line-height:120%;">
YAML地獄から解き放たれるのか…!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>


## cdk8sとは

公式紹介ブログはこちら。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://aws.amazon.com/jp/blogs/containers/introducing-cdk-for-kubernetes/" target="_blank">Introducing CDK for Kubernetes | Containers</a>
</p>

cdk8sで出来ることを単純に定義すると、以下の図に集約されます。シンプル！

010


以下、公式ブログの内容を紐解きながら、自分の理解をざっくり書いていきます。


### Kubernetesの運用におけるYAMLの課題

Kubernetesの大きな特徴として、統一的なYAMLによるインフラの管理が挙げられます。Infrastructure as Codeの文脈で語られることも多いですが、YAMLファイルを定義してそれをクラスタにapplyすることで、コードベースでインフラ（Kubernetesクラスタ）全体を統一的に管理できます。

冪等性を持った構造化テキストファイルを元にクラスタ全体を管理することが出来るので、クラスタ運用のパイプライン実装による自動化が容易であったり、開発チームへのインフラ操作権限委譲がやりやすいなど多くのメリットがあります。

ただ、本格的にクラスタを運用していくなかで、多数のYAMLファイルを定義し管理する必要があり、アプリケーションの成長に伴い加速度的に、YAMLファイル自体の管理や他者へのスキルトランスファーが難しくなっていくという問題がありました。

そもそも、YAMLは優れた構造化フォーマットですが、ロジックや再利用可能な表現をするための機能を持っていません。

### cdk8sが解決してくれること

これらの問題について、既にAWSのインフラを構築するためにAWSがリリースし開発している、AWS Cloud Developmnet Kit(AWS CDK)が利用できることに気づきました。

CDK for Kubernetes(cdk8s)は、TypeScriptやPythonなどのプログラミング言語でKubernetesのYAMLを生成できます。

#### いろんなクラスターで動く

cdk8sの責任範囲はYAMLの生成までなので、AWSが提供しているEKSだけではなく、オンプレミスや他のパブリッククラウドなど、全てのKubernetesクラスターが適用対象となります。汎用的なプログラミング言語をベースとなっているため従来のIDEやツールを利用可能で、YAMLを直接管理〜運用するより、より高次元のレベルでKubernetesクラスタを管理することができます。

#### 命令的言語からの宣言的状態へのアプローチ

cdk8sは命令的言語を使って書かれていますが、出力されるYAMLは宣言的なものです。つまりは、従来からある宣言的アプローチによる堅牢性を維持したまま、命令的表現によるプログラミング言語ならではのアプローチを楽しむことができます。

#### 任意のKubernetes APIのバージョンとCRD（カスタムリソース）を利用可能

cdk8sは、出力するYAMLのKubernetesクラスターバージョンを指定する方法があります。また、カスタムリソース定義をインポートすることも可能です。

#### 複数言語のサポート

TypeScript、JavaScript、Pythonに対応しています。将来的にはGo言語のサポートもロードマップに含まれます。


## cdk8s関連各種リソース

その他全体的な特徴は、上で紹介したAWSブログを見てもらえればと思いますが、その他のcdk8s関連リソースをまとめます。

### 公式ページ

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://cdk8s.io/" target="_blank">Contributors and maintainers are governed by the CNCF Code of Conduct</a>
</p>

020

cdk8sの公式トップページです。ツールの概要と各種リソースへのリンクがあります。公式ページみてて思ったのは、ページ最下部に以下の記述があったこと。

<blockquote>Contributors and maintainers are governed by the CNCF Code of Conduct</blockquote>

ということで、CNCFのCoCに基づいて運用されるということですね。現在、[awslabs](https://github.com/awslabs)に格納されているツール群の中でこれは珍しいと思います。

### GitHub

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://github.com/awslabs/cdk8s" target="_blank">awslabs/cdk8s: Define Kubernetes native apps and abstractions using object-oriented programming</a>
</p>

1番メインとなるページ。Getting Startedや、ロードマップなども全てここで管理されてます。

### Getting Started

なにはなくとも手で動かさないとわかんないぜ！という方も多いですよね。現在、TypeScriptとPython用のGetting Startedが用意されています。

- TypeScript
  - <a href="https://github.com/awslabs/cdk8s/blob/master/docs/getting-started/typescript.md" target="_blank">cdk8s/typescript.md at master · awslabs/cdk8s</a>
- Python
  - <a href="https://github.com/awslabs/cdk8s/blob/master/docs/getting-started/python.md" target="_blank">cdk8s/python.md at master · awslabs/cdk8s</a>

TypeScriptでちょっと試してみましたが、Kubernetes界隈のリソースをコード補完しながら定義していける感覚は、超新鮮！よござんすな。

030.png

### Roadmap

<a href="https://github.com/awslabs/cdk8s/projects/1" target="_blank">Roadmap</a>


040

ロードマップもGitHub上で公開されています。Import Helm Chartsなんてのもアガってますね。


### cdk8s cli

<a href="https://github.com/awslabs/cdk8s/blob/v0.21.0/packages/cdk8s-cli/README.md" target="_blank">cdk8s/README.md at v0.21.0 · awslabs/cdk8s</a>

### Library

<a href="https://awscdk.io/packages/cdk8s@0.21.0#/" target="_blank">cdk8s</a>

### Webinar

<a href="https://www.cncf.io/webinars/end-yaml-engineering-with-cdk8s/" target="_blank">CNCF Member Webinar: End YAML engineering with cdk8s! - Cloud Native Computing Foundation</a>

先日、CNCFのWebinarで、cdk8sが紹介されていました。自分も一通り見てたのですが、cdk8sが解決しようとしていること、実際のライブコーディングによるデモやQ&Aなども全て収録されているので参考になると思います。


## cdk8sはKubernetesYAML管理のデファクトスタンダードになるか

もともとKubernetesのマニフェストファイルについては、その管理ツールとして様々な手段が提供されていました（kustomize, jsonnet, jkcfg, kubecfg, kubegen, Pulume）。

cdk8sは、最近AWSインフラの定義において非常に存在感を増しているAWS CDKの思想が元になっているのが大きな特徴と言えます。最近ウチのチームでも、主にサーバーレス周辺のリソースでは、CDKを使ってインフラをコード化し管理することがデフォルトになりつつあります。

まだ、αリリースの段階ではありますが、AWS CDKなみの盛り上がりを見せてくると、次代のKubernetesのYAML管理のデファクトスタンダードになり得る可能性も秘めていると思っています。今後の展開に注目ですな。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。
