# 【速報】コンテナ実行専用OSのBottlerocketがパブリックプレビューで発表されました！

「ふわぁ、今日も仕事すっか…って、Bottlerocket、なんやねんこれ？」

朝の日課、twitterのTLを眺めていたら、いきなりこんなツイートが飛び込んできました。

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">🆕 Check out the public preview of Bottlerocket: a new Linux-based open source operating system that we designed and optimized specifically for use as a container host: <a href="https://t.co/i3tT3rG56g">https://t.co/i3tT3rG56g</a></p>&mdash; Nathan Peck (@nathankpeck) <a href="https://twitter.com/nathankpeck/status/1237500674822324225?ref_src=twsrc%5Etfw">March 10, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<strong>Bottlerocketは、Linuxベースのオープンソースのコンテナ専用OS</strong>で、それがパブリックプレビューとして発表されたとのニュースです。ほえぇ。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://aws.amazon.com/jp/about-aws/whats-new/2020/03/announcing-bottlerocket-a-new-open-source-linux-based-operating-system-optimized-to-run-containers/" target="_blank">Announcing Bottlerocket, a new open source Linux-based operating system purpose-built to run containers</a>
</p>

現在パブリックプレビューのため、実際に動かしてみた情報などはブログ化できませんが、ドキュメントベースで、Bottlerocketについての速報をまとめてみます。

<pre style="line-height:120%;">
コンテナ専用OSきたか…!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>


## Bottlerocket概要

公式ページはこちら。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://aws.amazon.com/jp/bottlerocket/" target="_blank">Bottlerocket — Amazon Web Services</a>
</p>

Bottlerocketは、コンテナをホストするために最適化されたOSです。現在はEKSをサポートしており、ECSには

<p class="alert">2020年3月11日現在、Bottlerocketはパブリックプレビューとして提供されており、GA（正式一般公開）ではありません。ご注意ください。</p>

### メリット１：パッケージ単位の更新ではなく、単一ステップによる更新

従来の汎用OSは、パッケージ単位で更新されるため、OSの更新を自動化することは難しかったです。それが、Bottlerocketでは、パッケージ単位での更新ではなく、単一のステップでOSを更新できるため、EKSなどのオーケストレーションサービス経由でのOS更新の自動化が容易となります。また、ロールバックも簡単です。

### メリット２：コンテナ実行に不可欠なソフトウェアのみの提供

コンテナを実行するホストOSにおいて、セキュリティやパフォーマンスの観点から、コンテナ実行に不要なパッケージはできるだけ含めないことが推奨されています。Bottlerocketは、コンテナ実行のために最適化されているため、汎用OSに含まれているコンテナ実行に不要なミドルウェアがそもそも含まれていません。SSHアクセスも推奨されておらず、別途有効化が必要になります。トラブルシューティング時には、別途個別の管理コンテナを利用します。

これにより、リソースの利用効率が向上し、脅威からの攻撃対象領域が減少します。

### メリット3：AMIの提供によるAWS環境での使いやすさ

従来、ECSやEKSのデータプレーンをEC2で動作させる場合、それぞれにAmazon Linux2を元に最適化されたOptimized AMIを利用することが普通でした。

- <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-optimized_AMI.html" target="_blank">Amazon ECS-optimized AMIs - Amazon Elastic Container Service</a>
- <a href="https://docs.aws.amazon.com/eks/latest/userguide/eks-optimized-ami.html" target="_blank">Amazon EKS-Optimized Linux AMI - Amazon EKS</a>

今回発表されたBottlerocketもAmazon Machine Image（AMI）が無料提供されているため、従来のECSやEKSのデータプレーンと、ほぼ同じ使い勝手で利用できることが期待できます。

### メリット４：オープンソースによるカスタマイズが可能

GitHubにソースが公開されているため、パートナーはそれぞれの利用状況に応じたカスタマイズされたビルドを作成できます。たとえば、オーケストレーターサポートとしてAWSのECSやEKSだけではなく、他のオーケストレーションツールへの対応も可能です。

### メリット5:オーケストレーターツールへの統合が容易

