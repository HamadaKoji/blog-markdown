re:Invent 2019、Wernerのキーノートで、Amazon Builder's Libraryが発表されました１

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://aws.amazon.com/jp/builders-library/welcome-to-the-amazon-builders-library/" target="_blank">Welcome to the Amazon Builders’ Library</a>
</p>

弊社西澤の速報記事もでております。概要はこちらを参照ください。

<a href="https://dev.classmethod.jp/cloud/aws/reinvent-2019-amazon-builders-library/" target="_blank">[速報] The Amazon Builders’ Libraryが発表されました #reinvent ｜ Developers.IO</a>

現在、13個のライブラリが登録されていますが、<strong>この中から自分が気になったライブラリーを何個か紹介しようと思います。思った以上に内容が濃い。これは読むのに時間かかりそう。</strong>

## Challenges with distributed systems（分散システムの課題）

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://aws.amazon.com/jp/builders-library/challenges-with-distributed-systems/?did=ba_card&trk=ba_card" target="_blank">Challenges with distributed systems</a>
</p>

### 概要

Builder's Libraryの内容は、全体的にAWSが培ってきた複雑な分散システムにおける開発〜運用をテーマにしていますが、このライブラリーでは、<strong>そもそもなぜ分散システムが奇妙で難しいのか</strong>が、解説されています。<strong>このライブラリーを読み込んでいく時、一番最初に読んでおくべきものです。</strong>

分散システムの中でも、特に実装の難易度が高い、応答を即座に返す必要があるハードリアルタイム分散システムでの考慮事項の多さ。ネットワークを使わない場合に比べてどれぐれぐらい複雑なのか以下の例で解説します。

010

ここで、典型的な処理ステップを解説したのち、各コンポーネントにおいてエラーが発生する可能性を考慮したより現実的な処理ステップが解説されます。こうすることで、処理ステップがさらに15に爆発します。

これらを掘り返していくと、分散システムのエンジニアリング上の課題は、以下に要約されます。

- エンジニアによるエラー状態の組み合わせの不可能性
- ネットワーク経由での操作結果は不明になる可能性がある
- 問題は、物理レベルだけではなく論理レベルでもおこる可能性がある
- 高レベルシステムになると、再帰処理により問題が悪化する
- バグは多くの場合、全システムに展開されてから表出する
- バグはシステム全体に広がる可能性がある
- 問題の多くはネットワークの物理法則に由来するものであり、変更することができない

これら問題山積みな分散システムに対する解決のアプローチを提示しているのが、Builder's Libraryです。

### 濱田感想

分散システムが本質的に難しい点、特に、クライアント→ネットワーク→サーバーと、システムが物理的に隔離された場合に、低レベルから高レベルまでエラーが起こりうる可能性、その対処方法、考えたの基本がわかります。普段はここらへんの低レイヤーの部分はすべてクラウドなりネットワークに任せっぱなしだったので、基礎を理解するのに役立ちました。


## Avoiding fallback in distributed systems（分散システムにおけるフォールバックの回避）

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://aws.amazon.com/jp/builders-library/avoiding-fallback-in-distributed-systems/?did=ba_card&trk=ba_card" target="_blank">Avoiding fallback in distributed systems</a>
</p>

### 概要

サービスになにか重大な障害が起こったとき、それを処理する戦略は複数あります。

- 再試行
- プロアクティブな再試行
- フェールオーバー
- フェールバック

この記事では、Amazonはこれらをほとんど利用しないこと、代替戦略を用いていることを説明します。

フォールバックを使わない主な理由はいかの通りです。

- フォールバックロジックのテストの困難さ
- フォールバック自体が失敗する可能性
- 可用性が最優先の場合、フォールバック戦略がリスクに見合わない

Amazonでは、リアルタイムシステムを目的とした分散システムで、単一アプリケーションと同じトレードオフを提供しません。代わりに、Amazonでは以下の方法を用いて、分散システムのフォールバック戦略を提供しています。


- 非フォールバックケースの信頼性向上
- 呼び出し元によるエラー処理
- データのプッシュ
- フォールバックをフェールオーバーに変換
- 再試行とタイムアウトがフォールバックにならないようにする


### 濱田感想

ここらへんから、難しいです。現実世界では、なにか障害が起こった時にワークアラウンド的な代替策を用いて対処することが多いですが、分散システムの世界ではその戦略が基本的にうまくいかず、余計複雑さとコストを増大させることが事細かに説明されています。

それらの思想から、AWSのCLIやSDKが設計されていることも説明されています。

## Leader Election in Distributed Systems（分散システムにおけるリーダーの選出）

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://aws.amazon.com/jp/builders-library/leader-election-in-distributed-systems/?did=ba_card&trk=ba_card" target="_blank">Leader election in distributed systems</a>
</p>

### 概要

分散システム内の一つに特別な権限を与えるというアイデアは、単純だけど魅力的なものです。ただ、副作用があるのも事実。この記事では、リーダー選出のメリットとデメリット、Amazonにおけるリーダー選出のアプローチについて解説します。

#### リーダー選出のメリット

- 単一リーダーにより、考慮事項が単純になります
- コンセンサスを取る必要がないので、実装が簡単になります
- システムの状態すべてを変更、確認できるため、クライアントに簡単に一貫性を提供できます
- キャッシュを一元管理することでパフォーマンスを向上させることができます


#### リーダー選出のデメリット

- 単一障害点になるため、リーダーの故障はシステム全体に波及します
- 単一信頼店になるため、リーダーが間違った場合に誰もそれぞチェックできません

#### Amazonがリーダーを選ぶ方法

リーダー選出には一般的に以下の方法があります。

- Paxos
- Apache Zookeeper
- カスタムハードウェア
- leases

leasesは組み込みのフォールトトレランスを提供し、現在のリーダーをデータベースに格納しリーダーからのハートビートを受付ながらリーダーの正当性を確認します。

#### リーダー選出のベストプラクティス

- 残leases時間の頻繁なチェック
- ネットワーク遅延によるリース期限切れ
- バックグラウンドスレッドでのハートビートleasesを避ける
- リーダーの監査証跡の提示
  
### 濱田感想

Amazonのシステムで利用されているリーダー選出のロジックについて知ることができます。この点を意識して実装したことはなかったのですが、もし、構築する分散システムにおいてリーダー制御が必要になったときは、これをいの一番に参照してみようと思ってます。

## Amazonの分散システム構築の長い長い歴史を体現した珠玉のノウハウ

Amazon Builder's Library。本日リリースされましたが、既に13個ものライブラリーがリリースされています。思っていた以上にそれぞれのライブラリーの記載内容が濃く、内容を要約するのも精一杯でした。まだ理解が曖昧な部分も多いですが、実装依存した機能の話ではなく、実装の前に必要な分散システムにおける考え方が非常に細かくまとまっているので、ほとんどのエンジニアには参考になることが多いんじゃないでしょうか。
それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。










