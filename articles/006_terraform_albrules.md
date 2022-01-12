---
title: "[Terraform]ALBルールをterraformで管理" # 記事のタイトル
emoji: "💻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["aws","terraform","alb"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## 概要
ALBのルールをterraformで管理したいという要件で、
[公式モジュール](https://github.com/terraform-aws-modules/terraform-aws-alb)を使えばいい話なんだけど
そこまで汎用性は求めていなくて転送先の重み付けだけ管理したい場合のモジュールをカスタマイズした。



## 前提条件
- variables.tfを同一階層に設置済

```
variable "env" {
  default = "test"
}

variable "project" {
  default = "sample"
}

```


## ディレクトリ構成
```
├── environments
│   └── staging
│       ├── alb_rules.tf
│       ├── main.tf
│       └── variables.tf
└── modules
    └── alb_rules
        ├── main.tf
        ├── outputs.tf
        └── variables.tf
```



## 各種ファイル中身
- alb_rules.tf
```
module "alb_rules" {
  source  = "../../modules/alb_rules"
  env     = var.env
  project = var.project
  default_config = {
    alb_rule_enabled = true

    alb_rule_config = {
      test_rule = {
        priority = 101

        alb_name = "alb name"   # Change required

        /* 重み付けを利用しない場合
        action = {
          ip = {
            type             = "forward"
            target_group_arn = "target group arn" # Change required 
          }
        }
*/

        action = {
          ip = {
            type = "weighted-forward"
            target_groups = [
              {
                arn    = "target group arn" # Change required 
                weight = 0
              },
              {
                arn    = "target group arn" # Change required 
                weight = 100
              }
            ]
            stickiness = {
              enabled  = true
              duration = 3600
            }
          }
        }


        condition = {
          ip = {
            field = "source-ip"
            # Change required 
            value = [
              "XXX.XXX.XXX.XXX/32",
              "XXX.XXX.XXX.XXX/32",
              "XXX.XXX.XXX.XXX/32"
            ]
          }
        }

      }
    }
  }

  option_config = {
  }

}

output "alb_rules-info" {
  value = module.alb_rules.*
}


```

- modules/alb_rules/main.tf
```
data "aws_lb" "selected" {
  for_each = var.default_config.alb_rule_enabled == true ? var.default_config.alb_rule_config : {}
  name     = each.value.alb_name
}


data "aws_lb_listener" "selected443" {
  for_each          = var.default_config.alb_rule_enabled == true ? var.default_config.alb_rule_config : {}
  load_balancer_arn = data.aws_lb.selected[each.key].arn
  port              = 443
}


resource "aws_lb_listener_rule" "this" {

  for_each     = var.default_config.alb_rule_enabled == true ? var.default_config.alb_rule_config : {}
  listener_arn = data.aws_lb_listener.selected443[each.key].arn

  priority = each.value.priority

  # forward actions
  dynamic "action" {
    for_each = { for i in var.default_config.alb_rule_config[each.key].action : i.type => i if i.type == "forward" }

    content {
      type             = action.value["type"]
      target_group_arn = action.value["target_group_arn"]
    }
  }

  # weighted forward actions
  dynamic "action" {
    for_each = { for i in var.default_config.alb_rule_config[each.key].action : i.type => i if i.type == "weighted-forward" }

    content {
      type = "forward"
      forward {
        dynamic "target_group" {
          for_each = action.value["target_groups"]

          content {
            arn    = target_group.value["arn"]
            weight = target_group.value["weight"]
          }
        }
        dynamic "stickiness" {
          for_each = [lookup(action.value, "stickiness", {})]

          content {
            enabled  = try(stickiness.value["enabled"], false)
            duration = try(stickiness.value["duration"], 1)
          }
        }
      }
    }
  }



  condition {
    dynamic "host_header" {
      for_each = { for i in var.default_config.alb_rule_config[each.key].condition : i.field => i if i.field == "host-header" }
      content {
        values = [host_header.value.value]
      }
    }
  }

  condition {
    dynamic "path_pattern" {
      for_each = { for i in var.default_config.alb_rule_config[each.key].condition : i.field => i if i.field == "path-pattern" }
      content {
        values = path_pattern.value.value
      }
    }
  }


  condition {
    dynamic "source_ip" {
      for_each = { for i in var.default_config.alb_rule_config[each.key].condition : i.field => i if i.field == "source-ip" }
      content {
        values = source_ip.value.value
      }
    }
  }



}

```

- modules/alb_rules/variables.tf
```
# Variable
variable "project" {
}

variable "env" {
}

variable "default_config" {
}

variable "option_config" {
}


```

- modules/alb_rules/outputs.tf
特に中身は定義していないので省略


## まとめ
とりあえず、重み付けのルールだけ管理したい人向け
固定レスポンスとかはゆくゆく更新する予定