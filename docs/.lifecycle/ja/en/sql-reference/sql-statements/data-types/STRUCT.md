---
displayed_sidebar: English
---

# STRUCT

## 説明

STRUCTは、複雑なデータ型を表現するために広く使用されています。これは、異なるデータ型の要素（フィールドとも呼ばれます）のコレクションを表します。例えば、`<a INT, b STRING>`のように。

STRUCTのフィールド名は一意でなければなりません。フィールドはプリミティブデータ型（数値、文字列、日付など）や複合データ型（ARRAYやMAPなど）を持つことができます。

STRUCT内のフィールドは、別のSTRUCT、ARRAY、またはMAPにすることもでき、これによりネストされたデータ構造を作成できます。例えば、`STRUCT<a INT, b STRUCT<c INT, d INT>, c MAP<INT, INT>, d ARRAY<INT>>`のように。

STRUCTデータ型はv3.1以降でサポートされています。v3.1では、StarRocksテーブルを作成する際にSTRUCT列を定義し、そのテーブルにSTRUCTデータをロードし、MAPデータをクエリすることができます。

v2.5以降、StarRocksはデータレイクからの複雑なデータ型MAPおよびSTRUCTのクエリをサポートしています。StarRocksが提供する外部カタログを使用して、Apache Hive™、Apache Hudi、Apache IcebergからMAPおよびSTRUCTデータをクエリできます。ORCおよびParquetファイルからのデータのみクエリできます。外部カタログを使用して外部データソースをクエリする方法の詳細については、[カタログの概要](../../../data_source/catalog/catalog_overview.md)および必要なカタログタイプに関連するトピックを参照してください。

## 構文

```Haskell
STRUCT<name, type>
```

- `name`: フィールド名で、CREATE TABLEステートメントで定義されたカラム名と同じです。
- `type`: フィールドタイプで、サポートされている任意のタイプが可能です。

## StarRocksでSTRUCTカラムを定義する

テーブルを作成する際にSTRUCTカラムを定義し、このカラムにSTRUCTデータをロードすることができます。

```SQL
-- 一次元のSTRUCTを定義する。
CREATE TABLE t0(
  c0 INT,
  c1 STRUCT<a INT, b INT>
)
DUPLICATE KEY(c0);

-- 複雑なSTRUCTを定義する。
CREATE TABLE t1(
  c0 INT,
  c1 STRUCT<a INT, b STRUCT<c INT, d INT>, c MAP<INT, INT>, d ARRAY<INT>>
)
DUPLICATE KEY(c0);

-- NOT NULLのSTRUCTを定義する。
CREATE TABLE t2(
  c0 INT,
  c1 STRUCT<a INT, b INT> NOT NULL
)
DUPLICATE KEY(c0);
```

STRUCT型のカラムには以下の制約があります：

- テーブルのキーカラムとして使用することはできません。値カラムとしてのみ使用可能です。
- テーブルでのパーティションキーカラム（PARTITION BYに続く）として使用することはできません。
- テーブルでのバケッティングカラム（DISTRIBUTED BYに続く）として使用することはできません。
- 集約テーブルの値カラムとして使用される場合、replace()関数のみをサポートします（[集約テーブル](../../../table_design/table_types/aggregate_table.md)を参照）。

## SQLでSTRUCTを構築する

STRUCTは、以下の関数を使用してSQLで構築できます：[row, struct](../../sql-functions/struct-functions/row.md)、および[named_struct](../../sql-functions/struct-functions/named_struct.md)。struct()はrow()のエイリアスです。

- `row`と`struct`は名前のないSTRUCTをサポートします。フィールド名を指定する必要はありません。StarRocksは自動的にカラム名を生成します、例えば`col1`、`col2`など。
- `named_struct`は名前付きSTRUCTをサポートします。名前と値の式はペアでなければなりません。

StarRocksは入力値に基づいてSTRUCTの型を自動的に決定します。

```SQL
select row(1, 2, 3, 4) as numbers; -- {"col1":1,"col2":2,"col3":3,"col4":4}を返します。
select row(1, 2, null, 4) as numbers; -- {"col1":1,"col2":2,"col3":null,"col4":4}を返します。
select row(null) as nulls; -- {"col1":null}を返します。
select struct(1, 2, 3, 4) as numbers; -- {"col1":1,"col2":2,"col3":3,"col4":4}を返します。
select named_struct('a', 1, 'b', 2, 'c', 3, 'd', 4) as numbers; -- {"a":1,"b":2,"c":3,"d":4}を返します。
```

## STRUCTデータのロード

STRUCTデータをStarRocksにロードするには、[INSERT INTO](../../../loading/InsertInto.md)と[ORC/Parquetロード](../data-manipulation/BROKER_LOAD.md)の2つの方法があります。

StarRocksはデータ型を対応するSTRUCT型に自動的にキャストすることに注意してください。

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

### ORC/ParquetファイルからSTRUCTデータをロードする

StarRocksのSTRUCTデータ型はORCまたはParquet形式のネストされた列構造に対応しています。追加の指定は必要ありません。[ORC/Parquetロード](../data-manipulation/BROKER_LOAD.md)の手順に従ってORCまたはParquetファイルからSTRUCTデータをロードできます。

## STRUCTフィールドへのアクセス

STRUCTのサブフィールドをクエリするには、ドット(`.`)演算子を使用してフィールド名で値をクエリするか、`[]`を使用してインデックスで値を呼び出します。

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
| 4                  |
+--------------------+

mysql> select row(map{'a':1}, 2, 3, 4)[1];
+-----------------------------+
| row(map{'a':1}, 2, 3, 4)[1] |
+-----------------------------+
| {"a":1}                     |
+-----------------------------+
```
