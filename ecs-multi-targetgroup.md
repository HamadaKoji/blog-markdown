<strong>「なんか同じタスク定義から別々のECSサービス作成されてるけど、これなんなん？」</strong>

<strong>「しゃーないやん、ターゲットグループ1つしか登録できへんねんから」</strong>

待望のアップデートです！　Amazon ECS（EC2とFargate両方）において、ECSサービスに複数のターゲットグループをアタッチできるようになりました！

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://aws.amazon.com/jp/about-aws/whats-new/2019/07/amazon-ecs-services-now-support-multiple-load-balancer-target-groups/" target="_blank">Amazon ECS サービスで複数のロードバランサーターゲットグループのサポートを開始</a>
</p>

従来、ECSサービスに対してターゲットグループが1つしか登録できないために複数のロードバランサーを紐付けできなかったのが、複数登録できるようになりました。

<strong>ECSサービスを1つにまとめることで運用面でもランニング費用面でもメリットがありますが、逆に利用上注意が必要な点も多いので、そのあたりも合わせてお伝えいたします。</strong>

<pre style="line-height:120%;">
ターゲットグループつけ放題きたか…!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>

## ECSサービスに対してターゲットグループが1つ制限だったときの辛い点

例えば、ECSサービスをインターネット向けとオンプレミスからの両方に公開する場合を考えてみます。この場合、インターネット向けにはパブリックなALBを、オンプレミス向けにはインターナルなALBを用意するのが典型的な構成です。

この場合ALBが別となるので、以下のようにECSサービスを2つ用意する必要がありました。

010.png

## ECSサービスに対して複数ターゲットグループが登録できる場合

一つのECSサービスに対して複数のターゲットグループが登録できるので、インターネット経由のトラフィックとオンプレミス経由のトラフィックを異なるALBから一つのECSサービスで受けることができます。

020.png

## 実際に設定してみる

ほな、実際に複数ターゲットグループを設定して行きましょう。公式ドキュメントはこちら。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/register-multiple-targetgroups.html" target="_blank">サービスに複数のターゲットグループを登録する - Amazon Elastic Container Service</a>
</p>

<strong>2019年8月19日現在、Webコンソールからの複数ターゲットグループ登録はできません。</strong>

<blockquote>現在、複数のターゲットグループを指定してサービスを作成する場合は、Amazon ECS API、SDK、AWS CLI、AWS CloudFormation テンプレートを使用してサービスを作成する必要があります。<br ><p>引用：<a href="https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/register-multiple-targetgroups.html">サービスに複数のターゲットグループを登録する</a></p></blockquote>

ので、ここでは、CloudFormationを利用して登録していきます。

### ECSサービスに紐付けるロードバランサーとターゲットグループの作成

インターネット公開する<code>public-alb</code>と、内部公開用の<code>private-alb</code>、およびそれぞれに紐づくターゲットグループを事前に作成します。

Webコンソールだと、ECSサービス作る時にターゲットグループの作成をあわせて実施できますが、今回はWebコンソールは使えないため、事前にターゲットグループも作成して、それぞれのARNを控えておきます。

### CloudFormationによるECSサービスの作成

CloudFormationのテンプレートですが、上の公式ドキュメントにJSONの例が記載されているとおり、ECSサービスに紐づくLoadBalancerを配列で複数指定するだけです。下のCloudFormationテンプレートのハイライト部分を参考にしてみてください。

[yaml highlight="15-20"]
 AWSTemplateFormatVersion: '2010-09-09'
Description: ecs multiple tg service

Resources:
  service:
    Type: AWS::ECS::Service
    Properties:
      Cluster: container-insights-test
      DeploymentConfiguration:
        MaximumPercent: 200
        MinimumHealthyPercent: 50
      DesiredCount: 1
      LaunchType: FARGATE
      LoadBalancers:
        - ContainerName: php-container
          ContainerPort: 80
          TargetGroupArn: arn:aws:elasticloadbalancing:ap-northeast-1:629895769999:targetgroup/public-tg/a66f42619579a4b8
        - ContainerName: php-container
          ContainerPort: 80
          TargetGroupArn: arn:aws:elasticloadbalancing:ap-northeast-1:629895769999:targetgroup/internal-tg/583e97e2acd5ef55
      NetworkConfiguration:
        AwsvpcConfiguration:
          AssignPublicIp: ENABLED
          SecurityGroups:
            - sg-4681a620
          Subnets:
            - subnet-f90cd2b0
            - subnet-e15343b9
      TaskDefinition: fargate-php-task:3
[/yaml]

このテンプレートを元にECSサービスを作成します。指定内容にエラーが無ければ、無事サービス定義の中で、ターゲットグループが2つ登録されているのが表示されました！

030.png

ターゲットグループ内でコンテナに対してヘルスチェックがきちんと通っている状態であれば、無事双方のロードバランサーから中味のリクエストが帰ってきました。動作的にはOKです。

## 気になる挙動を試してみる

最低限の動作は確認できましたが、気になる点があったので、もう少し深堀りしてみます。

### ターゲットグループは3つ以上登録できるのか？

特にドキュメントなどには記載が見当たらなかったのですが、ターゲットグループは2つ以上登録できます。とりあえず、3つは登録できました。

040

### 複数テーゲットグループのうち1つのヘルスチェックがUnhealtyの場合、タスクはどうなるか？

3つターゲットグループが登録され全て<code>Healty</code>の状態から、1つのターゲットグループが<code>Unhealty</code>になったとき、タスクはどうなるか検証してみました。

