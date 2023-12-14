# 从本地文件系统加载数据

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocks提供两种从本地文件系统加载数据的方法：

- 使用[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)进行同步加载

- 使用[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)进行异步加载

这两种选项各有其优势：


- Stream Load支持CSV和JSON文件格式。如果要从不超过10GB的一小部分文件中加载数据，建议使用此方法。

- Broker Load支持Parquet、ORC和CSV文件格式。如果要从存储在网络附加存储（NAS）设备中或文件数量超过10GB的大量文件中加载数据，建议使用此方法。**此功能从v2.5版本开始支持。请注意，如果选择此方法，必须在存储数据文件的机器上[部署Broker](../deployment/deploy_broker.md)。**


对于CSV数据，请注意以下几点：

- 您可以使用不超过50字节的UTF-8字符串，如逗号（，）、制表符或管道符（|），作为文本分隔符。
- 使用`\N`表示空值。例如，数据文件由三列组成，文件中的某一行在第一列和第三列中包含数据，但第二列中不包含数据。在这种情况下，您需要在第二列中使用`\N`表示空值。这意味着该记录必须编制为`a,\N,b`，而不是`a,,b`。`a,,b`表示该记录的第二列包含空字符串。

Stream Load和Broker Load都支持加载数据时的数据转换，并支持通过UPSERT和DELETE操作进行的数据更改。有关更多信息，请参阅[加载时进行数据转换](../loading/Etl_in_loading.md)和[通过加载更改数据](../loading/Load_to_Primary_Key_tables.md)。

## 开始之前

### 检查权限

<InsertPrivNote />

## 通过Stream Load从本地文件系统加载数据

Stream Load是基于HTTP PUT的同步加载方法。在提交加载作业后，StarRocks将同步运行该作业，并在作业完成后返回作业结果。您可以根据作业结果确定作业是否成功。

> **注意**
>
> 使用Stream Load将数据加载到StarRocks表中后，还会更新在该表上创建的materialized view的数据。

### 工作原理

您可以根据HTTP在客户端向FE提交加载请求，然后FE使用HTTP重定向将加载请求转发到指定的BE。您也可以直接在客户端向您选择的BE提交加载请求。

:::note

如果您将加载请求提交给FE，FE会使用轮询机制来决定哪个BE将作为协调器接收和处理加载请求。轮询机制有助于在StarRocks集群中实现负载均衡。因此，我们建议您向FE发送加载请求。

:::

接收加载请求的BE作为协调器BE运行，根据使用的模式将数据拆分为部分，并将数据的每部分分配给其他相关的BE。加载完成后，协调器BE将加载作业的结果返回给您的客户端。请注意，在加载期间停止协调器BE将导致加载作业失败。

以下图显示了Stream Load作业的工作流程。

![Stream Load的工作流程](../assets/4.2-1.png)

### 限制

Stream Load不支持加载包含JSON格式列的CSV文件的数据。

### 典型示例

