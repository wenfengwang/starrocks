---
displayed_sidebar: English
---

# 使用 Stream Load 事务接口加载数据

从 v2.4 开始，StarRocks 提供 Stream Load 事务接口，用于实现从 Apache Flink® 和 Apache Kafka® 等外部系统加载数据的两阶段提交（2PC）。Stream Load 事务接口有助于提高高并发流加载的性能。

本主题介绍 Stream Load 事务接口以及如何使用该接口将数据加载到 StarRocks 中。

<InsertPrivNote />


## 描述

Stream Load 事务接口支持使用兼容 HTTP 协议的工具或语言调用 API 操作。本主题以 curl 为例，介绍如何使用该接口。该接口提供了事务管理、数据写入、事务预提交、事务去重和事务超时管理等多种功能。

### 事务管理

Stream Load 事务接口提供以下 API 操作，用于管理事务：

- `/api/transaction/begin`：开始一个新事务。

- `/api/transaction/commit`：提交当前事务以使数据更改持久化。

- `/api/transaction/rollback`：回滚当前事务以中止数据更改。

### 事务预提交

Stream Load 事务接口提供 `/api/transaction/prepare` 操作，用于预提交当前事务并使数据更改暂时持久化。预提交事务后，您可以继续提交或回滚事务。如果您的 StarRocks 集群在事务预提交后发生故障，您仍然可以在 StarRocks 集群恢复正常后继续提交事务。

> **注意**
> 事务预提交后，不要继续使用事务写入数据。如果您继续使用事务写入数据，您的写入请求将返回错误。

### 数据写入

Stream Load 事务接口提供 `/api/transaction/load` 操作，用于写入数据。您可以在一个事务中多次调用此操作。

### 事务去重

Stream Load 事务接口延续了 StarRocks 的标签机制。您可以为每个事务绑定一个唯一的标签，以实现事务的至多一次保证。

### 事务超时管理

您可以使用每个 FE 的配置文件中的 `stream_load_default_timeout_second` 参数来指定该 FE 的默认事务超时时间。

创建事务时，您可以使用 HTTP 请求头中的 `timeout` 字段来指定事务的超时时间。

创建事务时，您还可以使用 HTTP 请求头中的 `idle_transaction_timeout` 字段来指定事务可以保持空闲的超时时间。如果在超时时间内没有写入数据，事务将自动回滚。

## 好处

Stream Load 事务接口带来以下好处：

- **精确一次语义**

  事务分为预提交和提交两个阶段，这使得跨系统加载数据变得容易。例如，该接口可以保证从 Flink 加载数据的精确一次语义。

- **提升加载性能**

  如果您使用程序运行加载作业，Stream Load 事务接口允许您按需合并多个小批量数据，然后通过调用 `/api/transaction/commit` 操作在一个事务中一次性发送所有数据。这样可以减少需要加载的数据版本，提升加载性能。

## 限制

Stream Load 事务接口有以下限制：

- 仅支持**单数据库单表**事务。对**多数据库多表**事务的支持正在开发中。

- 仅支持**单客户端并发数据写入**。对**多客户端并发数据写入**的支持正在开发中。

- `/api/transaction/load` 操作可以在一个事务内多次调用。在这种情况下，为所有调用的 `/api/transaction/load` 操作指定的参数设置必须相同。

- 使用 Stream Load 事务接口加载 CSV 格式的数据时，请确保数据文件中的每条数据记录以行分隔符结尾。

## 注意事项

- 如果您调用的 `/api/transaction/begin`、`/api/transaction/load` 或 `/api/transaction/prepare` 操作返回错误，则事务失败并自动回滚。
- 调用 `/api/transaction/begin` 操作开始新事务时，您可以选择指定一个标签。如果您不指定标签，StarRocks 将为事务生成一个标签。请注意，后续的 `/api/transaction/load`、`/api/transaction/prepare` 和 `/api/transaction/commit` 操作必须使用与 `/api/transaction/begin` 操作相同的标签。
- 如果您使用前一个事务的标签调用 `/api/transaction/begin` 操作来启动新事务，则前一个事务将失败并回滚。
- StarRocks 支持的 CSV 格式数据的默认列分隔符和行分隔符是 `\t` 和 `\n`。如果您的数据文件不使用默认的列分隔符或行分隔符，则必须在调用 `/api/transaction/load` 操作时使用 `"column_separator: <column_separator>"` 或 `"row_delimiter: <row_delimiter>"` 来指定数据文件中实际使用的列分隔符或行分隔符。

