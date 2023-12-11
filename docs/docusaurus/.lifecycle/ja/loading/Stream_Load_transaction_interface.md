---
displayed_sidebar: "Japanese"
---

# ストリームロードトランザクションインターフェースを使用したデータの読み込み

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

v2.4以降、StarRocksでは、Apache Flink® や Apache Kafka® などの外部システムからデータをロードするトランザクションを実装するためのストリームロードトランザクションインターフェースが提供されます。ストリームロードトランザクションインターフェースを使用すると、高度に同時動作するストリームロードのパフォーマンスが向上します。

本トピックでは、ストリームロードトランザクションインターフェースの説明と、このインターフェースを使用してStarRocksにデータをロードする方法について説明します。

<InsertPrivNote />

## 説明

ストリームロードトランザクションインターフェースは、HTTPプロトコル互換のツールや言語を使用してAPI操作を呼び出すことをサポートします。本トピックでは、このインターフェースの使用方法を説明する例として、curlを使用します。このインターフェースにはトランザクション管理、データ書き込み、トランザクション事前コミット、トランザクションの重複排除、トランザクションタイムアウト管理など、さまざまな機能が提供されます。

### トランザクション管理

ストリームロードトランザクションインターフェースは、トランザクションを管理するために使用される以下のAPI操作を提供します:

- `/api/transaction/begin`: 新しいトランザクションを開始します。

- `/api/transaction/commit`: 現在のトランザクションをコミットしてデータ変更を永続化します。

- `/api/transaction/rollback`: 現在のトランザクションをロールバックしてデータ変更を中止します。

### トランザクション事前コミット

ストリームロードトランザクションインターフェースは、 `/api/transaction/prepare` 操作を提供し、現在のトランザクションを事前にコミットしてデータ変更を一時的に永続化します。トランザクションが事前にコミットされた後は、トランザクションをコミットまたはロールバックすることができます。StarRocksクラスタがトランザクションを事前にコミットした後に停止した場合でも、StarRocksクラスタが正常に復旧した後にトランザクションをコミットすることができます。

> **注意**
>
> トランザクションが事前にコミットされた後は、トランザクションを使用してデータの書き込みを継続しないでください。トランザクションを使用してデータの書き込みを継続すると、書き込みリクエストがエラーを返します。

### データ書き込み

ストリームロードトランザクションインターフェースは、 `/api/transaction/load` 操作を提供し、データの書き込みに使用します。この操作は、1つのトランザクションで複数回呼び出すことができます。

### トランザクションの重複排除

ストリームロードトランザクションインターフェースには、StarRocksのラベリングメカニズムを引き継いでいます。各トランザクションに固有のラベルをバインドして、トランザクションの重複排除を実現できます。

### トランザクションタイムアウト管理

各FEの設定ファイルで `stream_load_default_timeout_second` パラメータを使用して、そのFEのデフォルトトランザクションタイムアウト期間を指定できます。

トランザクションを作成する際に、HTTPリクエストヘッダの `timeout` フィールドを使用して、トランザクションのタイムアウト期間を指定できます。

トランザクションを作成する際に、HTTPリクエストヘッダの `idle_transaction_timeout` フィールドを使用して、トランザクションがアイドル状態に留まることができるタイムアウト期間を指定できます。指定されたタイムアウト期間内にデータが書き込まれない場合、トランザクションは自動的にロールバックされます。

## メリット

ストリームロードトランザクションインターフェースには、次のメリットがあります:

- **正確に一度** の意味論

  トランザクションは、事前コミットとコミットの2つの段階に分割され、システム間でデータを簡単にロードできます。例えば、このインターフェースを使用すると、Flinkからデータをロードする際に正確に一度の意味論を保証することができます。

- **改善されたロードのパフォーマンス**

  プログラムを使用してロードジョブを実行する場合、ストリームロードトランザクションインターフェースを使用すると、複数のミニバッチのデータを必要に応じて結合し、`/api/transaction/commit` 操作を呼び出して1つのトランザクションで一度にすべて送信することができます。そのため、ロードする必要があるデータバージョンが少なくなり、ロードのパフォーマンスが向上します。

## 制限

ストリームロードトランザクションインターフェースには、以下の制限があります:

- **単一のデータベース単一のテーブル** トランザクションのみがサポートされています。**複数のデータベース複数のテーブル** トランザクションのサポートは開発中です。

- **1つのクライアントからの同時データ書き込みのみ** がサポートされています。**複数のクライアントからの同時データ書き込み** のサポートは開発中です。

- `/api/transaction/load` 操作は1つのトランザクション内で複数回呼び出すことができます。この場合、呼び出される `/api/transaction/load` 操作のすべてのパラメータ設定は同じでなければなりません。

- ストリームロードトランザクションインターフェースを使用してCSV形式のデータをロードする場合、データファイル内の各データレコードが行区切り記号で終了するようにしてください。

## 注意事項

- `/api/transaction/begin`、`/api/transaction/load`、または `/api/transaction/prepare` 操作を呼び出した場合にエラーが返されると、トランザクションは失敗し、自動的にロールバックされます。

