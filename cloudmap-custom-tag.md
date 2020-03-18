# Cloud Mapの主要機能であるサービスのカスタム属性編集がコンソールでできるようになりました！

みなさん、Cloud Map、使ってますか？<strong>むしろ覚えてますか？</strong>

re:Invent 2018において颯爽と登場したサービスで、クラウド上のありとあらゆるリソースに対して任意の名前をつけることで、より柔軟に各サービスの接続を管理できる渋い機能なのですが、そのCloudMapに久しぶりにアップデートがやってきました！

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://aws.amazon.com/jp/about-aws/whats-new/2020/01/aws-cloud-map-supports-editing-custom-service-instance-attributes-in-the-aws-console/" target="_blank">AWS Cloud Map が AWS コンソールでのカスタムサービスのインスタンスの属性の編集をサポート</a>
</p>

機能拡充というよりは、今までAPIのみで提供されていた機能がWebコンソールでもできるようになったよ！ということなのですが、進化は進化。これを機会にCloud Mapの奥深い世界に踏み込んでくれる人が増えると、隠れCloud Mapファンのハマコーが喜びます。

喜びます。

<pre style="line-height:120%;">
それだけかよ!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>

## Cloud Mapとは？

クラウドリソースの検出を柔軟にできるようにするサービスです。イメージ的には、クラウドで利用する各サービスへの接続情報（AWSのリソースであればARN、HTTPアクセスするエンドポイントであればURIなど）を、一括管理し任意のタイミングで切り替えることができるサービスです。

概念図は、公式のこちらがわかりやすい。AWSのいろんなリソースに、任意の名前でアクセスする雰囲気がわかりますでしょうか。

010
引用元：https://aws.amazon.com/jp/blogs/news/new-application-integration-with-aws-cloud-map-for-service-discovery/

合わせて全体の概要については、公式ブログを参照いただくとともに、こちらも見ていただければと思います。

- <a href="https://dev.classmethod.jp/cloud/aws/cloud-map-perfect/" target="_blank">詳細解説「AWS Cloud Map」とは #reinvent ｜ Developers.IO</a>

## 今回のアップデートの何が嬉しいのか？

「AWS コンソールでカスタムサービスのインスタンスの属性の編集ができるようになった」です。Cloud Mapは実際にサービスに紐づくインスタンスを検出するときに、<code>discover-instances</code>APIを使います。

[bash]
$ aws servicediscovery discover-instances --namespace-name hamako9999.com --service-name payments --query-parameters ver=1.0,env=stg
[/bash]

このとき、<code>--query-parameters</code>で指定した属性が紐づくリソースが返ってくるので、この属性を予め各種リソースにつけておくことで、その属性に紐づくリソース一式を簡単に取得できるわけです。

020

030

040

この属性の編集（追加、変更、削除）が、Webコンソールでできるようになった！というわけです。

## Webコンソール上での変更点

[Register service instance]画面の一番下で、カスタム属性が編集できるようになっています。

050

今までは、ここをコンソールで設定する方法がなく、インスタンス登録時に属性を設定する場合は、別途CLIやSDKなどを使ってのAPI利用しか方法がありませんでした。

## Cloud Map、みんなどんどん使おうぜ！

アップデートの内容自体は、非常に地味でしたね。ただ、リソースへの属性の付与はCloud Mapのキモの機能ということがおわかりいただけましたでしょうか。それがWebコンソールで編集できるのは、管理上の手間を減らす効果があります。

実は、CloudMapのDocument Historyをみると、履歴がありません。

- <a href="https://docs.aws.amazon.com/cloud-map/latest/dg/doc-history.html" target="_blank">Document History for AWS Cloud Map - AWS Cloud Map</a>

ドキュメント履歴がない→APIに変更がない→機能拡張がないということで、自分としては非常に寂しく思っているのですが、使い方次第では非常に応用範囲が広いサービスだと思うので、今回のアップデートをきっかけに、Cloud Mapの活用方法について思いを馳せてみるのはどうでしょうか？

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。
