---
displayed_sidebar: English
---

# 代理负载

## 1. Broker Load 是否支持重新运行已成功运行且处于 FINISHED 状态的加载作业？

Broker Load 不支持重新运行已成功运行且处于 FINISHED 状态的加载作业。此外，为了防止数据丢失和重复，Broker Load 不允许重复使用成功运行的加载作业的标签。您可以使用 [SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) 查看加载作业的历史记录，并查找要重新运行的加载作业。然后，您可以复制该加载作业的信息，并使用该作业信息（标签除外）创建另一个加载作业。

## 2. 使用 Broker Load 从 HDFS 加载数据时，如果目标 StarRocks 表中加载的日期和时间值比源数据文件中的日期和时间值晚了 8 小时，该怎么办？

目标 StarRocks 表和 Broker Load 作业在创建时都会编译为使用中国标准时间 （CST） 时区（使用参数指定 `timezone` ）。但是，服务器设置为基于协调世界时 （UTC） 时区运行。因此，在数据加载期间，源数据文件中的日期和时间值会额外增加 8 小时。为避免出现该问题，请在创建目标 StarRocks 表时不要指定 `timezone` 参数。

## 3. 使用Broker Load加载ORC格式的数据时，如果 `ErrorMsg: type:ETL_RUN_FAIL; msg:Cannot cast '<slot 6>' from VARCHAR to ARRAY<VARCHAR(30)>` 出现错误，该怎么办？

源数据文件的列名与目标 StarRocks 表的列名不同。在这种情况下，必须使用 `SET` load 语句中的子句来指定文件和表之间的列映射。`SET` StarRocks 在执行该子句时需要进行类型推断，但无法调用 [cast](../../sql-reference/sql-functions/cast.md) 函数将源数据转换为目标数据类型。要解决此问题，请确保源数据文件的列名与目标 StarRocks 表的列名相同。因此，`SET`不需要该子句，因此 StarRocks 不需要调用 cast 函数来执行数据类型转换。然后，可以成功运行 Broker Load 作业。

## 4. Broker Load 作业不报告错误，但为什么我无法查询加载的数据？

Broker Load 是一种异步加载方法。即使 load 语句未返回错误，加载作业仍可能失败。运行 Broker Load 作业后，可以使用 [SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) 查看加载作业的结果和`errmsg`结果。然后，您可以修改作业配置并重试。

## 5. 如果出现“发送批次失败”或“TabletWriter 添加未知 ID 批次”错误，我该怎么办？

写入数据所花费的时间超过上限，导致超时错误。为了解决该问题，请根据业务需求修改会话变量`query_timeout`和 BE配置项`streaming_load_rpc_max_alive_time_sec`的设置。

## 6. 如果“LOAD-RUN-FAIL;msg：OrcScannerAdapter：：init_include_columns。col name = xxx not found“错误发生？

如果您加载的是 Parquet 或 ORC 格式的数据，请检查源数据文件首行的列名是否与目标 StarRocks 表的列名一致。

```SQL
(tmp_c1,tmp_c2)
SET
(
   id=tmp_c2,
   name=tmp_c1
)
```

上述示例将 `tmp_c1` `tmp_c2` 源数据文件的 and 列分别映射到`name` `id` 目标 StarRocks 表的 and 列上。如果未指定 `SET` 子句，则使用参数中指定的列名 `column_list` 来声明列映射。有关更多信息，请参阅 [BROKER LOAD](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

> **通知**
>
> 如果源数据文件是 Apache Hive™ 生成的 ORC 格式文件，并且文件的第一行为 ， `(_col0, _col1, _col2, ...)`则可能会出现“列名无效”错误。如果发生此错误，则需要使用子 `SET` 句指定列映射。

## 7. 如何处理错误，例如导致 Broker Load 作业运行时间过长的错误？

查看 FE 日志文件 **fe.log**，并根据作业标签搜索加载作业的 ID。然后，查看 BE 日志文件 **be.INFO**，并根据作业 ID 检索加载作业的日志记录，以定位错误的根本原因。

## 8. 如何配置以 HA 模式运行的 Apache HDFS 集群？

如果HDFS集群在高可用（HA）模式下运行，请进行如下配置：

- `dfs.nameservices`：HDFS集群的名称，例如。 `"dfs.nameservices" = "my_ha"`

- `dfs.ha.namenodes.xxx`：HDFS集群中NameNode的名称。如果指定多个NameNode名称，请用逗号（）分隔。`,``xxx` 是您在 中指定的 HDFS 集群名称`dfs.nameservices`，例如 `"dfs.ha.namenodes.my_ha" = "my_nn"`。

- `dfs.namenode.rpc-address.xxx.nn`：NameNode在HDFS集群中的RPC地址。 `nn` 是您在 中指定的 NameNode 名称， `dfs.ha.namenodes.xxx`例如 `"dfs.namenode.rpc-address.my_ha.my_nn" = "host:port"`。

- `dfs.client.failover.proxy.provider`：客户端将连接到的 NameNode 的提供者。默认值： `org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider`。

例：

```SQL
(
    "dfs.nameservices" = "my-ha",
    "dfs.ha.namenodes.my-ha" = "my-namenode1, my-namenode2",
    "dfs.namenode.rpc-address.my-ha.my-namenode1" = "nn1-host:rpc_port",
    "dfs.namenode.rpc-address.my-ha.my-namenode2" = "nn2-host:rpc_port",
    "dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
)
```

HA 模式可用于简单身份验证或 Kerberos 身份验证。例如，使用简单认证访问高可用模式的HDFS集群，需要指定以下配置：

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

您可以将HDFS集群的配置添加到 **hdfs-site.xml** 文件中。这样，您只需在使用 broker 从 HDFS 集群加载数据时指定文件路径和认证信息即可。

## 9. 如何配置 Hadoop ViewFS 联邦？

将 ViewFs 相关的配置文件 `core-site.xml` 复制到 `hdfs-site.xml` **broker/conf** 目录。

如果您有自定义文件系统，还需要将与文件系统相关的 **.jar** 文件复制到 **broker/lib** 目录下。

## 10. 访问需要Kerberos认证的HDFS集群时，出现“Can't get Kerberos realm”错误怎么办？

检查是否 ** 在部署了代理的所有主机上都配置了** /etc/krb5.conf 文件。

如果错误仍然存在，请添加到 `-Djava.security.krb5.conf:/etc/krb5.conf` 代理启动脚本中变量`JAVA_OPTS`的末尾。
