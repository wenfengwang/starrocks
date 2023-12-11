---
displayed_sidebar: "Japanese"
---

# パイプを表示

## 説明

指定されたデータベースまたは現在使用中のデータベースに格納されているパイプをリストします。

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

## パラメータ

### FROM `<db_name>`

パイプをクエリしたいデータベースの名前。このパラメータを指定しない場合、システムは現在使用中のデータベースのパイプを返します。

### WHERE

パイプをクエリするための基準。

### ORDER BY `<field_name>`

返されるレコードをソートしたいフィールド。

### LIMIT

システムに返してほしいレコードの最大数。

## 戻り値

コマンドの出力には、次のフィールドが含まれます。

| **Field**     | **Description**                                              |
| ------------- | ------------------------------------------------------------ |
| DATABASE_NAME | パイプが格納されているデータベースの名前。                   |
| PIPE_ID       | パイプの固有ID。                                              |
| PIPE_NAME     | パイプの名前。                                                |
| TABLE_NAME    | 送信先のStarRocksテーブルの名前。                           |
| STATE         | パイプの状態。有効な値：`RUNNING`、`FINISHED`、`SUSPENDED`、`ERROR`。 |
| LOAD_STATUS   | パイプを介して読み込まれるデータファイルの全体的なステータス。サブフィールドには以下が含まれます：<ul><li>`loadedFiles`: 読み込まれたデータファイルの数。</li><li>`loadedBytes`: バイト単位で測定された読み込まれたデータの容量。</li><li>`LastLoadedTime`: 最後のデータファイルが読み込まれた日時。フォーマット：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。</li></ul> |
| LAST_ERROR    | パイプ実行中に発生した最後のエラーに関する詳細。デフォルト値：`NULL`。 |
| CREATED_TIME  | パイプが作成された日時。フォーマット：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |

## 例

### すべてのパイプをクエリする

`mydatabase`という名前のデータベースに切り替え、それに含まれるすべてのパイプを表示します。

```SQL
USE mydatabase;
SHOW PIPES \G
```

### 指定されたパイプをクエリする

`mydatabase`という名前のデータベースに切り替え、その中の`user_behavior_replica`という名前のパイプを表示します。

```SQL
USE mydatabase;
SHOW PIPES WHERE NAME = 'user_behavior_replica' \G
```