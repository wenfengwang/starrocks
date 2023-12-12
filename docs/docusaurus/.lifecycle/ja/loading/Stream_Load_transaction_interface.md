---
displayed_sidebar: "Japanese"
---

# Stream Loadトランザクションインタフェースを使用してデータをロードする

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

v2.4以降、StarRocksは、Stream Loadトランザクションインタフェースを提供し、Apache Flink®やApache Kafka®などの外部システムからデータをロードするためのトランザクションの実装に2フェーズコミット（2PC）を行います。Stream Loadトランザクションインタフェースは、高度な同時ストリームロードのパフォーマンスを向上させるのに役立ちます。

このトピックでは、Stream Loadトランザクションインタフェースと、このインタフェースを使用してStarRocksにデータをロードする方法について説明します。

<InsertPrivNote />

## 説明

Stream Loadトランザクションインタフェースは、HTTPプロトコル互換のツールや言語を使用してAPI操作を呼び出すことをサポートします。このトピックでは、このインタフェースの使用方法を説明するために、curlを例にとって説明します。このインタフェースは、トランザクション管理、データ書き込み、トランザクションの事前コミット、トランザクションの重複排除、トランザクションのタイムアウト管理などのさまざまな機能を提供します。

### トランザクション管理

Stream Loadトランザクションインタフェースでは、次のAPI操作が提供され、これらを使用してトランザクションを管理します。

- `/api/transaction/begin`：新しいトランザクションを開始します。

- `/api/transaction/commit`：現在のトランザクションをコミットしてデータ変更を永続化します。

- `/api/transaction/rollback`：現在のトランザクションをロールバックしてデータ変更を中止します。

### トランザクションの事前コミット

Stream Loadトランザクションインタフェースでは、`/api/transaction/prepare`操作が提供され、これを使用して現在のトランザクションを事前コミットし、データ変更を一時的に永続化します。トランザクションを事前コミットした後、トランザクションを続行するか、ロールバックすることができます。StarRocksクラスターがトランザクションを事前コミットした後に障害が発生した場合、StarRocksクラスターが正常に復元された後、トランザクションをコミットすることができます。

> **注意**
>
> トランザクションが事前コミットされた後は、トランザクションを使用してデータの書き込みを続けないでください。トランザクションを使用してデータを書き続けると、書き込みリクエストがエラーを返します。

### データの書き込み

Stream Loadトランザクションインタフェースでは、`/api/transaction/load`操作が提供され、これを使用してデータを書き込むことができます。1つのトランザクション内で複数回この操作を呼び出すことができます。

### トランザクションの重複排除

Stream Loadトランザクションインタフェースは、StarRocksのラベリングメカニズムを引き継ぐことができます。各トランザクションにユニークなラベルをバインドし、トランザクションのために最大1回の保証を実現できます。

### トランザクションのタイムアウト管理

各FEの設定ファイルで`stream_load_default_timeout_second`パラメータを使用して、そのFEのデフォルトのトランザクションタイムアウト期間を指定できます。

トランザクションを作成する際、HTTPリクエストヘッダーの`timeout`フィールドを使用して、トランザクションのタイムアウト期間を指定できます。

トランザクションを作成する際、HTTPリクエストヘッダーの`idle_transaction_timeout`フィールドを使用して、トランザクションがアイドル状態で滞在できるタイムアウト期間を指定できます。タイムアウト期間内にデータが書き込まれない場合、トランザクションは自動的にロールバックされます。

## ベネフィット

Stream Loadトランザクションインタフェースは、以下のベネフィットをもたらします。

- **正確に1回だけのセマンティクス**

  トランザクションは事前コミットとコミットの2つのフェーズに分かれており、システム全体でデータを簡単にロードすることができます。たとえば、このインタフェースを使用すると、Flinkからのデータロードに対して正確に1回だけのセマンティクスを保証できます。

- **ロードのパフォーマンス向上**

  プログラムを使用してロードジョブを実行する場合、Stream Loadトランザクションインタフェースを使用すると、複数のミニバッチのデータを必要に応じてマージし、`/api/transaction/commit`操作を呼び出して1つのトランザクション内でまとめて送信することができます。そのため、ロードする必要があるデータバージョンが少なくなり、ロードのパフォーマンスが向上します。

## 制限

Stream Loadトランザクションインタフェースには以下の制限があります。

