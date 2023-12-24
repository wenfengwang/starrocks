---
displayed_sidebar: English
---

# 使用 Spark Load 进行批量数据加载

此加载使用外部 Apache Spark™ 资源对导入的数据进行预处理，从而提高导入性能并节省计算资源。主要用于 **初始迁移** 和 **大数据导入** 到 StarRocks（数据量可达TB级）。

Spark load 是一种**异步**导入方法，需要用户通过 MySQL 协议创建 Spark 类型的导入作业，并使用 `SHOW LOAD` 查看导入结果。

> **注意**
>
> - 只有对 StarRocks 表具有 INSERT 权限的用户才能将数据加载到该表中。您可以按照[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)中提供的说明授予所需的权限。
> - Spark Load 不能用于将数据加载到主键表中。

## 术语解释

- **Spark ETL**：主要负责导入过程中数据的ETL，包括全局字典构建（BITMAP类型）、分区、排序、聚合等。
- **Broker**：Broker 是一个独立的无状态进程。它封装了文件系统接口，为 StarRocks 提供了从远程存储系统读取文件的能力。
- **全局字典**：保存将数据从原始值映射到编码值的数据结构。原始值可以是任何数据类型，而编码值是整数。全局字典主要用于预先计算精确计数非重复的场景。

## 背景信息

在 StarRocks v2.4 及之前版本中，Spark Load 依赖 Broker 进程来建立 StarRocks 集群与存储系统的连接。创建 Spark Load 作业时，需要输入 `WITH BROKER "<broker_name>"` 以指定要使用的 Broker。Broker 是一个独立的无状态进程，它与文件系统接口集成在一起。通过 Broker 进程，StarRocks 可以访问和读取存储在存储系统中的数据文件，并可以使用自己的计算资源对这些数据文件的数据进行预处理和加载。

从 StarRocks v2.5 开始，Spark Load 不再需要依赖 Broker 进程来建立 StarRocks 集群和存储系统之间的连接。创建 Spark Load 作业时，不再需要指定 Broker，但仍需要保留 `WITH BROKER` 关键字。

> **注意**
>
> 在某些情况下，不使用 Broker 进程进行加载可能不起作用，例如，当您有多个 HDFS 集群或多个 Kerberos 用户时。在这种情况下，您仍然可以使用 Broker 进程加载数据。

## 基础

用户通过 MySQL 客户端提交 Spark 类型的导入作业；FE 记录元数据并返回提交结果。

火花加载任务的执行分为以下主要阶段。

1. 用户将火花加载作业提交到 FE。
2. FE 调度将 ETL 任务提交到 Apache Spark™ 集群执行。
3. Apache Spark™ 集群执行的 ETL 任务包括全局字典构建（BITMAP 类型）、分区、排序、聚合等。
4. ETL 任务完成后，FE 获取每个预处理切片的数据路径，并调度相关的 BE 执行 Push 任务。
5. BE 通过 Broker 进程从 HDFS 中读取数据，并将其转换为 StarRocks 存储格式。
    > 如果您选择不使用 Broker 进程，则 BE 将直接从 HDFS 读取数据。
6. FE 调度有效版本并完成导入作业。

下图说明了火花负载的主要流程。

![Spark load](../assets/4.3.2-1.png)

---

## 全局字典

### 适用场景

目前 StarRocks 的 BITMAP 列是使用 Roaringbitmap 实现的，只有整数作为输入数据类型。因此，如果要在导入过程中实现 BITMAP 列的预计算，则需要将输入数据类型转换为整数。

在 StarRocks 现有的导入流程中，全局字典的数据结构是基于 Hive 表实现的，保存了从原始值到编码值的映射。

### 构建过程

