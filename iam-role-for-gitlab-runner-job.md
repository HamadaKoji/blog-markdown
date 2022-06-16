# GitLab RunnerのジョブにAWSのIAMロールを適用し権限を適切に管理する

「この環境のGitLab Runnerのジョブ、めっちゃ強権限もってるやん。駄目だよ。怖いわ」<br />
「えー、NodeのIAMロールに全部ひっくるめて設定してるんじゃだめ？」<br />
「あかんよ。それぞれのジョブはやること全然違うんだから」

現プロジェクトでは、GitLab RunnerをEKSでセルフホスティングで運用しているんですが、ジョブの数が100を超えてきたところから、こんな課題が出てきました。

以前から課題感はあったのですが、根本的にやり方を変えて、各ジョブで既定のIAMロールのみを利用してジョブが起動するように対応したので、その軌跡をお伝えします。

EKSにおいて、Podに適切にIAMロールを適用して運用するときにも汎用的に使えるノウハウなので、このあたり興味がある方は是非、この記事参考にしてくださいませ。

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     IAM Role ﾏﾂﾘﾀﾞﾜｯｼｮｲ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>

## このブログの範囲

このブログでは、以下の流れで、GitLab RunnerのジョブにAWSのIAMロールを適用する方法を解説していきます。いろんな要素がごっちゃにならないよう順序立てて書いていくので、この全体像を最初におさえておいてください。

- そもそも、なぜジョブにAWSのIAMロールを紐付けたいのか？
- GitLab RunnerのジョブとKubernetes Podの関係
- IAMロールをKubernetes Service Accountに紐づけてPodを起動する方法を理解する
- 実際に既存のEKSクラスターに設定してみる
- ジョブを起動し、IAMロールが実際に利用されているか動作確認する


## そもそも、なぜジョブにAWSのIAMロールを紐付けたいのか？

割り切った運用するならば、以下の方法もありです（以前はこうだった）。

1. GitLab Runnerを動作させるEKSのノード（EC2）にIAMロールを適用
2. ジョブで利用するPolicyを上記IAMロールに付与
3. ジョブ起動時、EKSノードの<code>kubelet</code>デーモンが、自動的にAWS APIへの呼び出しを実行

設定方法や動作原理は、<a href="https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/create-node-role.html">Amazon EKS ノードの IAM ロール</a>を参照ください。

ただ、上記運用では、以下が問題になってきました

### 問題点1：1IAMロールあたりのIAMポリシー数のハードリミット（上限20）に抵触

