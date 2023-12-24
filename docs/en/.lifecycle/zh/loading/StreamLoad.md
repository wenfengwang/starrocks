---
displayed_sidebar: English
---

# 从本地文件系统加载数据

从 '../assets/commonMarkdown/insertPrivNote.md' 导入 InsertPrivNote

StarRocks 提供了两种从本地文件系统加载数据的方法：

- 使用 [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) 进行同步加载
- 使用 [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 进行异步加载

每种方法都有其各自的优点：

- Stream Load 支持 CSV 和 JSON 文件格式。如果要从个别文件中加载数据，且每个文件的大小均不超过 10 GB，则建议使用此方法。
- Broker Load 支持 Parquet、ORC 和 CSV 文件格式。如果要从大量文件中加载数据，且每个文件的大小超过 10 GB，或者文件存储在网络连接存储（NAS）设备中，则建议使用此方法。**从 v2.5 版本开始支持此功能。请注意，如果选择此方法，则必须在存储数据文件的机器上[部署代理](../deployment/deploy_broker.md)。**

对于 CSV 数据，请注意以下几点：

- 您可以使用长度不超过 50 个字节的 UTF-8 字符串（例如逗号（,）、制表符或竖线（|））作为文本分隔符。
- 使用 `\N` 表示空值。例如，数据文件包含三列，其中一条记录在第一列和第三列中保存数据，但在第二列中不保存任何数据。在这种情况下，您需要在第二列使用 `\N` 表示空值，而不是使用 `a,,b`。`a,,b` 表示记录的第二列包含一个空字符串。

Stream Load 和 Broker Load 都支持在数据加载时进行数据转换，并支持在数据加载期间通过 UPSERT 和 DELETE 操作进行的数据更改。更多信息，请参见[加载时的数据转换](../loading/Etl_in_loading.md)和[通过加载更改数据](../loading/Load_to_Primary_Key_tables.md)。

## 开始之前

### 检查权限

<InsertPrivNote />

## 通过 Stream Load 从本地文件系统加载

Stream Load 是一种基于 HTTP PUT 的同步加载方法。提交加载作业后，StarRocks 会同步运行该作业，并在作业完成后返回作业结果。您可以根据作业结果判断作业是否成功。

> **注意**
>
> 使用 Stream Load 将数据加载到 StarRocks 表中后，在该表上创建的物化视图的数据也会更新。

### 工作原理

您可以根据 HTTP 向 FE 提交客户端的加载请求，然后 FE 使用 HTTP 重定向将加载请求转发到特定的 BE。您也可以直接将客户端上的加载请求提交到您选择的 BE。

:::注意

如果向 FE 提交加载请求，FE 会使用轮询机制来决定哪个 BE 将充当接收和处理加载请求的协调器。轮询机制有助于实现 StarRocks 集群内的负载均衡。因此，建议您向 FE 发送加载请求。

:::

接收加载请求的 BE 作为协调器 BE 运行，以根据使用的架构将数据拆分为多个部分，并将数据的每个部分分配给其他涉及的 BE。加载完成后，协调器 BE 将加载作业的结果返回给客户端。请注意，如果在加载期间停止协调器 BE，则加载作业将失败。

下图显示了 Stream Load 作业的工作流程。

![Stream Load 的工作流程](../assets/4.2-1.png)

### 限制

Stream Load 不支持加载包含 JSON 格式列的 CSV 文件的数据。

### 典型案例

本节以 curl 为例，介绍如何将本地文件系统中的 CSV 或 JSON 文件数据加载到 StarRocks 中。有关语法和参数的详细说明，请参见 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。

需要注意的是，在 StarRocks 中，一些文字被 SQL 语言用作保留关键字。不要在 SQL 语句中直接使用这些关键字。如果要在 SQL 语句中使用此类关键字，请将其括在一对反引号（`）中。请参见 [关键字](../sql-reference/sql-statements/keywords.md)。

#### 加载 CSV 数据

##### 准备数据集

在本地文件系统中，创建名为 `example1.csv` 的 CSV 文件。该文件由三列组成，分别依次表示用户 ID、用户名和用户分数。

```Plain
1,Lily,23
2,Rose,23
3,Alice,24
4,Julia,25
```

##### 创建数据库和表

创建一个数据库并切换到它：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

创建一个名为 `table1` 的主键表。该表由三列组成：`id`、`name`和`score`，其中 `id` 是主键。

```SQL
CREATE TABLE `table1`
(
    `id` int(11) NOT NULL COMMENT "user ID",
    `name` varchar(65533) NULL COMMENT "user name",
    `score` int(11) NOT NULL COMMENT "user score"
)
ENGINE=OLAP
PRIMARY KEY(`id`)
DISTRIBUTED BY HASH(`id`);
```

:::注意

从 v2.5.7 开始，StarRocks 可以在您创建表或添加分区时自动设置 BUCKET 数量。您不再需要手动设置存储桶数量。有关详细信息，请参见 [确定存储桶数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

:::

##### 启动流加载

执行以下命令，将 `example1.csv` 的数据加载到 `table1`：

```Bash
curl --location-trusted -u <username>:<password> -H "label:123" \
    -H "Expect:100-continue" \
    -H "column_separator:," \
    -H "columns: id, name, score" \
    -T example1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/mydatabase/table1/_stream_load
```

:::注意

- 如果您使用未设置密码的帐户，则只需输入 `<username>:`。
- 您可以使用 [SHOW FRONTENDS](../sql-reference/sql-statements/Administration/SHOW_FRONTENDS.md) 查看 FE 节点的 IP 地址和 HTTP 端口。

:::

`example1.csv` 由三列组成，用逗号（,）分隔，可以依次映射到 `table1` 的 `id`、`name`和`score` 列上。因此，您需要使用 `column_separator` 参数将逗号（,）指定为列分隔符。您还需要使用 `columns` 参数临时将 `example1.csv` 的 `id`、`name`和`score` 三列命名为 `id`、`name`和`score`，它们按顺序映射到 `table1` 的三列上。

加载完成后，可以查询 `table1` 以验证加载是否成功：

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    23 |
|    2 | Rose  |    23 |
|    3 | Alice |    24 |
|    4 | Julia |    25 |
+------+-------+-------+
4 rows in set (0.00 sec)
```

#### 加载 JSON 数据

##### 准备数据集

在本地文件系统中，创建名为 `example2.json` 的 JSON 文件。该文件由两列组成，分别依次表示城市 ID 和城市名称。

```JSON
{"name": "Beijing", "code": 2}
```

##### 创建数据库和表

创建一个数据库并切换到它：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

创建一个名为 `table2` 的主键表。该表由两列组成：`id` 和 `city`，其中 `id` 是主键。

```SQL
CREATE TABLE `table2`
(
    `id` int(11) NOT NULL COMMENT "city ID",
    `city` varchar(65533) NULL COMMENT "city name"
)
ENGINE=OLAP
PRIMARY KEY(`id`)
DISTRIBUTED BY HASH(`id`);
```

:::注意

从 v2.5.7 开始，StarRocks 可以在创建表或添加分区时自动设置 BUCKETS 的数量。您不再需要手动设置存储桶数量。有关详细信息，请参见 [确定存储桶数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

:::

##### 启动流加载

执行以下命令，将 `example2.json` 的数据加载到 `table2`：

```Bash
curl -v --location-trusted -u <username>:<password> -H "strict_mode: true" \
    -H "Expect:100-continue" \
    -H "format: json" -H "jsonpaths: [\"$.name\", \"$.code\"]" \
    -H "columns: city,tmp_id, id = tmp_id * 100" \
    -T example2.json -XPUT \
    http://<fe_host>:<fe_http_port>/api/mydatabase/table2/_stream_load
```

:::注意

- 如果您使用未设置密码的帐户，则只需输入 `<username>:`。
- 您可以使用 [SHOW FRONTENDS](../sql-reference/sql-statements/Administration/SHOW_FRONTENDS.md) 查看 FE 节点的 IP 地址和 HTTP 端口。

:::

`example2.json` 由两个键 `name` 和 `code` 组成，它们映射到 `table2` 的 `id` 和 `city` 列上，如下图所示。

![JSON - 列映射](../assets/4.2-2.png)

上图所示的映射关系如下：

- StarRocks 从 `example2.json` 中提取 `name` 和 `code` 键，并将其映射到 `jsonpaths` 参数中声明的 `name` 和 `code` 字段上。

- StarRocks 从 `jsonpaths` 参数中声明的 `name` 和 `code` 字段中提取，并依次映射到 `columns` 参数中声明的 `city` 和 `tmp_id` 字段上。

- StarRocks 从 `columns` 参数中声明的 `city` 和 `tmp_id` 字段中提取，并按名称映射到 `table2` 的 `city` 和 `id` 列上。

:::注意

在前面的示例中，`example2.json` 中的 `code` 值在加载到 `table2` 的 `id` 列中之前乘以 100。

:::

有关 StarRocks 表的 `jsonpaths`、`columns` 和列之间的详细映射关系，请参见 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) 中的“列映射”章节。

加载完成后，可以查询 `table2` 以验证加载是否成功：

```SQL
SELECT * FROM table2;
+------+--------+
| id   | city   |
+------+--------+
| 200  | Beijing|
+------+--------+
4 rows in set (0.01 sec)
```

#### 检查 Stream Load 进度

加载作业完成后，StarRocks 会以 JSON 格式返回作业结果。更多信息，请参见 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) 中的“返回值”部分。

