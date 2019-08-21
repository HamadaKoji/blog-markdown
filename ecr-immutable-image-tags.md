# latestタグ絶対禁止！？ECRでコンテナイメージタグの上書き禁止設定がサポートされました！

アップデート続きのECRですが、先日、コンテナのイメージタグを上書き禁止にする設定がサポートされました。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://aws.amazon.com/jp/about-aws/whats-new/2019/07/amazon-ecr-now-supports-immutable-image-tags/" target="_blank">Amazon ECR Now Supports Immutable Image Tags</a>
</p>

この設定をしたECRではタグの別イメージへの付け替えが禁止されます。latestタグを使った運用もできなくなります。

<pre style="line-height:120%;">
ええ！latest、もう、使えないの…！？

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>

別に設定しなければ、従来どおり使えますYO

## イメージタグの上書き禁止設定は何が嬉しいのか？

従来は、リポジトリに対してプッシュしたイメージにつけたタグを他のイメージに付け替えることが容易に可能でした。ということは、開発者がどのイメージが実際に運用環境で展開し動いているかどうかを厳密に知るためには、イメージのSHAの利用が必須でした。

<strong>Amazon ECRでイメージタグを上書き禁止の設定にすることで、開発チームが作成したイメージのバージョンと、実際に運用されているイメージの識別をイメージタグを利用して実施することができるようになります。</strong>

設定方法の詳細など、公式ドキュメントはこちら。

- <a href="https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-tag-mutability.html" target="_blank">Image Tag Mutability - Amazon ECR</a>


## イメージタグの上書き禁止設定方法

ECRリポジトリの新規作成時、および既存のリポジトリの変更の両方に対応しています。

### 新規作成時の設定方法

#### Webコンソールから新規作成

リポジトリの新規作成時に「イメージタグの変更可能性」に関する項目が増えているので、ここで「変更不可能」を選択します。

010.png

#### AWWS CLIから新規作成

AWS CLIを最新版にバージョンアップしておきます。

[bash]
$ aws --version
aws-cli/1.16.207 Python/3.6.4 Darwin/18.6.0 botocore/1.12.197
[/bash]

<code>create-repository</code>コマンドにあらたに、<code>--image-tag-mutability</code>オプションが増えているので、それを使います。

- <a href="https://docs.aws.amazon.com/cli/latest/reference/ecr/create-repository.html" target="_blank">create-repository — AWS CLI 1.16.207 Command Reference</a>

コマンド例はこちら。<code>immutable-repo</code>リポジトリを<code>IMMUTABLE</code>として作成します。

[bash]
$ aws ecr create-repository --repository-name immutable-repo --image-tag-mutability IMMUTABLE
{
    "repository": {
        "repositoryArn": "arn:aws:ecr:ap-northeast-1:629895769338:repository/immutable-repo",
        "registryId": "123456789012",
        "repositoryName": "immutable-repo",
        "repositoryUri": "123456789012.dkr.ecr.ap-northeast-1.amazonaws.com/immutable-repo",
        "createdAt": 1564290230.0,
        "imageTagMutability": "IMMUTABLE"
    }
}
[/bash]

#### Webコンソールから既存のリポジトリを変更

既存のリポジトリを選択して「編集」ボタンを押すと、イメージタグの変更可能性を編集できます。

020.png

#### AWS CLIから既存のリポジトリを変更

<code>put-image-tag-mutability</code>コマンドを使います。

[bash]
$ aws ecr put-image-tag-mutability --repository-name immutable-repo --image-tag-mutability IMMUTABLE
{
    "registryId": "123456789012",
    "repositoryName": "immutable-repo",
    "imageTagMutability": "IMMUTABLE"
}
[/bash]

## イメージタグの変更不可能性の動作を確認

実際に、イメージタグの変更不可能性の動作を確認してみます。<code>IMMUTABLE</code>なリポジトリ<code>immutable-repo</code>を使います。