## 基本操作

### 准备样本数据

本主题以 CSV 格式的数据为例。

1. 在您的本地文件系统的 `/home/disk1/` 路径中，创建一个名为 `example1.csv` 的 CSV 文件。该文件包含三列，依次代表用户 ID、用户名和用户分数。

   ```Plain
   1,Lily,23
   2,Rose,23
   3,Alice,24
   4,Julia,25
   ```

2. 在您的 StarRocks 数据库 `test_db` 中，创建一个名为 `table1` 的主键表。该表包含三列：`id`、`name` 和 `score`，其中 `id` 是主键。

   ```SQL
   CREATE TABLE `table1`
   (
       `id` int(11) NOT NULL COMMENT "user ID",
       `name` varchar(65533) NULL COMMENT "user name",
       `score` int(11) NOT NULL COMMENT "user score"
   )
   ENGINE=OLAP
   PRIMARY KEY(`id`)
   DISTRIBUTED BY HASH(`id`) BUCKETS 10;
   ```

### 开始事务

#### 语法

```Bash
curl --location-trusted -u <username>:<password> -H "label:<label_name>" \
    -H "Expect:100-continue" \
    -H "db:<database_name>" -H "table:<table_name>" \
    -XPOST http://<fe_host>:<fe_http_port>/api/transaction/begin
```

#### 示例

```Bash
curl --location-trusted -u <jack>:<123456> -H "label:streamload_txn_example1_table1" \
    -H "Expect:100-continue" \
    -H "db:test_db" -H "table:table1" \
    -XPOST http://<fe_host>:<fe_http_port>/api/transaction/begin
```

> **注意**
> 对于本示例，`streamload_txn_example1_table1` 被指定为事务的标签。

#### 返回结果

- 如果事务成功启动，将返回以下结果：

  ```Bash
  {
      "Status": "OK",
      "Message": "",
      "Label": "streamload_txn_example1_table1",
      "TxnId": 9032,
      "BeginTxnTimeMs": 0
  }
  ```

- 如果事务绑定了重复的标签，将返回以下结果：

  ```Bash
  {
      "Status": "LABEL_ALREADY_EXISTS",
      "ExistingJobStatus": "RUNNING",
      "Message": "Label [streamload_txn_example1_table1] has already been used."
  }
  ```

- 如果出现除重复标签以外的其他错误，将返回以下结果：

  ```Bash
  {
      "Status": "FAILED",
      "Message": ""
  }
  ```

### 写入数据

#### 语法

```Bash
curl --location-trusted -u <username>:<password> -H "label:<label_name>" \
    -H "Expect:100-continue" \
    -H "db:<database_name>" -H "table:<table_name>" \
    -T <file_path> \
    -XPUT http://<fe_host>:<fe_http_port>/api/transaction/load
```

> **注意**
> 调用 `/api/transaction/load` 操作时，必须使用 `<file_path>` 来指定您要加载的数据文件的保存路径。
```
> 调用`/api/transaction/load`操作时，必须使用`<file_path>`指定要加载的数据文件的保存路径。

#### 示例

```Bash
curl --location-trusted -u <jack>:<123456> -H "label:streamload_txn_example1_table1" \
    -H "Expect:100-continue" \
    -H "db:test_db" -H "table:table1" \
    -T /home/disk1/example1.csv \
    -H "column_separator: ," \
    -XPUT http://<fe_host>:<fe_http_port>/api/transaction/load
```

> **注意**
> 在本示例中，数据文件`example1.csv`使用的列分隔符是逗号(`,`)，而不是StarRocks的默认列分隔符(`\t`)。因此，在调用`/api/transaction/load`操作时，必须使用`"column_separator: <column_separator>"`来指定逗号(`,`)作为列分隔符。

#### 返回结果

- 如果数据写入成功，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Seq": 0,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": "",
      "NumberTotalRows": 5265644,
      "NumberLoadedRows": 5265644,
      "NumberFilteredRows": 0,
      "NumberUnselectedRows": 0,
      "LoadBytes": 10737418067,
      "LoadTimeMs": 418778,
      "StreamLoadPutTimeMs": 68,
      "ReceivedDataTimeMs": 38964,
  }
  ```

