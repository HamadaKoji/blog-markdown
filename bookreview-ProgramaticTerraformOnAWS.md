# Terraformのエッセンスが凝縮された「Pragmatic Terraform on AWS」が素晴らしい

010.png

<strong>「Terraform触りたい。触りたくない？」</strong>

クラスメソッドには、「CloudFormation派」と「Terraform派」があり、それぞれの派閥間には微妙な緊張感が漂っています。

自分は、ここ半年ほどずっぽりCloudFormationを使ってたんですが、Terraform派の「いやぁ、扱いやすいですよこれ」という言葉を聞くにつれ「まじか、どないなもんやねん？」と思ってました。

ただね、おっちゃんになると全くの未知の領域を学ぶのも腰が重くなる。「なんか良い本無いかなぁ」とグダグダしてたときに、<strong>「Pragmatic Terraform on AWS」</strong>という、もう、AWSど真ん中の自分にはこれしかないやろという本が技術書典で発売されたと知って、秒で購入しました。

最高でした。

<pre style="line-height:120%;">
Terraform入門本きたか…!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>


## 「Pragmatic Terraform on AWS」について

技術書展で販売されていただけあって、購入はBOOTH経由となります（物理本ほしかった…）。PDF版とEPUB版両方あるのはありがたい。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://booth.pm/ja/items/1318735" target="_blank">【ダウンロード版】Pragmatic Terraform on AWS - KOS-MOS - BOOTH</a>
</p>

### 著者

KOS-MOSさん（<a href="https://kosmos.booth.pm/" target="_blank">KOS-MOS - BOOTH</a>）

twitterアカウントは<a href="https://twitter.com/tmknom" target="_blank">tmk.nom(@tmknom)</a>。

### 本書の推しポイント

<blockquote>
・Terraform 0.12で登場する「HCL2」に早くも対応<br />
　・HCL2対応の書籍はたぶん世界初？<br />
・コードはすべてGitHubでも公開<br />
　・「0.11.13」で動作確認したHCL1系のコードもあわせて公開<br />
・多様なサービスを網羅<br />
　・ECS Fargate、CodePipeline、Session Manager、ALB、RDSなどなど<br />
・Terraformの設計に関する知見を凝縮<br />
　・コードの構造化やベストプラクティス、モジュール設計などなど</blockquote>


### 対象読者

<blockquote>Terraformの初級者から中級者です。たくさんのAWSのサービスを扱うため、AWSの公式ドキュメントも参照しながら本書を読むと、より理解が深まります。</blockquote>

### 書籍内ソースコード

こちらのGitHubで公開されています。

* <a href="https://github.com/tmknom/example-pragmatic-terraform-on-aws" target="_blank">tmknom/example-pragmatic-terraform-on-aws: 技術書典6で頒布した『Pragmatic Terraform on AWS 』のサンプルコードを公開しています</a>

### 目次

* 第1章　はじめに
* 第2章　インストール
* 第3章　Terraformの基本
* 第4章　全体設計
* 第5章　権限管理
* 第6章　ストレージ
* 第7章　ネットワーク
* 第8章　ロードバランサとDNS
* 第9章　コンテナオーケストレーション
* 第10章　バッチ
* 第11章　鍵管理
* 第12章　設定管理
* 第13章　データストア
* 第14章　デプロイメントパイプライン
* 第15章　SSHレスオペレーション
* 第16章　ロギング
* 第17章　構造化
* 第18章　Terraformベストプラクティス
* 第19章　AWSベストプラクティス
* 第20章　モジュール設計
* 第21章　落ち穂拾い
* 第22章　巨人の肩の上に乗る

以下、赴くままに本書の紹介をしていきます。


## 入門編（1〜3章）でTerraformの基礎を学ぶ

最初の3章で、Terraform実行のための環境セットアップと基本的なコマンドを習得します。

### （重要）Terraform環境セットアップ時の注意

冒頭環境セットアップについて、以下の通り著者の方が補足されています。<strong>本書読む人は必ず確認して初期セットアップを実施しておきましょう。</strong>

