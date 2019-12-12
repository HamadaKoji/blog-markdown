こっそりとしたECS関連のアップデートですが、re:Inventにおいて、ECSにおいて、Active Directoryによる認証にWindows Accounts gMSAが利用可能となりました。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://aws.amazon.com/jp/about-aws/whats-new/2019/12/amazon-elastic-container-service-now-supports-active-directory-authentication-using-windows-group-managed-service-accounts/" target="_blank">Amazon ECS now supports Active Directory Authentication using Windows Accounts gMSA</a>
</p>

## Windows gMSAが使えることでなにができるのか？

ECSにおけるWindows gMSAのECSサポートにより、Active Directory（AD）を使用して、ネットワークリソースとして、Windowsコンテナを認証および承認できるようになります。

ユーザーアカウントIDの構成自体をコンテナイメージから分離したまま、複数のサービスにおけるActive Directory Security Contextの適用が可能で、ECSで、.NETアプリケーションを利用したい場合、パスワードを使わずに、Active Directry統合認証によりSQL Serverに接続できるなど、様々なメリットがあります。

ECSタスク定義のdocker Security Optionsフィー風土に資格情報を定義することで、gMSAを利用することができます。

## ECS WindowsコンテナによるgMSAによる認証を試してみたい

利用方法の概要は、公式ブログのこちらをご参考。ECS Windowsコンテナから実際にgMSAを利用してActive Directoryに参加するところまでが紹介されています。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://aws.amazon.com/jp/blogs/containers/how-to-run-ecs-windows-task-with-group-managed-service-account-gmsa/" target="_blank">How to Run ECS Windows Task with group Managed Service Account (gMSA) | Containers</a></p>

ECS WindowsコンテナからのgMSAを利用したAD認証について、GitHubに挙げられたサンプルアプリケーション（<a href="https://github.com/aws-samples/amazon-ecs-gmsa/blob/master/README.md#1-infrastructure-setup" target="_blank">amazon-ecs-gmsa/README.md at master · aws-samples/amazon-ecs-gmsa</a>）を元に、実装していく手順がステップバイステップで紹介されています。

実際に手を動かして試してみたい方は、上記ドキュメントをご参照ください。

## さらに詳しくgMSA対応について知りたい

公式ドキュメントはこちら。

<p style="padding: 12px;border-color: #E97F50;border-width: 1px;border-style: solid;border-radius: 5px;background-color: rgba(233, 127, 80, 0.07);">
<a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/windows-gmsa.html" target="_blank">Using gMSAs for Windows Containers - Amazon Elastic Container Service</a>
</p>

仕様の詳細は上記で確認いただけますが、利用上の考慮事項と実施にあたっての前提条件を抜粋します。

### 考慮事項

コンテナインスタンスに、Amazon ECS-Optimized Windows 2016 AMIを利用する場合、コンテナのホスト名は、認証用呉電車リファイルに定義されたgMSAアカウント名と同一である必要があります。コンテナのホスト名の確認には、コンテナ定義中の当該パラメータをご確認ください（参考：<a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html#container_definition_network" target="_blank">Task Definition Parameters-Network Settings</a>）。


### 前提条件

- ECS Windowsコンテナで参加可能なActive Directoryは、以下をサポート
  - AWS Direcotry Service（参考:<a href="https://docs.aws.amazon.com/directoryservice/latest/admin-guide/ms_ad_getting_started.html" target="_blank">Getting Started with AWS Managed Microsoft AD</a>）
  - Windowsコンテナが参加可能なオンプレミスのドメイン（参考：<a href="https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/aws-direct-connect-network-to-amazon.html" target="_blank">AWS Direct Connect - Amazon Virtual Private Cloud Connectivity Options</a>）
- Active Directory内の既存のgMSAアカウント（参考：<a href="https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/manage-serviceaccounts" target="_blank">Create gMSAs for Windows containers</a>）
- ECSタスクをホストするコンテナインスタンスは、ActiveDirectoryにドメイン参加し、gMSAアカウントにアクセスできるActive Directoryセキュリティグループのメンバーである必要があります


## ECSにおけるWindowsコンテナ利用拡充の要となるアップデート

もともと、ECSのWindowsコンテナにおいて、ActiveDirectoryへの参加自体は可能でした。それが、Windows Server 2012から新機能として導入されたgMSAにも拡充対応されたことで、より柔軟にWindows コンテナをグループ単位で管理、運用ができるようになることが期待できます。

ECSにおけるWindowsコンテナも機能拡充が進んでいるので、今までコンテナでWindowsを利用する機会がなかった人も、これをきっかけに利用を検討してみてはいかがでしょうか。

それでは、今日はこのへんで。濱田（<a href="https://twitter.com/hamako9999" target="_blank">@hamako9999</a>）でした。

## gMSAについての参考記事

グループ管理サービスアカウント（Group Managed Service Accouts(gMSA)）についてや、それをAWS Managed Microsoft ADで作成する方法などについては、弊社<a href="https://dev.classmethod.jp/author/shibata-takuya/" target="_blank">Takuya Shibata</a>のこちらの、記事をご参照ください。

https://dev.classmethod.jp/cloud/aws/how-to-create-gmsa-on-microsoft-ad/