- 如果交易被认为是未知的，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "TXN_NOT_EXISTS"
  }
  ```

- 如果交易被认为处于无效状态，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transaction State Invalid"
  }
  ```

- 如果出现除未知交易和无效状态以外的错误，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": ""
  }
  ```

### 预提交交易

#### 语法

```Bash
curl --location-trusted -u <username>:<password> -H "label:<label_name>" \
    -H "Expect:100-continue" \
    -H "db:<database_name>" \
    -XPOST http://<fe_host>:<fe_http_port>/api/transaction/prepare
```

#### 示例

```Bash
curl --location-trusted -u <jack>:<123456> -H "label:streamload_txn_example1_table1" \
    -H "Expect:100-continue" \
    -H "db:test_db" \
    -XPOST http://<fe_host>:<fe_http_port>/api/transaction/prepare
```

#### 返回结果

- 如果预提交成功，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": "",
      "NumberTotalRows": 5265644,
      "NumberLoadedRows": 5265644,
      "NumberFilteredRows": 0,
      "NumberUnselectedRows": 0,
      "LoadBytes": 10737418067,
      "LoadTimeMs": 418778,
      "StreamLoadPutTimeMs": 68,
      "ReceivedDataTimeMs": 38964,
      "WriteDataTimeMs": 417851,
      "CommitAndPublishTimeMs": 1393
  }
  ```

- 如果交易被认为不存在，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transaction Not Exist"
  }
  ```

- 如果预提交超时，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "commit timeout",
  }
  ```

- 如果出现除交易不存在和预提交超时以外的错误，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "publish timeout"
  }
  ```

### 提交交易

#### 语法

```Bash
curl --location-trusted -u <username>:<password> -H "label:<label_name>" \
    -H "Expect:100-continue" \
    -H "db:<database_name>" \
    -XPOST http://<fe_host>:<fe_http_port>/api/transaction/commit
```

#### 示例

```Bash
curl --location-trusted -u <jack>:<123456> -H "label:streamload_txn_example1_table1" \
    -H "Expect:100-continue" \
    -H "db:test_db" \
    -XPOST http://<fe_host>:<fe_http_port>/api/transaction/commit
```

#### 返回结果

- 如果提交成功，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": "",
      "NumberTotalRows": 5265644,
      "NumberLoadedRows": 5265644,
      "NumberFilteredRows": 0,
      "NumberUnselectedRows": 0,
      "LoadBytes": 10737418067,
      "LoadTimeMs": 418778,
      "StreamLoadPutTimeMs": 68,
      "ReceivedDataTimeMs": 38964,
      "WriteDataTimeMs": 417851,
      "CommitAndPublishTimeMs": 1393
  }
  ```

- 如果交易已经提交，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": "Transaction already committed",
  }
  ```

- 如果交易被认为不存在，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transaction Not Exist"
  }
  ```

- 如果提交超时，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "commit timeout",
  }
  ```

- 如果数据发布超时，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "publish timeout",
      "CommitAndPublishTimeMs": 1393
  }
  ```

- 如果出现除交易不存在和超时以外的错误，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": ""
  }
  ```

### 回滚交易

#### 语法

```Bash
curl --location-trusted -u <username>:<password> -H "label:<label_name>" \
    -H "Expect:100-continue" \
    -H "db:<database_name>" \
    -XPOST http://<fe_host>:<fe_http_port>/api/transaction/rollback
```

#### 示例

```Bash
curl --location-trusted -u <jack>:<123456> -H "label:streamload_txn_example1_table1" \
    -H "Expect:100-continue" \
    -H "db:test_db" \
    -XPOST http://<fe_host>:<fe_http_port>/api/transaction/rollback
```

#### 返回结果

- 如果回滚成功，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": ""
  }
  ```

