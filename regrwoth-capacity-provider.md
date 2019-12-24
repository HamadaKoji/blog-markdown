https://dev.classmethod.jp/cloud/aws/regrwoth-capacity-provider/

Capacity Providerとは？ECSの次世代スケーリング戦略 #reinvent #cmregrowth

「俺は、すっかりCapacity Provider好きになった」

ECSのスケーリング戦略をさらに柔軟に設定でき、さらにFargate Spotを活用できるCapacity Provider、皆さんすでにお使いでしょうか？

re:Invent2019の開催中に突如発表された新機能なのですが、「Fargate Spotとかいうのが使えるらしいで！」というキャッチーな側面はありつつも、その概念はEC2でも使えるECS全体を包括するもので、正直自分も最初とっかかりが難しかったのをよく覚えています。

この記事は、<a href="https://dev.classmethod.jp/news/191211-re-growth-tokyo/" target="_blank">CM re:Growth 2019 TOKYO</a>で発表した内容を元に、Capacity Providerの全体像を掴んでもらうことを目的に書きました。<strong>ECSにおけるかなり大きな機能拡張なので、普段からECSを使われている方は、是非この記事でこの新機能を掴んでもらえればと思います。</strong>

## 登壇資料

Speakerdeckにアップしてます。

<script async class="speakerdeck-embed" data-id="48d276603e724f62b247cc4c576ceafe" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>


## Capacity Providerについて聞いてもらいたい方

この機能についてお話する前に、一体どういう方にこの機能について聞いてもらいたいかを共有します。

### 既にECSを活用されている方

スケーリング戦略を根本から見直しできるかも！

### ECSをFargateで使っている方

Fargate Spotの安さと使いやすさにびっくり！

### コンテナまだ使ってない方

<strong>ごっつええ感じのがでた</strong>と頭の片隅においといてください！

というわけで、はい。全ての人に聞いてもらいたいです。今回の内容は、一度でもECSを触ったことがある人向けの発表になっているので、ECSの経験が無い方にはよくわからない部分も多いです。ただ、今後、ECSを活用していく上では必須の機能になっていくので、是非、そういうものがあったなぁ記憶の片隅にとどめておいてください。

## アジェンダ

今日お話する内容は以下の通り。

- Capacity Providerと
  - ECS on EC2の場合
  - ECS on Fargateの場合
  - Capacity Provider Strategyによるタスク配分戦略
- Capacity Providerのユースケース
- あなたにも今すぐ10分でできるFARGATE_SPOTの活用方法

## Capacity Providerとは

Capacity Providerとは<strong>「ECSにおけるタスク実行のインフラをより柔軟に設定する新しい仕組み」</strong>です。タスク実行のインフラといえば、ECSの場合データプレーンとしてEC2とFargateが利用できますが、Capacity Providerはその両方に対応しています。

さらに、Capacity Providerを複数まとめたものが、Capacity Provider strategyとなります。まずは、この２つについて、頭に入れておいてください。

|  **名称** | **特徴** |
| :--- | :--- |
|  Capacity provider | ECSのタスクを実行するインフラを決定する<br/> ・EC2：Auto Scaling Group<br/> ・Fargate：FARGATE, FARGATE SPOT |
|  Capacity provider strategy | 複数のCapacity providerの組み合わせ方（最小タスク数、タスク数比率）を決定する |

### ECS on EC2の場合

早速、EC2で使う場合のCapacity Providerの構造を見ていきましょう。こんな感じになっています。

s13

Auto Scaling groupは、従来ある仕組みです。この中に、以下のネットワークや実際に起動するEC2インスタンスの情報を入れて作成しておきます。

- VPC
- サブネット
- AMI
- インスタンスタイプ
- オンデマンド or スポット

それを、Capacity Providerでラップしておきます。組み合わせるCapacity Providerの数は任意です。で、それらをCapacity Provider Strategyとして組み合わせて、ECS Clusterに適用します。

これらを作成していく順番です。この起動順序にそって作っていくわけなんですが、一点従来と違うのは、<strong>タスク実行後にEC2インスタンスが起動する</strong>ところです。

s

これは、最初のAuto Scalig groupを作るときには、Scaling Policiesそのものは設定されておらず、Capacity Providerでラップした後、

該当するAuto Scaling Groupには、このように、Capacity Providerと連携する用に表示されます。スケールインポリシーなども、Capacity Providerの設定を引継ぎます。

