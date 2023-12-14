# 通过流式加载事务接口加载数据

从v2.4开始，StarRocks提供了流式加载事务接口，用于实现两阶段提交（2PC）的事务，用于从外部系统（例如Apache Flink®和Apache Kafka®）加载数据。流式加载事务接口有助于提高高并发流式加载的性能。

本主题描述了流式加载事务接口以及如何通过使用此接口将数据加载到StarRocks中。

<InsertPrivNote />

## 描述

流式加载事务接口支持使用与HTTP协议兼容的工具或语言调用API操作。本主题以curl为例，解释了如何使用此接口。此接口提供了各种功能，例如事务管理、数据写入、事务预提交、事务重复消除和事务超时管理。

### 事务管理

流式加载事务接口提供以下API操作，用于管理事务：

- `/api/transaction/begin`：启动一个新事务。

- `/api/transaction/commit`：提交当前事务以使数据更改持久化。

- `/api/transaction/rollback`：回滚当前事务以中止数据更改。

### 事务预提交

流式加载事务接口提供了`/api/transaction/prepare`操作，用于预提交当前事务并使数据更改临时持久化。预提交事务后，您可以继续提交或回滚事务。如果在预提交事务后，StarRocks集群发生故障，则可以在StarRocks集群恢复到正常状态后继续提交事务。

> **注意**

>

> 事务预提交后，请不要继续使用该事务写入数据。如果继续使用该事务写入数据，则您的写入请求将返回错误。

### 数据写入

流式加载事务接口提供了`/api/transaction/load`操作，用于写入数据。可以在一个事务中多次调用此操作。

### 事务重复消除

流式加载事务接口继承了StarRocks的标记机制。您可以将唯一标签绑定到每个事务，以实现事务的最多一次保证。

### 事务超时管理

可以使用每个FE配置文件中的`stream_load_default_timeout_second`参数来指定该FE的默认事务超时期限。

创建事务时，可以使用HTTP请求头中的`timeout`字段来指定事务的超时期限。

创建事务时，还可以使用HTTP请求头中的`idle_transaction_timeout`字段来指定事务可以保持空闲的超时期限。如果在超时期限内没有写入数据，则事务会自动回滚。

## 好处

流式加载事务接口带来以下好处：

- **确保一次语义**

  事务分为预提交和提交两个阶段，可以轻松地在系统之间加载数据。例如，此接口可以保证从Flink加载的数据具有确保一次的语义。

- **提高加载性能**

  如果使用程序运行加载作业，则流式加载事务接口允许您按需合并多个小批次的数据，然后通过调用`/api/transaction/commit`操作一次性发送所有数据版本。因此，需要加载的数据版本较少，并且加载性能得到改善。

## 限制

流式加载事务接口具有以下限制：

- 仅支持**单数据库单表**事务。**多数据库多表**事务的支持正在开发中。

- 仅支持**来自单个客户端的并发数据写入**。**来自多个客户端的并发数据写入**的支持正在开发中。

- 可以在一个事务中多次调用`/api/transaction/load`操作。在这种情况下，调用所有`/api/transaction/load`操作的参数设置必须相同。

- 通过流式加载事务接口加载CSV格式数据时，请确保数据文件中的每条数据记录以行分隔符结束。

## 注意事项

- 如果您调用了`/api/transaction/begin`、`/api/transaction/load`或`/api/transaction/prepare`操作返回错误，则事务将失败并自动回滚。

- 调用`/api/transaction/begin`操作启动新事务时，您可以选择指定标签。如果不指定标签，StarRocks将为事务生成标签。请注意，随后的`/api/transaction/load`、`/api/transaction/prepare`和`/api/transaction/commit`操作必须使用与`/api/transaction/begin`操作相同的标签。

- 如果您使用先前事务的标签调用`/api/transaction/begin`操作来启动新事务，则将使先前的事务失败并回滚。
- StarRocks支持CSV格式数据的默认列分隔符和行分隔符分别为`\t`和`\n`。如果您的数据文件未使用默认列分隔符或行分隔符，则在调用`/api/transaction/load`操作时，必须使用`"column_separator: <column_separator>"`或`"row_delimiter: <row_delimiter>"`来指定数据文件中实际使用的列分隔符或行分隔符。

## 基本操作

### 准备示例数据

本主题以CSV格式数据为例。

1. 在本地文件系统的`/home/disk1/`路径下，创建名为`example1.csv`的CSV文件。该文件由三列组成，分别表示用户ID、用户名和用户分数。

   ```Plain

   1,Lily,23
   2,Rose,23
   3,Alice,24
   4,Julia,25
   ```


