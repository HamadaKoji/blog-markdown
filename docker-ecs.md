# DockerとAWSのコラボによりdocker ecsコマンドが爆誕したので使ってみた

<strong>「docker ecsコマンド？なにこれ？」</strong>

先日、突如、DockerのECSインテグレーションなるものが発表されました！

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://aws.amazon.com/jp/blogs/containers/aws-docker-collaborate-simplify-developer-experience/" target="_blank">AWS and Docker collaborate to simplify the developer experience | Containers</a>
</p>

従来あるdockerコマンドに、なんと<code>docker ecs</code>コマンドが追加され、docker-composeファイルを利用したECSへのデプロイがAWS CLIなどのAWS製ツールを使わずに、全てdockerコマンドだけで完結するという、ちょっと想像がつかないアップデートです。

まだDocker社ではベータ版の扱いということですが、なかなかにおもしろいアプローチだったので、基本的な操作を一通り確認してみた様子をお届けします。

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     dockerとecsﾏﾂﾘﾀﾞﾜｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>


## dockerとAWSのインテグレーションとは

概要は、dockerの公式ブログを参照。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://www.docker.com/blog/from-docker-straight-to-aws/" target="_blank">From Docker Straight to AWS - Docker Blog</a>
</p>

複数のコンテナを協調動作させつつデプロイし、環境をセットアップするための、docker composeは広く使われています。今回のDocker社とAWSとのパートナーシップにより、dockerに、<code>docker ecs</code>コマンドが追加され、ECSやFargateに統合さることになりました。7月20日時点では、ベータ版での提供となります。

動画はこちら。たった30秒ほどの動画ですが、何ができるのか直感的にわかるので、最初に見ていただくことをオススメします。

<iframe width="560" height="315" src="https://www.youtube.com/embed/UE162PzO2s0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## dockerのECS Integrationを動かしてみる！

正直、話を聞いていてもなかなかイメージが沸かないので（自分も）、Docker社でホストされているGitHubを参考に実際に動かしてみた様子をお届けします。基本的な流れは、すべてGitHubにアップされているものを利用しています。ただ、一部情報を保管するために、Dockerの公式ドキュメントも合わせて参照しています。

- Github
  - <a href="https://github.com/docker/ecs-plugin" target="_blank">docker/ecs-plugin: CLI plugin for using Fargate/ECS with Docker CLI</a>
- Docker Document
  - <a href="https://docs.docker.com/engine/context/ecs-integration/" target="_blank">Deploying Docker containers on ECS | Docker Documentation</a>

手順は以下の通り。

- Docker Desktop Edgeのインストール
- プライベートDocker Hubリポジトリアクセスのためのクレデンシャル格納
- docker contestの用意
- サンプルアプリケーションのローカル動作確認
- ECSで利用するイメージをDockerHubにプッシュ
- ECSへのデプロイ
- デプロイされたECSの構造を理解する
- docker ecs compose関連コマンド
- デプロイリソースの削除




## Docker Desktop Edgeのインストール

前提として、Docker Desktop Edgeが必要。EdgeはStableの安定版とは違うため、通常の利用ではインストールしている人は少ないと思います。自分の環境はmacOSなので、公式ページの「Get Edge」ボタンより、ダウンロードしてインストールしておきます。

<a href="https://hub.docker.com/editions/community/docker-ce-desktop-mac" target="_blank">Docker Desktop for Mac - Docker Hub</a>

Edge版インストールするときは、「クライアントのコンテナやイメージをバッサリ削除するで！」と注意書きがでます。docker使って現在バリバリ開発中の方は、もしかしたら痛い目にあうかもなので、ある程度覚悟してインストールしましょう。

インストールして起動すると、Channel部分にedgeと表示されます。

010

インストール完了したら、次にいってみましょ！

## プライベートDocker Hubリポジトリアクセスのためのクレデンシャルを格納

事前に自身のDockrHubアカウントにアクセストークンをセットアップする必要があります。まだアクセストークンを取得していない場合は、こちら（<a href="https://docs.docker.com/docker-hub/access-tokens/" target="_blank">Managing access tokens | Docker Documentation</a>）を参考にトークンを作成しておきます。


## docker contextの用意

dockerコマンドを利用してAWSに接続するためのContextをセットアプします。コマンドは<code>docker ecs setup</code>で、これを入力し、awsにアクセスするprofile名とregion内容をセットアップします。すでにAWS CLIなどでAWS環境に接続できる環境になっているのであれば、そのAWS Profileを選択することで設定が完了します。

下の例では、Profileに<code>docker-ecs-user</code>を選択しています。既存Profileを選択するのであれば、別途Credentialsの入力は不要です。

[bash]
$ docker ecs setup
Enter context name: aws█
✔ docker-ecs-user
? Enter region: ap-northeast-1
? Enter credentials? [y/N] N█
[/bash]