OSの更新がワンステップのため、Amazon EKSなどのオーケストレーターと統合し、ワンクリックでOS自体の更新やロールバックが管理できます。これで、データプレーンOSの更新を極力自動化することができ、運用コストを削減できます。

### メリット8：OCIイメージ準拠

BottlerocketはDockerイメージとOCI（<a href="https://www.opencontainers.org/" target="_blank">Open Containers Initiative</a>）に準拠しています。

### メリット7：３年間サポート付き

AWS提供のBottlerocketビルドは、リリースされてから３年間のサポートがついています。さらには、GitHubでのコミュニティサポートもあります。

## メリット8：追加料金無しでの利用

Bottlerocketの利用に特別な料金な不要。利用するワーカーノードとしてのコンピューティングリソース（EC2）にかかる料金だけが請求されます。


## Bottlerocketの使い方

100
（公式ページhttps://aws.amazon.com/jp/bottlerocket/より引用）

Bottlerocketの利用には以下の手順を踏みます。

1. Bottlerocket AMIを選択
2. Bottlerocket AMIを利用してEC2インスタンスを起動
3. EKSクラスターへのBottlerocketの組み込み
4. EKSを利用してBottlerocketの更新を管理

## Bottlerocket採用事例とテクノロジーパートナー

すでに、Bottlerocketの採用事例があるようです。

### Veeva

<a href="https://www.veeva.com/jp/" target="_blank">Veeva Systems</a>

Veeva Systemsは、ライフサイエンス業界向けのクラウドベースソフトウェアのリーダー。Bottlerocketへの移行は非常にスムーズで、他のEKSノードの置換で完了したそうです。

### テクノロジーパートナー一覧

元々コンテナテクノロジーパートナーに名を連ねていた企業のうち何社かが、Bottlerocketのテクノロジーパートナーにも入っています。

- Alcide
- Armory
- CrowdStrike
- DATADOG
- New Relic
- Sysdig
- Tigera
- Trend Micro
- Weaveworks


## Bottlerocket関連リソース

### FAQ

公式ページによくある質問とその回答が一通りまとまっています。

<a href="https://aws.amazon.com/jp/bottlerocket/faqs/?nc=sn&loc=2" target="_blank">Bottlerocket FAQ — Amazon Web Services</a>

### ブログ

<a href="https://aws.amazon.com/jp/blogs/aws/author/jbarr/" target="_blank">Jeff Barr</a>によるブログが公開されています。

<a href="https://aws.amazon.com/jp/blogs/aws/bottlerocket-open-source-os-for-container-hosting/" target="_blank">Bottlerocket – Open Source OS for Container Hosting | AWS News Blog</a>

### GitHub

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://github.com/bottlerocket-os/bottlerocket" target="_blank">bottlerocket-os/bottlerocket: An operating system designed for hosting containers</a>
</p>

詳細を知るには必須のリソースです。Bottlerocketについての詳細な情報は、カスタムビルドの実行方法、EKSで動かすための<a href="https://github.com/bottlerocket-os/bottlerocket/blob/develop/QUICKSTART.md" target="_blank">QUICKSTART</a>、SSHアクセスできないが、トラブルシュートするための方法などが紹介されています。

トラブルシュートの方法ですが、ざっと見た限りは、SSMエージェントが動作するコンテナを別途用意し、そこからシェルのセッションをBottlerocketのEC2インスタンスに起動するようです。

## データプレーンの管理を劇的に簡素化できる可能性あり

AWS上のコンテナワークロードにおいて、オーケストレーターやコンテナそのもの管理とは別で、データプレーンの管理〜運用も必要でした。Fargateではそもそもデータプレーンの管理が不要となりますが、EC2ベースではできないことがあったり、EKSでは基本的にはワーカーノードをホストインスタンスで管理するのが主流です。

まだパブリックプレビューの段階ですが、Bottlerocketが、これからのコンテナワークロードのより一層のセキュア化、リソース効率化、TCO削減に関わってくると思うと楽しいですね。自分も実際に触ってみつつフィードバックしながら、この新しいOSを味わってみようと思います。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。