1. 从上游数据源读取数据，并生成一个临时 Hive 表，命名为 `hive-table`。
2. 提取 `hive-table` 的去重字段的值，生成一个新的 Hive 表，命名为 `distinct-value-table`。
3. 创建一个新的全局字典表，名为 `dict-table`，其中一列用于原始值，一列用于编码值。
4. 在 `distinct-value-table` 和 `dict-table` 之间进行左连接，然后使用窗口函数对此集进行编码。最后，将去重列的原始值和编码值写回 `dict-table`。
5. 在 `dict-table` 和 `hive-table` 之间进行连接，完成将 `hive-table` 中的原始值替换为整数编码值的工作。
6. `hive-table` 将在下次数据预处理时读取，并在计算后导入 StarRocks。

## 数据预处理

数据预处理的基本流程如下：

1. 从上游数据源（HDFS 文件或 Hive 表）读取数据。
2. 完成读取数据的字段映射和计算，然后根据分区信息生成 `bucket-id`。
3. 根据 StarRocks 表的 Rollup 元数据生成 RollupTree。
4. 遍历 RollupTree 并执行分层聚合操作。下一个层次结构的汇总可以从上一个层次结构的汇总计算得出。
5. 每次完成聚合计算后，数据都会被根据 `bucket-id` 进行分桶，然后写入 HDFS。
6. 后续的 Broker 进程将从 HDFS 中拉取文件，并将其导入 StarRocks BE 节点。

## 基本操作

### 先决条件

如果您继续通过 Broker 进程加载数据，则必须确保 StarRocks 集群中部署了 Broker 进程。

您可以使用 [SHOW BROKER](../sql-reference/sql-statements/Administration/SHOW_BROKER.md) 语句检查 StarRocks 集群中部署的 Broker。如果未部署 Broker，则必须按照[部署 Broker](../deployment/deploy_broker.md)中提供的说明部署 Broker。

### 配置 ETL 集群

Apache Spark™ 用作 StarRocks 中的外部计算资源进行 ETL 工作。StarRocks 可能还添加了其他外部资源，例如用于查询的 Spark/GPU、用于外部存储的 HDFS/S3、用于 ETL 的 MapReduce 等。因此，我们引入 `Resource Management` 来管理 StarRocks 使用的这些外部资源。

在提交 Apache Spark™ 导入作业之前，请配置 Apache Spark™ 群集以执行 ETL 任务。操作语法如下：

~~~sql
-- 创建 Apache Spark™ 资源
CREATE EXTERNAL RESOURCE resource_name
PROPERTIES
(
 type = spark,
 spark_conf_key = spark_conf_value,
 working_dir = path,
 broker = broker_name,
 broker.property_key = property_value
);

-- 删除 Apache Spark™ 资源
DROP RESOURCE resource_name;

-- 显示资源
SHOW RESOURCES
SHOW PROC "/resources";

-- 权限
GRANT USAGE_PRIV ON RESOURCE resource_name TO user_identityGRANT USAGE_PRIV ON RESOURCE resource_name TO ROLE role_name;
REVOKE USAGE_PRIV ON RESOURCE resource_name FROM user_identityREVOKE USAGE_PRIV ON RESOURCE resource_name FROM ROLE role_name;
~~~

- 创建资源

**例如**：

~~~sql
-- yarn 集群模式
CREATE EXTERNAL RESOURCE "spark0"
PROPERTIES
(
    "type" = "spark",
    "spark.master" = "yarn",
    "spark.submit.deployMode" = "cluster",
    "spark.jars" = "xxx.jar,yyy.jar",
    "spark.files" = "/tmp/aaa,/tmp/bbb",
    "spark.executor.memory" = "1g",
    "spark.yarn.queue" = "queue0",
    "spark.hadoop.yarn.resourcemanager.address" = "127.0.0.1:9999",
    "spark.hadoop.fs.defaultFS" = "hdfs://127.0.0.1:10000",
    "working_dir" = "hdfs://127.0.0.1:10000/tmp/starrocks",
    "broker" = "broker0",
    "broker.username" = "user0",
    "broker.password" = "password0"
);

