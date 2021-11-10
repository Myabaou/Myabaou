---
title: "[Git]Git config ナレッジ" # 記事のタイトル
emoji: "👨🏻‍💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["git","aws"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

よく忘れるので自分用メモ

## 基本操作(よく利用する。)

- add
```bash
git add .
```

- commit
```bash
git commit -m "TestCommit"
```

- push
```bash
git push origin HEAD
```
`HEAD`を指定するとカレントブランチ
macOSだと大文字小文字が区別されないので`head`でもpush可能 他OSだと以下エラーになる可能性がある。
```
error: src refspec head does not match any
error: failed to push some refs to 'https://github.com/XXXXXXXXXXX.git'
```

- fetch
```bash
git fetch
```

- pull
```bash
git pull origin HEAD
```

- checkout(ブランチ切り替え)
```bash
git checkout develop
```


- checkout(新規ブランチ作成)
```bash
git checkout -b develop
```
既にある場合はエラーになる。

- branch確認
```bash
git branch -a
```


---
## CodeCommit設定
### 前提条件
codecommit操作可能な権限を所持かつ AWS profile設定済（AWS　SSOでもOK)

- SSH版
1. AWSコンソールでIAM SSH鍵を払い出す。
@[card](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-ssh-unixes.html)
2. `~/.ssh/config`に以下を定義
```
Host git-codecommit.*.amazonaws.com
User AXXXXXXXXXXXXX001
```
3. Clone
```bash
git clone ssh://AXXXXXXXXXXXXX001@git-codecommit.us-east-2.amazonaws.com/v1/repos/MyDemoRepo my-demo-repo
```
複数定義が必要がなければ`AXXXXXXXXXXXXX001@`の箇所は省略可能
@[card](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-ssh-unixes.html#setting-up-ssh-unixes-connect-console)


- HTTPS版
公式だと以下だが、想定した通りの挙動にならなかったので、個人用のまとめる。
@[card](https://docs.aws.amazon.com/ja_jp/codecommit/latest/userguide/setting-up-gc.html)
1. AWSコンソールでもCLIでもいいのでcodecommitのリポジトリを作成する。
2. git ローカル作業
```bash
REPONAME=[１で作成したリポジトリ名]
PROFILE=[AWSプロファイル名]
REGION=$(aws configure get region --profile ${PROFILE})
git init ${REPONAME}
cd ${REPONAME}
```
3. git初期設定
```bash
git config --local credential.helper "\!aws --profile ${PROFILE} codecommit credential-helper \$@"
git config --local credential.UseHttpPath true
git remote add origin https://git-codecommit.${REGION}.amazonaws.com/v1/repos/${REPONAME}
```



---

## トラブルシュート
### fatal: unable to auto-detect email address
ユーザ設定しないと`fatal: unable to auto-detect email address`が起こる。
- 参考
@[card](https://qiita.com/w-tdon/items/24348728c9256e5bf945)

- global(全体)

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

- local(該当リポジトリのみ)


```bash
git config --local user.name "ユーザー名"
```

```bash
git config --local user.email メールアドレス
```

`リポジトリ名/.git/config` に情報が追記されている。