<strong>結論としては、ECSサービスに紐づく複数ターゲットグループのうち、どれか1つが<code>Unhealty</code>になると、タスクは停止されます。</strong>

1つのターゲットグループのヘルスチェックパスを存在しないパスに変更し、404が変えるように変更しました。タスクのログには以下で表示されています。

以下、ログの抜粋。

050

この状態でしばらく待つと、ヘルスチェックに失敗したターゲットグループを明示し、タスクが停止します。以下のメッセージを出力し停止します。

<code>task failed ELB health checks in (target-group arn:aws:elasticloadbalancing:ap-northeast-1:629895769338:targetgroup/public-tg2/1c38bf81f6e9e283)</code>

このように、どのターゲットグループであろうが、ECSサービスに対するヘルスチェックが<code>Unhealty</code>になった時点で、タスクの停止処理が起こることは注意しておきましょう。

## その他注意事項

サービス定義で複数ターゲットグループを登録したときの注意点が公式マニュアルに記載されています。

<blockquote>・複数のターゲットグループは、Application Load Balancer または Network Load Balancer ロードバランサータイプを使用する場合にのみサポートされます。<br />・複数のターゲットグループは、サービスがローリング更新 (ECS) のデプロイコントローラータイプを使用する場合にのみサポートされます。CodeDeploy または外部デプロイコントローラーを使用している場合、複数のターゲットグループはサポートされていません。<br />・Fargate および EC2 の両方の起動タイプを使用するタスクを含むサービスでは、複数のターゲットグループがサポートされています。<br ><p>引用：<a href="https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/register-multiple-targetgroups.html#multiple-targetgroups-considerations">サービスに複数のターゲットグループを登録する - Amazon Elastic Container Service</a></p></blockquote>

ロードバランサーにCLB（Classic LoadBalancer）は利用できません。ただ、ALBとNLBの併用ができるのは嬉しいですね。インターネットアクセスはパブリックALBで受けて、プライベートリンク経由のアクセスをNLBで受けることも可能です。

ローリングデプロイでしか使えません。まぁここらへんは当然といえば当然でしょう。また、FargateとEC2の両方で使えるのも便利です。

## ECSサービス複数ターゲットグループ登録のメリット

### メリット①「TCOの削減に寄与する」

改めてこの機能のメリットですが、運用の簡素化とランニングコストの低減が可能です。

同じ機能を提供したいのに、トラフィックの種別が異なるため仕方なく別サービスにしていた場合、一つのECSサービスに集約することでコンテナ更新時の手間が省略でき、またコンテナリソースの有効活用が期待できます。

扱うリソースをへらすことはTCO削減の第一歩なので、集約できるところはどんどん集約してしまって良いでしょう。

### メリット②「複数サービスを提供するECSタスクが作りやすくなる」

例えば、Jenkinsコンテナの場合、1つのコンテナでウェブインターフェース用にポート8080を、API用にポート50000を公開するといったことが簡単になります。

また、1タスク内にWebserverコンテナとDatabaseコンテナを同梱させ、それぞれをポートを分けて外部に公開するといったことも可能です。

1コンテナに複数のアプリやプロセスを同居させるのはアンチパターンになりますが、うまくタスクやコンテナの流動を設計することで、従来できなかったサービスの提供が可能となります。

## ECSサービス複数ターゲットグループ登録のデメリット

ただ、この機能うかつに導入するとデメリットも結構あります。いきなり構成変更する前に、以下の点が本当に許容できるか必ず確認しておきましょう。

### デメリット①「ECSサービスの更新と運用が同じになる」

インターネットからのアクセスと社内からのアクセスは、サービス提供レベルが異なることが多いでしょう。もし双方のアクセスがECSサービス1つに集約されている場合、社内アクセスのみ緊急停止したりテスト的にタスク数を変更したりといった対応が難しくなります。

また、コンテナの更新も1つのECSサービスに対して行うことになるので、ECSサービスが1つだけだと先に社内向けにテスト用に最新バージョンのコンテナをデプロイして動作確認するといったこともできなくなります。

### デメリット②「別トラフィックのログが混ざる」

利用用途によりますが、1つのECSサービスに集約することで、それぞれのトラフィックのコンテナログが混ざることも注意が必要です。基本的にコンテナログは、CloudWatch Logsを使う場合タスクIDでユニークに出力されますが、別トラフィックを1つのECSサービスで受ける場合、両方のログが混ざることも注意が必要です。

### デメリット③「セキュリティグループやサブネットなどのネットワーク設定も同じになる」

Fargateなど、ECSタスク定義でネットワークモードがawsvpcの場合、ECSサービスでセキュリティグループやサブネットなどの情報を登録します。本来的には各サービスの利用用途に応じてセキュリティグループは適切に制限すべきなので、ECSサービスを統一したからと言って不要な経路を公開しないように気をつけましょう。


## 「ECSによるコンテナワークロードが自由度が飛躍的に増加するアップデート」

現状、Webコンソールでの登録ができないのは難点ですが、設定自体は非常にシンプルと言える今回のアップデート。

上のデメリットに記載したとおり、アプリケーションの全ての機能を1つのECSサービスに集約する必要はありません。無理にやると余計めんどくさくなりますが、<strong>うまく使えばあなたのECSコンテナワークロードの運用を飛躍的に簡単にする可能性があります。</strong>

一度、現在のシステムに適用箇所があるかどうか、コレをきっかけに検討いただければと思います。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。
