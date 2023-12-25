---
displayed_sidebar: Chinese
---

# STRUCT

## 説明

STRUCTは複雑なデータ型で、異なるデータ型の要素（フィールドとも呼ばれる）を格納できます。例えば `<a INT, b STRING>` のように。

Structのフィールド名は重複することはできません。フィールドは基本データ型（Primitive Type）、例えば数値、文字列、日付型など、または複雑なデータ型、例えばARRAYやMAPも可能です。

Structのフィールドは、別のStruct、Map、またはArrayを指定でき、ユーザーがネストされたデータ構造を定義するのに便利です。例：`STRUCT<a INT, b STRUCT<c INT, d INT>, c MAP<INT, INT>, d ARRAY<INT>>`。

StarRocksはバージョン3.1からSTRUCTデータ型の格納とインポートをサポートしています。テーブル作成時にSTRUCT列を定義し、STRUCTデータをテーブルにインポートし、STRUCTデータをクエリすることができます。

StarRocksはバージョン2.5から、データレイク内の複雑なデータ型MAPとSTRUCTのクエリをサポートしています。StarRocksが提供するExternal Catalogを通じて、Apache Hive™、Apache Hudi、Apache Iceberg内のMAPとSTRUCTデータをクエリできます。ORCおよびParquet形式のファイルのみクエリがサポートされています。

External Catalogを使用して外部データソースをクエリする方法については、[Catalog 概要](../../../data_source/catalog/catalog_overview.md)および対応するCatalogドキュメントを参照してください。

## 構文

```SQL
STRUCT<name type>
```

- `name`：フィールド名。テーブル作成ステートメントの列名と同じです。
- `type`：フィールドの型。StarRocksがサポートする任意の型が可能です。

## STRUCT型列の定義

テーブル作成時にCREATE TABLEステートメントでSTRUCT型の列を定義し、その後STRUCTデータをその列にインポートできます。

```SQL
-- シンプルなstructを定義。
CREATE TABLE t0(
  c0 INT,
  c1 STRUCT<a INT, b INT>
)
DUPLICATE KEY(c0);

-- 複雑なstructを定義。
CREATE TABLE t1(
  c0 INT,
  c1 STRUCT<a INT, b STRUCT<c INT, d INT>, c MAP<INT, INT>, d ARRAY<INT>>
)
DUPLICATE KEY(c0);

-- NULL非許容のstructを定義。
CREATE TABLE t2(
  c0 INT,
  c1 STRUCT<a INT, b INT> NOT NULL
)
DUPLICATE KEY(c0);
```

STRUCT列には以下の使用制限があります：

- テーブルのKey列としては使用できず、Value列としてのみ使用可能です。
- テーブルのパーティション列（PARTITION BYで定義される列）としては使用できません。
- テーブルのバケット列（DISTRIBUTED BYで定義される列）としては使用できません。
- STRUCTが[集約モデルテーブル](../../../table_design/table_types/aggregate_table.md)のValue列として使用される場合、集約関数としてはreplace()のみがサポートされます。

## SQLでSTRUCTを構築

[row/struct](../../sql-functions/struct-functions/row.md)または[named_struct](../../sql-functions/struct-functions/named_struct.md)関数を使用してStructを構築できます。

- `row`と`struct`は機能が同じで、フィールド名が指定されていないunnamed structをサポートしています。フィールド名はStarRocksによって自動生成されます。例：`col1, col2,...colN`。
- `named_struct`はフィールド名が指定されたnamed structをサポートしています。KeyとValueの式はペアでなければならず、そうでない場合は構築に失敗します。

StarRocksは入力された値に基づいてStructのデータ型を自動的に決定します。

```SQL
select row(1, 2, 3, 4) as numbers; -- {"col1":1,"col2":2,"col3":3,"col4":4}を返します。
select row(1, 2, null, 4) as numbers; -- {"col1":1,"col2":2,"col3":null,"col4":4}を返します。
select row(null) as nulls; -- {"col1":null}を返します。
select struct(1, 2, 3, 4) as numbers; -- {"col1":1,"col2":2,"col3":3,"col4":4}を返します。
select named_struct('a', 1, 'b', 2, 'c', 3, 'd', 4) as numbers; -- {"a":1,"b":2,"c":3,"d":4}を返します。
```

## STRUCT型データのインポート

[INSERT INTO](../data-manipulation/INSERT.md)と[ORC/Parquetファイルインポート](../data-manipulation/BROKER_LOAD.md)の2つの方法でSTRUCTデータをStarRocksにインポートできます。

インポートプロセス中にStarRocksは重複するKey値を削除します。

### INSERT INTOでインポート

```SQL
CREATE TABLE t0(
  c0 INT,
  c1 STRUCT<a INT, b INT>
)
DUPLICATE KEY(c0);

INSERT INTO t0 VALUES(1, row(1, 1));

SELECT * FROM t0;
+------+---------------+
| c0   | c1            |
+------+---------------+
|    1 | {"a":1,"b":1} |
+------+---------------+
```

### ORCおよびParquetファイルからのインポート

StarRocksのSTRUCT型はORCおよびParquet形式のネストされた列構造（nested columns structure）に対応しており、追加の変換や定義を行う必要はありません。具体的なインポート操作については、[Broker Loadドキュメント](../data-manipulation/BROKER_LOAD.md)を参照してください。

## STRUCTデータのクエリ

`.`演算子を使用して指定されたフィールドの値をクエリするか、`[]`を使用して指定されたインデックスの値を返します。

```Plain
mysql> select named_struct('a', 1, 'b', 2, 'c', 3, 'd', 4).a;
+------------------------------------------------+
| named_struct('a', 1, 'b', 2, 'c', 3, 'd', 4).a |
+------------------------------------------------+
| 1                                              |
+------------------------------------------------+

mysql> select row(1, 2, 3, 4).col1;
+-----------------------+
| row(1, 2, 3, 4).col1  |
+-----------------------+
| 1                     |
+-----------------------+

mysql> select row(2, 4, 6, 8)[2];
+--------------------+
| row(2, 4, 6, 8)[2] |
+--------------------+
|                  4 |
+--------------------+

mysql> select row(map{'a':1}, 2, 3, 4)[1];
+-----------------------------+
| row(map{'a':1}, 2, 3, 4)[1] |
+-----------------------------+
| {"a":1}                     |
+-----------------------------+
```
