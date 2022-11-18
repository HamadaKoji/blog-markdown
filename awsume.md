# AWSのスイッチロールを全て解決する素晴らしいツール「AWSume」の紹介

<strong>「スイッチロールからの作業、AWSのベストプラクティスだけれど、ツールの設定がまぁまぁめんどくさいよね」</strong>

最近、会社のパソコンをM1 Macに切替たこがきっかけで、いろんなツールを再度セットアップしてました。

普段AWS触っている身としては、スイッチロールして作業する環境も必須なので改めて最新ツールを物色していたところ、弊社技術番長の[岩田](https://dev.classmethod.jp/author/iwata-tomoya/)に教えてもらった[AWSume](https://awsu.me/)というツールが圧倒的に便利だったので、前のめりに紹介します。

- 標準公式のconfigとcredentialファイルのみで動作し、<strong>別途設定ファイルは不要</strong>
- コマンドラインから、<strong>プロファイル名の自動補完に対応</strong>
- 指定したプロファイルから、<strong>マネジメントコンソールの起動も可能</strong>

と、スイッチロールに関わるほぼすべての領域を完全網羅してます。セットアップもめちゃくちゃ簡単なので、普段スイッチロールして作業している全てのAWSユーザーにおすすめしたい逸品でございます。

<pre style="line-height:120%;">
これが神ツールか…!!

　 ( ﾟдﾟ)　ｶﾞﾀｯ
　 /　　 ヾ
＿_L| /￣￣￣/＿
　 ＼/　　　/
</pre>

## この記事を読むときの前提知識

この記事は、スイッチ用IAMロールの概念と作成方法、AWS CLIを利用するときの認証情報の設定などの基礎知識があることを前提としています。このあたりがまだ不安な方は、以下の記事を参考に、学びながらIAMロールの作成や、AWS CLIの設定を実施しておきましょう。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="IAM ユーザーにアクセス許可を委任するロールの作成 - AWS Identity and Access Management" src="https://hatenablog-parts.com/embed?url=https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/id_roles_create_for-user.html" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="AWS CLI の設定 - AWS Command Line Interface" src="https://hatenablog-parts.com/embed?url=https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-chap-configure.html" width="300" height="150" frameborder="0" scrolling="no"> </iframe>


## AWS CLIでスイッチロールするために必要なファイルの用意

前段階で必要なリソースを作成したら、以下の2つのファイルを用意しておきます。

[bash title="~/.aws/config"]
[default]
aws_access_key_id = AKIAXXXXXXXXXXXXXXXXXXXXX
aws_secret_access_key = vdYYYYYYYYYYYYYYYYYYYYYYYY 
[/bash]

[bash title="~/.aws/config "]
[default]
region = ap-northeast-1
output = json

[profile <スイッチロール時に指定するプロファイル名>]
role_arn = arn:aws:iam::<スイッチロール先のアカウントID>:role/cm-hamada.koji
mfa_serial = arn:aws:iam::<スイッチロール元のアカウントID>:mfa/cm-hamada.koji
source_profile = default
[/bash]

<code>mfa_serial</code>は、スイッチロール先のロールの信頼ポリシーでMFA認証が必須な場合に設定する必要があります。断言しますが、MFAを利用しないスイッチロールは超絶事故の原因になるので、必ず設定しておくことをオススメします。

利用するファイルの用途や設定内容の詳細は、以下を参考に。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="設定ファイルと認証情報ファイルの設定 - AWS Command Line Interface" src="https://hatenablog-parts.com/embed?url=https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-configure-files.html" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

## AWSumeとは？

「AWSume」はセッショントークンの管理とIAMロール用のクレデンシャルを引受けるためのシンプル、かつ高機能なコマンドラインツールです。

公式ページはこちら。

<iframe class="hatenablogcard" style="width:100%;height:155px;max-width:680px;" title="AWSume: AWS Assume Made Awesome! | AWSume" src="https://hatenablog-parts.com/embed?url=https://awsu.me/" width="300" height="150" frameborder="0" scrolling="no"> </iframe>

できることの概要は、公式ページから引用している下記動画GIFをみてもらえれば一目瞭然。どうです？便利そうでしょ？

010.gif


## AWSumeのセットアップ

セットアップはシンプル。以下公式手順に則って進めてください。

[Installation and Quick Start \| AWSume](https://awsu.me/general/quickstart.html)

自分のMacでは、クライアントに<code>pipx</code>がなかったので、以下の手順で進めました。

[bash]
$ brew install pipx
$ pipx install awsume
[/bash]

自分の環境では、上記インストール実施後、以下のメッセージが表示されたので、メッセージにそって、<code>awsume-configure</code>を実行することでエイリアス関連の設定が書き込まれました。

[bash]
$ pipx install awsume

Warning: the awsume shell script is not being sourced, please use awsume-configure to install the alias
/Users/hamada.koji/.local/bin/awsume: line 183: return: can only `return' from a function or sourced script

$ awsume-configure

<以下の内容が、.bash_profileに書き込まれる>
#AWSume alias to source the AWSume script
alias awsume="source awsume"

#Auto-Complete function for AWSume
_awsume() {
    local cur prev opts
    COMPREPLY=()
    cur="${COMP_WORDS[COMP_CWORD]}"
    prev="${COMP_WORDS[COMP_CWORD-1]}"
    opts=$(awsume-autocomplete)
    COMPREPLY=( $(compgen -W "${opts}" -- ${cur}) )
    return 0
}
complete -F _awsume awsume
[/bash]

<code>awsume-configure</code>は、awsumeの初期セットアップ用のツールで、インストール時に自動セットアップがうまくいかなかったときのために、用意されているツールです。インストールはできたはずだが、どうにもエラーでうまく動かないときは、こちらのドキュメントを参考に再設定してみてください。

[awsume\-configure \| AWSume](https://awsu.me/utilities/awsume-configure.html)

<code>.bash_profile</code>が更新されているので、ターミナルを再起動します。

## AWSumeを利用して、スイッチロールしてみる

実際のスイッチロール方法は、ものすごく簡単。これだけです。

[bash]
awsume <profile_name>
[/bash]

例えば、以下の<code>config</code>がある状態だと、こんな感じでスイッチロールできます。

[bash title="~/.aws/config "]
[default]
region = ap-northeast-1
output = json

[profile cm-hamada]
role_arn = arn:aws:iam::BBBBBBBBBB:role/cm-hamada.koji
mfa_serial = arn:aws:iam::AAAAAAAAAA:mfa/cm-hamada.koji
source_profile = default
[/bash]

こんな感じです。MFAが必須な場合は、途中でMFA tokenを聞かれるので、入力します。

[bash]
$ awsume cm-hamada
Enter MFA token: 030152
Session token will expire at 2022-11-18 11:20:32
[cm-hamada] Role credentials will expire 2022-11-18 00:20:33
[/bash]

認証情報を確認する<code>aws sts get-caller-identity</code>で、意図した認証情報を取得できていればOK。

[bash]
aws sts get-caller-identity
{
    "UserId": "AAAAAAAAAAAAAAAAAAA:cm-hamada",
    "Account": "BBBBBBBBBB",
    "Arn": "arn:aws:sts::BBBBBBBBBB:assumed-role/cm-hamada.koji/cm-hamada"
}
[/bash]

認証情報をクリアする場合は、<code>--unset</code>コマンドを使います。

[bash]
awsume --unset
[/bash]

## AWSumeの便利機能の数々！！

これだけでも自分としては、「めっちゃ便利やん！！」と感動しきりなんですが、様々な便利機能がAWSumeには搭載されているので、いくらか紹介します。

### プロファイル名のオートコンプリート

開発〜ステージング〜本番など、複数環境でスイッチロールして作業する人も多いと思います。そんなときに便利なのが、プロファイル名のオートコンプリート。

例えば、<code>config</code>がこんな感じになっていたとして。

[bash title="~/.aws/config "]
[default]
region = ap-northeast-1
output = json

[profile cm-hamada]
<中略>

[profile cm-dev-hamada]
<中略>

[profile cm-prd-hamada]
<中略>

[/bash]

プロファイル名を途中まで入力して、Tabキーを押すとこんな感じで、入力候補を表示してくれます。

[bash]
$ awsume cm-
cm-dev-hamada  cm-hamada      cm-prd-hamada  
[/bash]

### プロファイル一覧の表示

<code>-l, --list-profile</code>オプションで、設定されているプロファイルの一覧が表示可能。

[bash]
$ awsume --list-profile
Listing...

==========================AWS Profiles==========================
PROFILE        TYPE  SOURCE   MFA?  REGION          ACCOUNT     
cm-dev-hamada  Role  default  Yes   None            BBBBBBBBBBBB
cm-hamada      Role  default  Yes   None            BBBBBBBBBBBB
cm-prd-hamada  Role  default  Yes   None            BBBBBBBBBBBB
default        User  None     No    ap-northeast-1  Unavailable 
[/bash]

### クレデンシャル引受け時のコマンド表示

<code>-s, --show-commands</code>オプションで、ロール引受時に実行されるクレデンシャル設定用のコマンドが表示されます。

[bash]
$ awsume my-admin -s
export AWS_ACCESS_KEY_ID=<SECRET>
export AWS_SECRET_ACCESS_KEY=<SECRET
export AWS_SESSION_TOKEN=<SECRET>
export AWS_SECURITY_TOKEN=<SECRET>
export AWS_REGION=<REGION>
export AWS_DEFAULT_REGION=<REGION>
export AWSUME_PROFILE=my-admin
export AWSUME_EXPIRATION=<YYYY-mm-ddTHH:MM:SS>
[/bash]


## 「個人的に最高」マネジメントコンソールの起動

別途プラグインを導入することで、任意のプロファイルの認証情報を利用したマネジメントコンソールの起動ができます。これが神と言わずして何が神か！！

プラグインのインストール方法と利用法は、以下のページを参照。

[trek10inc/awsume\-console\-plugin: This is a plugin that enables you to use your assumed role credentials to open the AWS console in your default browser\.](https://github.com/trek10inc/awsume-console-plugin)

awsumeインストール時の方法に合わせて、<code>pipx</code>または<code>pip</code>でプラグインをインストールします。

[bash]
pip install awsume-console-plugin
or
pipx inject awsume awsume-console-plugin
[/bash]

ここから先は、興奮しっぱなしですよ。

### コンソールの起動

<code>-c</code>オプションで、指定したプロファイルの認証情報を利用した状態で、デフォルトブラウザでマネジメントコンソールが起動します。最高に便利。

[bash]
awsume <profile_name> -c
[/bash]

現在利用しているプロファイルを利用するだけなら、<code>profile_name</code>は省略可能。

[bash]
awsume -c
[/bash]


### コンソールリンクの取得

<code>-cl</code>オプションで、コンソールのURLが標準出力されます。

[bash]
awsume <profile_name> -cl
[/bash]

### サービスページを指定したマネジメントコンソールの起動

<code>-cs</code>オプションとサービス名を指定することで、特定のサービスを指定した状態でマネジメントコンソールの起動が可能です。

[bash]
awsume <profile_name> -cs <service>
[/bash]

例えば、こんな感じにすれば、指定したプロファイルでCloudFormationの画面が開きます。なんとまぁ小気味よい機能ですこと！！

[bash]
awsume cm-hamada -cs cloudformation
[/bash]

その他、いろんなカスタム方法もあるようなので、あれこれ気になってみた方は、上で紹介した公式ページみて、あれこれ試してみてください！

## AWSumeはスイッチロールに関わる作業を全て完結できる素晴らしいツール

以前は似たようなものとして、<code>assume-role</code>というツールを使っていました。

[remind101/assume\-role: Easily assume AWS roles in your terminal\.](https://github.com/remind101/assume-role)

これをdirenvを併用して、ディレクトリ移動で自動的に対象プロファイルにスイッチロールする仕組みを利用していました。このブログには何度お世話になったことか…

[\[小ネタ\]ディレクトリ移動した際に自動で一時クレデンシャルを取得・設定する \| DevelopersIO](https://dev.classmethod.jp/articles/assumerole-with-direnv/)

なのですが、最近Apple Silicon Macに変えたところ、<code>assume-role</code>が動かなくなっており、そのための代替策として、以下のブログを試そうとしていたところでした。

- [AWS CLIでスイッチロールして一時クレデンシャル情報を環境変数に設定するスクリプト \| DevelopersIO](https://dev.classmethod.jp/articles/aws-cli-switch-role-script/)
- [お世話になってたディレクトリ移動だけでAWSのクレデンシャルを切り替える仕組みがM1 Macで動かなかったので替わりのスクリプトを書いた \| DevelopersIO](https://dev.classmethod.jp/articles/direnv-assumerole-on-m1mac/)

そんな悩みを社内Slackにつぶやいたところ、弊社の技術番長[岩田](https://dev.classmethod.jp/author/iwata-tomoya/)先生に教えてもらったのが、このAWSumeだったということです。

また、今まではマネジメントコンソールでの操作は、この便利ブラウザ拡張機能、[aws\-extend\-switch\-roles](https://github.com/tilfinltd/aws-extend-switch-roles)に大変お世話になっていたのですが、AWSumeでブラウザ起動も完結できてしまいました。


