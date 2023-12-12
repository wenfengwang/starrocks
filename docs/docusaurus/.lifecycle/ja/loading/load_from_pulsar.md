```yaml
---
displayed_sidebar: "Japanese"
---
# [プレビュー] Apache® Pulsar™ からデータを連続して読み込む

StarRocksのバージョン2.5では、Routine Load はApache® Pulsar™ からデータを連続して読み込むことをサポートしています。Pulsarは、分散型のオープンソースのパブサブメッセージングおよびストリーミングプラットフォームであり、ストアコンピュート分離アーキテクチャを持っています。Routine Loadを使用してPulsarからデータを読み込む方法は、Apache Kafka からデータを読み込むのと類似しています。このトピックでは、CSV形式のデータを例に挙げ、Apache PulsarからデータをRoutine Loadを介して読み込む方法を紹介します。

## サポートされているデータファイル形式

Routine Load は、PulsarクラスタからCSVおよびJSON形式のデータを消費することをサポートしています。

> 注意
>
> CSV形式のデータの場合、StarRocksでは、列の区切りとして50バイト以内のUTF-8エンコード文字列がサポートされています。一般的に使用される列の区切り文字には、カンマ(,)、タブ、およびパイプ(|)があります。

## Pulsarに関連する概念

**[トピック](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#topics)**

Pulsarのトピックは、プロデューサーからコンシューマーへのメッセージ送信用の名前付きチャンネルです。Pulsarのトピックは、パーティション化されたトピックと非パーティション化されたトピックに分かれています。

- **[パーティション化されたトピック](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#partitioned-topics)** は、複数のブローカーによって処理される特別なタイプのトピックであり、それによりより高いスループットが可能となります。パーティション化されたトピックは、実際にはN個の内部トピックとして実装されており、Nはパーティションの数です。
- **非パーティション化されたトピック** は、単一のブローカーによってのみ処理される通常のタイプのトピックであり、トピックの最大スループットが制限されます。

**[メッセージID](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#messages)**

メッセージのメッセージIDは、メッセージが永続的に保存されるとすぐに[BookKeeperインスタンス](https://pulsar.apache.org/docs/2.10.x/concepts-architecture-overview/#apache-bookkeeper)によって割り当てられます。メッセージIDは、メッセージがレジャー内の特定の位置を示し、Pulsarクラスタ内でユニークです。

Pulsarは、消費者がconsumer.*seek*(*messageId*)を使用して初期位置を指定することをサポートしています。ただし、Kafkaの消費者オフセットが長整数値であるのに対し、メッセージIDは"ledgerId:entryID:partition-index:batch-index"の4つの部分で構成されています。

したがって、メッセージIDをメッセージから直接取得することはできません。その結果、現時点では、Routine LoadはPulsarからデータを読み込む際に初期位置を指定することをサポートしておらず、パーティションの先頭または末尾からのデータの消費のみをサポートしています。

**[サブスクリプション](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#subscriptions)**

サブスクリプションは、メッセージがどのようにコンシューマーに配信されるかを決定する名前付きの構成ルールです。Pulsarでは、複数のトピックを同時に購読するコンシューマーもサポートされています。1つのトピックには複数のサブスクリプションを持つことができます。

サブスクリプションのタイプは、コンシューマーが接続する際に定義され、そのタイプは異なる構成で全てのコンシューマーを再起動させることで変更することができます。Pulsarでは、以下の4つのサブスクリプションタイプが利用可能です：

- `exclusive`(デフォルト): 1つのコンシューマーのみがサブスクリプションに接続することができます。メッセージは1つのカスタマーにのみ配信されます。
- `shared`: 複数のコンシューマーが同じサブスクリプションに接続することができます。メッセージはコンシューマー間でラウンドロビン配信され、特定のメッセージは1つのコンシューマーにのみ配信されます。
- `failover`: 複数のコンシューマーが同じサブスクリプションに接続することができます。パーティション化されていないトピックの場合、またはパーティション化されたトピックの各パーティションの場合、マスターコンシューマーが選択され、メッセージを受信します。マスターコンシューマーが切断されると、全ての（未確認のおよびその後の）メッセージは次の順番のコンシューマーに配信されます。
- `key_shared`: 複数のコンシューマーが同じサブスクリプションに接続することができます。メッセージはコンシューマー間で分散され、同じキーまたは同じ順序キーを持つメッセージは1つのコンシューマーにのみ配信されます。

> 注：
>
> 現在、Routine Loadはexclusiveタイプを使用しています。

## Routine Loadジョブの作成

以下の例は、PulsarでCSV形式のメッセージを消費し、Routine Loadジョブを作成してStarRocksにデータをロードする方法を説明しています。詳細な手順と参照については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

```SQL
CREATE ROUTINE LOAD load_test.routine_wiki_edit_1 ON routine_wiki_edit
COLUMNS TERMINATED BY ",",
ROWS TERMINATED BY "\n",
COLUMNS (order_id, pay_dt, customer_name, nationality, temp_gender, price)
WHERE event_time > "2022-01-01 00:00:00",
PROPERTIES
(
    "desired_concurrent_number" = "1",
    "max_batch_interval" = "15000",
    "max_error_number" = "1000"
)
FROM PULSAR
(
    "pulsar_service_url" = "pulsar://localhost:6650",
    "pulsar_topic" = "persistent://tenant/namespace/topic-name",
    "pulsar_subscription" = "load-test",
    "pulsar_partitions" = "load-partition-0,load-partition-1",
    "pulsar_initial_positions" = "POSITION_EARLIEST,POSITION_LATEST",
    "property.auth.token" = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUJzdWIiOiJqaXV0aWFuY2hlbiJ9.lulGngOC72vE70OW54zcbyw7XdKSOxET94WT_hIqD5Y"
);
```

Pulsarからデータを消費するためにRoutine Loadを作成する際、`data_source_properties`以外のほとんどの入力パラメータは、Kafkaからデータを消費する場合と同じです。`data_source_properties`を除くパラメータの説明については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

`data_source_properties`に関連するパラメータとその説明は以下の通りです：

| **パラメータ**                               | **必須** | **説明**                                              |
| ------------------------------------------ | -------- | ----------------------------------------------------- |
| pulsar_service_url                          | はい      | Pulsarクラスタに接続するために使用されるURL。形式：`"pulsar://ip:port"`または`"pulsar://service:port"`。例：`"pulsar_service_url" = "pulsar://localhost:6650"` |
| pulsar_topic                                | はい      | 購読するトピック。例："pulsar_topic" = "persistent://tenant/namespace/topic-name" |
| pulsar_subscription                         | はい      | トピックに構成されたサブスクリプション。例："pulsar_subscription" = "my_subscription" |
| pulsar_partitions, pulsar_initial_positions | いいえ    | `pulsar_partitions`：トピックのサブスクライブされたパーティション。 `pulsar_initial_positions`：`pulsar_partitions`で指定されたパーティションの初期位置。初期位置は、`pulsar_partitions`に相当する必要があります。有効な値：`POSITION_EARLIEST`（デフォルト値）：パーティション内の最初の利用可能なメッセージからのサブスクリプションが開始されます。 `POSITION_LATEST`：パーティション内の最新の利用可能なメッセージからのサブスクリプションが開始されます。注意：`pulsar_partitions`が指定されていない場合、トピックのすべてのパーティションが購読されます。`pulsar_partitions`と`property.pulsar_default_initial_position`の両方が指定されている場合、`pulsar_partitions`の値が`property.pulsar_default_initial_position`の値を上書きします。`pulsar_partitions`または`property.pulsar_default_initial_position`のいずれもが指定されていない場合、サブスクリプションが最新の利用可能なメッセージから開始されます。例：`"pulsar_partitions" = "my-partition-0,my-partition-1,my-partition-2,my-partition-3", "pulsar_initial_positions" = "POSITION_EARLIEST,POSITION_EARLIEST,POSITION_LATEST,POSITION_LATEST"` |

Routine Loadは、Pulsarのための以下のカスタムパラメータをサポートしています。

| パラメータ                                  | 必須 | 説明                                                   |
| ---------------------------------------- | ---- | ------------------------------------------------------ |
| property.pulsar_default_initial_position | いいえ | トピックのパーティションが購読されている際のデフォルトの初期位置。 `pulsar_initial_positions`が指定されていない場合、このパラメータが効果を発揮します。有効な値は、`pulsar_initial_positions`の有効な値と同じです。例：`"property.pulsar_default_initial_position" = "POSITION_EARLIEST"` |
| property.auth.token                      | いいえ | Pulsarがセキュリティトークンを使用してクライアントの認証を有効にしている場合、トークン文字列が必要です。例：`"property.auth.token" = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUJzdWIiOiJqaXV0aWFuY2hlbiJ9.lulGngOC72vE70OW54zcbyw7XdKSOxET94WT_hIqD"` |

