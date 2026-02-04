# OVN Northbound DB 管理工具 (ovn-nbctl) 概览与选项

## 简介 (Synopsis)

```bash
ovn-nbctl [options] command [arg...]
```

## 描述 (Description)

`ovn-nbctl` 程序用于配置 OVN Northbound 数据库。它提供了一个连接到其配置数据库的高级接口。关于数据库模式的详细文档，请参见 `ovn-nb(5)`。

`ovn-nbctl` 连接到维护 OVN Northbound 配置数据库的 `ovsdb-server` 进程。使用此连接，它根据提供的命令查询并可能应用更改到数据库。

`ovn-nbctl` 可以在一次运行中执行任意数量的命令，这些命令作为针对数据库的单个原子事务实现。

`ovn-nbctl` 命令行以全局选项开始（详情见下文的“选项”）。全局选项后跟一个或多个命令。每个命令应以单独的 `--` 作为命令行参数开始，以将其与后续命令分隔开。（第一个命令前的 `--` 是可选的。）命令本身以命令特定选项（如果有）开始，后跟命令名称和任何参数。

## 守护进程模式 (Daemon Mode)

在最普通的调用方式中，`ovn-nbctl` 连接到托管 Northbound 数据库的 OVSDB 服务器，检索足以完成其工作的数据库部分副本，向服务器发送事务请求，并接收和处理服务器的回复。在常见的交互式使用中，这很好，但如果数据库很大，`ovn-nbctl` 检索数据库部分副本的步骤可能会花费很长时间，从而导致整体性能不佳。

为了在这种情况下提高性能，`ovn-nbctl` 提供了一种“守护进程模式”。在这种模式下，用户首先启动在后台运行的 `ovn-nbctl`，然后使用该守护进程执行操作。在多次 `ovn-nbctl` 命令调用中，这种方式整体性能更好，因为它只在开始时检索一次数据库副本，而不是每次程序运行时都检索。

使用 `--detach` 选项启动 `ovn-nbctl` 守护进程。使用此选项，`ovn-nbctl` 会将控制套接字的名称打印到标准输出。客户端应将此名称保存在环境变量 `OVN_NB_DAEMON` 中。在 Bourne shell 下，可以这样操作：

```bash
export OVN_NB_DAEMON=$(ovn-nbctl --pidfile --detach)
```

当设置了 `OVN_NB_DAEMON` 时，`ovn-nbctl` 会自动且透明地使用守护进程来执行其命令。

当不再需要守护进程时，杀死它并取消设置环境变量，例如：

```bash
kill $(cat $OVN_RUNDIR/ovn-nbctl.pid)
unset OVN_NB_DAEMON
```

使用守护进程模式时，除了 `OVN_NB_DAEMON` 环境变量外，另一种方法是指定 Unix 套接字的路径。启动 `ovn-nbctl` 守护进程时，使用 `-u` 选项指定套接字文件的完整路径。例如：

```bash
ovn-nbctl --detach -u /tmp/mysock.ctl
```

然后，要连接到正在运行的守护进程，请使用 `-u` 选项并指定守护进程启动时创建的套接字的完整路径：

```bash
ovn-nbctl -u /tmp/mysock.ctl show
```

### 守护进程命令 (Daemon Commands)

守护进程模式内部使用与 `ovn-appctl` 相同的机制实现。也可以直接使用 `ovn-appctl` 执行以下命令：

*   `run [options] command [arg...] [-- [options] command [arg...] ...]`
    指示守护进程运行上述一个或多个 `ovn-nbctl` 命令，并回复运行这些命令的结果。除了命令特定选项外，还接受 `--no-wait`, `--wait`, `--timeout`, `--dry-run`, `--oneline` 以及“表格格式化选项”下描述的选项。

*   `exit`
    使 `ovn-nbctl` 优雅终止。

## 选项 (Options)

下列选项影响 `ovn-nbctl` 的整体行为。一些单独的命令也接受自己的选项，这些选项紧跟在命令名称之前给出。如果命令行上的第一个命令有选项，则这些选项必须通过 `--` 与全局选项分隔。

`ovn-nbctl` 也接受来自 `OVN_NBCTL_OPTIONS` 环境变量的选项，格式与命令行相同。命令行选项覆盖环境变量中的选项。

*   `--no-wait | --wait=none`
*   `--wait=sb`
*   `--wait=hv`
    这些选项控制 `ovn-nbctl` 是否以及如何等待 OVN 系统与 `ovn-nbctl` 调用中所做的更改保持同步。
    *   默认情况下，或使用 `--no-wait` 或 `--wait=none`，`ovn-nbctl` 在确认更改已提交到 Northbound 数据库后立即退出，而不进行等待。
    *   使用 `--wait=sb`，在 `ovn-nbctl` 退出之前，它会等待 `ovn-northd` 将 Northbound 数据库的更新同步到 Southbound 数据库。
    *   使用 `--wait=hv`，在 `ovn-nbctl` 退出之前，它不仅等待 `ovn-northd`，还等待所有 OVN 机箱（Hypervisors 和网关）更新为 Northbound 数据库的最新状态。（如果任何机箱发生故障，这可能会变成无限期等待。）
    *   通常，`--wait=sb` 或 `--wait=hv` 仅等待当前 `ovn-nbctl` 调用所做的更改生效。这意味着，如果提供给 `ovn-nbctl` 的命令没有更改数据库，则该命令根本不会等待。使用 `sync` 命令可覆盖此行为。

