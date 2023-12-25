---
displayed_sidebar: Chinese
---

# SHOW BACKUP

## 機能

指定されたデータベースのバックアップタスクを表示します。詳細については、[バックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

> **注意**
>
> StarRocksでは、最新のバックアップタスク情報のみが保存されます。

## 文法

```SQL
SHOW BACKUP [FROM <db_name>]
```

## パラメータ説明

| **パラメータ** | **説明**                   |
| -------------- | -------------------------- |
| db_name        | バックアップタスクが属するデータベース名。 |

## 戻り値

| **戻り値**             | **説明**                                                     |
| ---------------------- | ------------------------------------------------------------ |
| JobId                  | このバックアップジョブのID。                                 |
| SnapshotName           | ユーザーが指定したこのバックアップジョブの名前。             |
| DbName                 | バックアップジョブに対応するデータベース。                   |
| State                  | バックアップジョブの現在の状態：<ul><li>PENDING：ジョブの初期状態。</li><li>SNAPSHOTING：スナップショット操作を行っています。</li><li>UPLOAD_SNAPSHOT：スナップショットが完了し、アップロードの準備ができました。</li><li>UPLOADING：スナップショットをアップロードしています。</li><li>SAVE_META：ローカルでメタデータファイルを生成しています。</li><li>UPLOAD_INFO：メタデータファイルとこのバックアップジョブの情報をアップロードしています。</li><li>FINISHED：バックアップが完了しました。</li><li>CANCELLED：バックアップが失敗したかキャンセルされました。</li></ul> |
| BackupObjs             | このバックアップに関連するオブジェクト。                     |
| CreateTime             | ジョブの作成時間。                                           |
| SnapshotFinishedTime   | スナップショットの完了時間。                                 |
| UploadFinishedTime     | スナップショットのアップロードが完了した時間。               |
| FinishedTime           | このジョブの完了時間。                                       |
| UnfinishedTasks        | SNAPSHOTTING、UPLOADINGなどの段階で、複数のサブタスクが同時に進行している場合、ここには現在の段階で未完了のサブタスクのTask IDが表示されます。 |
| Progress               | スナップショットのアップロード進捗。                         |
| TaskErrMsg             | サブタスクの実行にエラーがあった場合、ここに該当するサブタスクのエラーメッセージが表示されます。 |
| Status                 | ジョブプロセス全体で発生可能ないくつかの状態情報を記録するために使用されます。 |
| Timeout                | ジョブのタイムアウト時間（秒単位）。                         |

## 例

例1：データベース `example_db` の最後のバックアップタスクを表示します。

```SQL
SHOW BACKUP FROM example_db;
```
