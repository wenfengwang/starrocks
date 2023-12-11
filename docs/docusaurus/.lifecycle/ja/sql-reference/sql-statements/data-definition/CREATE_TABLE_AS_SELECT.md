---
displayed_sidebar: "Japanese"
---

# CREATE TABLE AS SELECT

## Description
クエリを使用してテーブルを同期的にまたは非同期的に問い合わせ、クエリの結果に基づいて新しいテーブルを作成し、その後にクエリの結果を新しいテーブルに挿入するためにCREATE TABLE AS SELECT（CTAS）ステートメントを使用できます。

[SUBMIT TASK](../data-manipulation/SUBMIT_TASK.md)を使用して非同期のCTASタスクを提出できます。

## Syntax

- テーブルを同期的に問い合わせ、クエリの結果に基づいて新しいテーブルを作成し、その後にクエリの結果を新しいテーブルに挿入します。

  ```SQL
  CREATE TABLE [IF NOT EXISTS] [database.]table_name
  [(column_name [, column_name2, ...]]
  [key_desc]
  [COMMENT "table comment"]
  [partition_desc]
  [distribution_desc]
  [PROPERTIES ("key"="value", ...)]
  AS SELECT query
  [ ... ]
  ```

- テーブルを非同期的に問い合わせ、クエリの結果に基づいて新しいテーブルを作成し、その後にクエリの結果を新しいテーブルに挿入します。

  ```SQL
  SUBMIT [/*+ SET_VAR(key=value) */] TASK [[database.]<task_name>]AS
  CREATE TABLE [IF NOT EXISTS] [database.]table_name
  [(column_name [, column_name2, ...]]
  [key_desc]
  [COMMENT "table comment"]
  [partition_desc]
  [distribution_desc]
  [PROPERTIES ("key"="value", ...)]AS SELECT query
  [ ... ]
  ```

## Parameters

| **Parameter**     | **Required** | **Description**                                              |
| ----------------- | ------------ | ------------------------------------------------------------ |
| column_name       | Yes          | 新しいテーブルの列の名前です。列のデータ型を指定する必要はありません。StarRocksは、列の適切なデータ型を自動的に指定します。StarRocksはFLOATおよびDOUBLEのデータをDECIMAL(38,9)データに変換します。また、CHAR、VARCHAR、およびSTRINGのデータをVARCHAR(65533)データに変換します。 |
| key_desc          | No           | 構文は `key_type ( <col_name1> [, <col_name2> , ...])`です。<br />**パラメータ**:<ul><li>`key_type`:[新しいテーブルのキーのタイプ](../../../table_design/table_types/table_types.md)。有効な値は`DUPLICATE KEY`および`PRIMARY KEY`です。デフォルト値は`DUPLICATE KEY`です。</li><li> `col_name`: キーを形成する列。</li></ul> |
| COMMENT           | No           | 新しいテーブルのコメントです。                                    |
| partition_desc    | No           | 新しいテーブルのパーティション方法です。このパラメータを指定しない場合、デフォルトで新しいテーブルにはパーティションがありません。パーティションについての詳細は、CREATE TABLEを参照してください。 |
| distribution_desc | No           | 新しいテーブルのバケット化方法です。このパラメータを指定しない場合、バケット列は、コストベースの最適化プログラム (CBO)によって収集されたカーディナリティが最も高い列にデフォルトで設定されます。バケット数のデフォルト値は10です。CBOがカーディナリティに関する情報を収集しない場合、バケット列は、新しいテーブルの最初の列にデフォルトで設定されます。バケット化についての詳細は、CREATE TABLEを参照してください。 |
| Properties        | No           | 新しいテーブルのプロパティです。                             |
| AS SELECT query   | Yes          | クエリの結果です。`... AS SELECT query`で列を指定できます。例えば、`... AS SELECT a, b, c FROM table_a;`。この例では、`a`、`b`、および`c`は、問い合わせられているテーブルの列名を示します。新しいテーブルの列名を指定しない場合、新しいテーブルの列名も`a`、`b`、および`c`です。`... AS SELECT query`で式を指定できます。例えば、`... AS SELECT a+1 AS x, b+2 AS y, c*c AS z FROM table_a;`。この例では、`a+1`、`b+2`、および`c*c`は、問い合わせれているテーブルの列名を示し、`x`、`y`、および`z`は、新しいテーブルの列名を示します。注: 新しいテーブルの列の数は、SELECTステートメントで指定された列の数と同じである必要があります。識別しやすい列名を使用することをお勧めします。 |

## Usage notes

- CTASステートメントは、次の要件を満たす新しいテーブルのみを作成できます：
  - `ENGINE`が`OLAP`であること。

  - テーブルはデフォルトでDuplicate Keyテーブルです。`KEY_DESC`でPrimary Keyテーブルとして指定することもできます。

  - ソートキーは最初の3つの列であり、これら3つの列のデータ型のストレージスペースが36バイトを超えていないこと。

- CTASステートメントは、新しく作成されたテーブルにインデックスを設定することをサポートしていません。

- CTASステートメントがリスタートなどの理由で実行に失敗した場合、次のいずれかの問題が発生する場合があります:
  - 新しいテーブルは作成されますが、データが含まれていません。

  - 新しいテーブルの作成に失敗します。

