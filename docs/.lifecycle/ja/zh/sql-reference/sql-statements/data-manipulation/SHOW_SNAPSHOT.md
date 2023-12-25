---
displayed_sidebar: Chinese
---

# スナップショットの表示

## 機能

指定されたリポジトリのバックアップを表示します。詳細については、[バックアップと復元](../../../administration/Backup_and_restore.md)を参照してください。

## 構文

```SQL
SHOW SNAPSHOT ON <repo_name>
[WHERE SNAPSHOT = <snapshot_name> [AND TIMESTAMP = <backup_timestamp>]]
```

## パラメータ説明

| **パラメータ**   | **説明**           |
| ---------------- | ------------------ |
| repo_name        | バックアップのリポジトリ名。 |
| snapshot_name    | バックアップ名。       |
| backup_timestamp | バックアップのタイムスタンプ。 |

## 戻り値

| **戻り値**  | **説明**                                    |
| ----------- | ------------------------------------------- |
| Snapshot    | バックアップ名。                            |
| Timestamp   | バックアップのタイムスタンプ。              |
| Status      | バックアップが正常な場合は OK、そうでなければエラーメッセージを表示。 |
| Database    | バックアップされたデータベース名。          |
| Details     | バックアップされたデータディレクトリとファイル構造。JSON 形式。 |

## 例

例1：リポジトリ `example_repo` にあるバックアップを表示します。

```SQL
SHOW SNAPSHOT ON example_repo;
```

例2：リポジトリ `example_repo` にある `backup1` という名前のバックアップを表示します。

```SQL
SHOW SNAPSHOT ON example_repo
WHERE SNAPSHOT = "backup1";
```

例3：リポジトリ `example_repo` にある `backup1` という名前で、タイムスタンプが `2018-05-05-15-34-26` のバックアップを表示します。

```SQL
SHOW SNAPSHOT ON example_repo 
WHERE SNAPSHOT = "backup1" AND TIMESTAMP = "2018-05-05-15-34-26";
```
