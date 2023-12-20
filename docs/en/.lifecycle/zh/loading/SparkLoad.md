---
displayed_sidebar: English
---

# 使用 Spark Load 批量加载数据

此负载使用外部 Apache Spark™ 资源来预处理导入的数据，从而提高导入性能并节省计算资源。它主要用于**初始迁移**和**大数据量导入**到 StarRocks（数据量可达 TB 级别）。

Spark Load 是一种**异步**导入方法，要求用户通过 MySQL 协议创建 Spark 类型的导入作业，并使用 `SHOW LOAD` 查看导入结果。

> **注意**
- 只有对 StarRocks 表具有 INSERT 权限的用户才能将数据加载到该表中。您可以按照 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 中提供的说明来授予所需的权限。
- Spark Load 不能用于将数据加载到主键表中。

## 术语解释

- **Spark ETL**：主要负责导入过程中的数据 ETL，包括全局字典构建（BITMAP 类型）、分区、排序、聚合等。
- **Broker**：Broker 是一个独立的无状态进程。它封装了文件系统接口，为 StarRocks 提供了从远程存储系统读取文件的能力。
- **全局字典**：保存将数据从原始值映射到编码值的数据结构。原始值可以是任何数据类型，而编码值是整数。全局字典主要用于预先计算精确计数不同的场景。

## 背景信息

在 StarRocks v2.4 及更早版本中，Spark Load 依赖 Broker 进程来建立 StarRocks 集群和存储系统之间的连接。创建 Spark Load 作业时，需要输入 `WITH BROKER "<broker_name>"` 来指定要使用的 Broker。Broker 是一个独立的、无状态的进程，与文件系统接口集成。通过 Broker 进程，StarRocks 可以访问和读取存储在您的存储系统中的数据文件，并可以使用自己的计算资源来预处理和加载这些数据文件的数据。

从 StarRocks v2.5 开始，Spark Load 不再需要依赖 Broker 进程来建立 StarRocks 集群和存储系统之间的连接。创建 Spark Load 作业时，不再需要指定 Broker，但仍需要保留 `WITH BROKER` 关键字。

> **注意**
> 在某些情况下，不使用 Broker 进程进行加载可能不起作用，例如当您有多个 HDFS 集群或多个 Kerberos 用户时。在这种情况下，您仍然可以使用 Broker 进程加载数据。

## 基础知识

用户通过 MySQL 客户端提交 Spark 类型的导入作业；FE 记录元数据并返回提交结果。

Spark Load 任务的执行分为以下几个主要阶段：

1. 用户向 FE 提交 Spark Load 作业。
2. FE 安排将 ETL 任务提交到 Apache Spark™ 集群执行。
3. Apache Spark™ 集群执行 ETL 任务，包括全局字典构建（BITMAP 类型）、分区、排序、聚合等。
4. ETL 任务完成后，FE 获取每个预处理分片的数据路径，并调度相关 BE 执行 Push 任务。
5. BE 通过 Broker 进程从 HDFS 读取数据并转换为 StarRocks 存储格式。
      > 如果选择不使用 Broker 进程，则 BE 直接从 HDFS 读取数据。
6. FE 调度生效版本并完成导入作业。

下图说明了 Spark Load 的主要流程。

![Spark load](../assets/4.3.2-1.png)

## 全局字典

### 适用场景

目前，StarRocks 中的 BITMAP 列是使用 Roaringbitmap 实现的，其输入数据类型只有整数。因此，如果您想在导入过程中对 BITMAP 列实现预计算，则需要将输入数据类型转换为整数。

在 StarRocks 现有的导入过程中，全局字典的数据结构是基于 Hive 表实现的，它保存了从原始值到编码值的映射。

### 构建过程

1. 从上游数据源读取数据，生成名为 `hive-table` 的临时 Hive 表。
2. 提取 `hive-table` 中去重字段的值，生成一个新的 Hive 表，名为 `distinct-value-table`。
3. 创建一个新的全局字典表，名为 `dict-table`，其中一列用于原始值，一列用于编码值。
4. 在 `distinct-value-table` 和 `dict-table` 之间进行左连接，然后使用窗口函数对这个集合进行编码。最终，去重列的原始值和编码值都被写回 `dict-table`。
5. 在 `dict-table` 和 `hive-table` 之间进行连接，完成将 `hive-table` 中的原始值替换为整数编码值的工作。
6. `hive-table` 将在下一次数据预处理时被读取，然后在计算完成后导入 StarRocks。

## 数据预处理

数据预处理的基本流程如下：

