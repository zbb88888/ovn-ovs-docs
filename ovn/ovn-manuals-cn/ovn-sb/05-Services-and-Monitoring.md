# 服务与监控 (Services and Monitoring)

## Service_Monitor 表

用于监控负载均衡器后端的健康状态。

- **ip**: 后端 IP。
- **port**: 后端端口。
- **protocol**: `tcp`, `udp`。
- **status**: `online`, `offline`。

## Controller_Event 表

`ovn-controller` 用于向 CMS 报告特定事件的机制。

- **event_type**: 事件类型（如 `empty_lb_backends`）。
- **event_info**: 详细信息。
- **chassis**: 报告事件的 Chassis。

## BFD (Bidirectional Forwarding Detection) 表

在 SB DB 中，BFD 表记录了 BFD 会话的运行状态。

- **src_ip** / **dst_ip**: 源/目的 IP。
- **status**: 会话状态。
- **detect_mult**: 检测倍数。

## DHCP_Options / DNS

这些表是从 NB DB 复制过来的，用于让 `ovn-controller` 在本地处理 DHCP 和 DNS 请求，而无需查询 NB DB。
