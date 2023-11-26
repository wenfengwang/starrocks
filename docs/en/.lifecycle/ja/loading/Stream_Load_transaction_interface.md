---
displayed_sidebar: "Japanese"
---

# Stream Loadトランザクションインターフェースを使用してデータをロードする

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

v2.4以降、StarRocksは、Apache Flink®やApache Kafka®などの外部システムからデータをロードするために実行されるトランザクションに対して、2フェーズコミット（2PC）を実装するためのStream Loadトランザクションインターフェースを提供しています。Stream Loadトランザクションインターフェースは、高並行ストリームロードのパフォーマンスを向上させるのに役立ちます。

このトピックでは、Stream Loadトランザクションインターフェースとこのインターフェースを使用してStarRocksにデータをロードする方法について説明します。

<InsertPrivNote />

## 説明

Stream Loadトランザクションインターフェースは、HTTPプロトコル互換のツールや言語を使用してAPI操作を呼び出すことをサポートしています。このトピックでは、このインターフェースの使用方法を説明するために、curlを例として使用します。このインターフェースは、トランザクション管理、データ書き込み、トランザクションの事前コミット、トランザクションの重複排除、トランザクションのタイムアウト管理など、さまざまな機能を提供しています。

### トランザクション管理

Stream Loadトランザクションインターフェースは、次のAPI操作を提供しています。これらの操作は、トランザクションを管理するために使用されます。

- `/api/transaction/begin`: 新しいトランザクションを開始します。

- `/api/transaction/commit`: 現在のトランザクションをコミットしてデータの変更を永続化します。

- `/api/transaction/rollback`: 現在のトランザクションをロールバックしてデータの変更を中止します。

### トランザクションの事前コミット

Stream Loadトランザクションインターフェースは、現在のトランザクションを事前コミットしてデータの変更を一時的に永続化するための`/api/transaction/prepare`操作を提供しています。トランザクションを事前コミットした後は、トランザクションをコミットまたはロールバックすることができます。StarRocksクラスタがトランザクションを事前コミットした後にダウンした場合でも、StarRocksクラスタが正常に復元された後にトランザクションをコミットすることができます。

> **注意**
>
> トランザクションが事前コミットされた後は、トランザクションを使用してデータを書き込むことは続けないでください。トランザクションを使用してデータを書き込み続けると、書き込みリクエストがエラーを返します。

### データの書き込み

Stream Loadトランザクションインターフェースは、データを書き込むための`/api/transaction/load`操作を提供しています。この操作を使用して、1つのトランザクション内で複数回呼び出すことができます。

### トランザクションの重複排除

Stream Loadトランザクションインターフェースは、StarRocksのラベリングメカニズムを引き継いでいます。各トランザクションに固有のラベルをバインドすることで、トランザクションのアトモストワンスガラントを実現することができます。

### トランザクションのタイムアウト管理

各FEの設定ファイルである`stream_load_default_timeout_second`パラメータを使用して、そのFEのデフォルトのトランザクションタイムアウト期間を指定することができます。

トランザクションを作成する際に、HTTPリクエストヘッダの`timeout`フィールドを使用してトランザクションのタイムアウト期間を指定することができます。

トランザクションを作成する際に、HTTPリクエストヘッダの`idle_transaction_timeout`フィールドを使用して、トランザクションがアイドル状態で滞在できるタイムアウト期間を指定することができます。タイムアウト期間内にデータが書き込まれない場合、トランザクションは自動的にロールバックされます。

## 利点

Stream Loadトランザクションインターフェースには、以下の利点があります。

- **正確に一度だけのセマンティクス**

  トランザクションは、事前コミットとコミットの2つのフェーズに分割されており、システム間でデータをロードすることが容易になっています。たとえば、このインターフェースは、Flinkからのデータロードに対して正確に一度だけのセマンティクスを保証することができます。

- **ロードパフォーマンスの向上**

  プログラムを使用してロードジョブを実行する場合、Stream Loadトランザクションインターフェースを使用すると、複数のミニバッチのデータを必要に応じてマージし、`/api/transaction/commit`操作を呼び出して1つのトランザクション内でまとめて送信することができます。そのため、ロードする必要があるデータバージョンが少なくなり、ロードパフォーマンスが向上します。

## 制限事項

Stream Loadトランザクションインターフェースには、以下の制限事項があります。

- **単一データベース単一テーブル**トランザクションのみがサポートされています。**複数データベース複数テーブル**トランザクションのサポートは開発中です。

- **1つのクライアントからの同時データ書き込みのみ**がサポートされています。**複数のクライアントからの同時データ書き込み**のサポートは開発中です。

- `/api/transaction/load`操作は、1つのトランザクション内で複数回呼び出すことができます。この場合、呼び出されるすべての`/api/transaction/load`操作のパラメータ設定は同じである必要があります。

- Stream Loadトランザクションインターフェースを使用してCSV形式のデータをロードする場合、データファイルの各データレコードが行区切り文字で終わることを確認してください。

## 注意事項

