# EC2のLaunchTemplates（起動テンプレート）に対するタグ付けがサポートされました！

皆さん、EC2の起動テンプレート機能、使っていますか？

先日のアップデートで、この起動テンプレートによるEC2へのタグ付ができるようになりました！

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://aws.amazon.com/jp/about-aws/whats-new/2019/07/amazon-ec2-now-supports-tagging-launch-templates-on-creation/" target="_blank">Amazon EC2 Now Supports Tagging Launch Templates on Creation</a>
</p>

<pre style="line-height:120%;">
タグ付け祭りきたか…!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>

## 起動テンプレートとは？

<a href="https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ec2-launch-templates.html" target="_blank">起動テンプレートからのインスタンスの起動 - Amazon Elastic Compute Cloud</a>

簡単に言うと、<strong>インスタンスを起動するための設定情報を含むテンプレートを作成する機能</strong>です。通常、EC2インスタンスの起動には、AMIやインスタンスタイプ、ネットワークインターフェース、ディスク、セキュリティグループなどの設定情報が必須なのですが、それらをまとめてテンプレートとして作成し、そのテンプレートをインスタンス起動時に使い回すことができます。

オートスケーリンググループや、AWS BatchのEC2に適用したり、CloudFormationとも連携できたり、その応用範囲は非常に幅広い。

インスタンス起動時の手間の軽減はもとより、<strong>同一のテンプレートを複数人で使うことで、EC2インスタンス起動時の設定の差や操作差分をなくすことができ、インフラ構成管理の品質向上に寄与できる、地味だけど素晴らしい機能です。</strong>


## 起動テンプレートのタグ付け対応とは？

起動テンプレートそのものに対して、タグ付ができるようになっています。これにより、起動テンプレートに対しても、リソースレベルでの制限を<a href="https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_CreateLaunchTemplate.html" target="_blank">CreateLaunchTemplate</a>APIに対して設定できます。これにより、このAPIに対してのより柔軟で強力なポリシー制御が実現でき、例えば環境毎（開発、ステージング、本番）にテンプレートをタグ付けして分けておくことで、この起動テンプレートからのEC2インスタンスの起動をIAMポリシーで制御するといったことも可能です。

LaunchTemplateで実現できる制御の一覧は、こちらを参照。

- <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-supported-iam-actions-resources.html" target="_blank">Supported Resource-Level Permissions for Amazon EC2 API Actions - Amazon Elastic Compute Cloud</a>

今回のアップデートはEC2のドキュメントヒストリーの一番上に記載されています。

<a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/DocumentHistory.html" target="_blank">Document History - Amazon Elastic Compute Cloud</a>

X010

一点注意なのは、起動テンプレートで起動するEC2に対して設定するタグ付け機能では無いということです（この機能自体は昔からありました）。

## 実際に起動テンプレートにタグ付けしてみる

タグ付け自体は非常に簡単。既存の起動テンプレートの選択状態で、「アクション」をクリックすると、「タグの追加/編集」が選択できるようになっています。

X020

あとは、いつものタグの追加/編集画面で、テンプレートに対してタグを付けることができます。

X030

## 起動テンプレートのさらなる活用を目指して

起動テンプレートからのEC2起動は、使いこなせば非常に運用を簡潔で便利なものにしてくれる可能性があります。今回のアップデートを機に、まだ起動テンプレートを触ったことが無い方は、一度試してみることをオススメします。

合わせてこのあたりの記事も参考にしてみてください。

- <a href="https://dev.classmethod.jp/cloud/aws/try-launch-template-for-ec2/" target="_blank">[新機能]Launch Templates for Amazon EC2 instancesを試してみた #reinvent ｜ DevelopersIO</a>
- <a href="https://dev.classmethod.jp/cloud/aws/cfn-launch-template-ec2/" target="_blank">CloudFormationに対応したEC2の起動テンプレートを試してみた ｜ DevelopersIO</a>

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。








