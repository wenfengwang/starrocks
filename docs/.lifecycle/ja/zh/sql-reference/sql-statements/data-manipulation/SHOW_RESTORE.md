---
displayed_sidebar: Chinese
---

# SHOW RESTORE の表示

## 機能

指定されたデータベースの復元タスクを表示します。詳細については、[バックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

> **注記**
>
> StarRocks では、最新の復元タスク情報のみが保存されます。

## 構文

```SQL
SHOW RESTORE [FROM <db_name>]
```

## パラメータ説明

| **パラメータ** | **説明**               |
| -------------- | ---------------------- |
| db_name        | 復元タスクが属するデータベース名。 |

## 戻り値

| **戻り値**             | **説明**                                                     |
| ---------------------- | ------------------------------------------------------------ |
| JobId                  | この復元ジョブのID。                                          |
| Label                  | 復元するバックアップの名前。                                  |
| Timestamp              | バックアップのタイムスタンプ。                                |
| DbName                 | 復元ジョブに対応するデータベース。                            |
| State                  | 復元ジョブの現在のステージ：<ul><li>PENDING：ジョブの初期状態。</li><li>SNAPSHOTING：ローカルで新しいテーブルのスナップショットを取る作業中。</li><li>DOWNLOAD：スナップショットのダウンロードタスクを送信中。</li><li>DOWNLOADING：スナップショットがダウンロード中。</li><li>COMMIT：ダウンロードしたスナップショットを適用する準備中。</li><li>COMMITTING：ダウンロードしたスナップショットを適用中。</li><li>FINISHED：復元完了。</li><li>CANCELLED：復元失敗またはキャンセルされた。</li></ul> |
| AllowLoad              | 復元中にインポートを許可するかどうか。                        |
| ReplicationNum         | 復元するレプリカの数。                                        |
| RestoreObjs            | 復元されるオブジェクト（テーブルとパーティション）。          |
| CreateTime             | ジョブの作成時間。                                            |
| MetaPreparedTime       | ローカルのメタデータが生成された時間。                        |
| SnapshotFinishedTime   | スナップショットが完了した時間。                              |
| DownloadFinishedTime   | リモートスナップショットのダウンロードが完了した時間。        |
| FinishedTime           | このジョブが完了した時間。                                    |
| UnfinishedTasks        | SNAPSHOTTING、DOWNLOADING、COMMITTINGなどのステージで、複数のサブタスクが同時に進行している場合、ここには現在のステージで未完了のサブタスクのTask IDが表示されます。 |
| Progress               | スナップショットのダウンロード進捗。                          |
| TaskErrMsg             | サブタスクの実行にエラーがあった場合、ここに該当するサブタスクのエラーメッセージが表示されます。 |
| Status                 | ジョブの全プロセスで発生する可能性のあるいくつかのステータス情報を記録するために使用されます。 |
| Timeout                | ジョブのタイムアウト時間（秒単位）。                          |

## 例

例1：データベース `example_db` の最後の復元タスクを表示します。

```SQL
SHOW RESTORE FROM example_db;
```