以下のDockerfileを用意します。

[bash title="Dockerfile"]
FROM scratch

LABEL maintainer "hamako0001"
[/bash]

ECRのプッシュコマンドに則り、ログインしてタグに<code>latest</code>を付けて、プッシュします。

[bash]
$ $(aws ecr get-login --no-include-email --region ap-northeast-1)
$ docker build -t immutable-repo .
$ docker tag immutable-repo:latest 629895769338.dkr.ecr.ap-northeast-1.amazonaws.com/immutable-repo:latest
$ docker push 629895769338.dkr.ecr.ap-northeast-1.amazonaws.com/immutable-repo:latest
[/bash]

リポジトリの内容を確認すると、<code>latest</code>タグでイメージが1つ登録されていることがわかります。

[bash]
$ aws ecr describe-images --repository-name immutable-repo
{
    "imageDetails": [
        {
            "registryId": "123456789012",
            "repositoryName": "immutable-repo",
            "imageDigest": "sha256:ebce9620b3c20dcdea9ff4653e8740ec76cbf2594f0f4e328b3dbfb8d9f1ed8f",
            "imageTags": [
                "latest"
            ],
            "imageSizeInBytes": 32,
            "imagePushedAt": 1564290849.0
        }
    ]
}
[/bash]

この状態で、先程ビルドしたDockerfileの内容を変更して別のイメージを作成します。maintainerの内容を変更したDockerfileを用意。

[bash title="Dockerfile"]
FROM scratch

LABEL maintainer "hamako0002"
[/bash]

上の手順と同様に、ビルドしてプッシュしようとすると、<code>tag invalid</code>のエラーが表示されます。

[bash]
$ $(aws ecr get-login --no-include-email --region ap-northeast-1)
$ docker build -t immutable-repo .
$ docker tag immutable-repo:latest 629895769338.dkr.ecr.ap-northeast-1.amazonaws.com/immutable-repo:latest
$ docker push 629895769338.dkr.ecr.ap-northeast-1.amazonaws.com/immutable-repo:latest
The push refers to repository [629895769338.dkr.ecr.ap-northeast-1.amazonaws.com/immutable-repo]
tag invalid: The image tag 'latest' already exists in the 'immutable-repo' repository and cannot be overwritten because the repository is immutable.
hamada.koji@HL00228:~/prj/hamada[/bash]

<strong>このように、イメージタグの変更可能性を変更不可能に設定（"imageTagMutability"が"IMMUTABLE"）なECRリポジトリに対しては、latestタグを別イメージにつけてプッシュすることができなくなります。</strong>

## タグの変更を不可能にすることによる、より安全で明確なコンテナ運用が可能に

コンテナワークロードにおいては、ソースコードとコンテナイメージのトレーサビリティを確保するため、CI/CDの中でDockerビルド時のgitのコミットハッシュやタグの名前を、イメージタグにつける場合が多いかと思います。

上で見たとおり、タグの変更が不可能に設定されたECRでは、イメージタグのlatestを利用した運用が事実上できなくなります。また、一度イメージに付与したタグの変更もできません。例えばECSにおいても、コンテナ定義でのlatest指定も使えないので、新しいイメージがビルドされるたびに、タスク定義の変更からサービス定義の変更が必須となります。

<strong>一見これは運用がめんどくさくなりそうですが、これを強制しておくことで、ソースコードとコンテナイメージの一貫性の把握が容易になり、トラブルシューティング時に無駄な時間を使うことが減るでしょう。また、イメージタグの変更を禁止しておくことで、イレギュラーな運用や不正イメージの混入などのセキュリティ上の脅威に対しても、一定の抑止効果が期待できます。</strong>

既存の運用環境に対していきなりこの設定変更をするのはリスクが高すぎますが、一度開発環境などでこの設定を試してみて、より明確で一貫性のあるコンテナ運用をすることができると思います。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

