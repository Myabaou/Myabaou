---
title: "[CDK]カスタムリソースのLambdaのアップグレード" # 記事のタイトル
emoji: "💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["AWS","cdk"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---


## 背景

https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/lambda-runtimes.html

Lambdaのnodejs16が2024 年 6 月 12 日でEOLを迎えるため確認を進めたところ
対象LambdaがカスタムリソースとしてCDKで作成されていたため、
どのように対応するかを検討したときのメモ書きです。


## カスタムリソース対象

S3をCDKで管理しているのですがオプションでautoDeleteObjectsを利用しています。
- 参考記事
@[card](https://dev.classmethod.jp/articles/aws-cdk-s3-bucket-autodeleteobjects-feature/)

このオプションを利用するとLambdaが作成されるのですが
バージョンが16系で作成されています。


## 実施内容

自身の場合ではこれで正常に機能しましたが、実施不要な手順もあるかもしれません。
参考までに


1. ローカルのCDKのバージョンアップ

```sh
npm install -g aws-cdk
```

2. package.jsonのaws-cdkとaws-cdk-libインストールしたVersionに合わせて変更

3. node_modules ディレクトリを削除
```sh
rm -rf node_modules
```

4. node_modules再インストール
```sh
npm install
```

5. Diff確認

```sh
npx cdk diff 
```

- 差分内容
```
 ├─ [~] Runtime
 │   ├─ [-] nodejs16.x
 │   └─ [+] nodejs18.x
```
という内容に加えS3バケットのポリシー変更も発生しましたが
PutBucketPolicyのポリシー追加のみなので影響ないとみなしました。


CDKでCI/CDを実装している場合CDKのバージョン管理はどのようにやってるんだろうなー。。
毎回最新でCDKをインストールすると想定外の変更が発生しそう。

