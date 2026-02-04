# ovn-northd(8)

## 名称
ovn-northd - 开放虚拟网络 (Open Virtual Network) 中央控制守护进程

## 概要
`ovn-northd [options]`

## 描述
`ovn-northd` 是一个集中式守护进程，负责将高级 OVN 配置转换为可由 `ovn-controller` 等守护进程使用的逻辑配置。它将来自 OVN 北向数据库（参见 `ovn-nb(5)`）的传统网络概念方面的逻辑网络配置转换为其下方的 OVN 南向数据库（参见 `ovn-sb(5)`）中的逻辑数据路径流。

## 选项
-   `--ovnnb-db=database`
    包含 OVN 北向数据库的 OVSDB 数据库。如果设置了 `OVN_NB_DB` 环境变量，则使用其值作为默认值。否则，默认为 `unix:/ovnnb_db.sock`。

-   `--ovnsb-db=database`
    包含 OVN 南向数据库的 OVSDB 数据库。如果设置了 `OVN_SB_DB` 环境变量，则使用其值作为默认值。否则，默认为 `unix:/ovnsb_db.sock`。

-   `--dry-run`
    使 `ovn-northd` 以暂停状态启动。在暂停状态下，`ovn-northd` 不会对数据库应用任何更改，尽管它会继续监视它们。有关更多信息，请参阅下面的“运行时管理命令”下的 `pause` 命令。

-   `n-threads N`
    在某些情况下，可能需要在系统上启用并行化以减少延迟（可能会增加 CPU 使用率）。

    此选项将导致 `ovn-northd` 在构建逻辑流时使用 N 个线程，其中 N 在 [2-256] 范围内。如果 N 为 1，则禁用并行化（默认行为）。如果 N 小于 1，则 N 设置为 1，禁用并行化并记录警告。如果 N 大于 256，则 N 设置为 256，启用并行化（使用 256 个线程）并记录警告。

-   上述选项中的 `database` 必须是 `ovsdb(7)` 中描述的 OVSDB 主动或被动连接方法。

### 守护进程选项
-   `--pidfile[=pidfile]`
    导致创建一个文件（默认为 `program.pid`），指示正在运行的进程的 PID。如果未指定 `pidfile` 参数，或者它不以 `/` 开头，则它在 `.` 中创建。

    如果未指定 `--pidfile`，则不创建 pidfile。

-   `--overwrite-pidfile`
    默认情况下，当指定 `--pidfile` 并且指定的 pidfile 已经存在且被正在运行的进程锁定时，守护进程拒绝启动。指定 `--overwrite-pidfile` 以使其覆盖 pidfile。

    当未指定 `--pidfile` 时，此选项无效。

-   `--detach`
    作为后台进程运行此程序。该进程分叉，并在子进程中启动一个新会话，关闭标准文件描述符（这具有禁用向控制台记录日志的副作用），并将其当前目录更改为根目录（除非指定了 `--no-chdir`）。子进程完成其初始化后，父进程退出。

-   `--monitor`
    创建一个额外的进程来监视此程序。如果它因指示编程错误的信号（SIGABRT, SIGALRM, SIGBUS, SIGFPE, SIGILL, SIGPIPE, SIGSEGV, SIGXCPU 或 SIGXFSZ）而死亡，则监视进程将启动它的一个新副本。如果守护进程因其他原因死亡或退出，监视进程将退出。

    此选项通常与 `--detach` 一起使用，但它也可以在没有它的情况下运行。

-   `--no-chdir`
    默认情况下，当指定 `--detach` 时，守护进程在分离后将其当前工作目录更改为根目录。否则，从粗心选择的目录调用守护进程将阻止管理员卸载包含该目录的文件系统。

    指定 `--no-chdir` 会抑制此行为，防止守护进程更改其当前工作目录。这对于收集核心文件很有用，因为将核心转储写入当前工作目录是常见行为，而根目录不是一个好的使用目录。

    当未指定 `--detach` 时，此选项无效。

