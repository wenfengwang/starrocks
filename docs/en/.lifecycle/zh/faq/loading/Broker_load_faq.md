---
displayed_sidebar: English
---

# Broker Load

## 1. Broker Load 是否支持重新运行已成功运行且处于 FINISHED 状态的加载作业？

Broker Load 不支持重新运行已成功运行且处于 FINISHED 状态的加载作业。此外，为了防止数据丢失和重复，Broker Load 不允许重复使用成功运行加载作业的标签。您可以使用 [SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) 查看加载作业的历史记录并找到您想要重新运行的加载作业。然后，您可以复制该加载作业的信息，并使用该作业信息（标签除外）来创建另一个加载作业。

## 2. 使用 Broker Load 从 HDFS 加载数据时，如果加载到目标 StarRocks 表中的日期和时间值比源数据文件中的日期和时间值晚 8 小时，该怎么办？

目标 StarRocks 表和 Broker Load 作业在创建时都被编译为使用中国标准时间（CST）时区（通过使用 `timezone` 参数指定）。但是，服务器设置为基于协调世界时（UTC）时区运行。因此，在数据加载期间，源数据文件中的日期和时间值会被额外增加 8 小时。为了防止这个问题，创建目标 StarRocks 表时请不要指定 `timezone` 参数。

## 3. 使用 Broker Load 加载 ORC 格式数据时，如果出现 `ErrorMsg: type:ETL_RUN_FAIL; msg:Cannot cast '<slot 6>' from VARCHAR to ARRAY<VARCHAR(30)>` 错误，该怎么办？

源数据文件的列名与目标 StarRocks 表的列名不同。在这种情况下，您必须在加载语句中使用 `SET` 子句来指定文件与表之间的列映射。执行 `SET` 子句时，StarRocks 需要进行类型推断，但它在调用 [cast](../../sql-reference/sql-functions/cast.md) 函数以将源数据转换为目标数据类型时失败了。要解决此问题，请确保源数据文件的列名与目标 StarRocks 表的列名相同。这样，就不需要 `SET` 子句，StarRocks 也就不需要调用 cast 函数来进行数据类型转换。然后 Broker Load 作业就可以成功运行。

## 4. Broker Load 作业没有报错，但为什么无法查询到加载的数据？

Broker Load 是一种异步加载方法。即使加载语句没有返回错误，加载作业仍可能失败。运行 Broker Load 作业后，您可以使用 [SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) 查看作业的结果和 `errmsg`。然后，您可以修改作业配置并重试。

## 5. 如果出现“failed to send batch”或“TabletWriter add batch with unknown id”错误，该怎么办？

写入数据所需的时间超过了上限，导致超时错误。为了解决此问题，请根据您的业务需求调整 [session variable](../../reference/System_variable.md) `query_timeout` 和 [BE 配置项](../../administration/BE_configuration.md#configure-be-static-parameters) `streaming_load_rpc_max_alive_time_sec` 的设置。

## 6. 如果出现“LOAD-RUN-FAIL; msg:OrcScannerAdapter::init_include_columns. col name = xxx not found”错误，该怎么办？

如果您正在加载 Parquet 或 ORC 格式的数据，请检查源数据文件第一行中的列名是否与目标 StarRocks 表的列名相同。

```SQL
(tmp_c1,tmp_c2)
SET
(
   id=tmp_c2,
   name=tmp_c1
)
```

上述示例将源数据文件中的 `tmp_c1` 和 `tmp_c2` 列映射到目标 StarRocks 表的 `name` 和 `id` 列。如果您没有指定 `SET` 子句，那么将使用 `column_list` 参数中指定的列名来声明列映射。更多信息，请参见 [BROKER LOAD](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

> **注意**
> 如果源数据文件是 Apache Hive™ 生成的 ORC 格式文件，并且文件的第一行包含 `(_col0, _col1, _col2, ...)`，则可能会出现“Invalid Column Name”错误。如果出现此错误，您需要使用 `SET` 子句来指定列映射。

## 7. 如何处理导致 Broker Load 作业运行时间过长的错误？

查看 FE 日志文件 **fe.log**，并根据作业标签搜索加载作业的 ID。然后，查看 BE 日志文件 **be.INFO**，并根据作业 ID 检索加载作业的日志记录，以定位错误的根本原因。

## 8. 如何配置运行在 HA 模式下的 Apache HDFS 集群？

如果 HDFS 集群运行在高可用性（HA）模式下，请按照以下方式配置：

- `dfs.nameservices`：HDFS 集群的名称，例如 `"dfs.nameservices" = "my_ha"`。

- `dfs.ha.namenodes.xxx`：HDFS 集群中 NameNode 的名称。如果指定了多个 NameNode 名称，请用逗号（`,`）分隔。`xxx` 是您在 `dfs.nameservices` 中指定的 HDFS 集群名称，例如 `"dfs.ha.namenodes.my_ha" = "my_nn"`。

- `dfs.namenode.rpc-address.xxx.nn`：HDFS 集群中 NameNode 的 RPC 地址。`nn` 是您在 `dfs.ha.namenodes.xxx` 中指定的 NameNode 名称，例如 `"dfs.namenode.rpc-address.my_ha.my_nn" = "host:port"`。

- `dfs.client.failover.proxy.provider`：客户端将连接的 NameNode 的提供者。默认值为 `org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider`。

示例：

```SQL
(
    "dfs.nameservices" = "my-ha",
    "dfs.ha.namenodes.my-ha" = "my-namenode1, my-namenode2",
    "dfs.namenode.rpc-address.my-ha.my-namenode1" = "nn1-host:rpc_port",
    "dfs.namenode.rpc-address.my-ha.my-namenode2" = "nn2-host:rpc_port",
    "dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
)
```

HA 模式可以与简单认证或 Kerberos 认证结合使用。例如，要使用简单认证方式访问运行在 HA 模式的 HDFS 集群，您需要指定以下配置：

```SQL
(
    "username" = "user",
    "password" = "passwd",
    "dfs.nameservices" = "my-ha",
    "dfs.ha.namenodes.my-ha" = "my_namenode1, my_namenode2",
    "dfs.namenode.rpc-address.my-ha.my_namenode1" = "nn1-host:rpc_port",
    "dfs.namenode.rpc-address.my-ha.my_namenode2" = "nn2-host:rpc_port",
    "dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
)
```

您可以将 HDFS 集群的配置添加到 **hdfs-site.xml** 文件中。这样，使用 brokers 从 HDFS 集群加载数据时，只需指定文件路径和认证信息即可。

## 9. 如何配置 Hadoop ViewFS 联邦？

将 ViewFs 相关的配置文件 `core-site.xml` 和 `hdfs-site.xml` 复制到 **broker/conf** 目录。

如果您有自定义文件系统，还需要将文件系统相关的 **.jar** 文件复制到 **broker/lib** 目录。

## 10. 当访问需要 Kerberos 认证的 HDFS 集群时，如果出现 “Can't get Kerberos realm” 错误，该怎么办？

检查是否在部署 broker 的所有主机上配置了 **/etc/krb5.conf** 文件。

如果错误仍然存在，请在 broker 启动脚本的 `JAVA_OPTS` 变量末尾添加 `-Djava.security.krb5.conf=/etc/krb5.conf`。