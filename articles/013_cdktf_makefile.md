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


```makefile
# (ex)
# cdktfの場合: make _ENV=hotfix diff
# terraformの場合: make _ENV=hotfix _TF=true "state list"
_AWSPROFILE=aws-sample
_ENV ?= dev
_TF ?= false


ifeq ($(_ENV),prd)
_environment=production
else ifeq ($(_ENV),stg)
_environment=staging
else
_environment=$(_ENV)
endif


.PHONY: ensure-aws-auth
# AWS SSO 認証していない場合再認証
ensure-aws-auth:
	@{ \
	set +e ;\
	IDENTITY=$$(aws sts get-caller-identity --profile $(_AWSPROFILE) 2>&1) ;\
	if echo $$IDENTITY | grep -q 'The SSO session associated with this profile has expired or is otherwise invalid' ; then \
		aws sso login --profile $(_AWSPROFILE) ;\
	else \
		echo "[INFO]: AWS SSO $(_AWSPROFILE) Authentication successful!" ;\
	fi \
	}


define TERRAFORM_CMD
	aws-vault exec $(_AWSPROFILE) -- terraform -chdir="cdktf.out/stacks/${_environment}" $@
endef

define CDKTF_CMD
	export ENV_ID=$(_environment) && aws-vault exec $(_AWSPROFILE) -- cdktf $@ ${_environment}
endef

%:
	@make ensure-aws-auth 
	@if [ "$(_TF)" = "true" ]; then \
		echo "[CMD]: $(TERRAFORM_CMD)"; \
		$(TERRAFORM_CMD); \
	else \
		echo "[CMD]: $(CDKTF_CMD)"; \
		$(CDKTF_CMD); \
	fi
```

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


