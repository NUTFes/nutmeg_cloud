# Proxmoxディレクトリ

## 概要

Proxmoxの設定をterraformで管理している

[参考サイト](https://blog.nutmeg.cloud/blog/post-ac-20241222/)

## お願い
- Makefileを使用してコマンドを叩いてね
- applyの前にplanの結果を教えてね

## 環境構築
### terraformのinstall

macの場合

```bash
brew install tfenv
tfenv install 1.10.0
tfenv use 1.10.0
```

windows(WSL)の場合

[このサイト](https://zenn.dev/shz/articles/0c237d00267be4)のコピペなので失敗したらごめんなさい

```bash
git clone https://github.com/tfutils/tfenv.git ~/.tfenv
echo 'export PATH="$HOME/.tfenv/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
tfenv install 1.10.0
tfenv use 1.10.0
```

確認

```bash
terraform -v
```

以下のようにでるはず
```bash
Terraform v1.10.0
on darwin_arm64

Your version of Terraform is out of date! The latest version
is 1.10.1. You can update by downloading from https://www.terraform.io/downloads.html
```


### クレデンシャルファイルの準備
2つのファイルを取得する必要がある
modulesの中のlxcとvmにそれぞれprovider.tfというファイルを配置する必要がある
[settingsリポジトリ](https://github.com/NUTFes/settings/tree/main/nutmegCloud/server/proxmox/modules)の中から二つのprovider.tfを、
- server/proxmox/modules/lxc
- server/proxmox/modules/vm

の中にそれぞれ配置する。

### VPNの設定
アクセスするためには、nutmeg CloudのVPNを登録する必要がある
1人では完了できないため、できるところまでの準備を記載する

1. tailscaleをinstall

macの場合：
[AppStore](https://apps.apple.com/jp/app/tailscale/id1475387142?mt=12)からinstallする

windows(WSL)の場合：
```
curl -fsSL https://tailscale.com/install.sh | sh
```

2. tailscaleアカウントを作成

[このサイト](https://login.tailscale.com/login?next_url=/welcome)にアクセスしサインアップする。技大祭のメールアドレスを推奨。


3. 招待をもらう

インフラの誰かに連絡をし、nutmeg CloudのVPNに招待してもらう

## 作成したいとき
1. ディレクトリとファイルの準備
- コマンドの引数と説明
  - resource_type
    - vmかlxc
  - pruduct
    - プロダクトの名前を入力
```bash
# コンテナを作りたいとき
make init resource_type=lxc product=example

# VMを作りたいとき
make init resource_type=vm product=example
```
exampleにはプロダクト名や用途など、わかるように設定

2. 変数ファイルを編集
`/envs/example/terraform.tfvars` を修正する
「x」になってるところを変えて欲しい
コメントを読みながら作成してください
スペックに関する部分(cpu,cores,memory,swap,disksize)は相談しながら決めましょう

3. planを実行
下のコマンドを実行する
```bash
make plan product=example
```
planはドライランなので、本当に実行した際に起きる差分を出してくれます
この結果をPRに貼り付けてインフラの誰かに見てもらってください

4. アクセスする

作成したリソースに対してSSHする

`注意！`
```
nutmeg cloudへの場合、PVEを踏み台サーバにしてSSHする必要がある
インフラの誰かに聞いてね
```
叩くコマンド
```
ssh nutmeg@<変数ファイルに書いたIP>
```
作成したリソースに対してAnsibleを実行し、必要なモジュールを揃えよう