-- yarn HA 集群模式
CREATE EXTERNAL RESOURCE "spark1"
PROPERTIES
(
    "type" = "spark",
    "spark.master" = "yarn",
    "spark.submit.deployMode" = "cluster",
    "spark.hadoop.yarn.resourcemanager.ha.enabled" = "true",
    "spark.hadoop.yarn.resourcemanager.ha.rm-ids" = "rm1,rm2",
    "spark.hadoop.yarn.resourcemanager.hostname.rm1" = "host1",
    "spark.hadoop.yarn.resourcemanager.hostname.rm2" = "host2",
    "spark.hadoop.fs.defaultFS" = "hdfs://127.0.0.1:10000",
    "working_dir" = "hdfs://127.0.0.1:10000/tmp/starrocks",
    "broker" = "broker1"
);
~~~

`resource-name` 是 StarRocks 中配置的 Apache Spark™ 资源的名称。

`PROPERTIES` 包括与 Apache Spark™ 资源相关的参数，如下所示：
> **注意**
>

> 有关 Apache Spark™ 资源属性的详细说明，请参见 [CREATE RESOURCE](../sql-reference/sql-statements/data-definition/CREATE_RESOURCE.md)

- Spark相关参数：
  - `type`：资源类型，必填，目前仅支持 `spark`。
  - `spark.master`：必填，目前仅支持 `yarn`。
    - `spark.submit.deployMode`：Apache Spark™ 程序的部署模式，必填，目前支持 `cluster` 和 `client` 两种模式。
    - `spark.hadoop.fs.defaultFS`：如果 master 是 yarn，则为必填项。
    - 与 yarn 资源管理器相关的参数，必填。
      - 单节点上的一个 ResourceManager
        `spark.hadoop.yarn.resourcemanager.address`：单点资源管理器的地址。
      - ResourceManager 高可用性
        >  您可以选择指定 ResourceManager 的主机名或地址。
        - `spark.hadoop.yarn.resourcemanager.ha.enabled`：启用资源管理器 HA，设置为 `true`。
        - `spark.hadoop.yarn.resourcemanager.ha.rm-ids`：资源管理器逻辑 ID 的列表。
        - `spark.hadoop.yarn.resourcemanager.hostname.rm-id`：对于每个 rm-id，指定与资源管理器对应的主机名。
        - `spark.hadoop.yarn.resourcemanager.address.rm-id`：对于每个 rm-id，指定客户端提交作业的 `host:port`。

- `*working_dir`：ETL 使用的目录。如果将 Apache Spark™ 用作 ETL 资源，则为必需。例如： `hdfs://host:port/tmp/starrocks`。

- Broker相关参数：
  - `broker`：Broker 名称。如果将 Apache Spark™ 用作 ETL 资源，则为必填。您需要提前使用 `ALTER SYSTEM ADD BROKER` 命令完成配置。
  - `broker.property_key`：当 Broker 进程读取 ETL 生成的中间文件时需要指定的信息（例如身份验证信息）。

**注意**：

以上是通过 Broker 进程加载的参数说明。如果您打算在没有 Broker 进程的情况下加载数据，则应注意以下几点。

- 您无需指定 `broker`。
- 如果需要配置用户认证，以及NameNode节点的高可用性，则需要在HDFS集群的hdfs-site.xml文件中配置参数，[参数说明请参见](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs)broker_properties。并且需要将每个 FE 的 $FE_HOME/conf 下的 **hdfs-site.xml** 文件移动到每个 BE 的 **$BE_HOME/conf** 下。

> 注意
>
> 如果 HDFS 文件只能由特定用户访问，您仍然需要在 `broker.name` 中指定 HDFS 用户名，在 `broker.password` 中指定用户密码。

- 查看资源

普通账号只能查看自己有权访问的资源 `USAGE-PRIV`。root 和 admin 帐户可以查看所有资源。

- 资源权限

