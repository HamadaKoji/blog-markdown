# 25年前のAmazonの分散コンピューティングの考え方が要約された「The Distributed Computing Manifesto」文書

ワーナーのキーノート画像

re:Invent2022 4日目、Dr. Werner Vogelsのキーノートにおいて、「The Distributed Computing Manifesto」というドキュメントがNew Articleとして紹介されました。

[The Distributed Computing Manifesto \| All Things Distributed](https://www.allthingsdistributed.com/2022/11/amazon-1998-distributed-computing-manifesto.html)

Amazonという超巨大なサービスが、モノリスの状態からその時まさに分散コンピューティングを推進していく転換点における考え方を示した貴重なドキュメントになっており、また、Werner自身も言うように、今後のAWSの進化の方向性を示唆する内容にもなっています。

<pre style="line-height:120%;">
温故知新ってこと…?!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>

まさにそんな感触です。

## 文書の位置づけ

こちらのサイトに追加されたドキュメントという位置づけです。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="All Things Distributed" src="https://hatenablog-parts.com/embed?url=https://www.allthingsdistributed.com/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

このサイトは、Amazon.comのCTOを長年務めるWernerが、貢献した論文や記事が掲載されているサイトで、今回Keynoteで紹介された「The Distributed Computing Manifesto」は、2022年11月16日にすでに公開されていたものが、KEYNOTEで改めて紹介されたものです。

[About \| All Things Distributed](https://www.allthingsdistributed.com/about.html)

## The Distributed Computing Manifestoとは

文書の内容はこちら。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="The Distributed Computing Manifesto | All Things Distributed" src="https://hatenablog-parts.com/embed?url=https://www.allthingsdistributed.com/2022/11/amazon-1998-distributed-computing-manifesto.html" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

### 論文の位置付け

冒頭、引用します。

<blockquote>Today, I am publishing the Distributed Computing Manifesto, a canonical document from the early days of Amazon that transformed the architecture of Amazon’s ecommerce platform. It highlights the challenges we were facing at the end of the 20th century, and hints at where we were headed.<br /><br />本日、私は「分散コンピューティング宣言」を公開します。これは、Amazonの初期の頃に作成されたドキュメントで、Amazonのeコマース・プラットフォームのアーキテクチャを一変させたものです。20世紀末に私たちが直面していた課題を浮き彫りにし、私たちが向かうべき方向を示唆しています。</blockquote>

文書の内容は1998年に作成された非常に古いものです。2004年当時、Amazonのアーキテクチャ情報として公開されたものはほぼ無かったようですが、その時分散システムの研究をしていたワーナーは、Amazon主催の分散システムの研究講演に招待され、その内容に非常に驚き、仕事のオファーを断れなかったようです。

CTOとして18年過ごした今も、当時の驚きをよく覚えていて、改めてこのタイミングで当時の論文を紹介したとのことでした。

<blockquote>その後20年の間に、Amazonはモノリスからサービス指向アーキテクチャ、マイクロサービス、そして共有インフラプラットフォーム上で動作するマイクロサービスへと移行していきました。これらはすべて、サービス指向アーキテクチャのような言葉が存在する以前に行われたことです。その過程で、私たちはインターネット規模での運用について、多くの教訓を得ました。</blockquote>

Wernerは、この文書についてKEYNOTEで深堀りし、また、それぞれの分野についての追加の論文を投稿する予定とのことです。


### 論文の構成

論文は、以下で構成されています。

- Background
- Key Concepts
- Service-based model
- Workflow-based Model and Data Domaining
- Applying the Concepts
- Tracking State Changes
- Making Changes to In-flight Workflow Elements
- Workflow and DC Customer Order Processing


##　読んでみた感想

2週間ほど前に、Wernerのサイトにホストされていたドキュメントですが、25年前にAmazonのエンジニが考えていた分散コンピューティングへの移行への考え方が、サービスとしての考え方をベースとした3層アーキテクチャへの移行、サービス間の依存関係の制限、サービスとワークフローの組み合わせ、非同期処理と同期処理の組み合わせ、ワークフローベースモデルとデータドメイニングのキーワードと共に紹介されています。

近年、AWSのサービスの中でも、Step FunctionsやEvent Bridgeの充実ぶりが非常に活発ですが、それも、25年前の分散コンピューティングの考え方を、AWSの各サービスを軸にユーザーが誰しも使えるように提供しようとする意図が見えているなと感じています。特に今回のキーノートでは、EventBridge Pipeの発表など、それらを意識した新サービスの紹介が多くあった感触です。

25年前から今までに通じる考え方と、これからのAWSの進化の方向性を捉えることができる貴重な資料だと思うので、皆さん、是非時間をつくって読んでもらえればと思います。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。





