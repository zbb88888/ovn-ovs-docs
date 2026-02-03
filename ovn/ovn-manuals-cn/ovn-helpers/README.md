# 辅助与调试工具 (Helpers & Debug Tools)

本目录包含 OVN 生态系统中用于服务管理、故障排查和调试的辅助工具文档。

## 文档索引

1. **[ovn-ctl.md](./ovn-ctl.md)**
   - `ovn-ctl`: OVN 服务的启动、停止与状态管理脚本。
   - 包含 DB 启动、Daemon 启动、HA 切换命令。

2. **[ovn-detrace.md](./ovn-detrace.md)**
   - `ovn-detrace`: 追踪日志解析工具。
   - 作用：将 `ovs-appctl ofproto/trace` 输出的 OpenFlow cookie 反向解析为 OVN 逻辑流和北向配置（如 ACL）。

3. **[ovn-debug.md](./ovn-debug.md)**
   - `ovn-debug`: 开发者调试工具。
   - 作用：查询逻辑流阶段名称 (Stage Name) 与数字表 ID (Table ID) 的映射关系。
