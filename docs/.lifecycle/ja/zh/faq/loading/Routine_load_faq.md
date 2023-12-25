---
displayed_sidebar: Chinese
---

# Routine Loadのよくある質問

## 1. インポートパフォーマンスを向上させるには？

**方法1**：**実際のタスクの並列度を増やす**ことで、1つのインポートジョブをできるだけ多くのインポートタスクに分割して並行実行します。

> **注意**
>
> この方法はより多くのCPUリソースを消費し、インポートバージョンが多くなる可能性があります。

実際のタスクの並列度は、以下の複数のパラメータによって決定される式で、上限はBEノードの数または消費パーティションの数です。

```Plain
min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)
```

パラメータ説明：

- `alive_be_number`：稼働中のBEノード数。

- `partition_number`：消費パーティション数。

- `desired_concurrent_number`：Routine Loadインポートジョブで、単一のインポートジョブに設定された希望のタスク並列度で、デフォルト値は3です。
  - インポートジョブをまだ作成していない場合は、[CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を実行してインポートジョブを作成する際に、このパラメータを設定する必要があります。
  - すでにインポートジョブを作成している場合は、[ALTER ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md)を実行してこのパラメータを変更する必要があります。

- `max_routine_load_task_concurrent_num`：Routine Loadインポートジョブのデフォルトの最大タスク並列度で、デフォルト値は`5`です。このパラメータはFEの動的パラメータで、詳細な説明と設定方法については、[設定パラメータ](../../administration/FE_configuration.md#インポートとエクスポート)を参照してください。

したがって、消費パーティションとBEノードの数が多く、他の2つのパラメータよりも大きい場合、実際のタスクの並列度を増やす必要がある場合は、以下のパラメータを増やすことができます。

消費パーティション数が`7`、稼働中のBEノード数が`5`、`max_routine_load_task_concurrent_num`がデフォルト値の`5`であると仮定します。この場合、実際のタスクの並列度を上限まで増やすには、`desired_concurrent_number`を`5`（デフォルト値は`3`）に設定する必要があり、実際のタスクの並列度`min(5,7,5,5)`は`5`になります。

より多くのパラメータ説明については、[CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#例)を参照してください。

**方法2**：**単一のインポートタスクが消費するパーティションのデータ量を増やす**ことです。

> **注意**
>
> この方法はデータインポートの遅延を大きくする可能性があります。

単一のRoutine Loadインポートタスクがメッセージを消費する上限は、インポートタスクが最大で消費するメッセージ量`max_routine_load_batch_size`、または最大消費時間`routine_load_task_consume_second`によって決まります。インポートタスクがデータを消費して上記の要件に達した後、消費が完了します。これら2つのパラメータはFEの設定項目で、詳細な説明と設定方法については、[設定パラメータ](../../administration/FE_configuration.md#インポートとエクスポート)を参照してください。

**be/log/be.INFO**のログを確認することで、単一のインポートタスクが消費するデータ量の上限がどのパラメータによって決まるかを分析し、そのパラメータを増やすことで、単一のインポートタスクが消費するデータ量を増やすことができます。

```bash
I0325 20:27:50.410579 15259 data_consumer_group.cpp:131] consumer group done: 41448fb1a0ca59ad-30e34dabfa7e47a0. consume time(ms)=3261, received rows=179190, received bytes=9855450, eos: 1, left_time: -261, left_bytes: 514432550, blocking get time(us): 3065086, blocking put time(us): 24855
```

通常、このログの`left_bytes`フィールドは`0`以上であるべきです。これは、`routine_load_task_consume_second`の時間内に一度に読み取られるデータ量が`max_routine_load_batch_size`を超えていないことを意味し、スケジュールされた一連のインポートタスクがKafkaのデータを完全に消費できることを意味し、消費遅延が存在しないことを意味します。この場合、`routine_load_task_consume_second`を増やすことで、単一のインポートタスクが消費するパーティションのデータ量を増やすことができます。

`left_bytes < 0`の場合、`routine_load_task_consume_second`で定められた時間に達する前に、一度に読み取られるデータ量が`max_routine_load_batch_size`に達していることを意味します。これは、Kafkaのデータが毎回スケジュールされた一連のインポートタスクを満たしていることを意味し、Kafkaにはまだ消費されていないデータが残っている可能性が高いことを意味します。この場合、`max_routine_load_batch_size`を増やすことができます。

## 2. SHOW ROUTINE LOADを実行したときに、インポートジョブが**PAUSED**または**CANCELLED**状態になっている場合、どのようにエラーを調査して修正しますか？

- **エラーメッセージ**：インポートジョブが**PAUSED**状態になり、`ReasonOfStateChanged`で`Broker: Offset out of range`というエラーが発生しています。

  **原因分析**：インポートジョブの消費ポイントがKafkaパーティションに存在しません。

  **解決方法**：SHOW ROUTINE LOADを実行して`Progress`パラメータでインポートジョブの最新の消費ポイントを確認し、Kafkaパーティションでそのポイントのメッセージが存在するかどうかを確認します。存在しない場合は、以下の2つの理由が考えられます：

  - インポートジョブを作成する際に指定した消費ポイントが将来の時点である。
  - Kafkaパーティションのそのポイントのメッセージがインポートジョブによって消費される前に削除されていました。インポートジョブのインポート速度に基づいて、`log.retention.hours`、`log.retention.bytes`などのKafkaログの適切なクリーンアップポリシーとパラメータを設定することをお勧めします。

- **エラーメッセージ**：インポートジョブが**PAUSED**状態になります。

  **原因分析**：インポートタスクのエラー行数が閾値`max_error_number`を超えた可能性があります。

  **解決方法**：`ReasonOfStateChanged`、`ErrorLogUrls`のエラーメッセージに基づいて調査します。

  - データソースのデータ形式に問題がある場合は、データソースのデータ形式を確認して修正する必要があります。修正後、[RESUME ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)を使用して**PAUSED**状態のインポートジョブを再開できます。

  - データソースのデータ形式がStarRocksによって解析できない場合は、エラー行数の閾値`max_error_number`を調整する必要があります。
  まず[SHOW ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)を実行してエラー行数の閾値`max_error_number`を確認し、次に[ALTER ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md)を実行してエラー行数の閾値`max_error_number`を適切に増やします。閾値を変更した後、[RESUME ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md)を使用して**PAUSED**状態のインポートジョブを再開できます。

- **エラーメッセージ**：インポートジョブが**CANCELLED**状態になった場合。

  **原因分析**：インポートタスクの実行中に例外が発生した可能性があります。例えば、テーブルが削除されたなどです。
  
  **解決方法**：`ReasonOfStateChanged`、`ErrorLogUrls`のエラーメッセージを参考にして調査と修正を行います。しかし、修正後も**CANCELLED**状態のインポートジョブを復旧することはできません。

## 3. Routine Loadを使用してKafkaからStarRocksに書き込む際、一貫性のあるセマンティクスを保証できますか？

Routine LoadはExactly-onceセマンティクスを保証できます。

インポートタスクは個別のトランザクションであり、そのトランザクションが実行中にエラーが発生した場合、トランザクションは中止され、FEはそのインポートタスクに関連するパーティションの消費進行状況を更新しません。さらに、FEは次回のタスクスケジュールでキューに入っているインポートタスクを実行する際に、前回保存されたパーティションの消費ポイントから消費リクエストを開始するため、Exactly-onceセマンティクスを保証できます。
