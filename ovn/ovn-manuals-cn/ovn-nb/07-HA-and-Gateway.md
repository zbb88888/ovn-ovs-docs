# 高可用与网关 (HA and Gateway)

## HA_Chassis_Group 表

定义一组 Chassis，用于提供高可用服务（如分布式网关端口的主备切换）。

- **name**: 组名称。
- **ha_chassis**: 指向 `HA_Chassis` 表的引用列表。

## HA_Chassis 表

HA 组中的单个成员。

- **chassis_name**: 物理 Chassis 名称。
- **priority**: 优先级（数字越大优先级越高）。优先级最高的在线 Chassis 将成为 Master。

## Gateway_Chassis 表 (Legacy)

旧版本的网关高可用配置方式，现在推荐使用 `HA_Chassis_Group`。
用于将逻辑路由器端口绑定到特定的 Chassis 列表。

## BFD (Bidirectional Forwarding Detection) 表

双向转发检测，用于快速检测 Chassis 之间的链路故障。

- **logical_port**: 关联的逻辑端口。
- **dst_ip**: 目标 IP。
- **status**: BFD 会话状态（Up, Down, Admin Down）。
- **min_rx** / **min_tx**: 最小接收/发送间隔。
