---
displayed_sidebar: English
---

# ルーチン・ロード

## 読み込みパフォーマンスを向上させるにはどうすればいいですか?

**方法 1: 実際のロード タスクの並列性を増やす** ために、ロード ジョブをできるだけ多くの並列ロード タスクに分割します。

> **注意**
>
> この方法は、より多くのCPUリソースを消費し、多数のタブレットバージョンを引き起こす可能性があります。

実際のロード タスクの並列性は、いくつかのパラメータによって構成される次の式で決定され、生存しているBEノードの数または消費されるべきパーティションの数の上限があります。

```Plaintext
min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)
```

パラメータの説明:

- `alive_be_number`: 生存しているBEノードの数。
- `partition_number`: 消費されるべきパーティションの数。
- `desired_concurrent_number`: ルーチン・ロード・ジョブの望ましいロード タスクの並列性。デフォルト値は `3` です。このパラメータに高い値を設定することで、実際のロード タスクの並列性を増やすことができます。
  - ルーチン・ロード・ジョブをまだ作成していない場合、[CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を使用してルーチン・ロード・ジョブを作成する際に、このパラメータを設定する必要があります。
  - すでにルーチン・ロード・ジョブを作成している場合は、[ALTER ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md) を使用してこのパラメータを変更する必要があります。
- `max_routine_load_task_concurrent_num`: ルーチン・ロード・ジョブのデフォルトの最大タスク並列性。デフォルト値は `5` です。このパラメータはFEの動的パラメータです。詳細情報と設定方法については、[パラメータ設定](../../administration/FE_configuration.md#loading-and-unloading)を参照してください。

したがって、消費されるべきパーティションの数と生存しているBEノードの数が他の2つのパラメータよりも多い場合、`desired_concurrent_number` と `max_routine_load_task_concurrent_num` の値を増やすことで、実際のロード タスクの並列性を増やすことができます。

例えば、消費されるべきパーティションの数が `7`、生存しているBEノードの数が `5`、`max_routine_load_task_concurrent_num` がデフォルト値の `5` の場合、ロード タスクの並列性を上限まで増やす必要がある場合は、`desired_concurrent_number` を `5` に設定する必要があります（デフォルト値は `3`）。すると、実際のタスク並列性 `min(5,7,5,5)` は `5` と計算されます。

パラメータの詳細については、[CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

**方法 2: 1つ以上のパーティションからルーチン・ロード・タスクによって消費されるデータ量を増やす。**

> **注意**
>
> この方法は、データの読み込みに遅延が生じる可能性があります。

ルーチン・ロード・タスクが消費できるメッセージの数の上限は、ロード・タスクが消費できるメッセージの最大数を意味するパラメータ `max_routine_load_batch_size` またはメッセージ消費の最大期間を意味するパラメータ `routine_load_task_consume_second` のいずれかによって決定されます。ロード・タスクがいずれかの要件を満たすだけのデータを消費すると、消費は完了します。これら2つのパラメータはFEの動的パラメータです。詳細情報と設定方法については、[パラメータ設定](../../administration/FE_configuration.md#loading-and-unloading)を参照してください。

ロード・タスクによって消費されるデータ量の上限を決定するパラメータは、**be/log/be.INFO** のログを確認することで分析できます。そのパラメータを増やすことで、ロード・タスクによって消費されるデータ量を増やすことができます。

```Plaintext
I0325 20:27:50.410579 15259 data_consumer_group.cpp:131] consumer group done: 41448fb1a0ca59ad-30e34dabfa7e47a0. consume time(ms)=3261, received rows=179190, received bytes=9855450, eos: 1, left_time: -261, left_bytes: 514432550, blocking get time(us): 3065086, blocking put time(us): 24855
```

通常、ログの `left_bytes` フィールドは `0` 以上であり、ロード・タスクによって消費されるデータ量が `max_routine_load_batch_size` または `routine_load_task_consume_second` の範囲内で超えていないことを示します。これは、スケジュールされたロード・タスクのバッチが、消費の遅延なくKafkaからのすべてのデータを消費できることを意味します。このシナリオでは、`routine_load_task_consume_second` の値を大きく設定することで、1つ以上のパーティションからロード・タスクによって消費されるデータ量を増やすことができます。

`left_bytes` フィールドが `0` 未満の場合、ロード・タスクによって消費されたデータ量が `routine_load_task_consume_second` の範囲内で `max_routine_load_batch_size` に達したことを意味します。Kafkaからのデータがスケジュールされたロード・タスクのバッチを満たすたびに、Kafkaに未消費のデータが残り、消費の遅延が発生する可能性が高くなります。この場合、`max_routine_load_batch_size` の値を大きく設定することができます。

## SHOW ROUTINE LOAD の結果が、ロード・ジョブが `PAUSED` 状態にあることを示している場合はどうすればよいですか?

- `ReasonOfStateChanged` フィールドを確認し、エラーメッセージ `Broker: Offset out of range` が報告されているかを確認します。

  **原因分析:** ロード・ジョブのコンシューマーオフセットがKafkaパーティションに存在しません。

  **解決策:** [SHOW ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) を実行し、パラメータ `Progress` でロード・ジョブの最新のコンシューマーオフセットを確認します。次に、対応するメッセージがKafkaパーティションに存在するかどうかを確認します。存在しない場合は、以下の可能性があります。

  - ロード・ジョブの作成時に指定されたコンシューマーオフセットが未来のオフセットである。
  - Kafkaパーティション内の指定されたコンシューマーオフセットのメッセージが、ロード・ジョブによって消費される前に削除されている。読み込み速度に基づいて、適切なKafkaログ保持ポリシーとパラメータ（例：`log.retention.hours` と `log.retention.bytes`）を設定することを推奨します。

- `ReasonOfStateChanged` フィールドを確認し、エラーメッセージ `Broker: Offset out of range` が報告されていないことを確認します。

  **原因分析:** ロード・タスクのエラー行数がしきい値 `max_error_number` を超えています。

  **解決策:** `ReasonOfStateChanged` と `ErrorLogUrls` フィールドのエラーメッセージを使用して問題をトラブルシューティングし、修正します。

  - データソースのデータ形式が不正であることが原因の場合、データ形式を確認し、問題を修正する必要があります。問題を正常に修正した後、[RESUME ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) を使用して、一時停止したロード・ジョブを再開できます。

  - StarRocksがデータソースのデータ形式を解析できない場合は、しきい値 `max_error_number` を調整する必要があります。まず、[SHOW ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) を実行して `max_error_number` の値を確認し、次に [ALTER ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md) を使用してしきい値を増やします。しきい値を変更した後、[RESUME ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) を使用して、一時停止したロード・ジョブを再開できます。

## SHOW ROUTINE LOAD の結果が、ロード・ジョブが `CANCELLED` 状態にあることを示している場合はどうすればよいですか?

  **原因分析:** ロード・ジョブがロード中に例外（例えばテーブルが削除されたなど）に遭遇しました。

  **解決策:** `ReasonOfStateChanged` と `ErrorLogUrls` フィールドのエラーメッセージを参照して問題のトラブルシューティングと修正を行います。ただし、問題を修正した後でも、キャンセルされたロード・ジョブを再開することはできません。

## Routine Loadは、Kafkaからの消費とStarRocksへの書き込み時に一貫性のあるセマンティクスを保証できますか?

   Routine Loadは厳密に一度だけのセマンティクスを保証します。

   各ロード・タスクは個別のトランザクションです。トランザクションの実行中にエラーが発生した場合、トランザクションは中止され、FEはロード・タスクの関連パーティションの消費進行状況を更新しません。FEが次にタスクキューからロード・タスクをスケジュールする際、ロード・タスクはパーティションの最後に保存された消費位置から消費リクエストを送信し、厳密に一度だけのセマンティクスを保証します。