- 新しいテーブルが作成された後、新しいテーブルにデータを挿入するために複数の方法（例えば、INSERT INTO）が使用されると、最初にINSERT操作を完了した方法がそのデータを新しいテーブルに挿入します。

- 新しいテーブルが作成された後、このテーブルに対して手動でユーザーに権限を付与する必要があります。

- テーブルを非同期的に問い合わせて新しいテーブルを作成し、クエリの結果を新しいテーブルに挿入する場合、Taskの名前を指定しない場合、StarRocksはTaskの名前を自動的に生成します。

## Examples

Example 1: テーブル`order`を同期的に問い合わせ、クエリの結果に基づいて新しいテーブル`order_new`を作成し、その後にクエリの結果を新しいテーブルに挿入します。

```SQL
CREATE TABLE order_new
AS SELECT * FROM order;
```

Example 2: テーブル`order`の列`k1`、`k2`、および`k3`を同期的に問い合わせ、クエリの結果に基づいて新しいテーブル`order_new`を作成し、その後にクエリの結果を新しいテーブルに挿入します。さらに、新しいテーブルの列名を`a`、`b`、および`c`に設定します。

```SQL
CREATE TABLE order_new (a, b, c)
AS SELECT k1, k2, k3 FROM order;
```

または

```SQL
CREATE TABLE order_new
AS SELECT k1 AS a, k2 AS b, k3 AS c FROM order;
```

Example 3: テーブル`employee`の`salary`列の最大値を同期的に問い合わせ、クエリの結果に基づいて新しいテーブル`employee_new`を作成し、その後にクエリの結果を新しいテーブルに挿入します。さらに、新しいテーブルの列名を`salary_max`に設定します。

```SQL
CREATE TABLE employee_new
AS SELECT MAX(salary) AS salary_max FROM employee;
```

データが挿入された後、新しいテーブルをクエリします。

```SQL
SELECT * FROM employee_new;

+------------+
| salary_max |
+------------+
|   10000    |
+------------+
```

Example 4: CTASを使用してPrimary Keyテーブルを作成します。Primary Keyテーブルのデータ行数は、クエリの結果に含まれるデータ行よりも少ない場合があります。それは、[Primary Key](../../../table_design/table_types/primary_key_table.md)テーブルは、同じ主キーを持つ行のグループの中で最新のデータ行のみを格納するためです。

```SQL
CREATE TABLE employee_new
PRIMARY KEY(order_id)
AS SELECT
    order_list.order_id,
    sum(goods.price) as total
FROM order_list INNER JOIN goods ON goods.item_id1 = order_list.item_id2
GROUP BY order_id;
```

Example 5: テーブル`lineorder`、`customer`、`supplier`、および`part`を含む4つのテーブルを同期的に問い合わせ、クエリの結果に基づいて新しいテーブル`lineorder_flat`を作成し、その後にクエリの結果を新しいテーブルに挿入します。さらに、新しいテーブルのパーティション方法とバケット化方法を指定します。

```SQL
CREATE TABLE lineorder_flat
PARTITION BY RANGE(`LO_ORDERDATE`)
(
    START ("1993-01-01") END ("1999-01-01") EVERY (INTERVAL 1 YEAR)
)
DISTRIBUTED BY HASH(`LO_ORDERKEY`) AS SELECT
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
```SQL
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

```SQL
テーブル `order_detail` を非同期でクエリし、クエリ結果に基づいて新しいテーブル `order_statistics` を作成し、その後、クエリ結果を新しいテーブルに挿入します。

SUBMIT TASK AS CREATE TABLE order_statistics AS SELECT COUNT(*) as count FROM order_detail;

+-------------------------------------------+-----------+
| TaskName                                  | Status    |
+-------------------------------------------+-----------+
| ctas-df6f7930-e7c9-11ec-abac-bab8ee315bf2 | SUBMITTED |
+-------------------------------------------+-----------+
```

```SQL
タスクの情報を確認します。

SELECT * FROM INFORMATION_SCHEMA.tasks;

-- タスクの情報

TASK_NAME: ctas-df6f7930-e7c9-11ec-abac-bab8ee315bf2
CREATE_TIME: 2022-06-14 14:07:06
SCHEDULE: MANUAL
DATABASE: default_cluster:test
DEFINITION: CREATE TABLE order_statistics AS SELECT COUNT(*) as cnt FROM order_detail
EXPIRE_TIME: 2022-06-17 14:07:06
```

```SQL
TaskRunの状態を確認します。

SELECT * FROM INFORMATION_SCHEMA.task_runs;

-- TaskRunの状態

QUERY_ID: 37bd2b63-eba8-11ec-8d41-bab8ee315bf2
TASK_NAME: ctas-df6f7930-e7c9-11ec-abac-bab8ee315bf2
CREATE_TIME: 2022-06-14 14:07:06
FINISH_TIME: 2022-06-14 14:07:07
STATE: SUCCESS
DATABASE: 
DEFINITION: CREATE TABLE order_statistics AS SELECT COUNT(*) as cnt FROM order_detail
EXPIRE_TIME: 2022-06-17 14:07:06
ERROR_CODE: 0
ERROR_MESSAGE: NULL
```

```SQL
TaskRunの状態が`SUCCESS`の場合、新しいテーブルをクエリします。

SELECT * FROM order_statistics;
```