- **単一データベース単一テーブル**トランザクションのみがサポートされています。**複数データベース複数テーブル**トランザクションのサポートが開発中です。
- **1つのクライアントからの同時データ書き込み**のみがサポートされています。**複数クライアントからの同時データ書き込み**のサポートが開発中です。
- `/api/transaction/load`操作は1つのトランザクション内で複数回呼び出すことができます。この場合、呼び出されたすべての`/api/transaction/load`操作に指定されたパラメータ設定は同じである必要があります。
- Stream Loadトランザクションインタフェースを使用してCSV形式のデータをロードする場合、データファイル内の各データレコードが行区切り記号で終了することを確認してください。

## 注意事項

- 呼び出した`/api/transaction/begin`、`/api/transaction/load`、または`/api/transaction/prepare`操作がエラーを返した場合、トランザクションは失敗し、自動的にロールバックされます。
- 新しいトランザクションを開始するために`/api/transaction/begin`操作を呼び出す際に、ラベルを指定するオプションがあります。ラベルを指定しない場合、StarRocksはトランザクションのためにラベルを生成します。なお、その後の`/api/transaction/load`、`/api/transaction/prepare`、`/api/transaction/commit`操作は、`/api/transaction/begin`操作と同じラベルを使用する必要があります。
- 前のトランザクションのラベルを使用して新しいトランザクションを開始するために`/api/transaction/begin`操作を呼び出すと、前のトランザクションは失敗し、ロールバックされます。
- StarRocksがCSV形式のデータをサポートするデフォルトの列区切り記号と行区切り記号は`\t`および`\n`です。データファイルがデフォルトの列区切り記号や行区切り記号を使用していない場合は、`/api/transaction/load`操作を呼び出す際に、データファイルで実際に使用されている列区切り記号や行区切り記号を指定するために`"column_separator: <column_separator>"`または`"row_delimiter: <row_delimiter>"`を使用する必要があります。

## 基本操作

### サンプルデータの準備

このトピックでは、CSV形式のデータを例として使用します。

1. ローカルファイルシステムの`/home/disk1/`パスに、`example1.csv`という名前のCSVファイルを作成します。このファイルは、ユーザーID、ユーザー名、およびユーザースコアを順に表す3つの列から構成されています。

   ```Plain
   1,Lily,23
   2,Rose,23
   3,Alice,24
   4,Julia,25
   ```

2. StarRocksデータベース`test_db`で、`table1`というプライマリキー付きテーブルを作成します。このテーブルは、`id`、`name`、`score`の3つの列で構成されており、`id`がプライマリキーです。

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

### トランザクションの開始

#### 構文

```Bash
curl --location-trusted -u <username>:<password> -H "label:<label_name>" \
    -H "Expect:100-continue" \
    -H "db:<database_name>" -H "table:<table_name>" \
    -XPOST http://<fe_host>:<fe_http_port>/api/transaction/begin
```

#### 例

```Bash
curl --location-trusted -u <jack>:<123456> -H "label:streamload_txn_example1_table1" \
    -H "Expect:100-continue" \
    -H "db:test_db" -H "table:table1" \
    -XPOST http://<fe_host>:<fe_http_port>/api/transaction/begin
```

> **注意**
>
> この例では、トランザクションのラベルとして`streamload_txn_example1_table1`が指定されています。

#### 戻り結果

- トランザクションが正常に開始された場合、以下の結果が返されます：

  ```Bash
  {
      "Status": "OK",
      "Message": "",
      "Label": "streamload_txn_example1_table1",
      "TxnId": 9032,
      "BeginTxnTimeMs": 0
  }
  ```

- トランザクションが重複するラベルにバインドされている場合、以下の結果が返されます：

  ```Bash
  {
      "Status": "LABEL_ALREADY_EXISTS",
      "ExistingJobStatus": "RUNNING",
      "Message": "Label [streamload_txn_example1_table1] has already been used."
  }
  ```

- 重複ラベル以外のエラーが発生した場合、以下の結果が返されます：

  ```Bash
  {
      "Status": "FAILED",
      "Message": ""
  }
  ```

### データの書き込み

#### 構文

```Bash
curl --location-trusted -u <username>:<password> -H "label:<label_name>" \
    -H "Expect:100-continue" \
    -H "db:<database_name>" -H "table:<table_name>" \
    -T <file_path> \
    -XPUT http://<fe_host>:<fe_http_port>/api/transaction/load
```

