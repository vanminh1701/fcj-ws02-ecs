---
title: "Clean up"
date: "`r Sys.Date()`"
weight: 4
pre: "<b>4. </b>"
---

The `terraform destroy` command is a convenient way to destroy all remote objects managed by a particular Terraform configuration.

```
$ terraform destroy
   ...
   ...
  # module.vpc_endpoint_sg.aws_security_group_rule.ingress_rules[0] will be destroyed
  - resource "aws_security_group_rule" "ingress_rules" {
      - cidr_blocks            = [
          - "172.31.0.0/16",
        ] -> null
      - description            = "All protocols" -> null
      - from_port              = 0 -> null
      - id                     = "sgrule-1134506769" -> null
      - ipv6_cidr_blocks       = [] -> null
      - prefix_list_ids        = [] -> null
      - protocol               = "-1" -> null
      - security_group_id      = "sg-040a655a279c290be" -> null
      - security_group_rule_id = "sgr-02ab7a074d8880b6d" -> null
      - self                   = false -> null
      - to_port                = 0 -> null
      - type                   = "ingress" -> null
    }

Plan: 0 to add, 0 to change, 46 to destroy.

Changes to Outputs:
  - private_keypair = (sensitive value) -> null

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value:
```

Type `yes` and wait for the command finish.