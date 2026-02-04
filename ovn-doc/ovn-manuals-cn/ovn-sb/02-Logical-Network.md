# Address_Set 表

该表包含从 `OVN_Northbound` 数据库的 `Address_Set` 表同步的地址集，以及从 `OVN_Northbound` 数据库的 `Port_Group` 表生成的地址集。

有关详细信息，请参阅 `OVN_Northbound` 数据库中的 `Address_Set` 表和 `Port_Group` 表的文档。

## 汇总

*   `name`
    *   字符串 (在表中必须唯一)
*   `addresses`
    *   字符串集合

## 详细信息

*   `name`: 字符串 (在表中必须唯一)

*   `addresses`: 字符串集合

# Port_Group 表

该表包含属于 `OVN_Northbound` 数据库中定义的同一 `Port_Group` 的逻辑交换机端口的名称。

## 汇总

*   `name`
    *   字符串 (在表中必须唯一)
*   `ports`
    *   字符串集合

## 详细信息

*   `name`: 字符串 (在表中必须唯一)

*   `ports`: 字符串集合

# Logical_Flow 表

此表中的每一行代表一个逻辑流。`ovn-northd` 使用实现 `OVN_Northbound` 数据库中指定的 L2 和 L3 拓扑的逻辑流填充此表。每个管理程序通过 `ovn-controller` 将逻辑流转换为特定于其管理程序的 OpenFlow 流，并将它们安装到 Open vSwitch 中。

逻辑流以 OVN 特定的格式表示，此处对此进行了描述。逻辑数据路径流非常类似于 OpenFlow 流，不同之处在于流是用逻辑端口和逻辑数据路径而不是物理端口和物理数据路径编写的。逻辑流和物理流之间的转换有助于确保逻辑数据路径之间的隔离。（逻辑流抽象还允许 OVN 集中式组件做更少的工作，因为它们不必单独计算并将物理流推送到每个机箱。）

未匹配任何流时的默认操作是丢弃数据包。

## 数据包的架构逻辑生命周期

以下描述侧重于数据包通过逻辑数据路径的生命周期，忽略实现的物理细节。有关物理信息，请参阅 `ovn-architecture(7)` 中的“数据包的架构物理生命周期”。

这里的描述就像 OVN 本身执行这些步骤一样，但实际上 OVN（即 `ovn-controller`）通过 OpenFlow 和 OVSDB 对 Open vSwitch 进行编程以代表它执行这些步骤。

在高层次上，OVN 将每个数据包通过逻辑数据路径的逻辑入口管道传递，该管道可以将数据包输出到一个或多个逻辑端口或逻辑多播组。对于每个这样的逻辑输出端口，OVN 将数据包通过数据路径的逻辑出口管道传递，该管道可以丢弃数据包或将其传递到目的地。在两个管道之间，对逻辑多播组的输出被扩展为逻辑端口，以便出口管道一次只处理一个逻辑输出端口。在两个管道之间也是 OVN 在必要时将数据包封装在隧道（或多个隧道）中以传输到远程管理程序的地方。

更详细地说，首先，OVN 在 `Logical_Flow` 表中搜索具有正确的 `logical_datapath` 或 `logical_dp_group`、入口管道、`table_id` 为 0 且匹配项对数据包为真的行。如果未找到，OVN 将丢弃数据包。如果 OVN 找到多个，它会选择优先级最高的匹配项。然后 OVN 执行该行 `actions` 列中指定的每个操作，按照指定的顺序执行。某些操作，例如修改数据包头的操作，不需要进一步的细节。`next` 和 `output` 操作是特殊的。

`next` 操作导致递归重复上述过程，除了 OVN 搜索 `table_id` 为 1 而不是 0。同样，在该表中找到的行中的任何 `next` 操作都会导致进一步搜索 `table_id` 为 2，依此类推。当递归处理完成时，流控制返回到 `next` 之后的操作。

`output` 操作也引入了递归。其效果取决于 `outport` 字段的当前值。假设 `outport` 指定了一个逻辑端口。首先，OVN 比较 `inport` 和 `outport`；如果它们相等，则默认情况下它将输出视为无操作。在常见情况下，如果它们不同，数据包进入出口管道。这种向出口管道的转换会丢弃寄存器数据，例如 `reg0` ... `reg9` 和连接跟踪状态，以实现统一的行为，无论出口管道是否在不同的管理程序上（因为寄存器不会跨隧道封装保留）。

为了执行出口管道，OVN 再次在 `Logical_Flow` 表中搜索具有正确的 `logical_datapath` 或 `logical_dp_group`、`table_id` 为 0、匹配项对数据包为真，但现在寻找出口管道的行。如果未找到匹配的行，则输出变为无操作。否则，OVN 执行匹配流的操作（如前所述，如果需要，可以从多个流中选择）。

在出口管道中，`next` 操作如前所述起作用，当然，它搜索出口流。然而，`output` 操作现在直接将数据包输出到输出端口（现在是固定的，因为 `outport` 在出口管道内是只读的）。

前面的描述假设 `outport` 指的是逻辑端口。如果它指定的是逻辑多播组，则上述描述仍然适用，但增加了从逻辑多播组到组中每个逻辑端口的扇出。对于组中的每个成员，OVN 如所述执行逻辑管道，逻辑输出端口替换为组成员。

## 管道阶段

`ovn-northd` 使用 `ovn-northd(8)` 中详细描述的逻辑流填充 `Logical_Flow` 表。

## 汇总

*   `logical_datapath`
    *   可选的 `Datapath_Binding`
*   `logical_dp_group`
    *   可选的 `Logical_DP_Group`
*   `pipeline`
    *   字符串，`egress` 或 `ingress`
*   `table_id`
    *   整数，范围 0 到 32
*   `priority`
    *   整数，范围 0 到 65,535
*   `match`
    *   字符串
*   `actions`
    *   字符串
*   `tags`
    *   字符串对的映射
*   `controller_meter`
    *   可选字符串
*   `flow_desc`
    *   可选字符串
*   `external_ids : stage-name`
    *   可选字符串
*   `external_ids : stage-hint`
    *   可选字符串，包含 UUID
*   `external_ids : source`
    *   可选字符串
*   通用列:
    *   `external_ids`
        *   字符串对的映射

## 详细信息

*   `logical_datapath`: 可选的 `Datapath_Binding`
    *   逻辑流所属的逻辑数据路径。

*   `logical_dp_group`: 可选的 `Logical_DP_Group`
    *   逻辑流所属的逻辑数据路径组。这意味着相同的逻辑流属于组中的所有数据路径。

*   `pipeline`: 字符串，`egress` 或 `ingress`
    *   用于决定数据包目的地的主要流是入口流。出口流实现 ACL。有关详细信息，请参阅上面的“数据包的逻辑生命周期”。

*   `table_id`: 整数，范围 0 到 32
    *   逻辑管道中的阶段，类似于 OpenFlow 表号。

*   `priority`: 整数，范围 0 到 65,535
    *   流的优先级。数值优先级较高的流优先于较低优先级的流。如果两个具有相同优先级的逻辑数据路径流都匹配，则实际应用于数据包的流是未定义的。

