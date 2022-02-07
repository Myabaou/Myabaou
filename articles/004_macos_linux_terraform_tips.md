---
title: "[Terraform]MacOSとLinuxで併用する場合のTIPS" # 記事のタイトル
emoji: "💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["aws","awscli","terraform"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## 概要
Terraformコードをクライアントに依存せず、
MacOSとLinuxで運用する場合のTIPS

- 2022/02/07追記
```
M1 macでplan/apply時に失敗する
```
@[card](https://github.com/hashicorp/terraform-provider-aws/issues/22582)

---

## terraform init Error

```
Successfully configured the backend "s3"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...
- Reusing previous version of hashicorp/aws from the dependency lock file
- Installing hashicorp/aws v3.65.0...
╷
│ Error: Failed to install provider
│
│ Error while installing hashicorp/aws v3.65.0: the current package for registry.terraform.io/hashicorp/aws 3.65.0 doesn't match any of the checksums previously recorded in the
│ dependency lock file
```

- 解決方法
  - 参考URL
@[card](https://cha-shu00.hatenablog.com/entry/2021/05/23/134816)

既に作成済みだった場合は`.terraform.lock.hcl `を削除してから

```bash
terraform providers lock \
  -platform=darwin_amd64 \
  -platform=darwin_arm64 \
  -platform=linux_amd64 \
  -platform=linux_arm64
```
を実行すればOK(Linux Arm/Intel Mac M1/Intel)



---
## M1 Macでplan/applyが失敗する。

必ずではないものの、結構な頻度でplanやapplyが失敗するようになっていて
実行時「成功してくれ」と祈るようになっていた。
Twitterなどで検索かけてみたら解決方法を呟いてくれている人がいて感謝
@[card](https://twitter.com/kkoudev/status/1485507136449056768)

- 一度アンインストール
```sh
〉tfenv uninstall 1.1.5              
Uninstall Terraform v1.1.5
Terraform v1.1.5 is successfully uninstalled
```

- Armで改めてインストール
```sh
〉TFENV_ARCH=arm64 tfenv install 1.1.5
Installing Terraform v1.1.5
Downloading release tarball from https://releases.hashicorp.com/terraform/1.1.5/terraform_1.1.5_darwin_arm64.zip
############################################################################################################################## 100.0%
Downloading SHA hash file from https://releases.hashicorp.com/terraform/1.1.5/terraform_1.1.5_SHA256SUMS
Unable to verify OpenPGP signature unless logged into keybase and following hashicorp
Archive:  tfenv_download.eCOUA1/terraform_1.1.5_darwin_arm64.zip
  inflating: /opt/homebrew/Cellar/tfenv/2.2.2/versions/1.1.5/terraform  
Installation of terraform v1.1.5 successful. To make this your default version, run 'tfenv use 1.1.5'
```

- 確認
```sh
〉terraform -v
Terraform v1.1.5
on darwin_arm64
```
`darwin_arm64` となっていればOK `darwin_amd64` と間違いやすいので注意


すでに作成してある場合は
terraform init
や
lockファイルの再作成が必要※前述参照


---

## 実行するEC2環境にIAMロールを設定してあった場合

- エラー内容
```
│ Error: error reading IAM Group (Test_Group): AccessDenied: User: arn:aws:sts::XXXXXXX:assumed-role/cloud9role/i-XXXXXXX is not authorized to perform: iam:GetGroup on resource: group Test_Group
│       status code: 403, request id:
```

~~providerブロック内にprofileを定義して実行するようにしているが、~~
~~おそらくIAMロールのプロファイルを優先的に利用されるため、この権限がない状態になってしまう。~~

~~IAMロール　> provider 内のprofile~~
これは嘘でした。ちゃんとprofile設定がされてないことが原因

- 解決方法
設定をちゃんと見直す。

IAMロールをデタッチすれば成功すると思うが、それだと対象サーバにSSMログインできなくなってしまうため除外。

自戒を込めて残す。
@[card](https://zenn.dev/myabaou/scraps/0d47a24d000107)
