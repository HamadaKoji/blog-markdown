# （仮）「Splunk App for AWS」を試してみた

先日、こちらの記事で生まれて始めてSplunkを試してみたハマコーは、そのログ分析プラットフォームとしての柔軟かつパワフルな機能に惚れ込んでしまいました。

- <a href="https://dev.classmethod.jp/etc/about-splunk/" target="_blank">統合ログ分析ソリューションSplunkの凄さを公式チュートリアルで体感する</a>

「こんなんできるんやったら、AWSと統合したらもっとおもろいことできるんちゃうん？」と興奮して試してみたのがこちらの記事となります。

Splunkド初心者の自分には結構敷居が高かったのですが、<strong>AWS統合サービスとして用意されている「Splunk App for AWS」を使うことで、AWS上の下記いろんなサービスのログを扱うことができます。</strong>

- Billing
- CloudTrail
- CloudWatch
- Cloudfront Access Logs
- Config
- Config Rules
- Description
- ELB Access Logs
- Inspector
- S3 Access Logs
- VPC Flow Logs

今回は、一番基本となるDescriptionの設定手順解説だけですが、この記事読めば他のログの分析も可能になるので、SplunkでのAWSサービス分析に興味があるかたは、お試しくださいませ。

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     Splunk AWSﾀﾞﾜｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>



## Splunk App for AWSとは

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://docs.splunk.com/Documentation/AWS" target="_blank">Splunk® App for AWS - Splunk Documentation</a>
</p>

Amazon Web Servicesにおける致命的な操作やセキュリティの情報を集約してダッシュボード参照できるアプリケーション。

- 予め定義されたリアルタイムに分析可能なレポート、アラート
- AWSインフラ全体を最適化し、問題を見つけるためのダッシュボード
- AWSリソースをベストプラクティスに基づいて運用するためのノウハウの提示

## Splunk App for AWSのインストール方法

公式ドキュメントはこちら。

<p style="background-color: #e8e8e8;padding: 12px;">
<a href="https://docs.splunk.com/Documentation/AWS/5.1.3/Installation/Installon-prem" target="_blank">Install the Splunk App for AWS on Splunk Enterprise - Splunk Documentation</a>
</p>


必要なアプリケーションは以下の２つ。それぞれ役割が違ってドキュメントも別々に用意されているので、この2つの違いは意識しておきましょう。

- <a href="https://splunkbase.splunk.com/app/1274/" target="_blank">Splunk App for AWS | Splunkbase</a>
	- 主にインプットされたデータからUIを提供する
- <a href="https://splunkbase.splunk.com/app/1876/" target="_blank">Splunk Add-on for Amazon Web Services | Splunkbase</a>
	- AWSからデータを取得するアドオン。入力設定はこちらで実施

それぞれのファイルをダウンロードし、Webコンソール上の「Appの管理」より、「ファイルからAppをインストール」するのが簡単です。

インストール直後、このようなダッシュボードが表示されます。まだ設定が完了していないので、各種情報が取得できていない状態です。

s010

通常の運用では、クラスターモードで構成している場合が多いと思うので、そういう場合は別途上記マニュアルを参照の上対応しましょう。


## Splunk Add-on for AWSの設定方法

ここまでAppのインストールが完了したら、実際にAWS環境にアクセスするための設定を実施していきます。ここからは、Add-onのマニュアルを参照して設定していきます（余談ですが、公式ドキュメント、AppのマニュアルとAdd-onのマニュアルがわかれてて、最初違いがよくわかんなかった…）。

<p style="background-color: #e8e8e8;padding: 12px;">
<a href="https://docs.splunk.com/Documentation/AddOns/released/AWS/Description" target="_blank">About the Splunk Add-on for Amazon Web Services - Splunk Documentation</a>
</p>

### Splunk EnterpriseがインストールされているEC2へのIAMロール設定

今回自分の環境では、取得したいAWSアカウント上のEC2にSplunk Enterpriseをインストールしているため、このインスタンスにIAMロールを割り当てます。割り当てるポリシーは、ここでは簡易的に<code>AdministratorAccess</code>としておきます。

AWSから取得するサービスごとに必要なポリシーは異なるので、そのあたりの詳細はこちらを参照してください。

- <a href="https://docs.splunk.com/Documentation/AddOns/released/AWS/ConfigureAWSpermissions" target="_blank">Configure AWS permissions for the Splunk Add-on for AWS - Splunk Documentation</a>

### （重要）Splunkに入力するAWSサービスの事前設定

AWS Config、AWS Config Rules、CloutTrailなど、あらかじめSplunkからログを取得させたいサービスは、事前にAWS側での設定が必要になります。下記を参照に設定しておきましょう。

