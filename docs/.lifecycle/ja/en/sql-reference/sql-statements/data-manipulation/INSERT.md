---
displayed_sidebar: English
---

# INSERT

## 説明

特定のテーブルにデータを挿入するか、特定のテーブルをデータで上書きします。アプリケーションシナリオの詳細については、[INSERTを使用したデータのロード](../../../loading/InsertInto.md)を参照してください。v3.2.0以降、INSERTはリモートストレージ内のファイルへのデータの書き込みをサポートします。[INSERT INTO FILES()を使用して、StarRocksからリモートストレージにデータをアンロードする](../../../unloading/unload_using_insert_into_files.md)ことができます。

[SUBMIT_TASK](./SUBMIT_TASK.md)を使用して、非同期INSERTタスクをサブミットできます。

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

## パラメーター

| **パラメーター** | 説明                                                  |
| ------------- | ------------------------------------------------------------ |
| INTO          | テーブルにデータを追加します。                                 |
| OVERWRITE     | テーブルをデータで上書きします。                            |
| table_name    | データをロードするテーブルの名前。`db_name.table_name`としてデータベースとテーブルを指定できます。 |
| PARTITION     | データをロードするパーティション。複数のパーティションをコンマ (,) で区切ることができます。宛先テーブルに存在するパーティションに設定する必要があります。このパラメーターを指定すると、データは指定されたパーティションにのみ挿入されます。このパラメーターを指定しない場合、データはすべてのパーティションに挿入されます。 |
| TEMPORARY PARTITION|データをロードする[一時パーティション](../../../table_design/Temporary_partition.md)の名前。複数の一時パーティションをコンマ (,) で区切ることができます。|
| LABEL         | データベース内の各データロードトランザクションの一意の識別ラベル。指定しない場合、システムがトランザクション用に自動的に生成します。トランザクションのラベルを指定することを推奨します。そうでないと、接続エラーが発生して結果が返されない場合に、トランザクションの状態を確認できません。トランザクションの状態は`SHOW LOAD WHERE label="label"`ステートメントで確認できます。ラベルの命名に関する制限については、[システム制限](../../../reference/System_limit.md)を参照してください。 |
| column_name   | データをロードする宛先列の名前。宛先テーブルに存在する列として設定する必要があります。指定した宛先列は、宛先列の名前に関係なく、ソーステーブルの列に1対1で順番にマッピングされます。宛先列が指定されていない場合、デフォルト値は宛先テーブルのすべての列です。ソーステーブルの指定された列が宛先列に存在しない場合、デフォルト値がこの列に書き込まれ、指定された列にデフォルト値がない場合、トランザクションは失敗します。ソーステーブルの列タイプが宛先テーブルの列タイプと矛盾する場合、システムは不一致の列に対して暗黙の変換を実行します。変換に失敗すると、構文解析エラーが返されます。 |
| expression    | 列に値を割り当てる式です。                |
| DEFAULT       | 列にデフォルト値を割り当てます。                         |
| query         | 結果が宛先テーブルにロードされるクエリステートメントです。StarRocksがサポートする任意のSQLステートメントにすることができます。 |
| FILES()       | テーブル関数[FILES()](../../sql-functions/table-functions/files.md)です。この関数を使用して、リモートストレージにデータをアンロードすることができます。詳細については、[INSERT INTO FILES()を使用してリモートストレージにデータをアンロードする](../../../unloading/unload_using_insert_into_files.md)を参照してください。 |

## 戻り値

```Plain
Query OK, 5 rows affected, 2 warnings (0.05 sec)
{'label':'insert_load_test', 'status':'VISIBLE', 'txnId':'1008'}
```

| 戻り値        | 説明                                                  |
| ------------- | ------------------------------------------------------------ |
| rows affected | 読み込まれる行数を示します。`warnings`は、フィルターで除外される行を示します。 |
| label         | データベース内の各データロードトランザクションの一意の識別ラベルです。ユーザーによって割り当てられるか、システムによって自動的に割り当てられます。 |
| status        | 読み込まれたデータが表示されるかどうかを示します。`VISIBLE`は、データが正常にロードされて表示されています。`COMMITTED`は、データが正常にロードされていますが、現時点では表示されていません。 |
| txnId         | 各INSERTトランザクションに対応するID番号です。      |

## 使用上の注意

- 現在のバージョンでは、StarRocksがINSERT INTOステートメントを実行する際に、データ行が宛先テーブルの形式と一致しない場合（たとえば、文字列が長すぎる場合）、INSERTトランザクションはデフォルトで失敗します。セッション変数`enable_insert_strict`を`false`に設定して、宛先テーブルの形式と一致しないデータを除外し、トランザクションの実行を続行することができます。

- INSERT OVERWRITEステートメントを実行した後、StarRocksは元のデータを格納するパーティションのための一時パーティションを作成し、一時パーティションにデータを挿入し、元のパーティションと一時パーティションを交換します。これらの操作はすべてLeader FEノードで実行されます。したがって、INSERT OVERWRITEステートメントを実行中にLeader FEノードがクラッシュすると、ロードトランザクション全体が失敗し、一時パーティションが削除されます。

## 例

以下の例は、`c1`と`c2`の2つのカラムを含むテーブル`test`に基づいています。`c2`カラムにはデフォルト値が設定されています。

- `test`テーブルにデータ行をインポートします。

```SQL
INSERT INTO test VALUES (1, 2);
INSERT INTO test (c1, c2) VALUES (1, 2);
INSERT INTO test (c1, c2) VALUES (1, DEFAULT);
INSERT INTO test (c1) VALUES (1);
```

宛先列が指定されていない場合、列はデフォルトで宛先テーブルに順番にロードされます。したがって、上記の例では、最初と二番目のSQLステートメントの結果は同じです。

宛先列（データが挿入されているかどうかに関わらず）が値としてDEFAULTを使用する場合、その列はデフォルト値をロードされたデータとして使用します。したがって、上記の例では、三番目と四番目のステートメントの出力は同じです。

- 一度に複数のデータ行を`test`テーブルにロードします。

```SQL
INSERT INTO test VALUES (1, 2), (3, 2 + 2);
INSERT INTO test (c1, c2) VALUES (1, 2), (3, 2 * 2);
INSERT INTO test (c1, c2) VALUES (1, DEFAULT), (3, DEFAULT);
INSERT INTO test (c1) VALUES (1), (3);
```

式の結果が等しいため、最初と二番目のステートメントの結果は同じです。三番目と四番目のステートメントの結果は、どちらもデフォルト値を使用しているため同じです。

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

次の例では、AWS S3バケット`inserttest`内のParquetファイル**parquet/insert_wiki_edit_append.parquet**からデータ行をテーブル`insert_wiki_edit`に挿入します：

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
