---
displayed_sidebar: English
---

# スナップショットの表示

## 説明

指定されたリポジトリのデータスナップショットを表示します。詳細については、[データバックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

## 構文

```SQL
SHOW SNAPSHOT ON <repo_name>
[WHERE SNAPSHOT = <snapshot_name> [AND TIMESTAMP = <backup_timestamp>]]
```

## パラメータ

| **パラメータ**    | **説明**                                      |
| ---------------- | ---------------------------------------------------- |
| repo_name        | スナップショットが属するリポジトリの名前です。 |
| snapshot_name    | スナップショットの名前です。                                |
| backup_timestamp | スナップショットのバックアップタイムスタンプです。                    |

## 戻り値

| **戻り値** | **説明**                                              |
| ---------- | ------------------------------------------------------------ |
| Snapshot   | スナップショットの名前です。                                        |
| Timestamp  | スナップショットのバックアップタイムスタンプです。                            |
| Status     | スナップショットが正常であれば `OK` が表示されます。問題がある場合はエラーメッセージが表示されます。 |
| Database   | スナップショットが属するデータベースの名前です。           |
| Details    | スナップショットのディレクトリと構造をJSON形式で表示します。      |

## 例

例 1: リポジトリ `example_repo` のスナップショットを表示します。

```SQL
SHOW SNAPSHOT ON example_repo;
```

例 2: リポジトリ `example_repo` にある `backup1` という名前のスナップショットを表示します。

```SQL
SHOW SNAPSHOT ON example_repo
WHERE SNAPSHOT = "backup1";
```

例 3: リポジトリ `example_repo` にある `backup1` という名前のスナップショットを、バックアップタイムスタンプ `2018-05-05-15-34-26` で表示します。

```SQL
SHOW SNAPSHOT ON example_repo 
WHERE SNAPSHOT = "backup1" AND TIMESTAMP = "2018-05-05-15-34-26";
```