<a href="https://nekopunch.hatenablog.com/entry/2019/04/18/094914" target="_blank">『Pragmatic Terraform on AWS』のハマりどころと回避方法 #技術書典 - 憂鬱な世界にネコパンチ！</a>

もしくは、自分の環境ではついこの間（4月19日）にリリースされた、<code>Terraform v0.12.0-beta2</code>を利用して、最新のawsプロバイダーバージョンが利用できました。おそらく4月22日時点では、以下の組み合わせで実施するのが一番最新の状態で実行できるかと思います（安定性は保証しませんが…）。

[bash]
$ terraform version
Terraform v0.12.0-beta2
+ provider.aws v2.7.0
[/bash]

### Terraformの基本

ここで、以下のコマンドを中心に、terraformの骨格となるコマンドを学んでいきます。めっちゃ丁寧なので、一度環境できてしまえばおそらくハマりポイントは無いかと。

- terraform init
- terraform plan
- terraform apply
- terraform destroy

みんな大好きEC2を実際にAWSに構築、修正、削除しながら、planの確認方法、HCLの文法の基本などを一通り学びます。<strong>基本全てがハンズオン形式で記載されているので、写経しながらの学習が記憶の定着に良いですね。</strong>単に書籍の内容をなぞるだけじゃなく、「あれできんのかな？」「これやったらどないなるんやろ？」と想像を膨らませながら試行錯誤したほうが、絶対に頭に残ります。

最初にTerraform触ってて一番印象に残ったのが、リソース構築時の<code>terraform plan</code>のみやすさ。指定した値と、構築後に返される値が明示されていたり、

