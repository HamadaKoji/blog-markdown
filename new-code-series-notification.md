DevOpsやCI/CDの根幹をになうサービスとして、すっかりおなじみになった

- <a href="https://aws.amazon.com/jp/about-aws/whats-new/2019/11/introducing-notifications-for-aws-codecommit-aws-codebuild-aws-codedeploy-and-asw-codepipeline/" target="_blank">Introducing notifications for AWS CodeCommit, AWS CodeBuild, AWS CodeDeploy, and AWS CodePipeline</a>

<pre style="line-height:120%;">
Codeシリーズええ感じ通知サービスきたか…!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>



## 新notifications（通知ルール）の概要

AWSのいわゆるCodeシリーズに特化した通知ルールサービスで、AWS CodeCommit、AWS CodeBuild、AWS CodeDeploy、AWS CodePipelineなどのSNSなどへの通知を一括管理できます。


この通知ルール自体は無料で利用でき、この通知ルールから呼び出される<a href="https://aws.amazon.com/jp/sns/" target="_blank">Amazon SNS</a><a href="https://aws.amazon.com/jp/chatbot/" target="_blank">AWS Chatbot</a>に対してのみ課金されるとのこと。



### 従来あったCloudWatch Eventsとは何が違うのか？

（あとで）

## 新notifications（通知ルール）の概要

今回リリースされたNotificationsは、Developer Tools配下にあるサービスということで、いわゆるひとつの開発系ツールである以下のCodeシリーズに特化した通知サービスです。

- AWS CodeBuild
- AWS CodeCommit
- AWS CodeDeploy
- AWS CodePipeline
- AWS CodeStar

開発者は、ソースコードリポジトリやビルドプロジェクトやデプロイツールやCI/CDパイプラインの様々な通知を簡単に設定することができ、承認のための通知やエラーメッセージを統一的に管理できます。

### ドキュメント一式

この新機能の詳細ですが、今回のリリースに伴って、新たに以下のドキュメントカテゴリーが新規公開されています。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://docs.aws.amazon.com/en_pv/codestar-notifications/latest/userguide/what-is-dtconsole.html" target="_blank">What Is the Developer Tools Console? - Developer Tools Console</a>
</p>

「あれ、こんなドキュメントもともとあったっけ？」と思ったんですが、<a href="https://docs.aws.amazon.com/en_pv/codestar-notifications/latest/userguide/doc-history.html" target="_blank">Document History - Developer Tools Console</a>を参照すると、2019年11月5日にリリースと記載があり、今回の通知サービスのリリースにともない公開されたものとわかります。いきなりやな！

この中の<a href="https://docs.aws.amazon.com/en_pv/codestar-notifications/latest/userguide/welcome.html" target="_blank">What Are Notifications? - Developer Tools Console</a>から、今回リリースされたNotificationsについての詳細が記載されています。

Webコンソールだと、Codeシリーズを選択したときに表示されるこの「開発者用ツール」が対応しているようです。

- <a href="https://ap-northeast-1.console.aws.amazon.com/codesuite/home?region=ap-northeast-1" target="_blank">AWS Developer Tools</a>

010.png

## CodeCommitからの通知を設定してみる

どんな感じになるのか、試しにCodeCommitからの通知を設定してみます。

### SNSトピックの作成

最初に今回設定する通知サービスで使うためのSNSトピックを追加します。既存で使っているものでも大丈夫ですが、今回は新規で<code>DeveloperNotificationsTopic</code>という名前でトピックを作っておきます。

一点、アクセスポリシーですが、デフォルトだとトピックの所有者のみがトピックへの発行が可能となっていますが、簡単に設定するのであれば「全員」を選択し、Notificationsサービスからトピック発行ができるようにしておきましょう。

013

