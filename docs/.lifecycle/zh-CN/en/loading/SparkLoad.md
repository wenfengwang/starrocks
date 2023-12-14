---
displayed_sidebar: "Chinese"
---

# 使用 Spark Load 批量加载数据

使用此加载方式，可以利用外部 Apache Spark™ 资源对导入的数据进行预处理，以提高导入性能并节省计算资源。主要用于将数据首次迁移和大规模数据导入到 StarRocks（数据量达到 TB 级别）。

Spark load 是一种 **异步** 导入方法，需要用户通过 MySQL 协议创建 Spark 类型的导入作业，并使用 `SHOW LOAD` 查看导入结果。

> **注意**
>
> - 只有对 StarRocks 表具有 INSERT 权限的用户才能向该表加载数据。您可以按照 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) 中提供的说明授予所需的权限。
> - Spark Load 不能用于将数据加载到主键表中。

## 术语解释

- **Spark ETL**：主要负责导入过程中的数据ETL，包括全局字典构建（BITMAP 类型）、分区、排序、聚合等。
- **Broker**：Broker 是一个独立的无状态进程。它封装了文件系统接口，并为 StarRocks 提供了从远程存储系统读取文件的能力。
- **全局字典**：存储将数据从原始值映射到编码值的数据结构。原始值可以是任何数据类型，而编码值是整数。全局字典主要用于提前计算准确的去重数目。

## 背景信息

在 StarRocks v2.4 版本及更早版本中，Spark Load 依赖于 Broker 进程来在您的 StarRocks 集群和存储系统之间建立连接。创建 Spark Load 作业时，您需要输入 `WITH BROKER "<broker_name>"` 来指定要使用的 Broker。Broker 是一个独立的、无状态的进程，集成了一个文件系统接口。通过 Broker 进程，StarRocks 可以访问并读取存储在您的存储系统中的数据文件，并可以利用自己的计算资源对这些数据文件的数据进行预处理和加载。

从 StarRocks v2.5 版本开始，Spark Load 不再需要依赖 Broker 进程来在您的 StarRocks 集群和存储系统之间建立连接。创建 Spark Load 作业时，您不再需要指定 Broker，但仍然需要保留 `WITH BROKER` 关键字。

> **注意**
>
> 在某些情况下，不使用 Broker 进程进行加载可能不起作用，例如当您拥有多个 HDFS 集群或多个 Kerberos 用户时。在这种情况下，您仍然可以通过使用 Broker 进程来加载数据。

## 基础知识

用户通过 MySQL 客户端提交 Spark 类型的导入作业；前端记录元数据并返回提交结果。

Spark load 任务的执行分为以下主要阶段。

1. 用户提交 Spark load 作业至前端。
2. 前端调度 ETL 任务提交至 Apache Spark™ 集群执行。
3. Apache Spark™ 集群执行包括全局字典构建（BITMAP 类型）、分区、排序、聚合等的 ETL 任务。
4. ETL 任务完成后，前端获取每个预处理分片的数据路径，并调度相关的后端来执行 Push 任务。
5. 后端通过 Broker 进程从 HDFS 读取数据并将其转换为 StarRocks 存储格式。
    > 如果选择不使用 Broker 进程，后端将直接从 HDFS 读取数据。
6. 前端调度有效版本并完成导入作业。

以下图表展示了 Spark load 的主要流程。

![Spark load](../assets/4.3.2-1.png)

---

## 全局字典

### 适用场景

目前，StarRocks 中的 BITMAP 列使用 Roaringbitmap 实现，仅支持整数作为输入数据类型。因此，如果您希望在导入过程中对 BITMAP 列进行预计算，则需要将输入数据类型转换为整数。

在 StarRocks 现有的导入过程中，全局字典的数据结构基于 Hive 表实现，保存了原始值到编码值的映射。

### 构建过程

