# OVN 控制器 (ovn-controller) 概览与选项

## 简介 (Synopsis)

```bash
ovn-controller [options] [ovs-database]
```

## 描述 (Description)

`ovn-controller` 是 OVN (Open Virtual Network) 的本地控制器守护进程。它运行在 OVN 部署中的每个 Hypervisor (宿主机) 和软件网关上。

`ovn-controller` 的主要职责是：
1.  通过 OVSDB 协议连接到 **OVN Southbound 数据库** (`ovn-sb`)。
2.  通过 OVSDB 协议连接到本地的 **Open vSwitch 数据库** (`ovs-vswitchd.conf.db`)。
3.  通过 OpenFlow 协议连接到本地的 **`ovs-vswitchd`**。

每个 `ovn-controller` 实例都是独立的，其向下的连接（到本地 OVS）是机器本地的，不经过物理网络。

## 选项 (Options)

### 守护进程选项 (Daemon Options)

*   `--pidfile[=pidfile]`
    创建进程 ID 文件 (默认: `program.pid`)。
*   `--overwrite-pidfile`
    如果 pidfile 已存在且被锁定，强制覆盖并启动。
*   `--detach`
    在后台运行。
*   `--monitor`
    创建监控进程以在守护进程崩溃时自动重启它。
*   `--no-chdir`
    后台运行时不更改当前工作目录（默认为更改到根目录 `/`）。
*   `--user=user:group`
    以指定用户和组身份运行，降低权限。

### 日志选项 (Logging Options)

*   `-v[spec]`, `--verbose=[spec]`
    设置日志级别。例如 `-vconsole:emer` 仅在控制台显示紧急错误。
*   `--log-file[=file]`
    启用日志文件记录。默认位置通常为 `/usr/local/var/log/ovn/ovn-controller.log`。
*   `--syslog-target=host:port`
    将 syslog 消息发送到指定的 UDP 服务器。
*   `--syslog-method=method`
    指定 syslog 发送方法 (`libc`, `unix:file`, `udp:ip:port`, `null`)。

### PKI 选项 (PKI Options)

用于配置 SSL/TLS 连接到 Northbound 和 Southbound 数据库。

*   `-p privkey.pem`, `--private-key=privkey.pem`
    私钥文件。
*   `-c cert.pem`, `--certificate=cert.pem`
    证书文件。
*   `-C cacert.pem`, `--ca-cert=cacert.pem`
    CA 证书文件，用于验证对端。
*   `--ssl-server-name=servername`
    指定 TLS Server Name Indication (SNI) 的服务器名称。
*   `--bootstrap-ca-cert=cacert.pem`
    如果文件不存在，则在首次连接时获取 CA 证书并保存（有中间人攻击风险，用于初始引导）。

### 其他选项 (Other Options)

*   `--unixctl=socket`
    设置运行时管理命令的控制 socket 名称。
*   `-h`, `--help`
    显示帮助信息。
*   `-V`, `--version`
    显示版本信息。
