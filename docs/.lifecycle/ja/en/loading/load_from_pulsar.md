---
displayed_sidebar: English
---

# [プレビュー] Apache® Pulsar™ からデータを継続的にロード

StarRocks バージョン 2.5 以降、Routine Load は Apache® Pulsar™ からデータを継続的にロードすることをサポートしています。Pulsar は、ストアとコンピュートの分離アーキテクチャを備えた分散型のオープンソースの pub/sub メッセージングおよびストリーミングプラットフォームです。Routine Load による Pulsar からのデータのロードは、Apache Kafka からのデータのロードと似ています。このトピックでは、CSV 形式のデータを例に、Routine Load を介して Apache Pulsar からデータをロードする方法を紹介します。

## サポートされているデータファイル形式

Routine Load は、Pulsar クラスタから CSV および JSON 形式のデータを消費することをサポートしています。

> 注意
>
> CSV 形式のデータについては、StarRocks は 50 バイト以内の UTF-8 エンコード文字列を列区切り文字としてサポートしています。一般的に使用される列区切り文字には、コンマ (,)、タブ、パイプ (|) などがあります。

## Pulsar 関連の概念

**[トピック](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#topics)**
  
Pulsar のトピックは、プロデューサからコンシューマにメッセージを送信するための名前付きチャンネルです。Pulsar のトピックは、パーティション化されたトピックと非パーティション化トピックに分かれています。

- **[パーティション化トピック](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#partitioned-topics)** は、複数のブローカーによって処理される特殊なタイプのトピックであり、それによって高いスループットが可能になります。パーティション化トピックは、実際には N 個の内部トピックとして実装されており、N はパーティションの数です。
- **非パーティション化トピック** は、単一のブローカーによってのみ処理される通常のタイプのトピックであり、トピックの最大スループットが制限されます。

**[メッセージ ID](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#messages)**

メッセージのメッセージ ID は、メッセージが永続的に保存されるとすぐに [BookKeeper インスタンス](https://pulsar.apache.org/docs/2.10.x/concepts-architecture-overview/#apache-bookkeeper)によって割り当てられます。メッセージ ID は、台帳内のメッセージの特定の位置を示し、Pulsar クラスタ内で一意です。

Pulsar は、コンシューマが consumer.*seek*(*messageId*) を使用して初期位置を指定することをサポートしています。しかし、Kafka コンシューマオフセットが長整数値であるのに対し、メッセージ ID は `ledgerId:entryID:partition-index:batch-index` の 4 つの部分で構成されています。

そのため、メッセージから直接メッセージ ID を取得することはできません。その結果、現在のところ、**Routine Load は Pulsar からデータをロードする際の初期位置の指定をサポートしておらず、パーティションの先頭または末尾からのデータの消費のみをサポートしています。**

**[サブスクリプション](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#subscriptions)**

サブスクリプションは、コンシューマへのメッセージの配信方法を決定する名前付きの構成ルールです。Pulsar は、コンシューマが複数のトピックを同時に購読することもサポートしています。トピックは複数のサブスクリプションを持つことができます。

サブスクリプションのタイプは、コンシューマが接続するときに定義され、すべてのコンシューマを異なる構成で再起動することでタイプを変更できます。Pulsar では、以下の 4 つのサブスクリプションタイプが利用可能です：

- `exclusive`（デフォルト）：サブスクリプションにアタッチできるコンシューマは 1 つだけです。メッセージを消費できるのは 1 人のコンシューマだけです。
- `shared`：複数のコンシューマが同じサブスクリプションにアタッチできます。メッセージはコンシューマ間でラウンドロビン方式で配信され、特定のメッセージは 1 つのコンシューマにのみ配信されます。
- `failover`：複数のコンシューマが同じサブスクリプションにアタッチできます。マスターコンシューマが非パーティション化トピックまたはパーティション化トピックの各パーティションに対して選択され、メッセージを受信します。マスターコンシューマが切断されると、すべてのメッセージ（未確認のメッセージおよびそれ以降のメッセージ）が次のコンシューマに配信されます。
- `key_shared`：複数のコンシューマが同じサブスクリプションにアタッチできます。メッセージはコンシューマ間で配信され、同じキーまたは同じ順序キーを持つメッセージは 1 つのコンシューマにのみ配信されます。

> 注意：
>
> 現在、Routine Load は `exclusive` タイプを使用しています。

## Routine Load ジョブの作成

以下の例では、Pulsar で CSV 形式のメッセージを消費し、Routine Load ジョブを作成して StarRocks にデータをロードする方法を説明します。詳細な手順と参照については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。

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

Pulsar からデータを消費するために Routine Load を作成する場合、`data_source_properties` を除くほとんどの入力パラメータは Kafka からデータを消費する場合と同じです。`data_source_properties` 以外のパラメータについては、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。

`data_source_properties` に関連するパラメータとその説明は次のとおりです：

| **パラメータ**                               | **必須** | **説明**                                              |
| ------------------------------------------- | ------------ | ------------------------------------------------------------ |

| pulsar_service_url                          | はい          | Pulsar クラスターへの接続に使用される URL。 フォーマット: `"pulsar://ip:port"` または `"pulsar://service:port"`。例： `"pulsar_service_url" = "pulsar://localhost:6650"` |
| pulsar_topic                                | はい          | サブスクライブされたトピック。例: `"pulsar_topic" = "persistent://tenant/namespace/topic-name"` |
| pulsar_subscription                         | はい          | トピックに対して構成されたサブスクリプション。例： `"pulsar_subscription" = "my_subscription"` |
| pulsar_partitions, pulsar_initial_positions | いいえ           | `pulsar_partitions`: トピック内のサブスクライブされたパーティション。`pulsar_initial_positions`: `pulsar_partitions` で指定されたパーティションの初期位置。初期位置は `pulsar_partitions` のパーティションに対応している必要があります。有効な値: `POSITION_EARLIEST` (既定値): サブスクリプションは、パーティション内で利用可能な最も古いメッセージから開始されます。 `POSITION_LATEST`: サブスクリプションは、パーティション内で利用可能な最新のメッセージから開始されます。注: `pulsar_partitions` が指定されていない場合、トピックのすべてのパーティションがサブスクライブされます。`pulsar_partitions` と `property.pulsar_default_initial_position` の両方が指定されている場合、`pulsar_partitions` の値が `property.pulsar_default_initial_position` の値を上書きします。`pulsar_partitions` と `property.pulsar_default_initial_position` のどちらも指定されていない場合、サブスクリプションはパーティション内で利用可能な最新のメッセージから開始されます。例：`"pulsar_partitions" = "my-partition-0,my-partition-1,my-partition-2,my-partition-3", "pulsar_initial_positions" = "POSITION_EARLIEST,POSITION_EARLIEST,POSITION_LATEST,POSITION_LATEST"` |

Routine Load は、Pulsar に対する以下のカスタムパラメータをサポートしています。

| パラメーター                                  | 必須 | 説明                                                  |
| ---------------------------------------- | -------- | ------------------------------------------------------------ |
| property.pulsar_default_initial_position | いいえ       | トピックのパーティションがサブスクライブされたときのデフォルトの初期位置。このパラメーターは、`pulsar_initial_positions` が指定されていない場合に有効です。その有効な値は `pulsar_initial_positions` の有効な値と同じです。例： `"property.pulsar_default_initial_position" = "POSITION_EARLIEST"` |
| property.auth.token                      | いいえ       | Pulsar がセキュリティトークンを使用してクライアントの認証を有効にしている場合、身元を検証するためのトークン文字列が必要です。例： `"property.auth.token" = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUJzdWIiOiJqaXV0aWFuY2hlbiJ9.lulGngOC72vE70OW54zcbyw7XdKSOxET94WT_hIqD"` |

## ロードジョブとタスクの確認

### ロードジョブの確認

[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) ステートメントを実行して、ロードジョブ `routine_wiki_edit_1` のステータスを確認します。StarRocks は、実行状態 `State`、統計情報（消費された合計行数とロードされた合計行数を含む）`Statistics`、およびロードジョブの進行状況 `progress` を返します。

Pulsar からデータを消費する Routine Load ジョブをチェックする場合、`progress` を除くほとんどの返されるパラメータは Kafka からのデータ消費と同じです。`progress` はバックログ、つまりパーティション内の未確認メッセージの数を指します。

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

[SHOW ROUTINE LOAD TASK](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD_TASK.md) ステートメントを実行して、ロードジョブ `routine_wiki_edit_1` のロードタスクを確認します。実行中のタスクの数、消費されている Kafka トピックパーティション、消費の進行状況 `DataSourceProperties`、および対応するコーディネーター BE ノード `BeId` などが表示されます。

```SQL
MySQL [example_db]> SHOW ROUTINE LOAD TASK WHERE JobName = "routine_wiki_edit_1" \G
```

## ロードジョブの変更

ロードジョブを変更する前に、[PAUSE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/PAUSE_ROUTINE_LOAD.md) ステートメントを使用してジョブを一時停止する必要があります。その後、[ALTER ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md) ステートメントを実行できます。変更後、[RESUME ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) ステートメントを実行してジョブを再開し、[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) ステートメントを使用してその状態を確認できます。

Routine Load を使用して Pulsar からデータを消費する場合、`data_source_properties` を除くほとんどの返されるパラメータは Kafka からのデータ消費と同じです。

**次の点に注意してください**:

- `data_source_properties` 関連のパラメータのうち、`pulsar_partitions`、`pulsar_initial_positions`、およびカスタム Pulsar パラメータ `property.pulsar_default_initial_position` と `property.auth.token` のみが変更をサポートされています。パラメータ `pulsar_service_url`、`pulsar_topic`、および `pulsar_subscription` は変更できません。
- 消費するパーティションと一致する初期位置を変更する必要がある場合、Routine Load ジョブを作成するときに `pulsar_partitions` を使用してパーティションを指定し、指定されたパーティションの初期位置 `pulsar_initial_positions` のみを変更できるようにする必要があります。
- Routine Load ジョブを作成する際に `pulsar_topic` のみを指定し、`pulsar_partitions` は指定しない場合、`pulsar_default_initial_position` を使用してトピック内のすべてのパーティションの開始位置を変更することができます。
