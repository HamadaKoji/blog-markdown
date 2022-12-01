# Route 53のリージョン障害時の緊急回避手段「Application Recovery Controller」が発表されました！

Amazon Route 53が、AZ（Abailability Zone）の障害発生時に、特定ゾーンへのトラフィックをワンクリックで遮断する、Application Recovery Controllerが発表されました。

ALBから複数AZに対してトラフィックが発生している状態でAZ障害が発生した時、ALBからのトラフィックをワンクリックで特定ゾーンに向けることができる機能で、アプリケーションへの障害影響度を抑えることが期待できます。

まずは、速報で概要をお届けします。

<pre style="line-height:120%;">
特定AZへのトラフィック寄せきたか…!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>

## What's Newの更新情報サマリー

What's Newはこちら。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="Announcing preview for Amazon Route 53 Application Recovery Controller zonal shift" src="https://hatenablog-parts.com/embed?url=https://aws.amazon.com/jp/about-aws/whats-new/2022/11/amazon-route-53-application-recovery-controller-zonal-shift/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

### アップデートのサマリー

- Amazon Route 53 Application Recovery Controller（ARC）の追加機能という位置づけ
- 実際に実行するには、Amazon Route 53 Application Recovery Controllerコンソールにアクセスし、AWSアカウント内のAWSリージョンにあるロードバランサーのゾーンシフトを開始する
- AWS SDKを使用した制御が可能
- ゾーンシフトは、クロスゾーン構成が無効なALBとNLBで利用可能
- 2022年11月29日時点では、プレビュー扱いのため、動作内容やドキュメントが更新される可能性があります

### 料金について

ゾーンシフトの追加料金は特に無し

### 利用できるリージョンについて（2022年11月29日において）

- 米国東部（オハイオ州）
- 米国東部（北バージニア州）
- 米国西部（オレゴン州）
- ヨーロッパ（アイルランド）
- アジア太平洋（東京）
- アジア太平洋（シドニー）
- ヨーロッパ（フランクフルト）
- ヨーロッパ（ストックホルム）
- アジア太平洋（ジャカルタ）

### 関連ドキュメント

公式のローンチブログはこちら。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="Rapidly recover from application failures in a single AZ | Networking & Content Delivery" src="https://hatenablog-parts.com/embed?url=https://aws.amazon.com/jp/blogs/networking-and-content-delivery/rapidly-recover-from-application-failures-in-a-single-az/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

ドキュメントはこちら。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="Zonal shift in Amazon Route 53 Application Recovery Controller - Amazon Route 53 Application Recovery Controller" src="https://hatenablog-parts.com/embed?url=https://docs.aws.amazon.com/r53recovery/latest/dg/arc-zonal-shift.html" width="300" height="150" frameborder="0" scrolling="no"> </iframe>


## 動作イメージ

下の画像は、公式のアップデートブログから引用しています。このように、クロスゾーンのロードバランシングをオフにすると、各ゾーンのロードバランサーゾーンはローカルAZ内のみにリクエストをルーティングするようになります。これにより、ロードバランサーとターゲット間のゾーン障害コンテナを整列させ、障害をより簡単に検出できるようになります。

cross-zone.png

## 実際に設定してみる動かしてみるには？

公式ブログの「Trying out zonal shift」で、デモ用に利用できるCloudFormationがダウンロードできます。

[aws\-samples/arc\-zonal\-shift\-demo](https://github.com/aws-samples/arc-zonal-shift-demo)

公式ブログにはチュートリアル形式で、動作を確認する手順も記載されているので、関連リソースをデプロイ後、動作確認してみて体験してみるのが良さそうです。

## ELBの進化がとまらない

しばらく前に、ELBでは以下の機能がリリースされていました。

[Introducing Amazon Route 53 Application Recovery Controller \| AWS News Blog](https://aws.amazon.com/jp/blogs/aws/amazon-route-53-application-recovery-controller/)

今回のゾーンコントロールは、上記機能の追加リリースという位置づけで、ゾーンを特定することで、さらに柔軟なトラフィックの制御を可能とするものです。

これらを利用して、より柔軟かつ回復性の高いALBとアプリケーションの運用が可能になるので、ぜひ皆さん一度試してみることをおすすめします。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

