---
title: "[CDKForTerraform]Makefile化" # 記事のタイトル
emoji: "💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["terraform","cdk","cdktf"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## モチベーション
現時点(2023/06/03)ではcdktfでのimportなど一部サポートされていなかったりするので、
実行切り替えをスムーズにするためMakefile化しました。
GitHubで管理するまでもないので記事。


## 前提条件
- aws-vaultインストール済
- [infracost](https://www.infracost.io/)インストール済
- cdktf.jsonに以下を追記
```json
  "envconfig": {
    "dev": {
    },
    "staging": {
    },
    "production": {
    }

```

- main.tsで以下のように指定
```ts
const environments = Object.keys(config.envconfig);

for (const env of environments) {
  new MyStack(app, env);
}
```

- tfstateファイルの管理でS3を利用している場合
ENV_ID毎にtfstateファイルが生成されるようにする。
```
      key: `cdktf/sample${process.env.ENV_ID}.tfstate`,
```

## Makefile内容
- 2023/06/15 infracostを組み込んでapply時にファイル出力 plan時に差分確認できるようにしました。
- 2023/06/17 zenn記事に直接記載していたのをGitHubに移動

@[card](https://github.com/Myabaou/cdktfinit/tree/main)


## 実行方法

- cdktfの場合 (DryRun)
```sh
make _ENV=dev diff
```

- terraformで実行したい場合(DryRun)
```sh
make _ENV=dev _TF=true "plan"
```

## まとめ

他メリットとしてtfstateファイルも環境毎に切り替えられるので
利便性が向上しました。
Workspacesライクに使えるのも○