すべて正常に完了したら、<code>docker context ls</code>コマンドで、Docker contextsが正常にセットアップされているか確認します。<code>NAME</code>に、<code>aws</code>が表示されていればOK。

[bash]
$ docker context ls
NAME                TYPE                DESCRIPTION                               DOCKER ENDPOINT               KUBERNETES ENDPOINT   ORCHESTRATOR
aws                 aws                                                                                                               
default *           moby                Current DOCKER_HOST based configuration   unix:///var/run/docker.sock                         swarm
[/bash]

### docker ecs setup時のエラーについて

自分の環境では、<code>docker ecs setup</code>実行時、以下のエラーがでてました。

[bash]
this tool requires the "new ARN resource ID format"
[/bash]

こちら、GitHubにISSUEが上がってます。

<a href="https://github.com/docker/ecs-plugin/issues/175" target="_blank">Receiving a &apos;this tool requires the &quot;new ARN resource ID format&quot;&apos; error on setup in Linux · Issue #175 · docker/ecs-plugin</a>

エラー解決方法は、ECSのアカウント設定でサービス名の新しいARN形式のオプトインが必要になります。設定手順のマニュアルはこちら。

<a href="https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/userguide/ecs-account-settings.html" target="_blank">アカウント設定 - Amazon ECS</a>

確認方法を簡単に紹介。以下のコマンドでECSの各リソースのオプトイン状況が確認できます。<code>value</code>が<code>disabled</code>になっている値は、オプトインされていません。

[bash]
$ aws ecs list-account-settings --effective-settings
{
    "settings": [
        {
            "name": "awsvpcTrunking",
            "value": "disabled",
            "principalArn": "arn:aws:iam::629895769338:user/docker-ecs-user"
        },
        {
            "name": "containerInsights",
            "value": "disabled",
            "principalArn": "arn:aws:iam::629895769338:user/docker-ecs-user"
        },
        {
            "name": "containerInstanceLongArnFormat",
            "value": "disabled",
            "principalArn": "arn:aws:iam::629895769338:user/docker-ecs-user"
        },
        {
            "name": "serviceLongArnFormat",
            "value": "disabled",
            "principalArn": "arn:aws:iam::629895769338:root"
        },
        {
            "name": "taskLongArnFormat",
            "value": "disabled",
            "principalArn": "arn:aws:iam::629895769338:user/docker-ecs-user"
        }
    ]
}[/bash]

この際だから全てオプトインする場合は、それぞれのサービスに対して、オプトインを実施していきます。

[bash]
aws ecs put-account-setting-default --name awsvpcTrunking --value enabled
aws ecs put-account-setting-default --name containerInsights --value enabled
aws ecs put-account-setting-default --name containerInstanceLongArnFormat --value enabled
aws ecs put-account-setting-default --name serviceLongArnFormat --value enabled
aws ecs put-account-setting-default --name taskLongArnFormat --value enabled
[/bash]

再度設定確認し、それぞれの<code>value</code>が<code>enabled</code>になっていることを確認しましょう。設定が終わったら、再度<code>docker ecs setup</code>を実行してみてください。

このエラーについては、toriの声が天から降ってきたことをご報告しておきます。いつもありがとうございます！

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">ECS の long ARN format 有効にしてみるとどうですか？</p>&mdash; ポジティブな Tori (@toricls) <a href="https://twitter.com/toricls/status/1285951337423712256?ref_src=twsrc%5Etfw">July 22, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


## サンプルアプリケーションのローカル動作確認

サンプルアプリケーションをローカルで動作確認します。

[bash]
$ git clone https://github.com/docker/ecs-plugin.git
$ cd ecs-plugin/example/
[/bash]

<code>docker context</code>を<code>default</code>に切替。

[bash]
$ docker context use default
default
[/bash]

サンプルアプリケーションをローカルで起動し、画面が起動することを確認します。

[bash]
docker-compose up
[/bash]

ブラウザで<code>http://localhost:5000/</code>にアクセスし、下記のような画面が表示されたらOK.

020

## ECSで利用するイメージをDocker Hubにプッシュ

前提として、dockerコマンドを利用したECSのデプロイで使うイメージはDocker Hubの自分のアカウントにアップロードします。<strong>通常、ECSでは利用イメージをECRに置くことが多いですが、この場合はECRではないことにご注意ください。</strong>

プッシュ前に、<code>docker-compose.yml</code>の以下の部分を書き換えます。

[yaml title="docker-compose.yml"]
    image: <<<your docker hub user name>>>/timestamper
[/yaml]

プッシュ先のイメージのURIを指定するため、自分のDockr HubアカウントIDを埋めてください。自分の場合は、以下となります。

