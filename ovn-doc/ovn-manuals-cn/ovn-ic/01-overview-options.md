# OVN 互联控制器 (ovn-ic) 概览与选项

## 简介 (Synopsis)

```bash
ovn-ic [options]
```

## 描述 (Description)

`ovn-ic` (OVN Interconnection Controller) 是一个集中式守护进程，它与全局互联数据库 `IC_NB` / `IC_SB` 通信，以便配置本地 `NB` / `SB` 并与之交换数据，从而与其他 OVN 部署互连。

## 选项 (Options)

### 数据库选项

*   `--ovnnb-db=database`
    包含 OVN Northbound 数据库的 OVSDB 数据库。
    *   默认: `unix:/ovnnb_db.sock` (或 `OVN_NB_DB` 环境变量)。

*   `--ovnsb-db=database`
    包含 OVN Southbound 数据库的 OVSDB 数据库。
    *   默认: `unix:/ovnsb_db.sock` (或 `OVN_SB_DB` 环境变量)。

*   `--ic-nb-db=database`
    包含 OVN Interconnection Northbound 数据库的 OVSDB 数据库。
    *   默认: `unix:/ovn_ic_nb_db.sock` (或 `OVN_IC_NB_DB` 环境变量)。

*   `--ic-sb-db=database`
    包含 OVN Interconnection Southbound 数据库的 OVSDB 数据库。
    *   默认: `unix:/ovn_ic_sb_db.sock` (或 `OVN_IC_SB_DB` 环境变量)。

### 守护进程选项 (Daemon Options)

*   `--pidfile`, `--overwrite-pidfile`, `--detach`, `--monitor`, `--no-chdir`, `--no-self-confinement`, `--user`。
    (用法与其他 OVN 守护进程相同)

### 日志选项 (Logging Options)

*   `-v`, `--verbose`, `--log-file`, `--syslog-target`, `--syslog-method`。

### PKI 选项 (PKI Options)

用于配置 SSL/TLS 连接。

*   `-p`, `--private-key`
*   `-c`, `--certificate`
*   `-C`, `--ca-cert`
*   `--ssl-server-name`

### 其他选项 (Other Options)

*   `--unixctl=socket`
    设置运行时管理命令的控制套接字名称。
*   `-h`, `--help`
*   `-V`, `--version`
