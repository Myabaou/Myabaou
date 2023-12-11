---
title: "[Terraform] Failed to decode resource from state" # 記事のタイトル
emoji: "💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["terraform","cdktf"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---


## 発生事象
GitHubActions経由でCDKTFを実行した後にローカルで実行するとエラーが発生した。

- エラー内容

```
sample_auroracluster  Planning failed. Terraform encountered an error while generating this plan.


sample_auroracluster  ╷
                      │ Warning: Failed to decode resource from state
                      │
                      │ Error decoding "aws_cloudwatch_event_rule.rdsEventRule (rdsEventRule)" from prior state:
                      │ unsupported attribute "state"
                      ╵

⠇  Processing
[2023-12-11T17:46:50.177] [ERROR] default - ╷
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

⠇  Processing
Error: External Error: Stack failed to plan: sample_auroracluster. Please check the logs for more information.
make: *** [diff] Error 1

```

## 切り分け

- reconfigureの実行
```sh
 terraform -chdir=cdktf.out/stacks/sample_auroracluster init -reconfigure
```
→ 解消せず





## 解消方法

state rm してからimportすると解消するらしいとのことで実施

```sh
aws-vault exec goals-myabaou -- terraform -chdir=cdktf.out/stacks/sample_auroracluster state rm aws_cloudwatch_event_rule.rdsEventRule
```

```
Removed aws_cloudwatch_event_rule.rdsEventRule
Successfully removed 1 resource instance(s).
```

- import
```sh
	aws-vault exec goals-myabaou -- terraform -chdir=cdktf.out/stacks/sample_auroracluster import aws_cloudwatch_event_rule.rdsEventRule default/sample-aurora-recreate
aws_cloudwatch_event_rule.rdsEventRule: Importing from ID "default/sample-aurora-recreate"...
data.aws_db_cluster_snapshot.snapshotAurora: Reading...
data.aws_route53_zone.internalDns: Reading...
data.aws_caller_identity.accountId: Reading...
aws_cloudwatch_event_rule.rdsEventRule: Import prepared!
  Prepared aws_cloudwatch_event_rule for import
data.aws_security_group.SGData: Reading...
aws_cloudwatch_event_rule.rdsEventRule: Refreshing state... [id=default/sample-aurora-recreate]
data.aws_region.region: Reading...
data.aws_instance.ec2Instance: Reading...
data.aws_region.region: Read complete after 0s [id=ap-northeast-1]

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
```


その後diff確認し `No changes. Your infrastructure matches the configuration.`
となることを確認。

少々解せないが解消したのでよしとする。　GitHubActionsで実行した後にtfstateに破壊的な何か変更があったのかもしれない。