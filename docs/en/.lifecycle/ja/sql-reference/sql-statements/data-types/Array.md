---
displayed_sidebar: "Japanese"
---

# ARRAY（配列）

ARRAYは、PostgreSQL、ClickHouse、Snowflakeなど、さまざまなデータベースシステムでサポートされている、データベースの拡張型です。ARRAYは、A/Bテスト、ユーザータグ分析、ユーザープロファイリングなどのシナリオで広く使用されています。StarRocksは、多次元配列のネスト、配列のスライシング、比較、フィルタリングをサポートしています。

## ARRAY列の定義

テーブルを作成する際に、ARRAY列を定義することができます。

~~~SQL
-- 1次元配列を定義します。
ARRAY<型>

-- ネストされた配列を定義します。
ARRAY<ARRAY<型>>

-- 配列列をNOT NULLとして定義します。
ARRAY<型> NOT NULL
~~~

`型`は、配列の要素のデータ型を指定します。StarRocksは、次の要素型をサポートしています：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、CHAR、DATETIME、DATE、JSON、ARRAY（v3.1以降）、MAP（v3.1以降）、STRUCT（v3.1以降）。

配列の要素はデフォルトでnull許容です。例えば、`[null, 1, 2]`のように指定することはできますが、配列の要素をNOT NULLとして指定することはできません。ただし、テーブルを作成する際にARRAY列をNOT NULLとして指定することはできます。以下のコードスニペットの3番目の例のように指定します。

例：

~~~SQL
-- 要素の型がINTである1次元配列c1を定義します。
create table t0(
  c0 INT,
  c1 ARRAY<INT>
)
duplicate key(c0)
distributed by hash(c0);

-- 要素の型がVARCHAR(10)であるネストされた配列c1を定義します。
create table t1(
  c0 INT,
  c1 ARRAY<ARRAY<VARCHAR(10)>>
)
duplicate key(c0)
distributed by hash(c0);

-- NOT NULLの配列列c1を定義します。
create table t2(
  c0 INT,
  c1 ARRAY<INT> NOT NULL
)
duplicate key(c0)
distributed by hash(c0);
~~~

## 制限事項

StarRocksテーブルでARRAY列を作成する際には、以下の制限が適用されます：

- v2.1より前のバージョンでは、ARRAY列はDuplicate Keyテーブルのみで作成できます。v2.1以降では、Primary Key、Unique Key、Aggregateなどの他のタイプのテーブルでもARRAY列を作成できます。ただし、Aggregateテーブルでは、その列のデータを集約するために使用される関数がreplace()またはreplace_if_not_null()である場合にのみ、ARRAY列を作成できます。詳細については、[Aggregateテーブル](../../../table_design/table_types/aggregate_table.md)を参照してください。
- ARRAY列はキーカラムとして使用できません。
- ARRAY列は、パーティションキー（PARTITION BYに含まれる）またはバケットキー（DISTRIBUTED BYに含まれる）として使用できません。
- DECIMAL V3はARRAYでサポートされていません。
- 配列のネストは最大14レベルまでです。

## SQLでの配列の構築

配列は、角括弧`[]`を使用してSQLで構築することができます。各配列要素はカンマ`,`で区切られます。

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

StarRocksは、配列が複数の型の要素で構成されている場合、データ型を自動的に推論します。

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

宣言された配列の型を示すために、尖括弧（`<>`）を使用することができます。

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

空の配列の場合、宣言された型を示すために尖括弧を使用するか、StarRocksがコンテキストに基づいて型を推論するために直接`[]`を記述することができます。StarRocksが型を推論できない場合は、エラーが発生します。

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

StarRocksでは、次の3つの方法で配列データをロードすることができます：

- INSERT INTOは、テスト用の小規模データのロードに適しています。
- Broker Loadは、大規模なORCまたはParquetファイルの配列データをロードするのに適しています。
- Stream LoadおよびRoutine Loadは、大規模なCSVファイルの配列データをロードするのに適しています。

### INSERT INTOを使用して配列をロードする

INSERT INTOを使用して、小規模データを列ごとにロードするか、データをロードする前にETLを実行することができます。

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

  StarRocksの配列型は、ORCおよびParquetファイルのリスト構造に対応しており、StarRocksで異なるデータ型を指定する必要がなくなります。データのロードについての詳細は、[Broker load](../data-manipulation/BROKER_LOAD.md)を参照してください。

### Stream LoadまたはRoutine Loadを使用してCSV形式の配列をロードする

  CSVファイルの配列はデフォルトでカンマで区切られます。[Stream Load](../../../loading/StreamLoad.md#load-csv-data)または[Routine Load](../../../loading/RoutineLoad.md#load-csv-format-data)を使用して、CSVテキストファイルまたはKafkaのCSVデータをロードすることができます。

## 配列データのクエリ

`[]`と添字を使用して、配列内の要素にアクセスすることができます。添字は`1`から始まります。

~~~Plain Text
mysql> select [1,2,3][1];

+------------+
| [1,2,3][1] |
+------------+
|          1 |
+------------+
1 row in set (0.00 sec)
~~~

添字が`0`または負の数の場合、**エラーは報告されず、NULLが返されます**。

~~~Plain Text
mysql> select [1,2,3][0];

+------------+
| [1,2,3][0] |
+------------+
|       NULL |
+------------+
1 row in set (0.01 sec)
~~~

添字が配列の長さ（配列の要素数）を超える場合、**NULLが返されます**。

~~~Plain Text
mysql> select [1,2,3][4];

+------------+
| [1,2,3][4] |
+------------+
|       NULL |
+------------+
1 row in set (0.01 sec)
~~~

多次元配列の場合、要素には**再帰的に**アクセスできます。

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
