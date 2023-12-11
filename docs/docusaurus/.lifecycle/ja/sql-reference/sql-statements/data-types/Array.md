---
displayed_sidebar: "英語"
---

# ARRAY

データベースの拡張型としてのARRAYは、PostgreSQL、ClickHouse、Snowflakeなどの様々なデータベースシステムでサポートされており、A/Bテスト、ユーザータグ分析、ユーザープロファイリングなどのシナリオで広く使用されています。StarRocksは多次元の配列のネスト、配列のスライス、比較、フィルタリングをサポートしています。

## ARRAYカラムの定義

テーブルを作成する際に、ARRAYカラムを定義することができます。

~~~SQL
-- 一次元の配列を定義する。
ARRAY<type>

-- ネストされた配列を定義する。
ARRAY<ARRAY<type>>

-- NOT NULLとして配列カラムを定義する。
ARRAY<type> NOT NULL
~~~

`type`は配列の要素のデータ型を指定します。StarRocksは次の要素型をサポートしています: BOOLEAN, TINYINT, SMALLINT, INT, BIGINT, LARGEINT, FLOAT, DOUBLE, VARCHAR, CHAR, DATETIME, DATE, JSON, ARRAY（v3.1以降）、MAP（v3.1以降）、STRUCT（v3.1以降）。

配列の要素はデフォルトでnullableです。例えば、`[null, 1 ,2]`のようになります。配列の要素をNOT NULLとして指定することはできません。しかし、テーブルを作成するときに、ARRAYカラムをNOT NULLとして指定することはできます。以下のコードスニペットの3つ目の例を参照してください。

例：

~~~SQL
-- c1をINT型の要素を持つ一次元の配列として定義する。
create table t0(
  c0 INT,
  c1 ARRAY<INT>
)
duplicate key(c0)
distributed by hash(c0);

-- c1をVARCHAR型の要素を持つネストされた配列として定義する。
create table t1(
  c0 INT,
  c1 ARRAY<ARRAY<VARCHAR(10)>>
)
duplicate key(c0)
distributed by hash(c0);

-- c1をNOT NULLの配列カラムとして定義する。
create table t2(
  c0 INT,
  c1 ARRAY<INT> NOT NULL
)
duplicate key(c0)
distributed by hash(c0);
~~~

## 制限事項

StarRocksのテーブルでARRAYカラムを作成する場合、以下の制限が適用されます：

- v2.1より前のバージョンでは、Duplicate KeyテーブルでのみARRAYカラムを作成できます。v2.1以降では、他のタイプのテーブル（Primary Key, Unique Key, Aggregate）でもARRAYカラムを作成できます。ただし、Aggregateテーブルでは、そのカラムのデータを集約する関数がreplace()やreplace_if_not_null()の場合にのみARRAYカラムを作成できます。詳細については[Aggregate table](../../../table_design/table_types/aggregate_table.md)を参照してください。
- ARRAYカラムはキーとして使用することはできません。
- ARRAYカラムはパーティションキー（PARTITION BYに含まれる）またはバケットキー（DISTRIBUTED BYに含まれる）として使用することはできません。
- ARRAYではDECIMAL V3はサポートされていません。
- 配列は最大で14レベルのネストが可能です。

## SQLでの配列の構築

角括弧`[]`を使用し、カンマ（`,`）で要素を区切ることでSQL内で配列を構築できます。

~~~Plain Text
mysql> select [1, 2, 3] as numbers;

+---------+
| numbers |
+---------+
| [1,2,3] |
+---------+

mysql> select ["apple", "orange", "pear"] as fruit;

+---------------------------+
| fruit                     |
+---------------------------+
| ["apple","orange","pear"] |
+---------------------------+

mysql> select [true, false] as booleans;

+----------+
| booleans |
+----------+
| [1,0]    |
+----------+
~~~

StarRocksは、配列が複数の型の要素で構成されている場合、データ型を自動的に推測します：

~~~Plain Text
mysql> select [1, 1.2] as floats;
+---------+
| floats  |
+---------+
| [1.0,1.2] |
+---------+

mysql> select [12, "100"];

+--------------+
| [12,'100']   |
+--------------+
| ["12","100"] |
+--------------+
~~~

角括弧（`<>`）を使って宣言された配列型を示すことができます。