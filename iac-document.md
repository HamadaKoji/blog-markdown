# Infrastructure as Codeにおける理想のドキュメント管理を目指して #infra

<strong>「なんや、この視聴者数…　震えが来るぜ・・・」</strong>

先日開催された<a href="https://forkwell.connpass.com/event/173289/" target="_blank">Infra Study Meetup #2「VM時代の開発とCloud Native時代の開発」 - connpass</a>において、「IaCにおける理想のドキュメント管理を目指す」という内容でLTしてきましたので、その内容をお届けします。

当日は、イベント内容も登壇者も超絶豪華で、なんとリアルタイム視聴者数1000人超えということで、さすがに自分も緊張しました。まじで。

青山さんのメインテーマがKubernetesの話であり、前後それに関わるテーマが中心の中、Kubernetesもコンテナも1ミリもでてこない発表にしたのですが、IaCに関わる普遍的な考慮ポイントについて喋れたのかなと思います。

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     ｲﾝﾌﾗﾏﾂﾘﾀﾞﾜｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>

## 登壇スライド

<script async class="speakerdeck-embed" data-id="1458a925228e4643a901520e962328a3" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

概要は、上のスライド見れば全て載っていますが、各種資料へのリンクなど併記しているので参考にしていただければと思います。

## IaCに必要なドキュメントとは？

最初に自己紹介させてください。こちら。自分のプロフィール画像とtwitter乗せたらPowerPoint先生が提案してくれたデザインです。テクノロジーってすごいですね。超お気に入りの一枚です。

02

### IaCに期待されること

最初に皆さんと認識を共有しておきたいことがあります。今回の視聴者の方は、それぞれなんらかの形でインフラ領域に関わっているかと思うのですが、その中でInfraStructure as Code（IaC）で実現したいことってなんでしょうか？

IaCで期待されることの例として、以下がよく挙げられます。

- 構築手順書をなくしたい
- ヒューマンエラーを無くしたい
- 効率よくインフラを構築したい
- 不要なドキュメントを無くしたい

最後の、「不要なドキュメントを無くしたい」部分。例えば、インフラの設定内容をそのままよこになぞらえたエクセルのパラメータシートとか。「本当に必要なのかこれ？全部コードでよくない？」と思う人も多いと思います。

<strong>じゃ、IaCにしておけば、全てのドキュメントが不要になるでしょうか？決してそんなことはありません。</strong>インフラを扱う現場には、必ず作成しているドキュメントがあります。それがこちら。

09

### 構成図を書くことのメリット・デメリット

いわゆるひとつの<strong>構成図</strong>です。インフラ扱ってて作ったこと無い人は皆無だと思います。そんな構成図を作ることのメリット・デメリットがこちら。

#### メリット

- 管理対象領域にどんなリソースがあるのかひと目で把握できる
- 各リソースのリレーションが把握できる
- 他人と対象領域に対する認識を共有しやすい

複数人で同じインフラ領域を扱いさい、その意思疎通をスムーズにして認識をあわせるために構成図は必須のツールと言えます。

#### デメリット

- IaCとは別に管理しメンテする必要がある

多大なメリットを感じながらも、運用していく中でそのメンテナンスが忘れられがちな側面があると思います。

## 構成図の管理をどうするのか？

IaCのコードをリポジトリ管理しない現場はないです。インフラ変更のライフサイクルと等しく構成図も管理されているべきだと思うのですが、その管理方法は大きく分けて3つのアプローチがあります。

15

3つのうち、リポジトリ管理する方法としない方法にわかれます。この後、それぞれのアプローチについて自分の主観を述べていきます。

### ①インフラから構成図を常に自動生成

すでにあるインフラから、構成図を自動生成するアプローチです。

17

いくらか、代表的なツールを挙げてみます。

- <a href="https://cloudviz.io/" target="_blank">Cloudviz – Automated AWS Architecture Diagrams &amp; Documentation</a>
- <a href="https://cacoo.com/ja/" target="_blank">フローチャートやワイヤーフレーム、プレゼン資料まで作れる | Cacoo(カクー)</a>
- <a href="https://github.com/duo-labs/cloudmapper" target="_blank">duo-labs/cloudmapper: CloudMapper helps you analyze your Amazon Web Services (AWS) environments.</a>

ただ、これだけで構成図の管理が全て完結するかと言うと、自分の結論は<strong>No</strong>です。理由は、自動生成された構成図は情報量が足りない（もしくは多すぎる）ためです。

実際にやってみたことあるひとは何となく分かると思いますが、正直自動生成された構成図って「見にくい」ですよね？

構成図って、そこに人間の意志をこめたくなるんですよね。なぜなら機械ではなく人間が見るものだからです。そこには、システムごとの関連性や、並び順や、それぞれのリレーションの表現であったりとか。また、連携する外部サービスとかもあったり、サブシステムのグルーピングもしたくなります。

そういった組織とインフラで共有されているコンテキストを全てすっ飛ばして、自動生成される構成図は、はっきり言って人間がみるのには役に立たないことが多いです。

まだ、構成図がなにもないところからインフラの状態を把握するために初っ端生成する用途では悪くないと思いますが、常に自動生成された構成図のみで、人間がコミュニケーションを取るのは未来永劫難しいと考えています。

### ②コードで構成図を作成するツールを使う

以前からよくある、テキストやプログラミング言語で、構成図を生成するアプローチです。この場合、テキストやプログラミング言語をGitHubなどのVCSで管理します。

