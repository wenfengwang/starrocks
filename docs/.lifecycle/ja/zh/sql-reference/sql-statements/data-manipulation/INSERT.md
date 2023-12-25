---
displayed_sidebar: Chinese
---

# INSERT

## 機能

StarRocks のテーブルにデータを挿入または上書きする。このデータインポート方法が適用されるシナリオについては、[INSERT INTO インポート](../../../loading/InsertInto.md)を参照してください。

[SUBMIT TASK](./SUBMIT_TASK.md) を使用して、非同期の INSERT タスクを作成できます。

## 文法

- **インポート**:

  ```sql
  INSERT { INTO | OVERWRITE } [db_name.]<table_name>
  [ PARTITION (<partition_name> [, ...] ) ]
  [ TEMPORARY PARTITION (<temporary_partition_name> [, ...] ) ]
  [ WITH LABEL <label>]
  [ (<column_name>[, ...]) ]
  { VALUES ( { <expression> | DEFAULT } [, ...] ) | <query> }
  ```

- **エクスポート**:

  ```sql
  INSERT INTO FILES()
  [ WITH LABEL <label> ]
  { VALUES ( { <expression> | DEFAULT } [, ...] ) | <query> }
  ```

## パラメータ説明

| **パラメータ**    | **説明**                                                     |
| ----------- | ------------------------------------------------------------ |
| INTO        | データをターゲットテーブルに追加します。                                       |
| OVERWRITE   | データをターゲットテーブルに上書きします。                                       |
| table_name  | インポートデータのターゲットテーブル。`db_name.table_name` 形式でも可能です。         |
| PARTITION  | インポートのターゲットパーティション。このパラメータは、ターゲットテーブルに存在するパーティションでなければなりません。複数のパーティション名はコンマ（,）で区切ります。このパラメータを指定すると、データは指定されたパーティションにのみインポートされます。指定しない場合は、デフォルトでターゲットテーブルのすべてのパーティションにデータがインポートされます。 |
| TEMPORARY PARTITION | [一時パーティション](../../../table_design/Temporary_partition.md)にデータをインポートするパーティションを指定します。|
| label       | インポートジョブの識別子で、データベース内で一意です。指定しない場合、StarRocks は自動的にジョブにラベルを生成します。ラベルを指定することをお勧めします。そうでないと、現在のインポートジョブがネットワークエラーで結果を返せない場合、インポート操作が成功したかどうかを知ることができません。ラベルを指定した場合は、SQL コマンド `SHOW LOAD WHERE label="label";` でタスク結果を確認できます。ラベルの命名制限については、[システム制限](../../../reference/System_limit.md)を参照してください。 |
| column_name | インポートのターゲット列で、ターゲットテーブルに存在する列でなければなりません。このパラメータは列名とは無関係ですが、順序に従って対応しています。ターゲット列を指定しない場合、デフォルトでターゲットテーブルのすべての列が対象になります。ソーステーブルの列がターゲット列に存在しない場合は、デフォルト値が書き込まれます。現在の列にデフォルト値がない場合、インポートジョブは失敗します。クエリ結果の列の型がターゲット列の型と一致しない場合は、暗黙の型変換が行われます。変換できない場合は、INSERT INTO ステートメントは構文解析エラーを報告します。 |
| expression  | 対応する列に値を割り当てるための式。                                   |
| DEFAULT     | 対応する列にデフォルト値を割り当てます。                                         |
| query       | クエリステートメントで、その結果はターゲットテーブルにインポートされます。StarRocks がサポートする任意の SQL クエリ構文が使用できます。 |
| FILES()       | テーブル関数 [FILES()](../../sql-functions/table-functions/files.md)。この関数を使用して、データをリモートストレージにエクスポートできます。詳細については、[INSERT INTO FILES() を使用したデータのエクスポート](../../../unloading/unload_using_insert_into_files.md)を参照してください。 |

## 注意事項

