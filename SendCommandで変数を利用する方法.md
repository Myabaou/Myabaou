# SendCommandで変数を利用する方法


## 概要
`aws ssm send-command`の引数で変数を利用する場合、
少し工夫する必要があったので残しておく。


## 実行方法

### (例)AWS-ApplyAnsiblePlaybooksの実行の場合

```bash
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