[bash]
  # aws_instance.example will be created
  + resource "aws_instance" "example" {
      + ami                          = "ami-0f9ae750e8274075b"
      + arn                          = (known after apply)
      + associate_public_ip_address  = (known after apply)
      + availability_zone            = (known after apply)
      + cpu_core_count               = (known after apply)
      + cpu_threads_per_core         = (known after apply)
      + get_password_data            = false
      + host_id                      = (known after apply)
      + id                           = (known after apply)
      + instance_state               = (known after apply)
      + instance_type                = "t3.nano"
      + ipv6_address_count           = (known after apply)
      + ipv6_addresses               = (known after apply)
[/bash]

パラメータの変更点は明確に判別できるようになっていたり（Nameタグ変更例）

[bash]
~ tags                         = {
  ~ "Name" = "hamako instance" -> "hamako instance desuyo"
}
[/bash]

ここらへんの操作感がごっつ新鮮で、いろんなパターンを試したくなります。まさに、最初のとっかりとしてはうってつけの素晴らしい内容です。

## 実践編（4〜16章）でAWSを学びながら構築しまくる

<blockquote>いよいよここから本格的なTerraformの実装に入ります。</blockquote>

4章からいきなり実践編で、ここからはDocker化したWebサービスを提供するシステムを構築する、というテーマで順にそのシステムに必要な各種AWSリソースをTerraformで構築しまくります。

対象のAWSリソースこんだけあります。てんこ盛り。

* 第 5 章「権限管理」 - IAM ポリシー、IAM ロール
* 第 6 章「ストレージ」 - S3
* 第 7 章「ネットワーク」 - VPC、NAT ゲートウェイ、セキュリティグループ • 第 8 章「ロードバランサと DNS」 - ALB、Route53、ACM
* 第 9 章「コンテナオーケストレーション」 - ECS Fargate
* 第 10 章「バッチ」 - ECS Scheduled Tasks
* 第 11 章「鍵管理」 - KMS
* 第 12 章「設定管理」- SSM パラメータストア
* 第 13 章「データストア」 - RDS、ElastiCache
* 第 14 章「デプロイメントパイプライン」 - ECR、CodeBuild、CodePipeline • 第 15 章「SSH レスオペレーション」 - EC2、Session Manager
* 第 16 章「ロギング」 - CloudWatch Logs、Kinesis Data Firehose

この実践編の最大の特徴は、<strong>AWSの各種リソースの実践的な構築方法を学びながらTerraformでの記述方法を学ぶことができる点</strong>でしょう。

例えば、「第5章　権限管理」では、AWSの各サービスを構築するにあたり必須の知識であるIAM関連の以下内容を、順にterraformで構築しながら合わせてIAMの基礎も学習できます。

- ポリシードキュメント
- IAMポリシー
- ロール
- 信頼ポリシー
- IAMロール
- ポリシーのアタッチ

S3でも、単にS3バケットを作成するだけではなく、パブリックアクセスをブロックしたりACLやCORSを学んだり、バケットポリシーを設定したり、とかくセキュリティの穴になりがちだけど超重要なS3についても、その構築の基礎を学習可能。

<strong>まさに実践と言えるのがこの章で、terraformでハンズオン形式で各種リソースを構築しながら、同時にAWSを構築するときに必要な考え方も合わせて習得できるのは、楽ですね。</strong>

各解説ページにAWS公式ドキュメントへのリンクが都度記載されているのも、親切心満載。

コンテナの設定管理のところで、機密情報をSSMパラメータストア使ってあるあたりとか、そもそもリポジトリに機密情報を入れないためにどういう運用にすべきなのか、というところまで記載があったり、まさに実践といった無いようなので、あまり馴染みがないサービスであれば、一度目を通しておくことをオススメします。

## 設計編（17〜22章）で複数人での運用に耐えるTerraformの書き方を学ぶ

<blockquote>ここから大きく話を転換し、Terraformの設計について議論していきます。本章のテーマは、コードの構造化です。構造化のポイントは「モジュール」「環境」「コンポーネント」の3つです。</blockquote>

<strong>この章、最高！</strong>

CloudFormationなどのIaCでもそうですが、複数人で一定規模のインフラを管理しようとすると、以下の問題に必ずぶつかります。

- 環境差分（開発〜検証〜本番）をどう管理、吸収するか？
- リソースごとに、どうやってディレクトリを分割するか？
- メンテナンスがやりやすい、構造がなにか？

そういった悩みに真正面から答えてくれるのが、この設計編。<strong>ぶっちゃけ1人でちょっとしたリソースを扱うぐらいではあまり問題になりにくいのですが、ある程度Terraformを使い込んでくると必ず出てくる、こういったIaC特有の超有る有るの悩みにまで踏むこんで解説してくれます。</strong>

単一ファイルに全てが書かれているモノリスな状態から、徐々に保守性が高い管理しやすい構造にファイルを分割していきます。環境毎の<code>tfstate</code>を管理する方策や、コンポーネント分割の方法、依存関係の制御方法あたりは、CloudFormationでも必要な考え方。

この章で紹介されていた、Terraformの標準的なモジュール実装方式がこれまた役立ちます。

* <a href="https://www.terraform.io/docs/modules/index.html" target="_blank">Creating Modules - Terraform by HashiCorp</a>

<code>main.tf</code>、<code>variables.tf</code>、<code>outputs.tf</code>を必ず用意すべき理由、ファイルの命名、サブディレクトリの作り方などが、一目瞭然わかりやすい。

その他、ドキュメンテーションやバージョニング方針、公開モジュールの登録方法まで記載があり有用な知見が満載なので、この設計編も必ず読んでおくことをおすすめします。

## 一つの書籍で超効率よくTerraformを学ぶのに最適な一冊

010.png

<strong>書籍というフォーマットで、基礎編〜実践編〜設計編がギュッとまとまっているので、Terraformの知見を得るのにめっちゃくちゃ効率が良いです。</strong>

一度これを読んでおけばベースの考え方や基礎が身につくので、公式ドキュメントや関連情報を参照する速度も上がります。

自分、AWSとCloudFormationはある程度経験はありつつもTerraformは全く触ったことがないど初心者だったのですが、3時間ほど通読しながらこの本をもとにあれこれ手で動かすことで、一気に肌感的にTerraformのことがわかってきた感触があります。

全てのTerraformを学びたい人に推薦したい一冊ですね。これからは、この本を常に片手に携えながらTerraform道を邁進していきます。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。