20

いくらか、代表的なツールを挙げてみます。

- <a href="https://plantuml.com/ja/" target="_blank">シンプルなテキストファイルで UML が書ける、オープンソースのツール</a>
- <a href="http://blockdiag.com/ja/nwdiag/index.html" target="_blank">ネットワーク図生成ツール nwdiag — blockdiag 1.0 ドキュメント</a>
- <a href="https://diagrams.mingrammer.com/" target="_blank">Diagrams · Diagram as Code</a>
  
こういった、コードから構成図を作成するアプローチですが、正直これも自分はしんどいと思います。１番の理由は、<strong>図を書くための独自記法を習得する必要があるからです。</strong>

本来的にやりたいことは、構成図を書くこと。その構成図を書くためだけに、わざわざ新しいツールを導入したりそのための記法を習得してやっていく意味あるでしょうか？チームで複数人でやる場合、そのためのコストも馬鹿にはなりません。

本来的に、構成図をコードで管理するという目的に立ち返るのであれば、別のアプローチがあると思ってます。

### ③専用ツールを使って構成図（SVG）を修正

ものすごいシンプルです。SVGファイルを専用のドローイングツールで修正する方法です。

24

SVGファイルはいろんなドローイングツールが対応しています。自分のお気に入りだと、この2つ。l

- <a href="https://cacoo.com/ja/" target="_blank">フローチャートやワイヤーフレーム、プレゼン資料まで作れる | Cacoo(カクー)</a>
- <a href="https://draw.io" target="_blank">draw.io</a>
  
これが良い理由を2つあげます。

- SVGファイル（XML）を扱えるためリポジトリにのせやすい
- 専用ツール（Cacoo, draw.io）を使って構成図が書ける（導入コストが低い）

SVGファイル、皆さんご存知ですかね。中身はXMLファイルで書かれているベクター形式の画像です。バイナリファイルではないので、リポジトリに乗せやすいというメリットがあります。また、SVGファイルは画像形式として広く使われているため、ドローイングツールに広く対応しています。そのため、直感的にその構成図の編集ができます。

GitHubなどでもSVGファイルは、そのまま画像として表現されます。そのためREADMEに埋め込んだり、Pull requestするときにIaCのコードと合わせてレビュー対象にするのも簡単。

これだけでも良いんですが、IaCのコードと構成図を同じツールで編集するべく、最近非常に便利なVisualStudioのインテグレーションが発表されました。それがこちら。

26

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://marketplace.visualstudio.com/items?itemName=hediet.vscode-drawio" target="_blank">Draw.io Integration - Visual Studio Marketplace</a>
</p>

これを使うと、<strong>コードと一緒に構成図が書ける！</strong>

使い方は非常に直感的。このインテグレーションをインストールしておくと、Visual Studio CodeのエクスプローラーでSVGファイルをクリックするだけで、draw.ioのUIがエディタ内に起動。そのまま編集して保存すれば、それが即SVGファイルに反映されます。

ざっくり、こんな見た目です。左でTerraformのHCLを編集しながら、右側で構成図を編集するというね。

28

## まとめ

- インフラを扱うとき**構成図は必須のドキュメント**
- IaCのコードと合わせて**構成図もリポジトリ管理するべき**
- **餅は餅屋**。構成図の編集は専用ツールを使う
- draw.ioインテグレーションなどを使って、**IaCと同じデベロッパーエクスペリエンスを実現しよう！**

## その他マテリアル

### togetter

自分登壇の傍らtwitterも見てましたが、めちゃくちゃtwitterの流量が多いイベントでした。今回からYoutubeLiveのコメント欄は閉じられ、コメントがtwitterに集約されたという理由もあるかもですが、視聴者数1000人超のtweetは必見です。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="Infra Study Meetup #2「VM時代の開発とCloud Native時代の開発」 - Togetter" src="https://hatenablog-parts.com/embed?url=https://togetter.com/li/1521029" width="300" height="150" frameborder="0" scrolling="no"></iframe>

### youtube

動画ももちろん公開されています。他の登壇者の発表も必見の濃さですよ！

<iframe width="560" height="315" src="https://www.youtube.com/embed/9YpN9-RCwWE" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## 登壇時の緊張感

はっきり言って、LTだというのに珍しいぐらい緊張しました。というのも視聴者数が、自分にとってはありえない規模（1000人超）だったので、もし自分の回線の不具合や不手際で、発表自体がうまくいかなかったときのことを考えると、最初から最後まで不安が拭えなかったのが事実です。

ただ、運営の皆様も非常に入念なリハーサルもあり、進行自体は全く問題なくスムーズに進みました。今更ながら主催のForkwellさん、協賛のソニーさん、メドピアさん、ライブ配信のインフラを支えていただいた天神放送局さんには感謝しております。

Infra Study Meetup、今後もどんどん濃く展開されていく予定なので、次回以降もぜひみなさん参加してみると良いと思います。見るだけじゃなくて、LTも絶賛募集中とのこと。我こそはというかたは、どんどん応募しましょう。すでに第3回の募集も始まってますよ！

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="Infra Study Meetup #3「SREのこれまでとこれから」 - connpass" src="https://hatenablog-parts.com/embed?url=https://forkwell.connpass.com/event/176885/" width="300" height="150" frameborder="0" scrolling="no"></iframe>

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。


