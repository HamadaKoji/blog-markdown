Fargate Spotにおける各Provider毎のタスク数起動推移を検証してみた

re:Invent2019で突然発表されたFARGATE Spot、皆さん既にいろいろタスク起動しておられますでしょうか？

先日、Fargate Spotについて、公式ドキュメントを確認しながら以下の記事で、その内容と代表的な疑問点をまとめてみました。

<a href="https://dev.classmethod.jp/cloud/aws/fargate-spot-detail/" target="_blank">Fargateをスポットで7割引で使うFargate Spotとは？ #reinvent ｜ Developers.IO</a>

この記事では、各Providerの設定値を数パターン試しながら、実際にタスクを1〜10個まで起動しつつ各Providerが起動するロジックを検証ました。グラフも載せたので設定値ごとの各Providerの起動推移や、Fargate SpotのCapacity Provider戦略が一覧できます。

Fargate Spot、使ってみようと思っているけどよくわからないなぁと思う方は、是非上で紹介した記事と合わせて、この記事御覧ください。

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     Fargate Spot ﾏﾂﾘﾀﾞﾜｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>

## 検証前の事前準備

今回は、AWS CLIを利用してrun taskを使って検証していきます。事前に、AWS CLIを最新バージョンに更新しておきます。

[bash]
$ aws --version
aws-cli/1.16.298 Python/3.7.4 Darwin/19.0.0 botocore/1.13.34
[/bash]

<code>FARGATE</code>と<code>FARGATE_SPOT</code>を定義したクラスターを作成します。

[bash]
$ aws ecs create-cluster --cluster-name FargateSpotTestCluster --capacity-providers FARGATE FARGATE_SPOT
[/bash]

事前に任意の名前でECSタスク定義を作成しておきます。安定して起動するものであれば何でも構いません。ここでは、<code>simple-nginx-task</code>という名前で、nginxのイメージを指定しただけのシンプルなタスク定義を使います。

### run taskによるCapacity Providerを指定した起動方法

AWS CLIには、<code>run-task</code>に<code>--capacity-provider-strategy</code>オプションが追加されているので、これを使います。

参考：<a href="https://docs.aws.amazon.com/cli/latest/reference/ecs/run-task.html" target="_blank">run-task — AWS CLI 1.16.298 Command Reference</a>

下記が実際にrun taskするときのスニペットです。

[bash]
aws ecs run-task \
  --capacity-provider-strategy capacityProvider=FARGATE,weight=4,base=2 capacityProvider=FARGATE_SPOT,weight=1,base=0 \
  --network-configuration awsvpcConfiguration="{subnets=[subnet-010cd248,subnet-5456460c],securityGroups=[sg-09b8040d264243711],assignPublicIp=ENABLED}" \
  --cluster FargateSpotTestCluster \
  --task-definition simple-nginx-task:2 \
  --count 1 | jq '.tasks[].capacityProviderName'
[/bash]

- <code>capacity-provider-strategy</code>
  - FARGATE、およびFARGATE_SPOTの各プロバイダーの<code>base</code>と<code>weight</code>
- <code>network-configuration</code>
  - FARGATEのawsvpcネットワークモードだと指定が必須な、タスクを格納するネットワーク情報を指定
- <code>count</code>
  - タスク実行時のタスク数を指定

<code>run-task</code>コマンドは、実行時に標準出力に起動したタスクの詳細情報を返すので、それを受けて最後の<code>jq '.tasks[].capacityProviderName'</code> で、生成されたタスクの<code>capacityProviderName</code>を表示します。

### run-taskの実行例

例えば、以下の条件で実行したときのスクリプトと実行結果はこちら。

|  ProviderName | Base | Weight |
| --- | --- | --- |
|  FARGATE | 2 | 1 |
|  FARGATE_SPOT | 0 | 3 |

タスク数を8で指定。

[bash]
$ aws ecs run-task \
  --capacity-provider-strategy capacityProvider=FARGATE,weight=1,base=2 capacityProvider=FARGATE_SPOT,weight=3,base=0 \
  --network-configuration awsvpcConfiguration="{subnets=[subnet-010cd248,subnet-5456460c],securityGroups=[sg-09b8040d264243711],assignPublicIp=ENABLED}" \
  --cluster FargateSpotTestCluster \
  --task-definition simple-nginx-task:2 \
  --count 8 | jq '.tasks[].capacityProviderName'
"FARGATE"
"FARGATE"
"FARGATE"
"FARGATE"
"FARGATE_SPOT"
"FARGATE_SPOT"
"FARGATE_SPOT"
"FARGATE_SPOT"
[/bash]

こんな感じで、プロビジョニングされたタスクの<code>capacityProviderName</code>が表示され、各Provider毎の起動数がわかるというわけです。