- 新しいトランザクションを開始するために `/api/transaction/begin` 操作を呼び出す際、ラベルを指定することができます。ラベルを指定しない場合、StarRocksはトランザクションのためにラベルを生成します。なお、その後続する `/api/transaction/load`、`/api/transaction/prepare`、`/api/transaction/commit` 操作は、`/api/transaction/begin` 操作と同じラベルを使用しなければなりません。

- 以前のトランザクションのラベルを使用して新しいトランザクションの開始に `/api/transaction/begin` 操作を呼び出した場合、前のトランザクションは失敗し、ロールバックされます。

- StarRocksがCSV形式のデータでサポートしているデフォルトの列区切り記号と行区切り記号は、`\t` と `\n` です。データファイルがデフォルトの列区切り記号または行区切り記号を使用していない場合は、`"/api/transaction/load"` 操作を呼び出す際に、データファイルで実際に使用されている列区切り記号または行区切り記号を指定するために、`"column_separator: <column_separator>"` または `"row_delimiter: <row_delimiter>"` を使用する必要があります。

## 基本操作

### サンプルデータの準備

本トピックでは、CSV形式のデータを使用します。

1. ローカルファイルシステムの `/home/disk1/` パスに、 `example1.csv` という名前のCSVファイルを作成します。ファイルは、ユーザーID、ユーザー名、およびユーザースコアを順番に表す3つの列で構成されています。

   ```Plain
   1,Lily,23
   2,Rose,23
   3,Alice,24
   4,Julia,25
   ```

2. StarRocksデータベース `test_db` で、 `table1` という名前のプライマリキー表を作成します。この表は、 `id`、 `name`、および `score` の3つの列で構成されており、`id` がプライマリキーとなっています。

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
> この例では、トランザクションのラベルとして `streamload_txn_example1_table1` が指定されています。

#### 返却結果

- トランザクションが正常に開始された場合、次の結果が返されます:

  ```Bash
  {
      "Status": "OK",
      "Message": "",
      "Label": "streamload_txn_example1_table1",
      "TxnId": 9032,
      "BeginTxnTimeMs": 0
  }
  ```

- トランザクションが重複するラベルにバインドされている場合、次の結果が返されます:

  ```Bash
  {
      "Status": "LABEL_ALREADY_EXISTS",
      "ExistingJobStatus": "RUNNING",
      "Message": "Label [streamload_txn_example1_table1] has already been used."
  }
  ```

- 重複するラベル以外のエラーが発生した場合、次の結果が返されます:

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
> `/api/transaction/load` 操作を呼び出す際は、 `<file_path>` を使用してロードしたいデータファイルの保存パスを指定する必要があります。

#### 例

```Bash
curl --location-trusted -u <jack>:<123456> -H "label:streamload_txn_example1_table1" \
    -H "Expect:100-continue" \
    -H "db:test_db" -H "table:table1" \
    -T /home/disk1/example1.csv \
    -H "column_separator: ," \
    -XPUT http://<fe_host>:<fe_http_port>/api/transaction/load
```
> **注記**
>
> この例では、データファイル`example1.csv`で使用される列区切り記号は、StarRocksのデフォルトの列区切り記号（`\t`）ではなく、コンマ（`,`）です。したがって、`/api/transaction/load`操作を呼び出す際には、「`column_separator: <column_separator>`」を使用して、列区切り記号としてコンマ（`,`）を指定する必要があります。

#### 戻り結果

- データ書き込みが成功した場合、次の結果が返されます:

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

- トランザクションが不明と見なされた場合、次の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "TXN_NOT_EXISTS"
  }
  ```

- トランザクションが無効な状態と見なされた場合、次の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transcation State Invalid"
  }
  ```

- 不明なトランザクションや無効な状態以外のエラーが発生した場合、次の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": ""
  }
  ```

### トランザクションを事前にコミットする

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

#### 戻り結果

- 事前コミットが成功した場合、次の結果が返されます:

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

- トランザクションが存在しないと見なされた場合、次の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transcation Not Exist"
  }
  ```

- 事前コミットがタイムアウトした場合、次の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "commit timeout"
  }
  ```

- 存在しないトランザクションや事前コミットのタイムアウト以外のエラーが発生した場合、次の結果が返されます:

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

#### 戻り結果

- コミットが成功した場合、次の結果が返されます:

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

- トランザクションが既にコミットされている場合、次の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": "Transaction already commited"
  }
  ```

- トランザクションが存在しないと見なされた場合、次の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transcation Not Exist"
  }
  ```

- コミットがタイムアウトした場合、次の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "commit timeout"
  }
  ```

- データの公開がタイムアウトした場合、次の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "publish timeout",
      "CommitAndPublishTimeMs": 1393
  }
  ```

- 存在しないトランザクションやタイムアウト以外のエラーが発生した場合、次の結果が返されます:

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

#### 戻り結果

- ロールバックが成功した場合、次の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": ""
  }
  ```

- トランザクションが存在しないと見なされた場合、次の結果が返されます:

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transcation Not Exist"
  }
  ```

- 存在しないトランザクション以外のエラーが発生した場合、次の結果が返されます:

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
    + {R}
      + {R}
    + {R}
  + {R}
```

```
    + {T}
      + {T}
    + {T}
  + {T}
```