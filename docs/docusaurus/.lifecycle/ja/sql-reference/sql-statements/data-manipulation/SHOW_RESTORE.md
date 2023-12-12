---
displayed_sidebar: "Japanese"
---

# RESTOREの表示

## 説明

指定されたデータベースの最後のRESTOREタスクを表示します。詳細については、[データバックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

> **注**
>
> StarRocksでは、最後のRESTOREタスクの情報のみが保存されます。

## 構文

```SQL
SHOW RESTORE [FROM <db_name>]
```

## パラメータ

| **パラメータ** | **説明**                                      |
| ------------- | -------------------------------------------- |
| db_name       | RESTOREタスクが属するデータベース名。          |

## 戻り値

| **戻り値**           | **説明**                                              |
| -------------------- | ---------------------------------------------------- |
| JobId                | ユニークなジョブID。                                      |
| Label                | データスナップショットの名前。                           |
| Timestamp            | バックアップのタイムスタンプ。                             |
| DbName               | RESTOREタスクが属するデータベース名。                     |
| State                | RESTOREタスクの現在の状態：<ul><li>PENDING: ジョブを提出した直後の初期状態。</li><li>SNAPSHOTING: ローカルスナップショットを実行中。</li><li>DOWNLOAD: スナップショットのダウンロードタスクを提出中。</li><li>DOWNLOADING: スナップショットのダウンロード中。</li><li>COMMIT: ダウンロードしたスナップショットをコミットする。</li><li>COMMITTING: ダウンロードしたスナップショットをコミット中。</li><li>FINISHED: RESTOREタスクが完了。</li><li>CANCELLED: RESTOREタスクが失敗またはキャンセルされた。</li></ul> |
| AllowLoad            | RESTOREタスク中にデータのロードが許可されているかどうか。         |
| ReplicationNum       | 復元するレプリカの数。                                       |
| RestoreObjs          | 復元されたオブジェクト（テーブルとパーティション）。              |
| CreateTime           | タスクの提出時刻。                                           |
| MetaPreparedTime     | ローカルメタデータの完了時刻。                                 |
| SnapshotFinishedTime | スナップショットの完了時刻。                                   |
| DownloadFinishedTime | スナップショットのダウンロード完了時刻。                       |
| FinishedTime         | タスク完了時刻。                                             |
| UnfinishedTasks      | SNAPSHOTTING、DOWNLOADING、COMMITTINGフェーズで未完了のサブタスクID。  |
| Progress             | スナップショットのダウンロードタスクの進捗。                    |
| TaskErrMsg           | エラーメッセージ。                                           |
| Status               | ステータス情報。                                            |
| Timeout              | タスクのタイムアウト。単位：秒。                             |

## 例

例1: データベース `example_db` の最後のRESTOREタスクを表示します。

```SQL
SHOW RESTORE FROM example_db;
```