-   `--no-self-confinement`
    默认情况下，此守护进程将尝试自我限制，仅使用构建时确定的知名目录下的文件。最好坚持此默认行为，不要使用此标志，除非使用其他访问控制来限制守护进程。请注意，与通常从内核空间强制执行的其他访问控制实现（例如 DAC 或 MAC）相比，自我限制是从用户空间守护进程本身施加的，因此不应被视为完整的限制策略，而应被视为附加的安全层。

-   `--user=user:group`
    导致此程序以 `user:group` 中指定的不同用户运行，从而放弃大多数 root 权限。也允许使用简写形式 `user` 和 `:group`，分别假设当前用户或组。只有由 root 用户启动的守护进程才接受此参数。

    在 Linux 上，守护进程将在放弃 root 权限之前被授予 `CAP_IPC_LOCK` 和 `CAP_NET_BIND_SERVICES`。与数据路径交互的守护进程（如 `ovs-vswitchd`）将被授予三个额外的功能，即 `CAP_NET_ADMIN`、`CAP_NET_BROADCAST` 和 `CAP_NET_RAW`。即使新用户是 root，功能更改也会应用。

    在 Windows 上，目前不支持此选项。出于安全原因，指定此选项将导致守护进程无法启动。

### 日志记录选项
-   `-v[spec]`
-   `--verbose=[spec]`
    设置日志级别。如果不带任何 `spec`，则将每个模块和目标的日志级别设置为 `dbg`。否则，`spec` 是由空格、逗号或冒号分隔的单词列表，每个类别最多一个：

    *   有效的模块名称，如 `ovs-appctl(8)` 上的 `vlog/list` 命令所示，将日志级别更改限制为指定的模块。

    *   `syslog`, `console` 或 `file`，分别将日志级别更改限制为仅系统日志、控制台或文件。（如果指定了 `--detach`，守护进程将关闭其标准文件描述符，因此向控制台记录日志将无效。）

        在 Windows 平台上，`syslog` 被接受为一个单词，并且仅在与 `--syslog-target` 选项一起使用时才有用（否则该单词无效）。

    *   `off`, `emer`, `err`, `warn`, `info` 或 `dbg`，用于控制日志级别。给定严重性或更高级别的消息将被记录，较低严重性的消息将被过滤掉。`off` 过滤掉所有消息。有关每个日志级别的定义，请参阅 `ovs-appctl(8)`。

    `spec` 中不区分大小写。

    无论为 `file` 设置了什么日志级别，除非也指定了 `--log-file`（见下文），否则不会发生向文件记录日志。

    为了与旧版本的 OVS 兼容，`any` 被接受为一个单词，但没有效果。

-   `-v`
-   `--verbose`
    设置最大日志详细级别，相当于 `--verbose=dbg`。

-   `-vPATTERN:destination:pattern`
-   `--verbose=PATTERN:destination:pattern`
    将 `destination` 的日志模式设置为 `pattern`。有关 `pattern` 的有效语法的描述，请参阅 `ovs-appctl(8)`。

-   `-vFACILITY:facility`
-   `--verbose=FACILITY:facility`
    设置日志消息的 RFC5424 设施。`facility` 可以是 `kern`, `user`, `mail`, `daemon`, `auth`, `syslog`, `lpr`, `news`, `uucp`, `clock`, `ftp`, `ntp`, `audit`, `alert`, `clock2`, `local0`, `local1`, `local2`, `local3`, `local4`, `local5`, `local6` 或 `local7` 之一。如果未指定此选项，则 `daemon` 用作本地系统 syslog 的默认值，而在向通过 `--syslog-target` 选项提供的目标发送消息时使用 `local0`。

-   `--log-file[=file]`
    启用向文件记录日志。如果指定了 `file`，则将其用作日志文件的确切名称。如果省略 `file`，则使用的默认日志文件名为 `/usr/local/var/log/ovn/program.log`。

-   `--syslog-target=host:port`
    除了系统 syslog 之外，还将 syslog 消息发送到主机的 UDP 端口。主机必须是数字 IP 地址，而不是主机名。

