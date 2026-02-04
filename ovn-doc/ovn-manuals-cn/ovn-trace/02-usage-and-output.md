# OVN 追踪输出与状态化动作

## 输出格式 (Output Formats)

`ovn-trace` 支持三种不同形式的输出。无论选择哪种格式，`ovn-trace` 都会以一行显示正在追踪的微流（OpenFlow 语法）开始。

### 详细输出 (Detailed Output)

这是默认格式。它将输出分组为正在遍历的 Ingress 或 Egress 管道。每个管道列出：
1.  被访问的每个表（编号和名称）。
2.  添加该流的 `ovn-northd` 源代码文件和行号。
3.  匹配的逻辑流的匹配表达式和优先级。
4.  执行的动作。

为了可读性，对于“尾递归”（即 `next` 是逻辑流中的最后一个动作），`ovn-trace` 省略缩进。它也省略了微不足道的流（匹配 `1` 且动作为 `next;` 的流）。

**示例**:
```
ingress(dp="ls1", inport="lp111")
---------------------------------
0. ls_in_port_sec_l2: inport == "lp111", priority 50
next(1);
19. ls_in_l2_lkup: eth.dst == 00:00:00:00:ff:11, priority 50
outport = "lrp11-attachment";
output;
```

### 摘要输出 (Summary Output)

摘要输出包括数据包访问的逻辑管道和执行的逻辑动作。与详细输出相比，它删除了数据包遍历的表和逻辑流的细节。它使用更接近编程语言的格式，并且不尝试避免缩进。

**示例**:
```
ingress(dp="ls1", inport="lp111") {
    outport = "lrp11-attachment";
    output;
    ...
};
```

### 最小输出 (Minimal Output)

最小输出仅包括修改数据包数据的动作（不包括 OVN 寄存器或元数据，如 `outport`）以及实际将数据包传递到逻辑端口的 `output` 动作（不包括 Patch Port）。修改数据包数据的动作的操作数显示为简化后的常量。

这种格式最适合用于回归测试，因为它最不可能在逻辑流表重新排列而没有语义更改时发生变化。

## 状态化动作 (Stateful Actions)

某些 OVN 逻辑动作使用或更新 Southbound 数据库中不可用的状态。`ovn-trace` 按如下方式处理这些动作：

*   `ct_next`
    默认情况下，`ovn-trace` 将流视为“已跟踪”(`tracked`) 和“已建立”(`established`)。使用 `--ct` 选项可覆盖此行为。

*   `ct_dnat` (无参数)
    分叉管道。
    1.  在一个分支中，前进到下一个表，就像执行了 `next;` 一样。假设没有可用的 NAT 状态，数据包未更改。
    2.  在另一个分支中，管道在 `ct_dnat` 动作后继续，不发生变化。

*   `ct_snat` (无参数)
    *   **网关路由器**: 视为无操作 (`no-op`)。
    *   **分布式路由器**: 与 `ct_dnat;` 处理方式相同（分叉管道）。

*   `ct_dnat(ip)` / `ct_snat(ip)`
    分叉管道。
    1.  在一个分支中，设置 `ip4.dst` (或 `src`) 为 `ip`，设置 `ct.dnat` (或 `snat`) 为 1，并前进到下一个表。
    2.  在另一个分支中，管道继续不发生变化。

*   `ct_lb` / `ct_lb(ip[:port]...)`
    分叉管道。
    1.  在一个分支中，设置目的 IP 为负载均衡地址之一，设置 `ct.dnat` 为 1。
    2.  在另一个分支中，管道继续不发生变化。

*   `ct_commit`, `put_arp`, `put_nd`
    这些动作被视为无操作 (`no-ops`)。
