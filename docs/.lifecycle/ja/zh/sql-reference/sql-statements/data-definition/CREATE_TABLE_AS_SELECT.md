---
displayed_sidebar: Chinese
---

# CREATE TABLE AS SELECT

## 機能

CREATE TABLE AS SELECT（略称 CTAS）は、元のテーブルを同期的または非同期的にクエリし、その結果に基づいて新しいテーブルを作成し、その結果を新しいテーブルに挿入するために使用されます。

非同期 CTAS タスクを作成するには、[SUBMIT TASK](../data-manipulation/SUBMIT_TASK.md) を使用します。

## 文法

- 元のテーブルを同期的にクエリし、その結果に基づいて新しいテーブルを作成し、その結果を新しいテーブルに挿入します。

  ```SQL
  CREATE TABLE [IF NOT EXISTS] [database.]table_name
  [(column_name [, column_name2, ...])]
  [key_desc]
  [COMMENT "table comment"]
  [partition_desc]
  [distribution_desc]
  [PROPERTIES ("key"="value", ...)] AS SELECT query
  [ ... ]
  ```

- 元のテーブルを非同期的にクエリし、その結果に基づいて新しいテーブルを作成し、その結果を新しいテーブルに挿入します。

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

## パラメータ説明

| **パラメータ**    | **必須** | **説明**                                                     |
| ----------------- | -------- | ------------------------------------------------------------ |
| column_name       | はい     | 新しいテーブルの列名。列の型を指定する必要はありません。StarRocks は適切な列の型を自動的に選択し、FLOAT と DOUBLE を DECIMAL(38,9) に、CHAR、VARCHAR、STRING を VARCHAR(65533) に変換します。 |
| key_desc          | いいえ   | 構文は `key_type (<col_name1> [, <col_name2>, ...])` です。<br />**パラメータ**：<ul><li>`key_type`：新しいテーブルのキーの型。有効な値：`DUPLICATE KEY` と `PRIMARY KEY`。デフォルト値：`DUPLICATE KEY`。</li><li>`col_name`：キーを構成する列。</li></ul>|
| COMMENT           | いいえ   | 新しいテーブルのコメント。                                     |
| partition_desc    | いいえ   | 新しいテーブルのパーティション方式。このパラメータを指定しない場合、デフォルトではパーティションがないテーブルになります。パーティションの設定については、CREATE TABLE を参照してください。 |
| distribution_desc | いいえ   | 新しいテーブルのバケット方式。このパラメータを指定しない場合、デフォルトでは CBO オプティマイザーが収集した統計情報の中で基数が最も高い列がバケット列になり、バケット数はデフォルトで 10 になります。CBO オプティマイザーが基数情報を収集していない場合、デフォルトでは新しいテーブルの最初の列がバケット列になります。バケットの設定については、CREATE TABLE を参照してください。 |
| PROPERTIES        | いいえ   | 新しいテーブルのプロパティ。                                   |
| AS SELECT クエリ   | はい     | クエリ結果。このパラメータは以下の値をサポートします：列。例：`... AS SELECT a, b, c FROM table_a;` ここで `a`、`b`、`c` は元のテーブルの列名です。新しいテーブルに列名を指定していない場合、新しいテーブルの列名も `a`、`b`、`c` になります。式。例：`... AS SELECT a+1 AS x, b+2 AS y, c*c AS z FROM table_a;` ここで `a+1`、`b+2`、`c*c` は元のテーブルの列名で、`x`、`y`、`z` は新しいテーブルの列名です。注意：新しいテーブルの列数は `AS SELECT クエリ` で指定された列数と一致している必要があります。新しいテーブルの列には、後で識別しやすいようにビジネス上の意味を持つ名前を設定することをお勧めします。|

## 注意事項

- CTAS 文を使用して作成された新しいテーブルは、以下の条件を満たす必要があります：
  - `ENGINE` のタイプは `OLAP` です。

  - テーブルのタイプはデフォルトで詳細テーブルですが、`key_desc` で主キーテーブルとして指定することもできます。

  - ソート列は最初の3列で、これらの列のストレージサイズは36バイトを超えてはいけません。

- CTAS 文は新しいテーブルにインデックスを設定することをサポートしていません。

- FE の再起動やその他の理由で CTAS 文の実行に失敗した場合、以下の状況が発生する可能性があります：
  - 新しいテーブルは作成されましたが、データはありません。

  - 新しいテーブルの作成に失敗しました。

- 新しいテーブルが作成された後、複数の方法（例えば Insert Into）でデータを新しいテーブルに挿入することができますが、最初に挿入操作を完了したものが最初にデータを新しいテーブルに挿入します。