1. 从上游数据源（HDFS 文件或 Hive 表）读取数据。
2. 对读取的数据完成字段映射和计算，然后根据分区信息生成 `bucket-id`。
3. 根据 StarRocks 表的 Rollup 元数据生成 RollupTree。
4. 遍历 RollupTree 并执行分层聚合操作。下一层次的 Rollup 可以从前一层次的 Rollup 计算得出。
5. 每次聚合计算完成后，根据 `bucket-id` 对数据进行分桶，然后写入 HDFS。
6. 随后的 Broker 进程将从 HDFS 拉取文件并将其导入 StarRocks BE 节点。

## 基本操作

### 先决条件

如果您继续通过 Broker 进程加载数据，必须确保您的 StarRocks 集群中部署了 Broker 进程。

您可以使用 [SHOW BROKER](../sql-reference/sql-statements/Administration/SHOW_BROKER.md) 语句来检查 StarRocks 集群中部署的 Broker。如果没有部署 Broker，必须按照 [部署 Broker](../deployment/deploy_broker.md) 中提供的说明进行部署。

### 配置 ETL 集群

Apache Spark™ 在 StarRocks 中用作 ETL 工作的外部计算资源。StarRocks 中可能还添加了其他外部资源，例如用于查询的 Spark/GPU、用于外部存储的 HDFS/S3、用于 ETL 的 MapReduce 等。因此，我们引入了 `资源管理` 来管理 StarRocks 使用的这些外部资源。

在提交 Apache Spark™ 导入作业之前，配置 Apache Spark™ 集群以执行 ETL 任务。操作语法如下：

```sql
-- 创建 Apache Spark™ 资源
CREATE EXTERNAL RESOURCE resource_name
PROPERTIES
(
 type = 'spark',
 spark_conf_key = 'spark_conf_value',
 working_dir = 'path',
 broker = 'broker_name',
 broker.property_key = 'property_value'
);

-- 删除 Apache Spark™ 资源
DROP RESOURCE resource_name;

-- 展示资源
SHOW RESOURCES
SHOW PROC "/resources";

-- 权限
GRANT USAGE_PRIV ON RESOURCE resource_name TO user_identity;
GRANT USAGE_PRIV ON RESOURCE resource_name TO ROLE role_name;
REVOKE USAGE_PRIV ON RESOURCE resource_name FROM user_identity;
REVOKE USAGE_PRIV ON RESOURCE resource_name FROM ROLE role_name;
```

- 创建资源

**例如**：

```sql
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
```

`resource_name` 是在 StarRocks 中配置的 Apache Spark™ 资源的名称。

`PROPERTIES` 包括与 Apache Spark™ 资源相关的参数，如下所示：
```
> **注意**
> 有关Apache Spark™资源**属性**的详细说明，请参阅[CREATE RESOURCE](../sql-reference/sql-statements/data-definition/CREATE_RESOURCE.md)

- Spark 相关参数：
  - `type`：资源类型，必填，目前仅支持 `spark`。
  - `spark.master`：必填，目前仅支持 `yarn`。
    - `spark.submit.deployMode`：Apache Spark™ 程序的部署模式，必填，目前支持 `cluster` 和 `client`。
    - `spark.hadoop.fs.defaultFS`：如果 master 是 yarn，则需要。
    - 与 YARN 资源管理器相关的参数，必填。
      - 单个节点上的一个 ResourceManager
        - `spark.hadoop.yarn.resourcemanager.address`：单点资源管理器的地址。
      - ResourceManager HA
        > 您可以选择指定 ResourceManager 的主机名或地址。
        - `spark.hadoop.yarn.resourcemanager.ha.enabled`：启用资源管理器 HA，设置为 `true`。
        - `spark.hadoop.yarn.resourcemanager.ha.rm-ids`：资源管理器逻辑 ID 列表。
        - `spark.hadoop.yarn.resourcemanager.hostname.rm-id`：对于每个 rm-id，指定与资源管理器对应的主机名。
        - `spark.hadoop.yarn.resourcemanager.address.rm-id`：对于每个 rm-id，指定客户端要向其提交作业的 `host:port`。

- `working_dir`：ETL 使用的目录。如果 Apache Spark™ 用作 ETL 资源，则为必需。例如：`hdfs://host:port/tmp/starrocks`。

- Broker 相关参数：
  - `broker`：Broker 名称。如果 Apache Spark™ 用作 ETL 资源，则为必需。需要使用 `ALTER SYSTEM ADD BROKER` 命令提前完成配置。
  - `broker.property_key`：Broker 进程读取 ETL 生成的中间文件时需要指定的信息（例如认证信息）。

