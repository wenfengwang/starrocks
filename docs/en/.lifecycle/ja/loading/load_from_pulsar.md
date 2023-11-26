---
displayed_sidebar: "Japanese"
---

# [プレビュー] Apache® Pulsar™ からデータを連続的にロードする

StarRocks バージョン 2.5 以降、Routine Load は Apache® Pulsar™ からデータを連続的にロードすることをサポートしています。Pulsar は、ストアとコンピュートを分離したアーキテクチャを持つ、分散型のオープンソースのパブサブメッセージングおよびストリーミングプラットフォームです。Pulsar を介してデータをロードする方法は、Apache Kafka からデータをロードする方法と似ています。このトピックでは、CSV 形式のデータを例として使用して、Apache Pulsar から Routine Load を使用してデータをロードする方法を紹介します。

## サポートされるデータファイル形式

Routine Load は、Pulsar クラスタから CSV 形式および JSON 形式のデータを消費することができます。

> 注意
>
> CSV 形式のデータに関しては、StarRocks は、カラムセパレータとして 50 バイト以内の UTF-8 エンコードされた文字列をサポートしています。一般的に使用されるカラムセパレータには、カンマ (,)、タブ、パイプ (|) があります。

## Pulsar 関連の概念

**[トピック](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#topics)**

Pulsar のトピックは、プロデューサからコンシューマへメッセージを送信するための名前付きチャネルです。Pulsar のトピックは、パーティション化されたトピックと非パーティション化されたトピックに分けられます。

- **[パーティション化されたトピック](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#partitioned-topics)** は、複数のブローカーによって処理される特殊なタイプのトピックであり、より高いスループットを実現します。パーティション化されたトピックは、実際には N 個の内部トピックとして実装されます。ここで、N はパーティションの数です。
- **非パーティション化されたトピック** は、単一のブローカーによってのみ提供される通常のトピックであり、トピックの最大スループットが制限されます。

**[メッセージ ID](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#messages)**

メッセージのメッセージ ID は、メッセージが永続的に格納されるとすぐに [BookKeeper インスタンス](https://pulsar.apache.org/docs/2.10.x/concepts-architecture-overview/#apache-bookkeeper) によって割り当てられます。メッセージ ID は、メッセージがレジャー内の特定の位置を示し、Pulsar クラスタ内で一意です。

Pulsar では、コンシューマがメッセージ ID を指定して初期位置を指定することができます。ただし、Kafka コンシューマのオフセットが長整数値であるのに対して、メッセージ ID は `ledgerId:entryID:partition-index:batch-index` の 4 つの部分から構成されます。

そのため、メッセージからメッセージ ID を直接取得することはできません。結果として、現在のところ、**Routine Load は Pulsar からデータをロードする際に初期位置を指定することはサポートしておらず、パーティションの先頭または末尾からのデータの消費のみをサポートしています。**

**[サブスクリプション](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#subscriptions)**

サブスクリプションは、メッセージがコンシューマにどのように配信されるかを決定する名前付きの設定ルールです。Pulsar は、複数のトピックに同時にサブスクライブするコンシューマもサポートしています。トピックは複数のサブスクリプションを持つことができます。

サブスクリプションのタイプは、コンシューマが接続するときに定義され、異なる構成ですべてのコンシューマを再起動することでタイプを変更することができます。Pulsar では、次の 4 つのサブスクリプションタイプが利用可能です。

- `exclusive` (デフォルト): サブスクリプションには単一のコンシューマのみがアタッチできます。メッセージは 1 つのコンシューマにのみ配信されます。
- `shared`: 複数のコンシューマが同じサブスクリプションにアタッチできます。メッセージはラウンドロビン方式でコンシューマに配信され、特定のメッセージは 1 つのコンシューマにのみ配信されます。
- `failover`: 複数のコンシューマが同じサブスクリプションにアタッチできます。非パーティション化されたトピックの場合は、マスターコンシューマが選択されてメッセージを受信します。マスターコンシューマが切断されると、(未確認のメッセージとその後のメッセージを含む) すべてのメッセージは次のコンシューマに配信されます。
- `key_shared`: 複数のコンシューマが同じサブスクリプションにアタッチできます。メッセージはコンシューマ間で分散して配信され、同じキーまたは同じ順序キーを持つメッセージは 1 つのコンシューマにのみ配信されます。

> 注意
>
> 現在のところ、Routine Load は exclusive タイプのみを使用します。

## Routine Load ジョブの作成

以下の例では、Pulsar で CSV 形式のメッセージを消費し、Routine Load ジョブを作成してデータを StarRocks にロードする方法について説明します。詳しい手順と参照については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。

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

Pulsar からデータをロードするために Routine Load を作成する際、`data_source_properties` を除くほとんどの入力パラメータは、Kafka からデータをロードする場合と同じです。`data_source_properties` を除くパラメータの説明については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。

`data_source_properties` に関連するパラメータとその説明は以下の通りです:

| **パラメータ**                               | **必須** | **説明**                                              |
| ------------------------------------------- | ------------ | ------------------------------------------------------------ |
| pulsar_service_url                          | Yes          | Pulsar クラスタに接続するために使用する URL。フォーマット: `"pulsar://ip:port"` または `"pulsar://service:port"`。例: `"pulsar_service_url" = "pulsar://``localhost:6650``"` |
| pulsar_topic                                | Yes          | サブスクライブするトピック。例: "pulsar_topic" = "persistent://tenant/namespace/topic-name" |
| pulsar_subscription                         | Yes          | トピックに対して構成されたサブスクリプション。例: `"pulsar_subscription" = "my_subscription"` |
| pulsar_partitions, pulsar_initial_positions | No           | `pulsar_partitions` : トピック内のサブスクライブするパーティション。`pulsar_initial_positions`: `pulsar_partitions` で指定されたパーティションの初期位置。初期位置は `pulsar_partitions` のパーティションに対応している必要があります。有効な値:`POSITION_EARLIEST` (デフォルト値): サブスクリプションはパーティション内で利用可能なメッセージのうち最も古いメッセージから開始されます。 `POSITION_LATEST`: サブスクリプションはパーティション内で利用可能なメッセージのうち最新のメッセージから開始されます。注意:`pulsar_partitions` が指定されていない場合、トピックのすべてのパーティションがサブスクライブされます。`pulsar_partitions` と `property.pulsar_default_initial_position` の両方が指定されている場合、`pulsar_partitions` の値が `property.pulsar_default_initial_position` の値を上書きします。`pulsar_partitions` も `property.pulsar_default_initial_position` も指定されていない場合、サブスクリプションはパーティション内で利用可能な最新のメッセージから開始されます。例:`"pulsar_partitions" = "my-partition-0,my-partition-1,my-partition-2,my-partition-3", "pulsar_initial_positions" = "POSITION_EARLIEST,POSITION_EARLIEST,POSITION_LATEST,POSITION_LATEST"` |

Routine Load は、Pulsar に対して以下のカスタムパラメータをサポートしています。

| パラメータ                                | 必須 | 説明                                                  |
| ---------------------------------------- | -------- | ------------------------------------------------------------ |
| property.pulsar_default_initial_position | No       | トピックのパーティションがサブスクライブされるときのデフォルトの初期位置。`pulsar_initial_positions` が指定されていない場合にこのパラメータが有効になります。有効な値は `pulsar_initial_positions` の有効な値と同じです。例: `"``property.pulsar_default_initial_position" = "POSITION_EARLIEST"` |
| property.auth.token                      | No       | Pulsar がセキュリティトークンを使用してクライアントの認証を行う場合、トークン文字列が必要です。例: `"p``roperty.auth.token" = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUJzdWIiOiJqaXV0aWFuY2hlbiJ9.lulGngOC72vE70OW54zcbyw7XdKSOxET94WT_hIqD"` |

## ロードジョブとタスクの確認

### ロードジョブの確認

[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation//SHOW_ROUTINE_LOAD.md) ステートメントを実行して、ロードジョブ `routine_wiki_edit_1` のステータスを確認します。StarRocks は、実行状態 `State`、統計情報 (消費された総行数とロードされた総行数) `Statistics`、およびロードジョブの進行状況 `progress` を返します。

Pulsar からデータを消費する Routine Load ジョブを確認する場合、`progress` を除くほとんどの返されるパラメータは、Kafka からデータを消費する場合と同じです。`progress` はバックログを指し、パーティション内の未確認メッセージの数です。

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
DataSourceProperties: {"serviceUrl":"pulsar://localhost:6650","currentPulsarPartitions":"my-partition-0,my-partition-1","topic":"persistent://tenant/namespace/topic-name","subscription":"load-test"}
    CustomProperties: {"auth.token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUJzdWIiOiJqaXV0aWFuY2hlbiJ9.lulGngOC72vE70OW54zcbyw7XdKSOxET94WT_hIqD"}
           Statistic: {"receivedBytes":5480943882,"errorRows":0,"committedTaskNum":696,"loadedRows":66243440,"loadRowsRate":29000,"abortedTaskNum":0,"totalRows":66243440,"unselectedRows":0,"receivedBytesRate":2400000,"taskExecuteTimeMs":2283166}
            Progress: {"my-partition-0(backlog): 100","my-partition-1(backlog): 0"}
ReasonOfStateChanged: 
        ErrorLogUrls: 
            OtherMsg:
1 row in set (0.00 sec)
```

### ロードタスクの確認

[SHOW ROUTINE LOAD TASK](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD_TASK.md) ステートメントを実行して、ロードジョブ `routine_wiki_edit_1` のロードタスクを確認します。実行中のタスクの数、消費される Kafka トピックのパーティションと消費の進行状況 `DataSourceProperties`、および対応する Coordinator BE ノード `BeId` などが表示されます。

```SQL
MySQL [example_db]> SHOW ROUTINE LOAD TASK WHERE JobName = "routine_wiki_edit_1" \G
```

## ロードジョブの変更

ロードジョブを変更する前に、[PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md) ステートメントを使用して一時停止する必要があります。その後、[ALTER ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md) を実行して変更を行い、[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) ステートメントを実行して再開し、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) ステートメントを使用してステータスを確認することができます。

Pulsar からデータを消費する場合、`data_source_properties` を除くほとんどの返されるパラメータは、Kafka からデータを消費する場合と同じです。

次のポイントに注意してください:

- `data_source_properties` に関連するパラメータのうち、`pulsar_partitions`、`pulsar_initial_positions`、およびカスタム Pulsar パラメータ `property.pulsar_default_initial_position` と `property.auth.token` のみが変更可能です。`pulsar_service_url`、`pulsar_topic`、および `pulsar_subscription` のパラメータは変更できません。
- 消費するパーティションと一致する初期位置を変更する場合は、Routine Load ジョブを作成する際に `pulsar_partitions` を指定し、指定したパーティションの初期位置 `pulsar_initial_positions` のみを変更できるようにする必要があります。
- ルーチンロードジョブを作成する際にトピック `pulsar_topic` のみを指定し、パーティション `pulsar_partitions` を指定しない場合、`pulsar_default_initial_position` を使用してトピックのすべてのパーティションの開始位置を変更できます。
