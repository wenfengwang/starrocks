---
displayed_sidebar: "Japanese"
---

# INSERT

## 説明

特定のテーブルにデータを挿入したり、特定のテーブルにデータを上書きします。詳細な情報は、[INSERT を使用したデータの読み込み](../../../loading/InsertInto.md)を参照してください。v3.2.0以降、INSERTはリモートストレージにデータを書き込むことができます。StarRocksからリモートストレージにデータをアンロードするには、[INSERT INTO FILES() を使用します](../../../unloading/unload_using_insert_into_files.md)。

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

| **パラメータ** | 説明                                                    |
| ------------- | -------------------------------------------------------- |
| INTO          | テーブルにデータを追加する。                               |
| OVERWRITE     | テーブルをデータで上書きする。                            |
| table_name    | データをロードするテーブルの名前。`db_name.table_name`として指定することができます。 |
| PARTITION    | データをロードするパーティション。複数のパーティションを指定できます。カンマ(,)で区切る必要があります。指定するパーティションは、宛先のテーブルに存在する必要があります。このパラメータを指定すると、データは指定されたパーティション内にのみ挿入されます。このパラメータを指定しない場合、データはすべてのパーティションに挿入されます。 |
| TEMPORARY PARTITION|データをロードする[一時パーティション](../../../table_design/Temporary_partition.md)の名前。複数の一時パーティションを指定できます。カンマ(,)で区切る必要があります。|
| label         | データのロードトランザクションごとのユニークな識別ラベル。指定しないとシステムがトランザクションのために自動的に生成します。トランザクションのステータスを確認するには、このラベルを指定することをお勧めします。そうしないと、接続エラーが発生し結果が返らない場合に、トランザクションステータスを確認できません。`SHOW LOAD WHERE label="label"`ステートメントを使用してトランザクションのステータスを確認できます。ラベルの名前の制限については、[システム制限](../../../reference/System_limit.md)を参照してください。 |
| column_name   | データをロードする宛先カラムの名前。宛先のテーブルに存在する必要があります。指定する宛先カラムは、ソーステーブルのカラムと同じ順序で1対1にマッピングされます。宛先カラムが指定されていない場合、デフォルト値は宛先テーブルのすべてのカラムになります。ソーステーブルの指定されたカラムが宛先カラムに存在しない場合、このカラムにはデフォルト値が書き込まれ、指定されたカラムにデフォルト値がない場合はトランザクションが失敗します。ソーステーブルの列の型が宛先テーブルの列の型と一致しない場合、システムは不一致の列に対して暗黙の変換を行います。変換に失敗すると、構文解析エラーが返されます。 |
| expression    | カラムに値を割り当てる式。                                   |
| DEFAULT       | カラムにデフォルト値を割り当てる。                           |
| query         | クエリステートメントの結果を宛先テーブルにロードする。StarRocksでサポートされている任意のSQLステートメントであることができます。 |
| FILES()       | テーブル関数 [FILES()](../../sql-functions/table-functions/files.md)。この関数を使用してデータをリモートストレージにアンロードできます。詳細については、[INSERT INTO FILES() を使用してリモートストレージにデータをアンロードする](../../../unloading/unload_using_insert_into_files.md)を参照してください。 |

## 戻り値

```Plain
Query OK, 5 rows affected, 2 warnings (0.05 sec)
{'label':'insert_load_test', 'status':'VISIBLE', 'txnId':'1008'}
```

| 戻り値        | 説明                                                    |
| ------------- | -------------------------------------------------------- |
| rows affected | ロードされた行数を示します。`warnings`はフィルタリングされた行数を示します。 |
| label         | データのロードトランザクションごとのユニークな識別ラベル。ユーザーによって割り当てられることも、システムによって自動的に割り当てられることもあります。 |
| status        | ロードされたデータの表示状態を示します。`VISIBLE` :データは正常にロードされ、表示されます。`COMMITTED` :データは正常にロードされましたが現在は表示されません。 |
| txnId         | 各INSERTトランザクションに対応するID番号。              |

## 使用上の注意

- 現在のバージョンでは、StarRocksがINSERT INTOステートメントを実行する際、宛先テーブルの形式にデータが一部不一致する場合（たとえば、文字列が長すぎる場合など）、INSERTトランザクションはデフォルトで失敗します。セッション変数`enable_insert_strict`を`false`に設定すると、宛先テーブル形式に一部不一致するデータをシステムがフィルタリングし、トランザクションを継続することができます。

- INSERT OVERWRITEステートメントが実行された後、StarRocksは元のデータを保持するパーティションのための一時的なパーティションを作成し、データを一時的なパーティションに挿入し、元のパーティションと一時的なパーティションを入れ替えます。これらのすべての操作はLeader FEノードで実行されます。したがって、INSERT OVERWRITEステートメントの実行中にリーダーFEノードがクラッシュすると、ロードトランザクション全体が失敗し、一時的なパーティションが削除されます。

## 例

以下の例は、2つのカラム`c1`と`c2`を含む`test`というテーブルを基にしています。`c2`にはDEFAULTのデフォルト値が設定されています。

- データを`test`テーブルに1行挿入します。

```SQL
INSERT INTO test VALUES (1, 2);
INSERT INTO test (c1, c2) VALUES (1, 2);
INSERT INTO test (c1, c2) VALUES (1, DEFAULT);
INSERT INTO test (c1) VALUES (1);
```

宛先カラムが指定されていない場合、デフォルトで列は宛先テーブルにシーケンス順にロードされます。したがって、上記の例の1番目と2番目のSQLステートメントの結果は同じです。

宛先カラム（データが挿入されている場合はデフォルト値を使用している場合も含む）が値としてDEFAULTを使用する場合、そのカラムにはデフォルト値がロードされます。したがって、上記の例の3番目と4番目のステートメントの結果は同じです。

- 複数の行のデータを`test`テーブルに一度にロードします。

```SQL
INSERT INTO test VALUES (1, 2), (3, 2 + 2);
INSERT INTO test (c1, c2) VALUES (1, 2), (3, 2 * 2);
INSERT INTO test (c1, c2) VALUES (1, DEFAULT), (3, DEFAULT);
INSERT INTO test (c1) VALUES (1), (3);
```

式の結果が同等であるため、1番目と2番目のステートメントの結果は同じです。デフォルト値を使用しているため、3番目と4番目のステートメントの結果も同じです。

- クエリステートメントの結果を`test`テーブルにインポートします。

```SQL
INSERT INTO test SELECT * FROM test2;
INSERT INTO test (c1, c2) SELECT * from test2;
```

- クエリの結果を`test`テーブルにインポートし、パーティションとラベルを指定します。

```SQL
INSERT INTO test PARTITION(p1, p2) WITH LABEL `label1` SELECT * FROM test2;
INSERT INTO test WITH LABEL `label1` (c1, c2) SELECT * from test2;
```

- クエリの結果で`test`テーブルを上書きし、パーティションとラベルを指定します。

```SQL
INSERT OVERWRITE test PARTITION(p1, p2) WITH LABEL `label1` SELECT * FROM test3;
INSERT OVERWRITE test WITH LABEL `label1` (c1, c2) SELECT * from test3;
```

次の例は、AWS S3バケット`inserttest`内のParquetファイル **parquet/insert_wiki_edit_append.parquet** からデータ行を`insert_wiki_edit`テーブルに挿入します。

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