---
displayed_sidebar: English
---

# SHOW BACKUP

## 説明

指定されたデータベースで最後に行われたBACKUPタスクを表示します。詳細については、[データバックアップおよび復元](../../../administration/Backup_and_restore.md)を参照してください。

> **注記**
>
> StarRocksでは最後のBACKUPタスクの情報のみが保存されます。

## 構文

```SQL
SHOW BACKUP [FROM <db_name>]
```

## パラメーター

| **パラメーター** | **説明**                                       |
| ------------- | ----------------------------------------------------- |
| db_name       | BACKUPタスクが属するデータベースの名前。 |

## 戻り値

| **戻り値**           | **説明**                                              |
| -------------------- | ------------------------------------------------------------ |
| JobId                | 一意のジョブID。                                               |
| SnapshotName         | データスナップショットの名前。                                   |
| DbName               | BACKUPタスクが属するデータベースの名前。        |
| State                | BACKUPタスクの現在の状態：<ul><li>PENDING: ジョブ提出後の初期状態。</li><li>SNAPSHOTING: スナップショット作成中。</li><li>UPLOAD_SNAPSHOT: スナップショットが完了し、アップロード準備完了。</li><li>UPLOADING: スナップショットをアップロード中。</li><li>SAVE_META: ローカルメタデータファイルを作成中。</li><li>UPLOAD_INFO: メタデータファイルとBACKUPタスクの情報をアップロード中。</li><li>FINISHED: BACKUPタスク完了。</li><li>CANCELLED: BACKUPタスクが失敗またはキャンセルされた。</li></ul> |
| BackupObjs           | バックアップされたオブジェクト。                                           |
| CreateTime           | タスク提出時間。                                        |
| SnapshotFinishedTime | スナップショットの完了時間。                                    |
| UploadFinishedTime   | スナップショットのアップロード完了時間。                             |
| FinishedTime         | タスクの完了時間。                                        |
| UnfinishedTasks      | SNAPSHOTINGとUPLOADINGフェーズで未完了のサブタスクID。 |
| Progress             | スナップショットのアップロードタスクの進捗状況。                             |
| TaskErrMsg           | エラーメッセージ。                                              |
| Status               | ステータス情報。                                          |
| Timeout              | タスクのタイムアウト時間。単位は秒です。                                  |

## 例

例1: `example_db`データベースの最後のBACKUPタスクを表示します。

```SQL
SHOW BACKUP FROM example_db;
```
