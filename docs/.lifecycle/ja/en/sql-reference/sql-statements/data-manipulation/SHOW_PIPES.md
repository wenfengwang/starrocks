---
displayed_sidebar: English
---

# SHOW PIPES

## 説明

指定されたデータベース、または現在使用中のデータベースに保存されているパイプを一覧表示します。

## 構文

```SQL
SHOW PIPES [FROM <db_name>]
[
   WHERE [ NAME { = "<pipe_name>" | LIKE "pipe_matcher" } ]
         [ [AND] STATE = { "SUSPENDED" | "RUNNING" | "ERROR" } ]
]
[ ORDER BY <field_name> [ ASC | DESC ] ]
[ LIMIT { [offset, ] limit | limit OFFSET offset } ]
```

## パラメーター

### FROM `<db_name>`

クエリを行いたいデータベースの名前です。このパラメーターを指定しない場合、システムは現在使用中のデータベースのパイプを返します。

### WHERE

パイプをクエリする基準です。

### ORDER BY `<field_name>`

返されるレコードをソートするフィールドです。

### LIMIT

システムが返すレコードの最大数です。

## 戻り値

コマンドの出力には以下のフィールドが含まれます。

| **フィールド** | **説明**                                              |
| --------------- | ------------------------------------------------------ |
| DATABASE_NAME   | パイプが保存されているデータベースの名前。            |
| PIPE_ID         | パイプの一意なID。                                     |
| PIPE_NAME       | パイプの名前。                                         |
| TABLE_NAME      | 宛先のStarRocksテーブルの名前。                        |
| STATE           | パイプの状態。有効な値: `RUNNING`, `FINISHED`, `SUSPENDED`, `ERROR`。 |
| LOAD_STATUS     | パイプを介してロードされるデータファイルの全体的な状態。以下のサブフィールドが含まれます:<ul><li>`loadedFiles`: ロードされたデータファイルの数。</li><li>`loadedBytes`: ロードされたデータの量。単位はバイト。</li><li>`LastLoadedTime`: 最後のデータファイルがロードされた日時。フォーマット: `yyyy-MM-dd HH:mm:ss`。例: `2023-07-24 14:58:58`。</li></ul> |
| LAST_ERROR      | パイプ実行中に発生した最後のエラーの詳細。デフォルト値: `NULL`。 |
| CREATED_TIME    | パイプが作成された日時。フォーマット: `yyyy-MM-dd HH:mm:ss`。例: `2023-07-24 14:58:58`。 |

## 例

### すべてのパイプをクエリする

`mydatabase` という名前のデータベースに切り替え、その中のすべてのパイプを表示します。

```SQL
USE mydatabase;
SHOW PIPES \G
```

### 特定のパイプをクエリする

`mydatabase` という名前のデータベースに切り替え、その中の `user_behavior_replica` という名前のパイプを表示します。

```SQL
USE mydatabase;
SHOW PIPES WHERE NAME = 'user_behavior_replica' \G
```
