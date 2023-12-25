---
displayed_sidebar: English
---

# SHOW RESTORE の表示

## 説明

指定されたデータベースで最後に行われた RESTORE タスクを表示します。詳細については、[データバックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

> **注記**
>
> StarRocksでは最後のRESTOREタスクの情報のみが保存されます。

## 構文

```SQL
SHOW RESTORE [FROM <db_name>]
```

## パラメータ

| **パラメータ** | **説明**                                        |
| ------------- | ------------------------------------------------------ |
| db_name       | RESTOREタスクが属するデータベースの名前。 |

## 戻り値

| **戻り値**           | **説明**                                              |
| -------------------- | ------------------------------------------------------------ |
| JobId                | 一意のジョブID。                                               |
| Label                | データスナップショットの名前。                                   |
| Timestamp            | バックアップタイムスタンプ。                                            |
| DbName               | RESTOREタスクが属するデータベースの名前。       |
| State                | RESTOREタスクの現在の状態：<ul><li>PENDING: ジョブ提出後の初期状態。</li><li>SNAPSHOTING: ローカルスナップショットを実行中。</li><li>DOWNLOAD: スナップショットダウンロードタスクを提出中。</li><li>DOWNLOADING: スナップショットをダウンロード中。</li><li>COMMIT: ダウンロードしたスナップショットをコミットする。</li><li>COMMITTING: ダウンロードしたスナップショットをコミット中。</li><li>FINISHED: RESTOREタスク完了。</li><li>CANCELLED: RESTOREタスク失敗またはキャンセルされた。</li></ul> |
| AllowLoad            | RESTOREタスク中にデータのロードが許可されているかどうか。          |
| ReplicationNum       | 復元されるレプリカの数。                           |
| RestoreObjs          | 復元されるオブジェクト（テーブルとパーティション）。                |
| CreateTime           | タスク提出時間。                                        |
| MetaPreparedTime     | ローカルメタデータの準備完了時間。                              |
| SnapshotFinishedTime | スナップショットの完了時間。                                    |
| DownloadFinishedTime | スナップショットダウンロードの完了時間。                           |
| FinishedTime         | タスク完了時間。                                        |
| UnfinishedTasks      | SNAPSHOTTING、DOWNLOADING、COMMITTINGフェーズの未完了サブタスクID。 |
| Progress             | スナップショットダウンロードタスクの進捗状況。                  |
| TaskErrMsg           | エラーメッセージ。                                              |
| Status               | ステータス情報。                                          |
| Timeout              | タスクのタイムアウト。単位は秒。                                  |

## 例

例1: `example_db` データベースの最後のRESTOREタスクを表示します。

```SQL
SHOW RESTORE FROM example_db;
```
