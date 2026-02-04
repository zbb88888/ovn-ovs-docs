# Connection 表

Open vSwitch 数据库 (OVSDB) 客户端的数据库连接配置。

此表主要配置 Open vSwitch 数据库服务器 (`ovsdb-server`)。

Open vSwitch 数据库服务器可以发起并维护到远程客户端的主动连接。它还可以监听数据库连接。

## 汇总

*   核心特性:
    *   `target`
        *   字符串 (在表中必须唯一)
    *   `read_only`
        *   布尔值
    *   `role`
        *   字符串
*   客户端故障检测和处理:
    *   `max_backoff`
        *   可选整数，至少 1,000
    *   `inactivity_probe`
        *   可选整数
*   状态:
    *   `is_connected`
        *   布尔值
    *   `status : last_error`
        *   可选字符串
    *   `status : state`
        *   可选字符串，`ACTIVE`, `BACKOFF`, `CONNECTING`, `IDLE`, 或 `VOID` 之一
    *   `status : sec_since_connect`
        *   可选字符串，包含整数，至少 0
    *   `status : sec_since_disconnect`
        *   可选字符串，包含整数，至少 0
    *   `status : locks_held`
        *   可选字符串
    *   `status : locks_waiting`
        *   可选字符串
    *   `status : locks_lost`
        *   可选字符串
    *   `status : n_connections`
        *   可选字符串，包含整数，至少 2
    *   `status : bound_port`
        *   可选字符串，包含整数
*   通用列:
    *   `external_ids`
        *   字符串对的映射
    *   `other_config`
        *   字符串对的映射

## 详细信息

*   **核心特性**:

    *   `target`: 字符串 (在表中必须唯一)
        *   客户端的连接方法。
        *   目前支持以下连接方法：

        *   `ssl:host[:port]`
            *   给定主机上的指定 SSL/TLS 端口，该主机可以是 DNS 名称（如果使用 unbound 库构建）或 IP 地址。使用此形式时必须提供有效的 SSL 配置，此配置可以通过命令行选项或 `SSL` 表指定。
            *   如果未指定端口，则默认为 6640。
            *   SSL/TLS 支持是一项可选功能，并不总是作为 OVN 或 Open vSwitch 的一部分构建。

        *   `tcp:host[:port]`
            *   给定主机上的指定 TCP 端口，该主机可以是 DNS 名称（如果使用 unbound 库构建）或 IP 地址（IPv4 或 IPv6）。如果主机是 IPv6 地址，请将其括在方括号中，例如 `tcp:[::1]:6640`。
            *   如果未指定端口，则默认为 6640。

        *   `pssl:[port][:host]`
            *   监听指定 TCP 端口上的 SSL/TLS 连接。指定端口为 0 以让内核自动选择可用端口。如果指定了主机（可以是 DNS 名称（如果使用 unbound 库构建）或 IP 地址），则连接仅限于解析或指定的本地 IP 地址（IPv4 或 IPv6）。如果主机是 IPv6 地址，请将其括在方括号中，例如 `pssl:6640:[::1]`。如果未指定主机，则仅监听 IPv4（不监听 IPv6）地址。使用此形式时必须提供有效的 SSL/TLS 配置，这可以通过命令行选项或 `SSL` 表指定。
            *   如果未指定端口，则默认为 6640。
            *   SSL/TLS 支持是一项可选功能，并不总是作为 OVN 或 Open vSwitch 的一部分构建。

        *   `ptcp:[port][:host]`
            *   监听指定 TCP 端口上的连接。指定端口为 0 以让内核自动选择可用端口。如果指定了主机（可以是 DNS 名称（如果使用 unbound 库构建）或 IP 地址），则连接仅限于解析或指定的本地 IP 地址（IPv4 或 IPv6）。如果主机是 IPv6 地址，请将其括在方括号中，例如 `ptcp:6640:[::1]`。如果未指定主机，则仅监听 IPv4 地址。
            *   如果未指定端口，则默认为 6640。

        *   配置多个客户端时，`target` 值必须唯一。重复的 `target` 值会产生未指定的结果。

    *   `read_only`: 布尔值
        *   `true` 将这些连接限制为只读事务，`false` 允许它们修改数据库。

    *   `role`: 字符串
        *   包含此连接条目的角色名称的字符串。

