```yaml
---
displayed_sidebar: "Japanese"
---
# INSERT

## 説明

特定のテーブルにデータを挿入するか、特定のテーブルをデータで上書きします。アプリケーションシナリオの詳細については、[INSERTを使用したデータのロード](../../../loading/InsertInto.md)を参照してください。v3.2.0以降、INSERTはリモートストレージにファイルにデータを書き込むことをサポートしています。StarRocksからリモートストレージにデータをアンロードするためには、[INSERT INTO FILES()を使用してデータをアンロードする](../../../unloading/unload_using_insert_into_files.md)ことができます。

[SUBMIT TASK](./SUBMIT_TASK.md)を使用して非同期のINSERTタスクを送信できます。

## 構文

- **データのロード**:

  ```sql
  INSERT { INTO | OVERWRITE } [db_name.]<table_name>
  [ PARTITION (<partition_name> [, ...] ) ]
  [ TEMPORARY PARTITION (<temporary_partition_name> [, ...] ) ]
  [ WITH LABEL <label>]
  [ (<column_name>[, ...]) ]
  { VALUES ( { <expression> | DEFAULT } [, ...] ) | <query> }
  ```

- **データのアンロード**:

  ```sql
  INSERT INTO FILES()
  [ WITH LABEL <label> ]
  { VALUES ( { <expression> | DEFAULT } [, ...] ) | <query> }
  ```

## パラメータ

| **パラメータ** | 説明                                                  |
| ------------- | ------------------------------------------------------------ |
| INTO          | テーブルにデータを追加します。                                 |
| OVERWRITE     | テーブルをデータで上書きします。                            |
| table_name    | データをロードするテーブルの名前。`db_name.table_name`として指定することができます。 |
| PARTITION    |  データをロードするパーティション。複数のパーティションを指定できますが、カンマ（,）で区切る必要があります。宛先テーブルに存在するパーティションを設定する必要があります。このパラメータを指定すると、データは指定されたパーティションのみに挿入されます。このパラメータを指定しない場合、データはすべてのパーティションに挿入されます。 |
| TEMPORARY PARTITION|データをロードする[一時的なパーティション](../../../table_design/Temporary_partition.md)の名前。複数の一時的なパーティションを指定できますが、カンマ（,）で区切る必要があります。|
| label         | データロードトランザクションごとの一意の識別ラベル。指定しない場合、システムがトランザクションごとに自動的に生成します。トランザクション時に接続エラーが発生し、結果が返されない場合は、トランザクションのステータスを確認できませんので、トランザクション時にラベルを指定することをお勧めします。ラベルの命名に関する制限については、[システム制限](../../../reference/System_limit.md)を参照してください。 |
| column_name   | データをロードする宛先の列の名前。宛先テーブルに存在する列として設定する必要があります。指定された宛先の列は、ソーステーブルの列とシーケンスで1対1にマップされ、宛先の列名に関わらずマップされます。宛先列が指定されていない場合、デフォルト値は宛先テーブルのすべての列です。指定したソーステーブルの列が宛先列に存在しない場合は、この列にはデフォルト値が書き込まれ、指定された列にデフォルト値がない場合はトランザクションが失敗します。ソーステーブルの列の型が宛先テーブルの列の型と一致しない場合、システムは一致しない列で暗黙の変換を実行します。変換に失敗すると、構文解析エラーが返されます。 |
| expression    | 列に値を割り当てる式。                |
| DEFAULT       | 列にデフォルト値を割り当てます。          |
| query         | 結果が宛先テーブルにロードされるクエリステートメント。StarRocksがサポートする任意のSQLステートメントを使用できます。 |
| FILES()       | テーブル関数[FILES()](../../sql-functions/table-functions/files.md)。この関数を使用してデータをリモートストレージにアンロードできます。詳細については、[INSERT INTO FILES()を使用してリモートストレージにデータをアンロード](../../../unloading/unload_using_insert_into_files.md)を参照してください。 |

## 戻り値

```Plain
Query OK, 5 rows affected, 2 warnings (0.05 sec)
{'label':'insert_load_test', 'status':'VISIBLE', 'txnId':'1008'}
```

| 戻り値        | 説明                                                  |
| ------------- | ------------------------------------------------------------ |
| rows affected | ロードされた行数を示します。`warnings`はフィルタリングされた行数を示します。 |
| label         | データロードトランザクションごとの一意の識別ラベル。ユーザーによって割り当てられることも、システムによって自動的に割り当てられることもあります。 |
| status        | ロードされたデータの可視性を示します。`VISIBLE`:データは正常にロードされており、可視です。`COMMITTED`:データは正常にロードされていますが、現在は不可視です。 |
| txnId         | 各INSERTトランザクションに対応するID番号です。      |

## 使用上の注意

- 現在のバージョンでは、StarRocksがINSERT INTOステートメントを実行する際、宛先テーブルの形式に一致しないデータが1行でもあると、INSERTトランザクションはデフォルトで失敗します。システムが宛先テーブルの形式に一致しないデータをフィルタリングしトランザクションを続行するようにセッション変数`enable_insert_strict`を`false`に設定できます。

- INSERT OVERWRITEステートメントが実行された後、StarRocksは元のデータを保存するパーティションに一時的なパーティションを作成し、データを一時的なパーティションに挿入し、元のパーティションを一時的なパーティションと交換します。すべての操作はリーダーFEノードで実行されます。したがって、INSERT OVERWRITEステートメントを実行中にリーダーFEノードがクラッシュした場合、ロードトランザクション全体が失敗し、一時的なパーティションが削除されます。

## 例

以下の例は、2つの列`c1`と`c2`を含む`test`という名前のテーブルを基にしています。`c2`列はDEFAULTのデフォルト値を持っています。

- `test`テーブルに1行のデータをインポートします。

```SQL
INSERT INTO test VALUES (1, 2);
INSERT INTO test (c1, c2) VALUES (1, 2);
INSERT INTO test (c1, c2) VALUES (1, DEFAULT);
INSERT INTO test (c1) VALUES (1);
```

宛先列が指定されていない場合、列はデフォルトで宛先テーブルにシーケンスでロードされます。したがって、上記の例では、最初と2番目のSQL文の結果は同じです。

宛先列（データが挿入されているかどうかに関わらず）が値としてDEFAULTを使用する場合、列はデフォルト値をロードされたデータとして使用します。したがって、上記の例では、3番目と4番目のステートメントの出力は同じです。

- `test`テーブルに複数行のデータを一度にロードします。

```SQL
INSERT INTO test VALUES (1, 2), (3, 2 + 2);
INSERT INTO test (c1, c2) VALUES (1, 2), (3, 2 * 2);
INSERT INTO test (c1, c2) VALUES (1, DEFAULT), (3, DEFAULT);
INSERT INTO test (c1) VALUES (1), (3);
```

式の結果が同等であるため、1番目と2番目のステートメントの結果は同じです。デフォルト値が使用されているため、3番目と4番目のステートメントの結果も同じです。

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

次の例は、AWS S3バケット`inserttest`内のParquetファイル**parquet/insert_wiki_edit_append.parquet**からデータ行を`insert_wiki_edit`テーブルに挿入します：

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