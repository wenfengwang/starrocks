---
displayed_sidebar: "Japanese"
---

# ルーチンロード

## ロードのパフォーマンスを改善する方法はありますか？

**方法1: 実際のロードタスクの並列性を増やす**ことで、ロードジョブをできるだけ多くの並列ロードタスクに分割します。

> **注意**
>
> この方法は、より多くのCPUリソースを消費し、多くのタブレットバージョンを引き起こす可能性があります。

実際のロードタスクの並列性は、いくつかのパラメータで構成される以下の式によって決定されます。ただし、BEノードの数または消費するパーティションの数の上限があります。

```Plaintext
min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)
```

パラメータの説明:

- `alive_be_number`: BEノードの数。
- `partition_number`: 消費するパーティションの数。
- `desired_concurrent_number`: ルーチンロードジョブの実際のロードタスクの並列性。デフォルト値は `3` です。このパラメータをより高い値に設定すると、実際のロードタスクの並列性を増やすことができます。
  - ルーチンロードジョブを作成していない場合は、[CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を使用してルーチンロードジョブを作成する際にこのパラメータを設定する必要があります。
  - 既にルーチンロードジョブを作成している場合は、[ALTER ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md) を使用してこのパラメータを変更する必要があります。
- `max_routine_load_task_concurrent_num`: ルーチンロードジョブのデフォルトの最大タスク並列性。デフォルト値は `5` です。このパラメータはFEの動的パラメータです。詳細な情報と構成方法については、[パラメータの構成](../../administration/Configuration.md#loading-and-unloading)を参照してください。

したがって、消費するパーティションの数とBEノードの数が他の2つのパラメータよりも大きい場合、`desired_concurrent_number` と `max_routine_load_task_concurrent_num` パラメータの値を増やして、実際のロードタスクの並列性を増やすことができます。

パラメータの詳細については、[CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。

**方法2: 1つ以上のパーティションからのルーチンロードタスクが消費するデータ量を増やす**。

> **注意**
>
> この方法は、データのロードに遅延を引き起こす可能性があります。

ルーチンロードタスクが消費できるメッセージの数の上限は、ロードタスクが消費できるメッセージの最大数を示すパラメータ `max_routine_load_batch_size` またはメッセージの消費時間の最大値を示すパラメータ `routine_load_task_consume_second` のいずれかによって決まります。ロードタスクがこれらの要件のいずれかを満たすだけのデータを消費すると、消費が完了します。これらの2つのパラメータはFEの動的パラメータです。詳細な情報と構成方法については、[パラメータの構成](../../administration/Configuration.md#loading-and-unloading)を参照してください。

ロードタスクが消費するデータ量の上限を増やすためには、**be/log/be.INFO** のログを表示して、どのパラメータがロードタスクが消費するデータ量の上限を決定するかを分析することができます。そのパラメータを増やすことで、ロードタスクが消費するデータ量を増やすことができます。

```Plaintext
I0325 20:27:50.410579 15259 data_consumer_group.cpp:131] consumer group done: 41448fb1a0ca59ad-30e34dabfa7e47a0. consume time(ms)=3261, received rows=179190, received bytes=9855450, eos: 1, left_time: -261, left_bytes: 514432550, blocking get time(us): 3065086, blocking put time(us): 24855
```

通常、ログの `left_bytes` フィールドが `0` 以上である場合、ロードタスクが消費するデータ量が `routine_load_task_consume_second` 内で `max_routine_load_batch_size` を超えていないことを示しています。これは、スケジュールされたロードタスクのバッチがKafkaからデータを遅延なく消費できることを意味します。このシナリオでは、`routine_load_task_consume_second` の値を大きく設定して、1つ以上のパーティションからロードタスクが消費するデータ量を増やすことができます。

`left_bytes` フィールドが `0` 未満の場合、ロードタスクが `routine_load_task_consume_second` 内で `max_routine_load_batch_size` に達したことを意味します。Kafkaからのデータがスケジュールされたロードタスクのバッチを埋めるたびに、まだ消費されていない残りのデータがある可能性が非常に高いため、消費が遅延しています。この場合、`max_routine_load_batch_size` の値を大きく設定することができます。

## SHOW ROUTINE LOAD の結果が `PAUSED` 状態である場合、どうすればよいですか？

- `ReasonOfStateChanged` フィールドを確認し、エラーメッセージ `Broker: Offset out of range` が報告されている場合は、次の手順を実行します。

  **原因の分析:** ロードジョブのコンシューマオフセットがKafkaパーティションに存在しない。

  **解決策:** [SHOW ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) を実行し、パラメータ `Progress` の中にあるロードジョブの最新のコンシューマオフセットを確認します。その後、Kafkaパーティションに対応するメッセージが存在するかどうかを確認します。存在しない場合は、次のいずれかの理由が考えられます。

  - ロードジョブが作成された際に指定されたコンシューマオフセットが未来のオフセットである。
  - ロードジョブによって消費される前に、Kafkaパーティションの指定されたコンシューマオフセットのメッセージが削除されました。ロード速度に基づいて、`log.retention.hours` および `log.retention.bytes` などの適切なKafkaログクリーニングポリシーとパラメータを設定することをお勧めします。

- `ReasonOfStateChanged` フィールドを確認し、エラーメッセージ `Broker: Offset out of range` が報告されていない場合は、次の手順を実行します。

  **原因の分析:** ロードタスクのエラーレコードの数がしきい値 `max_error_number` を超えています。

  **解決策:** `ReasonOfStateChanged` および `ErrorLogUrls` のエラーメッセージを使用して、問題をトラブルシューティングおよび修正します。

  - データソースのデータ形式が正しくない場合は、データ形式を確認し、問題を修正する必要があります。問題を正常に修正した後、[RESUME ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) を使用して一時停止したロードジョブを再開することができます。

  - StarRocksがデータソースのデータ形式を解析できない場合は、しきい値 `max_error_number` を調整する必要があります。まず、[SHOW ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) を実行して `max_error_number` の値を確認し、その後、[ALTER ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md) を使用してしきい値を増やします。しきい値を変更した後、[RESUME ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) を使用して一時停止したロードジョブを再開することができます。

## SHOW ROUTINE LOAD の結果が `CANCELLED` 状態である場合、どうすればよいですか？

  **原因の分析:** ロードジョブのロード中に例外が発生し、テーブルが削除されたなどの問題が発生しました。

  **解決策:** 問題をトラブルシューティングおよび修正する際には、`ReasonOfStateChanged` および `ErrorLogUrls` のエラーメッセージを参照することができます。ただし、問題を修正した後は、キャンセルされたロードジョブを再開することはできません。

## ルーチンロードは、Kafkaから消費してStarRocksに書き込む際に一貫性のセマンティクスを保証できますか？

   ルーチンロードは、正確に一度だけのセマンティクスを保証します。

   各ロードタスクは個別のトランザクションです。トランザクションの実行中にエラーが発生した場合、トランザクションは中止され、FEは関連するロードタスクの消費進捗を更新しません。FEが次回タスクキューからロードタスクをスケジュールする際に、ロードタスクはパーティションの最後に保存された消費位置から消費リクエストを送信し、正確に一度だけのセマンティクスを保証します。
