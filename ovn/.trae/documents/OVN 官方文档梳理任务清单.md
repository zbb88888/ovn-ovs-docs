根据您的要求，我整理了基于 [OVN 官方文档库](https://www.ovn.org/support/dist-docs/) 的文档梳理任务清单。

我们将采用与 `ovn-architecture(7)` 相同的“拆分梳理 + 中文对照”模式，为每个组件创建独立的目录（或在统一目录下分类），并将长文档拆分为易读的模块。

以下是待梳理的文档列表，已按功能模块分类。请确认此任务范围：

### 1. 核心架构 (Core Architecture)
- [ ] **ovn-architecture(7)** (已完成初步梳理，可继续完善)
  - *URL*: `https://www.ovn.org/support/dist-docs/ovn-architecture.7.txt`
  - *说明*: OVN 总体架构、流程与资源模型。

### 2. 数据库与模式 (Databases & Schemas)
- [ ] **ovn-nb(5)** (Northbound DB Schema)
  - *URL*: `https://www.ovn.org/support/dist-docs/ovn-nb.5.txt`
  - *说明*: 北向数据库模式定义，包含 Logical Switch, Router, LB 等核心表结构。
- [ ] **ovn-sb(5)** (Southbound DB Schema)
  - *URL*: `https://www.ovn.org/support/dist-docs/ovn-sb.5.txt`
  - *说明*: 南向数据库模式定义，包含 Chassis, Logical Flow, Port Binding 等物理/运行时表结构。

### 3. 守护进程 (Daemons)
- [ ] **ovn-northd(8)**
  - *URL*: `https://www.ovn.org/support/dist-docs/ovn-northd.8.txt`
  - *说明*: 集中式控制器，负责将 NB 配置翻译为 SB 流表。
- [ ] **ovn-controller(8)**
  - *URL*: `https://www.ovn.org/support/dist-docs/ovn-controller.8.txt`
  - *说明*: 运行在每个计算节点的代理，负责 OpenFlow 下发与物理绑定。
- [ ] **ovn-controller-vtep(8)**
  - *URL*: `https://www.ovn.org/support/dist-docs/ovn-controller-vtep.8.txt`
  - *说明*: 用于管理硬件 VTEP 网关的控制器。

### 4. 命令行工具 (CLI Utilities)
- [ ] **ovn-nbctl(8)**
  - *URL*: `https://www.ovn.org/support/dist-docs/ovn-nbctl.8.txt`
  - *说明*: 管理北向数据库的命令行工具。
- [ ] **ovn-sbctl(8)**
  - *URL*: `https://www.ovn.org/support/dist-docs/ovn-sbctl.8.txt`
  - *说明*: 管理南向数据库的命令行工具。
- [ ] **ovn-trace(8)**
  - *URL*: `https://www.ovn.org/support/dist-docs/ovn-trace.8.txt`
  - *说明*: 极其重要的调试工具，用于模拟和追踪逻辑网络报文路径。
- [ ] **ovn-appctl(8)**
  - *URL*: `https://www.ovn.org/support/dist-docs/ovn-appctl.8.txt`
  - *说明*: 向运行中的 OVN 守护进程发送运行时命令。

### 5. 互联架构 (OVN Interconnection)
- [ ] **ovn-ic(8)** (Interconnection Daemon)
- [ ] **ovn-ic-nb(5)** (IC Northbound DB)
- [ ] **ovn-ic-sb(5)** (IC Southbound DB)
- [ ] **ovn-ic-nbctl(8)**
- [ ] **ovn-ic-sbctl(8)**
  - *说明*: 用于多 OVN 集群互联（AZ 互联）的相关组件。

### 6. 辅助与调试 (Helpers & Debug)
- [ ] **ovn-ctl(8)**: 服务启动与管理脚本。
- [ ] **ovn-detrace(1)**: 结合 ovn-trace 输出解析 OpenFlow cookie。
- [ ] **ovn-debug(8)**: 调试辅助工具。

---

**确认后，我将执行以下操作：**
1.  创建一个总目录 `ovn-manuals-cn/`。
2.  按照上述列表，逐个下载/读取 plain text 内容。
3.  为每个组件创建子目录（如 `ovn-nb/`, `ovn-controller/`）。
4.  将英文文档按资源/主题拆分并梳理为中文 Markdown。
