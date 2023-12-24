---
displayed_sidebar: English
---

# 使用 Stream Load 事务接口加载数据

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

从 v2.4 版本开始，StarRocks 提供了 Stream Load 事务接口，用于实现从外部系统（如 Apache Flink® 和 Apache Kafka®）加载数据的事务的两阶段提交（2PC）。Stream Load 事务接口有助于提高高并发流负载的性能。

本主题描述了 Stream Load 事务接口以及如何通过该接口将数据加载到 StarRocks 中。

<InsertPrivNote />

## 描述

Stream Load 事务接口支持使用兼容 HTTP 协议的工具或语言调用 API 操作。本主题以 curl 为例，说明如何使用该接口。该接口提供了事务管理、数据写入、事务预提交、事务重复数据删除和事务超时管理等功能。

### 事务管理

Stream Load 事务接口提供以下 API 操作，用于管理事务：

- `/api/transaction/begin`：启动新事务。

- `/api/transaction/commit`：提交当前事务以使数据更改持久化。

- `/api/transaction/rollback`：回滚当前事务以中止数据更改。

### 事务预提交

Stream Load 事务接口提供 `/api/transaction/prepare` 操作，用于预提交当前事务并使数据更改暂时持久化。预提交事务后，可以继续提交或回滚事务。如果您的 StarRocks 集群在交易预提交后发生故障，您仍然可以在 StarRocks 集群恢复正常后继续提交交易。

> **注意**
>
> 事务预提交后，不要继续使用事务写入数据。如果继续使用事务写入数据，则写入请求将返回错误。

### 数据写入

Stream Load 事务接口提供 `/api/transaction/load` 操作，用于写入数据。您可以在一个事务中多次调用该操作。

### 事务重复数据删除

Stream Load 事务接口继承了 StarRocks 的标注机制。您可以为每笔交易绑定一个唯一的标签，以实现交易最多一次的保证。

### 事务超时管理

您可以使用每个 FE 配置文件中的 `stream_load_default_timeout_second` 参数来指定该 FE 的默认事务超时时间。

创建事务时，可以使用 HTTP 请求标头中的 `timeout` 字段来指定事务的超时期限。

创建事务时，还可以使用 HTTP 请求标头中的 `idle_transaction_timeout` 字段来指定事务可以保持空闲状态的超时期限。如果在超时期限内没有写入任何数据，则事务会自动回滚。

## 好处

Stream Load 事务接口带来以下好处：

- **Exactly-once 语义**

  事务分为两个阶段，即预提交和提交，这使得跨系统加载数据变得容易。例如，该接口可以保证从 Flink 加载数据的语义是 exactly-once。

- **改进的负载性能**

  如果使用程序运行加载作业，则流加载事务接口允许您按需合并多个小批量数据，然后通过调用 `/api/transaction/commit` 操作在一个事务中一次性发送所有数据。因此，需要加载的数据版本更少，加载性能得到提高。

## 限制

Stream Load 事务接口具有以下限制：

- 仅支持**单数据库单表**事务。对**多数据库多表**事务的支持正在开发中。

- 仅支持**来自一个客户端的并发数据写入**。对**来自多个客户端的并发数据写入的支持**正在开发中。

- 可以在一个事务中多次调用 `/api/transaction/load` 操作。在这种情况下，所有调用操作指定的参数设置必须相同。

- 使用流加载事务接口加载 CSV 格式的数据时，请确保数据文件中的每条数据记录都以行分隔符结尾。

## 预防措施

