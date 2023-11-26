---
displayed_sidebar: "Japanese"
---

# STRUCT

## 説明

STRUCTは、複雑なデータ型を表現するために広く使用されています。これは、異なるデータ型の要素（フィールドとも呼ばれます）のコレクションを表します。たとえば、`<a INT, b STRING>`のようなものです。

STRUCT内のフィールド名は一意である必要があります。フィールドは、プリミティブなデータ型（数値、文字列、日付など）または複雑なデータ型（ARRAYやMAPなど）であることができます。

STRUCT内のフィールドは、他のSTRUCT、ARRAY、またはMAPであることもできます。これにより、ネストされたデータ構造を作成することができます。たとえば、`STRUCT<a INT, b STRUCT<c INT, d INT>, c MAP<INT, INT>, d ARRAY<INT>>`のようなものです。

STRUCTデータ型はv3.1以降でサポートされています。v3.1では、StarRocksテーブルを作成し、そのテーブルにSTRUCTデータをロードし、MAPデータをクエリすることができます。

v2.5以降、StarRocksはデータレイクからの複雑なデータ型MAPおよびSTRUCTのクエリをサポートしています。StarRocksが提供する外部カタログを使用して、Apache Hive™、Apache Hudi、およびApache IcebergからMAPおよびSTRUCTデータをクエリすることができます。ORCおよびParquetファイルからのデータのみクエリできます。外部データソースをクエリするための外部カタログの使用方法についての詳細は、[カタログの概要](../../../data_source/catalog/catalog_overview.md)および関連するトピックを参照してください。

## 構文

```Haskell
STRUCT<name, type>
```

- `name`: フィールド名。CREATE TABLEステートメントで定義された列名と同じである必要があります。
- `type`: フィールドの型。サポートされている任意の型であることができます。

## StarRocksでSTRUCT列を定義する

テーブルを作成し、この列にSTRUCTデータをロードするときに、STRUCT列を定義することができます。

```SQL
-- 1次元のSTRUCTを定義します。
CREATE TABLE t0(
  c0 INT,
  c1 STRUCT<a INT, b INT>
)
DUPLICATE KEY(c0);

-- 複雑なSTRUCTを定義します。
CREATE TABLE t1(
  c0 INT,
  c1 STRUCT<a INT, b STRUCT<c INT, d INT>, c MAP<INT, INT>, d ARRAY<INT>>
)
DUPLICATE KEY(c0);

-- NOT NULLのSTRUCTを定義します。
CREATE TABLE t2(
  c0 INT,
  c1 STRUCT<a INT, b INT> NOT NULL
)
DUPLICATE KEY(c0);
```

STRUCT型の列には、次の制限があります。

- テーブルのキーカラムとして使用することはできません。値カラムとしてのみ使用できます。
- テーブルのパーティションキーカラム（PARTITION BYに続くもの）として使用することはできません。
- テーブルのバケット化カラム（DISTRIBUTED BYに続くもの）として使用することはできません。
- [集計テーブル](../../../table_design/table_types/aggregate_table.md)の値カラムとして使用する場合、replace()関数のみサポートされます。

## SQLでSTRUCTを構築する

STRUCTは、SQLで以下の関数を使用して構築することができます：[row, struct](../../sql-functions/struct-functions/row.md)、および[named_struct](../../sql-functions/struct-functions/named_struct.md)。struct()はrow()のエイリアスです。

- `row`および`struct`は、無名のstructをサポートしています。フィールド名を指定する必要はありません。StarRocksは自動的に列名を生成します（`col1`、`col2`など）。
- `named_struct`は、名前付きのstructをサポートしています。名前と値の式はペアである必要があります。

StarRocksは、入力値に基づいてstructの型を自動的に決定します。

```SQL
select row(1, 2, 3, 4) as numbers; -- {"col1":1,"col2":2,"col3":3,"col4":4}を返します。
select row(1, 2, null, 4) as numbers; -- {"col1":1,"col2":2,"col3":null,"col4":4}を返します。
select row(null) as nulls; -- {"col1":null}を返します。
select struct(1, 2, 3, 4) as numbers; -- {"col1":1,"col2":2,"col3":3,"col4":4}を返します。
select named_struct('a', 1, 'b', 2, 'c', 3, 'd', 4) as numbers; -- {"a":1,"b":2,"c":3,"d":4}を返します。
```

## STRUCTデータのロード

STRUCTデータをStarRocksにロードするには、[INSERT INTO](../../../loading/InsertInto.md)と[ORC/Parquet loading](../data-manipulation/BROKER_LOAD.md)の2つの方法があります。

StarRocksは、データ型を対応するSTRUCT型に自動的にキャストします。

### INSERT INTO

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

### ORC/ParquetファイルからSTRUCTデータをロード

StarRocksのSTRUCTデータ型は、ORCまたはParquet形式のネストされた列構造に対応しています。追加の指定は必要ありません。ORCまたはParquetファイルからSTRUCTデータをロードするには、[ORC/Parquet loading](../data-manipulation/BROKER_LOAD.md)の手順に従ってください。

## STRUCTフィールドへのアクセス

STRUCTのサブフィールドをクエリするには、ドット（`.`）演算子を使用してフィールド名によって値をクエリするか、インデックスによって値を呼び出すために`[]`を使用することができます。

```Plain Text
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
