---
displayed_sidebar: Chinese
---

# 【パブリックベータ】Apache® Pulsar™ からの継続的なデータインポート

StarRocks バージョン2.5以降、Routine LoadはApache® Pulsar™からのメッセージを継続的に消費し、StarRocksにインポートする機能をサポートしています。Pulsarは分散型メッセージキューシステムで、ストレージと計算の分離アーキテクチャを採用しています。

Routine Loadを使用してPulsarからデータをインポートするプロセスは、Apache Kafkaからインポートするのと似ています。この記事では、CSV形式のデータファイルを例に、Routine Loadを通じてPulsarからデータを継続的にインポートする方法を説明します。

## サポートされるデータファイル形式

Routine Loadは現在、PulsarクラスタからCSVおよびJSON形式のデータを消費することをサポートしています。

> 説明
>
> CSV形式のデータについては、StarRocksは最大50バイトのUTF-8エンコード文字列を列の区切り文字として設定することをサポートしており、これにはコンマ (,)、タブ、パイプ (|) などが含まれます。

## Pulsarに関連する概念

**[トピック](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#topics)**

トピックはメッセージの保存と配信を担当します。プロデューサーはトピックにメッセージを書き込み、コンシューマーはトピックからメッセージを読み取ります。PulsarのトピックはPartitioned TopicとNon-Partitioned Topicの2種類に分かれています。<br />

- [Partitioned Topic](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#partitioned-topics)：複数のブローカーがサービスを提供し、より高いスループットをサポートできます。Pulsarは複数のInternal Topicを使用してPartitioned Topicを実現しています。
- Non-Partitioned Topic：単一のブローカーのみがサービスを提供し、トピックのスループットが制限されます。

**[メッセージID](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#messages)**

メッセージIDは、クラスター全体で一意のメッセージ識別子を表します。これはKafkaのオフセットに似ています。コンシューマーは特定のメッセージIDにシークして消費進行状況を移動させることができます。ただし、Kafkaのオフセットが長整数値であるのに対し、PulsarのメッセージIDは `ledgerId:entryID:partition-index:batch-index` の4部分で構成されています。

そのため、メッセージから直接メッセージIDを取得することはできません**。**現在、カスタムの開始ポジションをサポートしていないため、パーティションの開始または終了から消費を開始することのみがサポートされています**。

**[サブスクリプション](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#subscriptions)**

サブスクリプションは、メッセージがコンシューマーにどのように配信されるかを指示する命名された設定ルールです。以下の4種類のタイプがサポートされています：

- `exclusive` (デフォルト)：1つのサブスクリプションは1つのコンシューマーのみと関連付けられ、そのコンシューマーのみがトピックのすべてのメッセージを受信できます。
- `shared`：複数のコンシューマーが同じサブスクリプションに関連付けられ、メッセージはラウンドロビン方式でコンシューマーに配布されます。
- `failover`：複数のコンシューマーが同じサブスクリプションに関連付けられ、その中の一部がマスターコンシューマーとして機能します。Non-Partitioned Topicの場合、1つのトピックに1つのマスターコンシューマーが選出されます。Partitioned Topicの場合、1つのパーティションに1つのマスターコンシューマーが選出され、そのマスターコンシューマーがメッセージの消費を担当します。
- `key_shared`：複数のコンシューマーが同じサブスクリプションに関連付けられ、同じキーを持つメッセージは同じコンシューマーに配布されます。

> **注意**
>
> 現在、Routine Loadは `exclusive` モードを使用しています。

## インポートジョブの作成

[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) ステートメントを使用して、StarRocksにRoutine Loadインポートジョブ `routine_wiki_edit_1` を提出します。これは、Pulsarクラスター内のトピック `ordertest1` からのメッセージを継続的に消費し、サブスクリプション `load-test` を使用し、消費パーティションを `load-partition-0` と `load-partition-1` に指定し、それぞれのパーティションにデータがある位置から消費を開始し、パーティションの末尾から消費を開始します。そして、データベース `load_test` のテーブル `routine_wiki_edit` にインポートします。

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
    "pulsar_topic" = "persistent://tenant/namespace/ordertest1",
    "pulsar_subscription" = "load-test",
    "pulsar_partitions" = "load-partition-0,load-partition-1",
    "pulsar_initial_positions" = "POSITION_EARLIEST,POSITION_LATEST",
    "property.auth.token" = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUJzdWIiOiJqaXV0aWFuY2hlbiJ9.lulGngOC72vE70OW54zcbyw7XdKSOxET94WT_hIqD5Y"
);
```

Pulsarからのデータ消費において、`data_source_properties` 関連のパラメータ以外の設定方法は、[Kafkaを消費する](./RoutineLoad.md#インポートジョブ)場合と同じです。

`data_source_properties` 関連のパラメータとその説明は以下の表に示されています。

| **パラメータ**                                  | **必須**     | **説明**                                                     |
| ----------------------------------------------- | ------------ | ------------------------------------------------------------ |
| `pulsar_service_url`                            | はい         | Pulsarクラスターの接続情報、形式は `pulsar://ip:port` または `pulsar://service:port` です。例：`"pulsar_service_url" = "pulsar://localhost:6650"` |
| `pulsar_topic`                                  | はい         | 消費する必要があるトピック。例：`"pulsar_topic" = "persistent://tenant/namespace/topic-name"` |
| `pulsar_subscription`                           | はい         | 消費する必要があるトピックに対応するサブスクリプション。例：`"pulsar_subscription" = "my_subscription"` |

| `pulsar_partitions`、`pulsar_initial_positions` | はい         | `pulsar_partitions` は消費が必要なパーティションです。`pulsar_initial_positions` は対応するパーティションの消費開始位置です。パーティションを設定するごとに、対応するパーティションの消費開始位置も設定する必要があります。取りうる値は：`POSITION_EARLIEST`（デフォルト）：パーティションにデータがある位置から消費を開始します。`POSITION_LATEST`：パーティションの末尾から消費を開始します。**注意** `pulsar_partitions` を指定しない場合、Topic 下のすべてのパーティションが自動的に展開されて消費されます。`pulsar_initial_positions` と `property.pulsar_default_initial_position` の両方が指定されている場合、前者が後者の設定を上書きします。`pulsar_initial_positions` と `property.pulsar_default_initial_position` のどちらも指定されていない場合、サブスクリプションが確立された後に受信した最初のデータから消費を開始します。例：`"pulsar_partitions" = "my-partition-0,my-partition-1,my-partition-2,my-partition-3", "pulsar_initial_positions" = "POSITION_EARLIEST,POSITION_EARLIEST,POSITION_LATEST,POSITION_LATEST"` |

Routine Load はカスタム Pulsar パラメータもサポートしています。以下の表を参照してください。

| **パラメータ**                             | **必須**     | **説明**                                                     |
| ------------------------------------------ | ------------ | ------------------------------------------------------------ |
| `property.pulsar_default_initial_position` | いいえ       | 消費予定のパーティションのデフォルトの消費開始位置。`pulsar_initial_positions` が指定されていない場合にのみ有効です。有効な値は `pulsar_initial_positions` と同じです。例：`"property.pulsar_default_initial_position" = "POSITION_EARLIEST"` |
| `property.auth.token`                      | いいえ       | 認証と認可に使用される [token](https://pulsar.apache.org/docs/2.10.x/security-overview/) で、文字列です。例：`"property.auth.token" = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUJzdWIiOiJqaXV0aWFuY2hlbiJ9.lulGngOC72vE70OW54zcbyw7XdKSOxET94WT_hIqD"` |

## インポートジョブとタスクの確認

### インポートジョブの確認

Routine Load インポートジョブを提出した後、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) ステートメントを実行してインポートジョブの情報を確認できます。例えば、`routine_wiki_edit_1` という名前のインポートジョブの情報を以下のコマンドで確認できます：

```Plaintext
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
DataSourceProperties: {"serviceUrl":"pulsar://localhost:6650","currentPulsarPartitions":"my-partition-0,my-partition-1","topic":"persistent://tenant/namespace/ordertest1","subscription":"load-test"}
    CustomProperties: {"auth.token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUJzdWIiOiJqaXV0aWFuY2hlbiJ9.lulGngOC72vE70OW54zcbyw7XdKSOxET94WT_hIqD"}
           Statistic: {"receivedBytes":5480943882,"errorRows":0,"committedTaskNum":696,"loadedRows":66243440,"loadRowsRate":29000,"abortedTaskNum":0,"totalRows":66243440,"unselectedRows":0,"receivedBytesRate":2400000,"taskExecuteTimeMs":2283166}
            Progress: {"my-partition-0(backlog): 100","my-partition-1(backlog): 0"}
ReasonOfStateChanged: 
        ErrorLogUrls: 
            OtherMsg:
1 row in set (0.00 sec)
```

Pulsar クラスターからのデータ消費において、`progress` を除く他のパラメータおよびその意味は [Kafka の消費](./RoutineLoad.md#インポートジョブの確認) と同様です。`progress` は消費されたパーティションのバックログを表します。

### インポートタスクの確認

Routine Load インポートジョブを提出した後、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) を実行して、インポートジョブに含まれる一つまたは複数のインポートタスクの情報を確認できます。例えば、`routine_wiki_edit_1` という名前のインポートジョブに含まれるタスクの情報を以下のコマンドで確認できます。現在実行中のタスクの数、消費されるパーティションと進捗状況、`DataSourceProperties`、および対応する Coordinator BE ノード `BeId` などです。

```SQL
MySQL [example_db]> SHOW ROUTINE LOAD TASK WHERE JobName = "routine_wiki_edit_1" \G
```

Pulsar クラスターからのデータ消費において、パラメータおよびその意味は [Kafka の消費](./RoutineLoad.md#インポートタスクの確認) と類似しています。

## インポートジョブの変更

変更する前に、[PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md) を実行してインポートジョブを一時停止する必要があります。その後、[ALTER ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md) ステートメントを実行して、インポートジョブのパラメータ設定を変更します。変更が成功したら、[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) を実行してインポートジョブを再開します。その後、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) ステートメントを実行して、変更後のインポートジョブを確認します。

Pulsar クラスターからのデータ消費において、`data_source_properties` を除く他のパラメータの変更方法は [Kafka の消費](./RoutineLoad.md#インポートジョブの変更) と同様です。

変更する前に、[PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md) を実行してインポートジョブを一時停止する必要があります。その後、[ALTER ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md) ステートメントを実行して、インポートジョブのパラメータ設定を変更します。変更が成功したら、[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) を実行してインポートジョブを再開します。その後、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) ステートメントを実行して、変更後のインポートジョブを確認します。


Pulsar クラスターからデータを消費する場合、`data_source_properties` 以外のパラメータの変更方法は [Kafka を消費する](./RoutineLoad.md#変更インポートジョブ)場合と同じです。

注意：

- `data_source_properties` 関連のパラメータでは、現在 `pulsar_partitions`、`pulsar_initial_positions` の変更と、カスタム Pulsar パラメータ `property.pulsar_default_initial_position`、`property.auth.token` のみをサポートしています。`pulsar_service_url`、`pulsar_topic`、`pulsar_subscription` の変更はサポートしていません。
- 待機中の partition とそれに対応する起始 position を変更する必要がある場合は、Routine Load インポートジョブを作成する際に `pulsar_partitions` を使用して partition を指定していることを確認してください。また、指定された partition の起始 position `pulsar_initial_positions` のみ変更が可能で、partition の追加や削除はサポートしていません。
- Routine Load インポートジョブを作成する際に `pulsar_topic` のみを指定し、`pulsar_partitions` を指定していない場合は、`pulsar_default_initial_position` を使用して、その topic のすべての partition の起始 position を変更することができます。
