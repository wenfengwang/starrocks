---
displayed_sidebar: English
---

# 使用Stream Load事务接口加载数据

从'../assets/commonMarkdown/insertPrivNote.md'导入InsertPrivNote

自v2.4起，StarRocks提供了Stream Load事务接口，用于实现外部系统（如Apache Flink®和Apache Kafka®）加载数据事务的两阶段提交（2PC）。Stream Load事务接口有助于提升高并发流加载的性能。

本主题介绍Stream Load事务接口及如何利用此接口向StarRocks加载数据。

<InsertPrivNote />


## 描述

Stream Load事务接口支持使用兼容HTTP协议的工具或编程语言来调用API操作。本主题将以curl为例，说明如何使用此接口。该接口提供了诸如事务管理、数据写入、事务预提交、事务去重和事务超时管理等多种功能。

### 事务管理

Stream Load事务接口提供以下API操作，用于管理事务：

- /api/transaction/begin：启动一个新事务。

- /api/transaction/commit：提交当前事务，以使数据更改持久化。

- /api/transaction/rollback：回滚当前事务，以放弃数据更改。

### 事务预提交

Stream Load事务接口提供/api/transaction/prepare操作，用于预提交当前事务，使数据更改暂时持久化。在预提交事务之后，您可以选择提交或回滚事务。如果您的StarRocks集群在事务预提交后出现故障，您仍然可以在StarRocks集群恢复正常后继续提交事务。

> **注意**
> 事务预提交后，请勿继续使用该事务写入数据。如果您继续使用该事务写入数据，您的写请求返回错误。

### 数据写入

Stream Load事务接口提供/api/transaction/load操作，用于写入数据。您可以在同一事务中多次调用此操作。

### 事务去重

Stream Load事务接口延续了StarRocks的标签机制。您可以为每个事务绑定一个唯一标签，以实现事务的至多一次执行保证。

### 事务超时管理

您可以在每个FE的配置文件中使用stream_load_default_timeout_second参数来指定该FE的默认事务超时时长。

创建事务时，您可以在HTTP请求头中使用timeout字段来指定事务的超时时长。

创建事务时，您还可以在HTTP请求头中使用idle_transaction_timeout字段来指定事务可以保持空闲状态的超时时长。如果在该超时时长内未写入任何数据，事务将自动回滚。

## 好处

Stream Load事务接口带来以下优势：

- **精确一次语义**

  事务分为预提交和提交两个阶段，便于跨系统加载数据。例如，该接口可以确保从Flink加载的数据精确一次语义。

- **提升加载性能**

  如果您通过程序运行加载作业，Stream Load事务接口允许您根据需要合并多个小批量数据，然后通过调用/api/transaction/commit操作在单个事务中一次性发送所有数据。如此一来，减少了需要加载的数据版本数量，提升了加载性能。

## 限制

Stream Load事务接口有以下限制：

- 仅支持**单数据库单表**事务。支持**多数据库多表**事务正在开发中。

- 仅支持**单个客户端**的并发数据写入。支持**多个客户端**并发数据写入的功能正在开发中。

- /api/transaction/load操作可以在一个事务中多次调用。在这种情况下，所有调用的/api/transaction/load操作必须具有相同的参数设置。

- 使用Stream Load事务接口加载CSV格式数据时，请确保您的数据文件中的每条数据记录都以行分隔符结束。

## 注意事项

- 如果您调用的/api/transaction/begin、/api/transaction/load或/api/transaction/prepare操作返回错误，则事务失败并将自动回滚。
- 调用/api/transaction/begin操作以启动新事务时，您可以选择指定一个标签。如果不指定标签，StarRocks将为事务生成一个标签。请注意，后续的/api/transaction/load、/api/transaction/prepare和/api/transaction/commit操作必须使用与/api/transaction/begin操作相同的标签。
- 如果您使用先前事务的标签来调用/api/transaction/begin操作以启动新事务，先前的事务将失败并被回滚。
- StarRocks支持的CSV格式数据默认的列分隔符和行分隔符分别是\t和\n。如果您的数据文件未使用默认的列分隔符或行分隔符，当调用/api/transaction/load操作时，您必须使用"column_separator: <column_separator>"或"row_delimiter: <row_delimiter>"来指定数据文件中实际使用的列分隔符或行分隔符。

## 基本操作

### 准备样本数据

本主题以CSV格式数据为例。

1. 在您的本地文件系统的/home/disk1/路径下创建一个名为example1.csv的CSV文件。文件包含三列，分别代表用户ID、用户名和用户分数。

   ```Plain
   1,Lily,23
   2,Rose,23
   3,Alice,24
   4,Julia,25
   ```

2. 在您的StarRocks数据库test_db中，创建一个名为table1的主键表。表包含三列：id、name和score，其中id是主键。

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
> 在此示例中，`streamload_txn_example1_table1`被指定为事务的标签。

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

- 如果事务与重复标签绑定，将返回以下结果：

  ```Bash
  {
      "Status": "LABEL_ALREADY_EXISTS",
      "ExistingJobStatus": "RUNNING",
      "Message": "Label [streamload_txn_example1_table1] has already been used."
  }
  ```

- 如果出现其他错误，将返回以下结果：

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
> 调用`/api/transaction/load`操作时，您必须使用`\u003cfile_path\u003e`来指定要加载的数据文件的保存路径。

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
> 在此示例中，数据文件`example1.csv`中使用的列分隔符是逗号（`,`），而不是StarRocks默认的列分隔符（`\t`）。因此，在调用`/api/transaction/load`操作时，您必须使用`"column_separator: \\u003ccolumn_separator\\u003e"`来指定逗号（`,`）作为列分隔符。

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

- 如果事务被视为未知，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "TXN_NOT_EXISTS"
  }
  ```

- 如果事务被视为处于无效状态，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transcation State Invalid"
  }
  ```

- 如果出现其他错误，将返回以下结果：

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

- 如果事务被视为不存在，将返回以下结果：

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

- 如果出现其他错误，将返回以下结果：

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

- 如果事务已被提交，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": "Transaction already commited",
  }
  ```

- 如果事务被视为不存在，将返回以下结果：

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

- 如果出现其他错误，将返回以下结果：

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

- 如果回滚成功，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": ""
  }
  ```

- 如果事务被视为不存在，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transcation Not Exist"
  }
  ```

- 如果出现其他错误，将返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": ""
  }
  ```

## 参考资料

有关Stream Load的适用场景、支持的数据文件格式以及Stream Load工作原理的信息，请参见[通过Stream Load从本地文件系统加载](../loading/StreamLoad.md#loading-from-a-local-file-system-via-stream-load)。

有关创建Stream Load作业的语法和参数的信息，请参见[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。
