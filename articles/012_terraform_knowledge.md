---
title: "[Terraform]細かいナレッジ" # 記事のタイトル
emoji: "💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["terraform"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---


## jsonencodeで"<>&"などがエスケープされる。


このことは公式ドキュメントにも記載されている。

@[card](https://developer.hashicorp.com/terraform/language/functions/jsonencode)

- Issue
@[card](https://github.com/hashicorp/terraform/issues/26110)
つまり関数replaceを使えってことらしい。


- 解消方法

こうなってしまうので。。
```
> jsonencode("test-<ID>")
"\"test-\\u003cID\\u003e\""
```

こうする。
```
>  replace(replace(jsonencode("test-<ID>"),"u003c","<"),"u003e",">")
"\"test-\\<ID\\>\""
```