- 如果交易被认为不存在，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transaction Not Exist"
  }
  ```

- 如果出现除交易不存在以外的错误，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
# 使用流式加载事务接口加载数据

从v2.4版本开始，StarRocks提供了一个流式加载事务接口，用于实现从外部系统（如Apache Flink®和Apache Kafka®）加载数据的事务的两阶段提交（2PC）。流式加载事务接口有助于提高高并发流式加载的性能。

本主题介绍了流式加载事务接口以及如何使用该接口将数据加载到StarRocks中。

<InsertPrivNote />


## 描述

流式加载事务接口支持使用与HTTP协议兼容的工具或语言调用API操作。本主题以curl为例，说明如何使用该接口。该接口提供了各种功能，如事务管理、数据写入、事务预提交、事务去重和事务超时管理。

### 事务管理

流式加载事务接口提供以下API操作，用于管理事务：

- `/api/transaction/begin`：启动一个新事务。

- `/api/transaction/commit`：提交当前事务，使数据更改持久化。

- `/api/transaction/rollback`：回滚当前事务，中止数据更改。

### 事务预提交

流式加载事务接口提供`/api/transaction/prepare`操作，用于预提交当前事务并使数据更改暂时持久化。在预提交事务后，您可以继续提交或回滚事务。如果StarRocks集群在事务预提交后发生故障，您可以在StarRocks集群恢复正常后继续提交事务。

> **注意**
> 事务预提交后，请勿继续使用该事务写入数据。如果继续使用该事务写入数据，您的写入请求将返回错误。

### 数据写入

流式加载事务接口提供`/api/transaction/load`操作，用于写入数据。您可以在一个事务中多次调用此操作。

### 事务去重

流式加载事务接口继承了StarRocks的标签机制。您可以为每个事务绑定一个唯一的标签，以实现事务的至多一次保证。

### 事务超时管理

您可以使用每个FE的配置文件中的`stream_load_default_timeout_second`参数来指定该FE的默认事务超时时间。

创建事务时，您可以使用HTTP请求头中的`timeout`字段来指定事务的超时时间。

创建事务时，您还可以使用HTTP请求头中的`idle_transaction_timeout`字段来指定事务可以保持空闲的超时时间。如果在超时时间内没有写入数据，则事务会自动回滚。

## 优势

流式加载事务接口带来以下优势：

- **精确一次语义**

  事务分为预提交和提交两个阶段，使得在系统之间加载数据变得容易。例如，该接口可以保证从Flink加载的数据的精确一次语义。

- **提高加载性能**

  如果您使用程序运行加载作业，流式加载事务接口允许您按需合并多个小批量数据，然后通过调用`/api/transaction/commit`操作一次性发送所有数据。这样，需要加载的数据版本较少，加载性能得到提高。

## 限制

流式加载事务接口具有以下限制：

- 仅支持**单数据库单表**事务。对于**多数据库多表**事务的支持正在开发中。

- 仅支持**来自一个客户端的并发数据写入**。对于**来自多个客户端的并发数据写入**的支持正在开发中。

- `/api/transaction/load`操作可以在一个事务中多次调用。在这种情况下，调用的所有`/api/transaction/load`操作的参数设置必须相同。

- 当使用流式加载事务接口加载CSV格式的数据时，请确保数据文件中的每条数据记录以行分隔符结尾。

## 注意事项

- 如果您调用的`/api/transaction/begin`、`/api/transaction/load`或`/api/transaction/prepare`操作返回错误，则事务失败并自动回滚。
- 调用`/api/transaction/begin`操作启动新事务时，您可以选择指定一个标签。如果不指定标签，StarRocks将为事务生成一个标签。请注意，后续的`/api/transaction/load`、`/api/transaction/prepare`和`/api/transaction/commit`操作必须使用与`/api/transaction/begin`操作相同的标签。
- 如果您使用上一个事务的标签调用`/api/transaction/begin`操作启动新事务，上一个事务将失败并回滚。
- StarRocks支持CSV格式数据的默认列分隔符和行分隔符分别为`\t`和`\n`。如果您的数据文件不使用默认的列分隔符或行分隔符，请在调用`/api/transaction/load`操作时使用`"column_separator: <column_separator>"`或`"row_delimiter: <row_delimiter>"`来指定您的数据文件中实际使用的列分隔符或行分隔符。

## 基本操作

### 准备示例数据

本主题以CSV格式的数据为例。

1. 在本地文件系统的`/home/disk1/`路径下创建一个名为`example1.csv`的CSV文件。该文件由三列组成，分别表示用户ID、用户名和用户分数。

   ```Plain
   1,Lily,23
   2,Rose,23
   3,Alice,24
   4,Julia,25
   ```

2. 在StarRocks数据库`test_db`中创建一个名为`table1`的主键表。该表由三列组成：`id`、`name`和`score`，其中`id`是主键。

   ```SQL
   CREATE TABLE `table1`
   (
       `id` int(11) NOT NULL COMMENT "用户ID",
       `name` varchar(65533) NULL COMMENT "用户名",
       `score` int(11) NOT NULL COMMENT "用户分数"
   )
   ENGINE=OLAP
   PRIMARY KEY(`id`)
   DISTRIBUTED BY HASH(`id`) BUCKETS 10;
   ```

### 启动事务

#### 语法

```Bash
curl --location-trusted -u <用户名>:<密码> -H "label:<标签名>" \
    -H "Expect:100-continue" \
    -H "db:<数据库名>" -H "table:<表名>" \
    -XPOST http://<fe主机>:<fe_http端口>/api/transaction/begin
