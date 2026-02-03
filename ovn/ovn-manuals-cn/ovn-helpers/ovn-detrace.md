# ovn-detrace (Trace Decoder)

## 简介

`ovn-detrace` 是一个强大的命令行工具，用于关联物理流表与逻辑配置。

- **背景**：当你使用 `ovs-appctl ofproto/trace` 调试 Open vSwitch 的物理流表时，输出中包含大量的 OpenFlow cookie，但很难直观地知道这些流表对应哪条 OVN 逻辑流或 ACL。
- **功能**：`ovn-detrace` 读取标准输入中的 trace 日志，提取 cookie，查询 Southbound DB，打印出生成该物理流表的原始 OVN 逻辑流信息（甚至 Northbound ACL）。

## 用法

通常配合管道使用：

```bash
ovs-appctl ofproto/trace br-int <flow> | ovn-detrace
```

## 示例输出

```text
cookie=0x12345678, duration=... table=10, ... actions=output:1
    OVN: <Logical Flow Information>
    bridge("br-int")
    table(10)
    match(reg0=0x1)
    action(output:1)
    
    related-lb: ...
    related-acl: ...
```

## 选项

- `--ovnnb-db=server`: 指定 NB DB 地址。
- `--ovnsb-db=server`: 指定 SB DB 地址。
- `--no-highlight`: 禁用输出高亮。
