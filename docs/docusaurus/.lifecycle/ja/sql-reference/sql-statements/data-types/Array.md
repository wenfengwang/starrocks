---
displayed_sidebar: "Japanese"
---

# 配列

配列は、データベースの拡張タイプとして、PostgreSQL、ClickHouse、Snowflakeなどのさまざまなデータベースシステムでサポートされています。配列は、A/Bテスト、ユーザータグ分析、ユーザープロファイリングなどのシナリオで広く使用されています。StarRocksは、多次元配列の入れ子、配列のスライス、比較、フィルタリングをサポートしています。

## 配列列を定義する

テーブルを作成する際に、配列列を定義することができます。

~~~SQL
-- 一次元配列を定義する。
ARRAY<型>

-- 入れ子の配列を定義する。
ARRAY<ARRAY<型>>

-- 配列列をNOT NULLとして定義する。
ARRAY<型> NOT NULL
~~~

`型` は、配列内の要素のデータ型を指定します。StarRocksは、次の要素型をサポートしています: BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、CHAR、DATETIME、DATE、JSON、ARRAY（v3.1以降）、MAP（v3.1以降）、およびSTRUCT（v3.1以降）。

配列内の要素はデフォルトでNULL可能ですが、例えば `[null, 1 ,2]` のように、配列内の要素をNOT NULLとして指定することはできません。ただし、テーブルを作成する際に配列列をNOT NULLとして指定することはできます。以下のコード例の3番目の例のようにです。

例:

~~~SQL
-- c1をINT型の一次元配列として定義する。
create table t0(
  c0 INT,
  c1 ARRAY<INT>
)
duplicate key(c0)
distributed by hash(c0);

-- c1をVARCHAR(10)型の入れ子配列として定義する。
create table t1(
  c0 INT,
  c1 ARRAY<ARRAY<VARCHAR(10)>>
)
duplicate key(c0)
distributed by hash(c0);

-- c1をNOT NULL配列列として定義する。
create table t2(
  c0 INT,
  c1 ARRAY<INT> NOT NULL
)
duplicate key(c0)
distributed by hash(c0);
~~~

## 制限

StarRocksテーブルで配列列を作成する際には、以下の制限が適用されます:

- v2.1より前のバージョンでは、配列列を重複キーのテーブルにのみ作成できます。v2.1以降では、他のタイプのテーブル（プライマリキー、ユニークキー、集計）にも配列列を作成できます。ただし、集計テーブルでは、その列のデータを集計するために使用される関数がreplace()またはreplace_if_not_null()の場合のみ、配列列を作成できます。詳細については、[集計テーブル](../../../table_design/table_types/aggregate_table.md)を参照してください。
- 配列列はキーカラムとして使用できません。
- 配列列はパーティションキー（PARTITION BYに含まれる）またはバケット化キー（DISTRIBUTED BYに含まれる）として使用できません。
- DECIMAL V3は配列でサポートされません。
- 配列の入れ子の最大レベルは14です。

## SQLで配列を作成する

配列は、角括弧`[]`を使用してSQLで構築することができます。各配列要素はカンマ（`,`）で区切ります。

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

StarRocksは、複数の型の要素から構成される配列を自動的にデータ型推論します。

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

宣言された配列型を示すために、尖括弧（`<>`）を使用できます。

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

要素にNULLを含めることができます。

~~~Plain Text
mysql> select [1, NULL];

+----------+
| [1,NULL] |
+----------+
| [1,null] |
+----------+
~~~

空の配列の場合、宣言された型を示すために尖括弧を使用するか、StarRocksがコンテキストに基づいて型を推論するために直接`\[\]`を記述することができます。StarRocksが型を推論できない場合は、エラーが報告されます。

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

## 配列データをロードする

StarRocksでは、配列データを次の3つの方法でロードすることができます:

- INSERT INTO は、テスト用の小規模データをロードするのに適しています。
- Broker Load は、大規模なデータを持つORCまたはParquetファイルをロードするのに適しています。
- Stream Load およびRoutine Load は、大規模なデータを持つCSVファイルをロードするのに適しています。

### INSERT INTOを使用して配列をロードする

INSERT INTOを使用して、小規模データを列ごとにロードしたり、データをロードする前にETLを行ったりすることができます。

  ~~~SQL
create table t0(
  c0 INT,
  c1 ARRAY<INT>
)
duplicate key(c0)
distributed by hash(c0);

INSERT INTO t0 VALUES(1, [1,2,3]);
~~~

### Broker Loadを使用してORCやParquetファイルから配列をロードする

  StarRocksの配列型は、ORCやParquetファイルのリスト構造に対応しており、StarRocksで異なるデータ型を指定する必要がなくなります。データの詳細については、[Broker load](../data-manipulation/BROKER_LOAD.md)を参照してください。

### Stream LoadまたはRoutine Loadを使用してCSV形式の配列をロードする

  CSVファイル内の配列はデフォルトでカンマで区切られています。CSVテキストファイルやKafka内のCSVデータをロードするには、 [Stream Load](../../../loading/StreamLoad.md#load-csv-data) または [Routine Load](../../../loading/RoutineLoad.md#load-csv-format-data) を使用できます。

## 配列データをクエリする

`[]` およびサブスクリプトを使用して、配列の要素にアクセスすることができます。サブスクリプトは`1`から始まります。

~~~Plain Text
mysql> select [1,2,3][1];

+------------+
| [1,2,3][1] |
+------------+
|          1 |
+------------+
1 row in set (0.00 sec)
~~~

サブスクリプトが0または負の数の場合、**エラーは報告されず、NULLが返されます**。

~~~Plain Text
mysql> select [1,2,3][0];

+------------+
| [1,2,3][0] |
+------------+
|       NULL |
+------------+
1 row in set (0.01 sec)
~~~

サブスクリプトが配列の長さ（配列内の要素数）を超える場合、**NULLが返されます**。

~~~Plain Text
mysql> select [1,2,3][4];

+------------+
| [1,2,3][4] |
+------------+
|       NULL |
+------------+
1 row in set (0.01 sec)
~~~

多次元配列の場合、要素に**再帰的に**アクセスすることができます。

~~~Plain Text
mysql(ARRAY)> select [[1,2],[3,4]][2];

+------------------+
| [[1,2],[3,4]][2] |
+------------------+
| [3,4]            |
+------------------+
1 row in set (0.00 sec)

mysql> select [[1,2],[3,4]][2][1];

+---------------------+
| [[1,2],[3,4]][2][1] |
+---------------------+
|                   3 |
+---------------------+
1 row in set (0.01 sec)
~~~