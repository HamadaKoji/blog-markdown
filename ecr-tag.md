【今後が胸熱】ECRにタグ付けができるようになりました

先程、ECRのドキュメントヒストリーがしれっと更新され、ECRへのタグ付けができるようになりました。

<blockquote>Amazon ECR added support for adding metadata tags to your repositories.<br /><a href="https://docs.aws.amazon.com/AmazonECR/latest/userguide/doc-history.html" target="_blank">Document History - Amazon ECR</a></blockquote>

スルメを噛むような地味なアップデートですが、将来的にすげぇ機能追加がECRに来ることを予感させるものなので、ハマコー興奮しております。

<pre style="line-height:120%;">
ECRすげぇのくんの!?

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>


## アップデート内容

<a href="https://docs.aws.amazon.com/AmazonECR/latest/userguide/doc-history.html" target="_blank">Document History - Amazon ECR</a>にて、アップデート内容の概要が案内されています。

|  **Change** | **Description** | **Date** |
|  :------ | :------ | :------ |
|  Resource tagging | Amazon ECR added support for adding metadata tags to your repositories.<br/>For more information, see Tagging an Amazon ECR Repository. | 18 Dec 2018 |

以下のドキュメントが追加されています。

<a href="https://docs.aws.amazon.com/AmazonECR/latest/userguide/ecr-using-tags.html" target="_blank">Tagging an Amazon ECR Repository - Amazon ECR</a>

アップデート自体は非常にシンプルで、ECRコンソールに<span class="tt">tags</span>メニューが追加されており、リポジトリに対して任意のtagを指定することができます。EC2やその他いろんなリソースに対して提供されている、あのtags機能です。

020

こんな感じでリポジトリに対してタグが指定できます。

025

ある程度AWSの経験がある方には、見慣れたUIだと思います。

## ECRにタグをつけて何が嬉しいのか？

タグ機能はなんにでも使えるのですが、ユースケースとしては、下記が考えられます。

* 利用目的、環境、所有者などをタグ付けして整理する
* IAMポリシーでリポジトリへのアクセス権限を制御する
* 各リポジトリごとの課金状況を把握する
  * 特定のアプリケーション名などをタグで付与しておき、各アプリケーション毎にどの程度のECR課金があるかを可視化する

今まで細かい管理ができなかったECRですが、タグ機能付与により、EC2などのリソースと同様の管理が可能となってます。ぶっちゃけ地味っちゃ地味なアップデートですが、これ、次以降のアップデートにつながっていくと思うんですよね。

## ECRの将来アップデートへの布石とみて間違いない

先日のアップデートで、ECRはECSとは別の独自コンソールが提供されています。

030.png

また、突如、AWSのコンテナ関連サービスのロードマップがGitHubで公開されています。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">AWS のコンテナ関連サービスのロードマップを実験的にこちらで公開しております！ / &quot;aws/containers-roadmap: This is the public roadmap for AWS container services (ECS, ECR, Fargate, and EKS).&quot; <a href="https://t.co/Me3dvUrSC1">https://t.co/Me3dvUrSC1</a> <a href="https://t.co/PyGOOUhTyh">pic.twitter.com/PyGOOUhTyh</a></p>&mdash; ポジティブな Tori (@toricls) <a href="https://twitter.com/toricls/status/1072590232325947392?ref_src=twsrc%5Etfw">2018年12月11日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

その中にはこんなロードマップが！　<strong>全国5000万人のECRファンが熱望しているコンテナイメージに対する脆弱性スキャンです。</strong>

<a href="https://github.com/aws/containers-roadmap/issues/17" target="_blank">ECR Image vulnerability scanning · Issue #17 · aws/containers-roadmap</a>

040

勝手な推測ですが、今回リリースされたタグ機能によって、スキャン対象のECRリポジトリをタグ制御することができるんじゃないでしょうか。例えばプロダクション環境のリポジトリのみを定期スキャンの対象にするとか。

ここしばらくのECRのアップデートは、地味ですが将来のコンテナ関連アップデートの礎になるような重要な基礎固めとも言えます。今後のECRのアップデートからは目が離せません。皆さん、正座待機しておきましょうね！

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

