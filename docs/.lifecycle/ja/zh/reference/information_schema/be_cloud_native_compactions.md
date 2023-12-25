---
displayed_sidebar: Chinese
---

# be_cloud_native_compactions

`be_cloud_native_compactions` は、統合ストレージ・コンピューティングクラスターの CN（または v3.0 の BE）で実行される Compaction トランザクションに関する情報を提供します。各 Compaction タスクは Tablet 単位で複数のサブタスクに分割され、ビューの各行は Tablet の Compaction サブタスクに対応しています。

`be_cloud_native_compactions` は以下のフィールドを提供します：

| **フィールド** | **説明**                                                      |
| -------------- | ------------------------------------------------------------- |
| BE_ID          | CN（BE）の ID。                                               |
| TXN_ID         | Compaction タスクに対応するトランザクション ID。同じ Compaction トランザクションには複数のサブタスクがあるため、TXN_ID が同じ場合があります。 |
| TABLET_ID      | Compaction タスクに対応する Tablet ID。                       |
| VERSION        | Compaction タスクの入力データのバージョン。                   |
| SKIPPED        | タスクが実行をスキップしたかどうか。                          |
| RUNS           | タスクの実行回数。1 より大きい場合はリトライが発生したことを意味します。 |
| START_TIME     | 実行開始時間。                                                |
| FINISH_TIME    | 実行終了時間。タスクが実行中の場合、値は NULL。               |
| PROGRESS       | 進行状況のパーセンテージ。範囲は 0 から 100 です。            |
| STATUS         | タスクの実行状態。                                            |

