---
displayed_sidebar: "Chinese"
---

# Broker Load

## 1. Broker Load是否支持重新运行已成功运行并处于FINISHED状态的加载作业？

Broker Load不支持重新运行已成功运行并处于FINISHED状态的加载作业。此外，为了防止数据丢失和重复，Broker Load不允许重用成功运行的加载作业的标签。您可以使用[SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)查看加载作业的历史并找到要重新运行的加载作业。然后，您可以复制该加载作业的信息，并使用该作业信息（标签除外）创建另一个加载作业。

## 2. 当我使用Broker Load从HDFS加载数据时，如果目标StarRocks表中加载的日期和时间值比源数据文件中的日期和时间值晚8小时，我该怎么办？

目标StarRocks表和Broker Load作业都在创建时根据中国标准时间（CST）时区（使用`timezone`参数指定）编译。但是，服务器设置为基于协调世界时（UTC）时区运行。因此，在数据加载过程中，源数据文件的日期和时间值会额外增加8个小时。为避免此问题，在创建目标StarRocks表时不要指定`timezone`参数。

## 3. 当我使用Broker Load加载ORC格式的数据时，如果出现`ErrorMsg: type:ETL_RUN_FAIL; msg:Cannot cast '<slot 6>' from VARCHAR to ARRAY<VARCHAR(30)>`错误，我该怎么办？

源数据文件的列名与目标StarRocks表不同。在这种情况下，您必须在加载语句中使用`SET`子句来指定文件和表之间的列映射。在执行`SET`子句时，StarRocks需要执行类型推断，但在调用[cast](../../sql-reference/sql-functions/cast.md)函数将源数据转换为目标数据类型时失败。要解决此问题，请确保源数据文件的列名与目标StarRocks表的列名相同。因此，不需要`SET`子句，因此StarRocks不需要调用cast函数执行数据类型转换。然后Broker Load作业可以成功运行。

## 4. Broker Load作业没有报告错误，但为什么我不能查询已加载的数据？

Broker Load是一种异步加载方法。即使加载语句不返回错误，加载作业仍可能失败。运行Broker Load作业后，您可以使用[SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)查看加载作业的结果和`errmsg`。然后，您可以修改作业配置并重试。

## 5. 如果出现"failed to send batch"或"TabletWriter add batch with unknown id"错误，我该怎么办？

写入数据所花费的时间超过了上限，导致超时错误。要解决此问题，请根据业务需求修改[会话变量](../../reference/System_variable.md)`query_timeout`和[BE配置项](../../administration/Configuration.md#configure-be-static-parameters)`streaming_load_rpc_max_alive_time_sec`的设置。

## 6. 如果出现"LOAD-RUN-FAIL; msg:OrcScannerAdapter::init_include_columns. col name = xxx not found"错误，我该怎么办？

如果正在加载Parquet或ORC格式的数据，请检查源数据文件的第一行中的列名是否与目标StarRocks表的列名相同。

```SQL
(tmp_c1,tmp_c2)
SET
(
   id=tmp_c2,
   name=tmp_c1
)
```

上述示例将源数据文件的`tmp_c1`和`tmp_c2`列映射到目标StarRocks表的`name`和`id`列，分别。如果不指定`SET`子句，则使用`column_list`参数中指定的列名来声明列映射。有关更多信息，请参见[BROKER LOAD](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

> **注意**
>
> 如果源数据文件是由Apache Hive™生成的ORC格式文件，并且文件的第一行包含`(_col0, _col1, _col2, ...)`，可能会出现"Invalid Column Name"错误。如果出现此错误，则需要使用`SET`子句来指定列映射。

## 7. 如何处理导致Broker Load作业运行时间过长的错误？

查看FE日志文件**fe.log**，根据作业标签搜索加载作业的ID。然后，查看BE日志文件**be.INFO**，根据作业ID检索加载作业的日志记录，以定位错误的根本原因。

## 8. 如何配置运行在HA模式下的Apache HDFS集群？

如果HDFS集群以高可用性（HA）模式运行，请按以下步骤进行配置：

- `dfs.nameservices`：HDFS集群的名称，例如，`"dfs.nameservices" = "my_ha"`。

- `dfs.ha.namenodes.xxx`：HDFS集群中NameNode的名称。如果指定多个NameNode名称，请用逗号（`,`）分隔。`xxx`表示您在`dfs.nameservices`中指定的HDFS集群名称，例如，`"dfs.ha.namenodes.my_ha" = "my_nn"`。

- `dfs.namenode.rpc-address.xxx.nn`：HDFS集群中NameNode的RPC地址。`nn`是您在`dfs.ha.namenodes.xxx`中指定的NameNode名称，例如，`"dfs.namenode.rpc-address.my_ha.my_nn" = "host:port"`。

- `dfs.client.failover.proxy.provider`：客户端将连接的NameNode的提供者。默认值：`org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider`。

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

HA模式可与简单认证或Kerberos认证配合使用。例如，要使用简单认证访问运行在HA模式下的HDFS集群，需要指定以下配置：

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

您可以将HDFS集群的配置添加到**hdfs-site.xml**文件中。这样，当您使用broker从HDFS集群加载数据时，只需指定文件路径和认证信息即可。

## 9. 如何配置Hadoop ViewFS联邦系统？

将与ViewFs相关的配置文件`core-site.xml`和`hdfs-site.xml`复制到**broker/conf**目录。

如果有自定义文件系统，还需要将文件系统相关的**.jar**文件复制到**broker/lib**目录。

## 10. 当访问需要Kerberos认证的HDFS集群时，如果出现"Can't get Kerberos realm"错误，该怎么办？

检查所有部署broker的主机上是否配置了**/etc/krb5.conf**文件。

如果错误仍然存在，请在broker启动脚本的`JAVA_OPTS`变量末尾添加`-Djava.security.krb5.conf:/etc/krb5.conf`。