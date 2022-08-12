---
title: "[Terraform]SSOとaws-vaultを併用時のエラー" # 記事のタイトル
emoji: "💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["terraform"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---


## Failed to get credentials

```
aws-vault: error: exec: Failed to get credentials for awsprofilename: operation error SSO: GetRoleCredentials, https response error StatusCode: 401, RequestID: XXXXXXXXXXXXXXXXXXXXX2251, UnauthorizedException: Session token not found or invalid
```
認証情報がexpireしたことによるエラーになるが、
再度認証しようとしても認証されないことがある。


その場合一旦認証情報をクリアしてあげることで解消する可能性がある。
```sh
aws-vault clear       
```
Cleared 2 sessions.

## 参考
@[card](https://engineering.mobalab.net/2021/06/16/keep-aws-credentials-secure-using-aws-vault/)