- 新しいテーブルが作成された後、ユーザーにそのテーブルへの権限を手動で付与する必要があります。[テーブル権限](../../../administration/privilege_item.md#テーブル権限-table) および [GRANT](../account-management/GRANT.md) を参照してください。
- 非同期で元のテーブルをクエリし、その結果に基づいて新しいテーブルを作成する場合、タスク名を指定しないと、StarRocks は自動的にタスク名を生成します。

## 例

例1：元のテーブル `order` を同期的にクエリし、その結果に基づいて新しいテーブル `order_new` を作成し、その結果を新しいテーブルに挿入します。

```SQL
CREATE TABLE order_new
AS SELECT * FROM order;
```

例2：元のテーブル `order` の `k1`、`k2`、`k3` 列を同期的にクエリし、その結果に基づいて新しいテーブル `order_new` を作成し、その結果を新しいテーブルに挿入します。新しいテーブルの列名を `a`、`b`、`c` として指定します。

```SQL
CREATE TABLE order_new (a, b, c)
AS SELECT k1, k2, k3 FROM order;
```

または

```SQL
CREATE TABLE order_new
AS SELECT k1 AS a, k2 AS b, k3 AS c FROM order;
```

例3：元のテーブル `employee` の `salary` 列の最大値を同期的にクエリし、その結果に基づいて新しいテーブル `employee_new` を作成し、その結果を新しいテーブルに挿入します。新しいテーブルの列名を `salary_max` として指定します。

```SQL
CREATE TABLE employee_new
AS SELECT MAX(salary) AS salary_max FROM employee;
```

挿入が完了したら、新しいテーブル `employee_new` をクエリします。

```SQL
SELECT * FROM employee_new;

+------------+
| salary_max |
+------------+
|   10000    |
+------------+
```

例4：CTAS を使用して主キーテーブルを作成します。主キーテーブルには、クエリ結果のデータ行数よりも少ないデータ行が含まれる可能性があることに注意してください。これは、主キーテーブルが同じ主キーを持つデータ行グループの中で最新のデータ行のみを格納するためです。

```SQL
CREATE TABLE employee_new
PRIMARY KEY(order_id)
AS SELECT
    order_list.order_id,
    sum(goods.price) as total
FROM order_list INNER JOIN goods ON goods.item_id1 = order_list.item_id2
GROUP BY order_id;
```

例5：元のテーブル `lineorder`、`customer`、`supplier`、`part` を同期的にクエリし、その結果に基づいて新しいテーブル `lineorder_flat` を作成し、その結果を新しいテーブルに挿入します。新しいテーブルのパーティションとバケット方式を指定します。

```SQL
CREATE TABLE lineorder_flat
PARTITION BY RANGE (`LO_ORDERDATE`)
(
    START ("1993-01-01") END ("1999-01-01") EVERY (INTERVAL 1 YEAR)
)
DISTRIBUTED BY HASH (`LO_ORDERKEY`) BUCKETS 10 AS SELECT
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
    p.P_CONTAINER AS P_CONTAINER
FROM lineorder AS l 
INNER JOIN customer AS c ON c.C_CUSTKEY = l.LO_CUSTKEY
INNER JOIN supplier AS s ON s.S_SUPPKEY = l.LO_SUPPKEY
INNER JOIN part AS p ON p.P_PARTKEY = l.LO_PARTKEY;
```

例6：元のテーブル `order_detail` を非同期的にクエリし、その結果に基づいて新しいテーブル `order_statistics` を作成し、その結果を新しいテーブルに挿入します。

```Plain_Text
SUBMIT TASK AS CREATE TABLE order_statistics AS SELECT COUNT(*) as count FROM order_detail;

+-------------------------------------------+-----------+
| TaskName                                  | Status    |
+-------------------------------------------+-----------+
| ctas-df6f7930-e7c9-11ec-abac-bab8ee315bf2 | SUBMITTED |
+-------------------------------------------+-----------+
```

タスクの情報をクエリします。

```SQL
SELECT * FROM INFORMATION_SCHEMA.tasks;

-- タスク情報は以下の通りです

TASK_NAME: ctas-df6f7930-e7c9-11ec-abac-bab8ee315bf2
CREATE_TIME: 2022-06-14 14:07:06
SCHEDULE: MANUAL
DATABASE: default_cluster:test
DEFINITION: CREATE TABLE order_statistics AS SELECT COUNT(*) as cnt FROM order_detail
EXPIRE_TIME: 2022-06-17 14:07:06
```

タスクランの状態をクエリします。

```SQL
SELECT * FROM INFORMATION_SCHEMA.task_runs;

-- タスクランの状態は以下の通りです

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

タスクランの状態が `SUCCESS` である場合、新しいテーブルをクエリできます。

```SQL
SELECT * FROM order_statistics;
```
