---
title: "[Terraform]ターゲットリソースを置換" # 記事のタイトル
emoji: "💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["terraform"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---


##　概要

terraform state list を実行するとリソース名が
```
module.vpc.aws_eip.nat["test"]
module.vpc.aws_eip.nat[0]
```
と表示されるのですが実際にそのリソースを確認やインポートなので指定するとき
以下のようにエスケープしないといけないので面倒です。

```
module.vpc.aws_eip.nat\[\"test\"\]
module.vpc.aws_eip.nat\[0\]
```

## 解消方法

Makefile化し中でsedする。

- 実行例
```sh
make _ENV=dev show _TARGET='module.vpc.aws_eip.nat[0]'
```

sed実行箇所
https://github.com/Myabaou/terraform_init_repo/blob/7bc482514bc961026d49c30a43f465cdb25cdb29/Makefile#L251


## 備考

@[card](https://zenn.dev/nbr41to/articles/65ade1698477ca)
をやってみたかっただけというのもある。

