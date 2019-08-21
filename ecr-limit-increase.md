# 4時間かけてECRにイメージを1万個プッシュしほんとにエラーが出るか確かめてみた

みんな大好き、Amazon ECRですが、先日のアップデートでリポジトリとイメージの制限が引き上げられました。

<strong><a href="https://aws.amazon.com/jp/about-aws/whats-new/2019/07/amazon-ecr-now-supports-increased-repository-and-image-limits/" target="_blank">Amazon ECR で、リポジトリとイメージの制限の引き上げをサポート</a></strong>

これで、ECRのイメージ数上限に引っかかって夜中に叩き起こされるインフラ担当が少しでも減ることをハマコーは祈っております。

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     ECRﾏﾂﾘﾀﾞﾜｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>

## ECRのリポジトリ数とイメージ数のデフォルト上限緩和

今回のアップデートで、従来は1リージョンあたりのリポジトリ数、および1リポジトリあたりのイメージ数のデフォルト上限値が1000だったものが、両方とも10000に引き上げられ<strong>従来の10倍に制限緩和されました。</strong>以下の公式ドキュメントに記載があります。

- <a href="https://docs.aws.amazon.com/AmazonECR/latest/userguide/service_limits.html" target="_blank">Amazon ECR Service Limits - Amazon ECR</a>

おそらくリポジトリ数の上限1000に引っかかることはあまりないと思いますが、1リポジトリあたりのイメージ上限数1000は、頻繁に変更がありイメージビルドが自動化された環境だとこの制限にひっかかったのは聞いたことがあります。この場合、CI/CDパイプラインが止まってしまうので非常に致命的ですね。

この上限が10000に引き上げられたことで、そういった事故も防ぐことができるのではないでしょうか。

## 1リポジトリにイメージを1万個プッシュしてみた

というわけで、実際に上限が1万になっているか確認してみます。実施環境は以下の通り。

[bash]
$ aws --version
aws-cli/1.16.207 Python/3.6.4 Darwin/18.6.0 botocore/1.12.197
[/bash]

### ECRリポジトリの作成

イメージ制限テスト用のECRを作成します。名前は、<code>test-image-limit-repo</code>で作成。

[bash]
$ aws ecr create-repository --repository-name test-image-limit-repo
{
    "repository": {
        "repositoryArn": "arn:aws:ecr:ap-northeast-1:123456789012:repository/test-image-limit-repo",
        "registryId": "123456789012",
        "repositoryName": "test-image-limit-repo",
        "repositoryUri": "123456789012.dkr.ecr.ap-northeast-1.amazonaws.com/test-image-limit-repo",
        "createdAt": 1564186279.0,
        "imageTagMutability": "MUTABLE"
    }
}
[/bash]

### 事前に1万個イメージをプッシュしたときの料金を試算

1万個イメージをプッシュする時、料金がどれぐらいかかるかは気になるので、事前に調査。

- <a href="https://aws.amazon.com/jp/ecr/pricing/" target="_blank">料金 - Amazon ECR | AWS</a>

- ストレージコストは1ヶ月あたりGB単位で0.1USD
- すべてのデータ転送受信（イン）は無料

今回プッシュするイメージは、以下の<code>scratch</code>を利用したダミーファイル。これだと、作成したイメージのサイズは0になります。

[bash title="Dockerfile"]
FROM scratch
MAINTAINER hamako1 
[/bash]

実際にECRにプッシュしてみます。ECRにログイン。

[bash]
$ $(aws ecr get-login --no-include-email --region ap-northeast-1)
[/bash]

Dockerfileは以下で用意します。

[bash title="Dockerfile"]
FROM scratch
MAINTAINER hamako
[/bash]

DockerbuildしてECRにプッシュします。

[bash]
docker build -t 123456789012.dkr.ecr.ap-northeast-1.amazonaws.com/test-image-limit-repo:1 .
docker push 123456789012.dkr.ecr.ap-northeast-1.amazonaws.com/test-image-limit-repo:1
The push refers to repository [123456789012.dkr.ecr.ap-northeast-1.amazonaws.com/test-image-limit-repo]
1: digest: sha256:fef734996a075033ea83ccce717e3e6c8db2ea8c3c1e7d9ce7723aa6818f9687 size: 617
[/bash]