本节以curl为例，描述了如何从本地文件系统中加载CSV或JSON文件的数据到StarRocks。有关详细的语法和参数描述，请参阅[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。

请注意，在StarRocks中，SQL语言使用一些文本作为保留关键字。不要直接在SQL语句中使用这些关键字。如果要在SQL语句中使用此类关键字，请用一对反引号 (`) 括起来。请参阅[关键字](../sql-reference/sql-statements/keywords.md)。

#### 加载CSV数据

##### 准备数据集

在您的本地文件系统中，创建名为`example1.csv`的CSV文件。该文件由三列组成，依次表示用户ID、用户名和用户得分。

```Plain
1,Lily,23
2,Rose,23
3,Alice,24
4,Julia,25
```

##### 创建数据库和表

创建数据库并切换到该数据库：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

创建名为`table1`的主键表。该表由三列组成：`id`、`name`和`score`，其中`id`是主键。

```SQL
CREATE TABLE `table1`
(
    `id` int(11) NOT NULL COMMENT "用户ID",
    `name` varchar(65533) NULL COMMENT "用户名",
    `score` int(11) NOT NULL COMMENT "用户得分"
)
ENGINE=OLAP
PRIMARY KEY(`id`)
DISTRIBUTED BY HASH(`id`);
```

:::note

自v2.5.7以来，StarRocks在创建表或添加分区时可以自动设置桶数（BUCKETS）。您不再需要手动设置桶数。有关详细信息，请参阅[确定桶数](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

:::

##### 开始Stream Load

执行以下命令将`example1.csv`的数据加载到`table1`中：

```Bash
curl --location-trusted -u <username>:<password> -H "label:123" \
    -H "Expect:100-continue" \
    -H "column_separator:," \
    -H "columns: id, name, score" \
    -T example1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/mydatabase/table1/_stream_load
```

:::note

- 如果使用的帐户未设置密码，则只需输入`<username>:`。
- 您可以使用[SHOW FRONTENDS](../sql-reference/sql-statements/Administration/SHOW_FRONTENDS.md)查看FE节点的IP地址和HTTP端口。

:::

`example1.csv`由三列组成，用逗号（,）分隔，并且可以依次映射到`table1`的`id`、`name`和`score`列上。因此，您需要使用`column_separator`参数指定逗号（,）作为列分隔符。您还需要使用`columns`参数将`example1.csv`的三列临时命名为`id`、`name`和`score`，并依次映射到`table1`的三列上。

加载完成后，您可以查询`table1`以验证加载是否成功：

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

#### 加载JSON数据

##### 准备数据集

在您的本地文件系统中，创建名为`example2.json`的JSON文件。该文件由两列组成，依次表示城市ID和城市名称。

```JSON
{"name": "Beijing", "code": 2}
```

##### 创建数据库和表

创建数据库并切换到该数据库：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

创建名为`table2`的主键表。该表由两列组成：`id`和`city`，其中`id`是主键。

```SQL
CREATE TABLE `table2`
(
    `id` int(11) NOT NULL COMMENT "城市ID",
    `city` varchar(65533) NULL COMMENT "城市名称"
)
ENGINE=OLAP
PRIMARY KEY(`id`)
DISTRIBUTED BY HASH(`id`);
```

:::note

自v2.5.7以来，StarRocks可以在创建表或添加分区时自动设置桶数（BUCKETS）。您不再需要手动设置桶数。有关详细信息，请参阅[确定桶数](../table_design/Data_distribution.md#determine-the-number-of-buckets)。

:::

##### 开始Stream Load

执行以下命令将`example2.json`的数据加载到`table2`中：

```Bash
curl -v --location-trusted -u <username>:<password> -H "strict_mode: true" \
```
    -H "Expect:100-continue" \
    -H "format: json" -H "jsonpaths: [\"$.name\", \"$.code\"]" \
    -H "columns: city,tmp_id, id = tmp_id * 100" \
    -T example2.json -XPUT \
    http://<fe_host>:<fe_http_port>/api/mydatabase/table2/_stream_load
```

:::注意

- 如果您使用的帐户未设置密码，则只需输入 `<username>：`。
- 您可以使用 [SHOW FRONTENDS](../sql-reference/sql-statements/Administration/SHOW_FRONTENDS.md) 查看 FE 节点的IP地址和HTTP端口。

:::

`example2.json` 包括两个键，`name` 和 `code`，它们被映射到 `table2` 的 `id` 和 `city` 列中，如下图所示。

![JSON - 列映射](../assets/4.2-2.png)

前面图中的映射描述如下：

- StarRocks 提取 `example2.json` 的 `name` 和 `code` 键，并将它们映射到 `jsonpaths` 参数中声明的 `name` 和 `code` 字段上。

- StarRocks 提取 `jsonpaths` 参数中声明的 `name` 和 `code` 字段，并**按顺序**将它们映射到 `columns` 参数中声明的 `city` 和 `tmp_id` 字段上。

- StarRocks 提取 `columns` 参数中声明的 `city` 和 `tmp_id` 字段，并按名称将它们映射到 `table2` 的 `city` 和 `id` 列上。

:::注意

在前面的示例中，`example2.json` 中的 `code` 值在加载到 `table2` 的 `id` 列之前被乘以100。

:::

有关 `jsonpaths`、`columns` 和 StarRocks 表列之间的详细映射，请参见[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)中的“列映射”部分。

加载完成后，您可以查询 `table2` 以验证加载是否成功：

```SQL
SELECT * FROM table2;
+------+--------+
| id   | city   |
+------+--------+
| 200  | 北京   |
+------+--------+
4 rows in set (0.01 sec)
```

#### 检查 Stream Load 进度

加载作业完成后，StarRocks以 JSON 格式返回作业结果。有关更多信息，请参见[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)中的“返回值”部分。

Stream Load 不允许您使用 SHOW LOAD 语句查询加载作业的结果。

#### 取消 Stream Load 作业

Stream Load 不允许您取消加载作业。如果加载作业超时或遇到错误，StarRocks会自动取消作业。

### 参数配置

本节描述了您需要配置的一些系统参数，如果选择使用加载方法 Stream Load，则这些参数配置会对所有 Stream Load 作业生效。

- `streaming_load_max_mb` : 您想要加载的每个数据文件的最大大小。默认的最大大小为 10GB。有关更多信息，请参见[配置 BE 动态参数](../administration/Configuration.md#configure-be-dynamic-parameters)。

  我们建议您一次不要加载超过 10 GB 的数据。如果数据文件的大小超过 10 GB，建议您将数据文件拆分为小文件，每个文件的大小都小于 10 GB，然后依次加载这些文件。如果无法拆分超过 10GB 的数据文件，可以基于文件大小增加此参数值。

  增加此参数值后，新值只能在重启 StarRocks 集群的 BE 后生效。此外，系统性能可能下降，并且在加载失败时重试成本也会增加。

  :::注意

  当您加载 JSON 文件的数据时，请注意以下几点：

  - 文件中每个 JSON 对象的大小不能超过 4GB。如果文件中的任何 JSON 对象大小超过 4GB，StarRocks将抛出错误"这个解析器无法支持如此大的文档。"

  - 默认情况下，HTTP 请求中的 JSON body 不能超过 100MB。如果 JSON body 超过 100MB，StarRocks将抛出错误"这个批次的大小超过了 json 类型数据的最大大小 [104857600] [8617627793]。设置 ignore_json_size 来跳过检查，尽管它可能会消耗大量的内存。" 为了防止此错误，您可以在 HTTP 请求头中添加 `"ignore_json_size:true"` 来忽略对 JSON body 大小的检查。

  :::

- `stream_load_default_timeout_second`: 每个加载作业的超时期限。默认的超时期限为 600 秒。有关更多信息，请参见[配置 FE 动态参数](../administration/Configuration.md#configure-fe-dynamic-parameters)。

  如果您创建的许多加载作业超时，可以根据以下公式的计算结果增加此参数值：

  **每个加载作业的超时期限 > 要加载的数据量/平均加载速度**

  例如，如果您要加载的数据文件大小为 10GB，您的 StarRocks 集群的平均加载速度为 100MB/s，将超时期限设置为 100 秒以上。

  :::注意

  前面公式中的**平均加载速度**是指您的 StarRocks 集群的平均加载速度。这取决于磁盘I/O和您的StarRocks集群中 BE 的数量。

  :::

  Stream Load 还提供了 `timeout` 参数，允许您指定单个加载作业的超时期限。有关更多信息，请参见[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。

### 使用说明

如果要加载的数据文件记录的字段缺失，并且该字段映射到您的 StarRocks 表中已定义为 `NOT NULL`，StarRocks 在加载记录时会自动在 StarRocks 表的映射列中填充一个 `NULL` 值。您还可以使用 `ifnull()` 函数指定要填充的默认值。

例如，如果前面的 `example2.json` 文件中表示城市ID的字段缺失，并且您要在 `table2` 的映射列中填充一个 `x` 值，则可以指定 `"columns: city, tmp_id, id = ifnull(tmp_id, 'x')"`。

## 通过 Broker Load 从本地文件系统加载

除了 Stream Load，您还可以使用 Broker Load 从本地文件系统加载数据。从 v2.5 开始支持此功能。

Broker Load 是一种异步的加载方法。提交加载作业后，StarRocks会异步运行该作业，并不会立即返回作业结果。您需要手动查询该作业结果。请参见[检查 Broker Load 进度](#check-broker-load-progress)。

### 限制

- 目前，Broker Load 仅支持通过单个版本为 v2.5或更高版本的单个 broker 从本地文件系统中加载数据。
- 针对单个 broker 的高并发查询可能导致超时和OOM等问题。为减轻影响，您可以使用 `pipeline_dop` 变量（请参见[系统变量](../reference/System_variable.md#pipeline_dop)）设置 Broker Load 的查询并行度。对于对单个 broker 的查询，建议您将 `pipeline_dop` 设置为小于 `16` 的值。

### 开始之前

在使用 Broker Load 从本地文件系统加载数据之前，完成以下准备工作：

1. 根据 "[部署先决条件](../deployment/deployment_prerequisites.md)"、"[检查环境配置](../deployment/environment_configurations.md)" 和 "[准备部署文件](../deployment/prepare_deployment_files.md)" 中的说明配置存储本地文件的计算机，然后在该计算机上部署一个 broker。操作方式与在 BE 节点上部署代理相同。有关详细操作，请参见[部署和管理 Broker 节点](../deployment/deploy_broker.md)。

   > **提醒**
   >
   > 仅部署单个 broker，并确保该 broker 版本为 v2.5 或更高。

2. 执行 [ALTER SYSTEM](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md#broker) 将前面步骤中部署的 broker 添加到您的 StarRocks 集群中，并为 broker 定义一个新名称。下面示例将 broker `172.26.199.40:8000` 添加到 StarRocks 集群，并将 broker 名称定义为 `sole_broker`：

   ```SQL
   ALTER SYSTEM ADD BROKER sole_broker "172.26.199.40:8000";
   ```
   
### 示例

Broker Load 支持从单个数据文件加载到单个表，从多个数据文件加载到单个表，以及从多个数据文件加载到多个表。本节以从多个数据文件加载到单个表为例。
注意，在StarRocks中，一些文字被SQL语言作为保留关键词使用。不要直接在SQL语句中使用这些关键词。如果要在SQL语句中使用此类关键词，请用一对反引号（`）将其括起来。请参见[关键词](../sql-reference/sql-statements/keywords.md)。

#### 准备数据集

以CSV文件格式为例。登录到本地文件系统，创建两个CSV文件`file1.csv`和`file2.csv`，保存在特定存储位置（例如`/user/starrocks/`）中。两个文件均由三列组成，分别表示用户ID、用户名和用户分数。

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

创建名为`mytable`的主键表。该表由三列组成：`id`、`name`和`score`，其中`id`为主键。

```SQL
CREATE TABLE `mytable`
(
    `id` int(11) NOT NULL COMMENT "用户ID",
    `name` varchar(65533) NULL DEFAULT "" COMMENT "用户名",
    `score` int(11) NOT NULL DEFAULT "0" COMMENT "用户分数"
)
ENGINE=OLAP
PRIMARY KEY(`id`)
DISTRIBUTED BY HASH(`id`)
PROPERTIES("replication_num"="1");
```

#### 启动Broker Load

运行以下命令启动一个Broker Load作业，将所有位于本地文件系统路径`/user/starrocks/`中的数据文件（`file1.csv`和`file2.csv`）加载到StarRocks表`mytable`中：

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

该作业包含四个主要部分：

- `LABEL`：用于查询加载作业状态的字符串。
- `LOAD`声明：源URI、源数据格式和目标表名。
- `BROKER`：代理的名称。
- `PROPERTIES`：加载作业的超时值和任何其他属性。

有关详细语法和参数描述，请参见[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

#### 检查Broker Load进度

在v3.0及更早的版本中，使用[SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)语句或curl命令查看Broker Load作业的进度。

在v3.1及更高版本中，您可以从[`information_schema.loads`](../reference/information_schema/loads.md)视图中查看Broker Load作业的进度：

```SQL
SELECT * FROM information_schema.loads;
```

如果已提交多个加载作业，您可以根据作业关联的`LABEL`进行筛选。例如：

```SQL
SELECT * FROM information_schema.loads WHERE LABEL = 'label_local';
```

确认加载作业已经完成后，您可以查询表以查看数据是否已经成功加载。例如：

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

#### 取消Broker Load作业

当加载作业不处于**CANCELLED**或**FINISHED**阶段时，可以使用[CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md)语句取消作业。

例如，您可以执行以下语句取消名为`label_local`的加载作业，位于数据库`mydatabase`中：

```SQL
CANCEL LOAD
FROM mydatabase
WHERE LABEL = "label_local";
```

## 通过Broker Load从NAS加载

有两种方法可以通过Broker Load从NAS加载数据：

- 将NAS视为本地文件系统，并使用代理运行加载作业。请参见前一节"[通过Broker Load从本地文件系统加载](#loading-from-a-local-file-system-via-broker-load)"。
- （推荐）将NAS视为云存储系统，并无需代理运行加载作业。

本节介绍第二种方法。具体操作如下：

1. 将NAS设备挂载到StarRocks集群的所有BE节点和FE节点上的相同路径。这样，所有BE可以访问NAS设备，就像访问本地存储文件一样。

2. 使用Broker Load从NAS设备加载数据到目标的StarRocks表。例如：

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

   该作业包含四个主要部分：

   - `LABEL`：用于查询加载作业状态的字符串。
   - `LOAD`声明：源URI、源数据格式和目标表名。需注意，声明中的`DATA INFILE`用于指定NAS设备的挂载点文件夹路径，就像上述示例中展示的一样，`file:///`是前缀，`/home/disk1/sr`是挂载点的文件夹路径。
   - `BROKER`：您无需指定代理名称。
   - `PROPERTIES`：加载作业的超时值和任何其他属性。

   有关详细语法和参数描述，请参见[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

提交作业后，您可以根据需要查看加载进度或取消作业。有关详细操作，请参见本主题中的"[检查Broker Load进度](#检查broker-load进度)"和"[取消Broker Load作业](#取消broker-load作业)"。