ソフトウェアを書いている人
インフラ系の人間
DevOpsも両方やってる
ガチインフラ系→あんまりコード書かない

全体的にコード書く人が多い。

## AWSの環境を構築する方法

- 手動操作（マネジメントコンソール）
  - なによりひどいのは操作手順書が必要
- スクリプト
  - 操作の手順が書いてある
  - 3台作ると書くと、3台追加される
  - 状態を定義しているわけではない
- プロビジョニングツール
  - あるべき状態を定義する
  - 自動化が容易
  - 抽象化なし、詳細な記述が必要
- Document Object Models（DOMs）
  - Troposphere
  - SpakleFormation
  - GoFormation
  - 書くコードの量はあまり減らない
- AWS CDK
  - あるべき状態の定義がコードで可能+抽象化

## AWS CDKとは

コードで定義。AWSのベストプラクティスがライブラリーとして用意されている。オープンソースツールなのでPRも投げれる。

TypeScriptとPythonがGA

TypeScriptとPython、どっちが多い。→　同じぐらい

普段は？
TypeScriptとPython
Pythonが圧倒的に多い。

今日は、TypeScriptが多い。PythonはInovateで話す予定。

CDKはもともとTypeScriptをターゲットに作られていた。CDK自体はTypeScript自体で書かれているが、それをPythonからアクセスできるようにしている。jsii自体はオープンソース。jsii自体はCDKのために作られている。

### CDKの特徴

- CDKのメリット
  - 一般のプログラミング言語が使える
    - クラスや継承など抽象化の概念が利用できる
    - エディタによる型チェック、サジェスト、API仕様の参照が可能
  - 何も見なくて、コードが書きやすい
  - テストコードが記述できる
    - なんでこうなっているかわからない。運用が難しい、というところがカバーできる。
    - 非常に良いところ
  - 複数スタック間の依存関係が記述できる
  - バックエンドがCFnである（豊富な実績とツールの安定性）

抽象化されたコンストラクタライブラリだけで全部できるわけではない。CDKのコンパイルが通っているかといって、デプロイできるとは限らない。テンプレートを書くのは簡単。

## How to use AWS CDK

一連の流れのデモ。テストを含め。

## CDKのコンセプト

最上位はApp
その下に複数Stack

## CDK DiveDeep

### テスト

Testing infrastructure with the aws cloud development kit

- Snapshot test（golden master tests）
- Fine-grained test code

### CDK 複数スタックの管理

スタック間でConstructを参照すればOK。
スタックの依存順序が定義可。

### CDKアクセス許可の考え方

grantRead()、grantWrite()、grantReadWrite()

### CDKパラメータ

- Environment
- Context
- SSM ParameterStore





