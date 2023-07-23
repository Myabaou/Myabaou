---
title: "[CDKforTerraform]nodeをアップグレードしたらエラーになった" # 記事のタイトル
emoji: "💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["terraform","cdk","cdktf"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## 背景

利用していたnodeのバージョンがEOLだったため
v19からv20にあげたらcdktfがエラーになった。

## 事象内容

nodeは`nodebrew`で管理しています。

### nodeのアップグレード

- Version確認
```sh
node -v
```
v19.5.0

- v20.5.0 Install
```sh
nodebrew install-binary v20.5.0
nodebrew use v20.5.0
```

- Version確認
```sh
node -v
```
v20.5.0


- cdktf Verison確認

```sh
cdktf --version
```

  - エラー内容
  ```
  node:internal/fs/utils:351
    throw err;
    ^

Error: ENOENT: no such file or directory, scandir '/Users/username/.nodebrew/node/v20.5.0/lib/node_modules/@cdktf/cli-core/templates'
    at Object.readdirSync (node:fs:1534:3)
    at /Users/username/.nodebrew/node/v20.5.0/lib/node_modules/cdktf-cli/bundle/bin/cdktf.js:53:30496
    at /Users/username/.nodebrew/node/v20.5.0/lib/node_modules/cdktf-cli/bundle/bin/cdktf.js:1:226
    at /Users/username/.nodebrew/node/v20.5.0/lib/node_modules/cdktf-cli/bundle/bin/cdktf.js:53:30740
    at /Users/username/.nodebrew/node/v20.5.0/lib/node_modules/cdktf-cli/bundle/bin/cdktf.js:1:258
    at Object.<anonymous> (/Users/username/.nodebrew/node/v20.5.0/lib/node_modules/cdktf-cli/bundle/bin/cdktf.js:54:1932)
    at Module._compile (node:internal/modules/cjs/loader:1233:14)
    at Module._extensions..js (node:internal/modules/cjs/loader:1287:10)
    at Module.load (node:internal/modules/cjs/loader:1091:32)
    at Module._load (node:internal/modules/cjs/loader:938:12) {
  errno: -2,
  syscall: 'scandir',
  code: 'ENOENT',
  path: '/Users/username/.nodebrew/node/v20.5.0/lib/node_modules/@cdktf/cli-core/templates'
}

Node.js v20.5.0
  ```



## 対処方法

色々試行錯誤した内容は後述するとして`brew`でインストールするようにしたらうまくいった。
ただし`tfenv`をインストールしてある場合はunlinkする必要がある。
おそらくbrewでcdktfをインストールするとterraformも合わせてインストールするためと思われる。
参考: https://formulae.brew.sh/formula/cdktf

```sh
brew install cdktf
```

```sh
cdktf --version
```
0.17.1

- tfenvをunlinkする場合
tfenvが利用できなくなるので留意
```
brew unlink tfenv
```

---

## 試行錯誤記録

cdktf-cliの再インストール
cdktfの再インストール

- v20.5.0に切り替えてcdktfインストール
nodeのVersionを切り替えて実施する必要がある。
```sh
nodebrew use v20.5.0
npm install --global cdktf-cli@latest
```
エラーが発生する。


- v19.5.0に切り替えて確認
```sh
nodebrew use v19.5.0
```
→エラーは発生しない。

ただしv19.5.0でuninstallしてからinstallすると同じ事象が発生する。
```sh
npm uninstall --global cdktf-cli@latest
npm install --global cdktf-cli@latest
```


@[card](https://github.com/sass/node-sass/issues/1579)
rebuildすると解消するという記事もあったので試してみるものの事象変わらず。

実行環境共通化のため コンテナイメージ化する時には解消しておかないと。
brewが利用できないので。