一つのIAMロールにアタッチできるIAMポリシーの数はデフォルトで10、上限緩和しても最大20です（参考：[IAM と AWS STSクォータ](https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/reference_iam-quotas.html#reference_iam-quotas-entities)）

「20あったら十分だろう…」と思ったそこのあなた！自分も半年前まではそう思ってました。しかし、EKSのノードで利用するIAMロールには、<a href="https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/create-node-role.html">Amazon EKS ノードの IAM ロール</a>にあるように、以下の３種類が必要。

- AmazonEKSWorkerNodePolicy
- AmazonEC2ContainerRegistryReadOnly
- AmazonEKS_CNI_Policy 

さらにこのEKSでホストするGitLab Runnerは、顧客環境の新業務の様々なサブシステムのCIで利用するため、サブシステムが増えてきたところ、早くも上限の20に達してしまったというわけでした。

### 問題点2：ジョブの最小権限の法則を適用するため

NodeのIAMロールを利用する場合、そのNode上で動くKubernetesクラスターの全Podに同じIAMポリシーが適用されます。複数サブシステムでの利用を想定しているGitLab Runnerにおいて、全てのジョブが同一権限で動作するのは最小権限の法則に反するため、セキュリティ上あまり好ましくない状態でもありました。

これら問題を解決するため、GitLab Runnerの個別のジョブにIAMロールを適用させる検討をはじめました。

## GitLab RunnerのジョブとKubernetes Podの関係

GitLab RunnerをKubernetes Executorで動かしている場合、各ジョブの起動時にKubernetesのPodが起動し、そのPodの中でジョブが動作します。

動作のシーケンスなど詳細は、こちらのブログでまとめているので参考にしてください。

[GitLab RunnerのKubernetes Executorを完全に理解してカスタマイズの基礎を身につける \| DevelopersIO](https://dev.classmethod.jp/articles/kubernetes-executor/)


## IAMロールをKubernetes Service Accountに紐づけてPodを起動する方法を理解する

ジョブへのIAMロール適用方法するには、前提として、Podが動作するService Accountに対してIAMロールを適用する必要があります。これについては、AWSのこの公式ブログが素晴らしく詳細でわかりやすいです。まじで神記事ですよ。

[詳解: IAM Roles for Service Accounts \| Amazon Web Services ブログ](https://aws.amazon.com/jp/blogs/news/diving-into-iam-roles-for-service-accounts/)

実際に手を動かしながら、PodにマウントされるJWTトークンの検証などを通じて動作を理解する事ができるので、このあたり好きな方は是非、実際に手を動かしながら確認することをオススメします。めっちゃ面白いです。興奮します。

記事内容をざっくり要約すると、こんな感じです。

### PodがService Accountを通じて、Kuberntes APIサーバーと通信する仕組み

- Kubernetes Podは、Kubernetes Service Accountと呼ばれるKubernetesのコンセプトを通じてIDが付与される
- Service Accountが作成されると、JWTトークンがKubernetes Secretとして自動的に作成される
- このSecretをPodにマウントし、Secret内のService AccountのJWTトークンをKubernetes APIサーバーの認証に使用することができる

このデフォルトトークンを検証できるのはKubernetes APIサーバーのみなので、IAM認証には使用できません。ので、Service AccountでIAMロールを利用するためには、他の仕組みが必要です。

### Service Account側で準備する必要があるもの

- PodにマウントされているトークンはOIDCに完全に準拠しているが、AWS APIの利用に必要な2つ目のトークンをPodに注入するには、別の仕組みが必要
- EKSクラスターで有効化されている[Amazon EKS Pod Identity Webhook](https://github.com/aws/amazon-eks-pod-identity-webhook/)を利用して、追加のトークンを注入する
- Service AccountにAWS IAMロールのARNをAnnotationsとして付与することで、Pod起動時にAWS APIの利用に必要な追加のトークンをPodに注入できる

### IAMロール作成時に設定が必要なもの

- IAMロールの信頼関係ポリシーに上記で作成したServcice Accountの名前を指定し、ロールを引き受け可能とする

### EKSクラスターに設定が必要なもの

- EKSクラスターにOIDC ID プロバイダーを関連付ける

以上、ざっくり要約でした。どのリソースに対して、どういう設定が必要なのか流れがわかればここではOKです。

## 実際に既存のEKSクラスターに設定してみる

ここでは、実際に既存のEKSクラスターに、設定を実施していく様子を紹介します。上で紹介したAWS公式ブログでもチュートリアル形式で紹介されていて、基本同じ内容になっていますが、ここではeksctlを使わない個別設定の方法を紹介します。

何かしらのEKSクラスターが起動していて、GitLab Runnerがインストールされている状態を前提とします。

### 手順①：OIDCプロバイダーの作成

マネジメントコンソールで、EKSコンソールを開き、対象のクラスターを選択します。そうすると、ここにOpenID Connect プロバイダーURLが表示されているため、この値をコピーします。

010.png

その後、マネジメントコンソールで、IAMコンソールを開きます。[アクセス管理]->[IDプロバイダ]を選択し、[プロバイダを追加]ボタンをクリック。

<p style="background-color: #e8e8e8;padding: 12px;">
注意：このとき、クラスターの作成方法によっては、すでに対象EKSクラスターのIDプロバイダ設定が完了している場合があります。その時は、この手順は無視してください。
</p>

020.png

- プロバイダのタイプで[OpenID Connect]を選択
- [プロバイダのURL]に、EKSクラスター画面で取得した、プロバイダーURLを入力
- [サムプリントを取得]をクリック
- [対象者]に、<code>sts.amazonaws.com</code>を入力

上記完了したら、[プロバイダを追加]をクリックして、OIDCプロバイダを作成します。ID プロバイダの一覧に作成したプロバイダが表示されればOKです。

030.png

### 手順②：IAMロールの作成

適当なポリシーが設定されているIAMロールを作成します。ここでは、<code>test-iamrole</code>という名前で、アタッチポリシーに<code>AmazonS3ReadOnlyAccess</code>を付与します。

IAMロールの作成画面で、以下を入力します。

040.png

- 信頼されたエンティティタイプ：ウェブアイデンティティ
- アイデンティティプロバイダー：手順①で作成したOIDCプロバイダーを選択
- Audiens：<code>sts.amazonaws.com</code>を選択

許可ポリシーで、<code>AmazonS3ReadOnlyAccess</code>を選択。

ロール名に<code>test-iamrole</code>を入力後、信頼ポリシーを以下のように編集します。

[json highlight="14"]
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "clusterB",
            "Effect": "Allow",
            "Principal": {
                "Federated": "<OIDC_PROVIDER_ARN>"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "<OIDC_PROVIDER_ID>:aud": "sts.amazonaws.com",
                    "<OIDC_PROVIDER_ID>:sub": "<KUBERNETES_SERVICE_ACCOUNT_NAMESPACE:KUBERNETES_SERVICE_ACCOUNT_NAME>"
                }
            }
        },
[/json]

上記ポリシーの入力項目はこちら。この信頼ポリシーをIAMロールに指定することで、Service AccountはこのIAMロールを引き受けるトークンをPodに渡して起動することが可能になります。

- <code>OIDC_PROVIDER_ARN</code>
  - OIDCプロバイダーのARN
- <code>OIDC_PROVIDER_ID</code>
  - OIDCプロバイダーID
- <code>KUBERNETES_SERVICE_ACCOUNT_NAMESPACE:KUBERNETES_SERVICE_ACCOUNT_NAME</code>
  - このIAMロールを利用するService Accountの名前

以下に、信頼ポリシーの具体例を挙げておくので、上記ポリシー編集の参考にしてください。

Service AccountのNamespaceは<code>gitlab-runner</code>、Service Account名は<code>test-iamrole-sa</code>で指定しています。

[json title="信頼関係ポリシーの具体例"]
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "clusterB",
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::123456789012:oidc-provider/oidc.eks.us-east-1.amazonaws.com/id/EXAMPLEOIDCID77E02783D8273C"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "oidc.eks.us-east-1.amazonaws.com/id/EXAMPLEOIDCID77E02783D8273C:aud": "sts.amazonaws.com",
                    "oidc.eks.us-east-1.amazonaws.com/id/EXAMPLEOIDCID2963677E02783D8273C:sub": "system:serviceaccount:gitlab-runner:test-iamrole-sa"
                }
            }
        }
