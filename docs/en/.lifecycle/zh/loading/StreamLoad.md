---
displayed_sidebar: English
---

# 从本地文件系统加载数据

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocks 提供两种从本地文件系统加载数据的方法：

- 使用 [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) 进行同步加载
- 使用 [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) 进行异步加载

每种方法都有其自身的优势：

- Stream Load 支持 CSV 和 JSON 文件格式。如果您需要从文件数量较少且单个文件大小不超过 10 GB 的数据中加载，推荐使用此方法。
- Broker Load 支持 Parquet、ORC 和 CSV 文件格式。如果您需要从大量文件中加载数据，且单个文件大小超过 10 GB，或者文件存储在网络附加存储（NAS）设备中，推荐使用此方法。**该功能从 v2.5 版本开始支持。请注意，如果您选择此方法，必须在数据文件所在的机器上[部署 Broker](../deployment/deploy_broker.md)。**

对于 CSV 数据，请注意以下几点：

- 您可以使用不超过 50 字节长度的 UTF-8 字符串作为文本分隔符，例如逗号（,）、制表符或竖线（|）。
- 空值用 `\N` 表示。例如，一个数据文件包含三列，其中一条记录在第一列和第三列有数据，但第二列无数据。在这种情况下，您需要在第二列使用 `\N` 来表示空值。这意味着记录应编译为 `a,\N,b` 而非 `a,,b`。`a,,b` 表示第二列的记录包含一个空字符串。

Stream Load 和 Broker Load 都支持在数据加载时进行数据转换，并支持在数据加载过程中通过 UPSERT 和 DELETE 操作进行数据变更。更多信息，请参见 [加载时转换数据](../loading/Etl_in_loading.md) 和 [通过加载变更数据](../loading/Load_to_Primary_Key_tables.md)。

## 在您开始之前

### 检查权限

<InsertPrivNote />


## 通过 Stream Load 从本地文件系统加载

Stream Load 是一种基于 HTTP PUT 的同步加载方法。提交加载任务后，StarRocks 同步执行任务，并在任务完成后返回任务结果。您可以根据任务结果判断任务是否成功。

> **注意**
> 使用 Stream Load 将数据加载到 StarRocks 表后，该表上创建的物化视图的数据也会更新。

### 工作原理

您可以在客户端向 FE 提交基于 HTTP 的加载请求，FE 然后使用 HTTP 重定向将加载请求转发到特定的 BE。您也可以直接在客户端向您选择的 BE 提交加载请求。

:::note

如果您向 FE 提交加载请求，FE 会使用轮询机制决定哪个 BE 将作为协调器来接收和处理加载请求。轮询机制有助于实现 StarRocks 集群内的负载均衡。因此，我们建议您向 FE 发送加载请求。

:::

接收加载请求的 BE 将作为协调器 BE 运行，根据使用的模式将数据分割成多个部分，并将每个部分的数据分配给其他参与的 BE。加载完成后，协调器 BE 将加载任务的结果返回给客户端。请注意，如果在加载过程中停止协调器 BE，加载任务将失败。

下图展示了 Stream Load 任务的工作流程。

![Stream Load 工作流程](../assets/4.2-1.png)

### 限制

Stream Load 不支持加载包含 JSON 格式列的 CSV 文件数据。

### 典型示例

本节以 curl 为例，介绍如何从本地文件系统中将 CSV 或 JSON 文件的数据加载到 StarRocks。详细的语法和参数描述，请参见 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。

