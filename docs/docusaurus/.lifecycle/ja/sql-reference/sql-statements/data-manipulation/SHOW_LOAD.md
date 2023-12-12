---
displayed_sidebar: "Japanese"
---

# SHOW LOAD（ロードジョブの表示）

## 説明

データベース内のすべてのロードジョブの情報または指定されたロードジョブの情報を表示します。このステートメントは、[Broker Load](../data-manipulation/BROKER_LOAD.md)、[INSERT](./INSERT.md)、または[Spark Load](../data-manipulation/SPARK_LOAD.md)を使用して作成されたロードジョブのみを表示できます。`curl`コマンドを使用してロードジョブの情報を表示することもできます。v3.1以降、`information_schema`データベースの`loads`テーブルから[Broker Load](../data-manipulation/BROKER_LOAD.md)またはInsertジョブの結果をクエリするには[SELECT](../data-manipulation/SELECT.md)ステートメントを使用することをお勧めします。詳細については、[HDFSからのデータのロード](../../../loading/hdfs_load.md)、[クラウドストレージからのデータのロード](../../../loading/cloud_storage_load.md)、[INSERTを使用したデータのロード](../../../loading/InsertInto.md)、および[Apache Spark™を使用したバルクロード](../../../loading/SparkLoad.md)を参照してください。

前述のロード方法に加えて、StarRocksではStream LoadおよびRoutine Loadを使用してデータをロードすることもサポートしています。Stream Loadは同期操作であり、Stream Loadジョブの情報を直接返します。Routine Loadは非同期操作であり、[SHOW ROUTINE LOAD](../data-manipulation/SHOW_ROUTINE_LOAD.md)ステートメントを使用してRoutine Loadジョブの情報を表示できます。

## 構文

```SQL
SHOW LOAD [ FROM db_name ]
[
   WHERE [ LABEL { = "label_name" | LIKE "label_matcher" } ]
         [ [AND] STATE = { "PENDING" | "ETL" | "LOADING" | "FINISHED" | "CANCELLED" } ]
]
[ ORDER BY field_name [ ASC | DESC ] ]
[ LIMIT { [offset, ] limit | limit OFFSET offset } ]
```

