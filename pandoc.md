# 44種類のフォーマットに対応したPandocでMarkdownをHTML形式に変換する

<strong>「GitHubに格納したMarkdownのええ感じで公開したいんやで！！」</strong>

Markdownで書いたドキュメントを、何らかの形で公開したいというニーズは凄くあると思います。その中であれこれ探していたんですが、やっぱり<strong>「Pandoc」</strong>に落ち着きました。

- 変換フォーマットが超絶豊富
- HTML用テンプレートが用意されている
- 公式DockerイメージがあるのでCI/CDに組み込みやすい
- おまけにGitHub Actionsのサンプルもあるで！

結構今どきのツールだったので、前のめりに紹介いたします！！

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     ﾄﾞｷｭﾒﾝﾄ変換ﾏﾂﾘﾀﾞﾜｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>

## 利用するツールはPandoc

今回利用するツールは、Universal Documet Converterの<a href="https://pandoc.org/index.html" target="_blank">Pandoc</a>。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://pandoc.org/index.html" target="_blank">Pandoc - About pandoc</a>
</p>

<blockquote>If you need to convert files from one markup format into another, pandoc is your swiss-army knife. Pandoc can convert between the following formats:<br ><p>引用：<a href="https://pandoc.org/index.html">Pandoc - About pandoc</a></p></blockquote>

<blockquote>様々なマークアップ文章フォーマットをあらゆる形式に変換するツールです。そう、スイスアーミーナイフのように…</blockquote>

ということで、万能文書変換ツールといえばわかりやすい。

## Pandocの魅力

そんな万能文書変換ツールのPandocですが、ちょっと調べてみた限りでも、魅力が一杯です。

### 対応するフォーマットが膨大

Pandocの特徴は、対応フォーマットが非常に幅広い点。全部で44種類の文書形式に対応しており、変換元と変換先双方に対応しているフォーマットも非常に多いです。

|  フォーマット分類 | フォーマット | 変換元 | 変換先 |
| --- | --- | --- | --- |
|  **Lightweight markup formats** | Markdown | ◯ | ◯ |
|   | reStructuredText | ◯ | ◯ |
|   | AsciiDoc | ✗ | ◯ |
|   | Emacs Org-Mode | ◯ | ◯ |
|   | Emacs Muse | ◯ | ◯ |
|   | Textile | ◯ | ◯ |
|   | txt2tags | ◯ | ✗ |
|  **HTML formats** | HTML4 | ◯ | ◯ |
|   | HTML5 | ◯ | ◯ |
|  **Ebooks** | EPUB version 2 or 3 | ◯ | ◯ |
|   | FictionBook2 | ◯ | ◯ |
|  **Documentation formats** | GNU TexInfo | ✗ | ◯ |
|   | Haddock markup | ◯ | ◯ |
|  **Roff formats** | roff man | ◯ | ◯ |
|   | roff ms | ✗ | ◯ |
|  **TeX formats** | LaTeX | ◯ | ◯ |
|   | ConTeXt | ✗ | ◯ |
|  **XML formats** | DocBook version 4 or 5 | ◯ | ◯ |
|   | JATS | ◯ | ◯ |
|   | TEI Simple | ✗ | ◯ |
|  **Outline formats** | OPML | ◯ | ◯ |
|  **Data formats** | CSV tables | ◯ | ✗ |
|  **Word processor formats** | Microsoft Word docx | ◯ | ◯ |
|   | OpenOffice/LibreOffice ODT | ◯ | ◯ |
|   | OpenDocument XML | ✗ | ◯ |
|   | Microsoft PowerPoint | ✗ | ◯ |
|  **Interactive notebook formats** | Jupyter notebook (ipynb) | ◯ | ◯ |
|  **Page layout formats** | InDesign ICML | ✗ | ◯ |
|  **Wiki markup formats** | MediaWiki markup | ◯ | ◯ |
|   | DokuWiki markup | ◯ | ◯ |
|   | TikiWiki markup | ◯ | ✗ |
|   | TWiki markup | ◯ | ✗ |
|   | Vimwiki markup | ✗ | ◯ |
|   | XWiki markup | ✗ | ◯ |
|   | ZimWiki markup | ✗ | ◯ |
|   | Jira wiki markup | ◯ | ◯ |
|  **Slide show formats** | LaTeX Beamer | ✗ | ◯ |
|   | Slidy | ✗ | ◯ |
|   | reveal.js | ✗ | ◯ |
|   | Slideous | ✗ | ◯ |
|   | S5 | ✗ | ◯ |
|   | DZSlides | ✗ | ◯ |
|  **Custom formats** | custom writers can be written in lua. | ✗ | ◯ |
|  **PDF** | PDF | ✗ | ◯ |