[/json]

### 手順③：Service Accountの作成

最後に、上記IAMロールを引き受けるService Accountを作成します。Kubernetesリソースなので<code>kubectl apply</code>で作成します。用意するマニフェストファイルはこうなります。

[yaml title="test-iamrole-sa.yaml"]
apiVersion: v1
kind: ServiceAccount
metadata:
	name: test-iamrole-sa
	namespace: gitlab-runner 
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/test-iamrole
[/yaml]

- <code>name</code>
  - Service Accountの名前を指定
- <code>namespace</code>
  - GitLab Runnerをインストールした名前空間を指定します。ここがあっていないと、GitLab Runnerのジョブ起動時にPodがService Accountを利用できません
  - ここでは、GitLab Runnerを名前空間<code>gitlab-runner</code>にインストールした前提の記述になっています
- <code>annotations</code>
  - 手順②で作成したIAMロールのARNをannotationsとして指定
  - Keyに<code>eks.amazonaws.com/role-arn</code>、ValueにIAMロールのARNを指定
  - 具体的なサンプルは、上のマニフェストファイルを参照

あとは、<code>kubectl apply</code>で、Service Accountを作成します。

[bash]
$ kubectl apply -f test-iamrole-sa.yaml
[/bash]

作成したService Accountが意図したとおり設定されているか、確認します。Annotationsなど、確認しておきましょう。

[bash]
$ kubectl describe serviceaccount test-iamrole-sa -n gitlab-runner
Name:                test-iamrole-sa
Namespace:           gitlab-runner
Labels:              <none>
Annotations:         eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/test-iamrole
Image pull secrets:  <none>
Mountable secrets:   kcr-cs-sa-token-6sjq9
Tokens:              kcr-cs-sa-token-6sjq9
Events:              <none>
[/bash]

### 手順④：Service Accountの動作確認

ここまでで、Service AccountがIAMロールを利用できるための設定は完了しています。AWS CLIを利用できるPodを利用して、意図したIAMロールが利用できるか確認します。

以下のコマンドで、PodからAWS CLIの<code>s3 ls</code>を実行し、動作確認します。

[bash]
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: test-iamrole
  namespace: gitlab-runner
spec:
  serviceAccountName: test-iamrole-sa
  containers:
    - name: my-aws-cli
      image: amazon/aws-cli:latest
      args: ['s3', 'ls']
  restartPolicy: Never
EOF
[/bash]

<code>test-iamrole</code>PodのStatusがCompletedになっているか確認。

[bash]
$ kubectl get pod -n gitlab-runner
NAME                                          READY   STATUS      RESTARTS   AGE
gitlab-runner-gitlab-runner-65cbff94b-zp2vn   1/1     Running     0          2d7h
test-iamrole                                  0/1     Completed   0          9s
[/bash]

ログを確認し、無事<code>s3 ls</code>の実行結果が確認できれば、Podで利用したService AccountからのIAMロール動作確認はOKです！！

[bash]
$ kubectl logs test-iamrole -n gitlab-runner
2022-01-19 09:42:22 s3bucketXXXX
2022-01-19 09:42:22 s3bucketYYYY
2022-01-19 09:42:22 s3bucketZZZZ
[/bash]

## ジョブを起動し、IAMロールが実際に利用されているか動作確認する

ここまでで、一通りの設定は完了しました。あとは、この記事の最終目的である「GitLab RunnerのジョブでIAMロールを利用する」設定です。

