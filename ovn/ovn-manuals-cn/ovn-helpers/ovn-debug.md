# ovn-debug (Developer Debug Tool)

## 简介

`ovn-debug` 是一个主要面向开发者的辅助工具，用于查询 OVN 内部的映射关系。

## 命令

- `lflow-stage-to-ltable <stage-name>`
  - 将逻辑流阶段名称（源码中的定义，如 `ls_in_pre_acl`）转换为对应的数字表 ID。
  - **示例**: `ovn-debug lflow-stage-to-ltable ls_in_pre_acl` -> `3`

- `lflow-stage-list`
  - 列出所有可用的逻辑流阶段及其描述。
  - 这对于理解 `ovn-trace` 输出中的阶段名称非常有帮助。
