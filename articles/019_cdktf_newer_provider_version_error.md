---
title: "[CDKTF]Error Resource instance managed by newer provider version" # 記事のタイトル
emoji: "💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["AWS","cdk"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---


## 概要

CDK for Terraform で管理しているリソースで
cdktf diff を実行したところ以下エラーが発生した。

- エラー内容
```
[2024-01-09T17:03:56.714] [ERROR] default - ╷
│ Error: Resource instance managed by newer provider version
│
│ The current state of aws_cloudwatch_event_rule.rdsEventRule was created by
│ a newer provider version than is currently selected. Upgrade the aws
│ provider to work with this state.
sample_auroracluster  ╷
                      │ Error: Resource instance managed by newer provider version
                      │
                      │ The current state of aws_cloudwatch_event_rule.rdsEventRule (rdsEventRule) was created by
                      │ a newer provider version than is currently selected. Upgrade the aws
                      │ provider to work with this state.
                      ╵
```


## 対処方法

node_modulesで管理しているaws providerが古くなってしまったためと考えられる。
そのため再インストール

1. node_modulesディレクトリを削除

```sh
rm -rf node_modules
```

2. node_modules再インストール

```sh
npm install
```


## 確認

- Diff実行
```sh
export ENV_ID=sample && aws-vault exec my-profile -- cdktf diff sample_auroracluster
```

エラーが発生しなくなったことを確認。

