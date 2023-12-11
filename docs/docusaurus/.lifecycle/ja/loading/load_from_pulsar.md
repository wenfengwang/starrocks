---
displayed_sidebar: "Japanese"
---

# [プレビュー] Apache® Pulsar™ からのデータの連続ロード

StarRocks バージョン 2.5 から、Routine Load は Apache® Pulsar™ からのデータの連続ロードをサポートしています。Pulsar は、ストア・コンピュート分離アーキテクチャを持つ分散型のオープンソースのパブサブメッセージングおよびストリーミングプラットフォームです。Routine Load を使用して Pulsar からデータをロードする方法は、Apache Kafka からデータをロードする方法と似ています。このトピックでは、CSV 形式のデータを例に挙げ、Apache Pulsar から Routine Load を使用してデータをロードする方法を紹介します。

## サポートされているデータファイルフォーマット

Routine Load は、Pulsar クラスタから CSV および JSON フォーマットのデータを消費することをサポートしています。

> 注記
>
> CSV 形式のデータに関して、StarRocks は、50 バイト以内の UTF-8 エンコードされた文字列をカラムの区切り文字としてサポートしています。一般的に使用される区切り文字には、カンマ(,)、タブ、およびパイプ(|) が含まれます。

## Pulsar 関連の概念

**[トピック](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#topics)**

Pulsar のトピックは、プロデューサーからコンシューマーへメッセージを転送するための名前付きのチャンネルです。Pulsar のトピックは、パーティション化されたトピックと非パーティション化されたトピックに分けられます。

- **[パーティション化されたトピック](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#partitioned-topics)** は、複数のブローカーによって処理される特別なタイプのトピックであり、これによりスループットが高くなります。パーティション化されたトピックは実際には N 個の内部トピックとして実装されます。ここで、N はパーティションの数です。
- **非パーティション化されたトピック** は、単一のブローカーによってのみ提供される通常のタイプのトピックであり、それによりトピックの最大スループットが制限されます。

**[メッセージ ID](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#messages)**

メッセージのメッセージ ID は、メッセージが永続的に保存されるとすぐに [BookKeeper インスタンス](https://pulsar.apache.org/docs/2.10.x/concepts-architecture-overview/#apache-bookkeeper) によって割り当てられます。メッセージ ID は、メッセージが特定の位置にあることを示し、Pulsar クラスタ内で一意です。

Pulsar は、コンシューマーが consumer.*seek*(*messageId*) を使用して初期位置を指定することをサポートしています。ただし、Kafka コンシューマーオフセットが長整数値であるのに対し、メッセージ ID は「ledgerId:entryID:partition-index:batch-index」という 4 つのパートで構成されています。

そのため、メッセージ ID をメッセージから直接取得することはできません。その結果、現在のところ、**Routine Load は Pulsar からデータをロードする際に初期位置の指定をサポートしておらず、パーティションの先頭または末尾からのデータ消費のみをサポートしています。**

**[サブスクリプション](https://pulsar.apache.org/docs/2.10.x/concepts-messaging/#subscriptions)**

サブスクリプションは、メッセージがコンシューマーに配信される方法を決定する名前付きの構成ルールです。Pulsar はまた、コンシューマーが複数のトピックに同時にサブスクライブすることをサポートしています。1 つのトピックには複数のサブスクリプションが存在できます。

サブスクリプションのタイプは、コンシューマーがそれに接続する際に定義され、そのタイプは異なる構成で再起動することで変更できます。Pulsar では、4 つのサブスクリプションタイプが利用可能です。

- `exclusive` (デフォルト)*:* 単一のコンシューマーのみがサブスクリプションにアタッチできます。メッセージの消費は 1 人の顧客のみが許可されます。
- `shared`：複数のコンシューマーが同じサブスクリプションにアタッチできます。メッセージはラウンドロビン分配でコンシューマーに配信され、特定のメッセージは 1 人のコンシューマーにのみ配信されます。
- `failover`：複数のコンシューマーが同じサブスクリプションにアタッチできます。非パーティション化されたトピックの場合、マスターコンシューマーが選択され、メッセージはマスターコンシューマーに配信されます。マスターコンシューマーが切断されると、(未確認および以降の)メッセージは次のコンシューマーに配信されます。
- `key_shared`：複数のコンシューマーが同じサブスクリプションにアタッチできます。メッセージはコンシューマー間で分配され、同じキーまたは同じ順序キーを持つメッセージは 1 人のコンシューマーにのみ配信されます。

> 注記：
>
> 現在、Routine Load は exclusive タイプを使用しています。

## Routine Load ジョブの作成

次の例では、Pulsar で CSV 形式のメッセージを消費し、Routine Load ジョブを作成して StarRocks にデータをロードする方法について説明します。詳しい手順と参照については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。

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

Pulsar からデータを消費するように Routine Load を作成する場合、`data_source_properties` を除くほとんどの入力パラメータは、Kafka からデータを消費する場合と同じです。`data_source_properties` を除くパラメータについての説明は、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。

`data_source_properties` に関連するパラメータとその説明は次の通りです：

| **パラメータ**                       | **必須** | **説明**                                                      |
| ----------------------------------- | -------- | ------------------------------------------------------------ |
| pulsar_service_url                  | Yes      | Pulsar クラスタに接続するために使用される URL です。書式： `"pulsar://ip:port"` または `"pulsar://service:port"` 。例： `"pulsar_service_url" = "pulsar://``localhost:6650``"` |
| pulsar_topic                        | Yes      | 購読されるトピックです。例： "pulsar_topic" = "persistent://tenant/namespace/topic-name" |
| pulsar_subscription                 | Yes      | トピックに対して構成されたサブスクリプションです。例： `"pulsar_subscription" = "my_subscription"` |
| pulsar_partitions, pulsar_initial_positions | No       | `pulsar_partitions`：トピック内の購読されるパーティションです。 `pulsar_initial_positions`：`pulsar_partitions` によって指定されたパーティションの初期位置。 初期位置は、 `pulsar_partitions` のパーティションに対応する必要があります。有効な値： `POSITION_EARLIEST`（デフォルト値）：パーティション内で最も古い利用可能なメッセージから購読を開始します。 `POSITION_LATEST`：パーティション内で最新の利用可能なメッセージから購読を開始します。 注記： `pulsar_partitions` が指定されていない場合、トピックのすべてのパーティションが購読されます。`pulsar_partitions` および `property.pulsar_default_initial_position` の両方が指定されている場合、`pulsar_partitions` の値が `property.pulsar_default_initial_position` の値を上書きします。`pulsar_partitions` および `property.pulsar_default_initial_position` がどちらも指定されていない場合、パーティション内で最新の利用可能なメッセージから購読が開始されます。例：`"pulsar_partitions" = "my-partition-0,my-partition-1,my-partition-2,my-partition-3", "pulsar_initial_positions" = "POSITION_EARLIEST,POSITION_EARLIEST,POSITION_LATEST,POSITION_LATEST"` |

Routine Load は、Pulsar のために以下のカスタムパラメータをサポートしています。

| パラメータ                            | 必須     | 説明                                                         |
| -------------------------------------- | -------- | ------------------------------------------------------------ |
| property.pulsar_default_initial_position | No       | トピックのパーティションが購読される際のデフォルトの初期位置です。`pulsar_initial_positions` が指定されていない場合、このパラメータが有効になります。有効な値は、`pulsar_initial_positions` の有効な値と同じです。例：`"``property.pulsar_default_initial_position" = "POSITION_EARLIEST"` |
| property.auth.token                    | No       | Pulsar がセキュリティトークンを使用してクライアントの認証を有効にしている場合、トークン文字列が必要です。 例：`"p``roperty.auth.token" = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUJzdWIiOiJqaXV0aWFuY2hlbiJ9.lulGngOC72vE70OW54zcbyw7XdKSOxET94WT_hIqD"` |

## ロード ジョブおよびタスクを確認する

### ロード ジョブを確認する

[SHOW ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation//SHOW_ROUTINE_LOAD.md) ステートメントを実行して、ロード ジョブ `routine_wiki_edit_1` の状態を確認します。StarRocks は、実行状態 `State`、統計情報（消費された合計行数およびロードされた合計行数） `Statistics`、およびロード ジョブの進行状況 `progress` を返します。
```plaintext
SHOW ROUTINE LOAD for routine_wiki_edit_1 \G
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

```sql
SHOW ROUTINE LOAD TASK WHERE JobName = "routine_wiki_edit_1" \G
```

PAUSE ROUTINE LOAD statementを使ってルーティンロードジョブを停止してから、ALTER ROUTINE LOADステートメントを実行して、変更を加えます。変更後は、RESUME ROUTINE LOADステートメントを実行して再開し、SHOW ROUTINE LOADステートメントを使用してそのステータスを確認します。

Pulsarからデータを消費するとき、`data_source_properties`以外のほとんどの返されるパラメーターは、Kafkaからデータを消費する場合と同じです。

次のポイントに注意してください：
- `data_source_properties`関連のパラメーターの中で、現在変更がサポートされているのは、`pulsar_partitions`、`pulsar_initial_positions`、およびカスタムPulsarパラメーターの`property.pulsar_default_initial_position`と`property.auth.token`のみです。パラメーター`pulsar_service_url`、`pulsar_topic`、`pulsar_subscription`は変更できません。
- 消費されるパーティションと一致する初期位置を変更する必要がある場合は、ルーティンロードジョブを作成する際に`pulsar_partitions`を使用してパーティションを指定し、指定されたパーティションの初期位置`pulsar_initial_positions`のみを変更できるようにする必要があります。
- ルーティンロードジョブを作成する際にトピック`pulsar_topic`のみを指定し、パーティション`pulsar_partitions`を指定しない場合、すべてのトピック内のパーティションの開始位置を`pulsar_default_initial_position`を介して変更できます。