一点実施上の注意点ですが、<strong>ECSクラスター内でruntaskでタスク実行したのと都度全てのタスクを停止して検証しています。</strong>

ここまで準備ができたところで、実際に検証していきましょう。

## FARGATEとFARGATE_SPOTのWeightが同じ場合

### Baseが0の場合

最初に、Baseを設定せずに、Weightの比率を同じにします。

|  ProviderName | Base | Weight |
| --- | --- | --- |
|  FARGATE | 0 | 1 |
|  FARGATE_SPOT | 0 | 1 |

合計タスク数毎の各Providerタスク数遷移がこちら。

|  合計タスク数 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  FARGATE | 1 | 1 | 2 | 2 | 3 | 3 | 4 | 4 | 5 | 5 |
|  FARGATE_SPOT | 0 | 1 | 1 | 2 | 2 | 3 | 3 | 4 | 4 | 5 |

一点興味深いのは、<strong>タスク数が奇数でWeightで割り切れないときは常にFARGATEが優先して起動されているということです。</strong>ここらへんは、マニュアルにも特に記載が無いように見えるのですが、どちらかを利用する場合は、確実に起動できるオンデマンドのFARGATE側を優先するロジックのように見えます。

グラフにすると、常に、FARGATE側が多いのがわかります。

010

### Baseが2の場合

Weightの比率を同じにしたまま、FARGATEにBase=2を指定します。

|  ProviderName | Base | Weight |
| --- | --- | --- |
|  FARGATE | 2 | 1 |
|  FARGATE_SPOT | 0 | 1 |

合計タスク数毎の各Providerタスク数遷移がこちら。

|  合計タスク数 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  FARGATE | 1 | 2 | 3 | 3 | 4 | 4 | 5 | 5 | 6 | 6 |
|  FARGATE_SPOT | 0 | 0 | 0 | 1 | 1 | 2 | 2 | 3 | 3 | 4 |

Baseで指定した2が先に起動されそれがベースのタスクとなり、その後起動されたタスクはWeightの比率で割り振られています。ここでも、奇数タスクの場合は、FARGATEが優先されています。

020


## FARGATEとFARGATE_SPOTのWeightが異なる場合

### Baseが0の場合

次に、各ProviderごとのWeightが異なる場合を検証します。

|  ProviderName | Base | Weight |
| --- | --- | --- |
|  FARGATE | 0 | 4 |
|  FARGATE_SPOT | 0 | 1 |

合計タスク数毎の各Providerタスク数遷移がこちら。

|  合計タスク数 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  FARGATE | 1 | 2 | 2 | 3 | 4 | 5 | 6 | 6 | 7 | 8 |
|  FARGATE_SPOT | 0 | 0 | 1 | 1 | 1 | 1 | 1 | 2 | 2 | 2 |

Weight比率は4:1ですが、タスク数3つ目でFARGATE_SPOTが起動しています。ロジックとしては、恐らくWeight比率に対して一番タスク数比率が近くなるような順番で起動していると考えられます。

030

### Baseが4の場合

最後に、Baseを指定して、異なるWeight比率で検証してみます。

|  ProviderName | Base | Weight |
| --- | --- | --- |
|  FARGATE | 4 | 1 |
|  FARGATE_SPOT | 0 | 4 |

合計タスク数毎の各Providerタスク数遷移がこちら。

|  合計タスク数 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  FARGATE | 1 | 2 | 3 | 4 | 4 | 4 | 5 | 5 | 5 | 5 |
|  FARGATE_SPOT | 0 | 0 | 0 | 0 | 1 | 2 | 2 | 3 | 4 | 5 |

最初のうちは、Baseを最優先で確保。Base指定分のタスクが起動した後、Weight比率にのっとり、その後のProvider指定したタスクが起動しています。

040

## タスク起動優先順位についての検証まとめ

とりあえず、一通り動かしてみたところ、以下のロジックで各Providerごとのタスクが起動しているようです。

- Weight比率が指定の比率に一番近くなるProviderが追加される
- 追加後のWeight比率が同じ場合FARGATEがFARGATE_SPOTより優先される
- Baseのタスク数は、Weight比率計算の対象外

1つめと2つめについては、公式ドキュメントにその旨記載が見当たらなかったので将来変更される可能性もありますが、23019年12月時点では、このロジックになっています。

<strong>実際にタスク数遷移を追いながら各設定値の意味を噛み締めてみました。ドキュメント読むだけでは理解が曖昧な点が多々あったので、ようやくイメージが掴めた感じです。疲れたけど、やってみてよかった。</strong>

皆さんも、これらのグラフやタスク数遷移を参考にしていただきつつ、現場で運用されているコンテナワークロードにマッチするCapacity Providerの設定値を模索していただければと思います。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

