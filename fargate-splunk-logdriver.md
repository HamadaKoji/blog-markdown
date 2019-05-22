# 【祝】FargateでログドライバーにSplunkが利用可能になりました！

AWS Fargate、みなさん利用していますか？

ホストインスタンスの管理なしにコンテナワークロードを実行可能な超絶便利なサービスなんですが、<strong>そのAWS FargateのログドライバーにSplunkが追加されました！</strong>

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://aws.amazon.com/jp/about-aws/whats-new/2019/05/aws-fargate-pv1-3-now-supports-the-splunk-log-driver/" target="_blank">AWS Fargate PV1.3 now supports the Splunk log driver</a>
</p>

従来、FargateではログドライバーにCloudWatch Logsしか指定できなかったのですが、ついに初めてのFargateのログドライバー拡張です。こりゃやってみるしか無いでしょ！というわけで、早速試してみた様子をお届けします。

<pre style="line-height:120%;">
ログドライバー追加きたか…!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>


## ログドライバーアップデート内容の詳細

今回のアップデートは、大きく分けて２つの部分から構成されます。

### 1.ログドライバーAPIの拡張

タスク定義内のコンテナ定義において利用するログドライバーが拡張されました。ログドライバーのAPIドキュメントがこちら。

- <a href="https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_LogConfiguration.html" target="_blank">LogConfiguration - Amazon Elastic Container Service</a>

以下の記述「splunk」が追加されています。

<blockquote>For tasks using the Fargate launch type, the supported log drivers are awslogs and <strong>splunk</strong>.<br ><p>引用：<a href="https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_LogConfiguration.html">LogConfiguration - Amazon Elastic Container Service</a></p></blockquote>

### 2.ログドライバーで機密情報のためマネージドサービスの参照可能化

splunk-tokenのような機密情報を扱うために、AWS Secrets ManagerやAWS Systems Manager Parameter Storeの値を参照できるようになりました。

詳細はこちらに記載されていますが、新しく<code>SecretOptions</code>という項目が追加されています。

- <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/specifying-sensitive-data.html" target="_blank">Specifying Sensitive Data - Amazon Elastic Container Service</a>

具体的なタスク定義のスニペットは、以下で参照してみてください。

* <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/specifying-sensitive-data.html#secrets-logconfig" target="_blank">Specifying Sensitive Data - Amazon Elastic Container Service</a>

[json]
{
	"containerDefinitions": [{
		"logConfiguration": [{
			"logDriver": "splunk",
			"options": {
				"splunk-url": "https://cloud.splunk.com:8080"
			},
			"secretOptions": [{
				"name": "splunk-token",
				"valueFrom": "arn:aws:secretsmanager:region:aws_account_id:secret:secret_name-AbCdEf"
			}]
		}]
	}]
}
[/json]

もちろん、上記設定はWebコンソールからの登録も可能です。

## FargateのログをSplunkに転送してみた

というわけで、実際にFargateのログをSplunkに転送してみます。

### Splunk環境の準備

まだSplunk使ったことない人は、先にSplunkを使える状態にしておきます。Splunkは、ホストインストール型もSaaS版も両方ありますが、自分はMarketPlaceからの起動が簡単

なホストインストール型を利用しました。

* <a href="https://aws.amazon.com/marketplace/pp/B00PUXWXNE" target="_blank">AWS Marketplace: Splunk Enterprise</a>

### SplunkへのHTTPイベントコレクタの登録

ログドライバーからは、SplunkのHTTPイベントコレクタ経由でログを流し込むので、事前にトークンの発行を含めたHTTPイベントコレクターの動作確認をしていきます。

設定は、公式のこちらを参考に実施してみてください。手順最後の方で、サンプルデータをcurlで投げてみて、無事データが登録されれば、HTTPイベントコレクタの設定は完了です。

* <a href="http://dev.splunk.com/view/event-collector/SP-CAAAE7F" target="_blank">Walkthrough | HTTP Event Collector</a>

curlでの動作確認例です。

[bash]
$curl -k http://<host>:8088/services/collector -H 'Authorization: Splunk <token>' -d '{"sourcetype": "mysourcetype", "event":"Hello, World!"}'
{"text":"Success","code":0}
[/bash]

### パラメータストアに、Splunkのトークンを格納

Fargateログドライバーには、以下の設定が必要になります。

- SplunkのHTTPイベントコレクタのエンドポイントURL
- HTTPイベントコレクタに接続するためのトークン

