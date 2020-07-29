# ECSタスクのコンテナ異常終了を検知する3つの方法

「ECSのコンテナが突然異常終了すること、あるよね〜？」

ありますよね。ECSを運用する上では、コンテナの状態を常に把握して、不慮の自体に対応できるようにしておく必要があります。ECSはマネージドサービスの仕組み上、検知する方法も複数あるのですが、ECSタスクの構成パターンによってはその検知にはいくらかの少し作り込みが必要だったりします。

そのパターンを網羅的に解説するのが、このブログの趣旨。3パターン紹介するので、ECS安定運用のためのヒントを掴んで頂ければ幸いでございます。

<pre style="line-height:120%;">
いつもより地味なリード文！！
　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>

## ECSタスクの構成例

ECSタスクですが、その中には複数のコンテナを含ませることができます。今回は3パターンの構成を用意しました。

- 構成①：1タスクで1コンテナ
- 構成②：1タスクで2コンテナ。すべてEssentialがtrue
- 構成③：1タスクで2コンテナ。Essentialがfalseなコンテナが含まれる

### 構成①：1タスクで1コンテナ

01

よくあるパターンですね。1つのタスク中、コンテナが1つの構成です。

### 構成②：1タスクで2コンテナ。essentialがtrue

02

1タスクに2つのコンテナが含まれている構成です。すべてのコンテナの<code>essential</code>設定がtrueになります。

essentialとは、ECSタスク内でそのコンテナが必須かどうかを表します。essentialがtrue設定されているとき、そのコンテナがなんらかの原因で停止した場合はECSタスク全体が終了します。

参考：<a href="https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/task_definition_parameters.html#container_definitions" target="_blank">タスク定義パラメータ：コンテナ定義</a>


### 構成③：1タスクで2コンテナ。essentialがfalseなコンテナが含まれる

03

1タスクに2つのコンテナが含まれている構成です。コンテナのち、一つは<code>essential</code>設定が<code>true</code>でもう一つが<code>false</code>になります。

essentialがfalse設定されているコンテナは、例えそのコンテナが異常終了したとしても、ECSタスク自体の動作に影響を与えません。もう一つのコンテナが稼働している限りは、ECSタスクは生き残ります。

そもそもコンテナをessentail=falseに設定するシチュエーションってなにかって話ですが、多くはいわゆるアプリケーションコンテナに対して補助的に動くSidecarコンテナに設定されます。最近だと、FargateなどのEC2基盤がないコンテナの監視のためにDatadogやMackerelでは専用のSidecarコンテナが提供されていたり、Firelensなどのログコンテナも基本はSidecarでの運用が前提になっています。

<strong>「Sidecarコンテナがなんらかの拍子に異常終了したとしてもアプリケーションコンテナは動作させ続けたい」、という要件のとき、しばしばこういった構成が採用されます。</strong>

## それぞれの構成におけるコンテナ異常終了の検知方法

上記で紹介した3構成において、コンテナの異常終了を検知する代表的な方法を3つ紹介します。

| 停止するコンテナ                                                     | 構成①の<br/>コンテナ | 構成②のコンテナ | 構成③の<br/>essential=trueコンテナ | 構成③の<br/>essential=falseコンテナ |
| :------------------------------------------------------------------- | :------------------- | :-------------- | :--------------------------------- | :---------------------------------- |
| 検知方法①<br />サービスのランニングタスク数をメトリクスから検知      | ◯                    | ◯               | ◯                                  | ☓                                   |
| 検知方法②<br />ECSタスクの状態変更イベントを検知                     | ◯                    | ◯               | ◯                                  | ☓                                   |
| 検知方法③<br />タスクメタデータエンドポイントのContainerStatusを検知 | ◯                    | ◯               | ◯                                  | ◯                                   |

以降、それぞれの検知方法を紹介していきます。

### 検知方法①：サービスのランニングタスク数をメトリクスから検知

一番代表的な方法です。ECSコンテナエージェントは、タスク内のコンテナの状態をモニタリングしています。構成①と②においては、essential=trueのコンテナのみ含まれているので、コンテナの停止は即ECSタスクの停止となります。

あとは、ECSサービスにおけるDesiredTask Count（期待するタスク数）をしきい値としたランニングタスク数のCloudWatchメトリクスを用意しておき、しきい値を下回った時=タスクが異常終了したときにアラームを発火します。

一点、CloudWatch Alarmはある一定期間のメトリクスの状態からアラームを検知するものなので、検知までいくらかのタイムラグが有ることは注意しておきましょう。

