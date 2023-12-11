---
displayed_sidebar: "Japanese"
---

# バックアップ表示

## 説明

指定されたデータベース内の最後のバックアップタスクを表示します。詳細については、[データのバックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

> **注意**
>
> StarRocks では、最後のバックアップタスクの情報のみが保存されます。

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
| State                | バックアップタスクの現在の状態:<ul><li>PENDING: ジョブを送信した直後の初期状態。</li><li>SNAPSHOTING: スナップショットの作成中。</li><li>UPLOAD_SNAPSHOT: スナップショットが完了し、アップロードの準備ができています。</li><li>UPLOADING: スナップショットのアップロード中。</li><li>SAVE_META: ローカルのメタデータファイルを作成中。</li><li>UPLOAD_INFO: メタデータファイルとバックアップタスクの情報をアップロード中。</li><li>FINISHED: バックアップタスクが終了。</li><li>CANCELLED: バックアップタスクが失敗またはキャンセルされました。</li></ul> |
| BackupObjs           | バックアップされたオブジェクト。                                           |
| CreateTime           | タスクの送信時間。                                        |
| SnapshotFinishedTime | スナップショットの完了時間。                                    |
| UploadFinishedTime   | スナップショットのアップロード完了時間。                             |
| FinishedTime         | タスクの完了時間。                                        |
| UnfinishedTasks      | SNAPSHOTING および UPLOADING フェーズで未完了のサブタスクID。 |
| Progress             | スナップショットのアップロードタスクの進捗。                             |
| TaskErrMsg           | エラーメッセージ。                                              |
| Status               | ステータス情報。                                          |
| Timeout              | タスクのタイムアウト。 単位: 秒。                                  |

## 例

例1: データベース `example_db` 内の最後のバックアップタスクを表示する。

```SQL
SHOW BACKUP FROM example_db;
```