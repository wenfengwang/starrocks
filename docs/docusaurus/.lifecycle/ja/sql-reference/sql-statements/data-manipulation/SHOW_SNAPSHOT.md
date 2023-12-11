---
displayed_sidebar: "Japanese"
---

# スナップショットの表示

## 説明

指定されたリポジトリでデータのスナップショットを表示します。詳細については、[データのバックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

## 構文

```SQL
SHOW SNAPSHOT ON <repo_name>
[WHERE SNAPSHOT = <snapshot_name> [AND TIMESTAMP = <backup_timestamp>]]
```

## パラメータ

| **パラメータ**   | **説明**                                                   |
| ---------------- | ------------------------------------------------------------ |
| repo_name        | スナップショットが属するリポジトリの名前。                |
| snapshot_name    | スナップショットの名前。                                    |
| backup_timestamp | スナップショットのバックアップタイムスタンプ。             |

## 戻り値

| **戻り値**       | **説明**                                                   |
| ---------------- | ------------------------------------------------------------ |
| スナップショット   | スナップショットの名前。                                    |
| タイムスタンプ    | スナップショットのバックアップタイムスタンプ。             |
| ステータス       | スナップショットが正常なら`OK`を表示します。スナップショットが正常でない場合はエラーメッセージを表示します。 |
| データベース     | スナップショットが属するデータベースの名前。               |
| 詳細            | スナップショットのJSON形式のディレクトリと構造。           |

## 例

例 1: リポジトリ`example_repo`のスナップショットを表示します。

```SQL
SHOW SNAPSHOT ON example_repo;
```

例 2: リポジトリ`example_repo`で`backup1`という名前のスナップショットを表示します。

```SQL
SHOW SNAPSHOT ON example_repo
WHERE SNAPSHOT = "backup1";
```

例 3: リポジトリ`example_repo`で`backup1`という名前のスナップショットとバックアップタイムスタンプ`2018-05-05-15-34-26`を表示します。

```SQL
SHOW SNAPSHOT ON example_repo 
WHERE SNAPSHOT = "backup1" AND TIMESTAMP = "2018-05-05-15-34-26";
```