*   `match`: 字符串
    *   匹配表达式。OVN 提供了 OpenFlow 匹配功能的超集，使用类似于编程语言中布尔表达式的语法。

    *   匹配表达式最重要的组成部分是符号和常量之间的比较，例如 `ip4.dst == 192.168.0.1`，`ip.proto == 6`，`arp.op == 1`，`eth.type == 0x800`。逻辑与运算符 `&&` 和逻辑或运算符 `||` 可以将比较组合成更大的表达式。

    *   匹配表达式还支持用于分组的括号、逻辑非前缀运算符 `!` 以及表示“假”或“真”的字面量 `0` 和 `1`。后者本身作为一个匹配每个数据包的包罗万象的表达式很有用。

    *   匹配表达式还支持一种函数语法。支持以下函数：

        *   `is_chassis_resident(lport)`
            *   在逻辑端口 `lport`（带引号的字符串）驻留的机箱上计算为真，在其他地方计算为假。此函数是在 OVN 2.7 中引入的。

    *   **符号**

    *   **类型**。符号具有整数或字符串类型。整数符号具有以位为单位的宽度。

    *   **种类**。有三种符号：

        *   **字段**。字段符号表示数据包头或元数据字段。例如，名为 `vlan.tci` 的字段可能表示数据包中的 VLAN TCI 字段。
        
            字段符号可以具有整数或字符串类型。整数字段可以是名义上的或序数的（请参阅下面的“测量级别”）。

        *   **子字段**。子字段表示较大字段的位子集。例如，字段 `vlan.vid` 可以定义为 `vlan.tci[0..11]` 的别名。提供子字段是为了语法上的方便，因为总是可以直接引用字段的位子集。
        
            只有序数字段（请参阅下面的“测量级别”）可以有子字段。子字段总是序数的。

        *   **谓词**。谓词是布尔表达式的简写。谓词可以像 1 位字段一样使用。例如，`ip4` 可能扩展为 `eth.type == 0x800`。提供谓词是为了语法上的方便，因为总是可以直接指定底层表达式。
        
            其扩展引用任何名义字段或谓词（请参阅下面的“测量级别”）的谓词是名义上的；其他谓词具有布尔测量级别。

    *   **测量级别**。有关此分类所基于的统计概念，请参阅 http://en.wikipedia.org/wiki/Level_of_measurement。分为三个级别：

        *   **序数 (Ordinal)**。在统计学中，序数值可以在一个标度上排序。如果 OVN 字段（或子字段）的位可以单独检查，则 OVN 认为它是序数的。对于 OpenFlow 或 Open vSwitch 使其“可掩码”的 OpenFlow 字段，情况确实如此。
        
            序数字段的任何使用都可以指定单个位或位范围，例如 `vlan.tci[13..15]` 指的是 VLAN TCI 中的 PCP 字段，`eth.dst[40]` 指的是以太网目标地址中的多播位。
            
            OVN 支持序数字段及其子字段上的所有常用算术关系（`==`, `!=`, `<`, `<=`, `>`, `>=`），因为 OVN 可以在 OpenFlow 和 Open vSwitch 中将这些实现为按位测试的集合。

        *   **名义 (Nominal)**。在统计学中，名义值除了相等之外不能进行有用的比较。OpenFlow 端口号、以太网类型和 IP 协议就是这样的例子：所有这些都只是任意分配的标识符，没有更深层的含义。在 OpenFlow 和 Open vSwitch 中，这些字段中的位通常不能单独寻址。
        
            OVN 仅支持名义字段上的相等性算术测试，因为 OpenFlow 和 Open vSwitch 没有提供让流在它们上面有效地实现其他比较的方法。（不相等测试可以通过两个具有不同优先级的流构建出来，但 OVN 匹配表达式总是生成具有单一优先级的流。）
            
            字符串字段总是名义上的。

        *   **布尔 (Boolean)**。只有两个值 0 和 1 的名义字段有些例外，因为很容易支持该字段上的相等和不相等测试：任何一个都可以实现为对 0 或 1 的测试。
        
            只有谓词（见上文）具有布尔测量级别。
            
            这不是标准的测量级别。

    *   **先决条件**。任何符号都可以有先决条件，这是使用该符号所隐含的附加条件。例如，`icmp4.type` 符号可能具有先决条件 `icmp4`，这将导致表达式 `icmp4.type == 0` 被解释为 `icmp4.type == 0 && icmp4`，这反过来又会扩展为 `icmp4.type == 0 && eth.type == 0x800 && ip4.proto == 1`（假设 `icmp4` 是如上文“种类”下建议定义的谓词）。

    *   **关系运算符**

    *   支持所有标准关系运算符 `==`, `!=`, `<`, `<=`, `>`, `>=`。名义字段仅支持 `==` 和 `!=`，并且仅在考虑到外部 `!` 时以积极意义支持，例如，给定字符串字段 `inport`，`inport == "eth0"` 和 `!(inport != "eth0")` 是可以接受的，但 `inport != "eth0"` 是不可接受的。

    *   `==`（或被否定时的 `!=`）的实现比其他关系运算符的实现更有效。

    *   **常量**

    *   整数常量可以用十进制、前缀为 `0x` 的十六进制表示，或者表示为点分四组 IPv4 地址、标准形式的 IPv6 地址或冒号分隔的十六进制数字的以太网地址。任何这些形式的常量后面都可以跟一个斜杠和相同形式的第二个常量（掩码），以形成掩码常量。IPv4 和 IPv6 掩码可以作为整数给出，以表示 CIDR 前缀。

    *   字符串常量具有与 JSON 中引用的字符串相同的语法（因此，它们是 Unicode 字符串）。

    *   某些运算符支持写在花括号 `{ ... }` 内的常量集。集合元素之间以及最后一个元素之后的逗号是可选的。对于 `==`，`field == { constant1, constant2, ... }` 是 `field == constant1 || field == constant2 || ...` 的语法糖。类似地，`field != { constant1, constant2, ... }` 等价于 `field != constant1 && field != constant2 && ...`。

    *   您可以按名称引用存储在 `Address_Set` 表中的一组 IPv4、IPv6 或 MAC 地址。名称为 `set1` 的 `Address_Set` 可以引用为 `$set1`。

    *   您可以按名称引用存储在 `Port_Group` 表中的一组逻辑交换机端口。名称为 `port_group1` 的 `Port_Group` 可以引用为 `@port_group1`。

    *   此外，您可以按其名称后跟后缀 `_ip4` / `_ip6` 来引用属于存储在 `Port_Group` 表中的一组逻辑交换机端口的地址集。名称为 `port_group1` 的 `Port_Group` 的 IPv4 地址集可以引用为 `$port_group1_ip4`，同一 `Port_Group` 的 IPv6 地址集可以引用为 `$port_group1_ip6`。

    *   **杂项**

    *   比较可以先命名符号或常量，例如 `tcp.src == 80` 和 `80 == tcp.src` 都是可以接受的。

    *   范围测试可以使用类似 `1024 <= tcp.src <= 49151` 的语法表示，这等价于 `1024 <= tcp.src && tcp.src <= 49151`。

    *   对于一位字段或谓词，提及它的名称等价于 `symbol == 1`，例如 `vlan.present` 等价于 `vlan.present == 1`。对于一位子字段也是如此，例如 `vlan.tci[12]`。实现所有宽度的序数字段的相同功能没有技术限制，但实现足够昂贵，以至于语法解析器要求编写针对零的显式比较以减少错误的可能性，例如在 `tcp.src != 0` 中，需要针对 0 的比较。

    *   运算符优先级如下所示，从高到低。有两个例外，即使表格表明不需要括号，也需要括号：`&&` 和 `||` 一起使用时需要括号，`!` 应用于关系表达式时需要括号。因此，在 `(eth.type == 0x800 || eth.type == 0x86dd) && ip.proto == 6` 或 `!(arp.op == 1)` 中，括号是强制性的。

        *   `()`
        *   `==   !=   <   <=   >   >=`
        *   `!`
        *   `&&   ||`

    *   注释可以由 `//` 引入，它延伸到下一个换行符。行内的注释可以用 `/*` 和 `*/` 括起来。不支持多行注释。

    *   **符号**

    *   下面的大多数符号都具有整数类型。只有 `inport` 和 `outport` 具有字符串类型。`inport` 命名一个逻辑端口。因此，它的值是来自 `Port_Binding` 表的 `logical_port` 名称。`outport` 可以命名一个逻辑端口，如 `inport`，或者是在 `Multicast_Group` 表中定义的逻辑多播组。对于这两个符号，只能使用流的逻辑数据路径中的名称。

    *   `regX` 符号是 32 位整数。`xxregX` 符号是 128 位整数，它们覆盖了四个 32 位寄存器：`xxreg0` 覆盖 `reg0` 到 `reg3`，其中 `reg0` 提供 `xxreg0` 的最高有效位，`reg3` 提供最低有效位。`xxreg1` 类似地覆盖 `reg4` 到 `reg7`。

        *   `reg0`...`reg9`
        *   `xxreg0` `xxreg1`
        *   `inport` `outport`
        *   `flags.loopback`
        *   `pkt.mark`
        *   `eth.src` `eth.dst` `eth.type`
        *   `vlan.tci` `vlan.vid` `vlan.pcp` `vlan.present`
        *   `ip.proto` `ip.dscp` `ip.ecn` `ip.ttl` `ip.frag`
        *   `ip4.src` `ip4.dst`
        *   `ip6.src` `ip6.dst` `ip6.label`
        *   `arp.op` `arp.spa` `arp.tpa` `arp.sha` `arp.tha`
        *   `rarp.op` `rarp.spa` `rarp.tpa` `rarp.sha` `rarp.tha`
        *   `tcp.src` `tcp.dst` `tcp.flags`
        *   `udp.src` `udp.dst`
        *   `sctp.src` `sctp.dst`
        *   `icmp4.type` `icmp4.code`
        *   `icmp6.type` `icmp6.code`
        *   `nd.target` `nd.sll` `nd.tll`
        *   `ct_mark` `ct_label`
        *   `ct_state`，它有几个布尔子字段。`ct_next` 操作初始化以下子字段：
            *   `ct.trk`: 始终由 `ct_next` 设置为 true，以指示已发生连接跟踪。所有其他 `ct` 子字段都以 `ct.trk` 为先决条件。
            *   `ct.new`: 对于新流为真
            *   `ct.est`: 对于已建立的流为真
            *   `ct.rel`: 对于相关流为真
            *   `ct.rpl`: 对于回复流为真
            *   `ct.inv`: 对于处于错误状态的连接条目为真
            
            `ct_dnat`、`ct_snat` 和 `ct_lb` 操作初始化以下子字段：
            *   `ct.dnat`: 对于目标 IP 地址已更改的数据包为真。
            *   `ct.snat`: 对于源 IP 地址已更改的数据包为真。

    *   支持以下谓词：

        *   `eth.bcast` 扩展为 `eth.dst == ff:ff:ff:ff:ff:ff`
        *   `eth.mcast` 扩展为 `eth.dst[40]`
        *   `eth.mcastv6` 扩展为 `eth.dst[32..47] == 0x3333`
        *   `vlan.present` 扩展为 `vlan.tci[12]`
        *   `ip4` 扩展为 `eth.type == 0x800`
        *   `ip4.src_mcast` 扩展为 `ip4.src[28..31] == 0xe`
        *   `ip4.mcast` 扩展为 `ip4.dst[28..31] == 0xe`
        *   `ip6` 扩展为 `eth.type == 0x86dd`
        *   `ip` 扩展为 `ip4 || ip6`
        *   `icmp4` 扩展为 `ip4 && ip.proto == 1`
        *   `icmp6` 扩展为 `ip6 && ip.proto == 58`
        *   `icmp` 扩展为 `icmp4 || icmp6`
        *   `ip.is_frag` 扩展为 `ip.frag[0]`
        *   `ip.later_frag` 扩展为 `ip.frag[1]`
        *   `ip.first_frag` 扩展为 `ip.is_frag && !ip.later_frag`
        *   `arp` 扩展为 `eth.type == 0x806`
        *   `rarp` 扩展为 `eth.type == 0x8035`
        *   `ip6.mcast` 扩展为 `eth.mcastv6 && ip6.dst[120..127] == 0xff`
        *   `nd` 扩展为 `icmp6.type == {135, 136} && icmp6.code == 0 && ip.ttl == 255`
        *   `nd_ns` 扩展为 `icmp6.type == 135 && icmp6.code == 0 && ip.ttl == 255`
        *   `nd_ns_mcast` 扩展为 `ip6.mcast && icmp6.type == 135 && icmp6.code == 0 && ip.ttl == 255`
        *   `nd_na` 扩展为 `icmp6.type == 136 && icmp6.code == 0 && ip.ttl == 255`
        *   `nd_rs` 扩展为 `icmp6.type == 133 && icmp6.code == 0 && ip.ttl == 255`
        *   `nd_ra` 扩展为 `icmp6.type == 134 && icmp6.code == 0 && ip.ttl == 255`
        *   `tcp` 扩展为 `ip.proto == 6`
        *   `udp` 扩展为 `ip.proto == 17`
        *   `sctp` 扩展为 `ip.proto == 132`