> **ノート**
>
> 通常の水平テーブル形式ではなく、出力を縦に表示するには、ステートメントに`\G`オプションを追加できます（例: `SHOW LOAD WHERE LABEL = "label1"\G;`）。詳細については、[例 1](#examples)を参照してください。

## パラメータ

| **パラメータ**                     | **必須** | **説明**                                              |
| --------------------------------- | ------------ | ------------------------------------------------------------ |
| db_name                           | いいえ           | データベース名。このパラメータが指定されていない場合、デフォルトで現在のデータベースが使用されます。 |
| LABEL = "label_name"              | いいえ           | ロードジョブのラベル。                                     |
| LABEL LIKE "label_matcher"        | いいえ           | このパラメータが指定されている場合、`label_matcher`を含むロードジョブの情報が返されます。 |
| AND                               | いいえ           | <ul><li>WHERE句でフィルタ条件を1つだけ指定する場合、このキーワードは指定しないでください。例: `WHERE STATE = "PENDING"`。</li><li>WHERE句で2つまたは3つのフィルタ条件を指定する場合、このキーワードを指定する必要があります。例: `WHERE LABEL = "label_name" AND STATE = "PENDING"`。</li></ul>|
| STATE                             | いいえ           | ロードジョブの状態。ロード方法によって状態が異なります。<ul><li>Broker Load<ul><li>`PENDING`: ロードジョブが作成されました。</li><li>`QUEUEING`: ロードジョブはスケジュール待ちのキューにあります。</li><li>`LOADING`: ロードジョブが実行中です。</li><li>`PREPARED`: トランザクションが確定されました。</li><li>`FINISHED`: ロードジョブが成功しました。</li><li>`CANCELLED`: ロードジョブが失敗しました。</li></ul></li><li>Spark Load<ul><li>`PENDING`: StarRocksクラスターがETLに関連する構成を準備し、その後、ETLジョブをApache Spark™クラスターに提出しています。</li><li>`ETL`: SparkクラスターがETLジョブを実行し、その後、データを対応するHDFSクラスターに書き込みます。</li><li>`LOADING`: HDFSクラスターのデータがStarRocksクラスターにロードされています。つまり、ロードジョブが実行中です。</li><li>`PREPARED`: トランザクションが確定されました。</li><li>`FINISHED`: ロードジョブが成功しました。</li><li>`CANCELLED`: ロードジョブが失敗しました。</li></ul></li><li>INSERT<ul><li>`FINISHED`: ロードジョブが成功しました。</li><li>`CANCELLED`: ロードジョブが失敗しました。</li></ul></li></ul>`STATE`パラメータが指定されていない場合、デフォルトですべての状態のロードジョブの情報が返されます。`STATE`パラメータが指定されている場合、指定した状態のロードジョブの情報のみが返されます。たとえば、`STATE = "PENDING"`は`PENDING`状態のロードジョブの情報を返します。 |
| ORDER BY field_name [ASC \| DESC] | いいえ           | このパラメータが指定されている場合、出力はフィールドに基づいて昇順または降順でソートされます。次のフィールドがサポートされています: `JobId`、`Label`、`State`、`Progress`、`Type`、`EtlInfo`、`TaskInfo`、`ErrorMsg`、`CreateTime`、`EtlStartTime`、`EtlFinishTime`、`LoadStartTime`、`LoadFinishTime`、`URL`、および`JobDetails`。<ul><li>出力を昇順でソートするには、`ORDER BY field_name ASC`を指定します。</li><li>出力を降順でソートするには、`ORDER BY field_name DESC`を指定します。</li></ul>フィールドとソート順が指定されていない場合、デフォルトで`JobId`の昇順で出力がソートされます。 |
| LIMIT limit                       | いいえ           | 表示が許可されるロードジョブの数。このパラメータが指定されていない場合、フィルタ条件に一致するすべてのロードジョブの情報が表示されます。たとえば、`LIMIT 10`と指定されている場合、フィルタ条件に一致する10件のロードジョブの情報のみが返されます。 |
| OFFSET offset                     | いいえ           | `offset`パラメータはスキップするロードジョブの数を定義します。たとえば、`OFFSET 5`は最初の5つのロードジョブをスキップし、残りを返します。`offset`パラメータの値はデフォルトで`0`になります。 |

## 出力

```SQL
+-------+-------+-------+----------+------+---------+----------+----------+------------+--------------+---------------+---------------+----------------+-----+------------+
| JobId | Label | State | Progress | Type | Priority | EtlInfo | TaskInfo | ErrorMsg | CreateTime | EtlStartTime | EtlFinishTime | LoadStartTime | LoadFinishTime | URL | JobDetails |
+-------+-------+-------+----------+------+---------+----------+----------+------------+--------------+---------------+---------------+----------------+-----+------------+
```

このステートメントの出力は、ロード方法に応じて異なります。

| **フィールド**      | **Broker Load**                                              | **Spark Load**                                               | **INSERT**                                                   |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| JobId          | StarRocksがStarRocksクラスター内のロードジョブを識別するために割り当てたユニークID。 | Spark Loadジョブでのこのフィールドの意味は、Broker Loadジョブでのその意味と同じです。 | INSERTジョブでのこのフィールドの意味は、Broker Loadジョブでのその意味と同じです。 |
| Label          | ロードジョブのラベル。ロードジョブのラベルはデータベース内で一意ですが、異なるデータベース間で重複することがあります。 | Spark Loadジョブでのこのフィールドの意味は、Broker Loadジョブでのその意味と同じです。 | INSERTジョブでのこのフィールドの意味は、Broker Loadジョブでのその意味と同じです。 |
| State          | ロードジョブの状態。<ul><li>`PENDING`: ロードジョブが作成されました。</li><li>`QUEUEING`: ロードジョブはスケジュール待ちのキューにあります。</li><li>`LOADING`: ロードジョブが実行中です。</li><li>`PREPARED`: トランザクションが確定されました。</li><li>`FINISHED`: ロードジョブが成功しました。</li><li>`CANCELLED`: ロードジョブが失敗しました。</li></ul> | ロードジョブの状態。<ul><li>`PENDING`: StarRocksクラスターがETLに関連する構成を準備し、その後、ETLジョブをSparkクラスターに提出しています。</li><li>`ETL`: SparkクラスターがETLジョブを実行し、その後、データを対応するHDFSクラスターに書き込みます。</li><li>`LOADING`: HDFSクラスターのデータがStarRocksクラスターにロードされています。つまり、ロードジョブが実行中です。</li><li>`PREPARED`: トランザクションが確定されました。</li><li>`FINISHED`: ロードジョブが成功しました。</li><li>`CANCELLED`: ロードジョブが失敗しました。</li></ul> | ロードジョブの状態。<ul><li>`FINISHED`: ロードジョブが成功しました。</li><li>`CANCELLED`: ロードジョブが失敗しました。</li></ul> |
| Progress       | The stage of the load job. A Broker Load job only has the `LOAD` stage, which ranges from 0% to 100% to describe the progress of the stage. When the load job enters the `LOAD` stage, `LOADING` is returned for the `State` parameter. A Broker Load job does not have the `ETL` stage. The `ETL` parameter is valid only for a Spark Load job.<br />**Note**<ul><li>The formula to calculate the progress of the `LOAD` stage: Number of StarRocks tables that complete data loading/Number of StarRocks tables that you plan to load data into * 100%.</li><li>When all data is loaded into StarRocks, `99%` is returned for the `LOAD` parameter. Then, loaded data starts taking effect in StarRocks. After the data takes effect, `100%` is returned for the `LOAD` parameter.</li><li>The progress of the `LOAD` stage is not linear. Therefore, the value of the `LOAD` parameter may not change over a period of time even if data loading is still ongoing.</li></ul> | The stage of the load job. A Spark Load job has two stages:<ul><li>`ETL`: ranges from 0% to 100% to describe the progress of the `ETL` stage.</li><li>`LOAD`: ranges from 0% to 100% to describe the progress of the `Load` stage. </li></ul>When the load job enters the `ETL` stage, `ETL` is returned for the `State` parameter. When the load job moves to the `LOAD` stage, `LOADING` is returned for the `State` parameter. <br />The **Note** is the same as those for Broker Load. | The stage of the load job. An INSERT job only has the `LOAD` stage, which ranges from 0% to 100% to describe the progress of the stage. When the load job enters the `LOAD` stage, `LOADING` is returned for the `State` parameter. An INSERT job does not have the `ETL` stage. The `ETL` parameter is valid only for a Spark Load job.<br />The **Note** is the same as those for Broker Load. |
| Type           | The method of the load job. The value of this parameter defaults to `BROKER`. | The method of the load job. The value of this parameter defaults to `SPARK`. | The method of the load job. The value of this parameter defaults to `INSERT`. |
| Priority       | The priority of the load job. Valid values: LOWEST, LOW, NORMAL, HIGH, and HIGHEST. | - | - |
| EtlInfo        | The metrics related to ETL.<ul><li>`unselected.rows`: The number of rows that are filtered out by the WHERE clause.</li><li>`dpp.abnorm.ALL`: The number of rows that are filtered out due to data quality issues, which refers to mismatches between source tables and StarRocks tables in, for example, the data type and the number of columns.</li><li>`dpp.norm.ALL`: The number of rows that are loaded into your StarRocks cluster.</li></ul>The sum of the preceding metrics is the total number of rows of raw data. You can use the following formula to calculate whether the percentage of unqualified data exceeds the value of the `max-filter-ratio` parameter:`dpp.abnorm.ALL`/(`unselected.rows` + `dpp.abnorm.ALL` + `dpp.norm.ALL`). | The field has the same meaning in a Spark Load job as it does in a Broker Load job. | The metrics related to ETL. An INSERT job does not have the `ETL` stage. Therefore, `NULL` is returned. |
| TaskInfo       | The parameters that are specified when you create the load job.<ul><li>`resource`: This parameter is valid only in a Spark Load job.</li><li>`timeout`: The time period that a load job is allowed to run. Unit: seconds.</li><li>`max-filter-ratio`: The largest percentage of rows that are filtered out due to data quality issues.</li></ul>For more information, see [BROKER LOAD](../data-manipulation/BROKER_LOAD.md). | The parameters that are specified when you create the load job.<ul><li>`resource`: The resource name.</li><li>`timeout`: The time period that a load job is allowed to run. Unit: seconds.</li><li>`max-filter-ratio`: The largest percentage of rows that are filtered out due to data quality issues.</li></ul>For more information, see [SPARK LOAD](../data-manipulation/SPARK_LOAD.md). | The parameters that are specified when you create the load job.<ul><li>`resource`: This parameter is valid only in a Spark Load job.</li><li>`timeout`: The time period that a load job is allowed to run. Unit: seconds.</li><li>`max-filter-ratio`: The largest percentage of rows that are filtered out due to data quality issues.</li></ul>For more information, see [INSERT](./INSERT.md). |
| ErrorMsg       | The error message returned when the load job fails. When the state of the loading job is `PENDING`, `LOADING`, or `FINISHED`, `NULL` is returned for the `ErrorMsg` field. When the state of the loading job is `CANCELLED`, the value returned for the `ErrorMsg` field consists of two parts: `type` and `msg`.<ul><li>The `type` part can be any of the following values:<ul><li>`USER_CANCEL`: The load job was manually canceled.</li><li>`ETL_SUBMIT_FAIL`: The load job failed to be submitted.</li><li>`ETL-QUALITY-UNSATISFIED`: The load job failed because the percentage of unqualified data exceeds the value of the `max-filter-ratio` parameter.</li><li>`LOAD-RUN-FAIL`: The load job failed in the `LOAD` stage.</li><li>`TIMEOUT`: The load job failed to finish within the specified timeout period.</li><li>`UNKNOWN`: The load job failed due to an unknown error.</li></ul></li><li>The `msg` part provides the detailed cause of the load failure.</li></ul> | The error message returned when the load job fails. When the state of the loading job is `PENDING`, `LOADING`, or `FINISHED`, `NULL` is returned for the `ErrorMsg` field. When the state of the loading job is `CANCELLED`, the value returned for the `ErrorMsg` field consists of two parts: `type` and `msg`.<ul><li>The `type` part can be any of the following values:<ul><li>`USER_CANCEL`: The load job was manually canceled.</li><li>`ETL_SUBMIT_FAIL`: StarRocks failed to submit an ETL job to Spark.</li><li>`ETL-RUN-FAIL`: Spark failed to execute the ETL job. </li><li>`ETL-QUALITY-UNSATISFIED`: The load job failed because the percentage of unqualified data exceeds the value of the `max-filter-ratio` parameter.</li><li>`LOAD-RUN-FAIL`: The load job failed in the `LOAD` stage.</li><li>`TIMEOUT`: The load job failed to finish within the specified timeout period.</li><li>`UNKNOWN`: The load job failed due to an unknown error.</li></ul></li><li>The `msg` part provides the detailed cause of the load failure.</li></ul> | The error message returned when the load job fails. When the state of the loading job is `FINISHED`, `NULL` is returned for the `ErrorMsg` field. When the state of the loading job is `CANCELLED`, the value returned for the `ErrorMsg` field consists of two parts: `type` and `msg`.<ul><li>The `type` part can be any of the following values:<ul><li>`USER_CANCEL`: The load job was manually canceled.</li><li>`ETL_SUBMIT_FAIL`: The load job failed to be submitted.</li><li>`ETL_RUN_FAIL`: The load job failed to run.</li><li>`ETL_QUALITY_UNSATISFIED`: The load job failed due to quality issues of raw data.</li><li>`LOAD-RUN-FAIL`: The load job failed in the `LOAD` stage.</li><li>`TIMEOUT`: The load job failed to finish within the specified timeout period.</li><li>`UNKNOWN`: The load job failed due to an unknown error.</li><li>`TXN_UNKNOWN`: The load job failed because the state of the transaction of the load job is unknown.</li></ul></li><li>The `msg` part provides the detailed cause of the load failure.</li></ul> |
| CreateTime     | The time at which the load job was created.                  | The field has the same meaning in a Spark Load job as it does in a Broker Load job. | The field has the same meaning in a INSERT job as it does in a Broker Load job. |
| EtlStartTime   | `ETL`ステージがないブローカーロードジョブ。したがって、このフィールドの値は`LoadStartTime`フィールドの値と同じです。 | `ETL`ステージが開始する時間。                   | INSERTジョブには`ETL`ステージがないため、このフィールドの値は`LoadStartTime`フィールドの値と同じです。 |
| EtlFinishTime  | `ETL`ステージがないブローカーロードジョブ。したがって、このフィールドの値は`LoadStartTime`フィールドの値と同じです。 | `ETL`ステージが終了する時間。                 | INSERTジョブには`ETL`ステージがないため、このフィールドの値は`LoadStartTime`フィールドの値と同じです。 |
| LoadStartTime  | `LOAD`ステージが開始する時間。                  | このフィールドはSparkロードジョブと同様に、ブローカーロードジョブと同じ意味を持ちます。 | このフィールドはINSERTジョブと同様に、ブローカーロードジョブと同じ意味を持ちます。 |
| LoadFinishTime | ロードジョブが終了する時間。                   | このフィールドはSparkロードジョブと同様に、ブローカーロードジョブと同じ意味を持ちます。 | このフィールドはINSERTジョブと同様に、ブローカーロードジョブと同じ意味を持ちます。 |
| URL            | ロードジョブで検出された無資格データにアクセスするために使用されるURL。`curl`コマンドまたは`wget`コマンドを使用してURLにアクセスし、無資格データを取得できます。無資格データが検出されない場合は、`NULL`が返されます。 | このフィールドはSparkロードジョブと同様に、ブローカーロードジョブと同じ意味を持ちます。 | このフィールドはINSERTジョブと同様に、ブローカーロードジョブと同じ意味を持ちます。 |
| JobDetails     | ロードジョブに関連するその他の情報。<ul><li>`未完了のバックエンド`: データのロードが完了しないBEのID。</li><li>`スキャンされた行数`: StarRocksにロードされた行の総数およびフィルタリングされた行の数。</li><li>`タスク数`: ロードジョブは1つ以上のタスクに分割される場合があり、それらは同時に実行されます。このフィールドはロードタスクの数を示します。</li><li>`全バックエンド`: データロードを実行しているBEのID。</li><li>`ファイル数`: ソースデータファイルの数。</li><li>`ファイルサイズ`: ソースデータファイルのデータ容量。単位: バイト。</li></ul> | このフィールドはSparkロードジョブと同様に、ブローカーロードジョブと同じ意味を持ちます。 | このフィールドはINSERTジョブと同様に、ブローカーロードジョブと同じ意味を持ちます。 |

## 使用上の注意

- `LOAD`ジョブの`LoadFinishTime`から3日間有効な`SHOW LOAD`ステートメントの情報。3日経過すると情報を表示できなくなります。`label_keep_max_second`パラメータを使用してデフォルトの有効期間を変更できます。

    ```SQL
    ADMIN SET FRONTEND CONFIG ("label_keep_max_second" = "value");
    ```

- `LoadStartTime`フィールドの値が長時間`N/A`の場合、ロードジョブが大量に積み重なっていることを意味します。ロードジョブの作成頻度を減らすことをお勧めします。
- ロードジョブによって消費される合計時間期間 = `LoadFinishTime` - `CreateTime`。
- ロードジョブが`LOAD`ステージで消費する合計時間期間 = `LoadFinishTime` - `LoadStartTime`。

## 例

例1: 現在のデータベース内のすべてのロードジョブを垂直に表示します。

```plaintext
SHOW LOAD\G;
*************************** 1. 行 ***************************
         JobId: 976331
         Label: duplicate_table_with_null
         State: FINISHED
      Progress: ETL:100%; LOAD:100%
          Type: BROKER
      Priority: NORMAL
       EtlInfo: unselected.rows=0; dpp.abnorm.ALL=0; dpp.norm.ALL=65546
      TaskInfo: resource:N/A; timeout(s):300; max_filter_ratio:0.0
     ErrorMsg: NULL
    CreateTime: 2022-10-17 19:35:00
  EtlStartTime: 2022-10-17 19:35:04
 EtlFinishTime: 2022-10-17 19:35:04
 LoadStartTime: 2022-10-17 19:35:04
LoadFinishTime: 2022-10-17 19:35:06
           URL: NULL
    JobDetails: {"Unfinished backends":{"b90a703c-6e5a-4fcb-a8e1-94eca5be0b8f":[]},"ScannedRows":65546,"TaskNumber":1,"All backends":{"b90a703c-6e5a-4fcb-a8e1-94eca5be0b8f":[10004]},"FileNumber":1,"FileSize":548622}
```

例2: 現在のデータベース内のラベルに文字列`null`が含まれる2つのロードジョブを表示します。

```plaintext
SHOW LOAD 
WHERE LABEL LIKE "null" 
LIMIT 2;

+-------+---------------------------+----------+---------------------+--------+---------------------------------------------------------+---------------------------------------------------------------------------------------------------------+----------+---------------------+---------------------+---------------------+---------------------+---------------------+--------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| JobId | Label                     | State    | Progress            | Type   | EtlInfo                                                 | TaskInfo                                                                                                | ErrorMsg | CreateTime          | EtlStartTime        | EtlFinishTime       | LoadStartTime       | LoadFinishTime      | URL                                                                            | JobDetails                                                                                                                                                                                              |
+-------+---------------------------+----------+---------------------+--------+---------------------------------------------------------+---------------------------------------------------------------------------------------------------------+----------+---------------------+---------------------+---------------------+---------------------+---------------------+--------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 10082 | duplicate_table_with_null | FINISHED | ETL:100%; LOAD:100% | BROKER | unselected.rows=0; dpp.abnorm.ALL=0; dpp.norm.ALL=65546 | resource:N/A; timeout(s):300; max_filter_ratio:0.0                                                      | NULL     | 2022-08-02 14:53:27 | 2022-08-02 14:53:30 | 2022-08-02 14:53:30 | 2022-08-02 14:53:30 | 2022-08-02 14:53:31 | NULL                                                                           | {"Unfinished backends":{"4393c992-5da1-4e9f-8b03-895dc0c96dbc":[]},"ScannedRows":65546,"TaskNumber":1,"All backends":{"4393c992-5da1-4e9f-8b03-895dc0c96dbc":[10002]},"FileNumber":1,"FileSize":548622} |
| 10103 | unique_table_with_null    | FINISHED | ETL:100%; LOAD:100% | SPARK  | unselected.rows=0; dpp.abnorm.ALL=0; dpp.norm.ALL=65546 | resource:test_spark_resource_07af473a_1230_11ed_b483_00163e0e550b; timeout(s):300; max_filter_ratio:0.0 | NULL     | 2022-08-02 14:56:06 | 2022-08-02 14:56:19 | 2022-08-02 14:56:41 | 2022-08-02 14:56:41 | 2022-08-02 14:56:44 | http://emr-header-1.cluster-49091:20888/proxy/application_1655710334658_26391/ | {"Unfinished backends":{"00000000-0000-0000-0000-000000000000":[]},"ScannedRows":65546,"TaskNumber":1,"All backends":{"00000000-0000-0000-0000-000000000000":[-1]},"FileNumber":1,"FileSize":8790855}   |
+-------+---------------------------+----------+---------------------+--------+---------------------------------------------------------+---------------------------------------------------------------------------------------------------------+----------+---------------------+---------------------+---------------------+---------------------+---------------------+--------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

```

例3: `example_db`内のラベルに文字列`table`が含まれるロードジョブを、`LoadStartTime`フィールドの降順で表示します。

```plaintext
SHOW LOAD FROM example_db 
WHERE LABEL Like "table" 
ORDER BY LoadStartTime DESC;

+-------+---------------------------+----------+---------------------+--------+---------------------------------------------------------+---------------------------------------------------------------------------------------------------------+----------+---------------------+---------------------+---------------------+---------------------+---------------------+--------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```plaintext
| 10103 | unique_table_with_null                      | FINISHED | ETL:100%; LOAD:100% | SPARK  | unselected.rows=0; dpp.abnorm.ALL=0; dpp.norm.ALL=65546 | resource:test_spark_resource_07af473a_1230_11ed_b483_00163e0e550b; timeout(s):300; max_filter_ratio:0.0 | NULL | 2022-08-02 14:56:06 | 2022-08-02 14:56:19 | 2022-08-02 14:56:41 | 2022-08-02 14:56:41 | 2022-08-02 14:56:44 | http://emr-header-1.cluster-49091:20888/proxy/application_1655710334658_26391/ | {"Unfinished backends":{"00000000-0000-0000-0000-000000000000":[]},"ScannedRows":65546,"TaskNumber":1,"All backends":{"00000000-0000-0000-0000-000000000000":[-1]},"FileNumber":1,"FileSize":8790855}   |
| 10082 | duplicate_table_with_null                   | FINISHED | ETL:100%; LOAD:100% | BROKER | unselected.rows=0; dpp.abnorm.ALL=0; dpp.norm.ALL=65546 | resource:N/A; timeout(s):300; max_filter_ratio:0.0                                                      | NULL | 2022-08-02 14:53:27 | 2022-08-02 14:53:30 | 2022-08-02 14:53:30 | 2022-08-02 14:53:30 | 2022-08-02 14:53:31 | NULL                                                                           | {"Unfinished backends":{"4393c992-5da1-4e9f-8b03-895dc0c96dbc":[]},"ScannedRows":65546,"TaskNumber":1,"All backends":{"4393c992-5da1-4e9f-8b03-895dc0c96dbc":[10002]},"FileNumber":1,"FileSize":548622} |
```