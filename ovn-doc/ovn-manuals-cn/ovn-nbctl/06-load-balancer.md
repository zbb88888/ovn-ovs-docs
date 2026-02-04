# OVN 负载均衡命令 (Load Balancer Commands)

*   `[--may-exist | --add-duplicate | --reject | --event | --template] lb-add lb vip ips [protocol]`
    创建一个名为 `lb` 的新负载均衡器，或向现有 `lb` 添加 VIP。
    *   `vip`: 虚拟 IP 地址（可选端口）。例如 `192.168.1.5:8080`。
    *   `ips`: 逗号分隔的后端 IP 端点列表。例如 `10.0.0.1:8080,10.0.0.2:8080`。
    *   `protocol`: `tcp`, `udp`, `sctp`。如果 VIP 包含端口且未指定协议，默认为 `tcp`。
    *   `--reject`: 如果没有活动后端，发送 TCP RST 或 ICMP Port Unreachable。
    *   `--event`: 接收流量时在 `Controller_Event` 表中报告事件。
    *   `--template`: 创建模板化负载均衡器，使用 Chassis_Template_Var。

*   `[--if-exists] lb-del lb [vip]`
    删除 `lb` 或从 `lb` 中删除 `vip`。

*   `lb-list [lb]`
    列出负载均衡器。

*   `[--may-exist] ls-lb-add switch lb`
    将指定的 `lb` 添加到逻辑交换机 `switch`。

*   `[--if-exists] ls-lb-del switch [lb]`
    从逻辑交换机 `switch` 中移除 `lb`。

*   `ls-lb-list switch`
    列出给定 `switch` 的负载均衡器。

*   `[--may-exist] lr-lb-add router lb`
    将指定的 `lb` 添加到逻辑路由器 `router`。

*   `[--if-exists] lr-lb-del router [lb]`
    从逻辑路由器 `router` 中移除 `lb`。

*   `lr-lb-list router`
    列出给定 `router` 的负载均衡器。