请注意，在 StarRocks 中，某些字面量被 SQL 语言用作保留关键词。不要在 SQL 语句中直接使用这些关键词。如果您想在 SQL 语句中使用这类关键词，请将其用一对反引号 (`) 括起来。参见 [关键词](../sql-reference/sql-statements/keywords.md)。

#### 加载 CSV 数据

##### 准备数据集

在您的本地文件系统中，创建一个名为 `example1.csv` 的 CSV 文件。该文件包含三列，依次代表用户 ID、用户名和用户分数。

```Plain
1,Lily,23
2,Rose,23
3,Alice,24
4,Julia,25
```

##### 创建数据库和表

创建数据库并切换至该数据库：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

创建一个名为 `table1` 的主键表。该表包含三列：`id`、`name` 和 `score`，其中 `id` 是主键。

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

:::note

从 v2.5.7 版本开始，StarRocks 可以在创建表或添加分区时自动设置存储桶（BUCKETS）数量。您不再需要手动设置存储桶数量。详细信息，请参见 [确定存储桶数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

:::

##### 启动 Stream Load

运行以下命令将 `example1.csv` 的数据加载到 `table1`：

```Bash
curl --location-trusted -u <username>:<password> -H "label:123" \
    -H "Expect:100-continue" \
    -H "column_separator:," \
    -H "columns: id, name, score" \
    -T example1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/mydatabase/table1/_stream_load
```

:::note

- 如果您使用的账户未设置密码，只需输入 `<username>:`。
- 您可以使用 [SHOW FRONTENDS](../sql-reference/sql-statements/Administration/SHOW_FRONTENDS.md) 查看 FE 节点的 IP 地址和 HTTP 端口。

:::

`example1.csv` 包含三列，用逗号 (,) 分隔，可以依次映射到 `table1` 的 `id`、`name` 和 `score` 列。因此，需要使用 `column_separator` 参数指定逗号 (,) 作为列分隔符。还需要使用 `columns` 参数将 `example1.csv` 的三列临时命名为 `id`、`name` 和 `score`，依次映射到 `table1` 的三列。

加载完成后，您可以查询 `table1` 来验证加载是否成功：

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

在您的本地文件系统中，创建一个名为 `example2.json` 的 JSON 文件。该文件包含两列，依次代表城市 ID 和城市名称。

```JSON
{"name": "Beijing", "code": 2}
```

##### 创建数据库和表

创建数据库并切换至该数据库：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

创建一个名为 `table2` 的主键表。该表包含两列：`id` 和 `city`，其中 `id` 是主键。

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

:::note

从 v2.5.7 版本开始，StarRocks 可以在创建表或添加分区时自动设置存储桶数量。您不再需要手动设置存储桶数量。详细信息，请参见 [确定存储桶数量](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

:::

##### 启动 Stream Load

运行以下命令将 `example2.json` 的数据加载到 `table2`：

```Bash
curl -v --location-trusted -u <username>:<password> -H "strict_mode: true" \
    -H "Expect:100-continue" \
    -H "format: json" -H "jsonpaths: [\"$.name\", \"$.code\"]" \
    -H "columns: city,tmp_id, id = tmp_id * 100" \
    -T example2.json -XPUT \
    http://<fe_host>:<fe_http_port>/api/mydatabase/table2/_stream_load
```

:::note

- 如果您使用的账户未设置密码，只需输入 `<username>:`。
- 您可以使用 [SHOW FRONTENDS](../sql-reference/sql-statements/Administration/SHOW_FRONTENDS.md) 查看 FE 节点的 IP 地址和 HTTP 端口。

:::

`example2.json` 包含两个键 `name` 和 `code`，分别映射到 `table2` 的 `city` 和 `id` 列，如下图所示。

![JSON 到列的映射](../assets/4.2-2.png)

上述映射关系如下：

- StarRocks 提取 `example2.json` 的 `name` 和 `code` 键，并将它们映射到 `jsonpaths` 参数中声明的 `name` 和 `code` 字段。

- StarRocks 提取 `jsonpaths` 参数中声明的 `name` 和 `code` 字段，并**按顺序映射**到 `columns` 参数中声明的 `city` 和 `tmp_id` 字段。

- StarRocks 提取 `columns` 参数中声明的 `city` 和 `tmp_id` 字段，并**按名称映射**到 `table2` 的 `city` 和 `id` 列。

:::note

在上述示例中，`example2.json` 中的 `code` 值在加载到 `table2` 的 `id` 列之前被乘以 100。

:::

有关 `jsonpaths`、`columns` 和 StarRocks 表列之间的详细映射，请参见 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) 中的“列映射”部分。

加载完成后，您可以查询 `table2` 来验证加载是否成功：

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

加载任务完成后，StarRocks 以 JSON 格式返回任务结果。更多信息，请参见 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) 中的“返回值”部分。

Stream Load 不允许您使用 SHOW LOAD 语句查询加载任务的结果。

#### 取消 Stream Load 任务

Stream Load 不允许您取消加载任务。如果加载任务超时或遇到错误，StarRocks 会自动取消该任务。

### 参数配置

本节介绍选择 Stream Load 加载方式时需要配置的一些系统参数。这些参数配置对所有 Stream Load 任务生效。
```
- `streaming_load_max_mb`：您想要加载的每个数据文件的最大大小。默认最大大小是 10 GB。更多信息，请参见[配置 BE 动态参数](../administration/BE_configuration.md#configure-be-dynamic-parameters)。

  我们建议您一次性加载的数据不要超过 10 GB。如果数据文件的大小超过 10 GB，我们建议您将数据文件分割成小于 10 GB 的小文件，然后逐个加载这些文件。如果您无法分割大于 10 GB 的数据文件，您可以根据文件大小增加此参数的值。

  增加此参数的值后，只有在重启 StarRocks 集群的 BE 后，新的值才会生效。此外，系统性能可能会下降，且在加载失败时重试的成本也会增加。

  :::note

  当加载 JSON 文件的数据时，请注意以下几点：

- 文件中的每个 JSON 对象的大小不能超过 4 GB。如果文件中任何 JSON 对象超过 4 GB，StarRocks 会报错：“此解析器不能支持这么大的文档。”

- 默认情况下，HTTP 请求中的 JSON 正文不能超过 100 MB。如果 JSON 正文超过 100 MB，StarRocks 会报错：“此批次的大小超过了 json 类型数据的最大大小 [104857600] [8617627793]。设置 ignore_json_size 为 true 可以跳过检查，尽管这可能会导致巨大的内存消耗。”为了避免这个错误，您可以在 HTTP 请求头中添加 `"ignore_json_size:true"` 来忽略对 JSON 正文大小的检查。

  :::

- `stream_load_default_timeout_second`：每个加载作业的超时时间。默认超时时间是 600 秒。更多信息，请参见[配置 FE 动态参数](../administration/FE_configuration.md#configure-fe-dynamic-parameters)。

  如果您发现很多创建的加载作业都超时了，您可以根据以下公式计算的结果来增加此参数的值：

  **每个加载作业的超时时间 > 要加载的数据量 / 平均加载速度**

  例如，如果您要加载的数据文件大小是 10 GB，而您的 StarRocks 集群的平均加载速度是 100 MB/s，那么您应该将超时时间设置为超过 100 秒。

  :::note

  上述公式中的“平均加载速度”是指您的 StarRocks 集群的平均加载速度。它会根据磁盘 I/O 和 StarRocks 集群中 BE 的数量而有所不同。

  :::

  Stream Load 还提供了 `timeout` 参数，允许您为单个加载作业指定超时时间。更多信息，请参见 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。

### 使用说明

如果您要加载的数据文件中的记录缺少某个字段，并且在 StarRocks 表中映射到该字段的列被定义为 `NOT NULL`，那么在加载该记录时，StarRocks 会自动在映射列中填充 `NULL` 值。您也可以使用 `ifnull()` 函数来指定您希望填充的默认值。

例如，如果前面的 `example2.json` 文件中代表城市 ID 的字段缺失，并且您希望在 `table2` 的映射列中填充 `x` 值，您可以指定 `"columns: city, tmp_id, id = ifnull(tmp_id, 'x')"`。

## 通过 Broker Load 从本地文件系统加载

除了 Stream Load，您还可以使用 Broker Load 从本地文件系统加载数据。此功能从 v2.5 版本开始支持。

Broker Load 是一种异步加载方法。提交加载作业后，StarRocks 会异步运行作业，并不会立即返回作业结果。您需要手动查询作业结果。参见[检查 Broker Load 进度](#check-broker-load-progress)。

### 限制

- 目前 Broker Load 仅支持通过单个 Broker（版本为 v2.5 或更高）从本地文件系统加载。
- 针对单个 Broker 的高并发查询可能会导致超时和 OOM 等问题。为了减轻影响，您可以使用 `pipeline_dop` 变量（参见[系统变量](../reference/System_variable.md#pipeline_dop)）来设置 Broker Load 查询的并行度。对于针对单个 Broker 的查询，我们建议您将 `pipeline_dop` 设置为小于 `16` 的值。

### 开始之前

在使用 Broker Load 从本地文件系统加载数据之前，您需要完成以下准备工作：

1. 按照“[部署前提条件](../deployment/deployment_prerequisites.md)”、“[检查环境配置](../deployment/environment_configurations.md)”和“[准备部署文件](../deployment/prepare_deployment_files.md)”中的指导配置本地文件所在的机器，然后在该机器上部署 Broker。操作与在 BE 节点上部署 Broker 相同。详细操作，请参见[部署和管理 Broker 节点](../deployment/deploy_broker.md)。

      > **注意**
      > 仅部署一个 Broker，并确保 Broker 版本为 v2.5 或更高。

2. 执行 [ALTER SYSTEM](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md#broker) 将您在上一步中部署的 Broker 添加到 StarRocks 集群，并为该 Broker 定义一个新名称。以下示例将 Broker `172.26.199.40:8000` 添加到 StarRocks 集群，并将 Broker 名称定义为 `sole_broker`：

   ```SQL
   ALTER SYSTEM ADD BROKER sole_broker "172.26.199.40:8000";
   ```

### 典型示例

Broker Load 支持从单个数据文件到单个表的加载、从多个数据文件到单个表的加载，以及从多个数据文件到多个表的加载。本节以从多个数据文件到单个表的加载为例。

请注意，在 StarRocks 中，某些字面量被 SQL 语言用作保留关键字。不要直接在 SQL 语句中使用这些关键字。如果您想在 SQL 语句中使用这样的关键字，请将其包含在一对反引号 (`) 中。参见[关键字](../sql-reference/sql-statements/keywords.md)。

#### 准备数据集

以 CSV 文件格式为例。登录到您的本地文件系统，在特定存储位置（例如 `/user/starrocks/`）创建两个 CSV 文件 `file1.csv` 和 `file2.csv`。两个文件都包含三列，依次代表用户 ID、用户名和用户分数。

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

创建数据库并切换到该数据库：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

创建一个名为 `mytable` 的主键表。该表包含三列：`id`、`name` 和 `score`，其中 `id` 是主键。

```SQL
CREATE TABLE `mytable`
(
    `id` int(11) NOT NULL COMMENT "用户 ID",
    `name` varchar(65533) NULL DEFAULT "" COMMENT "用户名",
    `score` int(11) NOT NULL DEFAULT "0" COMMENT "用户分数"
)
ENGINE=OLAP
PRIMARY KEY(`id`)
DISTRIBUTED BY HASH(`id`)
PROPERTIES("replication_num"="1");
```

#### 启动 Broker Load

运行以下命令启动一个 Broker Load 作业，将您本地文件系统路径 `/user/starrocks/` 中存储的所有数据文件（`file1.csv` 和 `file2.csv`）加载到 StarRocks 表 `mytable`：

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

这个作业有四个主要部分：

- `LABEL`：查询加载作业状态时使用的字符串。
- `LOAD` 声明：源 URI、源数据格式和目标表名。
- `BROKER`：代理的名称。
- `PROPERTIES`：超时值和适用于加载作业的其他属性。

详细语法和参数描述，请参见 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

#### 检查 Broker Load 进度

在 v3.0 及之前的版本中，使用 [SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) 语句或 curl 命令来查看 Broker Load 作业的进度。

在 v3.1 及之后的版本中，您可以从 [`information_schema.loads`](../reference/information_schema/loads.md) 视图中查看 Broker Load 作业的进度：

```SQL
SELECT * FROM information_schema.loads;
```

如果您提交了多个加载作业，您可以根据作业的 `LABEL` 来进行筛选。例如：

```SQL
SELECT * FROM information_schema.loads WHERE LABEL = 'label_local';
```

确认加载作业完成后，您可以查询表来确认数据是否已成功加载。例如：

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
8 行在集合中 (0.07 秒)
```

#### 取消 Broker Load 作业

当加载作业不处于 **CANCELLED** 或 **FINISHED** 阶段时，您可以使用 [CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md) 语句来取消作业。

例如，您可以执行以下语句来取消数据库 `mydatabase` 中标签为 `label_local` 的加载作业：

```SQL
CANCEL LOAD
FROM mydatabase
WHERE LABEL = "label_local";
```

## 通过 Broker Load 从 NAS 加载

使用 Broker Load 从 NAS 加载数据有两种方法：

- 将 NAS 视为本地文件系统，并运行带有代理的加载作业。参见上一节“[通过 Broker Load 从本地系统加载](#loading-from-a-local-file-system-via-broker-load)”。
- （推荐）将 NAS 视为云存储系统，并运行不带代理的加载作业。

本节介绍第二种方法。具体操作如下：

1. 将 NAS 设备挂载到 StarRocks 集群的所有 BE 节点和 FE 节点的相同路径上。这样，所有 BE 就可以像访问本地存储的文件一样访问 NAS 设备。

2. 使用 Broker Load 将数据从 NAS 设备加载到目标 StarRocks 表中。例如：

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

   这个作业有四个主要部分：

   - `LABEL`：查询加载作业状态时使用的字符串。
   - `LOAD` 声明：源 URI、源数据格式和目标表名。注意，在声明中的 `DATA INFILE` 用于指定 NAS 设备的挂载点文件夹路径，如上例中的 `file:///` 是前缀，`/home/disk1/sr` 是挂载点文件夹路径。
   - `BROKER`：您不需要指定代理名称。
   - `PROPERTIES`：超时值和适用于加载作业的其他属性。

   详细语法和参数描述，请参见 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

提交作业后，您可以根据需要查看加载进度或取消作业。具体操作，请参见本主题中的“[检查 Broker Load 进度](#check-broker-load-progress)”和“[取消 Broker Load 作业](#cancel-a-broker-load-job)”。
```markdown
   - LOAD 声明：源 URI、源数据格式和目标表名称。注意声明中的 `DATA INFILE` 用于指定 NAS 设备的挂载点文件夹路径，如上例所示，其中 `file:///` 为前缀，`/home/disk1/sr` 为挂载点文件夹路径。
   - Broker：您不需要指定 Broker 名称。
   - 属性：超时值和适用于加载作业的任何其他属性。

   详细语法和参数说明请参见 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

提交作业后，您可以根据需要查看加载进度或取消作业。详细操作请参见本主题中的“[检查 Broker Load 进度](#check-broker-load-progress)”和“[取消 Broker Load 作业](#cancel-a-broker-load-job)”。