今回紹介するMarkdownやHTMLは当然として、reStructuredText、Textile、LaTeX、何故かCSV，EPUB、docx（Word）や、pptx（PowerPoint）、Jupyter notebook、各種Wiki形式、PDFに対応しています。

### メタデータや独自フォーマットへの豊富な対応

Pandocは、Markdonのシンタックス拡張や、ドキュメントメタデータ、テーブル、脚注、箇条書き、上付き文字、下付き文字、順序付きリストに対応しています。文書変換時に、このあたりのフォーマット情報がきちんと変換されるのは心強い。

### 定期的なアップデート

<a href="https://pandoc.org/releases.html" target="_blank">Pandoc - Releases</a>を見るとわかりますが、ほぼ3ヶ月に一度ほど定期的に更新されています。最新のリリースは2020年2月15日と非常にフレッシュ！　自分実はPandoc自体は3年ぐらい前からその存在はしっていて、かなり歴史が長いプロダクトだと思うんですが、今でもこれぐらいの頻度で更新されているのは心強い。

### 公式Dockerイメージが提供されていて、CI/CDで使いやすい

<strong>個人的に一番気に入っているポイントが、公式よりDockerイメージが提供されているところ。</strong>

PandocはHaskell性のツールで、通常の利用には各クライアントへのインストールが必要なのですが、公式でDockerイメージが提供されています。

- <a href="https://hub.docker.com/r/pandoc/core" target="_blank">pandoc/core - Docker Hub</a>
  - pandoc本体とpandoc-citeprocを含みます
- <a href="https://hub.docker.com/r/pandoc/latex" target="_blank">pandoc/latex - Docker Hub</a>
  - pandoc/coreに加えて、PDF作成用の最小のLaTeXが含まれます。
- <a href="https://github.com/pandoc/dockerfiles" target="_blank">pandoc/dockerfiles: Dockerfiles for various pandoc images</a>

ので、Dockerがインストールされている環境では、以下のようなワンライナーで即文書変換が可能です。この場合、カレントディレクトリにある<code>README.md</code>を<code>README.pdf</code>に変換します。

[bash]
docker run --rm --volume "$(pwd):/data" --user $(id -u):$(id -g) pandoc/latex README.md -o README.pdf
[/bash]

Dockerイメージが提供されているということは、各種CI／CDツールで使いやすい。例えば、MasterブランチへのMarkdownテキストプッシュをトリガーに自動的にHTMLやPDFに変換して、Web公開するという使い方もできます。

### GitHub Actions のサンプルあり

公式から、GitHub Actionsのサンプルも提供されていたりします。素敵！

<a href="https://github.com/pandoc/pandoc-action-example" target="_blank">pandoc/pandoc-action-example: using the pandoc document converter on GitHub Actions</a>

## HTMLへの変換オプション

### 基本的な使い方

pandocの基本的な使い方を最初に解説。今回自分は、pandocをdockerイメージ使って利用するので、Markdown（sample.md）をHTML（sample.html）に変換する基本的な構文は以下になります。

<code>sample.md</code>は、<a href="https://markdown-it.github.io/" target="_blank">markdown-it demo</a>のMarkdownを利用しています。

[bash]
docker run --rm --volume "`pwd`:/data" --user `id -u`:`id -g` pandoc/core sample.md -o sample.html
[/bash]

dockerコマンドでは、カレントディレクトリをコンテナの/dataフォルダにマウントして、ユーザーとグループを指定指定しています。pandocをDocker使わずにクライアントにインストールして利用する場合は、<code>pandoc/core　〜</code>以降を、<code>pandoc 〜</code>で書き換えて利用ください。

