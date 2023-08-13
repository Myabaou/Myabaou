---
title: "[GitHub]ラベル付与を自動化したい。" # 記事のタイトル
emoji: "🐱" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["github"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## 背景
GitHubを利用しておりフローで
develop branchからmain branchへのプルリクエスト作成時に
ラベルを付与する運用にしている。

[Releases機能](https://docs.github.com/ja/repositories/releasing-projects-on-github/managing-releases-in-a-repository)やタグ機能を
使った運用にしたほうがいい気がするんだけど、
まだGitHubの運用が定まってしない関係で気軽に始められそうなラベル運用にした。

### 対象ラベル
 - Release: 本番環境へのインフラ変更反映がある場合
 - Pre-Release: 検証環境へのインフラ変更反映がある場合


## 解消したい課題
feature branchからdevelop branchへの作成時にも
ラベルを付与しており（ここでレビューがはいる）、Approve後
**手作業**でdevelop branchからmain branchへのプルリクエスト（以下PR）を作成している。
ここの手作業の部分を解消したい。


## GitHubActionsで実装

### 処理概要
- すでにdevelop -> mainへのPRが存在するかチェックし存在しなければPRを作成する。
- 存在した場合は該当PRのNumberを取得、作成した場合は作成したPRのNumberを取得
- feature -> developのラベル情報を取得
- 取得したPRにラベル情報を追加する。

- ymlの中身
```yml
name: Create PR from develop to main

on:
  pull_request:
    branches:
      - develop
    types:
      - closed

jobs:
  create_pr:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v2



    - name: Check if PR exists
      id: check-pr
      run: |
        PR_NUMBER=$(gh pr list --base main --head develop --json number --jq '.[0].number' || echo "")
        echo "::set-output name=pr_number::$PR_NUMBER"

    - name: Create PR if not exists
      if: steps.check-pr.outputs.pr_number == ''
      uses: repo-sync/pull-request@v2
      with:
        source_branch: "develop"
        destination_branch: "main"
        github_token: ${{ secrets.GITHUB_TOKEN }}
        pr_title: "Merge develop into main"
        pr_body: "Automated PR created by GitHub Actions."


    - name: Get labels from merged PR
      id: get-labels
      run: |
        LABELS=$(gh pr view ${{ github.event.pull_request.number }} --json labels --jq '.labels[]?.name' | tr '\n' ',')
        echo "::set-output name=labels::$LABELS"
      env:
        GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}


    - name: Add labels to new PR
      run: |
        if [ -z "${{ steps.check-pr.outputs.pr_number }}" ]; then
          PR_NUMBER=$(gh pr list --base main --head develop --json number --jq '.[0].number')
        else
          PR_NUMBER=${{ steps.check-pr.outputs.pr_number }}
        fi
        LABELS=$(echo "${{ steps.get-labels.outputs.labels }}" | sed 's/,$//')  # Remove trailing comma
        LABELS_ARRAY=($(echo "$LABELS" | tr ',' '\n'))
        LABELS_JSON="[$(printf '"%s",' "${LABELS_ARRAY[@]}" | sed 's/,$//')]"
        curl -s -X POST \
        -H "Authorization: token $GH_TOKEN" \
        -H "Accept: application/vnd.github.v3+json" \
        -d "{\"labels\":$LABELS_JSON}" \
        "https://api.github.com/repos/${{ github.repository }}/issues/$PR_NUMBER/labels"
      env:
        GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## 所感

作ってみたものの利便性が劇的にあがるかどうかイメージなしw
なんか労力に見合わないというのとこんな処理で
GitHubActionsの貴重な無料枠時間を消費するのものなーという印象。

他運用を検討したほうがよさそうではあるが、
少し試してみつつ楽になりそうであれば正式運用にしようかなと。

ただ、github.eventの情報を取り回しする方法について
知識は上がった気がするのでよしとする。