*   `--db database`
    要连接的 OVSDB 数据库远程地址。如果设置了 `OVN_NB_DB` 环境变量，则使用其值作为默认值。否则，默认值为 `unix:/ovnnb_db.sock`，但这通常仅在单机 OVN 测试环境中有用。

*   `--leader-only`
*   `--no-leader-only`
    默认情况下，或使用 `--leader-only`，当数据库服务器是集群数据库时，`ovn-nbctl` 将避免连接除集群领导者 (Leader) 以外的服务器。这确保了 `ovn-nbctl` 读取和报告的任何数据都是最新的。使用 `--no-leader-only`，`ovn-nbctl` 将使用集群中的任何服务器，这意味着对于只读事务，它可能会报告并基于过时数据进行操作（即使使用 `--no-leader-only`，修改数据库的事务也总是序列化的）。有关更多信息，请参阅 `ovsdb(7)` 中的“理解集群一致性”。

*   `--shuffle-remotes`
*   `--no-shuffle-remotes`
    默认情况下，或使用 `--shuffle-remotes`，当 `--db` 或 `OVN_NB_DB` 环境变量指定的 OVSDB 连接字符串中有多个远程地址时，客户端在尝试连接之前会打乱远程地址的顺序。远程地址仅在第一次连接尝试之前被打乱一次。后续的重试（如果有）将遵循新的顺序。默认行为是确保集群数据库的客户端可以均匀分布到集群的所有成员。使用 `--no-shuffle-remotes`，`ovn-nbctl` 将使用连接字符串中指定的原始顺序进行连接。这允许用户指定首选顺序，这对测试特别有用。

*   `--no-syslog`
    默认情况下，`ovn-nbctl` 将其参数以及它对系统所做的任何更改的详细信息记录到系统日志中。此选项禁用此日志记录。这相当于 `--verbose=nbctl:syslog:warn`。

*   `--oneline`
    修改输出格式，使每个命令的输出打印在单行上。否则分隔行的换行符打印为 `\n`，输出中出现的任何 `\` 加倍。对于没有输出的每个命令打印一个空行。此选项不影响 `list` 或 `find` 命令的输出格式；参见下面的“表格格式化选项”。

*   `--dry-run`
    阻止 `ovn-nbctl` 实际修改数据库。

*   `-t secs`, `--timeout=secs`
    默认情况下，或 `secs` 为 0 时，`ovn-nbctl` 永远等待数据库的响应。此选项将运行时间限制为大约 `secs` 秒。如果超时，`ovn-nbctl` 将以 `SIGALRM` 信号退出。（超时通常仅在无法联系数据库或系统过载时发生。）

*   `--print-wait-time`
    当指定 `--wait` 时，可以使用选项 `--print-wait-time` 打印等待所花费的时间。如果指定了 `--wait=sb`，它会打印 "ovn-northd delay before processing"（命令更新 NB DB 到 `ovn-northd` 开始处理更新之间的时间）和 "ovn-northd completion"（NB DB 更新到 `ovn-northd` 成功完成 SB DB 更新之间的时间）。如果指定了 `--wait=hv`，除了上述信息外，它还会打印 "ovn-controller(s) completion"（NB DB 更新到最慢的 Hypervisor 完成处理更新之间的时间）。

### 守护进程选项 (Daemon Options)

*   `--pidfile[=pidfile]`
    创建进程 ID 文件（默认 `program.pid`）。
*   `--overwrite-pidfile`
    强制覆盖现有的 pidfile。
*   `--detach`
    在后台运行。
*   `--monitor`
    创建监控进程以在崩溃时重启守护进程。
*   `--no-chdir`
    防止守护进程将当前工作目录更改为 `/`。
*   `--no-self-confinement`
    禁用自我限制（不推荐，除非使用其他访问控制）。
*   `--user=user:group`
    以指定用户和组运行。

### 日志选项 (Logging Options)

*   `-v[spec]`, `--verbose=[spec]`
    设置日志级别。
*   `-v`, `--verbose`
    设置最大详细级别 (`dbg`)。
*   `--log-file[=file]`
    启用文件日志记录。
*   `--syslog-target=host:port`
    发送 syslog 到远程 UDP 端口。
*   `--syslog-method=method`
    指定 syslog 发送方法 (`libc`, `unix:file`, `udp`, `null`)。

### 表格格式化选项 (Table Formatting Options)

这些选项控制 `list` 和 `find` 命令的输出格式。

*   `-f format`, `--format=format`
    设置表格格式类型：
    *   `table`: 2-D 文本表格，列对齐。
    *   `list` (默认): 每行一列，行之间用空行分隔。
    *   `html`: HTML 表格。
    *   `csv`: 逗号分隔值 (RFC 4180)。
    *   `json`: JSON 格式 (RFC 4627)。

*   `-d format`, `--data=format`
    设置输出表格内单元格的格式（除非表格格式为 `json`）：
    *   `string` (默认): 简单格式。
    *   `bare`: 去除标点符号的简单格式（集合和映射周围省略 `[]` 和 `{}`，不带引号）。
    *   `json`: JSON 格式。

*   `--no-headings`
    抑制表格输出的第一行标题。

*   `--pretty`
    使 JSON 输出更易读（缩进）。

*   `--bare`
    相当于 `--format=list --data=bare --no-headings`。

### PKI 选项 (PKI Options)

*   `-p`, `--private-key=privkey.pem`
*   `-c`, `--certificate=cert.pem`
*   `-C`, `--ca-cert=cacert.pem`
*   `--ssl-server-name=servername`
*   `--bootstrap-ca-cert=cacert.pem`

### 其他选项 (Other Options)

*   `-h`, `--help`
*   `-V`, `--version`
