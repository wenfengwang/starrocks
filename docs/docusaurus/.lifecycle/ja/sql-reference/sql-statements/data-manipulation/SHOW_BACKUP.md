---
displayed_sidebar: "Japanese"
---

# バックアップの表示

## 説明

指定されたデータベース内の直近のバックアップタスクを表示します。詳細については、[データのバックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

> **注意**
>
> 直近のバックアップタスクの情報のみがStarRocksに保存されます。

## 構文

```SQL
SHOW BACKUP [FROM <db_name>]
```

## パラメータ

| **パラメータ** | **説明**                                       |
| ------------- | ----------------------------------------------------- |
| db_name       | バックアップタスクが属するデータベースの名前。 |

## 戻り値

| **戻り値**           | **説明**                                              |
| -------------------- | ------------------------------------------------------------ |
| JobId                | ユニークなジョブID。                                               |
| SnapshotName         | データスナップショットの名前。                                   |
| DbName               | バックアップタスクが属するデータベースの名前。        |
| State                | バックアップタスクの現在の状態：<ul><li>PENDING：ジョブの提出後の初期状態。</li><li>SNAPSHOTING：スナップショットの作成。</li><li>UPLOAD_SNAPSHOT：スナップショットが完了し、アップロードの準備ができています。</li><li>UPLOADING：スナップショットのアップロード。</li><li>SAVE_META：ローカルのメタデータファイルの作成。</li><li>UPLOAD_INFO：メタデータファイルおよびバックアップタスクの情報のアップロード。</li><li>FINISHED：バックアップタスクが完了。</li><li>CANCELLED：バックアップタスクが失敗またはキャンセルされました。</li></ul> |
| BackupObjs           | バックアップされたオブジェクト。                                           |
| CreateTime           | タスクの提出時間。                                        |
| SnapshotFinishedTime | スナップショットの完了時間。                                    |
| UploadFinishedTime   | スナップショットのアップロード完了時間。                             |
| FinishedTime         | タスクの完了時刻。                                        |
| UnfinishedTasks      | SNAPSHOTINGおよびUPLOADINGフェーズで未完了のサブタスクのID。 |
| Progress             | スナップショットのアップロードタスクの進捗状況。                             |
| TaskErrMsg           | エラーメッセージ。                                              |
| Status               | ステータス情報。                                          |
| Timeout              | タスクのタイムアウト。単位：秒。                                  |

## 例

例1：データベース`example_db`内の直近のバックアップタスクを表示します。

```SQL
SHOW BACKUP FROM example_db;
```