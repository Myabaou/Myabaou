---
title: "[SessionManager]KMSを有効にしてある場合エラーになる。" # 記事のタイトル
emoji: "💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["aws","awscli"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## 概要
`start-session`実行時以下エラーがでてログインできない。

```
----------ERROR-------
Encountered error while initiating handshake. KMSEncryption failed on client with status 2 error: Failed to process action KMSEncryption: Error calling KMS GenerateDataKey API: NoCredentialProviders: no valid providers in chain. Deprecated.
	For verbose messaging see aws.Config.CredentialsChainVerboseErrors
```


## 発生条件
以下条件にて発生する。マネージメントコンソールからのログインは問題なし。
- AWS SSOのプロファイルを有効にしている。
- SessionManagerの設定でKMSを有効にしている。
- AWS CLIでの実行


## 解決策
AWS公式回答によると 2022/02/02時点では解決策なし。
（アップデート待ち


## まとめ
KMSでの暗号化が必須ではない場合は無効にしましょう。
セキュリティ要件で暗号化必須の場合は不便ではありますが、
マネージメントコンソールからのログインを利用しましょう。

