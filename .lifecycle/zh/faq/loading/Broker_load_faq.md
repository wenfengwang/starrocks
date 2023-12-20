---
displayed_sidebar: English
---

# 经纪人负载

## 1. 经纪人负载是否支持重新运行已成功完成且处于“已完成”状态的加载任务？

经纪人负载不支持重新运行已成功完成且处于“已完成”状态的加载任务。此外，为了防止数据丢失和重复，经纪人负载不允许重用已成功完成的加载任务的标签。您可以使用 [SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) 命令查看加载任务的历史记录并找到您想要重新运行的加载任务。然后，您可以复制该加载任务的信息，并使用除了标签之外的作业信息来创建另一个加载任务。

## 2. 当我使用经纪人负载从 HDFS 加载数据时，如果加载到目标 StarRocks 表中的日期和时间值比源数据文件中的值晚 8 小时，该怎么办？

目标 StarRocks 表和经纪人负载任务在创建时都被编译为使用中国标准时间（CST）时区（通过使用 timezone 参数指定）。然而，服务器被设置为基于协调世界时（UTC）时区运行。因此，在数据加载过程中，源数据文件中的日期和时间值会被额外增加 8 小时。为了避免这个问题，请在创建目标 StarRocks 表时不要指定时区参数。

## 3. 当我使用经纪人负载加载 ORC 格式数据时，如果出现错误信息：类型：ETL_RUN_FAIL；信息：无法将 '<slot 6>' 从 VARCHAR 转换为 ARRAY<VARCHAR(30)>，我该怎么办？

源数据文件的列名与目标StarRocks表的列名不同。在这种情况下，您必须在加载语句中使用`SET`子句来指定文件与表之间的列映射。在执行`SET`子句时，StarRocks需要进行类型推断，但它在调用[转换](../../sql-reference/sql-functions/cast.md)函数以将源数据转换为目标数据类型时失败了。为了解决这个问题，请确保源数据文件的列名与目标StarRocks表的列名相同。这样一来，就不需要`SET`子句，StarRocks也就不需要调用转换函数来执行数据类型转换。之后，经纪人负载任务便可以成功运行。

## 4. 经纪人负载任务没有报错，但为什么我无法查询到已加载的数据？

经纪人负载是一种异步加载方法。即使加载语句没有返回错误，加载任务仍可能失败。在运行经纪人负载任务后，您可以使用 [SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) 命令查看加载任务的结果和 `errmsg`。然后，您可以修改任务配置并重试。

## 5. 如果出现“发送批次失败”或“TabletWriter 添加未知 ID 的批次”错误，我该怎么办？

写入数据所用的时间超出了上限，导致了超时错误。为了解决这个问题，请根据您的业务需求调整[会话变量](../../reference/System_variable.md) `query_timeout` 和 [BE 配置项](../../administration/BE_configuration.md#configure-be-static-parameters) `streaming_load_rpc_max_alive_time_sec` 的设置。

## 6. 如果出现“LOAD-RUN-FAIL；信息：OrcScannerAdapter::init_include_columns. 列名 = xxx 未找到”错误，我该怎么办？

如果您正在加载 Parquet 或 ORC 格式的数据，请检查源数据文件中第一行的列名是否与目标 StarRocks 表的列名相同。

```SQL
(tmp_c1,tmp_c2)
SET
(
   id=tmp_c2,
   name=tmp_c1
)
```

上述示例中，源数据文件的 `tmp_c1` 和 `tmp_c2` 列分别映射到目标 StarRocks 表的 `name` 和 `id` 列。如果您未指定 `SET` 子句，则会使用 `column_list` 参数中指定的列名来声明列映射。更多信息，请参考[BROKER LOAD](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)文档。

> **注意**
> 如果源数据文件是由 Apache Hive™ 生成的 ORC 格式文件，并且文件的第一行包含了 `(_col0, _col1, _col2, ...)`，则可能会出现“无效的列名”错误。如果发生这种错误，您需要使用 `SET` 子句来指定列映射。

## 7. 如果经纪人负载任务运行时间过长等错误发生，我该如何处理？

查看 FE 日志文件 **fe.log**，并根据作业标签搜索加载任务的 ID。然后，查看 BE 日志文件 **be.INFO**，并根据作业 ID 检索加载任务的日志记录，以确定错误的根本原因。

## 8. 如何配置运行在 HA 模式下的 Apache HDFS 集群？

如果 HDFS 集群运行在高可用性（HA）模式下，请按照以下方式配置：

- dfs.nameservices：HDFS 集群的名称，例如，“dfs.nameservices” = "my_ha"。

- dfs.ha.namenodes.xxx：HDFS 集群中的 NameNode 名称。如果指定了多个 NameNode 名称，请用逗号（,）分隔。xxx 是您在 dfs.nameservices 中指定的 HDFS 集群名称，例如，“dfs.ha.namenodes.my_ha” = "my_nn"。

- dfs.namenode.rpc-address.xxx.nn：HDFS 集群中的 NameNode 的 RPC 地址。nn 是您在 dfs.ha.namenodes.xxx 中指定的 NameNode 名称，例如，“dfs.namenode.rpc-address.my_ha.my_nn” = "host:port"。

- dfs.client.failover.proxy.provider：客户端连接的 NameNode 的代理提供者。默认值为 org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider。

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

HA 模式可以与简单认证或 Kerberos 认证一起使用。例如，要使用简单认证方式访问运行在 HA 模式的 HDFS 集群，您需要指定以下配置：

```SQL
(
    "username"="user",
    "password"="passwd",
    "dfs.nameservices" = "my-ha",
    "dfs.ha.namenodes.my-ha" = "my_namenode1, my_namenode2",
    "dfs.namenode.rpc-address.my-ha.my-namenode1" = "nn1-host:rpc_port",
    "dfs.namenode.rpc-address.my-ha.my-namenode2" = "nn2-host:rpc_port",
    "dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
)
```

您可以将HDFS集群的配置添加到**hdfs-site.xml**文件中。这样，在使用经纪人从HDFS集群加载数据时，您只需要指定文件路径和认证信息。

## 9. 如何配置 Hadoop ViewFS 联合？

将 ViewFs 相关的配置文件 `core-site.xml` 和 `hdfs-site.xml` 复制到 **broker/conf** 目录。

如果您有自定义的文件系统，还需要将与文件系统相关的 **.jar** 文件复制到 **broker/lib** 目录中。

## 10. 当我访问需要 Kerberos 认证的 HDFS 集群时，如果出现“无法获取 Kerberos 领域”错误，我该怎么办？

检查是否在所有部署经纪人的主机上配置了 **/etc/krb5.conf** 文件。

如果问题依旧存在，请在经纪人启动脚本中的 JAVA_OPTS 变量末尾添加 -Djava.security.krb5.conf=/etc/krb5.conf。