*   **客户端故障检测和处理**:

    *   `max_backoff`: 可选整数，至少 1,000
        *   连接尝试之间等待的最大毫秒数。默认值是特定于实现的。

    *   `inactivity_probe`: 可选整数
        *   在向客户端发送不活动探测消息之前，连接上空闲时间的最大毫秒数。如果 Open vSwitch 在指定的秒数内未与客户端通信，它将发送探测。如果在相同的额外时间内未收到响应，Open vSwitch 将假定连接已断开并尝试重新连接。默认值是特定于实现的。值为 0 禁用不活动探测。

*   **状态**:

    *   `is_connected` 键值对始终更新。`status` 列中的其他键值对可能会更新，具体取决于目标类型。
    *   当 `target` 指定侦听入站连接的连接方法（例如 `ptcp:` 或 `punix:`）时，`n_connections` 和 `is_connected` 也可能会更新，而其余键值对将被省略。
    *   另一方面，当 `target` 指定出站连接时，所有键值对都可能更新，除了上述两个与入站连接目标关联的键值对。它们被省略。

    *   `is_connected`: 布尔值
        *   如果当前已连接到此客户端，则为 `true`，否则为 `false`。

    *   `status : last_error`: 可选字符串
        *   到管理器的连接上最后一个错误的人类可读描述；即 `strerror(errno)`。仅当发生错误时此键才存在。

    *   `status : state`: 可选字符串，`ACTIVE`, `BACKOFF`, `CONNECTING`, `IDLE`, 或 `VOID` 之一
        *   到管理器的连接状态：
        *   `VOID` 连接已禁用。
        *   `BACKOFF` 尝试以递增的周期重新连接。
        *   `CONNECTING` 正在尝试连接。
        *   `ACTIVE` 已连接，远程主机响应。
        *   `IDLE` 连接空闲。等待保持活动的响应。
        *   这些值将来可能会更改。它们仅供人类阅读。

    *   `status : sec_since_connect`: 可选字符串，包含整数，至少 0
        *   自此客户端上次成功连接到数据库以来的时间量（以秒为单位）。如果客户端从未成功连接过，则值为空。

    *   `status : sec_since_disconnect`: 可选字符串，包含整数，至少 0
        *   自此客户端上次断开与数据库的连接以来的时间量（以秒为单位）。如果客户端从未断开连接，则值为空。

    *   `status : locks_held`: 可选字符串
        *   连接持有的 OVSDB 锁名称的空格分隔列表。如果连接不持有任何锁，则省略。

    *   `status : locks_waiting`: 可选字符串
        *   连接当前正在等待获取的 OVSDB 锁名称的空格分隔列表。如果连接没有等待任何锁，则省略。

    *   `status : locks_lost`: 可选字符串
        *   连接被另一个 OVSDB 客户端窃取的 OVSDB 锁名称的空格分隔列表。如果此连接未被窃取任何锁，则省略。

    *   `status : n_connections`: 可选字符串，包含整数，至少 2
        *   当 `target` 指定侦听入站连接的连接方法（例如 `ptcp:` 或 `pssl:`）并且实际上有多个连接处于活动状态时，该值为活动连接数。否则，省略此键值对。

    *   `status : bound_port`: 可选字符串，包含整数
        *   当 `target` 为 `ptcp:` 或 `pssl:` 时，这是 OVSDB 服务器正在监听的 TCP 端口。（当 `target` 指定端口 0，允许内核选择任何可用端口时，这特别有用。）

*   **通用列**:

    *   这些列的总体用途在本文档开头的“通用列”下进行了描述。

    *   `external_ids`: 字符串对的映射

    *   `other_config`: 字符串对的映射

# SSL 表

用于 ovn-sb 数据库访问的 SSL/TLS 配置。

## 汇总

*   `private_key`
    *   字符串
*   `certificate`
    *   字符串
*   `ca_cert`
    *   字符串
*   `bootstrap_ca_cert`
    *   布尔值
