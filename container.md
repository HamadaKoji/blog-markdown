# 「それコンテナにする意味あんの？」迷える子羊に捧げるコンテナ環境徹底比較 #cmdevio2019

s1

<strong>みなさんコンテナを使うことの意味を自信もって答えられるでしょうか？</strong>

ここ1年ほどコンテナ関連の仕事をメインでやっているハマコーですが、いろんなお客様からこういったお声をいただくことが多くありました。

* 「それはコンテナ化する意味があるの？」
* 「こんなコンテナ運用は危ない？」
* 「ECSの設定とか実際めんどい。docker runじゃだめ？」
* 「EKSって使えんの？」

そういう声を聴く中で、自分なりの答えを模索していたわけですが、岡山での弊社イベント<a href="https://dev.classmethod.jp/news/190406-developersio-okayama/" target="_blank">AWS最新技術の祭典Developers.IO 2019 at 岡山城</a>へ登壇するにあたり、そのあたりのもやもやを自分なりに昇華したのが、本日の内容です。

* 「このアプリをコンテナ化する意味があるのか、わからない」
* 「コンテナ化することで余計めんどくさくなった」
* 「AWSのコンテナサービスの何を使ったら良いのかわからない」

という悩みを抱えている方には、是非一度こちらの内容を見て、ヒントを掴んでいただければと想います。

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     ｺﾝﾃﾅ ﾀﾉｼｸ ﾔﾘﾏｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>

## 導入「迷いないコンテナ運用を目指して」

みなさん、コンテナ使っていますか？本日お話することは、一言、こちら。

s15

45分という短い時間なので、コンテナやAWS関連サービスの細かい技術要素の話はしません。どちらかというと、これからコンテナを導入してみようという人であったり、コンテナの運用に苦労している方に向けたセッションとなります。

それでは、<strong>「迷いないコンテナ運用を目指して」</strong>をテーマに始めていきます。

本日の話の流れは以下の通りです。

s18

## 「コンテナとは何かを理解する」

s20

最初に、コンテナそのものについて振り返りましょう。コンテナ型仮想化技術を図示したものがこちら。

s23

一般的には仮想化ソフトウェアを使わずにOSのリソースを隔離するので、起動のオーバーヘッドが少ないのが従来の仮想化技術との差別化要素です。

Dockerを使うには、何がなくともDockerのインストールが必要です。こちら（<a href="https://www.docker.com/products/docker-desktop" target="_blank">Docker Desktop for Mac and Windows | Docker</a>）からインストールしてみましょう。

一番最初に試してみて頂きたいのがこちら。<a href="https://pandoc.org/" target="_blank">Pandoc</a>というプロダクトを使ってmarkdownファイルをWord（docx）に変換します。1行です。Dockerインストールするだけでこれが動きます。

<code>sample.md</code>から<code>sample.docx</code>を作るコマンドがこちら。

[bash]
$docker run -v `pwd`:/source jagregory/pandoc -f markdown -t docx sample.md -o sample.docx
[/bash]

Pandocのインストールも何もしていないのに、いきなり文書変換できる理由がこちら。

s28

ここで、Dockerの使い方として覚えていただきたいことは、<strong>DockerはCLIっぽく使える</strong>ということです。

なので、従来の仮想化技術とは利用用途がかなりことなります。

s32

自分好みのイメージを作るには、Dockerfileを作る必要があります。

s34

Dockerfileの凄いところは<strong>「インフラのプロビジョニングからアプリケーションのインストールまで、一つのテキストファイルで記述できる」</strong>点です。

従来のEC2インスタンスをたてて、動かした場合との比較した表がこちら。

s37

インフラをコードで管理するのが最近のトレンドですが、コンテナの場合は、全てDockerfileで完結します。

Dockerfileを書いたり、container buildを極めたい方は、ぜひこちらの資料を御覧ください。

* <a href="https://docs.docker.com/develop/develop-images/dockerfile_best-practices/" target="_blank">Best practices for writing Dockerfiles | Docker Documentation</a>
* <a href="https://dev.classmethod.jp/tool/docker/docker-build-meetup-1/" target="_blank">【docker buildのマニアックすぎる狂宴】Container Build Meetup #1に参加してきた #container_build ｜ DevelopersIO</a>

## 「コンテナを使う意味を考える」

s40

<strong>そこにアプリケーションがあり、コンテナ化を迷った時の判断基準として必ず抑えておくべきなのは以下の3つです。</strong>

s45

この３つのうち、2つ以上YESなら、コンテナ化を検討してみましょう。

対して、コンテナ運用するには非常に微妙な例として、GitLabについて紹介します。

s47

GitLab CE、インストール方法は複数あるんですが、長きに渡って運用する場合、やらないほうが良いインストール方法があります。それがこちら。

s48

Docker Imageを利用したインストールって確かに簡単なんですね。ただ、それをきちんと長きに渡って運用していくとき、常にコンテナの中に状態をもつアプリケーションの運用は、はっきり言ってDockerには向いていません。普通にEC2で運用したほうが、絶対安定します。