> **注意**
>
> `/api/transaction/load`操作を呼び出す際に、`<file_path>`を使用してロードしたいデータファイルの保存パスを指定する必要があります。

#### 例

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
> この例では、データファイル `example1.csv` で使用されるカラムの区切り文字は、StarRocksのデフォルトのカラム区切り文字 (`\t`) の代わりにカンマ (`,`) が使用されます。そのため、`/api/transaction/load` 操作を呼び出す際には、カラムの区切り文字としてカンマ (`,`) を指定するために `"column_separator: <column_separator>"` を使用する必要があります。

#### 戻り値

- データの書き込みが成功した場合は、以下の結果が返されます:

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

- トランザクションが不明と判断された場合は、以下の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "TXN_NOT_EXISTS"
  }
  ```

- トランザクションが無効な状態と見なされた場合は、以下の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transcation State Invalid"
  }
  ```

- 不明なトランザクションと無効な状態以外のエラーが発生した場合は、以下の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": ""
  }
  ```

### トランザクションの事前コミット

#### 構文

```Bash
curl --location-trusted -u <username>:<password> -H "label:<label_name>" \
    -H "Expect:100-continue" \
    -H "db:<database_name>" \
    -XPOST http://<fe_host>:<fe_http_port>/api/transaction/prepare
```

#### 例

```Bash
curl --location-trusted -u <jack>:<123456> -H "label:streamload_txn_example1_table1" \
    -H "Expect:100-continue" \
    -H "db:test_db" \
    -XPOST http://<fe_host>:<fe_http_port>/api/transaction/prepare
```

#### 戻り値

- 事前コミットが成功した場合は、以下の結果が返されます:

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

- トランザクションが存在しないと見なされた場合は、以下の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transcation Not Exist"
  }
  ```

- 事前コミットがタイムアウトした場合は、以下の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "commit timeout",
  }
  ```

- 存在しないトランザクションと事前コミットのタイムアウト以外のエラーが発生した場合は、以下の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "publish timeout"
  }
  ```

### トランザクションのコミット

#### 構文

```Bash
curl --location-trusted -u <username>:<password> -H "label:<label_name>" \
    -H "Expect:100-continue" \
    -H "db:<database_name>" \
    -XPOST http://<fe_host>:<fe_http_port>/api/transaction/commit
```

#### 例

```Bash
curl --location-trusted -u <jack>:<123456> -H "label:streamload_txn_example1_table1" \
    -H "Expect:100-continue" \
    -H "db:test_db" \
    -XPOST http://<fe_host>:<fe_http_port>/api/transaction/commit
```

#### 戻り値

- コミットが成功した場合は、以下の結果が返されます:

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

- トランザクションが既にコミットされている場合は、以下の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": "Transaction already commited",
  }
  ```

- トランザクションが存在しないと見なされた場合は、以下の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transcation Not Exist"
  }
  ```

- コミットがタイムアウトした場合は、以下の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "commit timeout",
  }
  ```

- データの公開がタイムアウトした場合は、以下の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "publish timeout",
      "CommitAndPublishTimeMs": 1393
  }
  ```

- 存在しないトランザクションおよびタイムアウト以外のエラーが発生した場合は、以下の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": ""
  }
  ```

### トランザクションのロールバック

#### 構文

```Bash
curl --location-trusted -u <username>:<password> -H "label:<label_name>" \
    -H "Expect:100-continue" \
    -H "db:<database_name>" \
    -XPOST http://<fe_host>:<fe_http_port>/api/transaction/rollback
```

#### 例

```Bash
curl --location-trusted -u <jack>:<123456> -H "label:streamload_txn_example1_table1" \
    -H "Expect:100-continue" \
    -H "db:test_db" \
    -XPOST http://<fe_host>:<fe_http_port>/api/transaction/rollback
```

#### 戻り値

- ロールバックが成功した場合は、以下の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": ""
  }
  ```

- トランザクションが存在しないと見なされた場合は、以下の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transcation Not Exist"
  }
  ```

- 存在しないトランザクション以外のエラーが発生した場合は、以下の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": ""
  }
  ```

## 参照
```
For information about the suitable application scenarios and supported data file formats of Stream Load and about how Stream Load works, see [Loading from a local file system via Stream Load](../loading/StreamLoad.md#loading-from-a-local-file-system-via-stream-load).

For information about the syntax and parameters for creating Stream Load jobs, see [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md).
```