---
displayed_sidebar: English
---

# Stream Load トランザクションインターフェースを使用したデータのロード

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

v2.4以降、StarRocksはApache Flink®やApache Kafka®などの外部システムからデータをロードするトランザクションに対して2フェーズコミット（2PC）を実装するStream Loadトランザクションインターフェースを提供しています。Stream Loadトランザクションインターフェースは、高い同時実行性を持つストリームロードのパフォーマンスを向上させるのに役立ちます。

このトピックでは、Stream Loadトランザクションインターフェースについて、およびこのインターフェースを使用してStarRocksにデータをロードする方法について説明します。

<InsertPrivNote />

## 説明

Stream Loadトランザクションインターフェースは、HTTPプロトコル互換のツールや言語を使用してAPI操作を呼び出すことをサポートしています。このトピックでは、例としてcurlを使用してこのインターフェースの使用方法を説明します。このインターフェースは、トランザクション管理、データ書き込み、トランザクションの事前コミット、トランザクションの重複排除、トランザクションのタイムアウト管理など、さまざまな機能を提供します。

### トランザクション管理

Stream Loadトランザクションインターフェースは、トランザクションを管理するために使用される以下のAPI操作を提供します：

- `/api/transaction/begin`: 新しいトランザクションを開始します。

- `/api/transaction/commit`: 現在のトランザクションをコミットして、データの変更を永続化します。

- `/api/transaction/rollback`: 現在のトランザクションをロールバックして、データの変更を中止します。

### トランザクションの事前コミット

Stream Loadトランザクションインターフェースは`/api/transaction/prepare`操作を提供し、これは現在のトランザクションを事前コミットしてデータの変更を一時的に永続化するために使用されます。トランザクションを事前にコミットした後、トランザクションをコミットまたはロールバックに進むことができます。StarRocksクラスタがトランザクションが事前コミットされた後にダウンした場合でも、StarRocksクラスタが正常に復旧した後、トランザクションをコミットに進むことができます。

> **注記**
>
> トランザクションが事前にコミットされた後は、そのトランザクションを使用してデータの書き込みを続けないでください。トランザクションを使用してデータの書き込みを続けると、書き込みリクエストがエラーを返します。

### データ書き込み

Stream Loadトランザクションインターフェースは、`/api/transaction/load`操作を提供し、これはデータを書き込むために使用されます。この操作は、1つのトランザクション内で複数回呼び出すことができます。

### トランザクションの重複排除

Stream LoadトランザクションインターフェースはStarRocksのラベリングメカニズムを継承しています。各トランザクションに一意のラベルをバインドすることで、トランザクションの最大一度の保証を実現できます。

### トランザクションのタイムアウト管理

各FEの設定ファイルにある`stream_load_default_timeout_second`パラメータを使用して、そのFEのデフォルトのトランザクションタイムアウト期間を指定できます。

トランザクションを作成する際に、HTTPリクエストヘッダの`timeout`フィールドを使用してトランザクションのタイムアウト期間を指定することができます。

また、トランザクションを作成する際に、HTTPリクエストヘッダの`idle_transaction_timeout`フィールドを使用して、トランザクションがアイドル状態でいられるタイムアウト期間を指定することもできます。タイムアウト期間内にデータが書き込まれない場合、トランザクションは自動的にロールバックされます。

## 利点

Stream Loadトランザクションインターフェースは以下の利点をもたらします：

- **厳密に一度だけのセマンティクス**

  トランザクションはプリコミットとコミットの2フェーズに分けられ、システム間でのデータロードを容易にします。例えば、このインターフェースはFlinkからのデータロードに対して厳密に一度だけのセマンティクスを保証することができます。

- **ロードパフォーマンスの向上**

  プログラムを使用してロードジョブを実行する場合、Stream Loadトランザクションインターフェースを使用することで、複数のミニバッチのデータを必要に応じてマージし、`/api/transaction/commit`操作を呼び出して一度に一つのトランザクション内で全て送信することができます。これにより、ロードする必要のあるデータバージョンが少なくなり、ロードパフォーマンスが向上します。

## 制限事項

Stream Loadトランザクションインターフェースには以下の制限があります：

- **単一データベース単一テーブル**のトランザクションのみがサポートされています。**複数データベース複数テーブル**のトランザクションのサポートは開発中です。

- **一つのクライアントからの同時データ書き込み**のみがサポートされています。**複数クライアントからの同時データ書き込み**のサポートは開発中です。

- `/api/transaction/load`操作は、1つのトランザクション内で複数回呼び出すことができます。この場合、呼び出されるすべての`/api/transaction/load`操作に指定されるパラメータ設定は同じでなければなりません。

- Stream Loadトランザクションインターフェースを使用してCSV形式のデータをロードする場合、データファイル内の各データレコードが行区切り文字で終わっていることを確認してください。

## 注意事項

