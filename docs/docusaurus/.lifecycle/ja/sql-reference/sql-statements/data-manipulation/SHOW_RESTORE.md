---
displayed_sidebar: "Japanese"
---

# リストアの表示

## 説明

指定されたデータベース内の最後のリストアタスクを表示します。詳細については、[データのバックアップとリストア](../../../administration/Backup_and_restore.md)を参照してください。

> **注意**
>
> StarRocksでは、最後のリストアタスクの情報のみが保存されます。

## 構文

```SQL
SHOW RESTORE [FROM <db_name>]
```

## パラメータ

| **パラメータ** | **説明**                                               |
| ------------- | ----------------------------------------------------- |
| db_name       | リストアタスクが属するデータベースの名前。           |

## 戻り値

| **戻り値**           | **説明**                                                     |
| -------------------- | --------------------------------------------------------- |
| JobId                | ユニークなジョブID。                                         |
| Label                | データスナップショットの名前。                             |
| Timestamp            | バックアップのタイムスタンプ。                              |
| DbName               | リストアタスクが属するデータベースの名前。                  |
| State                | リストアタスクの現在のステータス：<ul><li>PENDING: ジョブを送信した後の初期状態。</li><li>SNAPSHOTING: ローカルスナップショットの実行。</li><li>DOWNLOAD: スナップショットダウンロードタスクを送信。</li><li>DOWNLOADING: スナップショットのダウンロード。</li><li>COMMIT: ダウンロードしたスナップショットをコミット。</li><li>COMMITTING: ダウンロードしたスナップショットをコミット中。</li><li>FINISHED: リストアタスク完了。</li><li>CANCELLED: リストアタスクが失敗またはキャンセルされました。</li></ul> |
| AllowLoad            | リストアタスク中にデータをロードすることが許可されているかどうか。 |
| ReplicationNum       | リストアするレプリカの数。                                   |
| RestoreObjs          | 復元されたオブジェクト（テーブルおよびパーティション）。       |
| CreateTime           | タスクの送信時刻。                                           |
| MetaPreparedTime     | ローカルメタデータの完了時刻。                               |
| SnapshotFinishedTime | スナップショットの完了時刻。                                 |
| DownloadFinishedTime | スナップショットのダウンロード完了時刻。                    |
| FinishedTime         | タスクの完了時刻。                                           |
| UnfinishedTasks      | SNAPSHOTING、DOWNLOADING、およびCOMMITTINGフェーズの未完了サブタスクのID。 |
| Progress             | スナップショットのダウンロードタスクの進行状況。              |
| TaskErrMsg           | エラーメッセージ。                                           |
| Status               | ステータス情報。                                             |
| Timeout              | タスクのタイムアウト。単位：秒。                            |

## 例

例1：データベース `example_db` で最後のリストアタスクを表示する。

```SQL
SHOW RESTORE FROM example_db;
```