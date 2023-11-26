---
displayed_sidebar: "Japanese"
---

# INSERT

## 説明

特定のテーブルにデータを挿入するか、特定のテーブルをデータで上書きします。アプリケーションシナリオの詳細については、[INSERTを使用したデータのロード](../../../loading/InsertInto.md)を参照してください。

[SUBMIT TASK](./SUBMIT_TASK.md)を使用して非同期のINSERTタスクを送信できます。

## 構文

```Bash
INSERT { INTO | OVERWRITE } [db_name.]<table_name>
[ PARTITION (<partition_name> [, ...) ]
[ TEMPORARY PARTITION (<temporary_partition_name>[, ...) ]
[ WITH LABEL <label>]
[ (<column_name>[, ...]) ]
{ VALUES ( { <expression> | DEFAULT }[, ...] )
  | <query> }
```

## パラメータ

| **パラメータ** | 説明                                                         |
| ------------- | ------------------------------------------------------------ |
| INTO          | テーブルにデータを追加します。                                 |
| OVERWRITE     | テーブルをデータで上書きします。                            |
| table_name    | データをロードするテーブルの名前です。`db_name.table_name`のように、テーブルが存在するデータベースで指定できます。 |
| PARTITION    |  データをロードするパーティションです。複数のパーティションを指定できますが、カンマ（,）で区切る必要があります。指定するパーティションは、宛先テーブルに存在するパーティションに設定する必要があります。このパラメータを指定すると、データは指定されたパーティションにのみ挿入されます。このパラメータを指定しない場合、データはすべてのパーティションに挿入されます。 |
| TEMPORARY PARTITION|データをロードする[一時パーティション](../../../table_design/Temporary_partition.md)の名前です。複数の一時パーティションを指定できますが、カンマ（,）で区切る必要があります。|
| label         | データベース内の各データロードトランザクションの一意の識別ラベルです。指定しない場合、システムはトランザクションのために自動的に1つ生成します。トランザクションのステータスを確認するには、`SHOW LOAD WHERE label="label"`ステートメントを使用できます。ラベルの命名に関する制限については、[システム制限](../../../reference/System_limit.md)を参照してください。 |
| column_name   | データをロードする宛先の列の名前です。宛先テーブルに存在する列として設定する必要があります。指定した宛先列は、宛先列の名前に関係なく、ソーステーブルの列と1対1でマッピングされます。宛先列が指定されていない場合、デフォルト値は宛先テーブルのすべての列です。ソーステーブルの指定された列が宛先列に存在しない場合、この列にはデフォルト値が書き込まれ、指定された列にデフォルト値がない場合はトランザクションが失敗します。ソーステーブルの列の型が宛先テーブルの列の型と一致しない場合、システムは不一致のある列に対して暗黙の型変換を実行します。変換に失敗すると、構文解析エラーが返されます。 |
| expression    | 列に値を割り当てる式です。                |
| DEFAULT       | 列にデフォルト値を割り当てます。                         |
| query         | 宛先テーブルにロードされるクエリステートメントです。StarRocksでサポートされている任意のSQLステートメントを使用できます。 |

## 戻り値

```Plain
Query OK, 5 rows affected, 2 warnings (0.05 sec)
{'label':'insert_load_test', 'status':'VISIBLE', 'txnId':'1008'}
```

| 戻り値        | 説明                                                         |
| ------------- | ------------------------------------------------------------ |
| rows affected | ロードされた行数を示します。`warnings`はフィルタリングされた行数を示します。 |
| label         | データベース内の各データロードトランザクションの一意の識別ラベルです。ユーザーによって割り当てられるか、システムによって自動的に割り当てられることができます。 |
| status        | ロードされたデータが表示可能かどうかを示します。`VISIBLE`：データが正常にロードされ、表示可能です。`COMMITTED`：データが正常にロードされましたが、現在は非表示です。 |
| txnId         | 各INSERTトランザクションに対応するID番号です。      |

## 使用上の注意

- 現在のバージョンでは、StarRocksがINSERT INTOステートメントを実行する際、データのいずれかの行が宛先テーブルの形式と一致しない場合（たとえば、文字列が長すぎる場合など）、INSERTトランザクションはデフォルトで失敗します。システムが宛先テーブルの形式と一致しないデータをフィルタリングしてトランザクションを続行するように、セッション変数`enable_insert_strict`を`false`に設定できます。

- INSERT OVERWRITEステートメントが実行されると、StarRocksは元のデータを格納するパーティションに一時パーティションを作成し、データを一時パーティションに挿入し、元のパーティションを一時パーティションと交換します。これらの操作はすべてリーダーFEノードで実行されます。したがって、INSERT OVERWRITEステートメントの実行中にリーダーFEノードがクラッシュすると、ロードトランザクション全体が失敗し、一時パーティションが削除されます。

## 例

以下の例は、2つの列`c1`と`c2`を含む`test`テーブルを基にしています。`c2`列はDEFAULTのデフォルト値を持っています。

- `test`テーブルに1行のデータをインポートします。

```SQL
INSERT INTO test VALUES (1, 2);
INSERT INTO test (c1, c2) VALUES (1, 2);
INSERT INTO test (c1, c2) VALUES (1, DEFAULT);
INSERT INTO test (c1) VALUES (1);
```

宛先列が指定されていない場合、デフォルトで列は順番に宛先テーブルにロードされます。したがって、上記の例の最初のSQLステートメントと2番目のSQLステートメントの結果は同じです。

宛先列（データが挿入されている場合でも）が値としてDEFAULTを使用する場合、列はロードされたデータとしてデフォルト値を使用します。したがって、上記の例の3番目と4番目のステートメントの出力は同じです。

- 複数の行のデータを一度に`test`テーブルにロードします。

```SQL
INSERT INTO test VALUES (1, 2), (3, 2 + 2);
INSERT INTO test (c1, c2) VALUES (1, 2), (3, 2 * 2);
INSERT INTO test (c1, c2) VALUES (1, DEFAULT), (3, DEFAULT);
INSERT INTO test (c1) VALUES (1), (3);
```

式の結果が等しいため、最初のステートメントと2番目のステートメントの結果は同じです。デフォルト値を使用しているため、3番目のステートメントと4番目のステートメントの結果も同じです。

- クエリステートメントの結果を`test`テーブルにインポートします。

```SQL
INSERT INTO test SELECT * FROM test2;
INSERT INTO test (c1, c2) SELECT * from test2;
```

- クエリ結果を`test`テーブルにインポートし、パーティションとラベルを指定します。

```SQL
INSERT INTO test PARTITION(p1, p2) WITH LABEL `label1` SELECT * FROM test2;
INSERT INTO test WITH LABEL `label1` (c1, c2) SELECT * from test2;
```

- クエリ結果で`test`テーブルを上書きし、パーティションとラベルを指定します。

```SQL
INSERT OVERWRITE test PARTITION(p1, p2) WITH LABEL `label1` SELECT * FROM test3;
INSERT OVERWRITE test WITH LABEL `label1` (c1, c2) SELECT * from test3;
```

以下の例では、AWS S3バケット`inserttest`内のParquetファイル**parquet/insert_wiki_edit_append.parquet**からデータ行を`insert_wiki_edit`テーブルに挿入します。

```Plain
INSERT INTO insert_wiki_edit
    SELECT * FROM FILES(
        "path" = "s3://inserttest/parquet/insert_wiki_edit_append.parquet",
        "format" = "parquet",
        "aws.s3.access_key" = "XXXXXXXXXX",
        "aws.s3.secret_key" = "YYYYYYYYYY",
        "aws.s3.region" = "ap-southeast-1"
);
```
