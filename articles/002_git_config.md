---
title: "[Git]Git config コマンド" # 記事のタイトル
emoji: "👨🏻‍💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["git"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

よく忘れるので自分用メモ

## ユーザ設定

設定しないと`fatal: unable to auto-detect email address`が起こる。
- 参考
https://qiita.com/w-tdon/items/24348728c9256e5bf945

### global(全体)

```bash
git config --global user.name "ユーザー名"
```

```bash
git config --global user.email メールアドレス
```

`~/.gitconfig` に情報が追記されている。

```
利用している環境でユーザが統一できていれば問題ないが、
特定のリポジトリは認証情報を変えたいという場合はlocalで設定する必要がある。
```

### local(該当リポジトリのみ)


```bash
git config --local user.name "ユーザー名"
```

```bash
git config --local user.email メールアドレス
```

`リポジトリ名/.git/config` に情報が追記されている。


