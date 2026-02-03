# 全局配置 (Global Configuration)

## NB_Global 表

`NB_Global` 表包含整个 OVN 北向数据库的全局配置。该表始终只有一行记录。

### 核心列 (Columns)

- **name**: 字符串。代表该 OVN 部署的名称。
- **nb_cfg**: 整数。配置序列号，CMS 每次修改配置时递增此值，用于同步状态。
- **sb_cfg**: 整数。指示 Southbound DB 已确认并处理到的配置版本。
- **hv_cfg**: 整数。指示所有 Hypervisor 已确认并处理到的配置版本。
- **options**: 键值对映射 (String: String)。
  - `mac_prefix`: 逻辑地址自动生成时使用的 MAC 前缀。
  - `northd_probe_interval`: ovn-northd 连接探测间隔。
  - `debug`: 调试选项。

## Connection 表

配置 ovsdb-server 接受客户端连接的方式。

- **target**: 连接目标（如 `ptcp:6641:0.0.0.0`）。
- **inactivity_probe**: 探活间隔。

## SSL 表

配置 SSL/TLS 连接所需的证书和密钥路径。

- **private_key**: 私钥文件路径。
- **certificate**: 证书文件路径。
- **ca_cert**: CA 证书文件路径。
