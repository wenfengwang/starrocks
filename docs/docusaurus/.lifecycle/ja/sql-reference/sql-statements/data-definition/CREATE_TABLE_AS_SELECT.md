---
displayed_sidebar: "Japanese"
---

# SELECT文の結果に基づいてテーブルを作成する

## 説明

CREATE TABLE AS SELECT (CTAS) ステートメントを使用して、テーブルを同期的または非同期的にクエリし、クエリの結果に基づいて新しいテーブルを作成し、そしてクエリの結果を新しいテーブルに挿入できます。

[SUBMIT TASK](../data-manipulation/SUBMIT_TASK.md) を使用して非同期の CTAS タスクを実行できます。

## 構文

- テーブルを同期的にクエリし、クエリの結果に基づいて新しいテーブルを作成し、そしてクエリの結果を新しいテーブルに挿入します。

  ```SQL
  CREATE TABLE [IF NOT EXISTS] [database.]テーブル名
  [(列名 [, 列名2, ...]]
  [key_desc]
  [COMMENT "テーブルコメント"]
  [partition_desc]
  [distribution_desc]
  [PROPERTIES ("key"="value", ...)]
  AS SELECT クエリ
  [ ... ]
  ```

- テーブルを非同期的にクエリし、クエリの結果に基づいて新しいテーブルを作成し、そしてクエリの結果を新しいテーブルに挿入します。

  ```SQL
  SUBMIT [/*+ SET_VAR(key=value) */] TASK [[database.]<task_name>]AS
  CREATE TABLE [IF NOT EXISTS] [database.]テーブル名
  [(列名 [, 列名2, ...]]
  [key_desc]
  [COMMENT "テーブルコメント"]
  [partition_desc]
  [distribution_desc]
  [PROPERTIES ("key"="value", ...)]AS SELECT クエリ
  [ ... ]
  ```

## パラメータ

| **パラメータ**  | **必須** | **説明**                                                      |
| ---------------- | -------- | ------------------------------------------------------------ |
| 列名             | はい     | 新しいテーブルの列名。列のデータ型を指定する必要はありません。StarRocks は、列のために適切なデータ型を自動的に指定します。StarRocks は、FLOAT および DOUBLE データを DECIMAL(38,9) データに変換します。StarRocks はまた、CHAR、VARCHAR、および STRING データを VARCHAR(65533) データに変換します。 |
| key_desc         | なし     | 構文は `key_type ( <col_name1> [, <col_name2> , ...])` です。<br />**パラメータ**: <ul><li>`key_type`: [新しいテーブルのキーのタイプ](../../../table_design/table_types/table_types.md)。有効な値: `DUPLICATE KEY`、`PRIMARY KEY`。デフォルト値: `DUPLICATE KEY`。</li><li> `col_name`: キーを形成する列。</li></ul> |
| COMMENT          | なし     | 新しいテーブルのコメント。                                    |
| partition_desc   | なし     | 新しいテーブルのパーティション方法。このパラメータを指定しない場合、新しいテーブルにはデフォルトでパーティションがありません。パーティションに関する詳細は、CREATE TABLE を参照してください。 |
| distribution_desc| なし     | 新しいテーブルのバケットング方法。このパラメータを指定しない場合、バケット列は最適化コストベースのオプティマイザ（CBO）によって収集された最大の基数を持つ列にデフォルトで設定されます。バケッ トの数はデフォルトで 10 です。CBO が基数に関する情報を収集しない場合、バケット列は新しいテーブルの最初の列にデフォルトで設定されます。バケッ ティングに関する詳細については、CREATE TABLE を参照してください。 |
| Properties       | なし     | 新しいテーブルのプロパティ。                                  |
| AS SELECT クエリ | はい     | クエリの結果。`... AS SELECT クエリ` で列を指定できます。たとえば、`... AS SELECT a, b, c FROM table_a;` のようにです。この例では、`a`、`b`、`c` はクエリされたテーブルの列名を示しています。新しいテーブルの列名を指定しない場合、新しいテーブルの列名も `a`、`b`、`c` になります。`... AS SELECT クエリ` で式を指定できます。たとえば、`... AS SELECT a+1 AS x, b+2 AS y, c*c AS z FROM table_a;` のようにです。この例では、`a+1`、`b+2`、`c*c` はクエリされたテーブルの列名を示し、`x`、`y`、`z` は新しいテーブルの列名を示しています。注: 新しいテーブルの列数は SELECT ステートメントで指定された列数と同じである必要があります。明確に識別可能な列名を使用することをお勧めします。 |