- 現行バージョンでは、StarRocks が INSERT ステートメントを実行する際、ターゲットテーブルのフォーマットに適合しないデータ（例えば文字列が長すぎるなど）がある場合、デフォルトでは INSERT 操作は失敗します。セッション変数 `enable_insert_strict` を `false` に設定することで、ターゲットテーブルのフォーマットに適合しないデータをフィルタリングし、操作を続行することができます。

- INSERT OVERWRITE ステートメントを実行した後、システムはターゲットパーティションに対応する一時パーティションを作成し、データを一時パーティションに書き込み、最終的に一時パーティションを使用してターゲットパーティションを原子的に置き換えて上書きを実現します。このプロセスはすべて Leader FE ノードで実行されます。したがって、Leader FE ノードが上書きプロセス中にダウンした場合、その INSERT OVERWRITE インポートは失敗し、プロセス中に作成された一時パーティションも削除されます。

## 例

以下の例は `test` テーブルを基にしており、`c1` と `c2` の2つの列が含まれています。`c2` 列にはデフォルト値 DEFAULT があります。

### 例1：test テーブルに1行のデータをインポート

```SQL
INSERT INTO test VALUES (1, 2);
INSERT INTO test (c1, c2) VALUES (1, 2);
INSERT INTO test (c1, c2) VALUES (1, DEFAULT);
INSERT INTO test (c1) VALUES (1);
```

- ターゲット列を指定しない場合、テーブルの列の順序がデフォルトのターゲット列のインポート順序として使用されます。したがって、上記の例では、最初と2番目の SQL ステートメントのインポート効果は同じです。
- ターゲット列にデータが挿入されていない場合や DEFAULT を値として使用する場合、その列はデフォルト値を使用してデータがインポートされます。したがって、上記の例では、3番目と4番目のステートメントのインポート効果は同じです。

### 例2：test テーブルに一度に複数行のデータをインポート

```SQL
INSERT INTO test VALUES (1, 2), (3, 2 + 2);
INSERT INTO test (c1, c2) VALUES (1, 2), (3, 2 * 2);
INSERT INTO test (c1) VALUES (1), (3);
INSERT INTO test (c1, c2) VALUES (1, DEFAULT), (3, DEFAULT);
```

- 式の結果が同じであるため、上記の例では、最初と2番目の SQL ステートメントのインポート効果は同じです。
- 3番目と4番目のステートメントは DEFAULT を値として使用しているため、インポート効果は同じです。

### 例3：test テーブルにクエリ結果をインポート

```SQL
INSERT INTO test SELECT * FROM test2;
INSERT INTO test (c1, c2) SELECT * from test2;
```

### 例4：test テーブルにクエリ結果をインポートし、パーティションとラベルを指定

```SQL
INSERT INTO test PARTITION(p1, p2) WITH LABEL `label1` SELECT * FROM test2;
INSERT INTO test WITH LABEL `label1` (c1, c2) SELECT * from test2;
```

### 例5：test テーブルにクエリ結果を上書きインポートし、パーティションとラベルを指定

```SQL
INSERT OVERWRITE test PARTITION(p1, p2) WITH LABEL `label1` SELECT * FROM test3;
INSERT OVERWRITE test WITH LABEL `label1` (c1, c2) SELECT * from test3;
```

### 例6：AWS S3 から Parquet データファイルをインポート

以下の例では、AWS S3 バケット `inserttest` 内の Parquet ファイル **parquet/insert_wiki_edit_append.parquet** からデータを `insert_wiki_edit` テーブルにインポートします：

```Plain
INSERT INTO insert_wiki_edit
    SELECT * FROM TABLE(
        "path" = "s3://inserttest/parquet/insert_wiki_edit_append.parquet",
        "format" = "parquet",
        "aws.s3.access_key" = "xxxxxxxxxx",
        "aws.s3.secret_key" = "yyyyyyyyyy",
        "aws.s3.region" = "aa-bbbb-c"
);
```