```

#### 示例

```Bash
curl --location-trusted -u <jack>:<123456> -H "label:streamload_txn_example1_table1" \
    -H "Expect:100-continue" \
    -H "db:test_db" -H "table:table1" \
    -XPOST http://<fe主机>:<fe_http端口>/api/transaction/begin
```

> **注意**
> 对于此示例，将`streamload_txn_example1_table1`指定为事务的标签。

#### 返回结果

- 如果成功启动事务，将返回以下结果：

  ```Bash
  {
      "Status": "OK",
      "Message": "",
      "Label": "streamload_txn_example1_table1",
      "TxnId": 9032,
      "BeginTxnTimeMs": 0
  }
  ```

- 如果事务绑定了重复的标签，将返回以下结果：

  ```Bash
  {
      "Status": "LABEL_ALREADY_EXISTS",
      "ExistingJobStatus": "RUNNING",
      "Message": "标签 [streamload_txn_example1_table1] 已被使用。"
  }
  ```

- 如果发生除重复标签以外的其他错误，将返回以下结果：

  ```Bash
  {
      "Status": "FAILED",
      "Message": ""
  }
  ```

### 写入数据

#### 语法

```Bash
curl --location-trusted -u <用户名>:<密码> -H "label:<标签名>" \
    -H "Expect:100-continue" \
    -H "db:<数据库名>" -H "table:<表名>" \
    -T <文件路径> \
    -XPUT http://<fe主机>:<fe_http端口>/api/transaction/load
```

> **注意**
> 在调用`/api/transaction/load`操作时，您必须使用`<文件路径>`来指定要加载的数据文件的保存路径。

#### 示例

```Bash
curl --location-trusted -u <jack>:<123456> -H "label:streamload_txn_example1_table1" \
    -H "Expect:100-continue" \
    -H "db:test_db" -H "table:table1" \
    -T /home/disk1/example1.csv \
    -H "column_separator: ," \
    -XPUT http://<fe主机>:<fe_http端口>/api/transaction/load
```

> **注意**
> 对于此示例，数据文件`example1.csv`中使用的列分隔符是逗号（`,`），而不是StarRocks的默认列分隔符（`\t`）。因此，在调用`/api/transaction/load`操作时，您必须使用`"column_separator: <column_separator>"`来指定逗号（`,`）作为列分隔符。

#### 返回结果

- 如果数据写入成功，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Seq": 0,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": "",
      "NumberTotalRows": 5265644,
      "NumberLoadedRows": 5265644,
      "NumberFilteredRows": 0,
      "NumberUnselectedRows": 0,
      "LoadBytes": 10737418067,
      "LoadTimeMs": 418778,
      "StreamLoadPutTimeMs": 68,
      "ReceivedDataTimeMs": 38964,
  }
  ```