Stream Load 不允许您使用 SHOW LOAD 语句查询加载作业的结果。

#### 取消 Stream Load 作业

Stream Load 不允许您取消加载作业。如果加载作业超时或遇到错误，StarRocks 会自动取消该作业。

### 参数配置

本节介绍一些系统参数，当您选择加载方式 Stream Load 时，需要配置这些参数。这些参数配置对所有 Stream Load 作业生效。

- `streaming_load_max_mb`：要加载的每个数据文件的最大大小。默认最大大小为 10 GB。有关更多信息，请参见 [配置BE动态参数](../administration/BE_configuration.md#configure-be-dynamic-parameters)。
  
  我们建议您一次不要加载超过 10 GB 的数据。如果数据文件的大小超过 10GB，建议您将数据文件拆分为小于 10GB 的小文件，然后逐个加载。如果无法将大于 10 GB 的数据文件拆分，您可以根据文件大小增加该参数的值。

  增加该参数后，新值只有在重启 StarRocks 集群的 BE 后才能生效。此外，系统性能可能会下降，并且在发生负载故障时重试的成本也会增加。

  :::注意
  
  加载 JSON 文件时，请注意以下几点：
  
  - 文件中每个 JSON 对象的大小不能超过 4 GB。如果文件中的 JSON 对象超过 4 GB，StarRocks 会抛出错误“This parser can't support a document that big”。
  
  - 默认情况下，HTTP 请求中的 JSON 正文不能超过 100 MB。当 JSON 正文超过 100 MB 时，StarRocks 会抛出错误“The size of this batch exceeded the max size [104857600] of json type [8617627793] data data data 。设置 ignore_json_size 为 true 可跳过检查，尽管这可能会导致大量内存消耗。为了避免此错误，您可以在 HTTP 请求标头中添加 `"ignore_json_size:true"` 以忽略对 JSON 正文大小的检查。

  :::

- `stream_load_default_timeout_second`：每个加载作业的超时时间。默认超时期限为 600 秒。有关更多信息，请参见 [配置FE动态参数](../administration/FE_configuration.md#configure-fe-dynamic-parameters)。
  
  如果创建的多个加载作业超时，则可以根据以下公式获得的计算结果增加此参数的值：

  **每个加载作业的超时时间 > 需要加载的数据量/平均加载速度**

  例如，您需要加载的数据文件大小为 10 GB，StarRocks 集群的平均加载速度为 100 MB/s，则将超时时间设置为 100 秒以上。

  :::注意
  
  **上式中的平均加载速度**是 StarRocks 集群的平均加载速度。具体值因磁盘 I/O 和 StarRocks 集群的 BE 数量而异。

  :::

  Stream Load 还提供了 `timeout` 参数，该参数允许您指定单个加载作业的超时时间。有关更多信息，请参见 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。

### 使用说明

如果要加载的数据文件中缺少某个字段，并且 StarRocks 表中该字段映射到的列定义为 `NOT NULL`，则在加载记录时，StarRocks 会自动在 StarRocks 表的映射列中填充一个 `NULL` 值。您还可以使用 `ifnull()` 函数指定要填充的默认值。

例如，如果缺少上述文件中表示城市 ID 的字段，并且您希望在 `table2` 的映射列中填充一个值 `x`，则可以指定 `"columns: city, tmp_id, id = ifnull(tmp_id, 'x')"`。

## 通过 Broker Load 从本地文件系统加载

除了 Stream Load 之外，您还可以使用 Broker Load 从本地文件系统加载数据。从 v2.5 开始支持此功能。

Broker Load 是一种异步加载方法。提交加载作业后，StarRocks 会异步运行该作业，不会立即返回作业结果。您需要手动查询作业结果。请参见 [检查代理加载进度](#check-broker-load-progress)。

### 限制

- 目前，Broker Load 仅支持通过版本为 v2.5 或更高版本的单个 broker 从本地文件系统加载。
- 针对单个代理的高并发查询可能会导致超时和 OOM 等问题。要减轻影响，您可以使用变量 `pipeline_dop` （请参见 [系统变量](../reference/System_variable.md#pipeline_dop)）来设置 Broker Load 的查询并行度。对于针对单个代理的查询，我们建议您将值设置为小于 `16` 的 `pipeline_dop`。

### 准备工作

在使用 Broker Load 从本地文件系统装入数据之前，请完成以下准备工作：

1. 按照“部署先决条件[”、“](../deployment/deployment_prerequisites.md)检查环境[配置”和“](../deployment/environment_configurations.md)准备部署[文件”中的说明配置本地文件所在的计算机](../deployment/prepare_deployment_files.md)。然后，在该计算机上部署代理。这些操作与在 BE 节点上部署代理相同。具体操作请参见[部署和管理Broker节点](../deployment/deploy_broker.md)。

   > **通知**
   >
   > 仅部署单个 Broker，并确保 Broker 版本为 v2.5 或更高版本。

2. 执行 [ALTER SYSTEM](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md#broker)，将上一步中部署的 broker 添加到 StarRocks 集群中，并为该 broker 定义一个新名称。以下示例 `172.26.199.40:8000` 将 Broker 添加到 StarRocks 集群中，并将 Broker 名称定义为 `sole_broker`：

   ```SQL
   ALTER SYSTEM ADD BROKER sole_broker "172.26.199.40:8000";
   ```

### 典型案例

Broker Load 支持从单个数据文件加载到单个表、从多个数据文件加载到单个表以及从多个数据文件加载到多个表。本节以从多个数据文件加载到单个表为例。

需要注意的是，在 StarRocks 中，一些文字被 SQL 语言用作保留关键字。不要在 SQL 语句中直接使用这些关键字。如果要在 SQL 语句中使用此类关键字，请将其括在一对反引号 （'） 中。请参阅 [关键字](../sql-reference/sql-statements/keywords.md)。

#### 准备数据集

以 CSV 文件格式为例。登录到本地文件系统，创建两个 CSV 文件，`file1.csv` 和 `file2.csv`，并在特定存储位置（例如 `/user/starrocks/`）创建。这两个文件都由三列组成，分别依次表示用户 ID、用户名和用户分数。

- `file1.csv`

  ```Plain
  1,Lily,21
  2,Rose,22
  3,Alice,23
  4,Julia,24
  ```

- `file2.csv`

  ```Plain
  5,Tony,25
  6,Adam,26
  7,Allen,27
  8,Jacky,28
  ```

#### 创建数据库和表

创建一个数据库并切换到它：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

创建一个名为 `mytable` 的主键表。该表由三列组成：`id`、`name`和`score`，其中 `id` 是主键。

```SQL
CREATE TABLE `mytable`
(
    `id` int(11) NOT NULL COMMENT "User ID",
    `name` varchar(65533) NULL DEFAULT "" COMMENT "User name",
    `score` int(11) NOT NULL DEFAULT "0" COMMENT "User score"
)
ENGINE=OLAP
PRIMARY KEY(`id`)
DISTRIBUTED BY HASH(`id`)
PROPERTIES("replication_num"="1");
```

#### 启动代理加载

执行以下命令，启动 Broker Load 作业，将本地文件系统路径 `file1.csv` 下的所有 `file2.csv` 数据文件（和 `/user/starrocks/`）中的数据加载到 StarRocks 表 `mytable`：

```SQL
LOAD LABEL mydatabase.label_local
(
    DATA INFILE("file:///home/disk1/business/csv/*")
    INTO TABLE mytable
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER "sole_broker"
PROPERTIES
(
    "timeout" = "3600"
);
```

这项工作有四个主要部分：

- `LABEL`：查询加载作业状态时使用的字符串。
- `LOAD` declaration：源 URI、源数据格式和目标表名称。
- `BROKER`：代理的名称。
- `PROPERTIES`：超时值和要应用于加载作业的任何其他属性。

有关语法和参数的详细描述，请参见 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

#### 检查代理加载进度

在 v3.0 及更低版本中，使用 [SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) 语句或 curl 命令查看 Broker Load 作业的进度。

在 v3.1 及更高版本中，您可以从以下视图查看 Broker Load 作业的进度 [`information_schema.loads`](../reference/information_schema/loads.md) ：

```SQL
SELECT * FROM information_schema.loads;
```

如果已提交多个加载作业，则可以筛选与该作业关联的 `LABEL`。例：

```SQL
SELECT * FROM information_schema.loads WHERE LABEL = 'label_local';
```

确认加载作业完成后，可以查询表查看数据是否已成功加载。例：

```SQL
SELECT * FROM mytable;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    3 | Alice |    23 |
|    5 | Tony  |    25 |
|    6 | Adam  |    26 |
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    4 | Julia |    24 |
|    7 | Allen |    27 |
|    8 | Jacky |    28 |
+------+-------+-------+
8 rows in set (0.07 sec)
```

#### 取消 Broker Load 作业

当加载作业不在 **CANCELLED** 或 **FINISHED** 阶段时，可以使用 [CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md) 语句取消该作业。

例如，您可以执行以下语句取消数据库中标签为 `label_local` 的加载作业：

```SQL
CANCEL LOAD
FROM mydatabase
WHERE LABEL = "label_local";
```

## 通过 Broker Load 从 NAS 加载

使用 Broker Load 从 NAS 加载数据有两种方法：

- 将 NAS 视为本地文件系统，并使用代理运行加载作业。请参阅上一节 “[通过 Broker Load 从本地系统加载](#loading-from-a-local-file-system-via-broker-load)”。
- （推荐）将 NAS 视为云存储系统，并在没有代理的情况下运行加载作业。

本节介绍第二种方式。具体操作如下：

1. 将 NAS 设备挂载到 StarRocks 集群的所有 BE 节点和 FE 节点上的同一路径。因此，所有 BE 都可以访问 NAS 设备，就像访问自己本地存储的文件一样。

2. 使用 Broker Load 将 NAS 设备中的数据加载到目标 StarRocks 表中。例：

   ```SQL
   LOAD LABEL test_db.label_nas
   (
       DATA INFILE("file:///home/disk1/sr/*")
       INTO TABLE mytable
       COLUMNS TERMINATED BY ","
   )
   WITH BROKER
   PROPERTIES
   (
       "timeout" = "3600"
   );
   ```

   这项工作有四个主要部分：

   - `LABEL`：查询加载作业状态时使用的字符串。

   - `LOAD` 声明：源 URI、源数据格式和目标表名称。请注意，声明中的 `DATA INFILE` 用于指定 NAS 设备的挂载点文件夹路径，就像上面的例子中显示的那样，`file:///` 是前缀，`/home/disk1/sr` 是挂载点文件夹路径。
   - `BROKER`：您无需指定代理名称。
   - `PROPERTIES`：超时值和要应用于加载作业的任何其他属性。

   有关详细的语法和参数描述，请参阅 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

提交作业后，您可以根据需要查看加载进度或取消作业。有关详细操作，请参见本主题中的“[检查 Broker Load 进度](#check-broker-load-progress)”和“[取消 Broker Load 作业](#cancel-a-broker-load-job)”。