*   `actions`: 字符串
    *   逻辑数据路径操作，当此行表示的逻辑流是最高优先级匹配时执行。

    *   操作与 `match` 列共享词法语法。一组空的操作（或仅包含空格或注释的操作），或仅包含 `drop;` 的一组操作，会导致丢弃匹配的数据包。否则，该列应包含一系列操作，每个操作以分号结尾。

    *   定义了以下操作：

        *   `output;`
            *   在入口管道中，此操作作为子程序执行出口管道。如果 `outport` 命名逻辑端口，则出口管道执行一次；如果它是多播组，则出口管道为组中的每个逻辑端口运行一次。
            *   在出口管道中，此操作执行到 `outport` 逻辑端口的实际输出。（在出口管道中，`outport` 从不命名多播组。）
            *   默认情况下，对输入端口的输出被隐式丢弃，也就是说，如果 `outport == inport`，则输出变为无操作。偶尔覆盖此行为可能很有用，例如向 ARP 请求发送 ARP 回复；为此，请使用 `flags.loopback = 1` 以允许数据包“发夹”回输入端口。

        *   `next;` `next(table);` `next(pipeline=pipeline, table=table);`
            *   作为子程序执行 `pipeline` 中的给定逻辑数据路径表。默认表就在当前表之后。如果指定了 `pipeline`，它可以是 `ingress` 或 `egress`；默认管道是当前正在执行的管道。入口和出口管道中的操作都可以使用 `next` 跳转到另一个管道。入口管道中的操作应该只有在确定数据包是本地的且没有隧道传输，并且希望跳过数据包处理中的某些阶段时，才使用 `next` 跳转到出口管道的特定表。

        *   `field = constant;`
            *   将数据或元数据字段 `field` 设置为常量值 `constant`，例如 `outport = "vif0";` 设置逻辑输出端口。要仅设置字段中的位子集，请为 `field` 指定子字段或掩码常量，例如可以使用 `vlan.pcp[2] = 1;` 或 `vlan.pcp = 4/4;` 设置 VLAN PCP 的最高有效位。
            *   分配给具有先决条件的字段会隐式地将这些先决条件添加到匹配中；因此，例如，设置 `tcp.dst` 的流仅适用于 TCP 流，无论其匹配是否提及任何 TCP 字段。
            *   并非所有字段都是可修改的（例如 `eth.type` 和 `ip.proto` 是只读的），并非所有可修改的字段都可以部分修改（例如 `ip.ttl` 必须整体赋值）。`outport` 字段在入口管道中是可修改的，但在出口管道中是不可修改的。

        *   `ovn_field = constant;`
            *   将 OVN 字段 `ovn_field` 设置为常量值 `constant`。
            *   OVN 支持设置某些尚未在 OpenFlow 中支持以设置或修改它们的字段的值。
            *   以下是支持的 OVN 字段：
                *   `icmp4.frag_mtu` `icmp6.frag_mtu`
                    *   此字段将 RFC 1191 中定义的 ICMP 规范中标记为“未使用”的 ICMP{4,6} 标头字段的低 16 位设置为 `constant` 中指定的值。
                    *   例如：`icmp4.frag_mtu = 1500;`

        *   `field1 = field2;`
            *   将数据或元数据字段 `field1` 设置为数据或元数据字段 `field2` 的值，例如 `reg0 = ip4.src;` 将 `ip4.src` 复制到 `reg0`。要仅修改字段位的子集，请为 `field1` 或 `field2` 或两者指定子字段，例如 `vlan.pcp = reg0[0..2];` 将 `reg0` 的最低有效位复制到 VLAN PCP。
            *   `field1` 和 `field2` 必须是同一类型，要么都是字符串，要么都是整数字段。如果它们都是整数字段，则它们必须具有相同的宽度。
            *   如果 `field1` 或 `field2` 具有先决条件，则它们被隐式添加到匹配中。可以编写具有矛盾先决条件的赋值，例如 `ip4.src = ip6.src[0..31];`，但矛盾意味着具有此类赋值的逻辑流将永远不会被匹配。

        *   `field1 <-> field2;`
            *   类似于 `field1 = field2;`，除了交换两个值而不是复制。`field1` 和 `field2` 都必须是可修改的。

        *   `push(field);`
            *   将 `field` 的值压入堆栈顶部。

        *   `pop(field);`
            *   弹出堆栈顶部并将值存储到 `field`，它必须是可修改的。

        *   `ip.ttl--;`
            *   递减 IPv4 或 IPv6 TTL。如果这将使 TTL 为零或负数，则数据包的处理停止；不再处理其他操作。（为了正确处理这种情况，更高优先级的流应该匹配 `ip.ttl == {0, 1};`。）
            *   先决条件：`ip`

        *   `ct_next;` `ct_next(dnat);` `ct_next(snat);`
            *   将连接跟踪应用于流，初始化 `ct_state` 以供后续表中的匹配使用。自动移动到下一个表，就像后面跟着 `next` 一样。
            *   作为副作用，IP 分片将被重组以进行匹配。如果输出了分片数据包，那么它将在任何重叠分片被压缩的情况下发送。当该操作用于逻辑交换机的流时，连接跟踪状态由逻辑端口限定，因此可以使用重叠地址。为了允许与匹配流相关的流量，执行 `ct_commit`。当该操作用于路由器的流时，连接跟踪状态由逻辑拓扑限定。
            *   可以让操作跟随 `ct_next`，但它们将无法访问其任何副作用，并且通常没有用处。

        *   `ct_commit { };` `ct_commit { ct_mark=value[/mask]; };` `ct_commit { ct_label=value[/mask]; };` `ct_commit { ct_mark=value[/mask]; ct_label=value[/mask]; };`
            *   将流提交到由先前调用 `ct_next` 与其关联的连接跟踪条目。当提供 `ct_mark=value[/mask]` 和/或 `ct_label=value[/mask]` 时，`ct_mark` 和/或 `ct_label` 将被设置为连接跟踪条目上 `value[/mask]` 指示的值。`ct_mark` 是 32 位字段。`ct_label` 是 128 位字段。如果要使用超过 64 位，则 `value[/mask]` 应以十六进制字符串指定。寄存器和其他命名字段可用于 `value`。`ct_mark` 和 `ct_label` 可以进行子寻址以设置特定位。
            *   请注意，如果您希望在下一个表中继续处理，则必须在 `ct_commit` 之后执行 `next` 操作。您也可以省略 `next`，这将提交连接跟踪状态，然后丢弃数据包。例如，这对于在丢弃数据包之前在连接跟踪条目上设置 `ct_mark` 很有用。

        *   `ct_commit_to_zone(dnat);` `ct_commit_to_zone(snat);`
            *   将流提交到连接跟踪器中的特定区域。然后数据包会自动发送到下一个表，就像后面跟着 `next;` 操作一样。下一个表将看到连接跟踪器引起的数据包变化。
            *   请注意，此操作仅在逻辑路由器数据路径中有意义，因为逻辑交换机数据路径不使用单独的连接跟踪区域。在逻辑交换机数据路径中使用此操作会回退到将流提交到逻辑端口的 conntrack 区域。

        *   `ct_dnat;` `ct_dnat(IP);`
            *   `ct_dnat` 将数据包通过连接跟踪表中的 DNAT 区域发送，以取消 DNAT 任何在相反方向上被 DNAT 的数据包。然后数据包会自动发送到下一个表，就像后面跟着 `next;` 操作一样。下一个表将看到连接跟踪器引起的数据包变化。
            *   `ct_dnat(IP)` 将数据包通过 DNAT 区域发送，以将数据包的目标 IP 地址更改为括号内提供的地址，并提交连接。然后数据包会自动发送到下一个表，就像后面跟着 `next;` 操作一样。下一个表将看到连接跟踪器引起的数据包变化。

        *   `ct_snat;` `ct_snat(IP);`
            *   `ct_snat` 将数据包通过 SNAT 区域发送，以取消 SNAT 任何在相反方向上被 SNAT 的数据包。数据包会自动发送到下一个表，就像后面跟着 `next;` 操作一样。下一个表将看到连接跟踪器引起的数据包变化。
            *   `ct_snat(IP)` 将数据包通过 SNAT 区域发送，以将数据包的源 IP 地址更改为括号内提供的地址，并提交连接。然后数据包会自动发送到下一个表，就像后面跟着 `next;` 操作一样。下一个表将看到连接跟踪器引起的数据包变化。

        *   `ct_dnat_in_czone;` `ct_dnat_in_czone(IP);`
            *   `ct_dnat_in_czone` 将数据包通过连接跟踪表中公共 NAT 区域（用于 DNAT 和 SNAT）发送，以取消 DNAT 任何在相反方向上被 DNAT 的数据包。然后数据包会自动发送到下一个表，就像后面跟着 `next;` 操作一样。下一个表将看到连接跟踪器引起的数据包变化。
            *   `ct_dnat_in_czone(IP)` 将数据包通过公共 NAT 区域发送，以将数据包的目标 IP 地址更改为括号内提供的地址，并提交连接。然后数据包会自动发送到下一个表，就像后面跟着 `next;` 操作一样。下一个表将看到连接跟踪器引起的数据包变化。

        *   `ct_snat_in_czone;` `ct_snat_in_czone(IP);`
            *   `ct_snat_in_czone` 将数据包通过公共 NAT 区域发送，以取消 SNAT 任何在相反方向上被 SNAT 的数据包。数据包会自动发送到下一个表，就像后面跟着 `next;` 操作一样。下一个表将看到连接跟踪器引起的数据包变化。
            *   `ct_snat_in_czone(IP)` 将数据包通过公共 NAT 区域发送，以将数据包的源 IP 地址更改为括号内提供的地址，并提交连接。然后数据包会自动发送到下一个表，就像后面跟着 `next;` 操作一样。下一个表将看到连接跟踪器引起的数据包变化。

        *   `ct_clear;`
            *   清除连接跟踪状态。

        *   `ct_commit_nat;`
            *   应用 NAT 并将连接提交到 CT。自动移动到下一个表，就像后面跟着 `next` 一样。这对于处于相关状态的现有连接非常有用，并允许将 NAT 也应用于它们。

        *   `clone { action; ... };`
            *   制作正在处理的数据包的副本，并对副本执行每个操作。`clone` 操作之后的操作（如果有）适用于原始的、未修改的数据包。这可以用作在可能修改它的一组操作周围“保存和恢复”数据包的一种方式，并且不应该持久化。

        *   `arp { action; ... };`
            *   暂时将正在处理的 IPv4 数据包替换为 ARP 数据包，并对 ARP 数据包执行每个嵌套操作。`arp` 操作之后的操作（如果有）适用于原始的、未修改的数据包。
            *   此操作操作的 ARP 数据包基于正在处理的 IPv4 数据包初始化，如下所示。这些是嵌套操作可能想要更改的默认值：
                *   `eth.src` 不变
                *   `eth.dst` 不变
                *   `eth.type = 0x0806`
                *   `arp.op = 1` (ARP 请求)
                *   `arp.sha` 从 `eth.src` 复制
                *   `arp.spa` 从 `ip4.src` 复制
                *   `arp.tha = 00:00:00:00:00:00`
                *   `arp.tpa` 从 `ip4.dst` 复制
            *   ARP 数据包具有与它替换的 IP 数据包相同的 VLAN 头（如果有）。
            *   先决条件：`ip4`

        *   `get_arp(P, A);`
            *   参数：逻辑端口字符串字段 `P`，32 位 IP 地址字段 `A`。
            *   在 `P` 的 mac 绑定表中查找 `A`。如果找到条目，则将其以太网地址存储在 `eth.dst` 中，否则将 `00:00:00:00:00:00` 存储在 `eth.dst` 中。
            *   例如：`get_arp(outport, ip4.dst);`

        *   `put_arp(P, A, E);`
            *   参数：逻辑端口字符串字段 `P`，32 位 IP 地址字段 `A`，48 位以太网地址字段 `E`。
            *   添加或更新逻辑端口 `P` 的 mac 绑定表中 IP 地址 `A` 的条目，将其以太网地址设置为 `E`。
            *   例如：`put_arp(inport, arp.spa, arp.sha);`

        *   `R = lookup_arp(P, A, M);`
            *   参数：逻辑端口字符串字段 `P`，32 位 IP 地址字段 `A`，48 位 MAC 地址字段 `M`。
            *   结果：存储到 1 位子字段 `R`。
            *   在 `P` 的 mac 绑定表中查找 `A` 和 `M`。如果找到条目，则在 1 位子字段 `R` 中存储 1，否则存储 0。
            *   例如：`reg0[0] = lookup_arp(inport, arp.spa, arp.sha);`

        *   `R = lookup_arp_ip(P, A);`
            *   参数：逻辑端口字符串字段 `P`，32 位 IP 地址字段 `A`。
            *   结果：存储到 1 位子字段 `R`。
            *   在 `P` 的 mac 绑定表中查找 `A`。如果找到条目，则在 1 位子字段 `R` 中存储 1，否则存储 0。
            *   例如：`reg0[0] = lookup_arp_ip(inport, arp.spa);`

        *   `P = get_fdb(A);`
            *   参数：48 位 MAC 地址字段 `A`。
            *   在 fdb 表中查找 `A`。如果找到条目，则将逻辑端口键存储到输出参数 `P`。
            *   例如：`outport = get_fdb(eth.src);`

        *   `put_fdb(P, A);`
            *   参数：逻辑端口字符串字段 `P`，48 位 MAC 地址字段 `A`。
            *   添加或更新 fdb 表中以太网地址 `A` 的条目，将其逻辑端口键设置为 `P`。
            *   例如：`put_fdb(inport, arp.spa);`

        *   `R = lookup_fdb(P, A);`
            *   参数：48 位 MAC 地址字段 `M`，逻辑端口字符串字段 `P`。
            *   结果：存储到 1 位子字段 `R`。
            *   在 fdb 表中查找 `A`。如果找到条目并且逻辑端口键是 `P`，则在 1 位子字段 `R` 中存储 1，否则存储 0。如果设置了 `flags.localnet`，则如果找到条目并且逻辑端口键是 `P`，或者如果找到条目并且条目端口类型是 VIF，则存储 1。
            *   例如：`reg0[0] = lookup_fdb(inport, eth.src);`

        *   `nd_ns { action; ... };`
            *   暂时将正在处理的 IPv6 数据包替换为 IPv6 邻居请求数据包，并对 IPv6 NS 数据包执行每个嵌套操作。`nd_ns` 操作之后的操作（如果有）适用于原始的、未修改的数据包。
            *   此操作操作的 IPv6 NS 数据包基于正在处理的 IPv6 数据包初始化，如下所示。这些是嵌套操作可能想要更改的默认值：
                *   `eth.src` 不变
                *   `eth.dst` 设置为 IPv6 多播 MAC 地址
                *   `eth.type = 0x86dd`
                *   `ip6.src` 从 `ip6.src` 复制
                *   `ip6.dst` 设置为 IPv6 被请求节点多播地址
                *   `icmp6.type = 135` (邻居请求)
                *   `nd.target` 从 `ip6.dst` 复制
            *   IPv6 NS 数据包具有与它替换的 IP 数据包相同的 VLAN 头（如果有）。
            *   先决条件：`ip6`

        *   `nd_na { action; ... };`
            *   暂时将正在处理的 IPv6 邻居请求数据包替换为 IPv6 邻居通告 (NA) 数据包，并对 NA 数据包执行每个嵌套操作。`nd_na` 操作之后的操作（如果有）适用于原始的、未修改的数据包。
            *   此操作操作的 NA 数据包基于正在处理的 IPv6 数据包初始化，如下所示。这些是嵌套操作可能想要更改的默认值：
                *   `eth.dst` 与 `eth.src` 交换
                *   `eth.type = 0x86dd`
                *   `ip6.dst` 从 `ip6.src` 复制
                *   `ip6.src` 从 `nd.target` 复制
                *   `icmp6.type = 136` (邻居通告)
                *   `nd.target` 不变
                *   `nd.sll = 00:00:00:00:00:00`
                *   `nd.tll` 从 `eth.dst` 复制
            *   ND 数据包具有与它替换的 IPv6 数据包相同的 VLAN 头（如果有）。
            *   先决条件：`nd_ns`

        *   `nd_na_router { action; ... };`
            *   暂时将正在处理的 IPv6 邻居请求数据包替换为 IPv6 邻居通告 (NA) 数据包，在 RSO 标志中设置 `ND_NSO_ROUTER`，并对 NA 数据包执行每个嵌套操作。`nd_na_router` 操作之后的操作（如果有）适用于原始的、未修改的数据包。
            *   此操作操作的 NA 数据包基于正在处理的 IPv6 数据包初始化，如下所示。这些是嵌套操作可能想要更改的默认值：
                *   `eth.dst` 与 `eth.src` 交换
                *   `eth.type = 0x86dd`
                *   `ip6.dst` 从 `ip6.src` 复制
                *   `ip6.src` 从 `nd.target` 复制
                *   `icmp6.type = 136` (邻居通告)
                *   `nd.target` 不变
                *   `nd.sll = 00:00:00:00:00:00`
                *   `nd.tll` 从 `eth.dst` 复制
            *   ND 数据包具有与它替换的 IPv6 数据包相同的 VLAN 头（如果有）。
            *   先决条件：`nd_ns`

        *   `get_nd(P, A);`
            *   参数：逻辑端口字符串字段 `P`，128 位 IPv6 地址字段 `A`。
            *   在 `P` 的 mac 绑定表中查找 `A`。如果找到条目，则将其以太网地址存储在 `eth.dst` 中，否则将 `00:00:00:00:00:00` 存储在 `eth.dst` 中。
            *   例如：`get_nd(outport, ip6.dst);`

        *   `put_nd(P, A, E);`
            *   参数：逻辑端口字符串字段 `P`，128 位 IPv6 地址字段 `A`，48 位以太网地址字段 `E`。
            *   添加或更新逻辑端口 `P` 的 mac 绑定表中 IPv6 地址 `A` 的条目，将其以太网地址设置为 `E`。
            *   例如：`put_nd(inport, nd.target, nd.tll);`

        *   `R = lookup_nd(P, A, M);`
            *   参数：逻辑端口字符串字段 `P`，128 位 IP 地址字段 `A`，48 位 MAC 地址字段 `M`。
            *   结果：存储到 1 位子字段 `R`。
            *   在 `P` 的 mac 绑定表中查找 `A` 和 `M`。如果找到条目，则在 1 位子字段 `R` 中存储 1，否则存储 0。
            *   例如：`reg0[0] = lookup_nd(inport, ip6.src, eth.src);`

        *   `R = lookup_nd_ip(P, A);`
            *   参数：逻辑端口字符串字段 `P`，128 位 IP 地址字段 `A`。
            *   结果：存储到 1 位子字段 `R`。
            *   在 `P` 的 mac 绑定表中查找 `A`。如果找到条目，则在 1 位子字段 `R` 中存储 1，否则存储 0。
            *   例如：`reg0[0] = lookup_nd_ip(inport, ip6.src);`

        *   `R = put_dhcp_opts(D1 = V1, D2 = V2, ..., Dn = Vn);`
            *   参数：一个或多个 DHCP 选项/值对，必须包括 `offerip` 选项（代码 0）。
            *   结果：存储到 1 位子字段 `R`。
            *   仅在入口管道中有效。
            *   当此操作应用于 DHCP 请求数据包（DHCPDISCOVER 或 DHCPREQUEST）时，它将数据包更改为 DHCP 回复（分别为 DHCPOFFER 或 DHCPACK），用参数指定的选项替换选项，并将 1 存储在 `R` 中。
            *   当此操作应用于非 DHCP 数据包或不是 DHCPDISCOVER 或 DHCPREQUEST 的 DHCP 数据包时，它保持数据包不变并将 0 存储在 `R` 中。
            *   `DHCP_Option` 表的内容控制此操作支持的 DHCP 选项名称和值。
            *   例如：`reg0[0] = put_dhcp_opts(offerip = 10.0.0.2, router = 10.0.0.1, netmask = 255.255.255.0, dns_server = {8.8.8.8, 7.7.7.7});`

        *   `R = put_dhcpv6_opts(D1 = V1, D2 = V2, ..., Dn = Vn);`
            *   参数：一个或多个 DHCPv6 选项/值对。
            *   结果：存储到 1 位子字段 `R`。
            *   仅在入口管道中有效。
            *   当此操作应用于 DHCPv6 请求数据包时，它将数据包更改为 DHCPv6 回复，用参数指定的选项替换选项，并将 1 存储在 `R` 中。
            *   当此操作应用于非 DHCPv6 数据包或无效的 DHCPv6 请求数据包时，它保持数据包不变并将 0 存储在 `R` 中。
            *   `DHCPv6_Options` 表的内容控制此操作支持的 DHCPv6 选项名称和值。
            *   例如：`reg0[3] = put_dhcpv6_opts(ia_addr = aef0::4, server_id = 00:00:00:00:10:02, dns_server={ae70::1,ae70::2});`

        *   `set_queue(queue_number);`
            *   参数：队列号 `queue_number`，范围 0 到 61440。
            *   这是 OpenFlow `set_queue` 操作的逻辑等价物。它影响通过物理接口离开管理程序的数据包。对于非零 `queue_number`，它配置数据包排队以匹配为具有 `options:qdisc_queue_id` 匹配 `queue_number` 的 `Port_Binding` 配置的设置。当 `queue_number` 为零时，它将排队重置为默认策略。
            *   例如：`set_queue(10);`

        *   `ct_lb;` `ct_lb(backends=ip[:port][,...][; hash_fields=field1,field2,...][; ct_flag]);`
            *   带有参数时，`ct_lb` 将数据包提交到连接跟踪表，并将数据包的目标 IP 地址（和端口）DNAT 到 `backends` 中指定的 IP 地址或地址（和可选端口）。如果指定了多个逗号分隔的 IP 地址，则每个地址在选择 DNAT 地址时具有相同的权重。默认情况下，`dp_hash` 用作 OpenFlow 组选择方法，但如果指定了 `hash_fields`，则使用 `hash` 作为选择方法，并且列出的字段用作哈希字段。`ct_flag` 字段表示支持的标志之一：`skip_snat` 或 `force_snat`，此标志将存储在 `ct_label` 寄存器中。
            *   不带参数时，`ct_lb` 将数据包发送到连接跟踪表以对数据包进行 NAT。如果数据包是先前通过 `ct_lb(...)` 提交到连接跟踪器的已建立连接的一部分，它将自动被 DNAT 到与该连接中的第一个数据包相同的 IP 地址。
            *   处理会自动移动到下一个表，就像指定了 `next;` 一样，后续表作用于由连接跟踪器修改的数据包。当该操作用于逻辑交换机的流时，连接跟踪状态由逻辑端口限定，因此可以使用重叠地址。当该操作用于路由器的流时，连接跟踪状态由逻辑拓扑限定。

        *   `ct_lb_mark;` `ct_lb_mark(backends=ip[:port][,...][; hash_fields=field1,field2,...][; ct_flag]);`
            *   与 `ct_lb` 相同，除了它内部使用 `ct_mark` 来存储 NAT 标志，而 `ct_lb` 使用 `ct_label` 用于相同的目的。

        *   `R = dns_lookup();`
            *   参数：无参数。
            *   结果：存储到 1 位子字段 `R`。
            *   仅在入口管道中有效。
            *   当此操作应用于有效的 DNS 请求（通常指向端口 53 的 UDP 数据包）时，它尝试使用 `DNS` 表的内容解析查询。如果成功，它将数据包更改为 DNS 回复并将 1 存储在 `R` 中。如果该操作应用于非 DNS 数据包、无效的 DNS 请求数据包或 `DNS` 表不提供答案的有效 DNS 请求，它将保持数据包不变并将 0 存储在 `R` 中。
            *   无论成功与否，该操作都不会对流进行任何必要的更改以将数据包引导回请求者。逻辑管道可以在后续表中使用匹配和操作来实现此行为。
            *   例如：`reg0[3] = dns_lookup();`
            *   先决条件：`udp`

        *   `R = put_nd_ra_opts(D1 = V1, D2 = V2, ..., Dn = Vn);`
            *   参数：RFC 4861 中定义的以下 IPv6 ND 路由器通告选项/值对。
            *   `addr_mode`
                *   强制参数，指定要在 RA 标志选项字段中设置的地址模式标志。此选项的值是一个字符串，可以定义以下值 - "slaac"、"dhcpv6_stateful" 和 "dhcpv6_stateless"。
            *   `slla`
                *   强制参数，指定发送路由器通告的接口的链路层地址。
            *   `mtu`
                *   可选参数，指定 MTU。
            *   `prefix`
                *   可选参数，如果 `addr_mode` 为 "slaac" 或 "dhcpv6_stateless"，则应指定该参数。该值应为将用于无状态 IPv6 地址配置的 IPv6 前缀。此选项可以定义多次。
            *   结果：存储到 1 位子字段 `R`。
            *   仅在入口管道中有效。
            *   当此操作应用于 IPv6 路由器请求数据包时，它将数据包更改为 IPv6 路由器通告回复并添加参数中指定的选项，并将 1 存储在 `R` 中。
            *   当此操作应用于非 IPv6 路由器请求数据包或无效的 IPv6 请求数据包时，它保持数据包不变并将 0 存储在 `R` 中。
            *   例如：`reg0[3] = put_nd_ra_opts(addr_mode = "slaac", slla = 00:00:00:00:10:02, prefix = aef0::/64, mtu = 1450);`

        *   `set_meter(rate);` `set_meter(rate, burst);`
            *   参数：速率限制整数字段 `rate` (kbps)，突发速率限制整数字段 `burst` (kbps)。
            *   此操作设置流的速率限制。
            *   例如：`set_meter(100, 1000);`

        *   `R = check_pkt_larger(L)`
            *   参数：要检查的数据包长度 `L`（以字节为单位）。
            *   结果：存储到 1 位子字段 `R`。
            *   这是 OpenFlow `check_pkt_larger` 操作的逻辑等价物。如果数据包大于 `L` 中指定的长度，则在子字段 `R` 中存储 1。
            *   例如：`reg0[6] = check_pkt_larger(1000);`

        *   `log(key=value, ...);`
            *   导致 `ovn-controller` 在处理数据包的机箱上记录数据包。数据包日志记录当前使用与其他 Open vSwitch 和 OVN 消息相同的日志记录机制，这意味着日志消息是否出现以及出现在何处取决于可以使用 `ovn-appctl` 等配置的本地日志记录配置。
            *   `log` 操作采用零个或多个以下键值对参数来控制记录的内容：
                *   `name=string`
                    *   ACL 的可选名称。该字符串目前限制为 64 字节。
                *   `severity=level`
                    *   指示事件的严重性。级别是以下之一（从严重到不严重）：`alert`、`warning`、`notice`、`info` 或 `debug`。如果未提供严重性，则默认为 `info`。
                *   `verdict=value`
                    *   匹配流的数据包的判决。该值必须是 `allow`、`deny` 或 `reject` 之一。
                *   `meter=string`
                    *   要应用于日志的可选速率限制仪表。该字符串应引用 `Meter` 表中的名称条目。唯一合适的仪表操作是 `drop`。

        *   `fwd_group(liveness=bool, childports=port, ...);`
            *   参数：可选的 `liveness`，`true` 或 `false`，默认为 `false`；`childports`，表示要负载平衡的逻辑端口的逗号分隔字符串列表。
            *   将流量负载平衡到逻辑交换机中的一个或多个子端口。`ovn-controller` 将 `fwd_group` 转换为 OpenFlow 组，每个子端口一个桶。如果指定了 `liveness=true`，它还将桶选择与对应于子端口的隧道接口上的 BFD 状态集成。
            *   例如：`fwd_group(liveness=true, childports="p1", "p2");`

        *   `icmp4 { action; ... };` `icmp4_error { action; ... };`
            *   暂时将正在处理的 IPv4 数据包替换为 ICMPv4 数据包，并对 ICMPv4 数据包执行每个嵌套操作。这些操作之后的操作（如果有）适用于原始的、未修改的数据包。
            *   这些操作操作的 ICMPv4 数据包基于正在处理的 IPv4 数据包初始化，如下所示。这些是嵌套操作可能想要更改的默认值。此处未列出的以太网和 IPv4 字段未更改：
                *   `ip.proto = 1` (ICMPv4)
                *   `ip.frag = 0` (不是分片)
                *   `ip.ttl = 255`
                *   `icmp4.type = 3` (目的地不可达)
                *   `icmp4.code = 1` (主机不可达)
            *   `icmp4_error` 操作应该用于生成 ICMPv4 数据包以响应原始 IP 数据包中的错误。当此操作生成 ICMPv4 数据包时，它还会根据 RFC 1122: 3.2.2 复制 ICMPv4 标头之后的原始 IP 数据报。
            *   先决条件：`ip4`

        *   `icmp6 { action; ... };` `icmp6_error { action; ... };`
            *   暂时将正在处理的 IPv6 数据包替换为 ICMPv6 数据包，并对 ICMPv6 数据包执行每个嵌套操作。`icmp6` 操作之后的操作（如果有）适用于原始的、未修改的数据包。
            *   此操作操作的 ICMPv6 数据包基于正在处理的 IPv6 数据包初始化，如下所示。这些是嵌套操作可能想要更改的默认值。此处未列出的以太网和 IPv6 字段未更改：
                *   `ip.proto = 58` (ICMPv6)
                *   `ip.ttl = 255`
                *   `icmp6.type = 1` (目的地不可达)
                *   `icmp6.code = 1` (管理上禁止)
            *   `icmp6_error` 操作应该用于生成 ICMPv6 数据包以响应原始 IPv6 数据包中的错误。
            *   先决条件：`ip6`

        *   `tcp_reset;`
            *   此操作根据以下伪代码转换当前 TCP 数据包：
                ```
                if (tcp.ack) {
                        tcp.seq = tcp.ack;
                } else {
                        tcp.ack = tcp.seq + length(tcp.payload);
                        tcp.seq = 0;
                }
                tcp.flags = RST;
                ```
            *   然后，该操作丢弃所有 TCP 选项和有效负载数据，并更新 TCP 校验和。IP ttl 设置为 255。
            *   先决条件：`tcp`

        *   `reject { action; ... };`
            *   如果原始数据包是 IPv4 或 IPv6 TCP 数据包，它将其替换为 IPv4 或 IPv6 TCP RST 数据包并执行内部操作。否则，它将其替换为 ICMPv4 或 ICMPv6 数据包并执行内部操作。
            *   内部操作不应尝试交换 eth 源与 eth 目的地以及 IP 源与 IP 目的地，因为此操作隐式地执行了该操作。

        *   `trigger_event;`
            *   此操作用于允许 `ovs-vswitchd` 报告 CMS 相关事件，将其写入 `Controller_Event` 表。可以将仪表与每个事件相关联，以免在重负载下使 `pinctrl` 线程过载；每个仪表通过定义的命名约定进行标识。支持的事件：
                *   `empty_lb_backends`。如果接收到的数据包发往没有配置后端目的地的负载均衡器 VIP，则会引发此事件。对于此事件，事件信息包括负载均衡器 VIP、负载均衡器 UUID 和传输协议。关联的仪表：`event-elb`

        *   `igmp;`
            *   此操作将数据包发送到 `ovn-controller` 进行多播侦听。
            *   先决条件：`igmp`

        *   `bind_vport(V, P);`
            *   参数：类型为 `virtual` 的逻辑端口字符串字段 `V`，逻辑端口字符串字段 `P`。
            *   绑定虚拟逻辑端口 `V` 并设置 `Port_Binding` 表的 `chassis` 列和 `virtual_parent`。`virtual_parent` 设置为 `P`。

        *   `handle_svc_check(P);`
            *   参数：逻辑端口字符串字段 `P`。
            *   处理从逻辑端口 `P` 的 VIF 接收的服务监视器回复。`ovn-controller` 定期发送在 `Service_Monitor` 表中配置的服务的服务监视器数据包，此操作更新这些服务的状态。
            *   例如：`handle_svc_check(inport);`

        *   `handle_dhcpv6_reply;`
            *   处理来自 IPv6 委托服务器的 DHCPv6 前缀委托通告/回复。对于从委托服务器接收的每个前缀，`ovn-controller` 将在 `options` 表中添加一个条目 `ipv6_ra_pd_list`。

        *   `R = select(N1[=W1], N2[=W2], ...);` `R = select(values=(N1[=W1], N2[=W2], ...); hash_fields="field1,field2,...");`
            *   参数：整数 `N1`, `N2`...，带有可选权重 `W1`, `W2`, ...
            *   结果：存储到逻辑字段或子字段 `R`。
            *   从整数列表 `N1`, `N2`... 中选择（每个都在 0 ~ 65535 范围内），并将所选整数存储在字段 `R` 中。必须列出 2 个或更多整数，每个整数具有可选权重，即 1 ~ 65535 范围内的整数。如果未指定权重，则默认为 100。选择方法基于数据包头的 5 元组哈希。
            *   默认情况下，`dp_hash` 用作 OpenFlow 组选择方法，但如果指定了 `values` 和 `hash_fields`，则使用 `hash` 作为选择方法，并且列出的字段用作哈希字段。
            *   处理会自动移动到下一个表，就像指定了 `next;` 一样。当有多个操作时，`select` 操作必须作为逻辑流的最后一个操作放置（放在 `select` 之后的操作将不生效）。
            *   例如：`reg8[16..31] = select(1=20, 2=30, 3=50);`
            *   例如：`reg8[16..31] = select(values=(1=20, 2=30, 3=50); hash_fields="ip_proto,src_ip,dst_ip");`

        *   `handle_dhcpv6_reply;`
            *   此操作用于解析来自 IPv6 委托路由器的 DHCPv6 回复并管理 IPv6 前缀委托状态机。

        *   `R = chk_lb_hairpin();`
            *   此操作检查正在考虑的数据包是否发往负载均衡器 VIP 并且是发夹的，即负载平衡后目标 IP 匹配源 IP。如果是这样，则将 1 位目标寄存器 `R` 设置为 1。

        *   `R = chk_lb_hairpin_reply();`
            *   此操作检查正在考虑的数据包是否来自负载均衡器 VIP 的后端 IP 之一，并且目标 IP 是负载均衡器 VIP。如果是这样，则将 1 位目标寄存器 `R` 设置为 1。

        *   `R = ct_snat_to_vip;`
            *   如果原始目标 IP 是负载均衡器 VIP，则此操作通过 SNAT 区域发送数据包，将数据包的源 IP 地址更改为负载均衡器 VIP，并提交连接。此操作仅在发夹流量（即如果操作 `chk_lb_hairpin` 返回成功）时才成功应用。此操作不带任何参数，它在内部确定 SNAT IP。数据包不会自动发送到下一个表。调用者必须在此操作之后显式执行 `next;` 操作以将数据包推进到下一阶段。

        *   `R = check_in_port_sec();`
            *   此操作检查正在考虑的数据包是否通过 `inport` 端口安全检查。如果数据包未通过端口安全检查，则将 1 存储在目标寄存器 `R` 中。否则存储 0。要检查的端口安全值是从 `inport` 逻辑端口检索的。
            *   此操作应在入口逻辑交换机管道中使用。
            *   例如：`reg8[0..7] = check_in_port_sec();`

        *   `R = check_out_port_sec();`
            *   此操作检查正在考虑的数据包是否通过 `outport` 端口安全检查。如果数据包未通过端口安全检查，则将 1 存储在目标寄存器 `R` 中。否则存储 0。要检查的端口安全值是从 `outport` 逻辑端口检索的。
            *   此操作应在出口逻辑交换机管道中使用。
            *   例如：`reg8[0..7] = check_out_port_sec();`

        *   `commit_ecmp_nh(ipv6);`
            *   参数：IPv4/IPv6 流量。
            *   此操作转换为 openflow "learn" 操作，在表 76 和 77 中插入两个新流。
            *   在表 76 中匹配 5 元组和预期的下一跳 mac 地址：`nw_src=ip0, nw_dst=ip1, ip_proto, tp_src=l4_port0, tp_dst=l4_port1, dl_src=ethaddr` 并设置 `reg9[5]`。
            *   在表 77 中匹配 5 元组：`nw_src=ip1, nw_dst=ip0, ip_proto, tp_src=l4_port1, tp_dst=l4_port0` 并将 `reg9[5]` 设置为 1。
            *   如果数据包通过 ECMP 路由到达或如果它是通过 ECMP 路由路由的，则应用此操作。

        *   `R = check_ecmp_nh_mac();`
            *   此操作检查正在考虑的数据包是否匹配表 76 中的任何流。如果是这样，则将 1 位目标寄存器 `R` 设置为 1。

        *   `R = check_ecmp_nh();`
            *   此操作检查正在考虑的数据包是否匹配表 77 中的任何流。如果是这样，则将 1 位目标寄存器 `R` 设置为 1。

        *   `commit_lb_aff(vip, backend, proto, timeout);`
            *   参数：负载均衡器虚拟 ip:port `vip`，负载均衡器后端 ip:port `backend`，负载均衡器协议 `proto`，亲和性超时 `timeout`。
            *   此操作转换为 openflow "learn" 操作，在表 78 中插入一个新流。
            *   在表 78 中匹配 4 元组：`nw_src=ip client, nw_dst=vip ip, ip_proto, tp_dst=vip port` 并将 `reg9[6]` 设置为 1，将 `reg4` 和 `reg8` 分别设置为后端 ip 和端口。对于 IPv6，寄存器 `xxreg1` 用于存储后端 ip。
            *   此操作适用于配置了亲和性超时的特定负载均衡器接收的新连接。

        *   `R = chk_lb_aff();`
            *   此操作检查正在考虑的数据包是否匹配表 78 中的任何流。如果是这样，则将 1 位目标寄存器 `R` 设置为 1。

        *   `R = ct_nw_dst();`
            *   此操作检查数据包是否被跟踪，并将 conntrack 原始目标 IPv4 地址存储在 32 位大小的寄存器 `R` 中。

        *   `R = ct_ip6_dst();`
            *   此操作检查数据包是否被跟踪，并将 conntrack 原始目标 IPv6 地址存储在 128 位大小的寄存器 `R` 中。

        *   `R = ct_tp_dst();`
            *   此操作检查数据包是否被跟踪，并将 conntrack 原始 L4 目标端口存储在 16 位大小的寄存器 `R` 中。

        *   `R = ct_state_save();`
            *   此操作检查数据包是否被跟踪，并将 conntrack 原始状态存储在 8 位寄存器 `R` 中。

        *   `sample(probability=packets, ...)`
            *   此操作导致匹配的流量使用 IPFIX 协议进行采样。有关 OVS 中每流 IPFIX 采样如何工作的更多信息，请参阅 `ovs-actions(7)` 和 `ovs-vswitchd.conf.db(5)`。
            *   为了在 IPFIX 收集器接收到每个采样数据包时可靠地识别它，此操作设置 `ObservationDomainID` 和 `ObservationPointID` IPFIX 字段的内容（请参阅下面的参数说明）。
            *   支持以下键值参数：
                *   `probability=packets`
                    *   65535 个数据包中的采样数据包数。它必须大于或等于 1。
                *   `collector_set=id`
                    *   将采样数据包发送到的样本收集器的无符号 32 位整数标识符。它必须匹配 OVS 中 `Flow_Sample_Collector_Set` 表中配置的值。默认为 0。
                *   `obs_domain=id`
                    *   标识采样应用程序的无符号 8 位整数。它将被放置在 IPFIX 样本的 `ObservationDomainID` 字段的 8 个最高有效位中。24 个较低有效位将自动用数据路径键填充。默认为 0。
                *   `obs_point=id`
                    *   用作 `ObsservationPointID` 的无符号 32 位整数或字符串 `@cookie`，表示应使用 `Logical_Flow` 的 UUID 的前 32 位。

        *   `mac_cache_use;`
            *   此操作重新提交到相应的表，该表更新 MAC 缓存的使用统计信息。

        *   `R = dhcp_relay_req_chk(relay-ip, server-ip);`
            *   参数：逻辑路由器端口 IP `relay-ip`，DHCP 服务器 IP `server-ip`。
            *   结果：存储到 1 位子字段 `R`。
            *   此操作在发起 DHCP 请求（DHCPDISCOVER 或 DHCPREQUEST）的源节点上执行。
            *   当此操作成功应用于 DHCP 请求数据包时，它使用 `relay-ip` 更新 DHCP 数据包中的 GIADDR，并将 1 存储在 `R` 中。
            *   当此操作未能应用于数据包时，它保持数据包不变并将 0 存储在 `R` 中。

        *   `R = dhcp_relay_resp_chk(relay-ip, server-ip);`
            *   参数：逻辑路由器端口 IP `relay-ip`，DHCP 服务器 IP `server-ip`。
            *   结果：存储到 1 位子字段 `R`。
            *   此操作在处理来自 DHCP 服务器的 DHCP 响应（DHCPOFFER, DHCPACK）的第一个节点（重定向机箱节点）上执行。
            *   当此操作成功应用于 DHCP 响应数据包时，它更新数据包中的目标 MAC 和目标 IP，并将 1 存储在 `R` 中。`relay-ip` 和 `server-ip` 用于验证 DHCP 响应数据包中的 GIADDR 和 SERVER-ID。
            *   当此操作未能应用于数据包时，它保持数据包不变并将 0 存储在 `R` 中。

        *   `mirror(P);`
            *   参数：类型为 `mirror` 的逻辑端口字符串字段 `P`。
            *   当使用 lport 镜像时，它克隆数据包并通过镜像端口 `P` 将其输出到本地/远程机箱到目标端口。