资源权限通过 `GRANT REVOKE` 进行管理，目前仅支持 `USAGE-PRIV` 权限。您可以向用户或角色授予 `USAGE-PRIV` 权限。

~~~sql
-- 向用户0授予对spark0资源的使用权限
GRANT USAGE_PRIV ON RESOURCE "spark0" TO "user0"@"%";

-- 向角色0授予对spark0资源的使用权限
GRANT USAGE_PRIV ON RESOURCE "spark0" TO ROLE "role0";

-- 向用户0授予对所有资源的使用权限
GRANT USAGE_PRIV ON RESOURCE* TO "user0"@"%";

-- 向角色0授予对所有资源的使用权限
GRANT USAGE_PRIV ON RESOURCE* TO ROLE "role0";

-- 撤销用户user0对spark0资源的使用权限
REVOKE USAGE_PRIV ON RESOURCE "spark0" FROM "user0"@"%";
~~~

### 配置 Spark 客户端

配置 FE 的 Spark 客户端，以便 FE 可以通过执行 `spark-submit` 命令提交 Spark 任务。建议使用官方版本的 Spark2 2.4.5 或更高版本     [spark下载地址](https://archive.apache.org/dist/spark/)。下载完成后，请按以下步骤完成配置。

- 配置 `SPARK-HOME`
  
将 Spark 客户端放在与 FE 相同的机器上的目录中，并将 FE 配置文件中的 `spark_home_default_dir` 配置为此目录，默认为 FE 根目录下的 `lib/spark2x` 路径，不能为空。

- **配置 SPARK 依赖包**
  
要配置依赖包，请将 Spark 客户端下的 jars 文件夹中的所有 jar 文件进行压缩和归档，并将 FE 配置中的 `spark_resource_path` 项配置为此 zip 文件。如果此配置为空，FE 将尝试在 FE 根目录中查找 `lib/spark2x/jars/spark-2x.zip` 文件。如果 FE 找不到它，将报错。

提交 spark 加载作业后，归档的依赖文件将上传到远程存储库。默认存储库路径位于 `working_dir/{cluster_id}` 目录下，命名为 `--spark-repository--{resource-name}`，这意味着集群中的资源对应一个远程存储库。目录结构如下：

~~~bash
---spark-repository--spark0/

   |---archive-1.0.0/

   |        |\---lib-990325d2c0d1d5e45bf675e54e44fb16-spark-dpp-1.0.0\-jar-with-dependencies.jar

   |        |\---lib-7670c29daf535efe3c9b923f778f61fc-spark-2x.zip

   |---archive-1.1.0/

   |        |\---lib-64d5696f99c379af2bee28c1c84271d5-spark-dpp-1.1.0\-jar-with-dependencies.jar

   |        |\---lib-1bbb74bb6b264a270bc7fca3e964160f-spark-2x.zip

   |---archive-1.2.0/

   |        |-...

~~~

除了 spark 依赖（默认命名为 `spark-2x.zip`），FE 还会将 DPP 依赖上传到远程存储库。如果 spark load 提交的所有依赖已经存在于远程存储库中，则无需再次上传依赖，节省了每次重复上传大量文件的时间。

### 配置 YARN 客户端

配置 FE 的 yarn 客户端，以便 FE 可以执行 yarn 命令来获取正在运行的应用程序的状态或终止它。建议使用官方版本的 Hadoop2 2.5.2 或更高版本（[hadoop下载地址](https://archive.apache.org/dist/hadoop/common/)）。下载完成后，请按以下步骤完成配置：

- **配置 YARN 可执行文件路径**
  
将下载的 yarn 客户端放在与 FE 相同的机器上的目录中，并将 FE 配置文件中的 `yarn_client_path` 项配置为 yarn 的二进制可执行文件，默认为 FE 根目录下的 `lib/yarn-client/hadoop/bin/yarn` 路径。

- **配置生成 YARN 所需的配置文件的路径（可选）**
  
当 FE 通过 yarn 客户端获取应用状态，或者终止应用时，StarRocks 默认会在 FE 根目录的路径下生成执行 yarn 命令所需的配置文件，`lib/yarn-config`可以通过在 FE 配置文件中配置条目来修改该路径 `yarn_config_dir` ，目前包括 `core-site.xml` 和 `yarn-site.xml`。

### 创建导入作业

**语法：**

~~~sql
LOAD LABEL load_label
    (data_desc, ...)
WITH RESOURCE resource_name 
[resource_properties]
[PROPERTIES (key1=value1, ... )]

* load_label:
    db_name.label_name

* data_desc:
    DATA INFILE ('file_path', ...)
    [NEGATIVE]
    INTO TABLE tbl_name
    [PARTITION (p1, p2)]
    [COLUMNS TERMINATED BY separator ]
    [(col1, ...)]
    [COLUMNS FROM PATH AS (col2, ...)]
    [SET (k1=f1(xx), k2=f2(xx))]
    [WHERE predicate]

    DATA FROM TABLE hive_external_tbl
    [NEGATIVE]
    INTO TABLE tbl_name
    [PARTITION (p1, p2)]
    [SET (k1=f1(xx), k2=f2(xx))]
    [WHERE predicate]

* resource_properties:
 (key2=value2, ...)
~~~

**示例1**：上游数据源为HDFS的情况

~~~sql
LOAD LABEL db1.label1
(
    DATA INFILE("hdfs://abc.com:8888/user/starrocks/test/ml/file1")
    INTO TABLE tbl1
    COLUMNS TERMINATED BY ","
    (tmp_c1,tmp_c2)
    SET
    (
        id=tmp_c2,
        name=tmp_c1
    ),
    DATA INFILE("hdfs://abc.com:8888/user/starrocks/test/ml/file2")
    INTO TABLE tbl2
    COLUMNS TERMINATED BY ","
    (col1, col2)
    where col1 > 1
)
WITH RESOURCE 'spark0'
(
    "spark.executor.memory" = "2g",
    "spark.shuffle.compress" = "true"
)
PROPERTIES
(
    "timeout" = "3600"
);
~~~

**示例2**：上游数据源为Hive的情况。

- 步骤 1：创建新的 hive 资源

~~~sql
CREATE EXTERNAL RESOURCE hive0
PROPERTIES
( 
    "type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:8080"
);
 ~~~

- 步骤 2：创建一个新的 Hive 外部表

~~~sql
CREATE EXTERNAL TABLE hive_t1
(
    k1 INT,
    K2 SMALLINT,
    k3 varchar(50),
    uuid varchar(100)
)
ENGINE=hive
PROPERTIES
( 
    "resource" = "hive0",
    "database" = "tmp",
    "table" = "t1"
);
 ~~~

- 步骤 3：提交加载命令，要求导入的 StarRocks 表中的列存在于 Hive 外部表中。

~~~sql
LOAD LABEL db1.label1
(
    DATA FROM TABLE hive_t1
    INTO TABLE tbl1
    SET
    (
        uuid=bitmap_dict(uuid)
    )
)
WITH RESOURCE 'spark0'
(
    "spark.executor.memory" = "2g",
    "spark.shuffle.compress" = "true"
)
PROPERTIES
(
    "timeout" = "3600"
);
 ~~~

Spark 负载参数介绍：

- **标签**
  
导入作业的标签。每个导入作业都有一个在数据库中唯一的标签，遵循与代理加载相同的规则。

- **数据描述类参数**
  
目前支持的数据源为 CSV 和 Hive 表。其他规则与代理加载相同。

- **导入作业参数**
  
导入作业参数指的是导入语句的 `opt_properties` 部分的参数。这些参数适用于整个导入作业。规则与代理加载相同。

- **Spark 资源参数**
  
需要预先将 Spark 资源配置到 StarRocks 中，并且用户需要获得 USAGE-PRIV 权限后才能将资源应用于 Spark 负载。
当用户有临时需求时，可以设置 Spark 资源参数，例如为作业添加资源、修改 Spark 配置等。该设置仅对该作业生效，不会影响 StarRocks 集群中已有的配置。

~~~sql
WITH RESOURCE 'spark0'
(
    "spark.driver.memory" = "1g",
    "spark.executor.memory" = "3g"
)
~~~

- **数据源为 Hive 时的导入**
  
目前，在导入过程中使用 Hive 表时，需要创建一个该类型的外部表，`Hive`，并在提交导入命令时指定其名称。

- **构建全局词典的导入过程**
  
在 load 命令中，您可以按以下格式指定构建全局字典的必填字段： `StarRocks field name=bitmap_dict(hive table field name)`。请注意，目前**仅当上游数据源为 Hive 表时才支持全局字典**。

- **加载二进制类型数据**

从 v2.5.17 开始，Spark Load 支持 bitmap_from_binary 函数，可以将二进制数据转换为位图数据。如果 Hive 表或 HDFS 文件的列类型为二进制，且 StarRocks 表中对应的列为位图型聚合列，则 load 命令中的字段指定格式如下`StarRocks field name=bitmap_from_binary(Hive table field name)`： .这样就无需构建全局字典。

## 查看导入作业

Spark 负载导入是异步的，代理负载也是如此。用户必须记录导入作业的标签，并在命令中使用该标签 `SHOW LOAD` 来查看导入结果。查看导入的命令是所有导入方法的通用命令。示例如下。

有关返回参数的详细说明，请参阅 Broker Load。区别如下。

~~~sql
mysql> show load order by createtime desc limit 1\G
*************************** 1. row ***************************
  JobId: 76391
  Label: label1
  State: FINISHED
 Progress: ETL:100%; LOAD:100%
  Type: SPARK
 EtlInfo: unselected.rows=4; dpp.abnorm.ALL=15; dpp.norm.ALL=28133376
 TaskInfo: cluster:cluster0; timeout(s):10800; max_filter_ratio:5.0E-5
 ErrorMsg: N/A
 CreateTime: 2019-07-27 11:46:42
 EtlStartTime: 2019-07-27 11:46:44
 EtlFinishTime: 2019-07-27 11:49:44
 LoadStartTime: 2019-07-27 11:49:44
LoadFinishTime: 2019-07-27 11:50:16
  URL: http://1.1.1.1:8089/proxy/application_1586619723848_0035/
 JobDetails: {"ScannedRows":28133395,"TaskNumber":1,"FileNumber":1,"FileSize":200000}
~~~

- **状态**
  
导入作业的当前阶段。
待处理：作业已提交。
ETL：提交 Spark ETL。
LOADING：FE 调度一个 BE 来执行推送操作。
FINISHED：推送完成，版本生效。

导入作业有两个最后阶段 –      `CANCELLED` 和 `FINISHED`，这两个阶段都表示加载作业已完成。 `CANCELLED` 表示导入失败，并 `FINISHED` 指示导入成功。

- **进度**
  
导入作业进度说明。有两种类型的进度 – ETL 和 LOAD，它们对应于导入过程的两个阶段，即 ETL 和 LOADING。

- LOAD 的进度范围为 0~100%。
  
`LOAD 进度 = 当前已完成的所有副本导入的片数 / 该导入作业的总片数 * 100%`。

- 如果所有表都已导入，则 LOAD 进度为 99%，当导入进入最终验证阶段时，将更改为 100%。

- 导入进度不是线性的。如果在一段时间内进度没有变化，并不意味着导入未执行。

- **类型**

 导入作业的类型。SPARK 用于 Spark 负载。

- **创建时间/EtlStartTime/EtlFinishTime/LoadStartTime/LoadFinishTime**

这些值表示导入的创建时间、ETL 阶段的开始时间、ETL 阶段的完成时间、LOADING 阶段的开始时间以及整个导入作业的完成时间。

- **作业详情**

显示作业的详细运行状态，包括导入文件数、总大小（以字节为单位）、子任务数、正在处理的原始行数等。例如：

~~~json
 {"ScannedRows":139264,"TaskNumber":1,"FileNumber":1,"FileSize":940754064}
~~~

- **网址**

您可以将输入复制到浏览器以访问相应应用程序的 Web 界面。

### 查看 Apache Spark™ 启动器提交日志

有时，用户需要查看 Apache Spark™ 作业提交期间生成的详细日志。默认情况下，日志保存在 `log/spark_launcher_log` FE 根目录中的路径为`spark-launcher-{load-job-id}-{label}.log`。日志会在此目录下保存一段时间，当 FE 元数据中的导入信息被清理干净时，日志将被删除。默认保留时间为 3 天。

### 取消导入

当 Spark 负载作业状态不为 `CANCELLED` 或 `FINISHED` 时，用户可以通过指定导入作业的标签手动取消。

---

## 相关系统配置

**FE 配置：** 以下配置是 Spark 负载的系统级配置，适用于所有 Spark 负载导入作业。配置值主要可以通过修改 `fe.conf` 来调整。

- enable-spark-load：启用 Spark 负载和资源创建，默认值为 false。
- spark-load-default-timeout-second：作业的默认超时为 259200 秒（3 天）。
- spark-home-default-dir：Spark 客户端路径（`fe/lib/spark2x`）。
- spark-resource-path：打包的 Spark 依赖文件的路径（默认为空）。
- spark-launcher-log-dir：Spark 客户端提交日志的存放目录（`fe/log/spark-launcher-log`）。
- yarn-client-path：Yarn 二进制可执行文件的路径（`fe/lib/yarn-client/hadoop/bin/yarn`）。
- yarn-config-dir：Yarn 的配置文件路径（`fe/lib/yarn-config`）。

---

## 最佳实践

使用 Spark 负载最适合的场景是原始数据位于文件系统（HDFS）中，数据量在数十 GB 到 TB 级别。对于较小的数据量，请使用 Stream Load 或 Broker Load。

有关完整的 Spark 负载导入示例，请参阅 github 上的演示：[https://github.com/StarRocks/demo/blob/master/docs/03_sparkLoad2StarRocks.md](https://github.com/StarRocks/demo/blob/master/docs/03_sparkLoad2StarRocks.md)

## 常见问题

- `错误：在使用主 'yarn' 运行时，环境中必须设置 HADOOP-CONF-DIR 或 YARN-CONF-DIR。`

 在 Spark 负载中使用时，未在 Spark 客户端的 `spark-env.sh` 中配置 `HADOOP-CONF-DIR` 环境变量。

- `错误：无法运行程序 "xxx/bin/spark-submit"：错误=2，没有那个文件或目录`

 使用 Spark 负载时，`spark_home_default_dir` 配置项未指定 Spark 客户端根目录。

- `错误：文件 xxx/jars/spark-2x.zip 不存在。`

 使用 Spark 负载时，`spark-resource-path` 配置项未指向打包的 zip 文件。

- `错误: yarn client 不存在于路径: xxx/yarn-client/hadoop/bin/yarn`

 在使用 Spark 加载时，yarn-client-path 配置项未指定 yarn 可执行文件。

- `错误: 无法执行 hadoop-yarn/bin/... /libexec/yarn-config.sh`

 在使用 CDH 部署 Hadoop 时，需要配置 `HADOOP_LIBEXEC_DIR` 环境变量。
 由于 `hadoop-yarn` 和 hadoop 目录不同，因此默认的 `libexec` 目录会去查找 `hadoop-yarn/bin/... /libexec`，而实际上 `libexec` 目录位于 hadoop 目录中。
 使用 ''yarn application status'' 命令获取 Spark 任务状态时报错，导致导入作业失败。