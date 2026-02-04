# OVN 数据库与连接命令

## 同步命令 (Synchronization Commands)

*   `sync`
    通常，`--wait=sb` 或 `--wait=hv` 仅等待当前 `ovn-nbctl` 调用所做的更改生效。如果命令未更改数据库，则根本不会等待。
    *   使用 `sync` 命令，`ovn-nbctl` 会等待甚至早期的数据库更改传播到 Southbound 数据库或所有 OVN 机箱（取决于 `--wait` 参数）。

## 远程连接命令 (Remote Connectivity Commands)

这些命令操作 `NB_Global` 表中的 `connections` 列和 `Connection` 表中的行。

*   `get-connection`
    打印配置的连接。

*   `del-connection`
    删除配置的连接。

*   `[--inactivity-probe=msecs] set-connection target...`
    设置配置的管理目标。
    *   `target`: 如 `ptcp:6641`, `pssl:6641`。
    *   `--inactivity-probe`: 覆盖默认的空闲连接探活时间。0 表示禁用。

## SSL/TLS 配置命令 (SSL/TLS Configuration Commands)

*   `get-ssl`
    打印 SSL/TLS 配置。

*   `del-ssl`
    删除当前的 SSL/TLS 配置。

*   `[--bootstrap] set-ssl private-key certificate ca-cert [ssl-protocol-list [ssl-cipher-list [ssl-ciphersuites]]]`
    设置 SSL/TLS 配置。

## 数据库命令 (Database Commands)

这些命令直接查询和修改 OVSDB 表的内容。它们是 OVSDB 接口的轻微抽象，比其他 `ovn-nbctl` 命令更底层。

### 语法参考

*   **Table**: 表名（如 `Logical_Switch`）。
*   **Record**: 记录标识符（UUID 或名称）。
*   **Column**: 列名。
*   **Value**: 值（整数、实数、布尔值、字符串、UUID、集合、映射）。

### 命令列表

*   `[--if-exists] [--columns=column[,column]...] list table [record]...`
    列出指定记录的数据。如果未指定记录，则列出表中的所有记录。

*   `[--columns=column[,column]...] find table [column[:key]=value]...`
    列出表中满足条件（列等于值）的所有记录。
    *   支持的操作符: `=`, `!=`, `<`, `>`, `<=`, `>=`, `{=}`, `{!=}`, `{in}`, `{not-in}`。

*   `[--if-exists] [--id=@name] get table record [column[:key]]...`
    打印给定记录中每个指定列的值。

*   `[--if-exists] set table record column[:key]=value...`
    将给定记录中每个指定列的值设置为 `value`。

*   `[--if-exists] add table record column [key=]value...`
    向记录中的列添加指定的值或键值对（用于集合或映射）。

*   `[--if-exists] remove table record column value...`
*   `[--if-exists] remove table record column key...`
*   `[--if-exists] remove table record column key=value...`
    从记录中的列移除指定的值或键值对。

*   `[--if-exists] clear table record column...`
    将记录中的每个列设置为空集或空映射。

*   `[--id=(@name|uuid)] create table column[:key]=value...`
    在表中创建一条新记录并设置每列的初始值。输出新行的 UUID。

*   `[--if-exists] destroy table record...`
    从表中删除每个指定的记录。

*   `--all destroy table`
    删除表中的所有记录。

*   `wait-until table record [column[:key]=value]...`
    等待直到表中包含一个名为 `record` 且满足指定列条件的记录。

*   `comment [arg]...`
    此命令对行为没有影响，但在数据库日志记录中包含该命令及其参数。
