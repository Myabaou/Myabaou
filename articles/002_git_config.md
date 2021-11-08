---
title: "[Git]Git config コマンド" # 記事のタイトル
emoji: "💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["git"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

よく忘れるので自分用メモ

## ユーザ設定

### local(該当リポジトリのみ)

```bash
git config --local user.name "ユーザー名"
```

```bash
git config --local user.email メールアドレス
```

### global(全体)

```bash
git config --global user.name "ユーザー名"
```

```bash
git config --global user.email メールアドレス
```