[yaml title="docker-compose.yml"]
    image: hamako9999/timestamper
[/yaml]

自分のDocker Hubアカウントに、前手順で作成したパーソナルアクセストークンを利用して、ログインします。

[bash]
$ docker login --username hamako9999
Password: 
Login Succeeded
[/bash]

ログイン成功したら、<code>docker-compose push</code>コマンドでイメージをプッシュします。

[bash]
$ docker-compose push
Pushing frontend (hamako9999/timestamper:latest)...
The push refers to repository [docker.io/hamako9999/timestamper]
c94062adef57: Pushed
dce45b6ec708: Pushed
8209fa3a4eda: Pushed
f7edcfe22288: Pushed
5afde73d5bad: Pushed
b10be96d1b4e: Layer already exists
003d0b48eda4: Pushed
408e53c5e3b2: Pushed
50644c29ef5a: Pushed
latest: digest: sha256:07066276826c22d9bbbbae05dc21eb48b39c0a5d9b45410bcb2cda8d565ea438 size: 2201
[/bash]

自身のDocker Hubアカウントにアクセスしてみてください。このように<code>timestamper</code>イメージがプッシュされていればOKです。

030


## ECSへのデプロイ

DockerのECSインテグレーションでは、プライベートイメージをECRに自動的に格納してくれます。そのため、AWSからDocker HubにアクセスするためのアクセストークンをAWS環境上にセットアップしておく必要があります。

このあたり、便利コマンドが用意されていて、AWS CLIを使わずにdockerコマンドを利用して、AWS上にアクセストークンを格納してくれます。

[bash]
$ docker ecs secret create dockerhubAccessToken --username hamako9999 --password --password <dockerhubtoken>
arn:aws:secretsmanager:ap-northeast-1:123456789012:secret:dockerhubAccessToken-qftyOp
[/bash]

AWSのSecrets Managerにアクセスすると、シークレットが格納されていることが確認できます。

040

この作成されたトークンのARNを<code>docker-compose.yml</code>の<code>x-aws-pull_credentials</code>に記入します。

[yaml title="docker-compose.yml"]
    x-aws-pull_credentials: <<<your arn for your secret you can get with docker ecs secret list>>>
[/yaml]

これで、ECSアップロードのための準備は完了ですよ！

### コンテキストをAWSに切り替えてアプリケーションの起動

前手順で作成したAWSアクセス用のContextに切り替えます。

[bash]
docker context use aws
[/bash]

いよいよ、AWSにアプリケーションをデプロイします。コマンドは単純至極。作られるリソースにLoadBalancerなどもあるため、作成完了まで5分程度必要になります。

[bash]
docker ecs compose up
[/bash]

無事すべての処理が正常に完了したでしょうか？まだベータ版のため、ちょくちょくエラーが発生すると思いますが頑張ってください。自分の場合は、一度Secrets ManagerのARNが誤っていて、修正後再デプロイしようとすると、CloudMapで作成したホストゾーンが名前重複して再作成できませんでした。

このあたりは、ECSと周辺サービスの構造をある程度理解していないと対処が難しいと思うので、それなりの覚悟が必要です。

## デプロイされたECSの構造を理解する

上記コマンドで、docker-compose.ymlに記載されている内容をAWSに展開するためにCloudFormationが展開され各種AWSリソースがガッツリ作成されます。CloudFormationのコンソールには、<code>app</code>スタックが作成されており、含まれているAWSリソースはこのあたり。

- LogGroup
- FrontendTaskExecutionRole
- FrontendTaskDefinition
- FrontendTCP5000TargetGroup
- FrontendTCP5000Listener
- FrontendServiceDiscoveryEntry
- FrontendService
- Cluster
- CloudMap
- BackendTaskExecutionRole
- BackendTaskDefinition
- BackendServiceDiscoveryEntry
- BackendService
- AppLoadBalancer
- AppDefaultNetworkIngress
- AppdefaultNetwork

構成図で表すとこうなります。

050

VPCはdefault-VPCが利用され、ECSクラスターは新規で作成されます。FrontEndコンテナとBackendコンテナがそれぞれFargateで起動します。EC2は作成されません。

LoadBalancerがNLBで作成され、外部通信はdocker-composeファイルに記載の通り、ポート5000で受け、ポート5000で受けるリスナーが作成され、ターゲットグループにFrontEndのECSタスクが登録されます。このポート5000を受けるためのセキュリティグループも自動的に設定されます。

FrontEndコンテナとBackendコンテナの通信はサービスディカバリー経由となるため、VPCに紐付いたプライベートゾーンがCloudMap経由で作成され、ECSタスクがCloudMapの各サービスに紐づきます。