-   `--syslog-method=method`
    指定应如何将 syslog 消息发送到 syslog 守护进程。支持以下形式：

    *   `libc`，使用 libc `syslog()` 函数。使用此选项的缺点是 libc 在实际上通过 `/dev/log` UNIX 域套接字发送到 syslog 守护进程之前，会向每条消息添加固定前缀。

    *   `unix:file`，直接使用 UNIX 域套接字。可以使用此选项指定任意消息格式。但是，rsyslogd 8.9 和更旧版本使用硬编码的解析器函数，这限制了 UNIX 域套接字的使用。如果您想在较旧的 rsyslogd 版本中使用任意消息格式，请改用 UDP 套接字到 localhost IP 地址。

    *   `udp:ip:port`，使用 UDP 套接字。使用此方法，也可以在较旧的 rsyslogd 中使用任意消息格式。当通过 UDP 套接字发送 syslog 消息时，需要考虑额外的预防措施，例如，syslog 守护进程需要配置为监听指定的 UDP 端口，意外的 iptables 规则可能会干扰本地 syslog 流量，并且有一些适用于 UDP 套接字但不适用于 UNIX 域套接字的安全注意事项。

    *   `null`，丢弃所有记录到 syslog 的消息。

    默认值取自 `OVS_SYSLOG_METHOD` 环境变量；如果未设置，则默认为 `libc`。

### PKI 选项
需要 PKI 配置才能使用 SSL/TLS 连接到北向和南向数据库。

-   `-p privkey.pem`
-   `--private-key=privkey.pem`
    指定包含用作传出 SSL/TLS 连接身份的私钥的 PEM 文件。

-   `-c cert.pem`
-   `--certificate=cert.pem`
    指定包含证书的 PEM 文件，该证书证明在 `-p` 或 `--private-key` 上指定的私钥是可信的。证书必须由 SSL/TLS 连接中的对等方将用于验证它的证书颁发机构 (CA) 签名。

-   `-C cacert.pem`
-   `--ca-cert=cacert.pem`
    指定包含 CA 证书的 PEM 文件，用于验证 SSL/TLS 对等方向此程序出示的证书。（这可能与 SSL/TLS 对等方用于验证在 `-c` 或 `--certificate` 上指定的证书的证书相同，或者可能是不同的证书，具体取决于使用的 PKI 设计。）

-   `-C none`
-   `--ca-cert=none`
    禁用验证 SSL/TLS 对等方出示的证书。这会引入安全风险，因为这意味着无法验证证书是否属于已知的受信任主机。

-   `--ssl-server-name=servername`
    指定用于 TLS 服务器名称指示 (SNI) 的服务器名称。默认情况下，连接字符串中的主机名用于 SNI。此选项允许覆盖 SNI 主机名，这在通过代理或服务网格连接时很有用，其中连接端点与预期的服务器名称不同。

### 其他选项
-   `--unixctl=socket`
    设置程序监听运行时管理命令的控制套接字的名称（参见下面的“运行时管理命令”）。如果 `socket` 不以 `/` 开头，则将其解释为相对于 `.`。如果根本不使用 `--unixctl`，则默认套接字为 `/program.pid.ctl`，其中 `pid` 是程序的进程 ID。

    在 Windows 上，使用本地命名管道来监听运行时管理命令。在 `socket` 指向的绝对路径中创建一个文件，或者如果根本不使用 `--unixctl`，则在配置的 `OVS_RUNDIR` 目录中创建一个名为 `program` 的文件。该文件的存在只是为了模仿 Unix 域套接字的行为。

    为 `socket` 指定 `none` 会禁用控制套接字功能。

-   `-h`
-   `--help`
    向控制台打印简短的帮助消息。

-   `-V`
-   `--version`
    向控制台打印版本信息。

## 运行时管理命令
`ovn-appctl` 可以向正在运行的 `ovn-northd` 进程发送命令。目前支持的命令如下所述。