*   `tags`: 字符串对的映射
    *   提供附加信息以帮助 `ovn-controller` 处理逻辑流的键值对。以下是 `ovn-controller` 使用的标签。

    *   `in_out_port`
        *   在逻辑流的 "match" 列中，如果逻辑端口 P 与 "inport" 比较且逻辑流位于逻辑交换机入口管道上，或者如果 P 与 "outport" 比较且逻辑流位于逻辑交换机出口管道上，并且表达式使用运算符 `&&` 与其他表达式（如果有）组合，则端口 P 应作为此标签中的值添加。如果有多个满足此标准的逻辑端口，则可以添加其中之一。`ovn-controller` 使用此信息来跳过解析机箱上不需要的流。未能添加标签会影响效率，而添加错误的值会影响正确性。

*   `controller_meter`: 可选字符串
    *   `Meter` 表中仪表的名称，用于逻辑流可能发送到 `ovn-controller` 的所有数据包。

*   `flow_desc`: 可选字符串
    *   流的人类可读解释，这是可选的，用于为给定流提供上下文。

*   `external_ids : stage-name`: 可选字符串
    *   管道中此流阶段的人类可读名称。

*   `external_ids : stage-hint`: 可选字符串，包含 UUID
    *   导致创建此逻辑流的 `OVN_Northbound` 记录的 UUID。

