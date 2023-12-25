---
displayed_sidebar: Chinese
---

# Stream Load トランザクションインターフェースを使用したインポート

Apache Flink®、Apache Kafka® など他のシステムとのクロスシステム2フェーズコミットをサポートし、高並行性の Stream Load インポートシナリオでのパフォーマンスを向上させるために、StarRocks はバージョン2.4から Stream Load トランザクションインターフェースを提供しています。

この記事では、Stream Load トランザクションインターフェースと、そのインターフェースを使用してデータを StarRocks にインポートする方法について説明します。

> **注意**
>
> インポート操作には対象テーブルの INSERT 権限が必要です。ユーザーアカウントに INSERT 権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md) を参照してユーザーに権限を付与してください。

## インターフェース説明

Stream Load トランザクションインターフェースは、HTTP プロトコルと互換性のあるツールや言語を使用してインターフェースリクエストを開始することをサポートしています。この記事では、curl ツールを例にしてインターフェースの使用方法を説明します。このインターフェースは、トランザクション管理、データ書き込み、トランザクションの事前コミット、トランザクションの重複排除、タイムアウト管理などの機能を提供します。

### トランザクション管理

以下の標準インターフェースを提供し、トランザクションを管理します：

- `/api/transaction/begin`：新しいトランザクションを開始します。

- `/api/transaction/commit`：現在のトランザクションをコミットし、変更を永続化します。

- `/api/transaction/rollback`：現在のトランザクションをロールバックし、変更を取り消します。

### トランザクションの事前コミット

`/api/transaction/prepare` インターフェースを提供し、現在のトランザクションを事前にコミットして変更を一時的に永続化します。トランザクションを事前にコミットした後、トランザクションをコミットまたはロールバックを続けることができます。このメカニズムにより、トランザクションが事前にコミットされた後に StarRocks がダウンした場合でも、システムが復旧した後にコミットを続行できます。

> **説明**
>
> トランザクションを事前にコミットした後は、データの書き込みを続けないでください。書き込みを続けると、リクエストがエラーになります。

### データ書き込み

`/api/transaction/load` インターフェースを提供し、データを書き込むために使用します。同じトランザクション内でこのインターフェースを複数回呼び出してデータを書き込むことができます。

### トランザクションの重複排除

StarRocks の既存のラベルメカニズムを再利用し、ラベルをトランザクションにバインドすることで、"最大一回 (At-Most-Once)" のセマンティクスを実現します。

### タイムアウト管理

FE の設定にある `stream_load_default_timeout_second` パラメータを使用して、デフォルトのトランザクションタイムアウト時間を設定できます。

トランザクションを開始する際には、HTTP リクエストヘッダの `timeout` フィールドを使用して、現在のトランザクションのタイムアウト時間を指定できます。

また、トランザクションを開始する際には、HTTP リクエストヘッダの `idle_transaction_timeout` フィールドを使用して、アイドルトランザクションのタイムアウト時間を指定できます。トランザクションが `idle_transaction_timeout` で設定されたタイムアウト時間を超えてデータの書き込みがない場合、トランザクションは自動的にロールバックされます。

## インターフェースの利点

Stream Load トランザクションインターフェースには以下の利点があります：

- **Exactly-Once セマンティクス**

  「トランザクションの事前コミット」と「トランザクションのコミット」を通じて、システム間の2フェーズコミットを容易に実現できます。例えば、Flink で「正確に一度 (Exactly-Once)」のセマンティクスを実現するインポートと組み合わせることができます。

- **インポートパフォーマンスの向上**

  プログラムを通じて Stream Load ジョブをサブミットするシナリオでは、Stream Load トランザクションインターフェースを使用して、一つのインポートジョブ内で必要に応じて複数回の小バッチデータをマージしてから「トランザクションのコミット」を行うことができ、データインポートのバージョンを減らし、インポートパフォーマンスを向上させることができます。

## 使用上の制限

トランザクションインターフェースには現在以下の使用制限があります：

- **単一データベース単一テーブル**のトランザクションのみをサポートしており、将来的には**複数データベース複数テーブル**のトランザクションをサポートする予定です。

- **単一クライアントの並行データ書き込み**のみをサポートしており、将来的には**複数クライアントの並行データ書き込み**をサポートする予定です。

- 単一のトランザクション内で `/api/transaction/load` データ書き込みインターフェースを複数回呼び出してデータを書き込むことをサポートしていますが、すべての `/api/transaction/load` インターフェースのパラメータ設定が一貫している必要があります。

- CSV 形式のデータをインポートする場合は、各行のデータの末尾に行区切り文字が必要です。

## 注意事項