*   `ssl_protocols`
    *   字符串
*   `ssl_ciphers`
    *   字符串
*   `ssl_ciphersuites`
    *   字符串
*   通用列:
    *   `external_ids`
        *   字符串对的映射

## 详细信息

*   `private_key`: 字符串
    *   包含私钥的 PEM 文件的名称，该私钥用作交换机与控制器进行 SSL/TLS 连接时的身份。

*   `certificate`: 字符串
    *   包含证书的 PEM 文件的名称，该证书由控制器和管理器使用的证书颁发机构 (CA) 签署，用于证明交换机的私钥，识别值得信赖的交换机。

*   `ca_cert`: 字符串
    *   包含 CA 证书的 PEM 文件的名称，用于验证交换机是否连接到值得信赖的控制器。

*   `bootstrap_ca_cert`: 布尔值
    *   如果设置为 true，则 Open vSwitch 将尝试在其第一个 SSL/TLS 连接上从控制器获取 CA 证书，并将其保存到命名的 PEM 文件中。如果成功，它将立即断开连接并重新连接，从那时起，所有 SSL/TLS 连接都必须通过由此获得的 CA 证书签署的证书进行身份验证。此选项将 SSL/TLS 连接暴露给获取初始 CA 证书的中间人攻击。它对于引导仍然很有用。

*   `ssl_protocols`: 字符串
    *   要为 SSL/TLS 连接启用的 SSL/TLS 协议的范围或逗号或空格分隔列表。
    *   支持的协议包括 TLSv1.2 和 TLSv1.3。范围可以以两个协议名称之间用破折号分隔的形式提供（TLSv1.2-TLSv1.3），或者作为带有加号的单个协议名称（TLSv1.2+）。该值可以是协议列表或恰好一个范围。范围是指定协议的首选方式，并且配置始终表现为好像提供了指定的最小和最大版本之间的范围，即，如果该值设置为 TLSv1.X,TLSv1.(X+2)，则 TLSv1.(X+1) 也将被启用，就像它是一个范围一样。无论顺序如何，建立连接时都将选择双方都支持的最高协议。
    *   如果省略此选项，默认值为 TLSv1.2+。

*   `ssl_ciphers`: 字符串
    *   要为使用 TLSv1.2 的 SSL/TLS 连接支持的密码列表（以 OpenSSL 密码字符串格式）。如果省略此选项，默认值为 `DEFAULT:@SECLEVEL=2`。

*   `ssl_ciphersuites`: 字符串
    *   要为使用 TLSv1.3 及更高版本的 SSL/TLS 连接支持的密码套件列表（以 OpenSSL 密码套件字符串格式）。如果省略此选项，将使用 OpenSSL 的默认值。

*   **通用列**:

    *   这些列的总体用途在本文档开头的“通用列”下进行了描述。

    *   `external_ids`: 字符串对的映射

# RBAC_Role 表

基于角色的访问控制的角色表。

## 汇总

*   `name`
    *   字符串
*   `permissions`
    *   字符串-`RBAC_Permission` 弱引用对的映射

## 详细信息

*   `name`: 字符串
    *   角色名称，对应于 `Connection` 表中的 `role` 列。

*   `permissions`: 字符串-`RBAC_Permission` 弱引用对的映射
    *   表名到 `RBAC_Permission` 表中行的映射。

# RBAC_Permission 表

基于角色的访问控制的权限表。

## 汇总

*   `table`
    *   字符串
*   `authorization`
    *   字符串集合
*   `insert_delete`
    *   布尔值
*   `update`
    *   字符串集合

## 详细信息

*   `table`: 字符串
    *   此行适用的表名。

*   `authorization`: 字符串集合
    *   标识要与客户端 ID 进行比较的列和 column:key 对的字符串集合。至少需要一个匹配项才能获得授权。零长度字符串被视为特殊值，表示所有客户端都应被视为已授权。

*   `insert_delete`: 布尔值
    *   当为 "true" 时，允许行插入和授权的行删除。

*   `update`: 字符串集合
    *   标识允许授权客户端修改的列和 column:key 对的字符串集合。