- <a href="https://docs.splunk.com/Documentation/AddOns/released/AWS/ConfigureAWS" target="_blank">Configure AWS services for the Splunk Add-on for AWS - Splunk Documentation</a>

特に、Splunkへのデータ送信のためにSQSを介すものがほとんどなので、そのあたり一つ一つ確認していく必要があります。

### Splunk Add-onのAWSへのアクセス許可設定

グローバルメニューから、「Splunk Add-on for AWS」をクリック。

s040

上部のメニュー「設定」をクリックし、Account一覧に、EC2に設定したIAMロール（ここでは、<code>ec2-splunk-role</code>）が表示されていれば、アクセス許可設定はOKです。

s050

### （重要）Splunk Add-onのAWS インプットの作成

いよいよ、Splunk Add-on for AWSのインプットの作成をやります。これを実施して初めて、Splunk側のデータソースに対してAWS側のデータが入力されます。ドキュメントはこちら。

- <a href="https://docs.splunk.com/Documentation/AddOns/released/AWS/ConfigureInputs" target="_blank">Configure inputs for the Splunk Add-on for AWS - Splunk Documentation</a>

Add-onのWebコンソールから、「Create New Input」ボタンをクリック。

s060

インプットに設定できるData Typeの一覧が表示されます。ここでは、一番お手軽に設定できる「Description」をクリック。

s070

設定画面が表示されるので、インプットの名前、AWS ACcount、Assume Role（オプション）、AWS Regionsを埋めていきます。また、取得対象のAPIをチェックし、「保存」ボタンをクリック。

S080

デフォルトの設定だとインターバルが3600秒なので、データがSplunkに送られてくるのをしばらく待ちます。

しばらく待った後、サーチ画面を起動。こんな感じで、ソースタイプに<code>aws:description</code>が増えていたら、無事Splunkにデータが入力されています。

S090

<code>aws:description</code>をクリックすると、送信データが表示されます。後は、Splunkの世界ですな。自由にログを検索できます。

s100

### Splunk App for AWSでのダッシュボード表示

<code>aws:description</code>のSplunkへの取り込みが確認できたら、Splunk App for AWSでの表示をみてみます。

グローバルメニューから、Splunk App for AWSをクリック。すると、Compute InstancesとStorageに値が取得できていることが確認できると思います。

s120

もちろんドリルダウンも可能で、クリックすることで、さらに詳細を表示することができます。

s130

ダッシュボード上表示されていないデータは、Splunk Add-onでのAWSインプットが未作成なのが原因なので、表示したいものについては、適宜インプットを追加していきながら対応していくことになります。

## AWSの各種情報をSplunkで統合管理するための第一歩として

以上、SplunkのAWS統合AppとAdd-onの基本的な使い方を紹介してきました。AWS用のAppとしては、この記事で紹介したもの以外にも、以下のAppが用意されています。

- <a href="https://splunkbase.splunk.com/app/3719/" target="_blank">Splunk Add-on for Amazon Kinesis Firehose | Splunkbase</a>
	- Kinesis Firehose経由のデータをインプットするApp
	- Kinesis Firehoseに流せるデータはすべてインプットできるので、汎用性が高い
- <a href="https://splunkbase.splunk.com/app/3790/" target="_blank">Amazon GuardDuty Add-on for Splunk | Splunkbase</a>
	- GuardDutyのデータをインプットするApp
	- GuardDutyのイベントを検知してアラームを起こすなどの対応が可能
	- こちらの記事が非常に参考になります
	- <a href="https://qiita.com/kikeyama/items/a0dc86e49197d1811ebe" target="_blank">Amazon GuardDutyのイベントをSplunkで検索・可視化 - Qiita</a>

Splunkを全く知らない場合は、各種設定の意味がわかりにくく苦労すると思うので、先に公式のチュートリアル（<a href="https://docs.splunk.com/Documentation/Splunk/7.2.6/SearchTutorial/WelcometotheSearchTutorial" target="_blank">About the Search Tutorial - Splunk Documentation</a>）を実施しておくなどしたほうが、理解は早いかと思います。

チュートリアルを実施してみた記事として、こちらも参考にしてみてください。

- <a href="https://dev.classmethod.jp/etc/about-splunk/" target="_blank">統合ログ分析ソリューションSplunkの凄さを公式チュートリアルで体感する</a>

また、シングルインスタンスモードではなく、Splunkをクラスター運用する場合は、こちらのQuickStartが全て網羅しており構築がめちゃくちゃ簡単なので、クラスターを自分で構築する前にまずは参照しておくことをおすすめします。

- <a href="https://aws.amazon.com/jp/quickstart/architecture/splunk-enterprise/" target="_blank">AWS での Splunk Enterprise - クイックスタート</a>

それでは、皆さんに良きSplunkライフを。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。