- 如果调用的 `/api/transaction/begin`、`/api/transaction/load` 或 `/api/transaction/prepare` 操作返回错误，则事务将失败并自动回滚。
- 调用 `/api/transaction/begin` 操作以启动新事务时，可以选择指定标签。如果您不指定标签，StarRocks 会为交易生成标签。请注意，后续的 `/api/transaction/load`、`/api/transaction/prepare` 和 `/api/transaction/commit` 操作必须使用与 `/api/transaction/begin` 操作相同的标签。
- 如果使用上一个事务的标签调用 `/api/transaction/begin` 操作以启动新事务，则上一个事务将失败并被回滚。
- StarRocks 默认支持的 CSV 格式数据的列分隔符和行分隔符是 `\t` 和 `\n`。如果数据文件不使用默认的列分隔符或行分隔符，则必须在调用 `/api/transaction/load` 操作时使用 `"column_separator: <column_separator>"` 或 `"row_delimiter: <row_delimiter>"` 指定数据文件中实际使用的列分隔符或行分隔符。

## 基本操作

### 准备示例数据

本主题以 CSV 格式数据为例。

1. 在本地文件系统的 `/home/disk1/` 路径中，创建名为 `example1.csv` 的 CSV 文件。该文件由三列组成，分别依次表示用户 ID、用户名和用户分数。

   ```Plain
   1,Lily,23
   2,Rose,23
   3,Alice,24
   4,Julia,25
   ```

2. 在 StarRocks 数据库 `test_db` 中，创建一个名为 `table1` 的主键表。该表由三列组成：`id`、`name` 和 `score`，其中 `id` 是主键。

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
>
> 在此示例中，`streamload_txn_example1_table1` 被指定为事务的标签。

#### 返回结果

- 如果事务成功启动，则返回以下结果：

  ```Bash
  {
      "Status": "OK",
      "Message": "",
      "Label": "streamload_txn_example1_table1",
      "TxnId": 9032,
      "BeginTxnTimeMs": 0
  }
  ```

- 如果事务绑定到重复标签，则返回以下结果：

  ```Bash
  {
      "Status": "LABEL_ALREADY_EXISTS",
      "ExistingJobStatus": "RUNNING",
      "Message": "Label [streamload_txn_example1_table1] has already been used."
  }
  ```

- 如果出现重复标签以外的错误，则返回以下结果：

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
>
> 在调用 `/api/transaction/load` 操作时，必须使用 `<file_path>` 来指定要加载的数据文件的保存路径。

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
>
> 对于这个示例，数据文件 `example1.csv` 中使用的列分隔符是逗号（`,`），而不是 StarRocks 默认的列分隔符（`\t`）。因此，在调用 `/api/transaction/load` 操作时，必须使用 `"column_separator: <column_separator>"` 来指定逗号（`,`）作为列分隔符。

#### 返回结果

- 如果数据写入成功，则返回以下结果：

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
      "ReceivedDataTimeMs": 38964
  }
  ```

- 如果事务被视为未知，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "TXN_NOT_EXISTS"
  }
  ```

- 如果事务被视为无效状态，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transaction State Invalid"
  }
  ```

- 如果出现未知事务和无效状态以外的错误，则返回以下结果：

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

- 如果预提交成功，则返回以下结果：

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

- 如果事务被视为不存在，则返回以下结果：

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
      "Message": "commit timeout"
  }
  ```

- 如果出现不存在的事务和预提交超时以外的错误，则返回以下结果：

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

- 如果提交成功，则返回以下结果：

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

- 如果事务已经提交，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": "Transaction already committed"
  }
  ```

- 如果事务被视为不存在，则返回以下结果：

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
      "Message": "commit timeout"
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

- 如果出现不存在的事务和超时以外的错误，则返回以下结果：

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

- 如果回滚成功，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": ""
  }
  ```

- 如果事务被视为不存在，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transaction Not Exist"
  }
  ```

- 如果出现不存在的事务以外的错误，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED"
  }
  ```
      "消息": ""
  }
  ```

## 参考资料

有关流加载的适用场景和支持的数据文件格式，以及流加载的工作原理的信息，请参见[通过流加载从本地文件系统加载](../loading/StreamLoad.md#loading-from-a-local-file-system-via-stream-load)。

有关创建流加载作业的语法和参数的信息，请参阅[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。