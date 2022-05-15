# （仮）GitLab RunnerのジョブにAWSのIAMロールを適用する方法


## そもそも何故、IAMロールをジョブに適用させたいのか？

割り切った運用するならば、以下の方法もありです（以前はこうだった）。

1. GitLab Runnerを動作させるEKSのノード（EC2）にIAMロールを適用
2. ジョブで利用するPolicyを上記IAMルールに付与
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

## IAMロールをKubernetesのServiceAccountに適用する方法の基本

ジョブへのIAMロール適用方法を検討する前に、そもそもPodが動作するときのServiceAccountに対してIAMロールを適用する必要があるのですが、その方法についてはAWSのこの公式ブログに全て記載されていました。

[詳解: IAM Roles for Service Accounts \| Amazon Web Services ブログ](https://aws.amazon.com/jp/blogs/news/diving-into-iam-roles-for-service-accounts/)

このブログは、チュートリアル形式になっているので、実際に手元で動作させながら理解を深めることができます。IAMロールが