- `/api/transaction/begin`、`/api/transaction/load`、または`/api/transaction/prepare`操作を呼び出した際にエラーが返された場合、トランザクションは失敗し、自動的にロールバックされます。
- 新しいトランザクションを開始するために`/api/transaction/begin`操作を呼び出す際には、ラベルを指定するオプションがあります。ラベルを指定しない場合、StarRocksはトランザクションに対してラベルを生成します。ただし、`/api/transaction/begin`操作で使用したラベルと同じラベルを、後続の`/api/transaction/load`、`/api/transaction/prepare`、および`/api/transaction/commit`操作で使用する必要があります。
- 以前のトランザクションのラベルを使用して`/api/transaction/begin`操作を呼び出し新しいトランザクションを開始した場合、以前のトランザクションは失敗し、ロールバックされます。
- StarRocksがCSV形式のデータに対してサポートするデフォルトの列区切り文字と行区切り文字は`\t`と`\n`です。データファイルでデフォルトの列区切り文字や行区切り文字を使用していない場合は、`/api/transaction/load`操作を呼び出す際に`"column_separator: <column_separator>"`や`"row_delimiter: <row_delimiter>"`を使用して、実際にデータファイルで使用されている列区切り文字や行区切り文字を指定する必要があります。

## 基本操作

### サンプルデータの準備

このトピックでは、CSV形式のデータを例として使用します。

1. ローカルファイルシステムの`/home/disk1/`パスに`example1.csv`という名前のCSVファイルを作成します。このファイルは、ユーザーID、ユーザー名、ユーザースコアを順に表す3つの列で構成されています。

   ```Plain
   1,Lily,23
   2,Rose,23
   3,Alice,24
   4,Julia,25
   ```

2. StarRocksデータベース`test_db`に`table1`という名前のプライマリキーテーブルを作成します。このテーブルは、`id`、`name`、`score`の3つの列で構成され、`id`がプライマリキーです。

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

```
curl --location-trusted -u <jack>:<123456> -H "label:streamload_txn_example1_table1" \
    -H "Expect:100-continue" \
    -H "db:test_db" -H "table:table1" \
    -XPOST http://<fe_host>:<fe_http_port>/api/transaction/begin
```

> **注記**
>
> この例では、`streamload_txn_example1_table1`がトランザクションのラベルとして指定されています。

#### 戻り値

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

- トランザクションが重複したラベルに紐づけられている場合、以下の結果が返されます：

  ```Bash
  {
      "Status": "LABEL_ALREADY_EXISTS",
      "ExistingJobStatus": "RUNNING",
      "Message": "Label [streamload_txn_example1_table1] is already in use."
  }
  ```

- 重複したラベル以外のエラーが発生した場合、以下の結果が返されます：

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

> **注記**
>
> `/api/transaction/load`エンドポイントを呼び出す際には、ロードしたいデータファイルの保存パスを`<file_path>`で指定する必要があります。

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
> この例では、データファイル`example1.csv`で使用されているカラムセパレータは、StarRocksのデフォルトのカラムセパレータ`\t`ではなくコンマ`,`です。したがって、`/api/transaction/load`エンドポイントを呼び出す際には、`"column_separator: <column_separator>"`を使用してカラムセパレータとしてコンマ`,`を指定する必要があります。

#### 戻り値

- データの書き込みが成功した場合、以下の結果が返されます：

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

- トランザクションが不明と判断された場合、以下の結果が返されます：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "TXN_NOT_EXISTS"
  }
  ```

- トランザクションが無効な状態と判断された場合、以下の結果が返されます：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transaction State Invalid"
  }
  ```

- 不明なトランザクションや無効な状態以外のエラーが発生した場合、以下の結果が返されます：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": ""
  }
  ```

### トランザクションのプリコミット

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

- プリコミットが成功した場合、以下の結果が返されます：

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

- トランザクションが存在しないと判断された場合、以下の結果が返されます：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transaction Not Exist"
  }
  ```

- プリコミットがタイムアウトした場合、以下の結果が返されます：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "commit timeout",
  }
  ```

- 存在しないトランザクションやプリコミットのタイムアウト以外のエラーが発生した場合、以下の結果が返されます：

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

- コミットが成功した場合、以下の結果が返されます：

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

- トランザクションが既にコミットされていると判断された場合、以下の結果が返されます：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": "Transaction already committed",
  }
  ```

- トランザクションが存在しないと判断された場合、以下の結果が返されます：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transaction Not Exist"
  }
  ```

- コミットがタイムアウトした場合、以下の結果が返されます：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "commit timeout",
  }
  ```

- データの公開がタイムアウトした場合、以下の結果が返されます：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "publish timeout",
      "CommitAndPublishTimeMs": 1393
  }
  ```

- 存在しないトランザクションやタイムアウト以外のエラーが発生した場合、以下の結果が返されます：

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

- ロールバックが成功した場合、以下の結果が返されます：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "OK",
      "Message": ""
  }
  ```

- トランザクションが存在しないと判断された場合、以下の結果が返されます：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transaction Not Exist"
  }
  ```

- 存在しないトランザクション以外のエラーが発生した場合、以下の結果が返されます：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": ""
  }
  ```

## 参照


Stream Loadの適切なアプリケーションシナリオとサポートされているデータファイル形式、およびStream Loadの動作についての情報は、[ローカルファイルシステム経由でのStream Loadによるローディング](../loading/StreamLoad.md#loading-from-a-local-file-system-via-stream-load)を参照してください。

Stream Loadジョブを作成するための構文とパラメータに関する情報については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。
