---
displayed_sidebar: "Japanese"
---

# STRUCT

## 説明

STRUCT は複雑なデータ型を表現するために広く使用されます。それは、例えば `<a INT, b STRING>` のように、異なるデータ型を持つ要素（またはフィールドとも呼ばれる）のコレクションを表します。

struct 内のフィールド名は一意である必要があります。フィールドはプリミティブなデータ型（数値、文字列、または日付など）または複雑なデータ型（ARRAY や MAP など）であることができます。

struct 内のフィールドは、さらに別の STRUCT や ARRAY、または MAP にすることができ、これにより入れ子構造のデータ構造を作成することができます。例えば、 `STRUCT<a INT, b STRUCT<c INT, d INT>, c MAP<INT, INT>, d ARRAY<INT>>`。

STRUCT データ型は v3.1 からサポートされています。v3.1 では、StarRocks テーブルを作成し、そのテーブルに STRUCT データをロードし、MAP データをクエリすることができます。

v2.5 以降、StarRocks はデータレイクからの複雑なデータ型 MAP および STRUCT のクエリをサポートしています。Apache Hive™、Apache Hudi、および Apache Iceberg から提供される外部カタログを使用して、ORC および Parquet ファイルから MAP および STRUCT データをクエリすることができます。外部カタログを使用して外部データソースをクエリする方法の詳細については、[カタログ概要](../../../data_source/catalog/catalog_overview.md) および必要なカタログタイプに関連するトピックを参照してください。

## 構文

```Haskell
STRUCT<name, type>
```

- `name`: フィールド名であり、CREATE TABLE 文で定義されている列の名前と同じです。
- `type`: フィールド型であり、サポートされている任意の型であることができます。

## StarRocks で STRUCT 列を定義する

テーブルを作成し、STRUCT データをこの列にロードするときに STRUCT 列を定義することができます。

```SQL
-- 1 次元の struct を定義する。
CREATE TABLE t0(
  c0 INT,
  c1 STRUCT<a INT, b INT>
)
DUPLICATE KEY(c0);

-- 複雑な struct を定義する。
CREATE TABLE t1(
  c0 INT,
  c1 STRUCT<a INT, b STRUCT<c INT, d INT>, c MAP<INT, INT>, d ARRAY<INT>>
)
DUPLICATE KEY(c0);

-- NOT NULL struct を定義する。
CREATE TABLE t2(
  c0 INT,
  c1 STRUCT<a INT, b INT> NOT NULL
)
DUPLICATE KEY(c0);
```

STRUCT 型の列には以下の制限があります。

- テーブルのキー列として使用することはできません。値列としてのみ使用できます。
- テーブルのパーティションキー列（PARTITION BY の後に従う）として使用することはできません。
- テーブルのバケット列（DISTRIBUTED BY の後に従う）として使用することはできません。
- [集計テーブル](../../../table_design/table_types/aggregate_table.md) で値列として使用される場合、replace() 関数のみをサポートします。

## SQL で struct を構築する

SQL で struct を構築するには、以下の関数を使用します：[row, struct](../../sql-functions/struct-functions/row.md)、[named_struct](../../sql-functions/struct-functions/named_struct.md)。struct() は row() のエイリアスです。

- `row` および `struct` は匿名の struct をサポートしています。フィールド名を指定する必要はありません。StarRocks は自動的に列名、例えば `col1`, `col2`... を生成します。
- `named_struct` は名前付きの struct をサポートします。名前と値の式はペアでなければなりません。

StarRocks は、入力値に基づいて struct の型を自動的に決定します。

```SQL
select row(1, 2, 3, 4) as numbers; -- {"col1":1,"col2":2,"col3":3,"col4":4} を返す。
select row(1, 2, null, 4) as numbers; -- {"col1":1,"col2":2,"col3":null,"col4":4} を返す。
select row(null) as nulls; -- {"col1":null} を返す。
select struct(1, 2, 3, 4) as numbers; -- {"col1":1,"col2":2,"col3":3,"col4":4} を返す。
select named_struct('a', 1, 'b', 2, 'c', 3, 'd', 4) as numbers; -- {"a":1,"b":2,"c":3,"d":4} を返す。
```

## STRUCT データのロード

STRUCT データは、[INSERT INTO](../../../loading/InsertInto.md) および [ORC/Parquet ローディング](../data-manipulation/BROKER_LOAD.md) の 2 つの方法で StarRocks にロードすることができます。

StarRocks はデータ型を対応する STRUCT 型に自動的にキャストします。

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

### ORC/Parquet ファイルから STRUCT データをロードする

StarRocks の STRUCT データ型は、ORC や Parquet 形式の入れ子の列構造に対応しています。追加の仕様は必要ありません。[ORC/Parquet ローディング](../data-manipulation/BROKER_LOAD.md) の手順に従って、ORC または Parquet ファイルから STRUCT データをロードすることができます。

## STRUCT フィールドへのアクセス

struct のサブフィールドをクエリするには、ドット (`.`) 演算子を使用してフィールド名によって値をクエリするか、インデックスによって値を呼び出すために `[]` を使用することができます。

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