基本的にECSのサービス運用においては、そのDesiredTask Countをしきい値としたアラームは、多くの場合そのサービスの健全性を表す代表的な指標となるため、設定しておくことを推奨します。

### 検知方法②：ECSタスクの状態変更イベントを検知

検知方法①よりも、さらに確実かつ迅速にコンテナの生死を検知する方法です。検知方法②と同じく、essential=trueに設定されたコンテナが停止したときは、タスク状態変更イベントが発火され、タスクのステータスが<code>STOPPED</code>に変更されます。

この時、EventBridgeにイベントが発行されるため、それをトリガーに通知する方法です。詳細はこちら。

<a href="https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/ecs_cwe_events.html#ecs_task_events" target="_blank">Amazon ECS イベント：タスク状態変更イベント</a>

このイベントから、タスクが停止したイベントをトリガーするイベントルールを事前に登録しておくことで、コンテナ停止→タスク停止からの検知を素早く行うことができます。

ここも、公式チュートリアルで丁寧に解説されているので、参考にしてみましょう！

<a href="https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/ecs_cwet2.html" target="_blank">チュートリアル: タスク停止時のイベントに関する Amazon Simple Notification Service アラートを送信する - Amazon Elastic Container Service</a>

### 検知方法③：タスクメタデータエンドポイントのContainerStatusを検知

構成③において、essential=falseに設定されたコンテナの異常終了検知は一筋縄ではいきません。なぜなら、essential=falseに設定されたコンテナは、それが終了してもそのコンテナが含まれるECSタスクのステータスに影響を与えないからです。まぁそういう設定ですからね、

ただ、コンテナが停止していること自体は異常な状態なので、それを検知する方法を考えてみたところ、ECSのタスクメタデータエンドポイントを利用するのが一番簡単そうでした。

<a href="https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/task-metadata-endpoint-v4.html" target="_blank">タスクメタデータエンドポイントバージョン 4 - Amazon Elastic Container Service</a>

タスクメタデータエンドポイントは、ECS内タスク内部のコンテナからHTTPリクエストを投げることにより、そのタスクのメタデータを取得できるエンドポイントです。その中の、<code>${ECS_CONTAINER_METADATA_URI_V4}/task</code>に、そのタスクに含まれる全コンテナの状態と終了コードが、<code>KnownStatus</code>と<code>ExitCode</code>に含まれているため、これを監視します。

ただ、essential=falseのコンテナが終了した場合、ECSコンテナエージェント側ではイベントが発生しないため、この監視は定期実行が必要になります。この定期実行の仕組み自体をコンテナ内に実装するのは、実装や運用が複雑になるため、あまりオススメはしません。

一つの方法としては、ALBからのヘルスチェックを受けているコンテナ内で、ヘルスチェックのエンドポイントにリクエストがあったときに、毎回、このメタデータを監視しておき、状態が<code>STOPPED</code>になっているコンテナがあったときは、エラーを通知するなどの実装が、定期処理をコンテナ内に組み込む必要がないので良いかと思います。

## 実際にタスクメタデータエンドポイントでコンテナの状態を検知できるかやってみた

ホンマにコンテナの状態が、このメタデータに反映されるのか不安だったので、実際に試してみました。タスクメタデータエンドポイントでどんなデータが取れるか合わせて紹介します。

### タスク定義の用意

タスク定義をJSONで用意します。今回は、コンテナ内に<code>docker exec</code>で入りたいため、EC2モードでECSタスクを起動します。このタスク内には、nginxとapacheのコンテナを2つ起動し、それぞれを8080ポートと8081ポートでコンテナと通信させます。また、肝心なessentialについては、nginxを<code>true</code>とし、apacheを<code>false</code>とします。

<code>executionRoleArn</code>と、<code>taskRoleArn</code>には、各環境で定義されたIAMロールを参照するように変更してください。


[json title="sample-task-def.json"]
{
  "containerDefinitions": [
    {
      "portMappings": [
        {
          "hostPort": 8080,
          "protocol": "tcp",
          "containerPort": 80
        }
      ],
      "cpu": 0,
      "environment": [],
      "mountPoints": [],
      "memory": 128,
      "volumesFrom": [],
      "image": "nginx",
      "essential": true,
      "name": "nginx-container"
    },
    {
      "portMappings": [
        {
          "hostPort": 8081,
          "protocol": "tcp",
          "containerPort": 80
        }
      ],
      "cpu": 0,
      "environment": [],
      "mountPoints": [],
      "memory": 128,
      "volumesFrom": [],
      "image": "httpd",
      "essential": false,
      "name": "apache-container"
    }
  ],
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "requiresCompatibilities": [
    "EC2"
  ],
  "networkMode": "bridge",
  "inferenceAccelerators": [],
  "volumes": [],
  "placementConstraints": []
}
[/json]

