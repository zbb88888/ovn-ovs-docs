# 全局配置 (Global Configuration)

## SB_Global 表

包含 Southbound DB 的全局状态。

### 核心列

- **nb_cfg**: 整数。已传播到 SB DB 的北向数据库配置序列号。ovn-northd 会更新此值。
- **options**: 全局选项。
- **external_ids**: 外部系统关联 ID。
- **connections**: 指向 `Connection` 表的引用。
- **ssl**: 指向 `SSL` 表的引用。

## RBAC (Role-Based Access Control)

OVN 使用 RBAC 来限制 `ovn-controller` 对 SB DB 的访问权限，增强安全性。通常，`ovn-controller` 仅被允许读取大部分表，且只能更新特定的列（如 `Chassis` 表）。

### RBAC_Role 表

定义角色名称和对应的权限集。

- **name**: 角色名称（如 `"ovn-controller"`）。
- **permissions**: 权限字典，映射表名到 `RBAC_Permission` 记录。

### RBAC_Permission 表

定义对特定表的具体操作权限。

- **table**: 目标表名。
- **authorization**: 授权规则（如检查 ID 是否匹配）。
- **insert_delete**: 是否允许插入/删除记录。
- **update**: 允许更新的列名列表。
