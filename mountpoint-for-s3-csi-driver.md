<strong>「Kubernetesで大型のファイル扱うの面倒くさい。NodeのAMIに埋め込むとかじゃだめ？」</strong>

<strong>「やりたいことはわかるけど、運用めっちゃ面倒くさいでしょ」</strong>

大型のファイルを扱う場合、ストレージの料金や運用に手間がかかるため、オブジェクトストレージ（S3）に格納することはよくあるシチュエーションだと思います。

今回のアップデートで、S3のマウントポイントがCSI（Container Storage Interface）に対応し、Kubernetesの永続ボリュームとしてS3をマウントすることができるようになりました。

Kubernetesでホストするアプリケーションの活用方法が大幅に広がる意欲的なアップデートとなります。大型のファイルの扱いに困っていた方は、今回のアップデートを検証することをおすすめします。このブログでは、アップデートの概要と、公式ドキュメントに準ずる形で、実際の設定と動作確認をした様子をお届けします。

<pre style="line-height:120%;">
（祭） ∧ ∧
　Y　 ( ﾟДﾟ)
　Φ[_ｿ__ｙ_l〉     S3ﾏﾂﾘﾀﾞﾜｯｼｮｲ ﾅﾝﾃﾞﾓｵｸﾖ
　　　 |_|＿|
　　　 し'´Ｊ
</pre>

用法用量は守って適切にお使いくださいませ。


## アップデート内容

アップデート：[Mountpoint for Amazon S3 CSI driver is now generally available](https://aws.amazon.com/jp/about-aws/whats-new/2023/11/mountpoint-amazon-s3-csi-driver/)

<blockquote>With the new Mountpoint for Amazon S3 Container Storage Interface (CSI) driver, your Kubernetes applications can access S3 objects through a file system interface, achieving high aggregate throughput without any changes to your application. Built on Mountpoint for Amazon S3, the CSI driver presents an S3 bucket as a volume accessible by containers in Amazon Elastic Kubernetes Service (Amazon EKS) and self-managed Kubernetes clusters. As a result, distributed machine learning training jobs in Amazon EKS and self-managed Kubernetes clusters can read data from Amazon S3 at high throughput to accelerate training times.</blockquote>

以下、日本語訳。

<blockquote>新しいMountpoint for Amazon S3 Container Storage Interface（CSI）ドライバにより、Kubernetesアプリケーションはファイルシステムインタフェースを通じてS3オブジェクトにアクセスでき、アプリケーションに変更を加えることなく高い集約スループットを達成できます。Mountpoint for Amazon S3上に構築されたCSIドライバは、Amazon Elastic Kubernetes Service（Amazon EKS）および自己管理型Kubernetesクラスタのコンテナからアクセス可能なボリュームとしてS3バケットを提示します。その結果、Amazon EKSと自己管理型Kubernetesクラスタの分散型機械学習トレーニングジョブは、高いスループットでAmazon S3からデータを読み取り、トレーニング時間を短縮できる。</blockquote>

2023年の8月に、S3バケットをローカルファイルシステムとしてマウントするためのMountpoit for Amazon S3はリリースされていました（参考：[Mountpoint for Amazon S3 – Generally Available and Ready for Production Workloads \| AWS News Blog](https://aws.amazon.com/jp/blogs/aws/mountpoint-for-amazon-s3-generally-available-and-ready-for-production-workloads/)）。

今回は、Container Storegae Interface（CSI）に対応したことで、Kubernetesアプリケーションのファイルシステムから直接S3オブジェクトにアクセスできます。

### 主な制約

- Windowsコンテナ家m−じでは利用不可
- AWS Fargateは未サポート。EC2上で動作するKubernetesに対応
- 静的プロビジョニングのみに対応。動的プロビジョニング（新バケットの作成）は未対応

### 前提条件

- Kubernetesクラスター用のIAM OpenID Connect(OIDC)プロバイダ
- AWS CLIバージョン2.12.3以降
- kubectlコマンドラインツールのインストール


## 実際にAmazon S3 CSI driverを利用してみる

というわけで、公式ドキュメント（[Mountpoint for Amazon S3 CSI driver \- Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/s3-csi.html)）に則り、実際に動作を試してみます。環境は以下の通り。

[bash]
$ sw_vers
ProductName:            macOS
ProductVersion:         13.4.1
BuildVersion:           22F82

$ aws --version
aws-cli/2.14.5 Python/3.11.6 Darwin/22.5.0 exe/x86_64 prompt/off

$ kubectl version
Client Version: v1.28.3-eks-e71965b
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.28.3-eks-4f4795d
[/bash]

### S3バケットの作成

事前に適当な名前でS3バケットを作成しておきます。自分は、<code>test-s3-csi-driver</code>という名前でマネジメントコンソールから作成しておきました。設定などは全てデフォルトのままです。


### IAMポリシーの作成

Mountpoint for Amazon S3 CSIドライバは、ファイルシステムとやり取りするためにAmazon S3の権限が必要です。

以下のポリシー例は、Mountpointに対するIAMパーミッションの推奨に従っています。代わりに、AWSのマネージドポリシーAmazonS3FullAccessを使用できますが、パーミッションとしては必要以上に大きい点は考慮しておきましょう。詳細な、IAMポリシーについては、[mountpoint\-s3/doc/CONFIGURATION\.md at main · awslabs/mountpoint\-s3 · GitHub](https://github.com/awslabs/mountpoint-s3/blob/main/doc/CONFIGURATION.md#iam-permissions)を参照。

以下のJSONを利用したIAMポリシーを作成します。<code>DOC-EXAMPLE-BUCKET1</code>は、自分の環境のS3バケット名を指定します。ポリシー名は<code>AmazonS3CSIDriverPolicy</code>とします。

[json]
{
   "Version": "2012-10-17",
   "Statement": [
        {
            "Sid": "MountpointFullBucketAccess",
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::DOC-EXAMPLE-BUCKET1"
            ]
        },
        {
            "Sid": "MountpointFullObjectAccess",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:AbortMultipartUpload",
                "s3:DeleteObject"
            ],
            "Resource": [
                "arn:aws:s3:::DOC-EXAMPLE-BUCKET1/*"
            ]
        }
   ]
}
[/json]

### IAMロールと紐づくKubernetesサービスアカウントの作成

本来は、サービスアカウントとIAMロールは別の手順で作成する必要がありますが、<code>eksctl</code>には、便利な<code>create iamserviceaccount</code>というコマンドがあり、それを使うことで、IAMロールの作成とサービスアカウントの作成を同時に実行できます。

コマンドはこちら。<code>POLICY_ARN</code>には、前手順で作成した<code>AmazonS3CSIDriverPolicy</code>のARNを入力しています。その他パラメータを各自の環境に合わせて変更し、実行してみてください。

[bash]
POLICY_ARN=$(aws iam list-policies --query 'Policies[?PolicyName==`AmazonS3CSIDriverPolicy`].Arn' --output text)
CLUSTER_NAME=my-cluster
REGION=region-code
ROLE_NAME=AmazonEKS_S3_CSI_DriverRole
eksctl create iamserviceaccount \
    --name s3-csi-driver-sa \
    --namespace kube-system \
    --cluster $CLUSTER_NAME \
    --attach-policy-arn $POLICY_ARN \
    --approve \
    --role-name $ROLE_NAME \
    --region $REGION
[/bash]

うまくいくと、IAMロールとKubernetesのサービスアカウントが<code>kube-system</code>名前空間に<code>s3-csi-driver-sa</code>という名前で作成されています。中身を参照すると、<code>Annotations</code>に、作成したRoleのARNが含まれていることがわかります。

[bash]
$ kubectl -n kube-system describe sa s3-csi-driver-sa
Name:                s3-csi-driver-sa
Namespace:           kube-system
Labels:              app.kubernetes.io/managed-by=eksctl
Annotations:         eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/AmazonEKS_S3_CSI_DriverRole
Image pull secrets:  <none>
Mountable secrets:   <none>
Tokens:              <none>
Events:              <none>
[/bash]

### Mountpoint for Amazon S3 CSI driverのインストール

ここまででdriverに適用するIAM Roleを作成しました。この手順では、いよいよS3 CSI driverをEKSクラスターに導入していきます。

Mountpoint for Amazon S3 CSI driverは、EKSのadd-onとしてインストールできます。インストール方法はいくらかありますが、ここではマネジメントコンソールから作業していきます。その他手順は、[Managing Amazon EKS add\-ons \- Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/managing-add-ons.html#creating-an-add-on)を参考にしてください。

マネジメントコンソールでEKSクラスターを参照し、[Add-ons]タブをクリックし[Get more add-ons]をクリックすると、クラスターに導入可能なAmazon EKS add-onsが表示されます。

010

[Amazon Mountpoint for S3 CSI Drive]を選択し、[Next]ボタンクリックでアドオンの設定画面になります。ここでは、[Select IAM role]で、先程作成した[AmazonEKS_S3_CSI_DriverRole]を、[Optional configuration settings]で、[Override]を指定します。

020

[Next]をクリックすると確認画面が表示されるので、そのまま[Create]をクリックします。

Add-onsの一覧画面に戻り、Amazon Mountpoint for S3 CSI DriverのStatusがActiveになれば、正常にdriverのインストールは完了です。

030

### サンプルアプリケーションによる動作確認

ここまでで前準備は完了です。いよいよ、実際にボリュームをマウントして動作確認をしていきましょう。

GitHub上に動作確認用のサンプルアプリケーションがアップロードされているので合わせて参考にしてみてください。

[mountpoint\-s3\-csi\-driver/examples/kubernetes/static\_provisioning/README\.md at main · awslabs/mountpoint\-s3\-csi\-driver · GitHub](https://github.com/awslabs/mountpoint-s3-csi-driver/blob/main/examples/kubernetes/static_provisioning/README.md)

ファイル名<code>static_provisioning.yaml</code>で、以下のyamlを用意します。変更点は、<code>Bucket name</code>と<code>Bucket region</code>。このマニフェストファイルで、S3のPersistentVolumeへのマウントと、ファイル作成用のcentosコンテナからシェル実行で、S3にファイルを書き込みます。

- Bucket name (required): PersistentVolume -> csi -> volumeAttributes -> bucketName
- Bucket region (if bucket and cluster are in different regions): PersistentVolume -> csi -> mountOptions

簡単に、このマニフェストファイルを解説すると、このyamlで以下3つのリソースが作成されます。

- name:s3-pv
  - kind: PersistentVolume
  - 解説： ここでcsiドライバーのs3.csi.aws.comを利用して、S3バケットをs3-csi-driver-volumeという名前でマウントします。capacityの設定は必須なのでありますが、S3にサイズ上限はないので実際的には無視されるようです。また、accessModesで、ボリュームの動作種別の設定も可能
- name:s3-claim
  - kind: PersistentVolumeClaim
  - 解説： 実際のボリュームマウント処理です。s3-pvという名前で、ボリュームをマウントします。これにより、S3がコンテナ内のファイルシステムとしてマウントされ利用可能となります
- name:s3-app
  - kind: Pod
  - 解説: centosのイメージを使い、シェルコマンドでボリュームしたディレクトリの<code>/data/</code>配下にファイルを書き込みます。これにより、正常にS3にファイルが作成されていれば、S3マウントが完了していることがわかります

[yaml title="static_provisioning.yaml"]
apiVersion: v1
kind: PersistentVolume
metadata:
  name: s3-pv
spec:
  capacity:
    storage: 1200Gi # ignored, required
  accessModes:
    - ReadWriteMany # supported options: ReadWriteMany / ReadOnlyMany
  mountOptions:
    - allow-delete
    - region ap-northeast-1
  csi:
    driver: s3.csi.aws.com # required
    volumeHandle: s3-csi-driver-volume
    volumeAttributes:
      bucketName: <your_bucketname>
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: s3-claim
spec:
  accessModes:
    - ReadWriteMany # supported options: ReadWriteMany / ReadOnlyMany
  storageClassName: "" # required for static provisioning
  resources:
    requests:
      storage: 1200Gi # ignored, required
  volumeName: s3-pv
---
apiVersion: v1
kind: Pod
metadata:
  name: s3-app
spec:
  containers:
    - name: app
      image: centos
      command: ["/bin/sh"]
      args: ["-c", "echo 'Hello from the container!' >> /data/$(date -u).txt; tail -f /dev/null"]
      volumeMounts:
        - name: persistent-storage
          mountPath: /data
  volumes:
    - name: persistent-storage
      persistentVolumeClaim:
        claimName: s3-claim
[/yaml]

上記作成したマニフェストファイルをデプロイ。

[bash]
$ kubectl apply -f static_provisioning.yaml
[/bash]

S3へのファイル書き込み用Podの<code>s3-app</code>が起動しているか確認します。

[bash]
$ kubectl get pod s3-app
[/bash]

正常にS3がマウントされてファイルシステムへの書き込みができていれば、指定したバケットにファイルが作成されています。お疲れ様でした！

[bash]
$ aws s3 ls <your_bucket_name>
2023-12-05 07:31:03         26 Mon Dec  4 22:31:02 UTC 2023.txt
[/bash]

ボリュームとPodの削除は以下コマンドで実施。

[bash]
$ kubectl delete -f static_provisioning.yaml
[/bash]

以上、S3へのVolumeマウントとファイル書き込みの様子をお届けしました。

## まとめ「Kubernetesからよりネイティブな方式でS3マウントを実現」

大量、あるいは大型のファイルをKubernetesで扱う場合、NodeのファイルシステムからEFSなどをマウントしてKubernetesからボリュームを割り当て利用しているパターンも多かったのではないでしょうか？今回のアップデートでは、大量あるいは大型のファイルはS3に用意しておきアプリケーション側から都度マウントして利用することで、より柔軟にS3の拡張性と耐久性を利用してファイルシステムとして手軽にKubernetesから利用することができるようになっています。

IAMロール周辺の事前設定やドライバーのインストールがそれなりに手間ですが、このあたりをIaC管理できれば、非常に手軽にS3をKubernetesのボリュームとして利用することが可能です。Kubernetesのアプリケーションワークロードのさらなる広がりを感じるアップデートなので、皆さんも是非活用してみることをおすすめします。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。
