---
title: "[Codebuild]AWS CLIで実行" # 記事のタイトル
emoji: "💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["aws","AWSCLI"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## 前提条件
- aws cli v2 インストール済

## CodeBuild

基本ここをみれば問題なし
@[card](https://docs.aws.amazon.com/ja_jp/codebuild/latest/userguide/cmd-ref.html)
@[card](https://docs.aws.amazon.com/ja_jp/codebuild/latest/userguide/run-build-cli.html)
@[card](https://docs.aws.amazon.com/cli/latest/reference/codebuild/start-build.html)

### 変数上書き実行
- input.json
```
{
  "projectName": "test2",
  "environmentVariablesOverride": [
  {
    "name": "_INSTANCE_ID",
    "value": "i-XXXXXXXXX",
    "type": "PLAINTEXT"
  },
  {
    "name": "_USER_NAME",
    "value": "'テスト ユーザ'",
    "type": "PLAINTEXT"
  },
    {
    "name": "_USER_EMAIL",
    "value": "test@test.co.jp",
    "type": "PLAINTEXT"
  }
  ]
}

```


```bash
aws codebuild start-build --cli-input-json input.json
```


### うまくいかない方法

jsonじゃない指定方法でもいけると思ったけど
_USER_NAMEの箇所でシングルクォーテーションの箇所が認識されず、エラーになってしまいます。
成功するやり方知っている方がいれば教えてください。。

```bash
aws codebuild start-build \
  --project-name "test2" \
  --environment-variables-override  name=_INSTANCE_ID,value=i-XXXXXXXXX,type=PLAINTEXT name=_USER_NAME,value="'テスト ユーザ'",type=PLAINTEXT name=_USER_EMAIL,value='test@test.co.jp',type=PLAINTEXT
```