- Stream Load トランザクションインターフェースを使用してデータをインポートする際は、`/api/transaction/begin`、`/api/transaction/load`、`/api/transaction/prepare` インターフェースでエラーが発生した場合、トランザクションは失敗し自動的にロールバックされます。
- `/api/transaction/begin` インターフェースを呼び出してトランザクションを開始する際には、ラベル (Label) を指定するかどうかを選択できます。ラベルを指定しない場合、StarRocks は自動的にトランザクションにラベルを生成します。その後の `/api/transaction/load`、`/api/transaction/prepare`、`/api/transaction/commit` の3つのインターフェースでは、`/api/transaction/begin` インターフェースと同じラベルを使用する必要があります。
- 同じラベルで `/api/transaction/begin` インターフェースを繰り返し呼び出すと、以前に同じラベルで開始されたトランザクションが失敗し、ロールバックされます。
- StarRocks がサポートする CSV 形式のデータのデフォルトの列区切り文字は `\t`、行区切り文字は `\n` です。ソースデータファイルの列区切り文字と行区切り文字が `\t` と `\n` でない場合は、`/api/transaction/load` インターフェースを呼び出す際に `"column_separator: <column_separator>"` と `"row_delimiter: <row_delimiter>"` を使用して、列区切り文字と行区切り文字を指定する必要があります。

## 基本操作

### データサンプルの準備

ここでは CSV 形式のデータを例にします。

1. ローカルファイルシステムの `/home/disk1/` パスに `example1.csv` という CSV 形式のデータファイルを作成します。ファイルには3列が含まれており、それぞれユーザーID、ユーザー名、ユーザースコアを表しています。以下のようになります：

   ```Plain
   1,Lily,23
   2,Rose,23
   3,Alice,24
   4,Julia,25
   ```

2. `test_db` データベースに `table1` という名前のプライマリキーモデルのテーブルを作成します。テーブルには `id`、`name`、`score` の3列が含まれ、プライマリキーは `id` 列です。以下のようになります：

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

> **説明**
>
> 上記の例では、トランザクションのラベルを `streamload_txn_example1_table1` と指定しています。

#### 戻り値

- トランザクションが成功した場合、以下の結果が返されます：

  ```Bash
  {
      "Status": "OK",
      "Message": "",
      "Label": "streamload_txn_example1_table1",
      "TxnId": 9032,
      "BeginTxnTimeMs": 0
  }
  ```

- トランザクションのラベルが重複している場合、以下の結果が返されます：

  ```Bash
  {
      "Status": "LABEL_ALREADY_EXISTS",
      "ExistingJobStatus": "RUNNING",
      "Message": "Label [streamload_txn_example1_table1] has already been used."
  }
  ```

- ラベルの重複以外の他のエラーが発生した場合、以下の結果が返されます：

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

> **説明**
>
> `/api/transaction/load` エンドポイントを呼び出す際には、`-T <file_path>` を使用してデータファイルのパスを指定する必要があります。

#### 例

```Bash
curl --location-trusted -u <jack>:<123456> -H "label:streamload_txn_example1_table1" \
    -H "Expect:100-continue" \
    -H "db:test_db" -H "table:table1" \
    -T /home/disk1/example1.csv \
    -H "column_separator: ," \
    -XPUT http://<fe_host>:<fe_http_port>/api/transaction/load
```

> **説明**
>
> 上記の例では、データファイル `example1.csv` で使用されている列の区切り文字がコンマ (`,`) であり、StarRocks のデフォルトの列区切り文字 (`\t`) ではないため、`/api/transaction/load` エンドポイントを呼び出す際には `"column_separator: <column_separator>"` を使用して列の区切り文字をコンマ (`,`) に指定する必要があります。

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

- トランザクションの状態が無効である場合、以下の結果が返されます：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "Transaction State Invalid"
  }
  ```

- トランザクションが不明または状態が無効である以外のエラーが発生した場合、以下の結果が返されます：

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

- トランザクションのプリコミットが成功した場合、以下の結果が返されます：

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

- トランザクションのプリコミットがタイムアウトした場合、以下の結果が返されます：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": "commit timeout",
  }
  ```

- トランザクションが存在しない、またはプリコミットがタイムアウトした以外のエラーが発生した場合、以下の結果が返されます：

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

- トランザクションのコミットが成功した場合、以下の結果が返されます：

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

- トランザクションが既にコミットされている場合、以下の結果が返されます：

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

- トランザクションのコミットがタイムアウトした場合、以下の結果が返されます：

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

- トランザクションが存在しない、またはタイムアウトした以外のエラーが発生した場合、以下の結果が返されます：

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

- トランザクションのロールバックが成功した場合、以下の結果が返されます：

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

- トランザクションが存在しない以外のエラーが発生した場合、以下の結果が返されます：

  ```Bash
  {
      "TxnId": 1,
      "Label": "streamload_txn_example1_table1",
      "Status": "FAILED",
      "Message": ""
  }
  ```

## 関連ドキュメント

Stream Load が適用されるビジネスシナリオ、サポートされるデータファイル形式、基本原理などの情報については、[Stream Load を使用してローカルからインポートする](../loading/StreamLoad.md#使用-stream-load-からローカルにインポートする)を参照してください。

Stream Load ジョブを作成するための構文とパラメータについては、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。
