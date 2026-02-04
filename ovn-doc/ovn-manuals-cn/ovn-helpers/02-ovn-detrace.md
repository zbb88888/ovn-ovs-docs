# OVN 追踪转换工具 (ovn-detrace)

## 简介 (Synopsis)

```bash
ovn-detrace < file
```

## 描述 (Description)

`ovn-detrace` 程序从标准输入读取 `ovs-appctl ofproto/trace` 的输出，查找流 Cookie，并为每个 Cookie 显示促成其创建的 OVN Southbound 记录。它还会进一步显示相关的 Northbound 信息（如果适用），例如生成被转换为具有给定 Cookie 的 OpenFlow 规则的逻辑流的 ACL。

这对于将 OpenFlow 层面的追踪结果映射回 OVN 逻辑层非常有帮助。

## 选项 (Options)

*   `--ovnsb=server`
    要联系的 OVN Southbound DB 远程地址。默认 `unix:@RUNDIR@/ovnsb_db.sock`。

*   `--ovnnb=server`
    要联系的 OVN Northbound DB 远程地址。默认 `unix:@RUNDIR@/ovnnb_db.sock`。

*   `--ovs`
    通过连接到 OVS DB，解码流中的信息（如 OVS `ofport`）。

*   `--no-leader-only`
    连接到任何集群成员，而不仅仅是领导者。

*   `--ovsdb=server`
    如果使用了 `--ovs`，指定 OVS DB 远程地址。默认 `unix:@RUNDIR@/db.sock`。

*   `-p`, `--private-key`
*   `-c`, `--certificate`
*   `-C`, `--ca-cert`
    SSL/TLS 配置。