*   `external_ids : source`: 可选字符串
    *   将此流添加到管道的代码的源文件和行号。

*   通用列:
    *   `external_ids`: 字符串对的映射
        *   有关详细信息，请参阅本文档开头的“通用列”。

# Logical_DP_Group 表

此表中的每一行表示 `Logical_Flow` 表中 `logical_dp_group` 列引用的逻辑数据路径组。

## 汇总

*   `datapaths`
    *   `Datapath_Binding` 的弱引用集合

## 详细信息

*   `datapaths`: `Datapath_Binding` 的弱引用集合
    *   `Datapath_Binding` 条目列表。

# Multicast_Group 表

此表中的行定义逻辑端口的多播组。多播组允许通过隧道传输到管理程序的单个数据包被传递到该管理程序上的多个 VM，从而更有效地使用带宽。

此表中的每一行定义了一个在 `datapath` 内编号为 `tunnel_key` 的逻辑多播组，其逻辑端口在 `ports` 列中列出。

## 汇总

*   `datapath`
    *   `Datapath_Binding`
*   `tunnel_key`
    *   整数，范围 32,768 到 65,535
*   `name`
    *   字符串
*   `ports`
    *   `Port_Binding` 的弱引用集合

## 详细信息

*   `datapath`: `Datapath_Binding`
    *   多播组所在的逻辑数据路径。

