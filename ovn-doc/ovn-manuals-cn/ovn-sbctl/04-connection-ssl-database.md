# OVN 数据库与连接配置

## 远程连接命令 (Remote Connectivity Commands)

这些命令操作 `SB_Global` 表中的 `connections` 列和 `Connection` 表中的行。

*   `get-connection`
    打印配置的连接。

*   `del-connection`
    删除配置的连接。

*   `[--inactivity-probe=msecs] set-connection target...`
    设置配置的管理目标。
    *   `--inactivity-probe`: 覆盖默认的空闲连接探活时间。

## SSL/TLS 配置命令 (SSL/TLS Configuration Commands)

*   `get-ssl`
    打印 SSL/TLS 配置。

*   `del-ssl`
    删除当前的 SSL/TLS 配置。

*   `[--bootstrap] set-ssl private-key certificate ca-cert [ssl-protocol-list [ssl-cipher-list [ssl-ciphersuites]]]`
    设置 SSL/TLS 配置。

## 数据库命令 (Database Commands)

这些命令直接查询和修改 OVSDB 表的内容。

*   `list table [record]...`
*   `find table [column[:key]=value]...`
*   `get table record [column[:key]]...`
*   `set table record column[:key]=value...`
*   `add table record column [key=]value...`
*   `remove table record column value...`
*   `clear table record column...`
*   `create table column[:key]=value...`
*   `destroy table record...`
*   `wait-until table record [column[:key]=value]...`
*   `comment [arg]...`

详细语法和用法请参考 `ovn-nbctl` 文档中的数据库命令部分，因为它们共享相同的 OVSDB 操作接口。
