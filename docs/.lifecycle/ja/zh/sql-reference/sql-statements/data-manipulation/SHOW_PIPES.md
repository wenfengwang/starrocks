---
displayed_sidebar: Chinese
---

# SHOW PIPES

## 機能

現在のデータベースまたは指定されたデータベースの下にあるPipeを表示します。このコマンドはバージョン3.2からサポートされています。

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

## パラメータ説明

### FROM `<db_name>`

検索対象のデータベース名を指定します。データベースが指定されていない場合は、現在のデータベースの下のPipeがデフォルトで検索されます。

### WHERE

検索条件を指定します。

### ORDER BY `<field_name>`

結果レコードのソートフィールドを指定します。

### LIMIT

システムが返す最大結果レコード数を指定します。

## 戻り値

戻り値には以下のフィールドが含まれます：

| **フィールド** | **説明**                                                     |
| -------------- | ------------------------------------------------------------ |
| DATABASE_NAME  | Pipeが属するデータベースの名前。                             |
| ID             | PipeのユニークなID。                                         |
| NAME           | Pipeの名前。                                                 |
| TABLE_NAME     | StarRocksのターゲットテーブルの名前。                        |
| STATE          | Pipeの状態で、`RUNNING`、`FINISHED`、`SUSPENDED`、`ERROR`が含まれます。 |
| LOAD_STATUS    | Pipeの下でインポートされるデータファイルの全体的な状態で、以下のフィールドが含まれます：<ul><li>`loadedFiles`：インポートされたデータファイルの総数。</li><li>`loadedBytes`：インポートされたデータの総量で、単位はバイトです。</li><li>`LastLoadedTime`：最後にインポートを実行した時間。形式：`yyyy-MM-dd HH:mm:ss`。例えば、`2023-07-24 14:58:58`。</li></ul> |
| LAST_ERROR     | Pipeの実行中に発生した最新のエラーの詳細情報。デフォルト値は`NULL`です。 |
| CREATED_TIME   | Pipeの作成時間。形式：`yyyy-MM-dd HH:mm:ss`。例えば：`2023-07-24 14:58:58`。 |

## 例

### すべてのパイプを照会する

データベース `mydatabase` に入り、そのデータベースの下にあるすべてのPipeを表示します：

```SQL
USE mydatabase;
SHOW PIPES \G
```

### 特定のパイプを照会する

データベース `mydatabase` に入り、そのデータベースの下にある `user_behavior_replica` という名前のPipeを表示します：

```SQL
USE mydatabase;
SHOW PIPES WHERE NAME = 'user_behavior_replica' \G
```