*   `tunnel_key`: 整数，范围 32,768 到 65,535
    *   用于在隧道封装中指定此逻辑出口端口的值。索引强制键在数据路径内唯一。不寻常的范围确保多播组 ID 不会与逻辑端口 ID 重叠。

*   `name`: 字符串
    *   逻辑多播组的名称。索引强制名称在数据路径内唯一。入口管道中的逻辑流可以像单个逻辑端口一样输出到组，方法是将组的名称分配给 `outport` 并执行 `output` 操作。
    *   多播组名称和逻辑端口名称共享单个命名空间，因此不应重叠（但数据库模式无法强制执行此操作）。为了尽量避免冲突，`ovn-northd` 使用以 `_MC_` 开头的名称。

*   `ports`: `Port_Binding` 的弱引用集合
    *   包含在多播组中的逻辑端口。所有这些端口必须在数据路径逻辑数据路径中（但数据库模式无法强制执行此操作）。

# Mirror 表

此表中的每一行代表一个可用于端口镜像的镜像。`Port_Binding` 表中的 `mirror_rules` 列引用了这些镜像。

## 汇总

*   `name`
    *   字符串 (在表中必须唯一)
*   `filter`
    *   字符串，`both`, `from-lport`, 或 `to-lport` 之一
*   `sink`
    *   字符串
