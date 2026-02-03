# ovn-trace (Logical Network Tracing)

## 简介

`ovn-trace` 是 OVN 最重要的调试工具。它用于模拟一个数据包在 OVN 逻辑网络中的传输路径。

- **原理**：它读取 Northbound 和 Southbound 数据库，在用户空间模拟 `ovn-northd` 和 `ovn-controller` 的逻辑，展示数据包如何匹配逻辑流表 (Logical Flows)。
- **用途**：排查 ACL 是否错误拦截、路由是否正确、NAT 是否生效等。
- **特点**：不需要实际发送数据包，不影响现网流量。

## 基本用法

```bash
ovn-trace [options] <datapath> <microflow>
```

- **datapath**: 逻辑交换机或逻辑路由器的名称（如 `ls1`）。
- **microflow**: 描述数据包特征的字符串（使用 OVN 匹配语法）。

## 示例

### 1. 基础追踪

追踪从逻辑端口 `vm1` 发往 `10.0.0.2` 的报文：

```bash
ovn-trace ls1 'inport == "vm1" && ip4.dst == 10.0.0.2'
```

### 2. 完整 L3 追踪

追踪包含详细以太网和 IP 头部的报文：

```bash
ovn-trace ls1 'inport == "vm1" && eth.src == 00:01:02:03:04:05 && eth.dst == 00:10:20:30:40:50 && ip4.src == 192.168.1.2 && ip4.dst == 192.168.1.3 && ip.ttl == 64'
```

## 输出解读

`ovn-trace` 的输出显示了报文经过的每一步逻辑流水线：

```text
# 示例输出
ct_next(ct_state=est|trk /* default */)
---------------------------------------
 0. ls_in_port_sec_l2 (ovn-northd.c:2828): ip4.src == {192.168.1.2} && eth.src == 00:01:02:03:04:05, priority 50, uuid 3a1b2c
    next;  <-- 匹配到的动作
 1. ls_in_port_sec_ip (ovn-northd.c:2231): ip4.src == {192.168.1.2}, priority 90, uuid 4d5e6f
    next;
 ...
19. ls_in_l2_lkup (ovn-northd.c:3120): eth.dst == 00:10:20:30:40:50, priority 50, uuid 7a8b9c
    outport = "lp2";  <-- 确定出端口
    output;

egress(dp="ls1", inport="lp1", outport="lp2")  <-- 进入出站流水线
---------------------------------------------
 0. ls_out_pre_lb ...
    allow;
 ...
 9. ls_out_port_sec_l2 ...
    output;
    /* output to "lp2", type "" */  <-- 最终结果
```

### 关键字段

- **pipeline**: 当前处于 `ingress` 还是 `egress` 阶段。
- **table**: 当前匹配的逻辑表名称（如 `ls_in_l2_lkup`）。
- **priority**: 匹配规则的优先级。
- **match**: 匹配条件。
- **action**: 执行的动作（如 `next`, `output`, `ct_commit`）。

## 进阶选项

- `--summary`: 仅显示最终结果（输出端口或丢弃），隐藏中间过程。
- `--minimal`: 减少输出的详细程度。
- `--ovnnb-db`, `--ovnsb-db`: 指定数据库连接地址。