2. 在StarRocks数据库`test_db`中，创建名为`table1`的主键表。该表包含三列：`id`、`name`和`score`，其中`id`为主键。

   ```SQL
   CREATE TABLE `table1`
   (
       `id` int(11) NOT NULL COMMENT "用户ID",
       `name` varchar(65533) NULL COMMENT "用户名称",
       `score` int(11) NOT NULL COMMENT "用户分数"
   )
   ENGINE=OLAP
   PRIMARY KEY(`id`)
   DISTRIBUTED BY HASH(`id`) BUCKETS 10;
   ```

### 启动事务

#### 语法

```Bash
curl --location-trusted -u <用户名>:<密码> -H "label:<标签名称>" \
    -H "Expect:100-continue" \
    -H "db:<数据库名称>" -H "table:<表名>" \
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
>
> 对于此示例，`streamload_txn_example1_table1`被指定为事务的标签。

#### 返回结果

- 如果成功启动事务，则返回以下结果：

  ```Bash
  {
      "Status": "OK",
      "Message": "",
      "Label": "streamload_txn_example1_table1",
      "TxnId": 9032,
      "BeginTxnTimeMs": 0
  }
  ```

- 如果事务绑定了重复标签，则返回以下结果：

  ```Bash
  {
      "Status": "LABEL_ALREADY_EXISTS",
      "ExistingJobStatus": "RUNNING",
      "Message": "Label [streamload_txn_example1_table1] has already been used."
  }
  ```

- 如果发生重复标签以外的错误，则返回以下结果：

  ```Bash
  {
      "Status": "FAILED",
      "Message": ""
  }
  ```

### 写入数据

#### 语法

```Bash
curl --location-trusted -u <用户名>:<密码> -H "label:<标签名称>" \
    -H "Expect:100-continue" \
    -H "db:<数据库名称>" -H "table:<表名>" \
    -T <文件路径> \
    -XPUT http://<fe主机>:<fe_http端口>/api/transaction/load
```

> **注意**
>
> 在调用`/api/transaction/load`操作时，必须使用`<文件路径>`指定要加载的数据文件的保存路径。

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
>
> 对于本示例，数据文件 `example1.csv` 中使用的列分隔符是逗号（`,`），而不是 StarRocks 的默认列分隔符（`\t`）。因此，在调用 `/api/transaction/load` 操作时，您必须使用 `"column_separator: <column_separator>"` 来指定逗号（`,`）作为列分隔符。

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
      "ReceivedDataTimeMs": 38964,
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
      "Message": "事务状态无效"
  }
  ```

- 如果发生除了未知事务和无效状态之外的其他错误，则返回以下结果：

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
      "WriteDataTimeMs": 417851
      "CommitAndPublishTimeMs": 1393
  }
  ```

- 如果事务被视为不存在，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "事务不存在"
  }
  ```

- 如果预提交超时，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "预提交超时",
  }
  ```

- 如果发生除了不存在事务和预提交超时之外的其他错误，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "发布超时"
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
      "WriteDataTimeMs": 417851
      "CommitAndPublishTimeMs": 1393
  }
  ```

- 如果事务已经提交，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": "事务已经提交",
  }
  ```

- 如果事务被视为不存在，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "事务不存在"
  }
  ```

- 如果提交超时，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "提交超时",
  }
  ```

- 如果数据发布超时，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "发布超时",
      "CommitAndPublishTimeMs": 1393
  }
  ```

- 如果发生除了不存在事务和超时之外的其他错误，则返回以下结果：

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
      "Message": "事务不存在"
  }
  ```

- 如果发生除了不存在事务之外的其他错误，则返回以下结果：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": ""
  }
  ```

## 参考资料

```markdown
      + For information about the suitable application scenarios and supported data file formats of Stream Load and about how Stream Load works, see [Loading from a local file system via Stream Load](../loading/StreamLoad.md#loading-from-a-local-file-system-via-stream-load).
      + For information about the syntax and parameters for creating Stream Load jobs, see [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md).
    + {R}
  + {R}
+ {R}
```
```markdown
      + 关于Stream Load的适用应用场景和支持的数据文件格式以及Stream Load的工作原理的信息，请参见[通过Stream Load从本地文件系统加载](../loading/StreamLoad.md#loading-from-a-local-file-system-via-stream-load)。
      + 有关创建Stream Load作业的语法和参数的信息，请参见[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)。
    + {T}
  + {T}
+ {T}
```