*   `type`
    *   字符串，`erspan`, `gre`, `local`, 或 `lport` 之一
*   `index`
    *   整数
*   `external_ids`
    *   字符串对的映射

## 详细信息

*   `name`: 字符串 (在表中必须唯一)
    *   代表镜像的名称。

*   `filter`: 字符串，`both`, `from-lport`, 或 `to-lport` 之一
    *   此字段的值表示镜像的选择标准。`to-lport` 镜像进入逻辑端口的数据包。`from-lport` 镜像从逻辑端口发出的数据包。`both` 镜像两个方向。

*   `sink`: 字符串
    *   此字段的值表示镜像的目的地/接收器。如果类型是 `gre` 或 `erspan`，该值指示隧道远程 IP（IPv4 或 IPv6）。对于 `local` 类型，此字段定义 OVS 集成网桥上的本地接口以用作镜像目的地。该接口必须拥有与此字符串匹配的 `external-ids:mirror-id`。

*   `type`: 字符串，`erspan`, `gre`, `local`, 或 `lport` 之一
    *   此字段的值指定镜像类型 - `gre`, `erspan`, `local` 或 `lport`。

*   `index`: 整数
    *   此字段的值表示隧道 ID。如果配置的隧道类型是 `gre`，此字段表示 GRE 键值，如果配置的隧道类型是 `erspan`，它表示 `erspan_idx` 值。如果类型是 `local`，则忽略它。