## ロードジョブおよびタスクの確認

### ロードジョブのチェック

[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation//SHOW_ROUTINE_LOAD.md)ステートメントを実行して、ロードジョブ`routine_wiki_edit_1`のステータスを確認します。StarRocksは、実行状態`State`、統計情報（消費された合計行数およびロードされた合計行数）`Statistics`、およびロードジョブの進行状況`progress`を返します。
```
```Plaintext
SHOW ROUTINE LOADを実行してPulsarからデータを消費するルーチンロードジョブを確認すると、 `progress` を除く多くの返されるパラメーターはKafkaからデータを消費する場合と同じです。`progress` はバックログを指し、つまりパーティション内のアンアックされたメッセージの数です。

```MySQL
MySQL [load_test] > SHOW ROUTINE LOAD for routine_wiki_edit_1 \G
*************************** 1. row ***************************
                  Id: 10142
                Name: routine_wiki_edit_1
          CreateTime: 2022-06-29 14:52:55
           PauseTime: 2022-06-29 17:33:53
             EndTime: NULL
              DbName: default_cluster:test_pulsar
           TableName: test1
               State: PAUSED
      DataSourceType: PULSAR
      CurrentTaskNum: 0
       JobProperties: {"partitions":"*","rowDelimiter":"'\n'","partial_update":"false","columnToColumnExpr":"*","maxBatchIntervalS":"10","whereExpr":"*","timezone":"Asia/Shanghai","format":"csv","columnSeparator":"','","json_root":"","strict_mode":"false","jsonpaths":"","desireTaskConcurrentNum":"3","maxErrorNum":"10","strip_outer_array":"false","currentTaskConcurrentNum":"0","maxBatchRows":"200000"}
DataSourceProperties: {"serviceUrl":"pulsar://localhost:6650","currentPulsarPartitions":"my-partition-0,my-partition-1","topic":"persistent://tenant/namespace/topic-name","subscription":"load-test"}
    CustomProperties: {"auth.token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUJzdWIiOiJqaXV0aWFuY2hlbiJ9.lulGngOC72vE70OW54zcbyw7XdKSOxET94WT_hIqD"}
           Statistic: {"receivedBytes":5480943882,"errorRows":0,"committedTaskNum":696,"loadedRows":66243440,"loadRowsRate":29000,"abortedTaskNum":0,"totalRows":66243440,"unselectedRows":0,"receivedBytesRate":2400000,"taskExecuteTimeMs":2283166}
            Progress: {"my-partition-0(backlog): 100","my-partition-1(backlog): 0"}
ReasonOfStateChanged: 
        ErrorLogUrls: 
            OtherMsg:
1 row in set (0.00 sec)
```

### ロードタスクを確認する

`routine_wiki_edit_1` のロードタスクをチェックするために、 [SHOW ROUTINE LOAD TASK](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD_TASK.md) ステートメントを実行して、実行中のタスクの数、消費されるKafkaトピックパーティション、および消費進捗 `DataSourceProperties` 、対応するコーディネータBEノード `BeId` をチェックします。

```SQL
MySQL [example_db]> SHOW ROUTINE LOAD TASK WHERE JobName = "routine_wiki_edit_1" \G
```

## ロードジョブを変更する

ロードジョブを変更する前に、[PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md) ステートメントを使用して一時停止する必要があります。その後、[ALTER ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md) を実行することができます。変更した後は、[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) ステートメントを実行して再開し、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) ステートメントを使用してそのステータスを確認できます。

Pulsarからデータを消費するためにRoutine Loadを使用する場合、`data_source_properties` を除く多くの返されるパラメーターはKafkaからデータを消費する場合と同じです。

**次の点に注意してください**：

- `data_source_properties` 関連のパラメーターの中で、現在は`pulsar_partitions`、`pulsar_initial_positions`、およびカスタムPulsarパラメーター`property.pulsar_default_initial_position`、`property.auth.token` のみを変更することができます。 `pulsar_service_url`、`pulsar_topic`、`pulsar_subscription` のパラメーターは変更できません。
- 消費するパーティションと一致する初期位置を変更する必要がある場合は、ルーチンロードジョブを作成する際に `pulsar_partitions` を使用してパーティションを指定し、指定されたパーティションの初期位置のみを変更できることを確認する必要があります。
- ルーチンロードジョブを作成する際にTopicである `pulsar_topic` のみを指定し、パーティションを指定しなかった場合は、`pulsar_default_initial_position` を通じてトピック下のすべてのパーティションの開始位置を変更できます。
```