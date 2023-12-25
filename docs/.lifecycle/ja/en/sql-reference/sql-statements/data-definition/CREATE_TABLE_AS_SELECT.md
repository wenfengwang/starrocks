---
displayed_sidebar: English
---

# CREATE TABLE AS SELECT

## 説明

CREATE TABLE AS SELECT (CTAS) ステートメントを使用して、テーブルを同期的または非同期的にクエリし、クエリ結果に基づいて新しいテーブルを作成してから、クエリ結果を新しいテーブルに挿入できます。

[SUBMIT TASK](../data-manipulation/SUBMIT_TASK.md) を使用して、非同期 CTAS タスクを送信できます。

## 構文

- テーブルを同期的にクエリし、クエリ結果に基づいて新しいテーブルを作成し、その後クエリ結果を新しいテーブルに挿入します。

  ```SQL
  CREATE TABLE [IF NOT EXISTS] [database.]table_name
  [(column_name [, column_name2, ...])]
  [key_desc]
  [COMMENT "table comment"]
  [partition_desc]
  [distribution_desc]
  [PROPERTIES ("key"="value", ...)]
  AS SELECT query
  [ ... ]
  ```

- テーブルを非同期的にクエリし、クエリ結果に基づいて新しいテーブルを作成し、その後クエリ結果を新しいテーブルに挿入します。

  ```SQL
  SUBMIT [/*+ SET_VAR(key=value) */] TASK [[database.]<task_name>] AS
  CREATE TABLE [IF NOT EXISTS] [database.]table_name
  [(column_name [, column_name2, ...])]
  [key_desc]
  [COMMENT "table comment"]
  [partition_desc]
  [distribution_desc]
  [PROPERTIES ("key"="value", ...)] AS SELECT query
  [ ... ]
  ```

## パラメーター

| **パラメーター**     | **必須** | **説明**                                              |
| ----------------- | ------------ | ------------------------------------------------------------ |
| column_name       | はい          | 新しいテーブルの列の名前です。列のデータ型を指定する必要はありません。StarRocks は、列に適切なデータ型を自動的に指定します。FLOAT および DOUBLE データは DECIMAL(38,9) データに、CHAR、VARCHAR、および STRING データは VARCHAR(65533) データに変換されます。 |
| key_desc          | いいえ           | 構文は `key_type ( <col_name1> [, <col_name2> , ...])` です。<br />**パラメーター**:<ul><li>`key_type`: [新しいテーブルのキータイプ](../../../table_design/table_types/table_types.md)。有効な値: `DUPLICATE KEY` と `PRIMARY KEY`。デフォルト値: `DUPLICATE KEY`。</li><li> `col_name`: キーを形成する列。</li></ul> |
| COMMENT           | いいえ           | 新しいテーブルのコメントです。                                |
| partition_desc    | いいえ           | 新しいテーブルのパーティション方式です。このパラメーターを指定しない場合、デフォルトでは新しいテーブルにはパーティションがありません。パーティションについての詳細は CREATE TABLE を参照してください。 |
| distribution_desc | いいえ           | 新しいテーブルの分散方式です。このパラメータを指定しない場合、バケット列はコストベースのオプティマイザ (CBO) によって収集された最高のカーディナリティを持つ列になります。バケット数はデフォルトで 10 です。CBO がカーディナリティの情報を収集していない場合、バケット列は新しいテーブルの最初の列になります。バケットについての詳細は CREATE TABLE を参照してください。 |
| PROPERTIES        | いいえ           | 新しいテーブルのプロパティです。                             |
| AS SELECT クエリ   | はい          | クエリ結果です。`... AS SELECT query` で列を指定できます。例えば `... AS SELECT a, b, c FROM table_a;` では、`a`、`b`、`c` はクエリされたテーブルの列名を示します。新しいテーブルの列名を指定しない場合、新しいテーブルの列名も `a`、`b`、`c` になります。`... AS SELECT query` で式を指定することもできます。例えば `... AS SELECT a+1 AS x, b+2 AS y, c*c AS z FROM table_a;` では、`a+1`、`b+2`、`c*c` はクエリされたテーブルの列名を示し、`x`、`y`、`z` は新しいテーブルの列名を示します。注意: 新しいテーブルの列数は、SELECT ステートメントで指定された列数と同じでなければなりません。識別しやすい列名の使用を推奨します。|

