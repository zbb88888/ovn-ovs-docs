# OVN 控制器运行时管理命令 (Runtime Management Commands)

`ovn-appctl` 可以向正在运行的 `ovn-controller` 进程发送命令。

## 通用命令

*   `exit`
    使 `ovn-controller` 优雅退出。它会清理 Southbound 数据库中的 Chassis 记录。

*   `connection-status`
    显示当前与 OVN Southbound 数据库的连接状态。

*   `recompute`
    触发 `ovn-controller` 基于 Southbound 数据库和本地 OVS 数据库的内容进行一次全量计算。
    *   通常仅在遇到 Bug 导致增量处理状态不一致时使用。全量计算消耗 CPU 资源。

## 调试与诊断

*   `inject-pkt microflow`
    将微流 (microflow) 注入到连接的 Open vSwitch 实例中，用于模拟和追踪数据包转发。
    *   `microflow`: 描述数据包的 OVN 逻辑表达式（如 `ip4.src=10.0.0.1`）。必须包含 `inport` 参数。

*   `debug/delay-nb-cfg-report seconds`
    延迟 `ovn-controller` 向 Southbound 数据库报告 `nb_cfg` 更新的时间。
    *   用于在大规模环境中测试端到端延迟（配合 `ovn-nbctl --wait=hv`）。

*   `sb-cluster-state-reset`
    重置 Southbound 数据库集群状态的本地索引。当数据库被删除重建导致本地索引失效时使用。

## 状态查看

*   `ct-zone-list`
    列出每个本地逻辑端口及其分配的连接跟踪区域 (CT Zone) ID。

*   `meter-table-list`
    列出每个计量器 (Meter) 表项及其本地 Meter ID。

*   `group-table-list`
    列出每个组 (Group) 表项及其本地 Group ID。

## 引擎与缓存统计

*   `lflow-cache/flush`
    清空逻辑流缓存。

*   `lflow-cache/show-stats`
    显示逻辑流缓存的统计信息（启用状态、条目数等）。

*   `inc-engine/show-stats [engine_node_name] [counter_name]`
    显示 `ovn-controller` 增量处理引擎 (Incremental Processing Engine) 的计数器。
    *   计数器包括: `recompute` (全量计算次数), `compute` (增量计算次数), `cancel` (取消次数)。
    *   可以指定特定的引擎节点（如 `flow_output`, `runtime_data` 等）进行查看。

*   `inc-engine/clear-stats`
    重置引擎计数器。
