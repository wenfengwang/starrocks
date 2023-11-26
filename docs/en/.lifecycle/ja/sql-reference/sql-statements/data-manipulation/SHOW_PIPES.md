---
displayed_sidebar: "Japanese"
---

# パイプの表示

## 説明

指定されたデータベースまたは現在使用中のデータベースに格納されているパイプを一覧表示します。

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

問い合わせるパイプのデータベースの名前です。このパラメータを指定しない場合、システムは現在使用中のデータベースのパイプを返します。

### WHERE

パイプをクエリする基準です。

### ORDER BY `<field_name>`

返されるレコードをソートするためのフィールドです。

### LIMIT

システムが返す最大レコード数です。

## 戻り値

コマンドの出力は、以下のフィールドで構成されています。

| **フィールド**   | **説明**                                                     |
| --------------- | ------------------------------------------------------------ |
| DATABASE_NAME   | パイプが格納されているデータベースの名前です。                 |
| PIPE_ID         | パイプの一意のIDです。                                        |
| PIPE_NAME       | パイプの名前です。                                            |
| TABLE_NAME      | StarRocksテーブルの宛先の名前です。                           |
| STATE           | パイプのステータスです。有効な値: `RUNNING`, `FINISHED`, `SUSPENDED`, `ERROR`。 |
| LOAD_STATUS     | パイプを介してロードされるデータファイルの全体的なステータスです。以下のサブフィールドを含みます:<ul><li>`loadedFiles`: ロードされたデータファイルの数。</li><li>`loadedBytes`: ロードされたデータのボリューム（バイト単位）。</li><li>`LastLoadedTime`: 最後のデータファイルがロードされた日時。形式: `yyyy-MM-dd HH:mm:ss`。例: `2023-07-24 14:58:58`。</li></ul> |
| LAST_ERROR      | パイプの実行中に発生した最後のエラーに関する詳細です。デフォルト値: `NULL`。 |
| CREATED_TIME    | パイプが作成された日時です。形式: `yyyy-MM-dd HH:mm:ss`。例: `2023-07-24 14:58:58`。 |

## 例

### すべてのパイプをクエリする

`mydatabase`という名前のデータベースに切り替えて、それに含まれるすべてのパイプを表示します:

```SQL
USE mydatabase;
SHOW PIPES \G
```

### 指定されたパイプをクエリする

`mydatabase`という名前のデータベースに切り替えて、それに含まれる`user_behavior_replica`という名前のパイプを表示します:

```SQL
USE mydatabase;
SHOW PIPES WHERE NAME = 'user_behavior_replica' \G
```