jsonが用意できたら、以下のCLIコマンドでタスク定義を登録します。無事に登録できましたでしょうか？

[bash]
$ aws ecs register-task-definition --family sample-task-def --cli-input-json file://sample-task-def.json
[/bash]


### ECSタスクの起動

起動元のEC2にログインした状態で<code>run-task</code>を実行します。クラスターは適当なものを用意しておいてください。

[bash]
aws ecs run-task --cluster ecs-container-test --task-definition sample-task-def
[/bash]

タスク状態を確認し、nginxとhttpdのコンテナが起動していたら、準備OKです。

040

### apacheコンテナの停止

2つ起動したコンテナのうち、apacheコンテナを<code>docker stop</code>コマンドで停止します。事前に<code>docker container ls</code>で、起動コンテナの名前を取得。

[bash]
$ docker container ls
CONTAINER ID        IMAGE                            COMMAND                  CREATED             STATUS                PORTS                  NAMES
08f2d941ecaa        nginx                            "/docker-entrypoint.…"   42 seconds ago      Up 40 seconds         0.0.0.0:8080->80/tcp   ecs-sample-task-def-3-nginx-container-8c96b5e5f2a8eafa8f01
4e3f4cc0a462        httpd                            "httpd-foreground"       42 seconds ago      Up 40 seconds         0.0.0.0:8081->80/tcp   ecs-sample-task-def-3-apache-container-ecc6918dd0e3ed9efc01
58ba84c67540        amazon/amazon-ecs-agent:latest   "/agent"                 4 days ago          Up 4 days (healthy)                          ecs-agent
[/bash]

コンテナを停止します。

[bash]
$ docker container stop ecs-sample-task-def-3-apache-container-ecc6918dd0e3ed9efc01
ecs-sample-task-def-3-apache-container-ecc6918dd0e3ed9efc01
[/bash]

タスクを確認してみると、apacheコンテナは停止していますが、nginxコンテナは起動している状態が実現できました。

05

### タスクメタデータエンドポイントを確認

最後に、コンテナに入って、タスクメタデータエンドポイントにアクセスします。

コンテナに<code>docker exec</code>でbashを利用して入り、curlをインストール後、メタデータエンドポイントの環境変数を参照します。

[bash]
$ docker container exec -it ecs-sample-task-def-3-nginx-container-8c96b5e5f2a8eafa8f01 bash
root@08f2d941ecaa:/# apt-get -y install curl
root@08f2d941ecaa:/# echo ${ECS_CONTAINER_METADATA_URI_V4}/task
http://169.254.170.2/v4/fd9cecec-dd87-4748-8e00-3cb65d1b417c/task
[/bash]

メタデータエンドポイント<code>${ECS_CONTAINER_METADATA_URI_V4}/task</code>にcurlを投げてメタデータを取得します。

