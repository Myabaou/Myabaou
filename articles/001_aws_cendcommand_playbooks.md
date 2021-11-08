---
title: "SendCommandで変数を利用する方法" # 記事のタイトル
emoji: "😸" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["aws"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## 概要
`aws ssm send-command`の引数で変数を利用する場合、
少し工夫する必要があったので残しておく。


## 実行方法

### (例)AWS-ApplyAnsiblePlaybooksの実行の場合

```bash
_INSTANCE_ID="i-XXXXXXXXX"
_USER_NAME="'テスト ユーザー'"
_USER_EMAIL=hoge@test.com
```

```bash
aws ssm send-command \
--document-name "AWS-ApplyAnsiblePlaybooks" \
--document-version "1" \
--targets "[{\"Key\":\"InstanceIds\",\"Values\":[\"${_INSTANCE_ID}\"]}]" \
--parameters "{\"SourceType\":[\"S3\"],\"SourceInfo\":[\"{\\\"path\\\":\\\"https://XXXXXXXXXXXXXXX.s3.us-west-2.amazonaws.com/ec2_cloud9/ansible.zip\\\"}\"],\"InstallDependencies\":[\"True\"],\"PlaybookFile\":[\"main.yml\"],\"ExtraVariables\":[\"SSM=True user_name=${_USER_NAME} user_email=${_USER_EMAIL}\"],\"Check\":[\"False\"],\"Verbose\":[\"-vvvv\"],\"TimeoutSeconds\":[\"3600\"]}" \
--timeout-seconds 600 \
--max-concurrency "50" \
--max-errors "0" \
--region us-west-2
```

`targets`の箇所はすんなりいくけど`parameters`の箇所がやっかい

