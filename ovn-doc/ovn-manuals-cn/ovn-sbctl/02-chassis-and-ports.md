# OVN Southbound 机箱与端口命令

## Southbound 数据库命令 (OVN_Southbound Commands)

这些命令作用于整个 OVN_Southbound 数据库。

*   `init`
    如果数据库为空，则对其进行初始化。

*   `[--filter=filter-rule[,filter-rule]...] show`
    打印数据库内容的简要概述。
    *   `--filter`: 根据规则过滤输出。规则格式为 `table-name(filter|filter...)`。默认过滤 `Chassis` 表。
    *   仅显示包含至少一个指定过滤子串的行。

## 机箱命令 (Chassis Commands)

这些命令操作 OVN_Southbound 机箱 (Chassis)。

*   `[--may-exist] chassis-add chassis encap-type encap-ip`
    创建一个名为 `chassis` 的新机箱。
    *   `encap-type`: 逗号分隔的隧道类型列表 (如 `geneve,vxlan`)。
    *   `encap-ip`: 隧道目的 IP。
    *   该机箱将为每个指定的隧道类型创建一个封装条目。

*   `[--if-exists] chassis-del chassis`
    删除 `chassis` 及其封装和网关端口。

## 端口绑定命令 (Port Binding Commands)

这些命令操作 OVN_Southbound 端口绑定。

*   `[--may-exist] lsp-bind logical-port chassis`
    将名为 `logical-port` 的逻辑端口绑定到 `chassis`。
    *   这通常由 `ovn-controller` 自动完成，但在某些测试或手动干预场景下可能有用。

*   `[--if-exists] lsp-unbind logical-port`
    移除 `logical-port` 的绑定。