先程の判断基準は、具体的なユースケースに基づいています。これらが必要な場合は、コンテナ化することのメリットは非常に大きくなります。

* 複数の環境でつかうか？
	* 開発〜検証〜本番で同じイメージを使う
* 頻繁に変更するか？
	* どんどん機能追加をしていきたい
* 増えたり減ったりするか？
	* 負荷に応じて弾力的に数を変えたい

アプリケーションをコンテナ化するときの指針はあります。まずは一番定番で王道なのがこちら。

* <a href="https://12factor.net/ja/" target="_blank">The Twelve-Factor App （日本語訳）</a>

もうすこし、Docker向けに焦点を絞っているのがこちら。

* <a href="https://dev.classmethod.jp/cloud/aws/black-belt-docker-on-aws-2017/" target="_blank">[AWS Black Belt Online Seminar] Docker on AWS レポート [12-factor App] ｜ DevelopersIO</a>

これらの情報は是非一通り目を通しておいてほしいのですが、重要なポイントを抜粋するとこうなります。<strong>もし、アプリケーションがこれを満たしていないのであれば、Dockerfileを作り始める前にこの対応をしましょう。</strong>

S53

## 「AWSにおけるコンテナについて理解する」

S55

AWSにおけるコンテナサービスを区別すると、まずはこうなります。

S57

### ECR（Amazon Elastic Container Registry）とは

S59

その他のサービスについて説明していきますが、まず最初にお知らせしておきたいのはこちら。

S62

ので、AWSのコンテナサービス郡を最初に理解するためには、<strong>コントロールプレーン</strong>と<strong>データプレーン</strong>について理解する必要があります。

### コントロールプレーンとデータプレーン

<strong>コントロールプレーンとは、コンテナの管理をする場所</strong>

* 動く場所の管理
	* VPC、Subnet、Load Balancer、Security Group
* コンテナ生死の管理
	* 監視設定、自動復旧
* コンテナ数の管理
	* 負荷に応じたコンテナ数のオートスケール

<strong>データプレーンとは、実際にコンテナが稼働する場所</strong>

* コントロールプレーンからの指示にしたがって起動
* コンピューティングリソースを消費（従量課金対象）
* 状態をコントロールプレーンに通知

これらの関係性を図示するとこうなります。

S66

今のサービス名とラベリングするとしたらこうなりますね。今回のセッションでも、この前提でお話していきます。

S67

### ECS（Amazon Elastic Container Service）とは

<strong>AWS完全マネージドのコンテナオーケストレーションツールです。</strong>

* 簡単なオートスケール設定
* ロードバランサー統合
* コンテナのIAM権限管理
* コンテナのセキュリティグループ管理
* CloudWatch メトリクス統合
* CloudWatch Logs統合
* スケジュール実行機能

最大の特徴は、<strong>元がAWSフルマネージドなサービスのため、他のAWSサービスとの連携が簡単</strong>という点です。

ただ、ECSの内部構造は決して簡単ではないです。ので、ここでは図示して一つずつ説明していきます。

まず、タスク定義とコンテナ定義の構造がこちら。

S75

コンテナ定義には、各コンテナ単位での設定情報（ほぼ、Docker run時に設定デキる内容）が含まれており、それらを統合してタスクとしてハードウェアリソースを割り当てます。

1タスクに複数コンテナを割り当てる代表的な例をあげます。

s76

さらに、クラスターとサービスの関係性がこちら。クラスターが全体的なサービスを論理的に統合し、サービスには、タスク定義から具体的にサービス提供するためのインフラ周辺の情報を登録することになります。

S77

実際にこれを、先程のタスク定義例で動かしたときの動作イメージがこちら。

S78

### EKS（Amazon Elastic Container Service for Kubernetes）とは

<strong>AWSマネージドなKubernetesサービス</strong>

* 正式名称はAmazon Elastic Container Service for Kubernetes
* Kubernetes正式準拠
* Kubernetesコミュニティが作成した既存のプラグインやツールを利用できる

最大の特徴は、<strong>元がオープンソースプロダクトなため、Kubernetes用に開発されたツールを
広く使うことができる</strong>という点です。

具体的には、Kubernetesのコントロールプレーン、ここをマネージドとして提供しているのがEKSです。

S83

こちらのデータプレーンは、EC2で提供され、実際の構築はCloudFormationなどを利用することになります。

S84

ここで、EKSの各特徴をECSに比べた観点で並べてみた表がこちら。

|   | **ECS** | **EKS** |
| --- | :---: | :---: |
|  **コンテナのIAM制御** | タスクへのIAMロール付与 | 公式未提供（OSSのkube2iam, kiam） |
|  **スケジュール実行** | スケジュールタスク | なし（Scheduler） |
|  **セキュリティグループ** | サービスへの付与 | EC2で自前管理 |
|  **監視** | CloudWatch統合 | なし（Datadog、Prometheus） |
|  **ログ** | CloudWatch Logs統合 | Fluentd |
|  **デプロイ** | CodePipeline統合 | 自前構築（helm, kustomize, spinnaker） |

