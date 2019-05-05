# クラウドネイティブ時代に必要なコンテナセキュリティの考え方と対応策 #devio2019sec

みなさん、コンテナワークロードのセキュリティについて、<strong>「そりゃもちろん、うちは完璧やで！！」</strong>と胸を張って言えるでしょうか？

先日、弊社主催のセキュリティ関連イベント、「Developers.IO 2019 Security」を開催させていただきました。

<a href="https://dev.classmethod.jp/news/developers-io-security/" target="_blank">【4/25（木）東京】Developers.IO 2019 Securityを開催します #cmdevio2019sec ｜ DevelopersIO</a>

私の枠では「クラウドネイティブ時代に必要なコンテナセキュリティの考え方と対応策」というタイトルで、コンテナワークロードにおいて必要なセキュリティの考え方と、具体的な製品<a href="https://www.aquasec.com/" target="_blank">Aqua - Container Security</a>を紹介させていただきました。

実際の製品デモでは、<a href="https://www.creationline.com/" target="_blank">クリエーションライン株式会社 (CREATIONLINE, INC.)</a>様にも、ご協力をいただきました。感謝です！！

<pre style="line-height:120%;">
Aqua Container Securityきたか…!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>

## 「クラウドネイティブ時代に必要なコンテナセキュリティの考え方」

<script async class="speakerdeck-embed" data-id="692b6daa18dd495d9411ffa4275aaa23" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

<blockquote>2019年現在、AWSには、ECS、Fargate、EKSなどが東京リージョンでリリースされており、いつでも本番環境でコンテナを運用できる環境が整っています。<br />
一方で、セキュリティ面に目を向けると、コンテナアプリケーションの対策手法はまだまだ確立されているとは言えません。アプリケーションの構築方法が従来とは異なるように、セキュリティ面でも、コンテナ独自の考え方が必要になります。とは言え、セキュリティを向上させたことで、コンテナの利点であるDevOpsスピードを損なってしまっては本末転倒です。<br />
このセッションでは、クラウドネイティブなコンテナ環境ならではのセキュリティ対策の考え方を説明した後、DevOpsスピードを損なうことなくセキュリティ対策を実現するためのソリューション「Aqua Container Security Platform」を実際のデモを交えてご紹介いたします。</blockquote>

こちらでは、従来のEC2などホストインスタンスにアプリケーションをデプロイする場合と、コンテナでアプリケーションを提供する場合の基本的な違いについて、お話させていただきました。具体的な話はほぼ全て省略していますが、Fargateの台頭など、新しいコンテナプラットフォームについて、紹介しています。


## 「Aqua Security概要」

<script async class="speakerdeck-embed" data-id="4e0bfc1214d44aa08926333d7922eeaa" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

<strong>こちらがメインコンテンツです。</strong>

クリエーションライン株式会社　マグルーダー健人様による講演。Aqua Securityについて、コンテナセキュリティの考え方と必要な対策、それに合わせてAqua Container Securityの主な製品仕様を、実際の製品画面のデモも交えて紹介いただきました。

### Aqua Container Securityの実際の画面と機能紹介

#### 0. ダッシュボード

ダッシュボードへの各種サマリの表示

123

#### 1. イメージスキャン

イメージスキャン結果による<strong>CVE（脆弱性）</strong>が含まれることの警告

124

イメージ内に<strong>機密データ</strong>が含まれることの警告

125

#### 2. イメージポリシー

イメージにリスクが含まれている場合のアクションを指定可能

126

リスク種別による有効/無効などの<strong>ポリシー定義</strong>が可能

127

#### 3. ランタイムポリシー

<strong>実行コマンドブラックリストが定義</strong>されている場合、コンテナ内で当該コマンドの<strong>実行を制御</strong>

128

<strong>機械学習でイメージをプロファイリング</strong>し、イメージで実行許可されるコマンドやログイン可能ユーザーなどを<strong>自動でポリシー定義を生成</strong>

129

#### 4. コンテナファイアウォール

コンテナの<strong>ネットワーク接続を視覚化。</strong>コンテナ単位での接続の<strong>拒否/許可のルール設定</strong>が可能

130

#### 5. Secrets（シークレット管理）

シークレット設定が可能

#### 6. CI/CD Integration（CI/CDツール統合）

131

多くの<strong>CI/CDツールのプラグイン</strong>が提供されており、統合することが可能。

#### 7. Compliance（コンプライアンス対応）

<strong>リスクを一覧化</strong>し、対応方法などを含む<strong>詳細レポート</strong>を生成

133

イメージやコンテナで発生した、各種の<strong>セキュリティイベント</strong>を収集

### コンポーネント構成

140

### セッション中の会場の様子

デモ中、ラインタイムポリシーのコマンド実行制御のところで、<strong>「すげぇ！」「まじかYO！」</strong>などの合いの手が入ったり、なんかすごい盛り上がりました。ご来場いただいた方には、こんな面白いコンセプトの製品があることを、興味深く見ていただいたのではと思っています。


## コンテナのセキュリティに迷われている方へ


ある程度大規模、かつ頻繁な変更がはいるコンテナワークロードを本番運用しながら、さらにCI/CDの中にまで踏み込んでセキュリティを担保するのは、存外難しいものです。

もしみなさんがそういった点でお悩みがあれば、弊社問い合わせフォームからお問い合わせいただくか、自分のtwitterアカウント（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）にでも雑にメンションいただければ、いつでも対応可能です。お気軽にお問い合わせください。

弊社ブログにも、Aqua製品についてのブログあります。こちらも随時更新していきますので、是非御覧ください。

https://dev.classmethod.jp/cloud/aws/about-aqua-csp/

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。