ポリシーは具体的には、Serviceに<code>codestar-notifications.amazonaws.com</code>を追加する必要があります。CodeStar系のサービスなんですね。ポリシーの詳細な設定方法は、<a href="https://docs.aws.amazon.com/en_pv/codestar-notifications/latest/userguide/set-up-sns.html" target="_blank">Configure Amazon SNS Topics for Notifications</a>を参考にしてみてください。

#### 既存のSNSトピックを使うときの注意点

<p class="alert">ターゲットには既存のSNSトピックも使えますが、既存のものはCodeCommitの通知には使えないなど制限があるようなので、新規でSNSトピックを作成することをオススメします。</p>

<blockquote>It has not been used for sending notifications for CodeCommit before November 5, 2019.<br ><p>引用：<a href="https://docs.aws.amazon.com/en_pv/codestar-notifications/latest/userguide/set-up-sns.html">Configure Amazon SNS Topics for Notifications - Developer Tools Console</a></p></blockquote>

### CodeCommitからの通知設定

手順は、<a href="https://docs.aws.amazon.com/en_pv/codestar-notifications/latest/userguide/getting-started-repository.html" target="_blank">Create a Notification Rule for a Repository</a>に記載されています。

CodeCommitで、作成済みの適当なリポジトリを選択します。リポジトリにファイルが有れば、上部に「通知」に関するドロップダウンが表示されています。

015

また、左側メニューに並んでいる「設定」クリックからの、「通知」タブからも通知作成画面に遷移できます。

020

030

設定自体は特に難しいものではなさそうです。通知名、通知に含まれる内容、トリガーイベント、ターゲットになるSNSトピックを選択して設定します。ターゲットはSNSトピック以外にも複数設定できます。

040

通知一覧に、先程作成した通知が表示されていればOKです。

### 実際に通知を確認

通知のテストをします。とりあえずWebコンソールで<code>feature-notification-test</code>ブランチを作成してみたところ、以下のJSONが通知されてきました。

[json]
{
	"account": "629895769338",
	"detailType": "CodeCommit Repository State Change",
	"region": "ap-northeast-1",
	"source": "aws.codecommit",
	"time": "2019-11-10T07:58:13Z",
	"notificationRuleArn": "arn:aws:codestar-notifications:ap-northeast-1:629895769338:notificationrule/1633bc86-5911-42b5-a919-f51792511d35",
	"detail": {
		"referenceFullName": "refs/heads/feature-notification-test",
		"repositoryId": "efcc56f3-e072-4366-bcfb-01909e030545",
		"referenceType": "branch",
		"commitId": "d1dc42514fe14a069db6557426a339f433ae3915",
		"callerUserArn": "arn:aws:sts::629895769338:assumed-role/cm-hamada.koji/1573371954495722000",
		"event": "referenceUpdated",
		"repositoryName": "proxy",
		"oldCommitId": "147c11b41dd3021fc27b74d927daa49c95517106",
		"referenceName": "feature-notification-test"
	},
	"resources": [
		"arn:aws:codecommit:ap-northeast-1:629895769338:proxy"
	],
	"additionalAttributes": {}
}
[/json]

また、プルリクエストを作成したときの通知も参考までにはっておきます。こちらでは、プルリクエストのタイトルと共に、マージ元、マージ先のブランチ名、マージの有無などの詳細情報を知ることができます。