プッシュ後、イメージサイズを確認すると、<code>ImageSizeInBytes</code>が、<code>32Bytes</code>になっていました。イメージの内容自体は0Byteでもメタ情報をもつのか、ECR上ではサイズが増えました。ただ、料金は気にしなくて良さそうです。

[bash]
$ aws ecr describe-images --repository-name test-image-limit-repo
{
    "imageDetails": [
        {
            "registryId": "123456789012",
            "repositoryName": "test-image-limit-repo",
            "imageDigest": "sha256:fef734996a075033ea83ccce717e3e6c8db2ea8c3c1e7d9ce7723aa6818f9687",
            "imageTags": [
                "1"
            ],
            "imageSizeInBytes": 32,
            "imagePushedAt": 1564212305.0
        }
    ]
}
[/bash]


### 実際に1万個イメージを作ってプッシュする

以下のシェルスクリプトを用意しました。

[bash title="index.html"]
#! /bin/bash

REPOSITORY_URI=123456789012.dkr.ecr.ap-northeast-1.amazonaws.com/test-image-limit-repo
$(aws ecr get-login --no-include-email --region ap-northeast-1)

for count in {1..10000};do

# write Dockerfile
cat << EOS > Dockerfile
FROM scratch
MAINTAINER hamako${count}
EOS

# docker build and push
docker build -t ${REPOSITORY_URI}:${count} .
docker push ${REPOSITORY_URI}:${count}

done
[/bash]

最初は、同じイメージを1万回タグを変えてプッシュすれば良いかと思ったのですが、同じハッシュイメージの場合ダイジェストが同じとなり、リポジトリ上では同一イメージとして判定されイメージ数が増えませんでした。そのため、1万回のループの中で、Dockerfileを都度書き換えてイメージのハッシュ値を変更してビルドしてプッシュしています。これにより、リポジトリ中のイメージ数を増やすことができます。

後は、このシェルスクリプトを実行して、1万イメージアップされるのを待ちます。実際やってみるとわかるのですが、都度Dockerbuildしてアップロードしているので、1メージあたり3秒程度かかります。そんなの1万個も待ってられないので、実際は10シェルに分けて並列アップロードしました。

### ついに1万個プッシュされ、その時が！！

徐々に増えるイメージ数…　あとちょっとやで…（この画面はWebコンソールのリポジトリを選択したときの数字。イメージ数をカウントするAPIがどこにあるのかわかりません！）

010.png

よーやく、1万個イメージがあがりました！！

020.png

コンソールをみると、無事、エラーが表示されました！！やっほい！！

030.png

さらにアップ！！

040.png

エラー内容としては以下の通り。

[bash gutter="false"]
denied: The repository with name 'test-image-limit-repo' in registry with id '123456789012' already has the maximum allowed number of images which is '10000'
[/bash]

もう満足やで。ターミナルで並列実行してたけど、途中でECRのログインセッションがきれたり、assume-roleが切れたりして、なんだかんだ4時間ほどかかりました。

## イメージ数が限界になる前にやるべきライフサイクルポリシーの設定

今回、むりやりイメージを1万個入れてみましたが、実際問題1万個もイメージが必要となる機会も無いでしょうし、イメージサイズによりもちろんECRのストレージ料金もかかります。

ので、実際にはECRのライフサイクルポリシーを設定して、古くなったイメージは自動的に削除するのが良いでしょう。こちらを記事を参考に事前の設定をオススメします。

https://dev.classmethod.jp/cloud/aws/ecr-lifecycle/

## 地味だけど進化を止めないECRがよござんす

ECRはDocker Hubのようにパブリックにイメージを公開することはできませんが、AWSのコンテナワークロード（ECS、Fargate、EKS）を運用する上では無くてはならないサービスです。地味に進化しているので、これからもECRのアップデート情報、追い続けていきます。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