s00

### ECS on Fargateの場合

Fargateの場合はもっとシンプルです。

s20

Capacity Providerに指定できるのは、FARGATE、FARGATE_SPOTの2種類のみ。このFARGATE_SPOTが、今回Capacity Providerと共に発表された大きな新機能です。FARGATE_SPOTの概要はこちら。


- EC2におけるスポットインスタンスのようにスポットでFARGATEを使う仕組み
- オンデマンドとは違うので、リージョンやAZでの利用状況に応じて突如終了される
  - Task State Change Events発行後SIGTERM
- <strong>お値段70%OFF（一律）</strong>

スポット的な扱いなので、いつでも使えるわけではないし使っている時にいつでも落ちる可能性がありますが、70%OFFで利用できるのは、ありがたい。

FARGATE_SPOTについての詳細は、下記ブログを参考ください。

- <a href="https://dev.classmethod.jp/cloud/aws/fargate-spot-detail/" target="_blank">Fargateをスポットで7割引で使うFargate Spotとは？ #reinvent ｜ Developers.IO</a>

FargateにおけるCapacity Providerの作り方は非常にシンプルです。

s25

### Capacity Provider strategyによるタスク配分戦略

ここまで、EC2の場合とFargateの場合での、Capacity Providerの作り方について話してきました。で、この作成したCapacity ProviderをCapacity Provider strategyで実際に組み合わせます。

組み合わせる際に必要なパラメータは以下の２つ。

- Base：最小タスク数（1つのみ指定）
  - 注意：run-taskのみ有効、create serviceでは無効（将来対応予定）
- Weight：タスク数比率

例えば、Capacity Provider strategyを以下で設定します。

|  **ProviderName** | **Base** | **Weight** |
| :--- | :--- | :--- |
|  FARGATE | 2 | 1 |
|  FARGATE_SPOT | 0 | 1 |

このとき、合計タスク数による各Capacity Providerの推移は以下の通り。Base=2で指定したFARGATEが先に起動し、その後は、FARGATEとFARGATE_SPOTが順番に起動します。

rgs1

こちらも、様々なパターンで検証した記事があるので、合わせてこちらもご参考ください。

- <a href="https://dev.classmethod.jp/cloud/aws/fargate-spot-task-count/" target="_blank">Fargate Spotにおける各Provider毎のタスク数起動推移を検証してみた #reinvent ｜ Developers.IO</a>

## Capacity Provider のユースケース

じゃぁ、実際にどんな感じで設定したらこの機能を業務に活かせそうか。2つほど紹介します。

### ユースケース1:オンデマンドandスポット

s30

### ユースケース2：AZのバランシング

s31

## あなたにも今すぐ10分できるFARGATE_SPOTの活用方法

最後に、Fargateを普段遣いしている人にむけて、すぐできるFARGATE_SPOTの活用方法を紹介します。

1. みなさんの開発環境のECSクラスターのCapacity Providerを以下の設定にする
   - FARGATE_SPOTのみ
2. そのクラスターは全てFARGATE_SPOTでタスクが起動する
3. <strong>落ちない限りずっと7割引で使える！</strong>

とはいっても、開発環境だからといってFARGATE_SPOTのみにしたら、落ちまくってめんどくさいのでは？と思うじゃないですか。自分、昨日からFARGATE_SPOTを10タスク起動していますが、46時間1つも落ちてないです。

S35

つまりこういうことです!!

S36

## まとめ

- ECSのタスク数を非常に柔軟に設定できるCapacity Providerがリリース
- はっきり言って分かりづらい（触らないとわからない）
- スケーリング戦略を低コストで実現できる可能性大
- 開発環境でFARGATE_SPOTから使ってみよう！

皆さんのECS運用がさらに安く良くなることを願っております。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。


## 後日談

このブログ書きながらさっき確認したんですが、80時間経過してもまだFARGATE_SPOT落ちませんでした…

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">さっき止めるまで80時間ぐらい、10タスク中1つも落ちませんでした。FARGATE_SPOT、マジで今は東京リージョン使い放題です。 <a href="https://t.co/qDIX3vu2wo">https://t.co/qDIX3vu2wo</a></p>&mdash; 濱田孝治（ハマコー） (@hamako9999) <a href="https://twitter.com/hamako9999/status/1205233554734473217?ref_src=twsrc%5Etfw">December 12, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

今だけですよ！知らんけど！

