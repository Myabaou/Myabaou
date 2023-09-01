---
title: "[GitHubActions]何もしてないのにCDKTFのplanがこけた" # 記事のタイトル
emoji: "💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["terraform","GitHub","cdktf"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---


## 背景
何もしていないかというと嘘になるのですが、GitHubActionsの設定は何も設定変更していないのに
GitHubActionsで実行しているCDKTFのジョブがこけた時の対処方法メモ書きです。


## 実行しているGitHubActions

- PullRequest作成時にDryRun　→ 差分内容をコメントに追記
- mergeされたらApply
DryRunの内容を２つ出力するようにしているため若干冗長感は否めないのですが、  
`terraform`での実行確認もしておきたいのでそうしています。

```yml
name: "Develop CDK for Terraform"
on:
   pull_request:
    branches:
     - develop

    types: [opened, synchronize, reopened, closed]
    paths:
      - "cdk_for_terraform/base/environments/develop.ts"
   workflow_dispatch:
#-------------------------#
# 環境変数
#-------------------------#
env:
    SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK_URL }}
    SLACK_USERNAME: GitHubActions
    SLACK_CHANNEL: infra_notifications
    SLACK_ICON: https://octodex.github.com/images/Robotocat.png
 
permissions:
    id-token: write
    contents: read
    pull-requests: write
  

jobs:
  terraform:
    name: "Terraform CDK Diff"
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Configure AWS credentials from account
        uses: aws-actions/configure-aws-credentials@v1
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ap-northeast-1

      - uses: actions/setup-node@v1
        with:
          node-version: "20"

      - name: Install dependencies
        run: yarn install

      - name: Generate module and provider bindings
        run: | 
         cd cdk_for_terraform/base
         npx cdktf-cli@0.18.0 get
         npm install @cdktf/provider-aws
         npm install --save-dev @types/node


      - name: setup tfcmt
        env:
          TFCMT_VERSION: v3.4.1
        run: |
          wget "https://github.com/suzuki-shunsuke/tfcmt/releases/download/${TFCMT_VERSION}/tfcmt_linux_amd64.tar.gz" -O /tmp/tfcmt.tar.gz
          tar xzf /tmp/tfcmt.tar.gz -C /tmp
          mv /tmp/tfcmt /usr/local/bin
          tfcmt --version



      - name: Run Terraform CDK DryRun
        uses: hashicorp/terraform-cdk-action@v0.1.0
        with:
          terraformVersion: 1.5.5
          cdktfVersion: 0.18.0
          stackName: develop
          mode: plan-only
          githubToken: ${{ secrets.GITHUB_TOKEN }}
          workingDirectory: cdk_for_terraform/base
        env:
          ENV_ID: develop



      # Generates an execution plan for Terraform
      - name: Terraform Plan
        run: |
          cd cdk_for_terraform/base
          tfcmt plan -patch -- terraform -chdir=cdktf.out/stacks/hotfix plan -no-color -input=false
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          PR_HEAD_SHA: ${{ github.event.pull_request.head.sha }}
          PR_NUMBER: ${{ github.event.number }}


      # developにマージされた時に実行される。
      - name: Run Terraform CDK Apply
        if: github.event.pull_request.merged == true
        uses: hashicorp/terraform-cdk-action@v0.1.0
        with:
          terraformVersion: 1.5.5
          cdktfVersion: 0.18.0
          stackName: develop
          mode: auto-approve-apply
          githubToken: ${{ secrets.GITHUB_TOKEN }}
          workingDirectory: cdk_for_terraform/base
        env:
          ENV_ID: develop
```


## 発生内容

- エラー出力
```
npm ERR! code ERESOLVE
npm ERR! ERESOLVE unable to resolve dependency tree
npm ERR! 
npm ERR! While resolving: base@1.0.0
npm ERR! Found: cdktf@0.18.0
npm ERR! node_modules/cdktf
npm ERR!   cdktf@"^0.18.0" from the root project
npm ERR! 
npm ERR! Could not resolve dependency:
npm ERR! peer cdktf@"^0.17.0" from @cdktf/provider-aws@15.0.0
npm ERR! node_modules/@cdktf/provider-aws
npm ERR!   @cdktf/provider-aws@"^15.0.0" from the root project
npm ERR! 
npm ERR! Fix the upstream dependency conflict, or retry
npm ERR! this command with --force or --legacy-peer-deps
npm ERR! to accept an incorrect (and potentially broken) dependency resolution.
npm ERR! 
npm ERR! 
npm ERR! For a full report see:
```
0.18 で作成されたので15.0.0では実行できないよ　15.0.0で実行するなら0.17で実行してね。  
みたいなエラー

さすがにダウングレードはしたくないので  
`provider-aws`を安易にterraformのバージョンと合わせて
`package.json`内の`dependencies`設定 15.0.0から15.5.0に変更。

```
  "dependencies": {
    "@cdktf/cli-core": "^0.18.0",
    "@cdktf/provider-aws": "^15.5.0",
    "cdktf": "^0.18.0",
    "constructs": "^10.2.40"
  },

```




### どうなったか。

エラー内容は変わったものの未だにこける。

- エラー出力
```sh
npm WARN exec The following package was not found and will be installed: cdktf-cli@0.18.0
npm WARN deprecated @npmcli/ci-detect@1.4.0: this package has been deprecated, use `ci-info` instead
Generated typescript constructs in the output directory: .gen
npm ERR! code ETARGET
npm ERR! notarget No matching version found for @cdktf/provider-aws@^15.5.0.
npm ERR! notarget In most cases you or one of your dependencies are requesting
npm ERR! notarget a package version that doesn't exist.
```

## 対処法

provider-awsのVersionを0.17以上にする必要がありました。

以下設定で正常に稼働するようになりました。
`package.json`の内容
```json
  "dependencies": {
    "@cdktf/cli-core": "^0.18.0",
    "@cdktf/provider-aws": "^17.0.1",
    "cdktf": "^0.18.0",
    "constructs": "^10.2.40"
  },

```


## 結論

cdktfはbrewでインストールしているのですが、  
awsのproviderのバージョンを意識するのを失念してました。  
@[card](https://formulae.brew.sh/formula/cdktf)


cdktf-provider-awsのVersionも意識して運用しましょうということで。

@[card](https://github.com/cdktf/cdktf-provider-aws)