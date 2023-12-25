---
displayed_sidebar: Chinese
---

# ARRAY

本文では、StarRocks での配列型の使用方法について説明します。

配列（Array）はデータベースで拡張されたデータ型で、多くのデータベースシステムでサポートされており、A/Bテストの比較、ユーザータグ分析、オーディエンスプロファイリングなどのシナリオで広く使用されています。StarRocks は現在、多次元配列のネスト、配列のスライス、比較、フィルタリングなどの機能をサポートしています。

## 配列型の列を定義する

テーブルを作成する際に、配列型の列を定義することができます。

~~~SQL
-- 1次元配列を定義する。
ARRAY<type>

-- ネストされた配列を定義する。
ARRAY<ARRAY<type>>

-- NOT NULL の配列列を定義する。
ARRAY<type> NOT NULL
~~~

配列列の定義形式は `ARRAY<type>` で、`type` は配列内の要素の型を表します。配列の要素は現在、以下のデータ型をサポートしています：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、CHAR、DATETIME、DATE、JSON、BINARY（バージョン3.0以降）、MAP（バージョン3.1以降）、STRUCT（バージョン3.1以降）、Fast Decimal（バージョン3.1以降）。

配列内の要素はデフォルトで NULL にすることができます。例えば [NULL, 1, 2] のように。現在、配列の要素を NOT NULL に指定することはサポートされていませんが、配列列自体を NOT NULL に定義することは可能です。上記の3番目の例を参照してください。

> **注意**
>
> 配列型の列を使用する際には以下の制限があります：
>
> * StarRocks バージョン2.1以前では、詳細モデルのテーブル（Duplicate Key）でのみ配列型の列を定義することができました。バージョン2.1以降では、Primary Key、Unique Key、Aggregate Key モデルのテーブルで配列型の列を定義することがサポートされています。ただし、集約モデルのテーブル（Aggregate Key）では、集約列の集約関数が replace または replace_if_not_null の場合に限り、その列を配列型として定義することができます。
> * 配列列は Key 列として使用することはできません。
> * 配列列は分散列（Distributed By）として使用することはできません。
> * 配列列は分割列（Partition By）として使用することはできません。
> * 配列列は DECIMAL V3 データ型をサポートしていません。
> * 配列列は最大で14層のネストをサポートしています。

例：

~~~SQL
-- テーブルを作成し、`c1` 列を INT 型の1次元配列として指定する。
create table t0(
  c0 INT,
  c1 ARRAY<INT>
)
duplicate key(c0)
distributed by hash(c0);

-- テーブルを作成し、`c1` を VARCHAR 型の2層ネスト配列として指定する。
create table t1(
  c0 INT,
  c1 ARRAY<ARRAY<VARCHAR(10)>>
)
duplicate key(c0)
distributed by hash(c0);

-- NOT NULL の配列列を定義するテーブルを作成する。
create table t2(
  c0 INT,
  c1 ARRAY<INT> NOT NULL
)
duplicate key(c0)
distributed by hash(c0);
~~~

## SELECT 文で配列を構築する

SQL 文内で角括弧（`[]`）を使用して配列定数を構築することができます。各配列要素はコンマ（`,`）で区切ります。

例：

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

配列要素が異なる型を持つ場合、StarRocks は適切な型（supertype）を自動的に推定します。

~~~Plain Text
mysql> select [1, 1.2] as floats;

+---------+
| floats  |
+---------+
| [1,1.2] |
+---------+

mysql> select [12, "100"];

+--------------+
| [12,'100']   |
+--------------+
| ["12","100"] |
+--------------+
~~~

角括弧（`<>`）を使用して配列の型を宣言することができます。

~~~Plain Text
mysql> select ARRAY<float>[1, 2];

+-----------------------+
| ARRAY<float>[1.0,2.0] |
+-----------------------+
| [1,2]                 |
+-----------------------+

mysql> select ARRAY<INT>["12", "100"];

+------------------------+
| ARRAY<int(11)>[12,100] |
+------------------------+
| [12,100]               |
+------------------------+
~~~

配列要素には NULL を含むことができます。

~~~Plain Text
mysql> select [1, NULL];

+----------+
| [1,NULL] |
+----------+
| [1,null] |
+----------+
~~~

空の配列を定義する場合、角括弧を使用してその型を宣言することができます。または `[]` と直接定義することもできますが、その場合 StarRocks はコンテキストに基づいて型を推定します。推定できない場合はエラーが発生します。

~~~Plain Text
mysql> select [];

+------+
| []   |
+------+
| []   |
+------+

mysql> select ARRAY<VARCHAR(10)>[];

+----------------------------------+
| ARRAY<unknown type: NULL_TYPE>[] |
+----------------------------------+
| []                               |
+----------------------------------+

mysql> select array_append([], 10);

+----------------------+
| array_append([], 10) |
+----------------------+
| [10]                 |
+----------------------+
~~~

## 配列データのインポート

StarRocks は現在、配列データを書き込むための3つの方法をサポートしています。

### INSERT INTO 文を使用して配列をインポートする

INSERT INTO 文を使用したインポートは、少量のデータを行ごとにインポートする場合や、StarRocks の内部または外部テーブルのデータを ETL 処理してインポートする場合に適しています。

例：

~~~SQL
create table t0(
  c0 INT,
  c1 ARRAY<INT>
)
duplicate key(c0)
distributed by hash(c0);
INSERT INTO t0 VALUES(1, [1,2,3]);
~~~

### Broker Load を使用して ORC または Parquet ファイルの配列をバッチインポートする

StarRocks の配列型は ORC または Parquet 形式の List 構造に対応しているため、追加の指定は必要ありません。具体的なインポート方法については [Broker load](../data-manipulation/BROKER_LOAD.md) を参照してください。

現在、StarRocks は ORC ファイルの List 構造を直接インポートすることをサポートしています。Parquet 形式のインポートは開発中です。

### Stream Load または Routine Load を使用して CSV 形式の配列をインポートする

[Stream Load](../../../loading/StreamLoad.md#导入-csv-格式的数据) または [Routine Load](../../../loading/RoutineLoad.md#导入-csv-数据) を使用して、CSV テキストファイルや Kafka の CSV 形式データをインポートすることができます。デフォルトではコンマで区切られます。

## 配列内の要素にアクセスする

角括弧（`[]`）とインデックスを使用して、配列内の特定の要素にアクセスすることができます。インデックスは `1` から始まります。

~~~Plain Text
mysql> select [1,2,3][1];

+------------+
| [1,2,3][1] |
+------------+
|          1 |
+------------+
~~~

インデックスを `0` または負の数にすると、StarRocks は **エラーを返さずに NULL を返します。**

~~~Plain Text
mysql> select [1,2,3][0];

+------------+
| [1,2,3][0] |
+------------+
|       NULL |
+------------+
~~~

インデックスが配列のサイズを超える場合、StarRocks は **NULL を返します。**

~~~Plain Text
mysql> select [1,2,3][4];

+------------+
| [1,2,3][4] |
+------------+
|       NULL |
+------------+
~~~

多次元配列については、**再帰的な方法**で内部要素にアクセスすることができます。

~~~Plain Text
mysql> select [[1,2],[3,4]][2];

+------------------+
| [[1,2],[3,4]][2] |
+------------------+
| [3,4]            |
+------------------+

mysql> select [[1,2],[3,4]][2][1];

+---------------------+
| [[1,2],[3,4]][2][1] |
+---------------------+
|                   3 |
+---------------------+
~~~
