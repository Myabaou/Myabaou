---
title: "[Terraform]ValidationError問題" # 記事のタイトル
emoji: "💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["aws","awscli","terraform"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---


#　事象内容
ARN名など合っているはずなのにも関わらず、ValidationErrorが発生してしまう問題

## 発生条件
- 0.12系から1.Xにterraformをバージョンアップ
- AWS SSO のプロファイルでの実行時
- アクセスキーのプロファイルであれば問題なし（試していないがおそらくスイッチロールのプロファイルでも問題ないかと思われ）
- state rm してからimportしてもNG（ただstateファイルをみるとimport自体はできているという。）

## エラー内容
```
│ Error: error retrieving ALB (arn:aws:elasticloadbalancing:ap-northeast-1:[AWSアカウントID]:loadbalancer/app/alb/XXXXXXXXX): ValidationError: 'arn:aws:elasticloadbalancing:ap-northeast-1:XX' is not a valid load balancer ARN
│ 	status code: 400, request id: d4a459e0-4706-4b57-a299-4582aed7b255
│
│   with module.elb.aws_lb.web-elb[0],
│   on elb/main.tf line 1, in resource "aws_lb" "main":
│    1: resource "aws_lb" "main" {
```




## 原因

@[card](https://github.com/hashicorp/terraform-provider-aws/issues/4552)
多分これだよね。。（一応コメントしてみたけど）

AWS　SSOのプロファイルでもregion情報は記載してあるのに。。


## 回避方法

AWS SSOのプロファイルでの実行時のみ発生するので
アクセスキーを設定した場合、そっちの認証情報が優先されることを利用する。

- 一時認証払い出し
@[card](https://docs.aws.amazon.com/singlesignon/latest/userguide/howtogetcredentials.html?icmpid=docs_sso_user_portal)


- 変数設定

```sh
export AWS_ACCESS_KEY_ID="AXX"
export AWS_SECRET_ACCESS_KEY="****"
export AWS_SESSION_TOKEN="***"
```

で設定してから

```sh
terraform plan
terraform apply
```
など成功するようになった。

毎回実行時に読み込み直さないといけないのは面倒だが、現状他回避方法見つからず。。