[json]
{
	"account": "629895769338",
	"detailType": "CodeCommit Pull Request State Change",
	"region": "ap-northeast-1",
	"source": "aws.codecommit",
	"time": "2019-11-10T08:03:17Z",
	"notificationRuleArn": "arn:aws:codestar-notifications:ap-northeast-1:629895769338:notificationrule/1633bc86-5911-42b5-a919-f51792511d35",
	"detail": {
		"sourceReference": "refs/heads/feature-notification-test",
		"lastModifiedDate": "Sun Nov 10 08:03:05 UTC 2019",
		"author": "AROAJBAZUR7WF5N7MZUAS:cm-hamada.koji",
		"isMerged": "False",
		"pullRequestStatus": "Open",
		"notificationBody": "A pull request event occurred in the following AWS CodeCommit repository: proxy. arn:aws:sts::629895769338:assumed-role/cm-hamada.koji/cm-hamada.koji made the following PullRequest 2. The pull request was created with the following information: Pull Request ID as 2 and title as さらに修正した分です。. For more information, go to the AWS CodeCommit console https://ap-northeast-1.console.aws.amazon.com/codesuite/codecommit/repositories/proxy/pull-requests/2?region=ap-northeast-1",
		"destinationReference": "refs/heads/master",
		"callerUserArn": "arn:aws:sts::629895769338:assumed-role/cm-hamada.koji/cm-hamada.koji",
		"creationDate": "Sun Nov 10 08:03:05 UTC 2019",
		"pullRequestId": "2",
		"title": "さらに修正した分です。",
		"revisionId": "53878bb564848da194cd6466578ebd3bf15ab74eaa1f666a5d6d5c9293473c62",
		"repositoryNames": [
			"proxy"
		],
		"destinationCommit": "147c11b41dd3021fc27b74d927daa49c95517106",
		"event": "pullRequestCreated",
		"sourceCommit": "d1dc42514fe14a069db6557426a339f433ae3915"
	},
	"resources": [
		"arn:aws:codecommit:ap-northeast-1:629895769338:proxy"
	],
	"additionalAttributes": {}
}[/json]

### SNSからの通知が届かない場合の対処方法

以下を参考にしてみてください。だいたいポリシー周辺の設定が原因だと思います。

- <a href="https://docs.aws.amazon.com/en_pv/codestar-notifications/latest/userguide/troubleshooting.html#troubleshooting-no-SNS" target="_blank">Troubleshooting - I am not receiving Amazon SNS notifications</a>

## 取得できるイベントの一覧

今回の新通知システムで利用できるCodeシリーズのイベント一覧です。

### CodeCommit

<a href="https://docs.aws.amazon.com/en_pv/codestar-notifications/latest/userguide/concepts.html#events-ref-repositories" target="_blank">Events for Notification Rules on Repositories</a>

|  **Category** | **Events** | **Event IDs** |
| :--- | :--- | :--- |
|  Build state | Failed<br/>Succeeded<br/>In-progress<br/>Stopped | codebuild-project-build-state-failed<br/>codebuild-project-build-state-succeeded<br/>codebuild-project-build-state-in-progress<br/>codebuild-project-build-state-stopped |
|  Build phase | Failure<br/>Success | codebuild-project-build-phase-failure<br/>codebuild-project-build-phase-success |

### CodeBuild

<a href="https://docs.aws.amazon.com/en_pv/codestar-notifications/latest/userguide/concepts.html#events-ref-buildproject" target="_blank">Events for Notification Rules on Build Projects</a>

|  **Category** | **Events** | **Event IDs** |
| :--- | :--- | :--- |
|  Build state | Failed<br/>Succeeded<br/>In-progress<br/>Stopped | codebuild-project-build-state-failed<br/>codebuild-project-build-state-succeeded<br/>codebuild-project-build-state-in-progress<br/>codebuild-project-build-state-stopped |
|  Build phase | Failure<br/>Success | codebuild-project-build-phase-failure<br/>codebuild-project-build-phase-success |

### CodeDeploy

<a href="https://docs.aws.amazon.com/en_pv/codestar-notifications/latest/userguide/concepts.html#events-ref-deployapplication" target="_blank">Events for Notification Rules on Deployment Applications</a>

|  **Category** | **Events** | **Event IDs** |
| :--- | :--- | :--- |
|  Deployment | Failed<br/>Succeeded<br/>Started | codedeploy-application-deployment-failed<br/>codedeploy-application-deployment-succeeded<br/>codedeploy-application-deployment-started |


### CodePipeline

<a href="https://docs.aws.amazon.com/en_pv/codestar-notifications/latest/userguide/concepts.html#events-ref-pipeline" target="_blank">Events for Notification Rules on Pipelines</a>

