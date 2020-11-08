# Amazon EKSがKubernetesバージョン1.18に対応しました！

<strong>「みんな大好き1.18がEKSに対応したよ！」</strong>

というわけで、EKSによるKubernetesのバージョン1.18への対応がアナウンスされました。すでにEKSのクラスターバージョン指定で、1.18が利用できるようになっています。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://aws.amazon.com/jp/about-aws/whats-new/2020/10/amazon-eks-supports-kubernetes-version-1-18/" target="_blank">Amazon EKS now supports Kubernetes version 1.18</a>
</p>

同時に既存のEKSクラスタバージョン1.14の、強制アップデートもアナウンスされています。もし手元に1.14がある方は、自動アップデートの前に対応必須ですので、気をつけましょう！

<pre style="line-height:120%;">
あの…噂の1.18が遂にきたか…!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>

なんか知ったふうやな。

## Kubernetes version 1.18で変わったこと

最新公式情報はこちら。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html" target="_blank">Amazon EKS Kubernetes versions - Amazon EKS</a>
</p>

ここでは、主にEKSクラスターでサポートされている機能について紹介します。

- Topology Managerのベータ対応
  - <a href="https://kubernetes.io/docs/tasks/administer-cluster/topology-manager/" target="_blank">Control Topology Management Policies on a node | Kubernetes</a>
- サーバーサイドApplyのベータ対応
  - <a href="https://kubernetes.io/blog/2020/04/01/kubernetes-1.18-feature-server-side-apply-beta-2/#what-is-server-side-apply" target="_blank">Kubernetes 1.18 Feature Server-side Apply Beta 2 | Kubernetes</a>
- Ingressへの設定仕様追加
  - <a href="https://kubernetes.io/blog/2020/04/02/improvements-to-the-ingress-api-in-kubernetes-1.18/" target="_blank">Improvements to the Ingress API in Kubernetes 1.18 | Kubernetes</a>
- horizontal pod autoscalingの対応
  - <a href="https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#support-for-configurable-scaling-behavior" target="_blank">Horizontal Pod Autoscaler | Kubernetes</a>
- Podのトポロジースプレッドのベータステータス対応
  - <a href="https://kubernetes.io/docs/concepts/workloads/pods/pod-topology-spread-constraints/" target="_blank">Pod Topology Spread Constraints | Kubernetes</a>

その他、Kubernetes 1.18の公式アップデート情報はこちらを参照。

<a href="https://kubernetes.io/blog/2020/03/25/kubernetes-1-18-release-announcement/" target="_blank">Kubernetes 1.18: Fit &amp; Finish | Kubernetes</a>



## 1.14のサポート終了に関する注意点

Amazon EKSのサポートは、最新の4バージョンに対して提供されます。1.18のサポートが提供開始されたことにより、サポートバージョンの一覧（2020年10月12日時点。）は以下となります。

参考：<a href="https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html" target="_blank">Amazon EKS Kubernetes versions - Amazon EKS</a>

- 1.18.8
- 1.17.9
- 1.16.13
- 1.15.11
- 1.14.9

1.18のサポート開始に伴い、<strong>1.14は2020年12月8日にサポートが終了されます。</strong>この日以降は、新規で1.14のEKSクラスターを起動できなくなり、さらに既存の1.14クラスターは強制的に1.15にアップデートされます。古いクラスター稼働させている方は要チェックですぜ！


## eksctlでバージョン1.18をデプロイしてみる

というわけで、新規でEKS上で1.18のクラスターを起動してみます。ここでは、みんな大好き<code>eksctl</code>を使います。

クライアント環境はこちら。

[bash]
$ sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.6
BuildVersion:   19G2021
[/bash]

### eksctlの1.18対応バージョンのインストール

2020年10月12日現在、eksctlは、version1.18のクラスターに対応していないためRC版をダウンロードしてインストールします。

[bash]
$ curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/0.30.0-rc.1/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
$ sudo mv /tmp/eksctl /usr/local/bin
[/bash]

バージョンを確認します。

[bash]
$ eksctl version
0.30.0-rc.1
[/bash]

### EKSクラスターの作成

パラメータは最小限にします。リージョンは東京、バージョンに<code>1.18</code>を指定。

[bash]
$ eksctl create cluster --region ap-northeast-1 --version 1.18
[ℹ]  eksctl version 0.30.0-rc.1
[ℹ]  using region ap-northeast-1
[ℹ]  setting availability zones to [ap-northeast-1a ap-northeast-1c ap-northeast-1d]
[ℹ]  subnets for ap-northeast-1a - public:192.168.0.0/19 private:192.168.96.0/19
[ℹ]  subnets for ap-northeast-1c - public:192.168.32.0/19 private:192.168.128.0/19
[ℹ]  subnets for ap-northeast-1d - public:192.168.64.0/19 private:192.168.160.0/19
[ℹ]  nodegroup "ng-6282565b" will use "ami-0e9f5606a6d10ffb1" [AmazonLinux2/1.18]
[ℹ]  using Kubernetes version 1.18
[ℹ]  creating EKS cluster "fabulous-party-1602659703" in "ap-northeast-1" region with un-managed nodes

〜〜省略〜〜


[✔]  EKS cluster "fabulous-party-1602659703" in "ap-northeast-1" region is ready
[/bash]

だいたい15分ほどかかりますが、これにてクラスターの作成完了です。

### クラスターバージョンの確認

<code>kubectl</code>で、クラスターのバージョンを確認します。

[bash]
$ kubectl version --short
Client Version: v1.17.2
Server Version: v1.18.8-eks-7c9bda
[/bash]

AWS CLIからも確認。

[bash]
$ aws eks describe-cluster --name=fabulous-party-1602659703 | jq .cluster.version
"1.18"
[/bash]

コンソール上でも、Kubernetes バージョンはこのように表示されます。

## その他関連リソースについて

既存EKSクラスターのKubernetesバージョンの更新については、以下の公式ドキュメントにまとまっています。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://docs.aws.amazon.com/eks/latest/userguide/update-cluster.html" target="_blank">Updating an Amazon EKS cluster Kubernetes version - Amazon EKS</a>
</p>

各クラスターバージョンでサポートされているVPC CNI plug-inや、DNS(CoreDNS)、KubeProxyの情報なども全て網羅されているので、実際のアップグレードの際は、必ずこの公式ドキュメントを参照し、方針をたててから実行してみましょう。

## KubernetesバージョンのアップグレードはEKS運用におけるキモ

以上、簡単に、EKSにおけるKubernetesバージョン1.18の対応について、その内容とアップグレード方法をまとめてみました。実際のアップグレード作業では、サービス中断を許容するか、クラスターによるB/Gデプロイで実施させるかなど、事前の検討必須事項は多々ありますが、バージョンアップによる新機能ももろもろ提供されているので、これを機会に手元のクラスターのアップグレード、実施いただければと思います。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

