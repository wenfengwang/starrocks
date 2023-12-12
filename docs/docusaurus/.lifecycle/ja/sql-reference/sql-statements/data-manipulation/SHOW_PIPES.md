---
displayed_sidebar: "Japanese"
---

# パイプの表示

## 説明

指定されたデータベースまたは使用中の現在のデータベースに格納されているパイプを一覧表示します。

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

クエリを実行するデータベースの名前。このパラメーターを指定しない場合、システムは使用中の現在のデータベースのパイプを返します。

### WHERE

パイプをクエリする基準。

### ORDER BY `<field_name>`

返されるレコードを並べ替えたいフィールド。

### LIMIT

システムに返してほしいレコードの最大数。

## 戻り値

コマンドの出力は以下のフィールドで構成されています。

| **フィールド** | **説明**                                                    |
| ------------- | ------------------------------------------------------------ |
| DATABASE_NAME | パイプが格納されているデータベースの名前。                |
| PIPE_ID       | パイプの固有のID。                                           |
| PIPE_NAME     | パイプの名前。                                               |
| TABLE_NAME    | デスティネーションStarRocksテーブルの名前。                |
| STATE         | パイプの状態。有効な値：`RUNNING`、`FINISHED`、`SUSPENDED`、`ERROR`。 |
| LOAD_STATUS   | パイプを介してロードされるデータファイルの全体的な状態、次のサブフィールドを含む：<ul><li>`loadedFiles`: ロードされたデータファイルの数。</li><li>`loadedBytes`: ロードされたデータの容量（バイト単位）。</li><li>`LastLoadedTime`: 最後のデータファイルがロードされた日時。フォーマット：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。</li></ul> |
| LAST_ERROR    | パイプの実行中に発生した直近のエラーに関する詳細。デフォルト値：`NULL`。 |
| CREATED_TIME  | パイプが作成された日時。フォーマット：`yyyy-MM-dd HH:mm:ss`。例：`2023-07-24 14:58:58`。 |

## 例

### すべてのパイプをクエリ

`mydatabase`という名前のデータベースに切り替え、その中にあるすべてのパイプを表示します：

```SQL
USE mydatabase;
SHOW PIPES \G
```

### 指定されたパイプをクエリ

`mydatabase`という名前のデータベースに切り替え、その中の`user_behavior_replica`という名前のパイプを表示します：

```SQL
USE mydatabase;
SHOW PIPES WHERE NAME = 'user_behavior_replica' \G
```