**注意**：

以上是通过 Broker 进程加载的参数说明。如果您打算在不使用 Broker 进程的情况下加载数据，则应注意以下事项。

- 您不需要指定 `broker`。
- 如果需要配置用户认证、NameNode 节点 HA 等，需要在 HDFS 集群的 `hdfs-site.xml` 文件中配置参数，参数说明参见 [broker_properties](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs)。您需要将每个 FE 的 `hdfs-site.xml` 文件移动到 **$FE_HOME/conf** 下，将每个 BE 的 `hdfs-site.xml` 文件移动到 **$BE_HOME/conf** 下。

> 注意
> 如果 HDFS 文件只能由特定用户访问，您仍然需要在 `broker.name` 中指定 HDFS 用户名，在 `broker.password` 中指定用户密码。

- 查看资源

普通账户只能查看他们具有 `USAGE_PRIV` 访问权限的资源。root 和 admin 账户可以查看所有资源。

- 资源权限

资源权限通过 `GRANT` 和 `REVOKE` 进行管理，目前仅支持 `USAGE_PRIV` 权限。您可以向用户或角色授予 `USAGE_PRIV` 权限。

```sql
-- 授予用户 user0 对资源 spark0 的访问权限
GRANT USAGE_PRIV ON RESOURCE "spark0" TO "user0"@"%";

-- 授予角色 role0 对资源 spark0 的访问权限
GRANT USAGE_PRIV ON RESOURCE "spark0" TO ROLE "role0";

-- 授予用户 user0 对所有资源的访问权限
GRANT USAGE_PRIV ON RESOURCE* TO "user0"@"%";

-- 授予角色 role0 对所有资源的访问权限
GRANT USAGE_PRIV ON RESOURCE* TO ROLE "role0";

-- 撤销用户 user0 对资源 spark0 的使用权限
REVOKE USAGE_PRIV ON RESOURCE "spark0" FROM "user0"@"%";
```

### 配置 Spark 客户端