## 使用上の注意

- CTAS ステートメントは、以下の要件を満たす新しいテーブルのみを作成できます:
  - `ENGINE` は `OLAP` です。

  - テーブルはデフォルトで Duplicate Key テーブルです。`key_desc` で Primary Key テーブルとして指定することもできます。

  - ソートキーは最初の3つの列で、これら3つの列のデータ型のストレージスペースは36バイトを超えません。

- CTAS ステートメントは、新しく作成されたテーブルのインデックスを設定することはサポートしていません。

- FE の再起動などの理由で CTAS ステートメントの実行に失敗した場合、以下のいずれかの問題が発生する可能性があります:
  - 新しいテーブルは正常に作成されるが、データが含まれていない。

  - 新しいテーブルの作成に失敗する。

- 新しいテーブルが作成された後、INSERT INTO などの複数の方法を使用して新しいテーブルにデータを挿入する場合、最初に INSERT 操作を完了した方法がそのデータを新しいテーブルに挿入します。

- 新しいテーブルを作成した後、そのテーブルに対する権限をユーザーに手動で付与する必要があります。

- テーブルに対して非同期クエリを行い、その結果に基づいて新しいテーブルを作成する際にタスク名を指定しない場合、StarRocksは自動的にタスク名を生成します。

## 例

例 1: テーブル`order`を同期的にクエリし、その結果に基づいて新しいテーブル`order_new`を作成し、その後クエリ結果を新しいテーブルに挿入します。

```SQL
CREATE TABLE order_new
AS SELECT * FROM order;
```

例 2: テーブル`order`の`k1`、`k2`、`k3`列を同期的にクエリし、その結果に基づいて新しいテーブル`order_new`を作成し、その後クエリ結果を新しいテーブルに挿入します。さらに、新しいテーブルの列名を`a`、`b`、`c`に設定します。

```SQL
CREATE TABLE order_new (a, b, c)
AS SELECT k1, k2, k3 FROM order;
```

または

```SQL
CREATE TABLE order_new
AS SELECT k1 AS a, k2 AS b, k3 AS c FROM order;
```

例 3: テーブル`employee`の`salary`列の最大値を同期的にクエリし、その結果に基づいて新しいテーブル`employee_new`を作成し、その後クエリ結果を新しいテーブルに挿入します。さらに、新しいテーブルの列名を`salary_max`に設定します。

```SQL
CREATE TABLE employee_new
AS SELECT MAX(salary) AS salary_max FROM employee;
```

データを挿入した後、新しいテーブルをクエリします。

```SQL
SELECT * FROM employee_new;

+------------+
| salary_max |
+------------+
|   10000    |
+------------+
```

例 4: CTASを使用してプライマリキーテーブルを作成します。プライマリキーテーブルのデータ行数がクエリ結果のデータ行数よりも少ない場合があることに注意してください。これは、[プライマリキー](../../../table_design/table_types/primary_key_table.md)テーブルが同じプライマリキーを持つ行のグループの中で最新のデータ行のみを保持するためです。

```SQL
CREATE TABLE employee_new
PRIMARY KEY(order_id)
AS SELECT
    order_list.order_id,
    sum(goods.price) as total
FROM order_list INNER JOIN goods ON goods.item_id1 = order_list.item_id2
GROUP BY order_id;
```

例 5: `lineorder`、`customer`、`supplier`、`part`の4つのテーブルを同期的にクエリし、その結果に基づいて新しいテーブル`lineorder_flat`を作成し、その後クエリ結果を新しいテーブルに挿入します。さらに、新しいテーブルのパーティショニング方法とバケッティング方法を指定します。