1. 从上游数据源中读取数据并生成临时 Hive 表，命名为 `hive-table`。
2. 提取 `hive-table` 的去重字段的值，生成名为 `distinct-value-table` 的新 Hive 表。
3. 创建名为 `dict-table` 的新全局字典表，包含一个用于原始值和一个用于编码值的列。
4. 在 `distinct-value-table` 和 `dict-table` 之间进行左连接，然后使用窗口函数对此集合进行编码。最终将去重列的原始值和编码值都重新写入 `dict-table`。
5. 将 `dict-table` 与 `hive-table` 进行连接，完成对 `hive-table` 中原始值的替换工作。
6. `hive-table` 将在下一次数据预处理时被读取，之后将经过计算导入到 StarRocks 中。

## 数据预处理

数据预处理的基本流程如下：

1. 从上游数据源（HDFS 文件或 Hive 表）读取数据。
2. 对读取的数据完成字段映射和计算，然后基于分区信息生成 `bucket-id`。
3. 基于 StarRocks 表的 Rollup 元数据生成 RollupTree。
4. 遍历 RollupTree 并执行分层聚合操作。下一级别的 Rollup 可以从前一级别的 Rollup 计算得出。
5. 每次完成聚合计算后，根据 `bucket-id` 对数据进行分桶，然后写入到 HDFS。
6. 随后的 Broker 进程将从 HDFS 拉取文件，并将其导入到 StarRocks BE 节点。

## 基本操作

### 先决条件

如果您继续通过 Broker 进程加载数据，则必须确保在您的 StarRocks 集群中已部署 Broker 进程。

您可以使用 [SHOW BROKER](../sql-reference/sql-statements/Administration/SHOW_BROKER.md) 语句来检查已部署在您的 StarRocks 集群中的 Broker。如果没有部署 Broker，您需要按照 [部署 Broker](../deployment/deploy_broker.md) 中提供的说明来部署 Broker。

### 配置 ETL 集群

Apache Spark™ 被用作 StarRocks 中 ETL 工作的外部计算资源。其他外部资源可能被添加到 StarRocks，如用于查询的 Spark/GPU、用于外部存储的 HDFS/S3、用于 ETL 的 MapReduce 等。因此，我们引入`资源管理`来管理 StarRocks 使用的这些外部资源。

在提交 Apache Spark™ 导入作业之前，配置 Apache Spark™ 集群以执行 ETL 任务。操作的语法如下所示：

~~~sql
-- 创建资源
CREATE EXTERNAL RESOURCE resource_name
PROPERTIES
(
 type = spark,
 spark_conf_key = spark_conf_value,
 working_dir = path,
 broker = broker_name,
 broker.property_key = property_value
);

-- 删除资源
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

其中，`resource-name` 是在 StarRocks 中配置的 Apache Spark™ 资源的名称。

`PROPERTIES` 包括与 Apache Spark™ 资源相关的参数，示例如下：
> **注意**
>
> 有关Apache Spark™资源属性的详细描述，请参阅[CREATE RESOURCE](../sql-reference/sql-statements/data-definition/CREATE_RESOURCE.md)

- Spark相关参数：
  - `type`：资源类型，必需，目前仅支持`spark`。
  - `spark.master`：必需，目前仅支持`yarn`。
    - `spark.submit.deployMode`：Apache Spark™程序的部署模式，必需，目前支持`cluster`和`client`。
    - `spark.hadoop.fs.defaultFS`：如果master是yarn，则必需。
    - 与yarn资源管理器相关的参数，必需。
      - 单节点上的一个ResourceManager
        `spark.hadoop.yarn.resourcemanager.address`：单点资源管理器的地址。
      - ResourceManager HA
        > 您可以选择指定ResourceManager的主机名或地址。
        - `spark.hadoop.yarn.resourcemanager.ha.enabled`：启用资源管理器HA，设置为`true`。
        - `spark.hadoop.yarn.resourcemanager.ha.rm-ids`：资源管理器逻辑ID列表。
        - `spark.hadoop.yarn.resourcemanager.hostname.rm-id`：对于每个rm-id，指定与资源管理器对应的主机名。
        - `spark.hadoop.yarn.resourcemanager.address.rm-id`：对于每个rm-id，指定用于客户端提交作业的`host:port`。

- `*working_dir`：ETL使用的目录。如果将Apache Spark™用作ETL资源，则需要此项，例如：`hdfs://host:port/tmp/starrocks`。

