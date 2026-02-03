# 负载均衡 (Load Balancer)

## Load_Balancer 表

定义 L3/L4 负载均衡器。

### 核心列

- **name**: 负载均衡器名称。
- **vips**: 虚拟 IP (VIP) 到后端 IP 的映射字典。
  - Key: `VIP:Port` (如 `"10.0.0.100:80"`)。
  - Value: 后端列表 (如 `"192.168.1.2:8080,192.168.1.3:8080"`)。
- **protocol**: `tcp`, `udp`, or `sctp`.
- **health_check**: 引用 `Load_Balancer_Health_Check` 表。

## Load_Balancer_Health_Check 表

配置后端健康检查。

- **vip**: 关联的 VIP。
- **options**:
  - `interval`: 检查间隔。
  - `timeout`: 超时时间。
  - `success_count`: 判定健康的连续成功次数。
  - `failure_count`: 判定故障的连续失败次数。

## Load_Balancer_VIP 表 (Newer Schema)

在新版本的 OVN 中，VIP 配置被拆分到独立的表中以支持更复杂的配置（如每个 VIP 单独的会话保持设置）。
