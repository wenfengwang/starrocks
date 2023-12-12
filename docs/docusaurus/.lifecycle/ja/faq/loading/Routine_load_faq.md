---
displayed_sidebar: "Japanese"
---

# ルーチンロード

## 読み込みパフォーマンスを向上させる方法

**方法1: 実際の読み込みタスクの並列性を高める** 読み込みジョブをできるだけ多くの並列読み込みタスクに分割することで、実際の読み込みタスクの並列性を高める。

> **注意**
>
> この方法はより多くのCPUリソースを消費し、多くのタブレットバージョンを引き起こす可能性があります。

実際の読み込みタスクの並列性は、いくつかのパラメータで構成された以下の式によって決定されます。BEノードが生きている数、または消費するパーティションの数の上限である。

```Plaintext
min(alive_be_number, partition_number, desired_concurrent_number, max_routine_load_task_concurrent_num)
```

パラメータの説明:

- `alive_be_number`: 生きているBEノードの数。
- `partition_number`: 消費するパーティションの数。
- `desired_concurrent_number`: ルーチンロードジョブのための希望の読み込みタスクの並列性。デフォルト値は `3` です。このパラメータの値を高く設定することで、実際の読み込みタスクの並列性を高めることができます。
  - ルーチンロードジョブを作成していない場合、[CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を使用してこのパラメータを設定する必要があります。
  - 既にルーチンロードジョブを作成している場合、[ALTER ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md) を使用してこのパラメータを変更する必要があります。
- `max_routine_load_task_concurrent_num`: ルーチンロードジョブのためのデフォルトの最大タスク並列性。デフォルト値は `5` です。このパラメータはFE動的パラメータです。詳細と構成方法については、[パラメータ構成](../../administration/Configuration.md#loading-and-unloading) を参照してください。

したがって、消費するパーティションの数と生きているBEノードの数が他の2つのパラメータよりも大きい場合、`desired_concurrent_number` および `max_routine_load_task_concurrent_num` パラメータの値を増やすことで、実際の読み込みタスクの並列性を高めることができます。

例えば、消費するパーティションの数が `7`、生きているBEノードの数が `5`、`max_routine_load_task_concurrent_num` がデフォルト値である `5` の場合、読み込みタスクの並列性を上限まで高める必要がある場合は、`desired_concurrent_number` を `5` に設定する必要があります（デフォルト値は `3` です）。その後、実際のタスクの並列性 `min(5,7,5,5)` は `5` に計算されます。

より詳細なパラメータの説明については、[CREATE ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。

**方法2: ルーチンロードタスクが消費するデータ量を増やす。**

> **注意**
>
> この方法はデータの読み込みに遅延を引き起こす可能性があります。

ルーチンロードタスクが消費できるメッセージ数の上限は、読み込みタスクが消費できるメッセージ数の最大数を意味するパラメータ`max_routine_load_batch_size`またはメッセージ消費の最大期間を意味するパラメータ`routine_load_task_consume_second`のいずれかによって決定されます。読み込みタスクがどちらかの条件に合致するデータを消費した時点で、消費が完了します。これらの2つのパラメータはFE動的パラメータです。詳細と構成方法については、[パラメータ構成](../../administration/Configuration.md#loading-and-unloading) を参照してください。

読み込みタスクが消費するデータ量の上限を決定するパラメータを分析するには、**be/log/be.INFO** のログを確認することができます。そのパラメータを増やすことで、読み込みタスクが消費するデータ量を増やすことができます。

```Plaintext
I0325 20:27:50.410579 15259 data_consumer_group.cpp:131] consumer group done: 41448fb1a0ca59ad-30e34dabfa7e47a0. consume time(ms)=3261, received rows=179190, received bytes=9855450, eos: 1, left_time: -261, left_bytes: 514432550, blocking get time(us): 3065086, blocking put time(us): 24855
```

通常、ログの `left_bytes` フィールドが `0` 以上である場合、読み込みタスクが`routine_load_task_consume_second`内で`max_routine_load_batch_size`を超えるデータを消費していないことを示します。これは、スケジュールされた読み込みタスクのバッチが遅延なくKafkaからすべてのデータを消費できることを意味します。この場合、`routine_load_task_consume_second` の値を大きく設定して、1つまたは複数のパーティションから読み込みタスクが消費するデータ量を増やすことができます。

`left_bytes` フィールドが `0` 未満の場合は、読み込みタスクが`routine_load_task_consume_second`内で`max_routine_load_batch_size`に達したことを意味します。Kafkaからのデータが毎回読み込みタスクのバッチを満たします。したがって、Kafkaに未消費のデータが残っている可能性が非常に高く、これにより消費の遅延が発生します。この場合、`max_routine_load_batch_size` の値を大きく設定することができます。

## SHOW ROUTINE LOADの結果が`PAUSED`状態であることを示す場合の対処方法は？

- `ReasonOfStateChanged` フィールドを確認し、エラーメッセージとして`Broker: Offset out of range`が報告されている場合

  **原因分析:** ロードジョブのコンシューマオフセットがKafkaパーティションに存在しません。

  **解決方法:** [SHOW ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md) を実行して、パラメータ`Progress`にロードジョブの最新のコンシューマオフセットをチェックし、対応するメッセージがKafkaパーティションに存在するかどうかを確認することができます。存在しない場合、次のいずれかの理由が考えられます。

  - ロードジョブが作成された際に指定されたコンシューマオフセットが未来のオフセットである。
  - ロードジョブによる消費前に、Kafkaパーティションの指定されたコンシューマオフセットのメッセージが削除された。読み込み速度に基づいて、適切なKafkaログのクリーニングポリシーやパラメータ（たとえば、`log.retention.hours` および `log.retention.bytes`）を設定することをお勧めします。

- `ReasonOfStateChanged` フィールドを確認し、エラーメッセージが`Broker: Offset out of range`と報告されていない場合

  **原因分析:** ロードタスクのエラー行数がしきい値 `max_error_number` を超えています。

  **解決方法:** `ReasonOfStateChanged` および `ErrorLogUrls` フィールドのエラーメッセージを使用して問題をトラブルシューティングおよび修正することができます。

  - データソースのデータ形式が間違っている場合、データ形式を確認して問題を修正する必要があります。問題を正常に修正した後、[RESUME ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) を使用して一時停止中のロードジョブを再開することができます。

  - StarRocksがデータソースのデータ形式を解析できない場合、しきい値 `max_error_number` を調整する必要があります。まず、[SHOW ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD.md)を実行して`max_error_number`の値を確認し、その後、しきい値を増やすために[ALTER ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/ALTER_ROUTINE_LOAD.md)を使用できます。しきい値を変更した後、[RESUME ROUTINE LOAD](../../sql-reference/sql-statements/data-manipulation/RESUME_ROUTINE_LOAD.md) を使用して一時停止中のロードジョブを再開することができます。

## SHOW ROUTINE LOADの結果が`CANCELLED`状態であることを示す場合、どのようにすればよいですか？

  **原因分析:** ロードジョブがロード中に例外に遭遇し、例えばテーブルが削除されました。

  **解決方法:** 問題をトラブルシューティングおよび修正する際は、`ReasonOfStateChanged` および `ErrorLogUrls` フィールドのエラーメッセージを参照することができます。ただし、問題を修正した後は、キャンセルされたロードジョブを再開することはできません。

## ルーチンロードは、KafkaからStarRocksへの書き込み時に一貫性のセマンティクスを保証できますか？

   ルーチンロードは正確に一度だけのセマンティクスを保証します。

   各読み込みタスクは個々のトランザクションです。トランザクションの実行中にエラーが発生した場合、トランザクションは中止され、FEはロードタスクの関連パーティションの消費進行状況を更新しません。次回、FEがタスクキューから読み込みタスクをスケジュールする際、読み込みタスクはパーティションの最後に保存された消費位置から消費リクエストを送信するため、正確に一度だけのセマンティクスが保証されます。