- Broker相关参数：
  - `broker`：Broker名称。如果将Apache Spark™用作ETL资源，则需要此项。您需要使用`ALTER SYSTEM ADD BROKER`命令提前完成配置。
  - `broker.property_key`：当Broker进程读取ETL生成的中间文件时，需要指定的信息（例如身份验证信息）。

**注意**：

以上是通过Broker进程加载参数的描述。如果您打算在没有Broker进程的情况下加载数据，则应注意以下事项。

- 您不需要指定`broker`。
- 如果需要配置用户认证以及NameNode节点的HA，则需要在HDFS集群的hdfs-site.xml文件中配置参数，有关参数的描述，请参见[broker_properties](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md#hdfs)，您还需要将**hdfs-site.xml**文件移动到每个FE的**$FE_HOME/conf**以及每个BE的**$BE_HOME/conf**下。

> **注意**
>
> 如果HDFS文件只能由特定用户访问，则仍然需要在`broker.name`中指定HDFS用户名，并在`broker.password`中指定用户密码。

- 查看资源

普通账户只能查看其具有`USAGE-PRIV`访问权限的资源。根账户和管理员账户可以查看所有资源。

- 资源权限

资源权限通过`GRANT REVOKE`进行管理，目前仅支持`USAGE-PRIV`权限。您可以为用户或角色分配`USAGE-PRIV`权限。

~~~sql
-- 将spark0资源的使用权限授权给用户user0
GRANT USAGE_PRIV ON RESOURCE "spark0" TO "user0"@"%";

-- 将spark0资源的使用权限授权给角色role0
GRANT USAGE_PRIV ON RESOURCE "spark0" TO ROLE "role0";

-- 将所有资源的使用权限授权给用户user0
GRANT USAGE_PRIV ON RESOURCE* TO "user0"@"%";

-- 将所有资源的使用权限授权给角色role0
GRANT USAGE_PRIV ON RESOURCE* TO ROLE "role0";

-- 从用户user0中撤销对spark0资源的使用权限
REVOKE USAGE_PRIV ON RESOURCE "spark0" FROM "user0"@"%";
~~~

### 配置Spark客户端

配置FE的Spark客户端，以便后者可以通过执行`spark-submit`命令提交Spark任务。建议使用官方版本的Spark2 2.4.5或更高版本 [spark下载地址](https://archive.apache.org/dist/spark/)。下载后，请按以下步骤完成配置。

- 配置`SPARK-HOME`
  
将Spark客户端放置在与FE相同的机器上的目录中，并将FE配置文件中的`spark_home_default_dir`配置为此目录，默认情况下为FE根目录中的`lib/spark2x`路径，并且不得为空。

- **配置SPARK依赖包**
  
要配置依赖包，请将Spark客户端下jars文件夹中的所有jar文件压缩并归档，并将FE配置中的`spark_resource_path`项配置为此zip文件。如果此配置为空，则FE将尝试在FE根目录中找到`lib/spark2x/jars/spark-2x.zip`文件。如果FE找不到它，将报告错误。

当提交spark加载作业时，归档的依赖文件将上传到远程存储库。默认存储库路径位于`working_dir/{cluster_id}`目录下，其名称为`--spark-repository--{resource-name}`，这意味着集群中的一个资源对应远程存储库。目录结构的引用如下：

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

除了Spark依赖（默认命名为`spark-2x.zip`）外，FE还会将DPP依赖上传到远程存储库。如果spark加载提交的所有依赖在远程存储库中已经存在，则无需再次上传依赖，可以节省再次上传大量文件的时间。

### 配置YARN客户端

为FE配置yarn客户端，以便FE可以执行yarn命令来获取运行应用程序的状态或终止应用程序。建议使用官方版本的Hadoop2 2.5.2或更高版本 ([hadoop下载地址](https://archive.apache.org/dist/hadoop/common/))。下载后，请按以下步骤完成配置：

- **配置YARN可执行路径**
  
将下载的yarn客户端放置在与FE相同的机器上的目录中，并将FE配置文件中的`yarn_client_path`配置为yarn的二进制可执行文件的路径，默认情况下为FE根目录中的`lib/yarn-client/hadoop/bin/yarn`路径。

- **配置生成YARN所需的配置文件路径（可选）**
  
当FE通过yarn客户端获取应用程序的状态或终止应用程序时，默认情况下，StarRocks会在FE根目录的`lib/yarn-config`路径下生成执行yarn命令所需的配置文件，此路径可通过FE配置文件中的`yarn_config_dir`条目进行修改，当前包括`core-site.xml`和`yarn-site.xml`。

### 创建导入作业

**语法:**

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
```
"spark.executor.memory" = "2g",
"spark.shuffle.compress" = "true"
)

PROPERTIES
(
"timeout" = "3600"
);
~~~

**例 2**: 上游数据源为Hive。

- 步骤1: 创建新的Hive资源

~~~sql
CREATE EXTERNAL RESOURCE hive0
PROPERTIES
( 
"type" = "hive",
"hive.metastore.uris" = "thrift://0.0.0.0:8080"
);
 ~~~

- 步骤2: 创建新的Hive外部表

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

- 步骤3: 提交加载命令，要求导入的StarRocks表中的列存在于Hive外部表中。

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

Spark加载参数简介:

- **Label**
  
导入作业的标签。每个导入作业都有一个唯一的标签，在数据库内是唯一的，遵循与broker加载相同的规则。

- **数据描述类参数**
  
当前支持的数据源是CSV和Hive表。其他规则与broker加载相同。

- **导入作业参数**
  
导入作业参数是指导入语句的`opt_properties`部分的参数。这些参数适用于整个导入作业。规则与broker加载相同。

- **Spark资源参数**
  
Spark资源需要提前配置到StarRocks中，用户在将资源应用于Spark加载之前需要给予使用权限。
当用户有临时需求时，例如为作业添加资源和修改Spark配置时，可以设置Spark资源参数。该设置仅对此作业生效，不影响StarRocks集群中的现有配置。

~~~sql
WITH RESOURCE 'spark0'
(
"spark.driver.memory" = "1g",
"spark.executor.memory" = "3g"
)
~~~

- **数据源为Hive时的导入**
  
目前，要在导入进程中使用Hive表，需要创建`Hive`类型的外部表，然后在提交导入命令时指定其名称。

- **构建全局字典的导入过程**
  
在加载命令中，可以按以下格式指定用于构建全局字典的字段: `StarRocks字段名称=bitmap_dict(hive表字段名称)` 目前**仅支持在上游数据源为Hive表时构建全局字典**。

- **位图二进制类型列的导入**

适用于StarRocks表聚合列的数据类型为位图类型，数据源Hive表或HDFS文件类型中相应列的数据类型为二进制（通过FE类序列化中的spark-dpp中的com.starrocks.load.loadv2.dpp.BitmapValue类型）。

无需构建全局字典，只需在加载命令中指定相应字段。格式为: ```StarRocks字段名称=bitmap_from_binary(Hive表字段名称)```

## 查看导入作业

Spark加载导入是异步的, 与Broker加载相同. 用户必须记录导入作业的标签，并在`SHOW LOAD`命令中使用它以查看导入结果。查看导入的命令对所有导入方法都是通用的。示例如下。

关于返回参数的详细解释，请参考Broker加载。不同之处如下。

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

- **State**
  
导入作业的当前阶段。
PENDING: 作业已提交。
ETL: Spark ETL已提交。
LOADING: FE调度BE执行推送操作。
FINISHED: 推送已完成，版本生效。

导入作业有两个最终阶段-`CANCELLED`和`FINISHED`，都表示加载作业已完成。`CANCELLED`表示导入失败，`FINISHED`表示导入成功。

- **Progress**
  
导入作业进度的描述。进度有两种类型-ETL和LOAD，对应于导入过程的两个阶段，ETL和LOADING。

LOAD的进度范围是0~100%。
`LOAD进度=导入作业的所有副本导入的当前已完成的tablet数/此导入作业的总tablet数*100%`。

如果所有表都已导入，则LOAD进度为99%，当导入进入最终验证阶段时变为100%。

导入进度是不线性的。如果一段时间内进度没有改变，并不意味着导入未执行。

- **Type**

 导入作业的类型。Spark加载的类型为SPARK。

- **CreateTime/EtlStartTime/EtlFinishTime/LoadStartTime/LoadFinishTime**

这些值表示导入作业创建时的时间，ETL阶段开始时的时间，ETL阶段完成时的时间，LOADING阶段开始时的时间以及整个导入作业完成时的时间。

- **JobDetails**

显示作业的详细运行状态，包括导入的文件数量、总大小（以字节为单位）、子任务数、正在处理的原始行数等。例如：

~~~json
 {"ScannedRows":139264,"TaskNumber":1,"FileNumber":1,"FileSize":940754064}
~~~

- **URL**

用户可以将其复制到浏览器中以访问相应应用的Web界面。

### 查看Apache Spark™ Launcher提交日志

有时用户需要查看Apache Spark™作业提交期间生成的详细日志。默认情况下，日志保存在FE根目录中的路径`log/spark_launcher_log`，命名为`spark-launcher-{load-job-id}-{label}.log`。这些日志在此目录中保存一段时间，并且在FE元数据中的导入信息被清理时将被删除。默认保留时间为3天。

### 取消导入

当Spark加载作业状态不为`CANCELLED`或`FINISHED`时，用户可以通过指定导入作业的标签手动取消导入。

---

## 相关系统配置

**FE配置:** 以下配置是Spark加载的系统级配置，适用于所有Spark加载导入作业。主要通过修改`fe.conf`来调整配置值。

- enable-spark-load: 启用Spark加载和资源创建，默认值为false。
- spark-load-default-timeout-second: 作业的默认超时时间为259200秒（3天）。
- spark-home-default-dir: Spark客户端路径（`fe/lib/spark2x`）。
- spark-resource-path: 打包的Spark依赖文件路径（默认为空）。
- spark-launcher-log-dir: 存储Spark客户端提交日志的目录（`fe/log/spark-launcher-log`）。
- yarn-client-path: Yarn二进制可执行文件路径（`fe/lib/yarn-client/hadoop/bin/yarn`）。
- yarn-config-dir: Yarn的配置文件路径（`fe/lib/yarn-config`）。

---

## 最佳实践

Spark加载最适合的场景是原始数据位于文件系统（HDFS）中，数据量为数十GB至TB级别。对于较小的数据量，请使用Stream Load或Broker Load。
对于完整的 Spark Load 导入示例，请参考 GitHub 上的演示: [https://github.com/StarRocks/demo/blob/master/docs/03_sparkLoad2StarRocks.md](https://github.com/StarRocks/demo/blob/master/docs/03_sparkLoad2StarRocks.md)

## 常见问题

- `错误: 在使用 master 'yarn' 时，必须在环境中设置 HADOOP-CONF-DIR 或 YARN-CONF-DIR。`

   在 Spark 客户端的 `spark-env.sh` 中没有配置 `HADOOP-CONF-DIR` 环境变量的情况下使用 Spark Load。

- `错误: 无法运行程序 "xxx/bin/spark-submit": error=2, 没有那个文件或目录`

   在使用 Spark Load 时，`spark_home_default_dir` 配置项未指定 Spark 客户端根目录。

- `错误: 文件 xxx/jars/spark-2x.zip 不存在。`

   在使用 Spark Load 时，`spark-resource-path` 配置项未指向打包的 zip 文件。

- `错误: yarn client 不存在于路径: xxx/yarn-client/hadoop/bin/yarn`

   在使用 Spark Load 时，`yarn-client-path` 配置项未指定 yarn 可执行文件。

- `错误: 无法执行 hadoop-yarn/bin/... /libexec/yarn-config.sh`

   在使用具有 CDH 的 Hadoop 时，需要配置 `HADOOP_LIBEXEC_DIR` 环境变量。
   由于 `hadoop-yarn` 和 hadoop 目录不同，因此默认的 `libexec` 目录将搜索 `hadoop-yarn/bin/... /libexec`，而 `libexec` 在 hadoop 目录中。
   执行 ```yarn application status``` 命令获取 Spark 任务状态报告错误，导致导入作业失败。