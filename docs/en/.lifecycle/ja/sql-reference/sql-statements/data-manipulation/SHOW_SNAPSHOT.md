---
displayed_sidebar: "Japanese"
---

# スナップショットの表示

## 説明

指定されたリポジトリ内のデータスナップショットを表示します。詳細については、[データのバックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

## 構文

```SQL
SHOW SNAPSHOT ON <repo_name>
[WHERE SNAPSHOT = <snapshot_name> [AND TIMESTAMP = <backup_timestamp>]]
```

## パラメータ

| **パラメータ**    | **説明**                                      |
| ---------------- | ---------------------------------------------------- |
| repo_name        | スナップショットが所属するリポジトリの名前。 |
| snapshot_name     | スナップショットの名前。                                |
| backup_timestamp | スナップショットのバックアップタイムスタンプ。                    |

## 戻り値

| **戻り値** | **説明**                                              |
| ---------- | ------------------------------------------------------------ |
| Snapshot   | スナップショットの名前。                                        |
| Timestamp  | スナップショットのバックアップタイムスタンプ。                            |
| Status     | スナップショットが正常であれば`OK`を表示します。スナップショットが正常でない場合はエラーメッセージを表示します。 |
| Database   | スナップショットが所属するデータベースの名前。           |
| Details    | スナップショットのJSON形式のディレクトリと構造。      |

## 例

例1: リポジトリ`example_repo`内のスナップショットを表示します。

```SQL
SHOW SNAPSHOT ON example_repo;
```

例2: リポジトリ`example_repo`内のスナップショット`backup1`を表示します。

```SQL
SHOW SNAPSHOT ON example_repo
WHERE SNAPSHOT = "backup1";
```

例3: リポジトリ`example_repo`内のスナップショット`backup1`とバックアップタイムスタンプ`2018-05-05-15-34-26`を表示します。

```SQL
SHOW SNAPSHOT ON example_repo 
WHERE SNAPSHOT = "backup1" AND TIMESTAMP = "2018-05-05-15-34-26";
```
