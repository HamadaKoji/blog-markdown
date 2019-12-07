# 全9項目に渡ってFargateの利用料金を一気に節約可能なFargate Spotを調べてみた

<strong>「え？？？FargateにSpot？何言ってるかわからない」</strong>

Fargate Spot、いきなりでましたね。AWSで言うところの「Spot」は通常Spotインスタンス（=EC2）のことを表現するので、ホストインスタンスを持たないFargateにSpotと聞いて私も意味がわかりませんでした。

先ほど、AWSよりFargate Spotについて発表がありました。

- <a href="https://aws.amazon.com/blogs/aws/aws-fargate-spot-now-generally-available/" target="_blank">AWS Fargate Spot Now Generally Available | AWS News Blog</a>

弊社ブログでも既に速報が挙がっています。

- <a href="https://dev.classmethod.jp/etc/aws-fargate-spot/" target="_blank">【最大70%引きで使用可能、東京でも利用可能】AWS Fargate Spotがリリースされました。#reInvent ｜ Developers.IO</a>
- <a href="https://dev.classmethod.jp/cloud/aws/einvent2019-new-feature-aws-fargate-spot-support/" target="_blank">[新機能] AWS FargateにSpotキャパシティプロバイダが追加されたので試してみた ｜ Developers.IO</a>

この記事では、Fargate Spotを使う上で必ず理解しておくべき上位概念のCapacity providerの解説と、利用上の各種注意事項をまとめます。<strong>はっきり言ってかなり複雑なので覚悟してください。</strong>ホンマに。

- ECS Cluter Capacity Providersとは？
- Fargat Spotの解説とその使い方
- Fargate Spotで起動しているタスクが終了したとき何が起こるのか？
- 通常のFargateに比べてどの程度安いのか？
- Compute Savings Plansとの併用は可能か？
- 既存クラスターの「Update Cluster」でProviderが設定できないときの対処法
- Update clusterに出てくる各ProviderのBaseとWeightの意味は？
- 一つのクラスターに複数Capacity provider（Fargate or Fargate Spot）のサービスを配置できるか？

<pre style="line-height:120%;">
俺に…理解…できるんだろうか…!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>

## ECS Cluter Capacity Providersとは？

Fargate Spotについて理解するまえに、その上位概念の「Amazon ECS Cluster Capacityn Providers」について理解する必要があります。公式ドキュメントはこちら。

- <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cluster-capacity-providers.html" target="_blank">Amazon ECS Cluster Capacity Providers - Amazon Elastic Container Service</a>

この中には、以下2点の要素「Capacity provider」と「Capacity provider strategy」が含まれます。

### Capacity provider

タスク実行時のインフラを指定する時に利用されます。

ECS on Fargateの場合は、「FARGATE」「FARGATE_SPOT」の2種類から選択します。今回FargateのSpotインスタンス的なものと言ってるのがこれです。

ECS on EC2の場合は、オートスケーリンググループのことを表し、タスクのスケーリングと終了方法を管理します。こちらは、別途<a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cluster-auto-scaling.html#asg-capacity-providers" target="_blank">Amazon ECS Cluster Auto Scaling</a>と提供されています。

こちらも既に、速報が弊社ブログに挙がっているので参考にしてみてください。

- <a href="https://dev.classmethod.jp/cloud/aws/aws-ecs-cluster-auto-scaling/" target="_blank">AWS ECS Cluster Auto ScalingがGAになったのでやってみた #reinvent ｜ Developers.IO</a>


### Capacity provider strategy

これを利用することで、タスクに対して複数のCapacity providerを設定できます。これはタスク設定時かサービス実行時に指定可能です。

Capacity provider strategyは、複数のCapacity providerから構成され、それぞれに<code>Base</code>と<code>Weight</code>を設定できます。

<code>Base</code>では最小実行タスク数を指定し、これが設定できるCapacity providerは1つだけです。

<code>Weight</code>は、そのCapacity providerで利用する起動タスク数の総体的な割合を指定します。

### 具体例で考えてみる

正直、上の説明だと全然わからない（自分もわからんかった）ので、Fargateの場合で具体例をあげて考えてみます。

#### 例1

|  capacity provider | Base | Weight |
| --- | --- | --- |
|  FARGATE | 1 | 1 |
|  FARGATE_SPOT | - | 1 |

この場合、まず、常にFargateとして1つのタスクが存在し、FargateとFARGATE_SPOTの割合が1:1なので、2つめのタスクはFARGATE_SPOTとして起動します。その後タスクが追加されるたびに、FARGATEとFARGATE_SPOTが同数追加されます。

#### 例2

|  capacity provider | Base | Weight |
| --- | --- | --- |
|  FARGATE | 2 | 1 |
|  FARGATE_SPOT | - | 4 |

この場合、まず、常にFARGATEとしてタスクが最低2個起動します。それ以上タスクが追加されると、FARGATEとFARGATE_SPOTがそれぞれ1:4の割合で追加されていきます。

この場合、合計タスク数の経過でそれぞれのCapacity providerごとのタスク数はこうなります。

|  合計タスク数 | 2 | 10 | 20 |
| --- | --- | --- | --- |
|  FARGATE | 2 | 2 | 4 |
|  FARGATE_SPOT | 0 | 8 | 16 |

### 使い方はどうなるのか？