|  **Category** | **Events** | **Event IDs** |
| :--- | :--- | :--- |
|  Action execution | Succeeded<br/>Failed<br/>Canceled<br/>Started | codepipeline-pipeline-action-execution-succeeded<br/>codepipeline-pipeline-action-execution-failed<br/>codepipeline-pipeline-action-execution-canceled<br/>codepipeline-pipeline-action-execution-started |
|  Stage execution | Started<br/>Succeeded<br/>Resumed<br/>Canceled<br/>Failed | codepipeline-pipeline-stage-execution-started<br/>codepipeline-pipeline-stage-execution-succeeded<br/>codepipeline-pipeline-stage-execution-resumed<br/>codepipeline-pipeline-stage-execution-canceled<br/>codepipeline-pipeline-stage-execution-failed |
|  Pipeline execution | Failed<br/>Canceled<br/>Started<br/>Resumed<br/>Succeeded<br/>Superseded | codepipeline-pipeline-pipeline-execution-failed<br/>codepipeline-pipeline-pipeline-execution-canceled<br/>codepipeline-pipeline-pipeline-execution-started<br/>codepipeline-pipeline-pipeline-execution-resumed<br/>codepipeline-pipeline-pipeline-execution-succeeded<br/>codepipeline-pipeline-pipeline-execution-superseded |
|  Manual approval | Failed<br/>Needed<br/>Succeeded | codepipeline-pipeline-manual-approval-failed<br/>codepipeline-pipeline-manual-approval-needed<br/>codepipeline-pipeline-manual-approval-succeeded |


## 新機能のNotificationsがCloudWatch Eventsと違う点

このアップデートを見たときに最初に自分も考えました。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">えーとこれはなんだ。昔からCWEventsここらへんあったと思うんだけど、別の仕組みってことなんすか？それはそれですげぇぞ。</p>&mdash; 濱田孝治（ハマコー） (@hamako9999) <a href="https://twitter.com/hamako9999/status/1192966075366772737?ref_src=twsrc%5Etfw">November 9, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

ここまでブログ書いて新機能のNotifications一通り動かしてみて、感じた点まとめてみます。

### AWS Chatbotと連携できす

一番大きく違う点だと思うのですが、NotiricationsはAWS Chatbotと連携できます。AWS Chatbotは2019年11月6日現在、まだベータ版ですが利用できます。

AWS Chatbotを利用すると、簡単にAmazon ChimeもしくはSlackにNotificationsの内容を連携できます。従来はSlackに通知内容などを連携する場合、SNSからLambdaを呼び出してそこからSlackに通知することが必須でしたが、そのあたりが全部不要になります。

すでにドキュメントも提供されているので、試してみたい方は是非どうぞ。一般提供が待ち遠しい。

- <a href="https://aws.amazon.com/jp/chatbot/" target="_blank">AWS Chatbot | AWS</a>

NotificationsとAWS Chatbotとの連携方法はこちら。

- <a href="https://docs.aws.amazon.com/en_pv/codestar-notifications/latest/userguide/notifications-chatbot.html" target="_blank">Configure Integration Between Notifications and AWS Chatbot - Developer Tools Console</a>



### CloudWatch Eventsのイベントパターンより拡充されている

例えば、CodePipelineについてですが、CloudWatch Eventsで定義されているイベントタイプは以下の３種類です。

- CodePipeline Pipeline Execution State Change
- CodePipeline Stage Execution State Change
- CodePipeline Action Execution State Change

対して、今回追加された新機能のNotificationsでは、CodePipelineのイベントタイプに<code>Manual approval</code>が追加されていて、承認周辺のイベントも取得できるようになっています。

もちろんCloudWatch EventsでもCloudTrailを取得元にすることで全API呼び出しを取得することができましたが設定が複雑で使いづらかったのも事実です。今回、Notificationsとしてまとめられたことで、通知イベントが拡充され全体的に設定が簡素化されています。