```SQL
CREATE TABLE lineorder_flat
PARTITION BY RANGE(`LO_ORDERDATE`)
(
    START ("1993-01-01") END ("1999-01-01") EVERY (INTERVAL 1 YEAR)
)
DISTRIBUTED BY HASH(`LO_ORDERKEY`) BUCKETS 10 AS SELECT
    l.LO_ORDERKEY AS LO_ORDERKEY,
    l.LO_LINENUMBER AS LO_LINENUMBER,
    l.LO_CUSTKEY AS LO_CUSTKEY,
    l.LO_PARTKEY AS LO_PARTKEY,
    l.LO_SUPPKEY AS LO_SUPPKEY,
    l.LO_ORDERDATE AS LO_ORDERDATE,
    l.LO_ORDERPRIORITY AS LO_ORDERPRIORITY,
    l.LO_SHIPPRIORITY AS LO_SHIPPRIORITY,
    l.LO_QUANTITY AS LO_QUANTITY,
    l.LO_EXTENDEDPRICE AS LO_EXTENDEDPRICE,
    l.LO_ORDTOTALPRICE AS LO_ORDTOTALPRICE,
    l.LO_DISCOUNT AS LO_DISCOUNT,
    l.LO_REVENUE AS LO_REVENUE,
    l.LO_SUPPLYCOST AS LO_SUPPLYCOST,
    l.LO_TAX AS LO_TAX,
    l.LO_COMMITDATE AS LO_COMMITDATE,
    l.LO_SHIPMODE AS LO_SHIPMODE,
    c.C_NAME AS C_NAME,
    c.C_ADDRESS AS C_ADDRESS,
    c.C_CITY AS C_CITY,
    c.C_NATION AS C_NATION,
    c.C_REGION AS C_REGION,
    c.C_PHONE AS C_PHONE,
    c.C_MKTSEGMENT AS C_MKTSEGMENT,
    s.S_NAME AS S_NAME,
    s.S_ADDRESS AS S_ADDRESS,
    s.S_CITY AS S_CITY,
    s.S_NATION AS S_NATION,
    s.S_REGION AS S_REGION,
    s.S_PHONE AS S_PHONE,
    p.P_NAME AS P_NAME,
    p.P_MFGR AS P_MFGR,
    p.P_CATEGORY AS P_CATEGORY,
    p.P_BRAND AS P_BRAND,
    p.P_COLOR AS P_COLOR,
    p.P_TYPE AS P_TYPE,
    p.P_SIZE AS P_SIZE,
    p.P_CONTAINER AS P_CONTAINER FROM lineorder AS l
INNER JOIN customer AS c ON c.C_CUSTKEY = l.LO_CUSTKEY
INNER JOIN supplier AS s ON s.S_SUPPKEY = l.LO_SUPPKEY
INNER JOIN part AS p ON p.P_PARTKEY = l.LO_PARTKEY;
```

例 6: テーブル`order_detail`を非同期的にクエリし、その結果に基づいて新しいテーブル`order_statistics`を作成し、その後クエリ結果を新しいテーブルに挿入します。

```plaintext
SUBMIT TASK AS CREATE TABLE order_statistics AS SELECT COUNT(*) as count FROM order_detail;

+-------------------------------------------+-----------+
| TaskName                                  | Status    |
+-------------------------------------------+-----------+
| ctas-df6f7930-e7c9-11ec-abac-bab8ee315bf2 | SUBMITTED |
+-------------------------------------------+-----------+
```

タスクの情報を確認します。

```SQL
SELECT * FROM INFORMATION_SCHEMA.TASKS;

-- タスクの情報

TASK_NAME: ctas-df6f7930-e7c9-11ec-abac-bab8ee315bf2
CREATE_TIME: 2022-06-14 14:07:06
SCHEDULE: MANUAL
DATABASE: default_cluster:test
DEFINITION: CREATE TABLE order_statistics AS SELECT COUNT(*) as count FROM order_detail
EXPIRE_TIME: 2022-06-17 14:07:06
```

TaskRunの状態を確認します。

```SQL
SELECT * FROM INFORMATION_SCHEMA.TASK_RUNS;

-- TaskRunの状態

QUERY_ID: 37bd2b63-eba8-11ec-8d41-bab8ee315bf2
TASK_NAME: ctas-df6f7930-e7c9-11ec-abac-bab8ee315bf2
CREATE_TIME: 2022-06-14 14:07:06
FINISH_TIME: 2022-06-14 14:07:07
STATE: SUCCESS
DATABASE: 
DEFINITION: CREATE TABLE order_statistics AS SELECT COUNT(*) as count FROM order_detail
```
EXPIRE_TIME: 2022-06-17 14:07:06
ERROR_CODE: 0
ERROR_MESSAGE: NULL
```

TaskRunの状態が`SUCCESS`の場合、新しいテーブルをクエリします。

```SQL
SELECT * FROM order_statistics;
```

