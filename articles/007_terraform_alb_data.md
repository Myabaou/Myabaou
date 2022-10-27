---
title: "[Terraform]ALB名からTargetGroupの情報を取得" # 記事のタイトル
emoji: "💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["aws","terraform","alb"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## 概要
前回の記事と類似ではあるんだけど
@[card](https://zenn.dev/myabaou/articles/006_terraform_albrules)
ALBのNameタグから設定されているターゲットグループの情報を取得したい場合。


## 前提条件
HTTPSのリスナー デフォルトルールに取得したいターゲットグループを設定してある前提
そのため別途リスナールールを設定している場合は工夫が必要。


## 取得方法
ALBの情報をdataで取得
443のリスナー情報を取得
リスナー情報から設定されてあるターゲットグループをelement関数を利用してフィルタ

```
variable "albname" {
  default = "ALB の名前"
}

data "aws_lb" "selected" {
  name = var.albname
}

data "aws_lb_listener" "selected443" {
  load_balancer_arn = data.aws_lb.selected.arn
  port              = 443
}

data "aws_lb_target_group" "selected" {
  arn = lookup(element(data.aws_lb_listener.selected443.default_action, 0), "target_group_arn", "")
}
```

もっといい方法がありそうだけど一旦これでOK

## module内で定義する場合

いつからかわからないけどmodule内でもdataリソースが利用できるようになっていた。（2022/10/27時点）
※localとかを介さないと利用できなかった気がする。（勘違いかも)

- Version情報
```
〉terraform -v
Terraform v1.3.1
on darwin_arm64
```


- 定義方法
```
data "aws_lb_listener" "selected80" {
  load_balancer_arn = data.aws_lb.alb.arn
  port              = 80
}

module "alb_listener_rules" {
  source  = "../../modules/alb_listener_rules"
  listener_arn = data.aws_lb_listener.selected80.arn
}
```

よき。