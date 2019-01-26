（仮）マネージドな仕組みだけで、ECSがBlue/Greenデプロイメントに対応しました


「」






<a href="https://aws.amazon.com/jp/blogs/devops/use-aws-codedeploy-to-implement-blue-green-deployments-for-aws-fargate-and-amazon-ecs/" target="_blank">Use AWS CodeDeploy to Implement Blue/Green Deployments for AWS Fargate and Amazon ECS | AWS DevOps Blog</a>




## 今まで辛かったこと

今までECSおデプロイは、基本的にローリングデプロイでした。一番主要な方法として、ECSのサービスのタスク定義を新しいバージョンに変更し、新しいデプロイを強制することで、新しいコンテナがサービス内で順次切り替わっていくものです。

最小ヘルス率と最大ヘルス率のパラメータを調整することで、入れ替わりの方式をある程度制御することは可能でしたが、新しいコンテナになにかしら不具合が見つかった時の切り戻しには同じ手順をふむ必要があり手間がかかるものでした。


## 今回新しく便利になった点

CodeDeployを利用して、ECSに対してのブルーグリーンデプロイが可能になっています。


==========補足==========

こちらをみると、ECSのデプロイタイプにローリングアップデートに加えて、CodeDeployによるｙブルーグリーンデプロイが追加されています。

* <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-types.html" target="_blank">Amazon ECS Deployment Types - Amazon Elastic Container Service</a>




## この記事の前提条件

この記事では、CodeDeployを利用したAWS Fargate for ECSへのブルー・グリーンデプロイを説明していきます。

  * DockerイメージリポジトリにDockerfileからビルドされたイメージが格納されていること
  * ECSのデフォルトクラスターがあること。もしくは、ネットワーキングのみのクラスターが存在していること

また、イメージリポジトリとECSクラスターは同一リージョンで構築しておく必要があります。

また読者の対象としては、あるていどECSを既に経験されている方を対象としているため、Webコンソール上の詳細な動作説明は省略していること、ご了承ください。

## ECSサービス作成まえの事前準備

### IAMサービスロールの作成

CodeDeployが、ECSのデプロイをハンドリングするために、CodeDeployにECS、ELB、Lambda、CloudWatch alarmsへのアクセス権限が必要となります。先にCodeDeployのIAMロールを作成しておきます。

こちらを参考に、事前に設定しておいてください。

* <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/codedeploy_IAM_role.html" target="_blank">Amazon ECS CodeDeploy IAM Role - Amazon Elastic Container Service</a>

### ALBの作成

CodeDeployとECSが複数バージョンのECSサービスのトラフィックを制御するためのALBを作成します。

基本的な流れは、こちらを参考に。

* <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-application-load-balancer.html" target="_blank">Creating an Application Load Balancer - Amazon Elastic Container Service</a>

差分だけ説明していきます。

* ロードバランサー名を「sample-website-alb」
* セキュリティグループの設定を以下の通り
  * セキュリティグループ名を「sample-website-sg」
  * ルールにTCP port 80 from anywhere (0.0.0.0/0)を追加
  * ルールにTCP port 8080 from anywhere (0.0.0.0/0)を追加
* ルーティング設定を以下の通り
  * ターゲットグループ名を「sample-website-tg-1」
  * ターゲットタイプを「IP」
* 「Create a Security Group Rule for Your Container Instances」はスキップ

これで、ALBの作成は完了です。

### タスク定義の作成

以下のJSONを参考に、タスク定義を作成します。

[json]
{
  "executionRoleArn": "arn:aws:iam::account_ID:role/ecsTaskExecutionRole",
  "containerDefinitions": [{
    "name": "sample-website",
    "image": "<YOUR ECR REPOSITORY URI>",
    "essential": true,
    "portMappings": [{
      "hostPort": 80,
      "protocol": "tcp",
      "containerPort": 80
    }]
  }],
  "requiresCompatibilities": [
    "FARGATE"
  ],
  "networkMode": "awsvpc",
  "cpu": "256",
  "memory": "512",
  "family": "sample-website"
}
[/json]