- `/api/transaction/begin`、`/api/transaction/load`、または`/api/transaction/prepare`操作がエラーを返した場合、トランザクションは失敗し、自動的にロールバックされます。
- 新しいトランザクションを開始するために`/api/transaction/begin`操作を呼び出す際に、ラベルを指定するオプションがあります。ラベルを指定しない場合、StarRocksはトランザクションのためにラベルを生成します。ただし、後続の`/api/transaction/load`、`/api/transaction/prepare`、および`/api/transaction/commit`操作は、`/api/transaction/begin`操作と同じラベルを使用する必要があります。
- 以前のトランザクションのラベルを使用して新しいトランザクションを開始するために`/api/transaction/begin`操作を呼び出すと、以前のトランザクションは失敗し、ロールバックされます。
- StarRocksは、CSV形式のデータに対してサポートしているデフォルトの列セパレータと行区切り文字は`\t`と`\n`です。データファイルがデフォルトの列セパレータや行区切り文字を使用していない場合は、`"column_separator: <column_separator>"`または`"row_delimiter: <row_delimiter>"`を使用して、実際にデータファイルで使用されている列セパレータや行区切り文字を指定する必要があります。

## 基本操作

### サンプルデータの準備

このトピックでは、CSV形式のデータを使用して説明します。

1. ローカルファイルシステムの`/home/disk1/`パスに、`example1.csv`という名前のCSVファイルを作成します。このファイルは、ユーザーID、ユーザー名、ユーザースコアを順に表す3つの列で構成されています。

   ```Plain
   1,Lily,23
   2,Rose,23
   3,Alice,24
   4,Julia,25
   ```

2. StarRocksの`test_db`データベースに、`table1`という名前のプライマリキーテーブルを作成します。このテーブルは、`id`、`name`、`score`の3つの列で構成されており、`id`がプライマリキーです。

   ```SQL
   CREATE TABLE `table1`
   (
       `id` int(11) NOT NULL COMMENT "ユーザーID",
       `name` varchar(65533) NULL COMMENT "ユーザー名",
       `score` int(11) NOT NULL COMMENT "ユーザースコア"
   )
   ENGINE=OLAP
   PRIMARY KEY(`id`)
   DISTRIBUTED BY HASH(`id`) BUCKETS 10;
   ```

### トランザクションを開始する

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

#### 戻り値

- トランザクションが正常に開始された場合、次の結果が返されます。

  ```Bash
  {
      "Status": "OK",
      "Message": "",
      "Label": "streamload_txn_example1_table1",
      "TxnId": 9032,
      "BeginTxnTimeMs": 0
  }
  ```

- トランザクションが重複するラベルにバインドされている場合、次の結果が返されます。

  ```Bash
  {
      "Status": "LABEL_ALREADY_EXISTS",
      "ExistingJobStatus": "RUNNING",
      "Message": "Label [streamload_txn_example1_table1] has already been used."
  }
  ```

- 重複するラベル以外のエラーが発生した場合、次の結果が返されます。

  ```Bash
  {
      "Status": "FAILED",
      "Message": ""
  }
  ```

### データを書き込む

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
> `/api/transaction/load`操作を呼び出す際には、`<file_path>`を使用してロードするデータファイルの保存パスを指定する必要があります。

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
> この例では、データファイル`example1.csv`で使用されている列セパレータはデフォルトの列セパレータ（`\t`）ではなく、カンマ（`,`）です。そのため、`/api/transaction/load`操作を呼び出す際には、`"column_separator: <column_separator>"`を使用してカンマ（`,`）を列セパレータとして指定する必要があります。

#### 戻り値

- データの書き込みが成功した場合、次の結果が返されます。

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

- トランザクションが不明と見なされる場合、次の結果が返されます。

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "TXN_NOT_EXISTS"
  }
  ```

- トランザクションが無効な状態と見なされる場合、次の結果が返されます。

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transcation State Invalid"
  }
  ```

- 不明なトランザクションと無効な状態以外のエラーが発生した場合、次の結果が返されます。

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": ""
  }
  ```

### トランザクションを事前コミットする

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

- 事前コミットが成功した場合、次の結果が返されます。

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

- トランザクションが存在しないと見なされる場合、次の結果が返されます。

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transcation Not Exist"
  }
  ```

- 事前コミットがタイムアウトした場合、次の結果が返されます。

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "commit timeout",
  }
  ```

- 不明なトランザクションとタイムアウト以外のエラーが発生した場合、次の結果が返されます。

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "publish timeout"
  }
  ```

### トランザクションをコミットする

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

- コミットが成功した場合、次の結果が返されます。

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

- トランザクションが既にコミットされている場合、次の結果が返されます。

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": "Transaction already commited",
  }
  ```

- トランザクションが存在しないと見なされる場合、次の結果が返されます。

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transcation Not Exist"
  }
  ```

- コミットがタイムアウトした場合、次の結果が返されます。

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "commit timeout",
  }
  ```

- データの公開がタイムアウトした場合、次の結果が返されます。

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "publish timeout",
      "CommitAndPublishTimeMs": 1393
  }
  ```

- 不明なトランザクションとタイムアウト以外のエラーが発生した場合、次の結果が返されます。

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": ""
  }
  ```

### トランザクションをロールバックする

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

- ロールバックが成功した場合、次の結果が返されます。

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": ""
  }
  ```

- トランザクションが存在しないと見なされる場合、次の結果が返されます。

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transcation Not Exist"
  }
  ```

- 存在しないトランザクション以外のエラーが発生した場合、次の結果が返されます。

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": ""
  }
  ```

## 参考情報

Stream Loadの適切なアプリケーションシナリオとサポートされているデータファイル形式、およびStream Loadの動作については、[Stream Loadを使用したローカルファイルシステムからのロード](../loading/StreamLoad.md#loading-from-a-local-file-system-via-stream-load)を参照してください。

Stream Loadジョブの作成に使用する構文とパラメータについては、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。