### 各オプションについての公式マニュアル

オプション膨大にあります。詳細はこちらの公式ドキュメントを参照

<a href="https://pandoc.org/MANUAL.html" target="_blank">Pandoc - Pandoc User’s Guide</a>

### アウトプットファイルの指定

<code>-o</code>オプションで、出力ファイル名を指定します。これを指定しない場合、変換結果は標準出力に出力されます。

### HTMLファイルとしての出力

<code>-s</code>オプションにより、HTMLファイルが適切な<code>haed</code>や<code>body</code>タグを出力してくれます。<strong>後続のスタイルシート指定の際にも必要なので、HTMLファイル出力時は必須のオプションです。</strong>

[bash]
pandoc -s sample.md -o sample.html
[/bash]


### スタイルシートの指定

pandocのデフォルトだと、出力されたHTMLのスタイルがそっけないので、独自のスタイルを埋め込みたい場合、<code>-c</code>オプションで、スタイルシートを指定できます。

[bash]
pandoc sample.md -c style.css -o sample.html
[/bash]

### 目次（Table of Contents）の生成

見出しタグ（H要素）をまとめた目次を生成します。

- --toc
  - 目次の生成
- --toc-depth=NUMBER
  - 目次を生成するH要素のレベル。デフォルトでは3となっていて、その場合H1〜H3が目次として生成されます

[bash]
pandoc -s --toc -c style.css --toc --toc-depth=2 sample.md -o sample.html
[/bash]

### テンプレートファイルの指定

HTMLの文書構造から指定するため、Pnadocにはテンプレート機能があり、そのテンプレートを指定することができます。

- <code>--template=FILE|URL</code>
  - テンプレートに利用するファイルへの相対パス、もしくはURL

テンプレートファイルの書き方のマニュアルはこちらに定義されています。

<a href="https://pandoc.org/MANUAL.html#templates" target="_blank">templates - Pandoc User’s Guide</a>

上記ドキュメントを参考にテンプレートファイルをイチから作成しても良いんですが、結構手間がかかるのでそれっぽいテンプレートを使って、自分好みに編集していくのが早いです。

自分が探した限りだと、このリポジトリがおすすめ。この中の<code>html</code>ディレクトリに、HTML用のテンプレートファイルが格納されています。

<a href="https://github.com/ryangrose/easy-pandoc-templates" target="_blank">ryangrose/easy-pandoc-templates: A collection of portable pandoc templates with no dependencies</a>

この中の<code>html/elegant_bootstrap_menu.html</code>がおすすめで、これを利用すると、以下の変換でこんな見た目になります。

[bash]
pandoc -s --toc --template=easy-pandoc-templates/html/elegant_bootstrap_menu.html sample.md -o sample.html
[/bash]

010

<code>--toc</code>を併用することで、ドキュメントなどによくある見出しを左側に固定して表示できるので、Markdownで書いたドキュメントをHTMLに変換するときに使いやすいと思います。細かい見た目は、ここからスタイルシートを変更していけば対応できるかと思います！


## 導入も簡単で手間も少なくMarkdownをHTMLに変換できる便利ツール

今回自分が変換したいドキュメントはこちら。まだ思いっきり書きかけやけど。

<a href="https://github.com/classmethod/aws-container-design-guide" target="_blank">classmethod/aws-container-design-guide: AWSにおけるコンテナスタンダード（AWS環境においてコンテナ基盤を構築するにあたり、そのベストプラクティスをまとめた文書。想定読者はアプリケーション開発者ではなく、インフラ設計者）</a>

GitHubでそのままMarkdownみてもらおうと思ったんですが、ある程度の分量になってくるとやっぱり目次もつけたHTMLで公開したいなぁという欲がでてきて、このPandocを触ってました。<strong>自分にとってはなんと言っても、公式でDockerイメージが提供されていることによるCI/CDへの組み込みやすさが魅力です。</strong>

まだ実装できていませんが、近いうちにGitHub Actionsに組み込んで、Markdownを自動的にHTMLにしてS3から公開することを目論んでいます。HTML以外にもWordやPDFへの変換もやりたいですね。先人の知恵、ありがたい。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。