## 使用上の注意

- CTAS ステートメントは、次の条件を満たす新しいテーブルのみを作成できます:
  - `ENGINE` が `OLAP` であること。

  - テーブルがデフォルトで Duplicate Key テーブルであること。また、`key_desc` で Primary Key テーブルとして指定することもできます。

  - ソートキーは最初の 3 列であること。これらの 3 列のデータ型のストレージスペースが 36 バイトを超えないこと。

- CTAS ステートメントは、新しく作成されたテーブルにインデックスを設定することをサポートしていません。

- CTAS ステートメントが FE の再起動などの理由で実行に失敗した場合、次のいずれかの問題が発生する可能性があります:
  - 新しいテーブルが作成されましたが、データが含まれていません。

  - 新しいテーブルの作成に失敗しました。

- 新しいテーブルが作成された後、新しいテーブルにデータを挿入するために複数の方法（INSERT INTO など）が使用された場合、最初に INSERT 操作を完了した方法がデータを新しいテーブルに挿入します。

- 新しいテーブルが作成された後、このテーブルに対してユーザーに手動で権限を付与する必要があります。

- テーブルを非同期的にクエリし、クエリの結果に基づいて新しいテーブルを作成するときにタスクに名前を指定しない場合、StarRocks は自動的にタスクに名前を生成します。

## 例

例 1: テーブル `order` を同期的にクエリし、クエリの結果に基づいて新しいテーブル `order_new` を作成し、そしてクエリの結果を新しいテーブルに挿入します。

```SQL
CREATE TABLE order_new
AS SELECT * FROM order;
```

例 2: テーブル `order` の `k1`、`k2`、`k3` 列を同期的にクエリし、クエリの結果に基づいて新しいテーブル `order_new` を作成し、そしてクエリの結果を新しいテーブルに挿入します。さらに、新しいテーブルの列名を `a`、`b`、`c` に設定します。

```SQL
CREATE TABLE order_new (a, b, c)
AS SELECT k1, k2, k3 FROM order;
```

または

```SQL
CREATE TABLE order_new
AS SELECT k1 AS a, k2 AS b, k3 AS c FROM order;
```

例 3: テーブル `employee` の `salary` 列の最大値を同期的にクエリし、クエリの結果に基づいて新しいテーブル `employee_new` を作成し、そしてクエリの結果を新しいテーブルに挿入します。さらに、新しいテーブルの列名を `salary_max` に設定します。

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

例 4: Primary Key テーブルを作成するために CTAS を使用します。Primary Key テーブルのデータ行数がクエリ結果の行数より少ないことに注意してください。これは、[Primary Key](../../../table_design/table_types/primary_key_table.md) テーブルは、同じ主キーを持つ行のグループの中で最新のデータ行のみを格納するためです。

```SQL
CREATE TABLE employee_new
PRIMARY KEY(order_id)
AS SELECT
    order_list.order_id,
    sum(goods.price) as total
FROM order_list INNER JOIN goods ON goods.item_id1 = order_list.item_id2
GROUP BY order_id;
```

例 5: テーブル `lineorder`、`customer`、`supplier`、`part` を含む 4 つのテーブルを同期的にクエリし、クエリの結果に基づいて新しいテーブル `lineorder_flat` を作成し、そしてクエリの結果を新しいテーブルに挿入します。さらに、新しいテーブルのパーティション方法とバケットング方法を指定します。

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
```
```plaintext
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

```plaintext
テーブル `order_detail` を非同期でクエリし、クエリ結果に基づいて新しいテーブル `order_statistics` を作成し、その後クエリ結果を新しいテーブルに挿入します。

+-------------------------------------------+-----------+
| TaskName                                  | Status    |
+-------------------------------------------+-----------+
| ctas-df6f7930-e7c9-11ec-abac-bab8ee315bf2 | SUBMITTED |
+-------------------------------------------+-----------+
```

```SQL
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
SELECT * FROM INFORMATION_SCHEMA.task_runs;

-- タスクランの状態

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
タスクランの状態が `SUCCESS` の場合、新しいテーブルをクエリします。

SELECT * FROM order_statistics;
```