*   `external_ids`: 字符串对的映射
    *   请参阅本文档开头的“外部 ID”。

# Meter 表

此表中的每一行代表一个可用于 QoS 或速率限制的仪表。

## 汇总

*   `name`
    *   字符串 (在表中必须唯一)
*   `unit`
    *   字符串，`kbps` 或 `pktps`
*   `bands`
    *   1 个或多个 `Meter_Band` 的集合

## 详细信息

*   `name`: 字符串 (在表中必须唯一)
    *   此仪表的名称。
    *   以 "__"（两个下划线）开头的名称保留供 OVN 内部使用，不应手动添加。

*   `unit`: 字符串，`kbps` 或 `pktps`
    *   `bands` 条目中 `rate` 和 `burst_rate` 参数的单位。`kbps` 指定每秒千比特，`pktps` 指定每秒数据包。

*   `bands`: 1 个或多个 `Meter_Band` 的集合
    *   与此仪表关联的频带。每个频带指定一个速率，高于该速率时，频带将采取操作 `action`。如果超过多个频带的速率，则选择超过频带中速率最高的频带。

# Meter_Band 表

此表中的每一行代表一个仪表频带，它指定了高于该速率时应应用配置的操作。`Meter` 表中的 `bands` 列引用了这些频带。

## 汇总

*   `action`
    *   字符串，必须是 `drop`
*   `rate`
    *   整数，范围 1 到 4,294,967,295
*   `burst_size`
    *   整数，范围 0 到 4,294,967,295

## 详细信息

*   `action`: 字符串，必须是 `drop`
    *   此频带匹配时执行的操作。唯一支持的操作是 `drop`。

*   `rate`: 整数，范围 1 到 4,294,967,295
    *   此频带的速率限制，以每秒千比特或每秒比特为单位，具体取决于父 `Meter` 条目的 `unit` 列指定的是 `kbps` 还是 `pktps`。

*   `burst_size`: 整数，范围 0 到 4,294,967,295
    *   频带允许的最大突发，以千比特或数据包为单位，具体取决于在父 `Meter` 条目的 `unit` 列中选择的是 `kbps` 还是 `pktps`。如果大小为零，交换机可以根据其配置自由选择一些合理的值。