Fargateのタスク定義内のコンテナ定義にこれらの値を設定しますが、トークンを平文で格納するのはリスクがあります。そのため、みんな大好きSystemsManagerのパラメータストアに、そのトークンをSecureStringとして登録します。

以下のコマンドで、トークンを登録します。パラメータストアの名前は<code>splunk-token</code>。この値は、後手順のログドライバーの設定で利用します。

[bash]
$ aws ssm put-parameter --name "splunk-token" --value "取得したトークンの値" --type "SecureString"
{
    "Version": 1
}
[/bash]

登録内容を確認します。

[bash]
$ aws ssm describe-parameters --filters "Key=Name,Values=splunk-token"
{
    "Parameters": [
        {
            "Name": "splunk-token",
            "Type": "SecureString",
            "KeyId": "alias/aws/ssm",
            "LastModifiedDate": 1556920691.404,
            "LastModifiedUser": "arn:aws:sts::XXXXYYYYZZZZ:assumed-role/cm-hamada.koji/AAAAAAAAAAAAAAAA",
            "Version": 1
        }
    ]
}
[/bash]

無事にパラメータストアに登録された値のメタデータが表示されればOKです。

### Fargateのタスク定義設定

ログドライバー含めたFargateのタスク定義を設定していきます。ここでは、Webコンソールで説明します。とりあえず動作確認のための最小限の設定です。

「新しいタスク定義の作成」→「FARGATE」を選択します。

- タスク定義名：「fargate-splunk-logdriver-sample」
- タスクロール：タスク実行中、ログドライバーのところでパラメータストアにアクセスするために、そのためのポリシーを付与しておきます
	- 参考ページ（<a href="https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/specifying-sensitive-data.html#secrets-iam" target="_blank">機密データの指定 - Amazon Elastic Container Service</a>）
- タスク実行ロール：任意
- タスクメモリ：任意
- コンテナ定義
	- コンテナ名：「splunk-logtest-container」
	- イメージ：「httpd:latest」（apacheの最新版をdockerhubから利用）

ログ設定部分は、以下の通り。

s010

- ログドライバー:splunk
- ログオプション
	- splunk-url:Value:HTTPイベントコレクタのURL（下の注意を参照）
	- splunk-token:ValueFrom:splunk-token（パラメータストアに登録した名前）

<strong>注意</strong>

splunk-urlに指定するURLは、実際のエンドポイントではなく、ドメイン、プロトコル、ポート部分のみです。

- 正しい例
	- https://cloud.splunk.com:8088
- 誤っている例
	- https://cloud.splunk.com:8088/services/collector

curlなどでエンドポイントの確認をするときは下のURLを使用しますが、ログドライバーに設定するURLは上になるのでご注意ください。

上記設定を完了し、無事にタスク定義が登録できれば、前準備は完了です。

### Fargateのタスクを実行し、動作を確認

いよいよ動作確認。上で定義したタスク定義を元にタスク実行します。詳細は省きますが、無事タスクが<code>RUNNING</code>状態になればOk。

s020

タスクの正常起動を確認したら、Splunkのサーチ画面で、ソースにhecの名前を指定して検索してみます。サーチクエリ例は以下の通り。

[bash]
source="http:fargate-logdriver-collector" (index="history" OR index="main" OR index="summary")
[/bash]

こんな感じで、コンテナ内のログが無事イベント検索できたらOkです。いやっほぅ！！

s030

後は、Splunkの世界の話なので、ログを煮るなり役やり好きにしてしまえば良いんじゃないでしょうか。

## ログドライバーにSplunkとCloudWatchLogsのコンソール相違点

ログドライバーにCloudWatchLogsを使用していた場合は、タスク画面に<code>Logs</code>タブがあり、そこでもログの簡易的な検索は可能でした。

s040

対して、ログドライバーにSplunkを使用する場合は、タスク画面にそもそも<code>Logs</code>タブがありません。すべてSplunk側で確認する必要がありますので注意しましょう。

s050


## Fargateのログドライバー拡張の夢が広がるアップデート

ECS（on EC2）では、元々ログドライバーとして、「json-file | syslog | journald | gelf | fluentd | awslogs | splunk」などが用意されています。対して、ECS（on Fargate）では、今までCloudWatchLogs（awslogs）しか対応していませんでした。

慣れたログプラットフォームをFargateでも使いたいという要望は常にあったわけですが、そこに最初にやってきたのが、Splunkというわけです。

今後、fluentdなどの対応も予定されていますので、Fargateのログドライバーの拡張、これからもどんどん広がっていけば、もっと便利にFargateを利用できるシチュエーションが増えそうです。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。