为 FE 配置 Spark 客户端，使后者可以通过执行 `spark-submit` 命令提交 Spark 任务。建议使用官方版本的 Spark 2.4.5 或以上版本     [Spark 下载地址](https://archive.apache.org/dist/spark/)。下载后，请按照以下步骤完成配置。

- 配置 `SPARK_HOME`

将 Spark 客户端放置在与 FE 同一机器的目录下，并将 FE 配置文件中的 `spark_home_default_dir` 配置为该目录，该目录默认为 FE 根目录下的 `lib/spark2x` 路径，且不能为空。

- **配置 SPARK 依赖包**

配置依赖包需要将 Spark 客户端下 jars 文件夹中的所有 jar 文件进行 zip 压缩并归档，并将 FE 配置中的 `spark_resource_path` 项配置为该 zip 文件。如果此配置为空，FE 将尝试在 FE 根目录中查找 `lib/spark2x/jars/spark-2x.zip` 文件。如果 FE 找不到，就会报错。

当提交 spark load 作业时，归档的依赖文件将被上传到远程存储库。默认存储库路径位于 `working_dir/{cluster_id}` 目录下，以 `--spark-repository--{resource-name}` 命名，这意味着集群中的资源对应于一个远程存储库。引用的目录结构如下：

```bash
---spark-repository--spark0/

   |---archive-1.0.0/

   |        |\---lib-990325d2c0d1d5e45bf675e54e44fb16-spark-dpp-1.0.0\-jar-with-dependencies.jar

   |        |\---lib-7670c29daf535efe3c9b923f778f61fc-spark-2x.zip

   |---archive-1.1.0/

   |        |\---lib-64d5696f99c379af2bee28c1c84271d5-spark-dpp-1.1.0\-jar-with-dependencies.jar

   |        |\---lib-1bbb74bb6b264a270bc7fca3e964160f-spark-2x.zip

   |---archive-1.2.0/

   |        |-...

```

除了 spark 依赖项（默认命名为 `spark-2x.zip`）之外，FE 还会将 DPP 依赖项上传到远程存储库。如果 spark load 提交的所有依赖都已经存在于远程仓库中，则不需要再次上传依赖，节省了每次重复上传大量文件的时间。

### 配置 YARN 客户端

为 FE 配置 yarn 客户端，以便 FE 可以执行 yarn 命令来获取正在运行的应用程序的状态或终止它。建议使用官方版本的 Hadoop 2.5.2 或以上版本（[Hadoop 下载地址](https://archive.apache.org/dist/hadoop/common/)）。下载后，请按照以下步骤完成配置：

- **配置 YARN 可执行路径**

将下载好的 yarn 客户端放在与 FE 同一台机器的目录下，并将 FE 配置文件中的 `yarn_client_path` 项配置为 yarn 的二进制可执行文件，默认为 FE 根目录下的 `lib/yarn-client/hadoop/bin/yarn` 路径。

- **配置生成 YARN 所需配置文件的路径（可选）**

当 FE 通过 yarn 客户端获取应用程序的状态或终止应用程序时，默认情况下 StarRocks 会在 FE 根目录的 `lib/yarn-config` 路径下生成执行 yarn 命令所需的配置文件。这个路径可以通过配置 FE 配置文件中的 `yarn_config_dir` 条目来修改，当前包括 `core-site.xml` 和 `yarn-site.xml`。

### 创建导入作业

**语法**：

```sql
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
    [COLUMNS TERMINATED BY separator]
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
```

**示例 1**：上游数据源为 HDFS 的情况

```sql
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
    WHERE col1 > 1
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
```

**示例 2**：上游数据源为 Hive 的情况。

- 步骤 1：创建新的 Hive 资源

```sql
CREATE EXTERNAL RESOURCE hive0
PROPERTIES
( 
    "type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:8080"
);
```
- 步骤2：创建新的Hive外部表

```sql
CREATE EXTERNAL TABLE hive_t1
(
    k1 INT,
    k2 SMALLINT,
    k3 VARCHAR(50),
    uuid VARCHAR(100)
)
ENGINE=hive
PROPERTIES
( 
    "resource" = "hive0",
    "database" = "tmp",
    "table" = "t1"
);
```

- 步骤3：提交load命令，要求导入StarRocks表的列在hive外部表中存在。

```sql
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
```

Spark Load中的参数介绍：

- **标签**

导入作业的标签。每个导入作业都有一个在数据库中唯一的标签，遵循与Broker Load相同的规则。

- **数据描述类参数**

目前支持的数据源有CSV和Hive表。其他规则与Broker Load相同。

- **导入作业参数**

导入作业参数是指属于导入语句的`opt_properties`部分的参数。这些参数适用于整个导入作业。规则与Broker Load相同。

- **Spark资源参数**

Spark资源需要提前配置到StarRocks中，并且需要赋予用户USAGE_PRIV权限，然后才能将资源应用到Spark Load。当用户有临时需要时，例如为某个作业添加资源和修改Spark配置，可以设置Spark资源参数。该设置仅对该作业生效，不会影响StarRocks集群中的现有配置。

```sql
WITH RESOURCE 'spark0'
(
    "spark.driver.memory" = "1g",
    "spark.executor.memory" = "3g"
)
```

- **当数据源为Hive时导入**

目前，导入过程中使用Hive表需要先创建一个Hive类型的外部表，然后在提交导入命令时指定其名称。

- **构建全局字典的导入流程**

在load命令中，可以指定构建全局字典所需的字段，格式如下：`StarRocks字段名=bitmap_dict(hive表字段名)`。注意，目前**全局字典仅支持上游数据源为Hive表时**。

- **加载二进制类型数据**

从v2.5.17开始，Spark Load支持bitmap_from_binary函数，可以将二进制数据转换为bitmap数据。如果Hive表或HDFS文件的列类型为二进制，而StarRocks表中对应的列为bitmap类型的聚合列，可以在load命令中指定字段，格式如下：`StarRocks字段名=bitmap_from_binary(Hive表字段名)`。这样就不需要构建全局字典。

## 查看导入作业

Spark Load导入是异步的，与Broker Load相同。用户必须记录导入作业的标签，并在`SHOW LOAD`命令中使用它来查看导入结果。查看导入的命令对于所有导入方法都是通用的。示例如下。

返回参数的详细解释请参见Broker Load。区别如下。

```sql
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
```

- **状态**

导入作业的当前阶段。PENDING：作业已提交。ETL：Spark ETL已提交。LOADING：FE调度BE执行推送操作。FINISHED：推送完成，版本生效。

导入作业有两个最终阶段 - `CANCELLED`和`FINISHED`，均表示加载作业已完成。`CANCELLED`表示导入失败，`FINISHED`表示导入成功。

- **进度**

描述导入作业进度。进度有两种类型 - ETL和LOAD，分别对应导入过程的两个阶段，ETL和LOADING。

- LOAD的进度范围为0~100%。

`LOAD进度 = 当前完成的所有副本导入的Tablet数量 / 本次导入作业的Tablet总数 * 100%`。

- 如果所有表都已导入，则LOAD进度为99%，当导入进入最后验证阶段时，进度变为100%。

- 导入进度不是线性的。如果一段时间内进度没有变化，并不意味着导入没有执行。

- **类型**

导入作业的类型。SPARK表示Spark Load。

- **CreateTime/EtlStartTime/EtlFinishTime/LoadStartTime/LoadFinishTime**

这些值分别代表创建导入的时间、ETL阶段开始的时间、ETL阶段完成的时间、LOADING阶段开始的时间以及整个导入作业完成的时间。

- **JobDetails**

显示作业的详细运行状态，包括导入的文件数、总大小（以字节为单位）、子任务数、正在处理的原始行数等。例如：

```json
 {"ScannedRows":139264,"TaskNumber":1,"FileNumber":1,"FileSize":940754064}
```

- **URL**

您可以将输入复制到浏览器中以访问相应应用程序的Web界面。

### 查看 Apache Spark™ 启动器提交日志

有时，用户需要查看Apache Spark™作业提交期间生成的详细日志。默认情况下，日志保存在FE根目录下的`log/spark_launcher_log`路径中，命名为`spark-launcher-{load-job-id}-{label}.log`。日志会在此目录中保存一段时间，并在清理FE元数据中的导入信息时被删除。默认保留时间为3天。

### 取消导入

当Spark Load作业状态不是`CANCELLED`或`FINISHED`时，用户可以通过指定导入作业的Label来手动取消。

## 相关系统配置

**FE配置：**以下配置是Spark Load的系统级配置，适用于所有Spark Load导入作业。主要通过修改`fe.conf`来调整配置值。

- enable_spark_load：启用Spark Load和资源创建，默认值为false。
- spark_load_default_timeout_second：作业的默认超时时间为259200秒（3天）。
- spark_home_default_dir：Spark客户端路径（`fe/lib/spark2x`）。
- spark_resource_path：打包后的Spark依赖文件的路径（默认为空）。
- spark_launcher_log_dir：Spark客户端提交日志的存放目录（`fe/log/spark_launcher_log`）。
- yarn_client_path：yarn二进制可执行文件的路径（`fe/lib/yarn-client/hadoop/bin/yarn`）。
- yarn_config_dir：Yarn的配置文件路径（`fe/lib/yarn-config`）。

## 最佳实践

最适合使用Spark Load的场景是原始数据位于文件系统（HDFS）且数据量在数十GB到TB级别时。对于较小的数据量，请使用Stream Load或Broker Load。

完整的Spark Load导入示例，请参考GitHub上的演示：[https://github.com/StarRocks/demo/blob/master/docs/03_sparkLoad2StarRocks.md](https://github.com/StarRocks/demo/blob/master/docs/03_sparkLoad2StarRocks.md)

## 常见问题解答

- 错误：使用master 'yarn'运行时，必须在环境中设置HADOOP_CONF_DIR或YARN_CONF_DIR。

使用Spark Load时未在Spark客户端的`spark-env.sh`中配置`HADOOP_CONF_DIR`环境变量。

- 错误：无法运行程序“xxx/bin/spark-submit”：错误=2，没有这样的文件或目录

在使用Spark Load时，`spark_home_default_dir`配置项没有指定Spark客户端根目录。

- 错误：文件xxx/jars/spark-2x.zip不存在。

使用Spark Load时，`spark_resource_path`配置项没有指向打包的zip文件。

- 错误：yarn客户端不存在于路径：xxx/yarn-client/hadoop/bin/yarn

使用Spark Load时，`yarn_client_path`配置项没有指定yarn可执行文件。

- 错误：无法执行hadoop-yarn/bin/.../libexec/yarn-config.sh

使用CDH版本的Hadoop时，需要配置`HADOOP_LIBEXEC_DIR`环境变量。
由于`hadoop-yarn`和hadoop目录不同，默认的`libexec`目录会在`hadoop-yarn/bin/.../libexec`中查找，而`libexec`实际上在hadoop目录中。
执行`yarn application -status`命令获取Spark任务状态时报错，导致导入作业失败。
- 错误：在路径中找不到 yarn 客户端：xxx/yarn-client/hadoop/bin/yarn

使用 Spark Load 时，`yarn-client-path` 配置项没有指定 yarn 的可执行文件路径。

- 错误：无法执行 hadoop-yarn/bin/.../libexec/yarn-config.sh