- 如果事务被认为是未知的，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "TXN_NOT_EXISTS"
  }
  ```

- 如果事务被认为处于无效状态，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transcation State Invalid"
  }
  ```

- 如果发生除未知事务和无效状态以外的其他错误，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": ""
  }
  ```

### 预提交事务

#### 语法

```Bash
curl --location-trusted -u <用户名>:<密码> -H "label:<标签名>" \
    -H "Expect:100-continue" \
    -H "db:<数据库名>" \
    -XPOST http://<fe主机>:<fe_http端口>/api/transaction/prepare
```

#### 示例

```Bash
curl --location-trusted -u <jack>:<123456> -H "label:streamload_txn_example1_table1" \
    -H "Expect:100-continue" \
    -H "db:test_db" \
    -XPOST http://<fe主机>:<fe_http端口>/api/transaction/prepare
```

#### 返回结果

- 如果预提交成功，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": "",
      "NumberTotalRows": 5265644,
      "NumberLoadedRows": 5265644,
      "NumberFilteredRows": 0,
      "NumberUnselectedRows": 0,
      "LoadBytes": 10737418067,
      "LoadTimeMs": 418778,
      "StreamLoadPutTimeMs": 68,
      "ReceivedDataTimeMs": 38964,
      "WriteDataTimeMs": 417851
      "CommitAndPublishTimeMs": 1393
  }
  ```

- 如果事务被认为不存在，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transcation Not Exist"
  }
  ```

- 如果预提交超时，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "commit timeout",
  }
  ```

- 如果发生除不存在事务和预提交超时以外的其他错误，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "publish timeout"
  }
  ```

### 提交事务

#### 语法

```Bash
curl --location-trusted -u <用户名>:<密码> -H "label:<标签名>" \
    -H "Expect:100-continue" \
    -H "db:<数据库名>" \
    -XPOST http://<fe主机>:<fe_http端口>/api/transaction/commit
```

#### 示例

```Bash
curl --location-trusted -u <jack>:<123456> -H "label:streamload_txn_example1_table1" \
    -H "Expect:100-continue" \
    -H "db:test_db" \
    -XPOST http://<fe主机>:<fe_http端口>/api/transaction/commit
```

#### 返回结果

- 如果提交成功，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": "",
      "NumberTotalRows": 5265644,
      "NumberLoadedRows": 5265644,
      "NumberFilteredRows": 0,
      "NumberUnselectedRows": 0,
      "LoadBytes": 10737418067,
      "LoadTimeMs": 418778,
      "StreamLoadPutTimeMs": 68,
      "ReceivedDataTimeMs": 38964,
      "WriteDataTimeMs": 417851
      "CommitAndPublishTimeMs": 1393
  }
  ```

- 如果事务已经提交，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": "Transaction already commited",
  }
  ```

- 如果事务被认为不存在，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transcation Not Exist"
  }
  ```

- 如果提交超时，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "commit timeout",
  }
  ```

- 如果数据发布超时，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "publish timeout",
      "CommitAndPublishTimeMs": 1393
  }
  ```

- 如果发生除不存在事务和超时以外的其他错误，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": ""
  }
  ```

### 回滚事务

#### 语法

```Bash
curl --location-trusted -u <用户名>:<密码> -H "label:<标签名>" \
    -H "Expect:100-continue" \
    -H "db:<数据库名>" \
    -XPOST http://<fe主机>:<fe_http端口>/api/transaction/rollback
```

#### 示例

```Bash
curl --location-trusted -u <jack>:<123456> -H "label:streamload_txn_example1_table1" \
    -H "Expect:100-continue" \
    -H "db:test_db" \
    -XPOST http://<fe主机>:<fe_http端口>/api/transaction/rollback
```

#### 返回结果

- 如果回滚成功，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": ""
  }
  ```

- 如果事务被认为不存在，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transcation Not Exist"
  }
  ```

- 如果发生除不存在事务以外的其他错误，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": ""
  }
  ```

## 参考

关于Stream Load的适用场景和支持的数据文件格式，以及Stream Load的工作原理，请参见[通过Stream Load从本地文件系统加载](../loading/StreamLoad.md#loading-from-a-local-file-system-via-stream-load)。