Kubernetes Executorの動作に関しての詳細は、以前書いたこちらのブログを参考にしてください。

[GitLab RunnerのKubernetes Executorを完全に理解してカスタマイズの基礎を身につける \| DevelopersIO](https://dev.classmethod.jp/articles/kubernetes-executor/)

GitLab RunnerのKubernetes Executorの内部では、普段GitLabからのジョブ起動を監視するためのPodが常駐しています。ジョブが起動された場合、そのジョブの情報を取得して、Executorの中でジョブ単位でPodが起動して、そこでジョブが実行されます。

GitLab Runnerのデフォルトの設定では、Runner本体のPodが利用しているService Accountでジョブ用のPodも起動するため、ジョブで個別のService Accountを利用するには、別の設定が必要になります。

### GitLab Runnerの設定を変更し、ジョブからService Accountを指定できるようにする

GitLab RunnerのKubernetes Executorの公式マニュアルはこちら。

[The Kubernetes executor for GitLab Runner \| GitLab](https://docs.gitlab.com/runner/executors/kubernetes.html)

- <code>service_account_overwrite_allowed</code>
  - Regular expression to validate the contents of the service account overwrite environment variable. When empty, it disables the service account overwrite feature.

以下に、RunnerのConfigの設定例を記載します。

[yaml highlight="16"]
runners:
  # runner configuration, where the multi line strings is evaluated as
  # template so you can specify helm values inside of it.
  #
  # tpl: https://helm.sh/docs/howto/charts_tips_and_tricks/#using-the-tpl-function
  # runner configuration: https://docs.gitlab.com/runner/configuration/advanced-configuration.html
  config: |
    [[runners]]
      [runners.kubernetes]
        image = "ubuntu:16.04"
        cpu_request = "500m"
        cpu_request_overwrite_max_allowed = "4000m"
        memory_request = "2Gi"
        memory_request_overwrite_max_allowed = "8Gi"
        # gitlab.ci.ymlでのServiceAccountの上書きを許可
        service_account_overwrite_allowed = ".*"
[/yaml]

<code>service_account_overwrite_allowed = ".*"</code>で、全てのServiceAccountに対しての上書き設定を許可します。

上記設定をGitLab RunnerにApplyしておきます。

後は、最後の仕上げです。ジョブで利用する<code>.gitlab-ci.yml</code>の冒頭で、<code>KUBERNETES_SERVICE_ACCOUNT_OVERWRITE</code>に、Servcie Accountを指定することで、そのジョブで任意のService Accountを利用したPodを起動できます。

以下に、サンプルのジョブを記載します。前手順で作成したService Accountの<code>test-iamrole-sa</code>を利用して、AWS CLIの<code>s3 ls</code>を実行します。

[bash title="index.html"]
variables:
  KUBERNETES_SERVICE_ACCOUNT_OVERWRITE: test-iamrole-sa

stages:
  - test

gitlab-runner-s3-ls:
  image:
    name: "amazon/aws-cli"
  stage: test
  script:
    - aws s3 ls
[/bash]

このジョブが無事完了したでしょうか。

これにて、当初の目的「GitLab Runnerのジョブを、任意のIAMロールを指定して実行する」の完了です。お疲れさまでした！

## 実運用に向けての課題感と次回予告

ここまで設定方法を細かく書いてきましたが、今後の実運用を考えるとこんな課題があるかなと思います。

- 利用するIAMロールが増えたときに都度、Service Accontを作る必要がある
- Service AccountというKubernetesリソースと、IAMロールのAWSリソースを両方、整合性もって更新する必要がある
- <code>eksctl</code>で両方同時に作成できるが、この方法はCLIベースになるので、IaCで各種リソース管理する前提だと、運用がつらい

AWSリソースとKubernetesリソースを、両方整合性をもって同じデベロパーエクスペリエンスで管理したい場合、このあたりがネックになりがちです。

今の顧客プロジェクトでは、このあたりをTerraformのAWSプロバイダーとKubernetesプロバイダーで一括管理することで解消しています。次の記事では、この構成を全てTerraformで管理するためのサンプルコードを紹介しますので、乞うご期待！

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。


## 参考記事

- [\[アップデート\] EKSでIAMロールを使ったPod単位のアクセス制御が可能になりました！ \| DevelopersIO](https://dev.classmethod.jp/articles/eks-supports-iam-roles-for-service-accounts/)
- [詳解: IAM Roles for Service Accounts \| Amazon Web Services ブログ](https://aws.amazon.com/jp/blogs/news/diving-into-iam-roles-for-service-accounts/)
- [Introducing fine\-grained IAM roles for service accounts \| AWS Open Source Blog](https://aws.amazon.com/jp/blogs/opensource/introducing-fine-grained-iam-roles-service-accounts/)
- 