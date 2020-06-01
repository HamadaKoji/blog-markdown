#（仮）AWS Chatbotを使ってSlackのプライベートチャンネルに連携できないときの対処方法

先日、めでたくGA（一般公開）された<a href="https://aws.amazon.com/jp/chatbot/" target="_blank">AWS Chatbot</a>、皆さん使っていますか？

特にLambdaなどのコーディングは必要なく、AWS上の様々なイベントをSlackにノンコーディングで連携できる非常に便利なサービスなのですが、Slackへのプライベートチャンネルへの連携で、結構はまったので共有です。

まぁ「ドキュメントちゃんと読んどけよ」という話ではあるんですが、検索してもそれっぽい情報がでてこずそれなりにハマってしまったので、インデックス目的で、記事作成しておきます。

<pre style="line-height:120%;">
もう同じことでハマる人いないの？

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>

そう願ってます。

## AWS Chatbotを使ったSlack連携の流れ

パブリックチャンネルに連携する場合の大まかな流れは、下記<a href="https://dev.classmethod.jp/author/fujii-genki/" target="_blank">藤井</a>の記事をご参考ください。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="ついに来た！　AWS Chatbot が一般公開(GA)になりました！　Slack連携が捗ります！ | Developers.IO" src="https://hatenablog-parts.com/embed?url=https://dev.classmethod.jp/articles/aws-chatbot-generally-available/" width="300" height="150" frameborder="0" scrolling="no"></iframe>

大雑把な手順は以下の通り。

- SNS Topicの作成
- 連携用Slackチャンネル作成
- Chat Botの作成
  - Slack WorkSpaceとの連携
  - 連携先チャンネルの指定
  - Chat BotへのIAMロール付与
  - SNS Topicの紐付け

ただ、上記手順で進めていったときに、Slackのパブリックチャンネルには連携されるが、プライベートチャンネルには連携されないという問題が発生しました。

AWS Chatbotはその動作ログをCloudWatch Logsに出力する機能があるんですが、エラーログを見てみるとこんな状態です。

[json]
Encountered error while sending message to Slack: Slack Web API returned unsuccessful response (HTTP status code: 200, ok: false, error code: channel_not_found, full response body: 
{
    "ok": false,
    "error": "channel_not_found"
}
).
[/json]

なんか、権限周辺でエラーになっているっぽい。のでIAMロールなどを見直してみたんですが、問題なさそうでハマることに。


## 解決方法は、連携先SlackチャンネルへのAWSユーザーのInvite

詳細は、公式ドキュメントに記載があります。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://docs.aws.amazon.com/chatbot/latest/adminguide/getting-started.html" target="_blank">Getting started with AWS Chatbot - AWS Chatbot</a>
</p>

<blockquote>If you configure a private Slack channel, run the /invite @AWS command in Slack to invite the AWS Chatbot to the chat room.<br ><p>引用：<a href="https://docs.aws.amazon.com/chatbot/latest/adminguide/getting-started.html#chat-client-setup">Getting started with AWS Chatbot - AWS Chatbot</a></p></blockquote>

というわけで、プライベートチャンネルへ連携する場合は、別途チャンネルにAWSユーザーを<code>invite</code>する必要があったというわけです。

こんな感じでコマンドうつと、そのチャンネルにAWSユーザーが招待されます。

010

後は、動かしてみてパブリックチャンネルと同じように通知されればOK。

## AWS Chatbot、使えるところたくさんあるので、どんどこ使いましょう

AWS Chatbot、CloudWatch Alarmの通知の場合、メール連携では確認できないような、メトリクスのグラフも通知先で見えたりとか、Lambdaが不要というアドバンテージ以上に通知の見える化、利便性に大いに使えるので、みなさんもどんどん使ってみてはいかがでしょうか。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。












