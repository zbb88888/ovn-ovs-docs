# OVN Southbound DB 管理工具 (ovn-sbctl) 概览与选项

## 简介 (Synopsis)

```bash
ovn-sbctl [options] command [arg...]
```

## 描述 (Description)

`ovn-sbctl` 程序用于配置 OVN Southbound 数据库。它提供了一个连接到其配置数据库的高级接口。关于数据库模式的详细文档，请参见 `ovn-sb(5)`。

`ovn-sbctl` 连接到维护 OVN Southbound 配置数据库的 `ovsdb-server` 进程。使用此连接，它根据提供的命令查询并可能应用更改到数据库。

`ovn-sbctl` 可以在一次运行中执行任意数量的命令，这些命令作为针对数据库的单个原子事务实现。

## 守护进程模式 (Daemon Mode)

为了提高性能，特别是在数据库很大的情况下，`ovn-sbctl` 提供了“守护进程模式”。

*   使用 `--detach` 选项启动守护进程。
*   将输出的控制套接字路径保存到 `OVN_SB_DAEMON` 环境变量中。
    ```bash
    export OVN_SB_DAEMON=$(ovn-sbctl --pidfile --detach)
    ```
*   或者使用 `-u` 选项指定套接字路径。

### 守护进程命令 (Daemon Commands)

*   `run [options] command [arg...]`
    指示守护进程运行命令。
*   `exit`
    优雅终止守护进程。

## 选项 (Options)

*   `--db database`
    要连接的 OVSDB 数据库远程地址。如果设置了 `OVN_SB_DB` 环境变量，则使用其值。默认 `unix:/ovnsb_db.sock`。

*   `--leader-only` / `--no-leader-only`
    控制是否仅连接到集群领导者。默认 `leader-only` 确保数据最新。

*   `--shuffle-remotes` / `--no-shuffle-remotes`
    控制是否打乱连接字符串中远程地址的顺序。

*   `--no-syslog`
    禁用系统日志记录。

*   `--oneline`
    单行输出格式。

*   `--dry-run`
    不实际修改数据库。

*   `-t secs`, `--timeout=secs`
    限制运行时间。

### 守护进程选项 (Daemon Options)

*   `--pidfile`, `--overwrite-pidfile`, `--detach`, `--monitor`, `--no-chdir`, `--no-self-confinement`, `--user`。

### 日志选项 (Logging Options)

*   `-v`, `--verbose`, `--log-file`, `--syslog-target`, `--syslog-method`。

### 表格格式化选项 (Table Formatting Options)

*   `-f`, `--format` (table, list, html, csv, json)
*   `-d`, `--data` (string, bare, json)
*   `--no-headings`, `--pretty`, `--bare`

### PKI 选项 (PKI Options)

*   `-p`, `--private-key`
*   `-c`, `--certificate`
*   `-C`, `--ca-cert`
*   `--ssl-server-name`
*   `--bootstrap-ca-cert`
