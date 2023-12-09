<strong>「ECS(Fargate)のランタイム脅威検知って、商用製品必須だよね。どれにすっかなぁ」</strong>

<strong>「1年の時を経てようやくGAですよ、どすこい！！」</strong>

去年のre:Invent2022において、予告だけされていたコンテナ環境のランタイム検知（参考：[【速報】GuardDutyによるコンテナランタイムの脅威検知サービスが発表されました！](https://dev.classmethod.jp/articles/container-runtime-threatdetection-guardduty/)）。

先日のre:Invent 2023におけるアップデートにより、待望のECS(Fargate)環境におけるランタイム検知が、GuardDutyで実現できるようになりました！！

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="Detect runtime security threats in Amazon ECS and AWS Fargate, new in Amazon GuardDuty | AWS News Blog" src="https://hatenablog-parts.com/embed?url=https://aws.amazon.com/jp/blogs/aws/introducing-amazon-guardduty-ecs-runtime-monitoring-including-aws-fargate/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

これまでは、ECS(Fargate)環境においては、ランタイム検知は商用製品の導入が必須でしたが、今回のアップデートにより、AWSのマネージドな仕組みだけで、ランタイム検知ができるようになっています。実際の設定の様子は、こちらの[トクヤマシュン](https://dev.classmethod.jp/author/toku-shun/)のブログを参考にしてください。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="[アップデート]Amazon GuardDuty ECS Runtime Monitoringが利用できるようになりました #AWSreinvent | DevelopersIO" src="https://hatenablog-parts.com/embed?url=https://dev.classmethod.jp/articles/guardduty-ecs-runtime-monitoring/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

今回のブログでは、以下の順番でお伝えします。これから、ECS(Fargate)環境で実際にランタイム脅威検知を設定しようと思った方向けの情報を全て盛り込んだので、ゆっくり眺めてもらえればと思います。ほないってみよ。

- ランタイム脅威検知設定前に必ず確認しておくべき諸注意
- （設定前に）特定クラスターをGuardDutyの脅威検知対象外にする方法
- ランタイム脅威検知のGuardDutyでの設定手順
- 動作確認用のECSタスクの起動
- （動作検証前に）GuardDutyが検出する脅威タイプの一覧
- 実際にECS Execを利用して、脅威検知してみる
- 実際に脅威を検知したあとはどうするのか？

<pre style="line-height:120%;">
なんでもかんでも検知きたか…!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>

なんでもかんでもとは言ってないし、検知後の運用も考えてね。

## ランタイム脅威検知設定前に必ず確認しておくべき諸注意

これから説明しますが、GuardDutyによるECS(Fargate)のランタイム脅威検知、設定自体はものすごく簡単です。まじで、ポチポチっとめっちゃ簡単です。なのですが、<strong>簡単なだけに影響範囲が大きく環境によってはリソース、料金、権限面で思わぬ影響が出る可能性もあるので、事前に以下の注意点を頭に入れておきましょう。</strong>

### ECSタスクのリソースについて

GuardDutyによるランタイム脅威検知には、AWSがマネージドで管理するサイドカーコンテナのエージェントが必要になります。これは、既存のタスク定義の中に自動的に追加されるコンテナで、ユーザー側がタスク定義で既定しているリソース（vCPU、Memory）の組み合わせによって、GuardDutyのエージェントが利用するメモリーが決まります。

サイドカーコンテナが利用するリソースの詳細：[CPU and memory limits](https://docs.aws.amazon.com/guardduty/latest/ug/prereq-runtime-monitoring-ecs-support.html#ecs-runtime-agent-cpu-memory-limits)

上記公式に記載があるとおり、リソースとしてそこまで気にするメモリ量ではないと思いますが、既存のタスクの性能に影響があることは頭に入れておきましょう。

### サイドカーコンテナの料金について

公式：[Intelligent Threat Detection – Amazon GuardDuty Pricing](https://aws.amazon.com/guardduty/pricing/)

Runtime Monitoringを確認すると、以下の記載があります。

- ECS Runtime Monitoring	
  - First 500 vCPUs / month (for monitored instances)	$1.50 per vCPU
  - Next 4,500 vCPUs / month (for monitored instances)	$0.75 per vCPU
  - Over 5,000 vCPUs / month (for monitored instances)	$0.25 per vCPU

東京リージョンのx86アーキテクチャのFargateの1ヶ月のランニング料金が1vCPUあたり約$38なので、それと比べるとざっくり4%ぐらいのコスト増になります。これも頭の片隅に置いておきましょう。

### サイドカーコンテナを利用するための権限について

公式：[Before enabling Runtime Monitoring \- Amazon GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/runtime-monitoring-before-enable-runtime-monitoring-ecs.html)

GuardDutyのサイドカーエージェントをECRからダウンロードするために、[AmazonECSTaskExecutionRolePolicy](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AmazonECSTaskExecutionRolePolicy.html)が、タスク実行ロールに必要となります。大抵の場合、タスク実行ロールには、このポリシーが含まれる<code>ecsTaskExecutionRole</code>を利用している人も多いと思いますが、以下の権限が含まれていることを確認しておく必要があります。

[json]
      ...
      "ecr:GetAuthorizationToken",
      "ecr:BatchCheckLayerAvailability",
      "ecr:GetDownloadUrlForLayer",
      "ecr:BatchGetImage",
      ...
[/json]

### その他前提条件

その他、利用にあたっての制限事項です。

公式：[Prerequisites for AWS Fargate \(Amazon ECS only\) support \- Amazon GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/prereq-runtime-monitoring-ecs-support.html)

- Fargateのバージョンは<code>1.4.0</code>以上、もしくは<code>latest</code>
- OS distribution
  - Linux
- Kernel support
  - eBPF, Tracepoints, Kprobe
- CPU architecture
  - x64 (AMD64)
  - Graviton (ARM64)

Windowsコンテナは未サポート。CPUアーキテクチャは、Gravitonにも対応しているのが良いですね。Fargateのプラットフォームバージョンは現時点での最新が必要です。


## （設定前に）特定クラスターをGuardDutyの脅威検知対象外にする方法

前節でGuardDutyランタイム脅威検知の設定前に確認しておくべき点を解説しましたが、設定から除外したいクラスターがある場合は、事前に以下のタグをECSクラスターに付与することで、ランタイム脅威検知の設定から除外することができます。

参考：[Managing the security agent for Fargate \(Amazon ECS only\) \- Amazon GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/managing-gdu-agent-ecs-automated.html)）。

- Key:<code>GuardDutyManaged</code>
- Value:<code>false</code>

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<strong>改めてこの設定は重要です。</strong>GuardDutyのランタイム脅威検知設定は、設定したアカウントのリージョンの全てのECSクラスターに影響します。設定後起動するECSタスクには、自動的にGuadDutyサイドカーエージェントが追加で起動することになるので、動作確認が取れていないECSクラスターには、必ずこのタグを付与しておき、脅威検知対象外にしておくことを推奨します。
</p>

## ランタイム脅威検知のGuardDutyでの設定手順

以上、長々と設定前の考慮事項を書きましたが、実際に設定していきましょう。ほんま、拍子抜けするぐらい簡単。

設定手順：[Enabling Runtime Monitoring for a standalone account \- Amazon GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/enable-untime-monitoring-standalone-acc.html)

GuardDutyのコンソールにアクセスして、左側メニューの[Protection plans] -> [Runtime Monitoring]を選択。

010

[Runtime Monitoring]と[AWS Fargate(ECS only)]を[Enable]設定すれば、これだけで完了です。

020

## 動作確認用のECSタスクの起動

設定が完了したので、実際にECSクラスターを作成し、タスクを起動して動作を検証していきます。ここでは、[AWS Copilot CLI](https://aws.github.io/copilot-cli/ja/)を利用して、ECSクラスター作成していきます。

その他、関連するツールのバージョンは以下の通り。コンテナビルド用のツールは、AWS謹製の[finch](https://github.com/runfinch/finch)を利用しています。必要なツールは事前にインストールしておいてください。

[bash]
$ sw_vers
ProductName:            macOS
ProductVersion:         13.4.1
BuildVersion:           22F82

$ aws --version
aws-cli/2.14.6 Python/3.11.6 Darwin/22.5.0 exe/x86_64 prompt/off

$ copilot --version
copilot version: v1.32.0

$ finch --version
finch version v1.0.0
[/bash]

### 動作検証用クラスターの作成

<code>test-runtime-detection-repo</code>という名前で、ECRを作成します。

[bash]
$ aws ecr create-repository --repository-name test-runtime-detection-repo
[/bash]

Finchを使って、ECRにログインします。<code>aws_account_id</code>は、皆さんの環境に合わせて書き換えてください。

[bash]
$ aws ecr get-login-password | finch login --username AWS --password-stdin <aws_account_id>.dkr.ecr.region.amazonaws.com
[/bash]

copilotのページで提供されているサンプルアプリケーションをダウンロード。

[bash]
$ git clone https://github.com/aws-samples/aws-copilot-sample-service example
$ cd example
[/bash]

サンプルアプリケーションのDockerfileをビルドしタグ付けした後、ECRにプッシュ。<code>account_id</code>は、皆さんの環境に合わせて書き換えてください。

[bash]
$ docker build -t test-runtime-detection-repo .
$ docker tag test-runtime-detection-repo:latest <account_id>.dkr.ecr.ap-northeast-1.amazonaws.com/test-runtime-detection-repo:latest
$ docker push <account_id>.dkr.ecr.ap-northeast-1.amazonaws.com/test-runtime-detection-repo:latest
[/bash]

ここまででコンテナイメージとECRへのプッシュが完了したら、その内容を受けて、copilotを利用して、ECSクラスターを作成します。copilotの基本的な使い方は、[Deploy your first application](https://aws.github.io/copilot-cli/ja/docs/getting-started/first-app-tutorial/)を参考にしてください。

<code>copilot init</code>コマンド後、対話形式でECSクラスターが作成できます。以下、設定したパラメーターを参考までに記します。途中コンテナイメージのURLを聞かれるので、前手順でPushしたイメージのURIを記入してください（dockerを使っている場合は、ここでローカルのDockerfileのビルドも同時に実施してくれますが、dockerがインストールされていない環境の場合、ECRのURLを指定する必要があります）。また、ランタイム脆弱性の検証にはECS Execを利用してコンテナにログインするため、Workload typeはBackend Serviceを選択しています。

[bash]
$ copilot init

Use existing application: No
Application name: test-runtime-detection-app
Workload type: Backend Service
Image: <account_id>.dkr.ecr.ap-northeast-1.amazonaws.com/test-runtime-detection-repo:latest
Port: 80
Deploy: Yes
Only found one option, defaulting to: Create a new environment
Environment name: test-runtime-env
[/bash]

### タスク起動後のリソース確認

無事、ECSサービスの作成とタスクの起動が正常に終了すると、起動したタスクの中に、GuardDutyのagentのコンテナが含まれていることがわかります。

030.png

GuardDutyのマネジメントコンソールから、左側メニューの[Protection plans] -> [Runtime Monitoring]を開き、[Runtime coverage]タブの[ECS cluster runtime coverage]を参照すると、エージェントが展開されているクラスターを参照できます。

040

また、クラスターが展開されているVPCには、GuardDutyサービス向けのVPCエンドポイントが自動的に追加されていることも確認できます。

050

### ECS Execの動作確認

この後の手順で、実際に脅威検知の動作検証をしますが、いちばん検証が簡単なのはコンテナにシェルログインする方法です。今回はECS(Fargate)のコンテナなので、起動したECSタスクにECS Execでログインできるように準備します。ここもかなり手間がかかる作業ではありますが、以下のドキュメントを参考に各環境でECS Execするための環境を準備してください。

参考：[Using Amazon ECS Exec for debugging \- Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/userguide/ecs-exec.html)

起動したタスクがECS Execを利用可能か？だめな場合は、どの設定がNGなのか？は、こちらのツールを使うと網羅的に把握することができるので、合わせての利用をおすすめします。

[GitHub \- aws\-containers/amazon\-ecs\-exec\-checker: 🚀 Pre\-flight checks for ECS Exec](https://github.com/aws-containers/amazon-ecs-exec-checker)


## （動作検証前に）GuardDutyが検出する脅威タイプの一覧

ここまでで、ランタイム脅威検出の動作準備は完了です。脅威検出を確認する前に、実際GuardDutyがどのような脅威を検出するのかおさらいしておきます。GuaredDutyが検知する全ての脅威タイプの一覧はこちら。

[Finding types \- Amazon GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-active.html)

今回のECS(Fargate)環境で検知可能なランタイムモニタリングの脅威対象は、以下の条件に該当。

- [Resource type]列：<code>Container</code>
- [Foundational data source/Feature]列が<code>Runtime Monitoring</code>

このうち、ランタイム脅威に絞った一覧がこちら。2023年12月現在で、31種類の脅威が用意されています。

[Runtime Monitoring finding types \- Amazon GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/findings-runtime-monitoring.html)

以下、ざっくりした要約。眺めてみることでどういった内容がランタイム脅威として検知されうるのかわかります。実際に本番環境でこのような挙動がコンテナで実施された場合を想像すると、背筋が凍る人も多いのではないでしょうか。

1. **通常ではないネットワークトラフィック**  
   コンテナからの予期しないアウトバウンド接続。

2. **ポートスキャン活動**  
   コンテナが他のシステムやサービスのポートをスキャン。

3. **既知の悪意のあるIPまたはドメインへのアクセス**  
   コンテナが既知の悪意あるIPやドメインと通信。

4. **予期せぬデータ量やパターン**  
   データ転送量やパターンの大幅な変化。

5. **疑わしいファイルやプロセスの活動**  
   コンテナ内での通常ではないファイルの変更やプロセスの実行。

6. **異常なユーザー行動**  
   コンテナ内からの通常ではないログインパターンや特権昇格試み。

7. **侵害されたコンテナイメージ**  
   既知の脆弱性がある、または悪意のあるコンテナイメージの使用。

8. **暗号通貨マイニング**  
   CPUまたはGPU使用率の予期しない急増。

9. **コマンド＆コントロール（C&C）通信**  
   既知のC&C通信プロトコルに一致するトラフィックパターン。

10. **リバースシェルまたは不正なリモートアクセス**  
   リバースシェルの作成や不正なリモートアクセス試みの兆候。


## 実際にECS Execを利用して、脅威検知してみる

いよいよ、実際にECS Execを利用して、コンテナにログインしいくらかコマンド実行して脅威検出させてみた例を紹介。脅威を検知するといっても、パブリックなインターネットで周囲に脅威を与えることは推奨されないので、コマンド自体はリスクが低いもので試しています。

実際に脅威検知されると、GuadDutyのFindingsの一覧に、このように検知結果が表示されます。

070.png

### 脅威検知例「暗号通貨関連ドメインへのDNSクエリ」

ECS Execでコンテナ内から実行したコマンド。

[bash]
# yum -y install bind-utils
# dig pool.supportxmr.com +short
[/bash]

脅威検知されたFindings：[CryptoCurrency:Runtime/BitcoinTool.B!DNS](https://docs.aws.amazon.com/guardduty/latest/ug/findings-runtime-monitoring.html#cryptocurrency-runtime-bitcointoolbdns)

暗号通貨関連のアクティビティに関連付けられているドメイン名へのDNSクエリにより、この脅威が検知されました。

このドメイン<code>pool.supportxmr.com</code>は、弊社[臼田](https://dev.classmethod.jp/author/usuda-keisuke/)の[GuardDutyのEC2タイプFindingsを簡単に検知させる方法 – CryptoCurrency編](https://dev.classmethod.jp/articles/guardduty-findings-test-cryptocurrency/)でも紹介されているものです。

同じような動作をする他のドメインを調査したのですが、GuardDutyの具体的な脅威検知のロジックは非公開のようで、具体的なドメインを見つけられませんでした。ChatGPTにも聞いた内容を、以下に引用します。納得の理由ですね。

<blockquote>AWS GuardDutyの「CryptoCurrency:Runtime/BitcoinTool.B!DNS」脅威検出は、暗号通貨マイニングに関連する特定のDNSクエリを識別します。これらのクエリは通常、暗号通貨マイニングプールのドメインに対して行われます。
<br /><br />
ただし、特定のドメイン名を挙げることは、そのドメインの正当性に影響を及ぼす可能性があるため、具体的なドメイン名を提供するのは適切ではありません。一般的に、GuardDutyが「CryptoCurrency:Runtime/BitcoinTool.B!DNS」で検出する可能性のあるドメインは、以下のような特徴を持つものです：
<br /><br />
- 暗号通貨マイニングプールに関連するドメイン。<br />
- マイニングに使用される既知のサーバーのドメイン。<br />
- 暗号通貨マイニングソフトウェアによって頻繁にアクセスされるドメイン。<br />
<br />
重要なのは、GuardDutyの検出はドメイン名だけでなく、ネットワークトラフィックのパターンやその他のコンテキスト情報に基づいているということです。したがって、単に特定のドメインにアクセスしたからといって、必ずしも脅威があるとは限りません。<br /><br />
セキュリティ上の理由から、特定のドメインを公開することは避けるべきです。また、GuardDutyの警告やアラートに基づいて行動を起こす場合は、適切な調査と分析を行うことが重要です。</blockquote>


### 脅威検知例「コンテナ内新バイナリの実行」

ECS Execでコンテナ内から実行したコマンド。

[bash]
# dig example.com +short
93.184.216.34

# yum install -y nmap

# nmap -Pn -p 80,443 93.184.216.34
Starting Nmap 6.40 ( http://nmap.org ) at 2023-12-09 16:34 JST
Nmap scan report for 93.184.216.34
Host is up (0.11s latency).
PORT    STATE SERVICE
80/tcp  open  http
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 0.57 seconds
[/bash]

脅威検知されたFindigs:[Execution:Runtime/NewBinaryExecuted](https://docs.aws.amazon.com/guardduty/latest/ug/findings-runtime-monitoring.html#execution-runtime-newbinaryexecuted)

この脅威は、コンテナ内で新しく作成された、または最近変更されたバイナリファイルが実行されたことを検知しています。具体的には、<code>yum</code>でインストールした<code>nmap</code>コマンドの即実行が脅威として検知されています。

コンテナはイミュータブルな状態に保つのがベストプラクティスで、コンテナ実行中に新バイナリがインストールされそれが即実行されることは、コンテナの挙動として非常に疑わしい事態。悪意のある行為者のワークロードに近しいので、GuardDutyで検知対象になったのでしょう。

以上、コンテナランタイムモニタリングの脅威検出結果の例でした。

## 実際に脅威を検知したあとはどうするのか？

以上、長々と説明してきたコンテナランタイムの脅威検知ですが、実際に脅威を検知したあとどうするのか？は、プロダクション環境で運用するさいに事前に考えておく必要があります。

AWSの公式ドキュメントには、検出された脅威をどう運用するのか？についての詳細なドキュメントがあるので、参考になります。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="Managing Amazon GuardDuty findings - Amazon GuardDuty" src="https://hatenablog-parts.com/embed?url=https://docs.aws.amazon.com/guardduty/latest/ug/findings_management.html" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

弊社[臼田](https://dev.classmethod.jp/author/usuda-keisuke/)が執筆したこちらのGuardDutyによる検出結果の運用に関する記事も参考になるので、合わせて確認することをオススメします。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="[2021年版]Amazon GuardDutyによるAWSセキュリティ運用を考える | DevelopersIO" src="https://hatenablog-parts.com/embed?url=https://dev.classmethod.jp/articles/aws-security-operation-with-guardduty-2021/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

さらに、弊社[トクヤマシュン](https://dev.classmethod.jp/author/toku-shun/)が、具体的にこのコンテナランタイム脅威検知の結果を利用して、該当タスクを強制停止するサンプルを記述しています。脅威検知結果の自動運用の方法として、非常に参考になる内容なので、こちらも参考にしてください！

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="Amazon GuardDuty ECS Runtime Monitoringで脅威を検出したECSタスクを自動停止してみた | DevelopersIO" src="https://hatenablog-parts.com/embed?url=https://dev.classmethod.jp/articles/guardduty-ecs-runtime-monitoring-stop-task/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

## セキュリティが重要なECS(Fargate)環境における脅威検知手法として、必ず選択肢にいれておきましょう

以上、GuardDutyによるECS(Fargate)環境のランタイム脅威検知について、概要をお伝えしました。実際に脅威検知される内容をみていると、セキュリティ要件が強めの環境においては、脅威検知設定を実施しておくことも一つの選択肢になりうるのでは？と思います。

商用製品には、ランタイム脅威検知とともに、事前に設定しておいたルールに基づいて脅威検知したコンテナの封じ込めとエビデンスの自動取得まで機能として保持しているものもあります。

例：[Aqua Runtime Protection detects sophisticated attacks in real time \- Aqua](https://www.aquasec.com/news/aqua-runtime-protection-detects-sophisticated-attacks-in-real-time/)

今回のGuardDutyによる脅威検知は、基本的には脅威検知までの仕組みとなっており、検知後の運用はEventBridgeなどを用いて独自に作り込む必要があります。そのあたりは、商用製品と比べて運用が手間の部分でもありますが、まずはGuardDutyでの検知結果をSlack通知などすることで、脅威そのものを把握しておくことでも一定の効果は見込めるかなと思います。

今回のこのアップデートが、みなさんのコンテナ環境のさらなる安全につながると良いですね。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