[json highlight="44-46"]
root@08f2d941ecaa:/# curl ${ECS_CONTAINER_METADATA_URI_V4}/task
{
    "Cluster": "ecs-container-test",
    "TaskARN": "arn:aws:ecs:ap-northeast-1:629895769338:task/7c793616-a20f-4be6-b55d-fb7ada7538cb",
    "Family": "sample-task-def",
    "Revision": "3",
    "DesiredStatus": "RUNNING",
    "KnownStatus": "RUNNING",
    "PullStartedAt": "2020-07-21T14:36:20.075263568Z",
    "PullStoppedAt": "2020-07-21T14:36:22.914895058Z",
    "AvailabilityZone": "ap-northeast-1c",
    "Containers": [
        {
            "DockerId": "4e3f4cc0a462c3db0185991508aba62badf879b2c930a9cda8cf4c2f93bab708",
            "Name": "apache-container",
            "DockerName": "ecs-sample-task-def-3-apache-container-ecc6918dd0e3ed9efc01",
            "Image": "httpd",
            "ImageID": "sha256:ccbcea8a67570043de0d0932f9d750e7d311415def699c60aa69e4cea4a25a7e",
            "Ports": [
                {
                    "ContainerPort": 80,
                    "Protocol": "tcp",
                    "HostPort": 8081
                }
            ],
            "Labels": {
                "com.amazonaws.ecs.cluster": "ecs-container-test",
                "com.amazonaws.ecs.container-name": "apache-container",
                "com.amazonaws.ecs.task-arn": "arn:aws:ecs:ap-northeast-1:629895769338:task/7c793616-a20f-4be6-b55d-fb7ada7538cb",
                "com.amazonaws.ecs.task-definition-family": "sample-task-def",
                "com.amazonaws.ecs.task-definition-version": "3"
            },
            "DesiredStatus": "RUNNING",
            "KnownStatus": "STOPPED",
            "ExitCode": 0,
            "Limits": {
                "CPU": 0,
                "Memory": 128
            },
            "CreatedAt": "2020-07-21T14:36:22.870838409Z",
            "StartedAt": "2020-07-21T14:36:23.921961421Z",
            "FinishedAt": "2020-07-21T14:37:29.628173826Z",
            "Type": "NORMAL",
            "Networks": [
                {
                    "NetworkMode": "bridge",
                    "IPv4Addresses": [
                        ""
                    ]
                }
            ]
        },
        {
            "DockerId": "08f2d941ecaad0e600f5a913af8f8d2dfee802ed09101903357dee8584aeff66",
            "Name": "nginx-container",
            "DockerName": "ecs-sample-task-def-3-nginx-container-8c96b5e5f2a8eafa8f01",
            "Image": "nginx",
            "ImageID": "sha256:0901fa9da894a8e9de5cb26d6749eaffb67b373dc1ff8a26c46b23b1175c913a",
            "Ports": [
                {
                    "ContainerPort": 80,
                    "Protocol": "tcp",
                    "HostPort": 8080
                }
            ],
            "Labels": {
                "com.amazonaws.ecs.cluster": "ecs-container-test",
                "com.amazonaws.ecs.container-name": "nginx-container",
                "com.amazonaws.ecs.task-arn": "arn:aws:ecs:ap-northeast-1:629895769338:task/7c793616-a20f-4be6-b55d-fb7ada7538cb",
                "com.amazonaws.ecs.task-definition-family": "sample-task-def",
                "com.amazonaws.ecs.task-definition-version": "3"
            },
            "DesiredStatus": "RUNNING",
            "KnownStatus": "RUNNING",
            "Limits": {
                "CPU": 0,
                "Memory": 128
            },
            "CreatedAt": "2020-07-21T14:36:22.924486464Z",
            "StartedAt": "2020-07-21T14:36:23.956366222Z",
            "Type": "NORMAL",
            "Networks": [
                {
                    "NetworkMode": "bridge",
                    "IPv4Addresses": [
                        "172.17.0.3"
                    ]
                }
            ]
        }
    ]
}
[/json]

タスクメタデータには様々な情報が格納されていますが、今回の注目ポイントは、apacheコンテナ側の以下の部分。KnownStatusが<code>STOPPED</code>になっており、起動しているnginxコンテナからタスクメタデータエンドポイント経由で、essential=falseなコンテナの起動状態を確認することができました。お疲れ様です！！

[json]
            "DesiredStatus": "RUNNING",
            "KnownStatus": "STOPPED",
            "ExitCode": 0,
[/json]

今回、<code>docker stop</code>を使ってコンテナを終了させているためか、<code>ExitCode</code>が0になっていますが、コンテナが何かしら異常終了した場合は、この<code>ExitCode</code>には何かしらの異常終了コードが格納されると思うので、エラーハンドリングする際は、ぜひそのあたりの関連情報を収集し、イベント通知することを推奨します。

## コンテナの異常終了を検知するのはECS運用の安定運用に不可欠

ECSサービスの機能を使えば、基本的にはタスクなどが異常終了したときも、既存のタスク定義を使って、自動的にDesiredのタスク数までタスクを起動してくれます。それによりアプリケーションの可用性は担保できる場合もありますが、異常終了したという事実は必ず把握しておき、できるだけ早く原因追求をしておくべきでしょう。

ECSサービスのレジリエンスに頼り切った運用では、コンテナの異常の兆候に気づかずある日大トラブルを起こすということにも繋がりかねません。転ばぬ先の杖ではないですが、コンテナのちょっとした異常も検知できるようにしておき、ECSの安定的な運用ができる体制を整えておくのが、サービス運用の肝になります。

そのための手段を何点かご紹介いたしました。このブログが皆さんの現場でお役に立てば幸いです。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

以上、