FrontendのECSタスク定義は、イメージをDocker Hubの自分のアカウントから取得する用に設定されています。そのときに必要な認証情報のパラメーターは、予め作成しておいたSecretsManagerのアクセストークンを利用するように認証情報パラメーターとして指定されます。この状態ではECRは作成されていません。

ECS関連コンポーネントについて説明しました。イメージ湧きますでしょうか？FrontendコンテナとBackendコンテナ間の通信をCloudMapによるサービスディスカバリーで実現しているのは、インターナルなロードバランサーを使うよりコストでも構築スピードでもメリットがあって良いですね。

実際にデプロイされたコンテナの動作確認には、作成されたNLBのエンドポイントにポート5000でアクセスしてみてください。

## docker ecs compose関連コマンドを理解する

docker ecs compose コマンドには他にもいくつかコマンドがあります。

[bash]
$ docker ecs compose --help

Usage:  docker ecs compose [OPTIONS] COMMAND

Options:
  -f, --file stringArray      Specify an alternate compose file
  -n, --project-name string   Specify an alternate project name (default: directory name)

Commands:
  convert     
  down        
  logs        
  ps          
  up 
[/bash]

<code>docker ecs compose ps</code>で、起動しているコンテナの状態を確認できます。

[bash]
$ docker ecs compose ps
ID                                     NAME                REPLICAS            PORTS
example-BackendService-LKYCYBC7UQS9    backend             1/1                 
example-FrontendService-LKMZSMS7EENO   frontend            1/1   
[/bash]

<code>docker ecs compose convert</code>で、AWSリソースの作成に利用されたCloudFormationテンプレートを出力できます。JSONなのが辛いですが。

<code>docker ecs compose logs</code>で、コンテナ内ログをtail状態でターミナルに出力できます。

[bash]
$ docker ecs compose logs
backend    | 1:C 23 Jul 2020 01:57:56.971 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
backend    | 1:C 23 Jul 2020 01:57:56.971 # Redis version=6.0.6, bits=64, commit=00000000, modified=0, pid=1, just started
backend    | 1:C 23 Jul 2020 01:57:56.971 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
backend    | 1:M 23 Jul 2020 01:57:56.973 # You requested maxclients of 10000 requiring at least 10032 max file descriptors.
backend    | 1:M 23 Jul 2020 01:57:56.973 # Server can't set maximum open files to 10032 because of OS error: Operation not permitted.
backend    | 1:M 23 Jul 2020 01:57:56.973 # Current maximum open files is 4096. maxclients has been reduced to 4064 to compensate for low ulimit. If you need higher maxclients increase 'ulimit -n'.
backend    | 1:M 23 Jul 2020 01:57:56.974 * Running mode=standalone, port=6379.
backend    | 1:M 23 Jul 2020 01:57:56.974 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
backend    | 1:M 23 Jul 2020 01:57:56.974 # Server initialized
backend    | 1:M 23 Jul 2020 01:57:56.974 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
backend    | 1:M 23 Jul 2020 01:57:56.974 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
backend    | 1:M 23 Jul 2020 01:57:56.974 * Ready to accept connections
frontend    |  * Serving Flask app "app" (lazy loading)
frontend    |  * Environment: production
frontend    |    WARNING: This is a development server. Do not use it in a production deployment.
frontend    |    Use a production WSGI server instead.
frontend    |  * Debug mode: on
frontend    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
frontend    |  * Restarting with stat
frontend    |  * Debugger is active!
frontend    |  * Debugger PIN: 206-344-428
[/bash]

## デプロイリソースの削除

ここは、docker-composeのお作法と同じですね。<code>docker ecs compose down</code>で、作成されたリソースはすべて削除されます。

[bash]
docker ecs compose down
[/bash]

このコマンドでどうしても削除できない場合は、マネジメントコンソールからCloudFormationを開いて、関連リソースを順次削除していきましょう。

## docker-composeを利用している環境を、ECS上で動作確認をするのに非常に便利

以上、長くなりましたが、docker-composeを利用したECSインテグレーションについて説明してきました。データベースをコンテナで動かしている場合でも、Fargateのストレージサイズで足りる容量であれば、すんなりECSにデプロイできるかと思います。

基本的には、既存のdocker-composeファイルにDocker Hubアクセス用のトークンが格納されたSecrets ManagerのARNを挿入するだけで、ECSデプロイされるのは革命的に便利なのではないでしょうか。また、別オプションを使えば、既存のECSクラスターやVPCやロードバランサーを利用してデプロイできたり、イメージをECRから取得するように変更可能です。

そのあたりの追加のコマンドや、プラグインについては、Dockerの公式ドキュメントに記載されているので、合わせて確認してみてください。

参考：<a href="https://docs.docker.com/engine/context/ecs-integration/#tuning-the-cloudformation-template" target="_blank">Tuning the CloudFormation template | Docker Documentation</a>