-   `exit`
    导致 `ovn-northd` 优雅地终止。

-   `pause`
    暂停 `ovn-northd`。当它暂停时，`ovn-northd` 照常接收来自北向和南向数据库的更改，但它不发送任何更新。暂停的 `ovn-northd` 还会丢弃数据库锁，这允许任何其他非暂停的 `ovn-northd` 实例接管。

-   `resume`
    恢复 `ovn-northd` 操作以处理北向和南向数据库内容并生成逻辑流。这也将指示 `ovn-northd` 争取 SB DB 上的锁。

-   `is-paused`
    如果 `ovn-northd` 当前已暂停，则返回 "true"，否则返回 "false"。

-   `status`
    打印此服务器的状态。如果 `ovn-northd` 已获取 SB DB 上的 OVSDB 锁，则状态将为 "active"，如果未获取则为 "standby"，或者如果此实例已暂停则为 "paused"。

-   `sb-cluster-state-reset`
    当数据库被销毁并重建时，重置南向数据库集群状态。

    如果集群南向数据库中的所有数据库都从磁盘中删除，则所有数据库的存储索引将重置为零。这将导致 `ovn-northd` 无法读取或写入南向数据库，因为它将始终检测到数据已过时。在这种情况下，运行此命令以便 `ovn-northd` 重置其本地索引，以便它可以再次与南向数据库交互。

-   `nb-cluster-state-reset`
    当数据库被销毁并重建时，重置北向数据库集群状态。

    这执行与 `sb-cluster-state-reset` 相同的任务，除了是针对北向数据库客户端。

-   `set-n-threads N`
    设置用于构建逻辑流的线程数。当 N 在 [2-256] 范围内时，启用并行化。当 N 为 1 时，禁用并行化。当 N 小于 1 或大于 256 时，返回错误。如果 `ovn-northd` 无法启动并行化（例如无法设置信号量），则禁用并行化并返回错误。

-   `get-n-threads`
    返回用于构建逻辑流的线程数。

-   `inc-engine/show-stats`
    显示 `ovn-northd` 引擎计数器。为每个引擎节点添加了以下计数器：
    *   `recompute`
    *   `compute`
    *   `abort`

-   `inc-engine/show-stats engine_node_name counter_name`
    显示指定 `engine_node_name` 的 `ovn-northd` 引擎计数器。`counter_name` 是可选的，可以是 `recompute`, `compute` 或 `abort` 之一。

-   `inc-engine/clear-stats`
    重置 `ovn-northd` 引擎计数器。

## 高可用性的主备模式 (Active-Standby)
您可以在 OVN 部署中多次运行 `ovn-northd`。当连接到独立或集群数据库设置时，OVN 将自动确保一次只有一个处于活动状态。如果运行了多个 `ovn-northd` 实例并且活动的 `ovn-northd` 失败，则其中一个热备 `ovn-northd` 实例将自动接管。

### 具有多个 OVN DB 服务器的主备模式
您可以在 OVN 部署中运行多个 OVN DB 服务器，其中包括：

*   以主/备模式部署的 OVN DB 服务器，具有一个主 `ovsdb-server` 和多个备 `ovsdb-server`。

*   `ovn-northd` 也部署在所有这些节点上，使用 unix ctl 套接字连接到本地 OVN DB 服务器。

在这类部署中，被动节点上的 `ovn-northd` 将处理 DB 更改并计算逻辑流，但随后将其丢弃，因为被动 `ovsdb-server` 不允许写入事务。这会导致不必要的 CPU 使用。

借助运行时管理命令 `pause`，您可以暂停这些节点上的 `ovn-northd`。当被动节点变为主节点时，您可以使用运行时管理命令 `resume` 来恢复 `ovn-northd` 以处理 DB 更改。

## 逻辑流表结构
`ovn-northd` 的主要目的之一是填充 OVN_Southbound 数据库中的 `Logical_Flow` 表。本节描述 `ovn-northd` 如何为交换机和路由器逻辑数据路径执行此操作。