いろいろ例示してきましたが、使い方の一例としてはこのようになるかと思います。

- 常に確保しておきたいタスクはベースを指定してCapacity providerをFARGATEとして指定
- オートスケーリングに応じて負荷が変更されスポットで割安で使いたいタスクをFARGATE_SPOTとして指定
- それぞれの割合を<code>Weight</code>で指定

### 注意事項

一点、注意事項ですが、Capacity provider strategyの<code>Base</code>がサポートされるのは、ECSのタスク実行を実施した場合のみです。ECSサービスを利用した場合は、<code>Base</code>値は無視され、<code>Weight</code>のみが利用されます。


## Fargat Spotの解説とその使い方

公式ドキュメントはこちら。

- <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/fargate-capacity-providers.html" target="_blank">Using AWS Fargate Capacity Providers - Amazon Elastic Container Service</a>

基本的な注意事項を下記にまとめます。

- FargateとFargate Spotは、別途作成は不要。全てのアカウントのクラスターでいつでも利用可能
- Fargateプラットフォームとして1.3.0以上が必要
  - （参考：<a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/platform_versions.html" target="_blank">AWS Fargate Platform Versions - Amazon Elastic Container Service</a>）
- タスク停止時には、<a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_cwe_events.html#ecs_task_events" target="_blank">Task State Change Events</a>が発動

## Fargate Spotで起動しているタスクが終了したとき何が起こるのか？

概念としてはSpotインスタンスに近いものなので、AWS側の都合によっていつタスクが終了するのかわからないのが、Fargate Spotで起動するタスク。これが終了になるとき、まず2分前に警告がタスク停止前に投げられます。これは、上で記した<a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_cwe_events.html#ecs_task_events" target="_blank">Task State Change Events</a>として発動し、同時にSIGTERMシグナルがタスクに対して送られます。

## 通常のFargateに比べてどの程度安いのか？

Fargateの価格は、以下のページにまとまっています。

<a href="https://aws.amazon.com/fargate/pricing/?nc=sn&loc=2" target="_blank">AWS Fargate Pricing - Run containers without having to manage servers or clusters</a>

東京リージョンにおける、オンデマンドタスクとSpotタスクの料金を比較してみます。

|  種別 | オンデマンド | Spot | オンデマンドに対するSpotの料金比率 |
| --- | --- | --- | --- |
|  per vCPU per hour | $0.050560 | $0.015168 | 30% |
|  per GB per hour | $0.005530 | $0.001659 | 30% |

図ったように30%ピッタリです。<strong>つまりオンデマンドに対してSpotは一律7割引で使えます。</strong>これは素晴らしい。

## Compute Savings Plansとの併用は可能か？

「Fargateに対してRI的なのが適用できるってどないなっとんねん！！」と物議を醸し出したSavnings Plan。

<a href="https://dev.classmethod.jp/cloud/aws/savings-plans-fargate/" target="_blank">Fargate利用費最大52%オフの衝撃！Savings PlansによるFargateの利用費削減を検討する</a>

従来のEC2では、RIとSpotインスタンスの併用はできませんでしたが、Fargateの場合、Savings PlanのCompute Savings Plansを利用すれば、単純にコンピューティングリソースの使用量に対してコミットするだけで割引を適用可能なため、Fargateタスクが、オンデマンドであろうがSpotインスタンスであろうが、使用量からの割引なため併用は可能です。

## 既存クラスターの「Update Cluster」でProviderが設定できないときの対処法

Fargate Stopを利用するとき、クラスターのアップデートをしてプロバイダーのクラスターへの登録が必要となります。しかし、最初既存のクラスターから「Update Cluster」するとき、「Default capacity provider strategy」の「Add provider」を押してもドロップダウンに内容が表示されませんでした。

解決策は、<strong>先に何でも良いので新規でECSクラスターを作ってこと</strong>です。そうすることで、無事既存のクラスターでも「Add provider」の一覧に、プロバイダーが表示されます。

## Update clusterに出てくる各ProviderのBaseとWeightの意味は？

下の部分です。

020

これは上で書いた、「Capacity provider strategy」を参照してください。


## 一つのクラスターに複数Capacity provider（Fargate or Fargate Spot）のサービスを配置できるか？

できます。最初にクラスターをアップデートし、2つのプロバイダーを登録しておくことで、サービス設定時のプロバイダーの登録が可能になります。


## まとめ「いきなりいろんな概念が登場して理解が難しい」

ドキュメントを読み込んでいて、一言そんな感想です。いやぁムズいですよ、今回のこのリリース。

いろんな説明をしましたが、ECSサービスをFargateで利用する場合、大体以下の手順で設定すれば良いと思います。

1. 一つ新規でクラスターを作成し、Capacity providerを選択可能な状態にする
2. 既存クラスターを更新し、2つのプロバイダー（FARGATE、FARGATE_SPOT）を追加し、それぞれのタスク数の比率を<code>Weight</code>で指定
3. クラスター内の各サービスに、どのプロバイダーを利用するか指定

これにより、手順としては非常にシンプルにFargate Spotを活用できると思います。

今回は、マニュアルを読みながら利用上の注意点をまとめてみました。次のブログでは、各プロバイダーの重み付けによるタスク数の割合の確認、および、Fargate Spotのタスクが落ちるときの挙動を試してみたいと思います。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

