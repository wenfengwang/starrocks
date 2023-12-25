---
displayed_sidebar: English
---

# ARRAY

ARRAYは、PostgreSQL、ClickHouse、Snowflakeなどのさまざまなデータベースシステムでサポートされている拡張データベースタイプです。ARRAYは、A/Bテスト、ユーザータグ分析、ユーザープロファイリングなどのシナリオで広く使用されています。StarRocksは、多次元配列のネスティング、配列のスライシング、比較、フィルタリングをサポートしています。

## ARRAYカラムの定義

テーブルを作成する際に、ARRAYカラムを定義することができます。

~~~SQL
-- 一次元配列を定義する。
ARRAY<type>

-- ネストされた配列を定義する。
ARRAY<ARRAY<type>>

-- NOT NULLとして配列カラムを定義する。
ARRAY<type> NOT NULL
~~~

`type`は配列内の要素のデータ型を指定します。StarRocksは以下の要素タイプをサポートしています：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、CHAR、DATETIME、DATE、JSON、ARRAY（v3.1以降）、MAP（v3.1以降）、STRUCT（v3.1以降）。

配列内の要素はデフォルトでNULL許容です。例：`[null, 1, 2]`。配列内の要素をNOT NULLとして指定することはできません。しかし、テーブルを作成する際に、ARRAYカラムをNOT NULLとして指定することは可能です。以下のコードスニペットの3番目の例を参照してください。

例：

~~~SQL
-- 要素タイプがINTの一次元配列としてc1を定義する。
create table t0(
  c0 INT,
  c1 ARRAY<INT>
)
duplicate key(c0)
distributed by hash(c0);

-- 要素タイプがVARCHARのネストされた配列としてc1を定義する。
create table t1(
  c0 INT,
  c1 ARRAY<ARRAY<VARCHAR(10)>>
)
duplicate key(c0)
distributed by hash(c0);

-- NOT NULL配列カラムとしてc1を定義する。
create table t2(
  c0 INT,
  c1 ARRAY<INT> NOT NULL
)
duplicate key(c0)
distributed by hash(c0);
~~~

## 制限事項

StarRocksテーブルでARRAYカラムを作成する際には、以下の制限が適用されます：

- v2.1より前のバージョンでは、Duplicate KeyテーブルでのみARRAYカラムを作成できます。v2.1以降では、他のテーブルタイプ（Primary Key、Unique Key、Aggregate）でもARRAYカラムを作成できます。ただし、Aggregateテーブルでは、そのカラムでデータを集計する関数がreplace()またはreplace_if_not_null()の場合にのみARRAYカラムを作成できます。詳細は[集計テーブル](../../../table_design/table_types/aggregate_table.md)を参照してください。
- ARRAYカラムはキーカラムとして使用できません。
- ARRAYカラムはパーティションキー（PARTITION BYに含まれる）やバケットキー（DISTRIBUTED BYに含まれる）として使用できません。
- DECIMAL V3はARRAYでサポートされていません。
- 配列は最大で14レベルのネスティングを持つことができます。

## SQLでの配列の構築

配列は、角括弧`[]`を使用してSQLで構築でき、各配列要素はコンマ`,`で区切られます。

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

StarRocksは、配列が複数の型の要素で構成されている場合、自動的にデータ型を推測します。

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

山括弧`<>`を使用して宣言された配列型を示すことができます。

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

要素にはNULLを含めることができます。

~~~Plain Text
mysql> select [1, NULL];

+----------+
| [1,NULL] |
+----------+
| [1,null] |
+----------+
~~~

空の配列の場合、山括弧`<>`を使用して宣言された型を示すか、またはStarRocksがコンテキストに基づいて型を推測するために`[]`を直接記述することができます。StarRocksが型を推測できない場合は、エラーが報告されます。

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

## 配列データのロード

StarRocksは、配列データをロードするための3つの方法をサポートしています：

- INSERT INTOは、テスト用の小規模データをロードするのに適しています。
- Broker Loadは、大規模データを含むORCまたはParquetファイルのロードに適しています。
- Stream LoadとRoutine Loadは、大規模データを含むCSVファイルのロードに適しています。

### INSERT INTOを使用して配列をロードする

INSERT INTOを使用して、小規模データを列ごとにロードしたり、データをロードする前にETLを実行したりできます。

  ~~~SQL
create table t0(
  c0 INT,
  c1 ARRAY<INT>
)
duplicate key(c0)
distributed by hash(c0);

INSERT INTO t0 VALUES(1, [1,2,3]);
~~~

### Broker Loadを使用してORCまたはParquetファイルから配列をロードする

StarRocksの配列型はORCファイルとParquetファイルのリスト構造に対応しており、StarRocksで異なるデータ型を指定する必要がありません。データロードの詳細については、[Broker Load](../data-manipulation/BROKER_LOAD.md)を参照してください。

### Stream LoadまたはRoutine Loadを使用してCSV形式の配列をロードする

CSVファイル内の配列はデフォルトでコンマで区切られています。[Stream Load](../../../loading/StreamLoad.md#load-csv-data)または[Routine Load](../../../loading/RoutineLoad.md#load-csv-format-data)を使用して、Kafka内のCSVテキストファイルやCSVデータをロードできます。

## ARRAYデータのクエリ

配列内の要素には、`[]`と添字（1から始まる）を使用してアクセスできます。

~~~Plain Text
mysql> select [1,2,3][1];

+------------+
| [1,2,3][1] |
+------------+
|          1 |
+------------+
1行セット（0.00秒）
~~~

添字が0または負の数の場合、**エラーは報告されず、NULLが返されます**。

~~~Plain Text
mysql> select [1,2,3][0];

+------------+
| [1,2,3][0] |
+------------+
|       NULL |
+------------+
1行セット（0.01秒）
~~~

添字が配列の長さ（配列内の要素数）を超える場合、**NULLが返されます**。

~~~Plain Text
mysql> select [1,2,3][4];

+------------+
| [1,2,3][4] |
+------------+
|       NULL |
+------------+
1行セット（0.01秒）
~~~

多次元配列の場合、要素には**再帰的にアクセスできます**。

~~~Plain Text
mysql(ARRAY)> select [[1,2],[3,4]][2];

+------------------+
| [[1,2],[3,4]][2] |
+------------------+
| [3,4]            |
+------------------+
1行セット（0.00秒）

mysql> select [[1,2],[3,4]][2][1];

+---------------------+
| [[1,2],[3,4]][2][1] |
+---------------------+
|                   3 |
+---------------------+
1行セット（0.01秒）
~~~
