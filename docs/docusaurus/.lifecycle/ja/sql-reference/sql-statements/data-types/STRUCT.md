---
displayed_sidebar: "Japanese"
---

# STRUCT

## 説明

STRUCTは、複雑なデータ型を表現するために広く使用されています。これは、例えば、`<a INT, b STRING>`のような異なるデータ型の要素（またはフィールドとも呼ばれます）のコレクションを表します。

STRUCT内のフィールド名はユニークである必要があります。フィールドはプリミティブデータ型（数値、文字列、日付など）または複雑なデータ型（ARRAYやMAPなど）であることができます。

STRUCT内のフィールドは、別のSTRUCT、ARRAY、またはMAPであることもでき、これにより入れ子になったデータ構造を作成することができます。例えば、`STRUCT<a INT, b STRUCT<c INT, d INT>, c MAP<INT, INT>, d ARRAY<INT>>`のようになります。

STRUCTデータ型はv3.1からサポートされています。v3.1では、StarRocksテーブルを作成し、そのテーブルにSTRUCTデータをロードし、MAPデータをクエリすることができます。

v2.5以降、StarRocksはデータレイクから複雑なデータ型MAPおよびSTRUCTのクエリをサポートしています。Apache Hive™、Apache Hudi、Apache Icebergから提供されているStarRocksの外部カタログを使用して、MAPおよびSTRUCTデータをクエリすることができます。ORCおよびParquetファイルからのみデータをクエリすることができます。外部カタログを使用して外部データソースをクエリする方法の詳細については、[カタログの概要](../../../data_source/catalog/catalog_overview.md)および必要なカタログタイプに関連するトピックを参照してください。

## 構文

```Haskell
STRUCT<name, type>
```

- `name`: フィールド名。CREATE TABLEステートメントで定義されたカラム名と同じです。
- `type`: フィールドの型。サポートされている任意の型であることができます。

## StarRocksでのSTRUCTカラムの定義

テーブルを作成し、このカラムにSTRUCTデータをロードする際にSTRUCTカラムを定義することができます。

```SQL
-- 一次元のstructを定義する。
CREATE TABLE t0(
  c0 INT,
  c1 STRUCT<a INT, b INT>
)
DUPLICATE KEY(c0);

-- 複雑なstructを定義する。
CREATE TABLE t1(
  c0 INT,
  c1 STRUCT<a INT, b STRUCT<c INT, d INT>, c MAP<INT, INT>, d ARRAY<INT>>
)
DUPLICATE KEY(c0);

-- NOT NULLなstructを定義する。
CREATE TABLE t2(
  c0 INT,
  c1 STRUCT<a INT, b INT> NOT NULL
)
DUPLICATE KEY(c0);
```

STRUCT型のカラムには次の制限があります:

- テーブルでキーカラムとして使用することはできません。値カラムとしてのみ使用できます。
- テーブルでのパーティションキーカラム（PARTITION BYに続く）として使用することはできません。
- テーブルでのバケティングカラム（DISTRIBUTED BYに続く）として使用することはできません。
- [集計テーブル](../../../table_design/table_types/aggregate_table.md)で値カラムとして使用する場合、replace()関数のみをサポートしています。

## SQLでのstructの構築

SQLでは、[row, struct](../../sql-functions/struct-functions/row.md)および[named_struct](../../sql-functions/struct-functions/named_struct.md)という関数を使用してSTRUCTを構築することができます。struct()はrow()のエイリアスです。

- `row`および`struct`は、無名のstructをサポートします。フィールド名を指定する必要はありません。StarRocksは自動的にカラム名（`col1`、`col2`など）を生成します。
- `named_struct`は、名前付きstructをサポートします。名前および値の式はペアで指定する必要があります。

StarRocksは、入力値に基づいてstructの型を自動的に決定します。

```SQL
select row(1, 2, 3, 4) as numbers; -- {"col1":1,"col2":2,"col3":3,"col4":4}を返します。
select row(1, 2, null, 4) as numbers; -- {"col1":1,"col2":2,"col3":null,"col4":4}を返します。
select row(null) as nulls; -- {"col1":null}を返します。
select struct(1, 2, 3, 4) as numbers; -- {"col1":1,"col2":2,"col3":3,"col4":4}を返します。
select named_struct('a', 1, 'b', 2, 'c', 3, 'd', 4) as numbers; -- {"a":1,"b":2,"c":3,"d":4}を返します。
```

## STRUCTデータのロード

STRUCTデータをStarRocksにロードする方法は、[INSERT INTO](../../../loading/InsertInto.md)と[ORC/Parquet loading](../data-manipulation/BROKER_LOAD.md)の2つの方法があります。

StarRocksはデータの型を対応するSTRUCT型に自動的にキャストします。

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

StarRocksのSTRUCTデータ型は、ORCまたはParquet形式の入れ子のカラム構造に対応しています。追加仕様は必要ありません。[ORC/Parquet loading](../data-manipulation/BROKER_LOAD.md)の手順に従って、ORCまたはParquetファイルからSTRUCTデータをロードすることができます。

## STRUCTフィールドのアクセス

STRUCTのサブフィールドをクエリするには、そのフィールド名によって値をクエリするためにドット（`.`）演算子を使用するか、インデックスによって値を呼び出すために`[]`を使用することができます。

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