<strong>コンテナのコントロールプレーンとしては、ECSよりもKubernetesのほうがはるかに自由度が高く高性能といえますが、マネージド未提供なものが多いです。</strong>

EKSを採用するときは、この点を真っ先に知っておく必要があります。

### Fargateとは

<strong>サーバーやクラスターを一切管理することなくコンテナを実行できます。</strong>

* 仮想マシンのクラスターのプロビジョニングが不要
* ECSおよびEKSに対応したコンピューティングエンジン
* コンピューティングリソースに対しての料金としてはEC2より割高（1割〜2割）

EC2オンデマンドに比べて、ランニング費用は割高ですが、以下のメリットを享受できます。

* ホストインスタンスの管理が省けること
* EC2の余剰リソースが不要
* セキュリティはAWS側で常にカバー
* オートスケールの設定も不要

この概念を、ごっつわかりやすい図で表したのがこちら。

S89

管理コストを氷山で表しています。水面から出ていて目に見える部分のコスト（ランニング費用）は、Fargateのほうが大きいですが、目に見えない運用上のコスト（スケーリング設定、セキュリティ管理、インスタンス管理）は、EC2のほうが圧倒的にでかい、ということです。

また、気になるランニング費用も先日のアップデートで、大幅に下がりました。是非利用を検討してみてください。

* <a href="https://dev.classmethod.jp/cloud/aws/fargate-lower-price/" target="_blank">【全世界のFargateファンに朗報】Fargate利用料が35%〜50%値下げされました！ ｜ DevelopersIO</a>

## 後悔しない選択をする

S92

最後、これらのAWSサービスから最適なコンテナサービスを選択する方法をお伝えします。

単純です。まずは、これを使ってください。

S94

Fargateは、EC2を隠蔽することでコンテナの管理〜運用に特化することができる素晴らしいサービスです。<strong>最初に検討すべきなのは、Fargateであると断言します。</strong>

<strong>ただ、ECS on EC2と比べてFargateではできないこともあるので、注意が必要です。</strong>

* ホストインスタンスにSSHログインできない
	* ホストインスタンスにSSHログインしてコンテナの状況を確認するコマンドが利用できない（docker ps、docker images、docker logs、docker execなど）
* ログドライバがawslogsのみ
	* 標準ではCloudWatch Logsのみに出力される
	* （Fluentdは将来対応予定）
* EFSが利用できない
	* 複数タスクで同一のローカルリソースを共有することが現状無理
* S3などを利用できないか検討する
	* スポットインスタンスが使えない
	* EC2でオートスケール部分をスポットインスタンスを使うことでコスト削減するなどの方法が使えない
* ホストインスタンスに導入する前提のセキュリティソフトが利用できない
	* コンテナ単体でセキュリティをチェックする仕組みが必要
* GPUインスタンスなど、リソース最適化されたEC2インスタンスが利用できない
* Windowsコンテナが利用できない

では、EKSを使うパターンとしては何があるでしょう？

<strong>代表的なパターンとしては、もともとk8sを自前（オンプレ、EC2）で運用していた場合。</strong>この場合は、もともとあるk8sのノウハウを活かしながら、コントロールプレーンの管理をマネージドに任せることでTCOを削減できます。

もうひとつ、EKSの導入に重要なポイントは？

S101

これ、案外真面目に言ってます。

<strong>Kubernetesを学ぶということは、モダンな分散アプリケーションのプラットフォームを管理する方法を学ぶということです。</strong>

S103

本来的にはアプリケーションの特性に応じて最適なプラットフォームを選択するのが筋ですが、オープンなプラットフォームであり、関連するサードパーティーやOSSが非常に充実して、今一番勢いがあるのは間違いなくKubernetesです。

そういった技術にど真ん中から飛び込みながら、貪欲に新しいものにチャレンジしていく組織であれば、EKSも非常に良い選択肢となりうるでしょう。

s109

## 迷いないコンテナ運用を目指して

S111

迷いないコンテナ運用を目指してということで、45分お話させていただきました。

* そもそも検討しているアプリケーションがコンテナに向いているかきちんと検討する
* そのアプリケーションのワークロードに当てはまるAWSのコンテナサービスを選定する
* 途中で状況が変わったら躊躇なく再検討しよう

様々な場面で運用しているアプリケーションをコンテナ化したほうが良いかどうか、AWSで何を使うか、検討はされると思いますが、今日の話が皆様のよりよいコンテナ運用の一助となれば、これほど嬉しいことはありません。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

## 発表資料

発表スライドはこちら。

<script async class="speakerdeck-embed" data-id="505ad2545635497ca80faf7340abe135" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>