### CloudWatch Eventsのリソースを作る必要がない

これは当たり前ですが、Notificationsは、各Codeシリーズのリソースに通知とそのターゲットを直接指定するため、別途CloudWatch Eventsのリソース作成が不要です。

### CloudWatch Eventsの料金が不要

Notifications自体は無料です。対してCloudWatch Eventsには、イベント100万件あたり1.00USDかかります。まぁCodeシリーズで使うにあたって、ここの課金が問題になることはほぼ無いと思いますが、Notificationsが無料で使えるのはメリットですね。

### 設定が簡単

CloudWatch Eventsで特定リソースの通知を設定する場合、イベントパターンのJSONの中の<code>resources</code>に、そのリソースのarnを指定する必要があります。複数リソースを対象とする場合、ここを併記したりなどそれなりに設定がめんどくさいです。

対して、Notificationsでは、最初に通知を設定する対象のリソースを選択し、そのリソースに対して必要な通知を指定するので設定が直感的です。

### 専用の管理コマンドが用意されている

今回、この機能のために専用の<code>codestar-notifications</code>AWS CLIコマンドが新規追加されています。

- <a href="https://docs.aws.amazon.com/cli/latest/reference/codestar-notifications/index.html" target="_blank">codestar-notifications — AWS CLI 1.16.278 Command Reference</a>

イベントタイプや通知ルールの一覧の出力や、通知ルールの一括更新、通知送信の一時停止など一通りのオプションが用意されているので、Codeシリーズの通知管理という面では、こちらが圧倒的に優れています。

### 文字列のカスタマイズができない

CloudWatch Eventsには、通知先のターゲットに対して文字列を編集するためのインプットトランスフォーマーという機能があります（参考：<a href="https://dev.classmethod.jp/cloud/cloudwatch-events-inputtransfer/" target="_blank">[新機能] Amazon CloudWatch EventsのInput Transformerを試してみた ｜ Developers.IO</a>）。

これを使えば、通知内容をJSON垂れ流しではなく、任意のメッセージに変更することが可能です。対して、Notificationsにはこの機能は今の所見当たりません。

代わりかどうかはわかりませんが、Notificationsには、通知の記載内容を2つの種類から選択できます。

- フル
  - 「リソースまたは通知機能によって提供されるイベントに関する補足情報が含まれます。」
- ベーシック
  - 「リソースイベントで提供される情報のみが含まれます」

通知先のセキュリティポリシーに応じて、表示内容をどちらにするかを選択する必要があります（上の実行例で示したJSONはフルのものです）。このあたりの考え方も、公式ドキュメントに記載されているので、合わせてご確認ください。

- <a href="https://docs.aws.amazon.com/codestar-notifications/latest/userguide/security.html#security-notifications" target="_blank">Security - Developer Tools Console</a>

## Codeシリーズに特化した通知サービスでアプリ開発〜運用をもっと便利に

もともと、CloudWatch EventsやCloudTrailでも実現できていた内容が、Codeシリーズに特化した形で新機能としてリリースされたのは、面白いですね。正直意外でした。

設定内容は非常にわかりやすく、従来のCodeシリーズに非常に馴染んでいるので、CloudWatch Eventsの設定がめんどくさいなぁと思っていた人にも、手軽に運用できると思います。

Codeシリーズは、その名の通り、AWSにおけるアプリケーション開発や運用のベースとなるサービスであり、CI/CDやDevOpsの文脈でその利用状況を通知で一括管理したいというニーズは昔からあったので、そこが、わかりやすいUIで提供されているのは一つの大きな進化じゃないでしょうか。

また、AWS ChimeやSlackと連携できるAWS Chatbotが組み込まれている点も楽しみなところです。まだ、AWS Chatbotはβ版が提供されているのみですが、はれて一般提供されたら、改めて動作検証してみようと思います。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

