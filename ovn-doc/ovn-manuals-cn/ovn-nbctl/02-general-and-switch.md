# OVN 逻辑交换机命令 (Logical Switch Commands)

## 通用命令 (General Commands)

*   `init`
    如果数据库为空，则对其进行初始化。如果数据库已初始化，此命令无效。

*   `show [switch | router]`
    打印数据库内容的简要概述。如果提供了 `switch`，仅显示与该逻辑交换机相关的记录。如果提供了 `router`，仅显示与该逻辑路由器相关的记录。

## 逻辑交换机管理 (Logical Switch Management)

*   `ls-add`
    创建一个新的、未命名的逻辑交换机，初始状态下没有端口。由于交换机没有名称，其他命令必须通过其 UUID 引用此交换机。

*   `[--may-exist | --add-duplicate] ls-add switch`
    创建一个名为 `switch` 的新逻辑交换机，初始状态下没有端口。
    *   OVN Northbound 数据库模式不要求逻辑交换机名称唯一，但名称的主要目的是为了方便人类引用，因此重复名称没有帮助。
    *   默认情况下，如果 `switch` 是重复名称，此命令将其视为错误。
    *   使用 `--may-exist`，添加重复名称会成功，但不会创建新的逻辑交换机。
    *   使用 `--add-duplicate`，命令确实会创建一个具有重复名称的新逻辑交换机。
    *   同时指定这两个选项是错误的。如果有多个具有重复名称的逻辑交换机，请使用 UUID 而不是交换机名称来配置它们。

*   `[--if-exists] ls-del switch`
    删除 `switch`。除非指定了 `--if-exists`，否则如果 `switch` 不存在，则为错误。

*   `ls-list`
    在标准输出上列出所有现有的交换机，每行一个。
