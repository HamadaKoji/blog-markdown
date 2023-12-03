コスト管理データの新機能「Data Export」で、請求データの任意データをSQLでエクスポートできるようになりました！

絶賛re:invent2023が開催中の現在、多数のアップデートが出ています。今回は、請求周辺のData Exportという機能と、合わせてCUR 2.0が新規リリースされましたので、その内容を紹介していきます。

<pre style="line-height:120%;">
請求データあれこれできるのきたか…!!
　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>


## Data Export機能のアップデート公式情報とサマリー

アップデートはこちら。

[Announcing Data Exports for AWS Billing and Cost Management](https://aws.amazon.com/jp/about-aws/whats-new/2023/11/aws-billing-cost-management-data-exports/)

今回Data Exportsの新機能で、SQLを利用して請求やコスト管理に関するデータをエクスポートできるようになりました。エクスポートデータは、BIやデータ分析で利用するためにS3バケットに定期的に配信することができ、既存のCost and Usage Reportと同じ情報が含まれています。

SQL形式でデータを選択することで、配信先に必要なデータのみを提供できるようになります。

もっと踏み込んだ利用方法が記載されている公式ブログはこちら。

[Introducing Data Exports for AWS Billing and Cost Management \| AWS Cloud Financial Management](https://aws.amazon.com/jp/blogs/aws-cloud-financial-management/introducing-data-exports-for-billing-and-cost-management/)

以前より、ユーザーではCost and Usage Reports(CUR)を利用して、AWS提供の詳細なデータをエクスポートして受け取ることはできましたが、CURではエクスポートされたカラムはユーザーのAWS利用状況に応じて時間とともに変化し、その中のデータを制御することができませんでした。

Data Exportでは、CUR 2.0のエクスポートを作成することで、エクスポートする列を選択したり、ビジネスユニットのコストをフィルタリングしたり、特定データを非表示にしたりするなど、任意のクエリを請求データにかけることができます。

### Data Exportが利用できるリージョン

AWS GovCloud(US)リージョンと中国リージョンを除く全てのAWSリージョンで利用可能です。

## 実際にData Exportを試してみる



<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<strong>2023年11月27日：追記</strong><br />
現状、作成したDataExportを参照しようとすると、何故かマネコントップページにリダイレクトされます。おそらくリリース直後でうまく動作していないようなので、実際の動作検証はあとから再度更新する予定です。
</p>


というわけで、実際にData Exportを試してみます。

AWSのマネコンからBilling and Cost Managementを開くと、コスト分析とレポートセクションにData Export（データエクスポート）メニューが増えているのでクリックします。

010

020

そうすると、こんな感じでData Exportの初期画面が表示されるので、中央下の作成ボタンをクリック。

030

すると、データエクスポートの作成画面に遷移します。エクスポートタイプも複数から選択できますが、今回は標準データエクスポートを利用。

040

データエクスポート名を入力し、データテーブルコンテンツ設定で、実際にエクスポートする列を選択できます。

050

この列は全部で114種類あり、詳細は公式の[Cost and Usage Report \(CUR\) 2\.0 \- Data exports](https://docs.aws.amazon.com/ja_jp/cur/latest/userguide/table-dictionary-cur2.html)より、参照可能。

列の選択結果は、このようなSQLとしてプレビューすることが可能。逆にSQLからデータエクスポオートを作成することも可能なので、組織間でクエリを共有するなり、CLIやSDKでメンテナンスするなどの用途にも使えます。

060

その他データテーブル配信オプションや配信先のデータエクスポートストレージ（S3バケット）を設定して、Data Exportを作成。そうすると、このようにData Exportが作成され、最初の一覧に追加されます。

070

ただ、本セクションの上部で書いた通り、現状作成したDataExportにうまく遷移できない状況でした。しばらくして、データもエクスポートされると思いますので、また、明日にでも結果を確認しようと思います。

## 請求データへの柔軟なアクセス方法を提供する意欲的なアップデート

これまで、CURを請求データの分析に使っていた組織もかなり多いと思いますが、このData Exportの機能を利用することで、より柔軟かつプログラマブルに請求データの管理が実現できそうです。大規模にAWSを利用している組織であれば、このデータを利用することで、管理上の手間が一気に省略できる可能性もありそうなので、請求データに興味があるかた、一度試してみてはいかがでしょうか。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。