上のJsonから、タスク定義で利用するIAMロールのARNと、ECRリポジトリのURIを編集するのを忘れないでください。

## ブルー/グリーンデプロイ対応のECSサービス作成

さぁさぁ、前作業は終了。これからいよいよブルーグリーンデプロイメント対応のECSを作っていきます。

### ECSサービスの作成

ECSデフォルトクラスターを選択し、サービスの作成をクリック。

020

サービスを設定していきます。

* 起動タイプは「Fargate」
* タスク定義は「sample-website」
* サービス名は「Sample-Website」
* タスクの数は「1」

030

今までなかったデプロイ方式が表示されています！！

* デプロイ方式で「Blue/green deployment (powered by AWS CodeDeploy)」
* サービスロール名に事前準備で作成した「ecsCodeDeployRole」

040

ネットワーク構成を設定します。

* クラスターVPC「DefaultVPC」
* サブネットに1aと1cサブネットを追加
* セキュリティグループに前段で作成した「sample-website-sg」
* パブリックIPの割当は「ENABLED」

050

いよいよ本番のALB周辺の設定です。

* ELBタイプで「Application Load Balancer」を選択
* ELB名で「sample-website-alb」を指定
* ELBへの追加をクリック
  * リスナーポートに80:HTTPを選択
  * Test listener portに新規作成で「8080」を指定
  * Test listener protocolに「HTTP」を指定

060

Additional configurationで以下を設定。

* Target group 1 name に「sample-website-tg-1」を選択
* Target group 2 name に「sample-website-tg-2」を入力

070

Auto Naming Serviceはここでは使わないのでオフにしておきます。

そのまま続けていき、サービスの作成が完了したらOkです。無事、以下のCodeDeploy for blue/green deploymentsが表示されましたでしょうか！？

080

ECSサービスのタスクを確認し、ALBのDNSにアクセスして、無事意図したコンテナが起動していることを確認します。とりあえず自分は、素のnginxコンテナを利用しています。

090

## 自動作成されたCodeDeployの詳細確認

CodeDeployコンソールにアクセスすると、自動的にECS用のCodeDeployアプリケーションが作成されています！なんじゃこりゃ！

100

デプロイメントグループには、デプロイタイプとして<span class="tt">Blue/Green</span>が、デプロイコンフィグに<span class="tt">CodeDeployDefault.ECSAllAtOnce</span>が設定されています。これは、ヘルスチェックが通ると、ALBはそのトラフィックを全てグリーン環境に流すことを意味します。

110

ロードバランサー設定を確認すると、デプロイメントグループで利用されるリスナーとターゲットグループの詳細を確認できます。

120

ほな、いよいよ本番、CodeDeployを利用した、ECSのブルーグリーンデプロイをやってみましょう！

## ECSサービスへのCodeDeployを利用したブルーグリーンデプロイの実行

まずは、タスク定義を更新します。なんでも良いので前段で作成したsample-websiteの新バージョンを作成します。ここでは、タスク定義一番下の所で、タグに以下を設定しています。

* Key:Name
* Value:Sample Website

125

その後、サービスを更新します。新しく定義したタスク定義（Version2）を設定。「新しいデプロイの強制」もチェックを入れておきましょう。

130

それ以外はそのままにしてサービスを更新します。すると、ECSサービスのデプロイタブに、Blue/Greenデプロイメントが表示されます！すげぇ！こんなの初めて！

140

DeploymentIDをクリックすると、CodeDeployを利用の方は見慣れたデプロイステータスとトラフィック移行の進行状況が表示されます。





## 参考資料

公式ドキュメント

* <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-types.html#deployment-type-bluegreen" target="_blank">Amazon ECS Deployment Types - Amazon Elastic Container Service</a>







































## 関連記事

関連アップデートとして、CodePipelineがSourceとしてECRに対応した記事もご覧ください。

* <a href="https://dev.classmethod.jp/cloud/aws/reinvent2018-codpipeline-source-ecr/" target="_blank">AWS CodePipelineがSourceとしてAmazon ECRを指定できるようななりました! #reinvent ｜ DevelopersIO</a>


