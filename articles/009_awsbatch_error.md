---
title: "[AWS Batch]Fargate起動タイプでエラー" # 記事のタイトル
emoji: "💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["aws"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## 事象内容
- Cloud9上のコンテナでPHPのジョブを実行すると正常終了するが、AWS Batch（Fargate）で実行すると異常終了する。

### エラー内容

```
fwrite(): Unable to create temporary file, Check permissions in temporary files directory.
```


## 原因

ulimit設定の差異
(ジョブ実行でなぜそんなにopenfileが必要なのかはジョブ特有のためここでは言及しない。)

当初はディスクが足りないのが原因かと思いEFSも検討したが、
現状Fargateでは20GBまではディスクに書き込めるというのと
10GBあたりで異常終了するため、違和感を覚えていた。
しかも増やせる。

@[card](https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/fargate-task-storage.html)

```
デフォルトでは Amazon ECS タスクは、プラットフォームバージョン 1.4.0 以降を使用している Fargate でホストされている Amazon ECS は最低 20 GiB のエフェメラルストレージを受け取ります。エフェメラルストレージの総量はタスク定義に ephemeralStorage パラメーターを指定して、最大 200 GiB まで増やすことができます。
```


## 解決方法

起動タイプ Fargateを選択している場合
ulmit値は変更できない（ECSであれば可）※2022/03/07時点
みたいなのでEC2を起動タイプにして解消

- Fargate
```json
"Ulimits": [
    {
        "Name": "nofile",
        "Hard": 4096,
        "Soft": 1024
    }
],
```

- EC2(デフォルト)
```json
"Ulimits": [
    {
        "Name": "nofile",
        "Hard": 65536,
        "Soft": 32768
    }
],
```
ジョブ定義の設定で値を指定することも可能




