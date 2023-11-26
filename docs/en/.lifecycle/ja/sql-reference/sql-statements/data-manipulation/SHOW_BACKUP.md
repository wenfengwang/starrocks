---
displayed_sidebar: "Japanese"
---

# バックアップの表示

## 説明

指定されたデータベースの最後のバックアップタスクを表示します。詳細については、[データのバックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

> **注意**
>
> StarRocksでは、最後のバックアップタスクの情報のみが保存されます。

## 構文

```SQL
SHOW BACKUP [FROM <db_name>]
```

## パラメータ

| **パラメータ** | **説明**                                       |
| ------------- | ----------------------------------------------------- |
| db_name       | バックアップタスクが所属するデータベースの名前。 |

## 戻り値

| **戻り値**           | **説明**                                              |
| -------------------- | ------------------------------------------------------------ |
| JobId                | ユニークなジョブID。                                               |
| SnapshotName         | データスナップショットの名前。                                   |
| DbName               | バックアップタスクが所属するデータベースの名前。        |
| State                | バックアップタスクの現在の状態:<ul><li>PENDING: ジョブの提出後の初期状態。</li><li>SNAPSHOTING: スナップショットの作成中。</li><li>UPLOAD_SNAPSHOT: スナップショットの作成が完了し、アップロードの準備ができています。</li><li>UPLOADING: スナップショットのアップロード中。</li><li>SAVE_META: ローカルのメタデータファイルの作成中。</li><li>UPLOAD_INFO: メタデータファイルとバックアップタスクの情報のアップロード中。</li><li>FINISHED: バックアップタスクが完了しました。</li><li>CANCELLED: バックアップタスクが失敗またはキャンセルされました。</li></ul> |
| BackupObjs           | バックアップされたオブジェクト。                                           |
| CreateTime           | タスクの提出時刻。                                        |
| SnapshotFinishedTime | スナップショットの完了時刻。                                    |
| UploadFinishedTime   | スナップショットのアップロード完了時刻。                             |
| FinishedTime         | タスクの完了時刻。                                        |
| UnfinishedTasks      | SNAPSHOTINGおよびUPLOADINGフェーズの未完了のサブタスクID。 |
| Progress             | スナップショットのアップロードタスクの進捗状況。                             |
| TaskErrMsg           | エラーメッセージ。                                              |
| Status               | ステータス情報。                                          |
| Timeout              | タスクのタイムアウト。単位：秒。                                  |

## 例

例1: データベース `example_db` の最後のバックアップタスクを表示します。

```SQL
SHOW BACKUP FROM example_db;
```
