---
displayed_sidebar: Chinese
---

# SELECT

## 機能

SELECT文は、単一または複数のテーブル、ビュー、マテリアライズドビューからデータを読み取るために使用されます。SELECT文は通常、以下の句で構成されます：

- [SELECT](#select)
  - [機能](#機能)
    - [WITH](#with)
    - [結合 (Join)](#连接-join)
      - [セルフジョイン](#self-join)
      - [クロスジョイン (Cross Join)](#笛卡尔积-cross-join)
      - [インナージョイン](#inner-join)
      - [アウタージョイン](#outer-join)
      - [セミジョイン](#semi-join)
      - [アンチジョイン](#anti-join)
      - [等値結合と非等値結合](#等值-join-和非等值-join)
    - [ORDER BY](#order-by)
    - [GROUP BY](#group-by)
  - [文法](#语法)
    - [パラメータ](#parameters)
    - [注記](#note)
  - [例](#示例)
    - [HAVING](#having)
    - [LIMIT](#limit)
      - [OFFSET](#offset)
    - [**UNION**](#union)
    - [**INTERSECT**](#intersect)
    - [**EXCEPT/MINUS**](#exceptminus)
    - [DISTINCT](#distinct)
    - [サブクエリ](#子查询)
      - [非相関サブクエリ](#不相关子查询)
      - [相関サブクエリ](#相关子查询)
    - [WHEREと演算子](#where-与操作符)
      - [算術演算子](#算数操作符)
      - [BETWEEN演算子](#between-操作符)
      - [比較演算子](#比较操作符)
      - [IN演算子](#in-操作符)
      - [LIKE演算子](#like-操作符)
      - [論理演算子](#逻辑操作符)
      - [正規表現演算子](#正则表达式操作符)
    - [エイリアス](#别名-alias)

SELECTは、独立した文としても、他の文の句としても使用でき、そのクエリ結果は別の文の入力値として機能します。

StarRocksのクエリ文は、基本的にSQL-92標準に準拠しています。以下に、サポートされているSELECTの使用法を紹介します。

> **説明**
>
> StarRocksのテーブル、ビュー、またはマテリアライズドビュー内のデータをクエリするには、対応するオブジェクトのSELECT権限が必要です。External Catalog内のデータをクエリするには、対応するCatalogのUSAGE権限が必要です。

### WITH

SELECT文の前に追加できる句で、SELECT内で複数回参照される複雑な式のエイリアスを定義するために使用されます。

CREATE VIEWと似ていますが、句で定義されたテーブル名と列名はクエリ終了後には持続せず、実際のテーブルやVIEWの名前と衝突することはありません。

WITH句を使用する利点：

- 便利でメンテナンスが容易で、クエリ内の重複を減らすことができます。

- クエリの最も複雑な部分を個別のブロックとして抽象化することで、SQLコードを読みやすく、理解しやすくなります。

例：

```sql
-- Define one subquery at the outer level, and another at the inner level as part of the
-- initial stage of the UNION ALL query.

with t1 as (select 1),t2 as (select 2)
select * from t1 union all select * from t2;
```

### 結合 (Join)

結合操作は2つ以上のテーブルのデータを結合し、特定のテーブルの特定の列の結果セットを返します。

現在、StarRocksはセルフジョイン、クロスジョイン、インナージョイン、アウタージョイン、セミジョイン、アンチジョインをサポートしています。アウタージョインには、レフトジョイン、ライトジョイン、フルジョインが含まれます。

結合の文法は以下のように定義されています：

```sql
SELECT select_list FROM
table_or_subquery1 [INNER] JOIN table_or_subquery2 |
table_or_subquery1 {LEFT [OUTER] | RIGHT [OUTER] | FULL [OUTER]} JOIN table_or_subquery2 |
table_or_subquery1 {LEFT | RIGHT} SEMI JOIN table_or_subquery2 |
table_or_subquery1 {LEFT | RIGHT} ANTI JOIN table_or_subquery2 |
[ ON col1 = col2 [AND col3 = col4 ...] |
USING (col1 [, col2 ...]) ]
[other_join_clause ...]
[ WHERE where_clauses ]
```

```sql
SELECT select_list FROM
table_or_subquery1, table_or_subquery2 [, table_or_subquery3 ...]
[other_join_clause ...]
WHERE
col1 = col2 [AND col3 = col4 ...]
```

```sql
SELECT select_list FROM
table_or_subquery1 CROSS JOIN table_or_subquery2
[other_join_clause ...]
[ WHERE where_clauses ]
```

#### セルフジョイン

StarRocksはセルフジョインをサポートしています。つまり、自分自身との結合です。例えば、同じテーブルの異なる列を結合する場合です。

実際には、セルフジョインを特別に示す文法はありません。セルフジョインでは、結合する両側の条件が同じテーブルから来ているため、異なるエイリアスを割り当てる必要があります。

例えば：

```sql
SELECT lhs.id, rhs.parent, lhs.c1, rhs.c2 FROM tree_data lhs, tree_data rhs WHERE lhs.id = rhs.parent;
```

#### クロスジョイン (Cross Join)

クロスジョインは多数の結果を生成するため、慎重に使用する必要があります。

クロスジョインを使用する場合でも、フィルタリング条件を使用し、返される結果の数を少なくする必要があります。例えば：

```sql
SELECT * FROM t1, t2;

SELECT * FROM t1 CROSS JOIN t2;
```

#### インナージョイン

インナージョインは最もよく知られている、最も一般的に使用される結合です。結果は、結合条件が両方のテーブルの列に同じ値を含む、2つの関連するテーブルから要求された列から来ます。

2つのテーブルに同じ名前の列がある場合は、フルネーム（table_name.column_name形式）を使用するか、列にエイリアスを付ける必要があります。

例えば：

以下の3つのクエリは同等です。

```sql
SELECT t1.id, c1, c2 FROM t1, t2 WHERE t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 JOIN t2 ON t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id = t2.id;
```

#### アウタージョイン

アウタージョインは、左側のテーブル、右側のテーブル、またはその両方のすべての行を返します。もう一方のテーブルに一致するデータがない場合は、`NULL`に設定されます。例えば：

```sql
SELECT * FROM t1 LEFT OUTER JOIN t2 ON t1.id = t2.id;

SELECT * FROM t1 RIGHT OUTER JOIN t2 ON t1.id = t2.id;

SELECT * FROM t1 FULL OUTER JOIN t2 ON t1.id = t2.id;
```

#### セミジョイン

レフトセミジョインは、左側のテーブルに右側のテーブルのデータに一致する行のみを返します。右側のテーブルに何行一致するデータがあっても、左側のテーブルのその行は最大で一度だけ返されます。ライトセミジョインの原理は似ていますが、返されるデータは右側のテーブルのものです。

例えば：

```sql
SELECT t1.c1, t1.c2, t1.c2 FROM t1 LEFT SEMI JOIN t2 ON t1.id = t2.id;
```

#### アンチジョイン

レフトアンチジョインは、左側のテーブルに右側のテーブルのデータに一致しない行のみを返します。

ライトアンチジョインはこの比較を逆にし、右側のテーブルに左側のテーブルのデータに一致しない行のみを返します。例えば：

```sql
SELECT t1.c1, t1.c2, t1.c2 FROM t1 LEFT ANTI JOIN t2 ON t1.id = t2.id;
```

#### 等値結合と非等値結合

結合条件に応じて、StarRocksがサポートする上記の各種結合は、等値結合と非等値結合に分けられます。以下の表に示す通りです：

| **等値結合**   | セルフジョイン、クロスジョイン、インナージョイン、アウタージョイン、セミジョイン、アンチジョイン |
| --------------- | ------------------------------------------------------------ |
| **非等値結合** | クロスジョイン、インナージョイン、レフトセミジョイン、レフトアンチジョイン、アウタージョイン  |

- 等値結合
  
  等値結合は等値条件を結合条件として使用します。例えば `a JOIN b ON a.id = b.id`。

- 非等値結合
  
  非等値結合は等値条件を使用せず、`<`、`<=`、`>`、`>=`、`<>`などの比較演算子を使用します。例えば `a JOIN b ON a.id < b.id`。等値結合と比較して、非等値結合は現在効率が低いため、慎重に使用することをお勧めします。

  以下は非等値結合の2つの例です：

  ```SQL
  SELECT t1.id, c1, c2 
  FROM t1 
  INNER JOIN t2 ON t1.id < t2.id;
    
  SELECT t1.id, c1, c2 
  FROM t1 
  LEFT JOIN t2 ON t1.id > t2.id;
  ```

### ORDER BY

ORDER BYは、一つまたは複数の列を比較して結果セットを並べ替えるために使用されます。

ORDER BYは時間とリソースを多く消費する操作であり、すべてのデータを1つのノードに送信してから並べ替える必要があるため、より多くのメモリが必要です。


LIMIT 子句を使用すると、上位 N 件のソート結果を返すことができます。メモリ使用量を制限するために、LIMIT 子句を指定しない場合、デフォルトで上位 65535 件のソート結果が返されます。

ORDER BY の構文は以下の通りです:

```sql
ORDER BY col [ASC | DESC]
[ NULLS FIRST | NULLS LAST ]
```

デフォルトのソート順は ASC（昇順）です。例：

```sql
select * from big_table order by tiny_column, short_column desc;
```

StarRocks では、ORDER BY の後に NULL 値が最初に来るか最後に来るかを宣言することができます。構文は `order by <> [ NULLS FIRST | NULLS LAST ]` です。`NULLS FIRST` は NULL 値のレコードが最初に来ることを意味し、`NULLS LAST` は NULL 値のレコードが最後に来ることを意味します。

例：NULL 値を常に最初にする。

```sql
select * from sales_record order by employee_id nulls first;
```

### GROUP BY

GROUP BY 句は通常、集約関数（COUNT(), SUM(), AVG(), MIN(), MAX() など）と共に使用されます。

GROUP BY で指定された列は集約操作には含まれません。GROUP BY 句には HAVING 句を追加して、集約関数の結果をフィルタリングすることができます。例：

```sql
select tiny_column, sum(short_column)
from small_table 
group by tiny_column;
```

```sql
+-------------+---------------------+
| tiny_column | sum(short_column)   |
+-------------+---------------------+
| 1           | 2                   |
| 2           | 1                   |
+-------------+---------------------+
```

## 構文

  ```sql
  SELECT ...
  FROM ...
  [ ... ]
  GROUP BY [
      , ... |
      GROUPING SETS [, ...] ( groupSet [ , groupSet [ , ... ] ] ) |
      ROLLUP(expr [ , expr [ , ... ] ]) |
      expr [ , expr [ , ... ] ] WITH ROLLUP |
      CUBE(expr [ , expr [ , ... ] ]) |
      expr [ , expr [ , ... ] ] WITH CUBE
      ]
  [ ... ]
  ```

### パラメーター

- `groupSet` は、select list 内の列、エイリアス、または式の集合を表します `groupSet ::= { ( expr [ , expr [ , ... ] ] )}`。

- `expr` は、select list 内の列、エイリアス、または式を表します。

### 注意

StarRocks は PostgreSQL に似た構文をサポートしており、実際の構文例は以下の通りです:

  ```sql
  SELECT a, b, SUM(c) FROM tab1 GROUP BY GROUPING SETS ((a, b), (a), (b), ());
  SELECT a, b, c, SUM(d) FROM tab1 GROUP BY ROLLUP(a, b, c)
  SELECT a, b, c, SUM(d) FROM tab1 GROUP BY CUBE(a, b, c)
  ```

`ROLLUP(a, b, c)` は以下の `GROUPING SETS` 文に相当します。

  ```sql
  GROUPING SETS (
  (a, b, c),
  (a, b),
  (a),
  ()
  )
  ```

`CUBE(a, b, c)` は以下の `GROUPING SETS` 文に相当します。

  ```sql
  GROUPING SETS (
  (a, b, c),
  (a, b),
  (a, c),
  (a),
  (b, c),
  (b),
  (c),
  ()
  )
  ```

## 例

実際のデータの例は以下の通りです:

  ```sql
  SELECT * FROM t;
  +------+------+------+
  | k1   | k2   | k3   |
  +------+------+------+
  | a    | A    | 1    |
  | a    | A    | 2    |
  | a    | B    | 1    |
  | a    | B    | 3    |
  | b    | A    | 1    |
  | b    | A    | 4    |
  | b    | B    | 1    |
  | b    | B    | 5    |
  +------+------+------+
  8 rows in set (0.01 sec)

  SELECT k1, k2, SUM(k3) FROM t
  GROUP BY GROUPING SETS ((k1, k2), (k2), (k1), ());
  +------+------+-----------+
  | k1   | k2   | sum(k3)   |
  +------+------+-----------+
  | b    | B    | 6         |
  | a    | B    | 4         |
  | a    | A    | 3         |
  | b    | A    | 5         |
  | NULL | B    | 10        |
  | NULL | A    | 8         |
  | a    | NULL | 7         |
  | b    | NULL | 11        |
  | NULL | NULL | 18        |
  +------+------+-----------+
  9 rows in set (0.06 sec)

  SELECT k1, k2, GROUPING_ID(k1, k2), SUM(k3) FROM t
  GROUP BY GROUPING SETS ((k1, k2), (k1), (k2), ());
  +------+------+-------------------+-----------+
  | k1   | k2   | GROUPING_ID(k1, k2) | sum(k3) |
  +------+------+-------------------+-----------+
  | a    | A    | 0                 | 3         |
  | a    | B    | 0                 | 4         |
  | a    | NULL | 1                 | 7         |
  | b    | A    | 0                 | 5         |
  | b    | B    | 0                 | 6         |
  | b    | NULL | 1                 | 11        |
  | NULL | A    | 2                 | 8         |
  | NULL | B    | 2                 | 10        |
  | NULL | NULL | 3                 | 18        |
  +------+------+-------------------+-----------+
  9 rows in set (0.02 sec)
  ```

GROUP BY `GROUPING SETS`、`CUBE`、`ROLLUP` は GROUP BY 句の拡張で、1つの GROUP BY 句で複数の集合のグループ化集約を実現することができます。その結果は、複数の対応する GROUP BY 句を UNION 操作することと同等です。

GROUP BY 句は、1つの要素のみを含む GROUP BY `GROUPING SETS` の特別なケースです。
例えば、GROUPING SETS 文は：

  ```sql
  SELECT a, b, SUM(c) FROM tab1 GROUP BY GROUPING SETS ((a, b), (a), (b), ());
  ```

そのクエリ結果は以下と同等です：

  ```sql
  SELECT a, b, SUM(c) FROM tab1 GROUP BY a, b
  UNION
  SELECT a, NULL, SUM(c) FROM tab1 GROUP BY a
  UNION
  SELECT NULL, b, SUM(c) FROM tab1 GROUP BY b
  UNION
  SELECT NULL, NULL, SUM(c) FROM tab1
  ```

`GROUPING(expr)` は列が集約列かどうかを示します。集約列の場合は 0、そうでない場合は 1 です。

`GROUPING_ID(expr [ , expr [ , ... ] ])` は GROUPING と似ていますが、GROUPING_ID は指定された列の順序に基づいて列リストのビットマップ値を計算し、各ビットは GROUPING の値です。GROUPING_ID() 関数はビットベクトルの十進数値を返します。

### HAVING

HAVING 句はテーブルの行データをフィルタリングするのではなく、集約関数の結果をフィルタリングします。

通常、HAVING は集約関数（COUNT(), SUM(), AVG(), MIN(), MAX() など）および GROUP BY 句と共に使用されます。

例：

```sql
select tiny_column, sum(short_column) 
from small_table 
group by tiny_column 
having sum(short_column) = 1;
```

```sql
+-------------+---------------------+
| tiny_column | sum(short_column)   |
+-------------+---------------------+
| 2           | 1                   |
+-------------+---------------------+

1 row in set (0.07 sec)
```

```sql
select tiny_column, sum(short_column) 
from small_table 
group by tiny_column 
having tiny_column > 1;
```

```sql
+-------------+---------------------+
| tiny_column | sum(short_column)   |
+-------------+---------------------+
| 2           | 1                   |
+-------------+---------------------+

1 row in set (0.07 sec)
```

### LIMIT

LIMIT 句は、返される結果の最大行数を制限するために使用されます。返される結果の最大行数を設定することで、StarRocks はメモリの使用を最適化するのに役立ちます。

この句は主に以下のシナリオで使用されます：

1. 上位 N 件のクエリ結果を返す。

2. テーブルに含まれる内容を簡単に確認する。

3. テーブルのデータ量が多い、または WHERE 句であまり多くのデータがフィルタリングされていない場合、クエリ結果セットのサイズを制限する必要があります。

使用説明：LIMIT 句の値は数値リテラルでなければなりません。

例：

```sql
mysql> select tiny_column from small_table limit 1;

+-------------+
| tiny_column |
+-------------+
| 1           |
+-------------+

1 row in set (0.02 sec)
```

```sql
mysql> select tiny_column from small_table limit 10000;

+-------------+
| tiny_column |
+-------------+
| 1           |
| 2           |
+-------------+

2 rows in set (0.01 sec)
```

#### OFFSET

OFFSET 句は、結果セットの最初のいくつかの行をスキップし、その後の結果を直接返します。

デフォルトでは最初の 0 行から始まるため、OFFSET 0 は OFFSET なしと同じ結果を返します。

通常、OFFSET 句は ORDER BY 句および LIMIT 句と共に使用されると効果的です。

例：

```sql
mysql> select varchar_column from big_table order by varchar_column limit 3;

+----------------+
| varchar_column |
+----------------+
|    beijing     | 
|    chongqing   | 
|    tianjin     | 
+----------------+

3行がセットされました (0.02秒)
```

```sql
mysql> select varchar_column from big_table order by varchar_column limit 1 offset 0;

+----------------+
|varchar_column  |
+----------------+
|     beijing    |
+----------------+

1行がセットされました (0.01秒)
```

```sql
mysql> select varchar_column from big_table order by varchar_column limit 1 offset 1;

+----------------+
|varchar_column  |
+----------------+
|    chongqing   | 
+----------------+

1行がセットされました (0.01秒)
```

```sql
mysql> select varchar_column from big_table order by varchar_column limit 1 offset 2;

+----------------+
|varchar_column  |
+----------------+
|     tianjin    |     
+----------------+

1行がセットされました (0.02秒)
```

> **注意**
>
> ORDER BY がない場合に OFFSET を使用することは許されていますが、その場合 OFFSET は意味をなしません。このような場合は LIMIT の値のみが取得され、OFFSET の値は無視されます。したがって、ORDER BY がない場合でも、OFFSET が結果セットの最大行数を超えていても結果が得られます。**OFFSET を使用する場合は、必ず ORDER BY を付けることをお勧めします。**

### **UNION**

UNION 句は複数のクエリ結果を結合するために使用され、つまり和集合を取得します。

**以下のような構文です：**

```SQL
query_1 UNION [DISTINCT | ALL] query_2
```

- DISTINCT：デフォルト値で、重複しない結果を返します。UNION と UNION DISTINCT は同じ効果を持ちます（第二の例を参照）。
- ALL: すべての結果を集合として返し、重複を排除しません。重複排除作業はメモリを多く消費するため、UNION ALL を使用するとクエリの速度が速くなり、メモリ消費も少なくなります（第一の例を参照）。

> **説明**
>
> 各 SELECT クエリは同じ数の列を返す必要があり、列の型は互換性がある必要があります。

**例：**

`select1` と `select2` のテーブルを例に説明します。

```SQL
CREATE TABLE select1(
    id          INT,
    price       INT
    )
DISTRIBUTED BY HASH(id);

INSERT INTO select1 VALUES
    (1,2),
    (1,2),
    (2,3),
    (5,6),
    (5,6);

CREATE TABLE select2(
    id          INT,
    price       INT
    )
DISTRIBUTED BY HASH(id);

INSERT INTO select2 VALUES
    (2,3),
    (3,4),
    (5,6),
    (7,8);
```

例1：2つのテーブルのすべての `id` の和集合を返し、重複を排除しません。

```Plaintext
mysql> (select id from select1) union all (select id from select2) order by id;

+------+
| id   |
+------+
|    1 |
|    1 |
|    2 |
|    2 |
|    3 |
|    5 |
|    5 |
|    5 |
|    7 |
+------+
11行がセットされました (0.02秒)
```

例2：2つのテーブルの `id` の和集合を返し、重複を排除します。以下の2つのクエリは機能的に等価です。

```Plaintext
mysql> (select id from select1) union (select id from select2) order by id;

+------+
| id   |
+------+
|    1 |
|    2 |
|    3 |
|    5 |
|    7 |
+------+
6行がセットされました (0.01秒)

mysql> (select id from select1) union distinct (select id from select2) order by id;
+------+
| id   |
+------+
|    1 |
|    2 |
|    3 |
|    5 |
|    7 |
+------+
5行がセットされました (0.02秒)
```

例3：2つのテーブルのすべての `id` の和集合を返し、重複を排除し、結果として3行のみを返します。以下の2つのクエリは機能的に等価です。

```SQL
mysql> (select id from select1) union distinct (select id from select2)
order by id
limit 3;
+------+
| id   |
+------+
|    1 |
|    2 |
|    3 |
+------+
4行がセットされました (0.11秒)

mysql> select * from (select id from select1 union distinct select id from select2) as t1
order by id
limit 3;
+------+
| id   |
+------+
|    1 |
|    2 |
|    3 |
+------+
3行がセットされました (0.01秒)
```

### **INTERSECT**

INTERSECT 句は複数のクエリ結果の交集を返すために使用され、つまり各結果に共通するデータを返し、結果セットを重複排除します。

**以下のような構文です：**

```SQL
query_1 INTERSECT [DISTINCT] query_2
```

> **説明**
>
> - INTERSECT は INTERSECT DISTINCT と同じ効果を持ちます。ALL キーワードはサポートされていません。
> - 各 SELECT クエリは同じ数の列を返す必要があり、列の型は互換性がある必要があります。

**例：**

UNION 句の2つのテーブルを使用し続けます。2つのテーブルの `id` と `price` の組み合わせの交集を返します。以下の2つのクエリは機能的に等価です。

```Plaintext
mysql> (select id, price from select1) intersect (select id, price from select2)
order by id;

+------+-------+
| id   | price |
+------+-------+
|    2 |     3 |
|    5 |     6 |
+------+-------+

mysql> (select id, price from select1) intersect distinct (select id, price from select2)
order by id;

+------+-------+
| id   | price |
+------+-------+
|    2 |     3 |
|    5 |     6 |
+------+-------+
```

### **EXCEPT/MINUS**

EXCEPT/MINUS 句は複数のクエリ結果の補集合を返すために使用され、つまり左側のクエリに含まれ、右側のクエリには存在しないデータを返し、結果セットを重複排除します。EXCEPT と MINUS は機能的に等価です。

**以下のような構文です：**

```SQL
query_1 {EXCEPT | MINUS} [DISTINCT] query_2
```

> **説明**
>
> - EXCEPT は EXCEPT DISTINCT と同じ効果を持ちます。ALL キーワードはサポートされていません。
> - 各 SELECT クエリは同じ数の列を返す必要があり、列の型は互換性がある必要があります。

**例：**

UNION 句の2つのテーブルを使用し続けます。`select1` テーブルに含まれ、`select2` テーブルには存在しない `(id, price)` の組み合わせを返します。

結果では `(1,2)` の組み合わせが重複排除されていることがわかります。

```Plaintext
mysql> (select id, price from select1) except (select id, price from select2)
order by id;
+------+-------+
| id   | price |
+------+-------+
|    1 |     2 |
+------+-------+

mysql> (select id, price from select1) minus (select id, price from select2)
order by id;
+------+-------+
| id   | price |
+------+-------+
|    1 |     2 |
+------+-------+
```

### DISTINCT

DISTINCT キーワードは結果セットから重複を排除します。例：

```SQL
-- 一列から重複を排除した値を返します。
select distinct tiny_column from big_table limit 2;

-- 複数列から重複を排除した組み合わせを返します。
select distinct tiny_column, int_column from big_table limit 2;
```

DISTINCT は集約関数（通常は count）と一緒に使用することができ、count(distinct) は一つの列または複数列に含まれる異なる組み合わせがいくつあるかを計算します。

```SQL
-- 一列から異なる値の数を計算します。
select count(distinct tiny_column) from small_table;
```

```sql
+-------------------------------+
| count(DISTINCT 'tiny_column') |
+-------------------------------+
|             2                 |
+-------------------------------+
1行がセットされました (0.06秒)
```

```SQL
-- 複数列から異なる組み合わせの数を計算します。
select count(distinct tiny_column, int_column) from big_table limit 2;
```

StarRocks は複数の集約関数が同時に distinct を使用することをサポートしています。

```SQL
-- 複数の集約関数が重複を排除した後の数を個別に返します。

select count(distinct tiny_column, int_column), count(distinct varchar_column) from big_table;
```

### サブクエリ

サブクエリは関連性によって非関連サブクエリと関連サブクエリに分けられます。

- 非関連サブクエリ（単純なクエリ）は外層のクエリの結果に依存しません。
- 関連サブクエリは外層のクエリの結果に依存して実行する必要があります。

#### 非関連サブクエリ

非関連サブクエリは [NOT] IN と EXISTS をサポートします。

例：

```sql
SELECT x FROM t1 WHERE x [NOT] IN (SELECT y FROM t2);

SELECT * FROM t1 WHERE (x,y) [NOT] IN (SELECT x,y FROM t2 LIMIT 2);

SELECT x FROM t1 WHERE EXISTS (SELECT y FROM t2 WHERE y = 1);
```

バージョン 3.0 から、`SELECT... FROM... WHERE... [NOT] IN` は WHERE 句で複数のフィールドを比較することをサポートしています。つまり、上記の2番目の例の `WHERE (x,y)` の使用法です。

#### 関連サブクエリ

関連サブクエリは [NOT] IN と [NOT] EXISTS をサポートします。

例：

```sql
SELECT * FROM t1 WHERE x [NOT] IN (SELECT a FROM t2 WHERE t1.y = t2.b);

SELECT * FROM t1 WHERE [NOT] EXISTS (SELECT a FROM t2 WHERE t1.y = t2.b);
```

サブクエリはスカラーサブクエリもサポートしています。非関連スカラーサブクエリ、関連スカラーサブクエリ、およびスカラーサブクエリが通常の関数のパラメータとして使用される場合があります。

例：

1. 非関連スカラーサブクエリ、述語は = です。例えば、最高給与の人の情報を出力します。
    ```sql
    SELECT name FROM table WHERE salary = (SELECT MAX(salary) FROM table);
    ```

2. 非相関スカラーサブクエリ、述語は >,< などです。例えば、平均給与より高い給与をもらっている人の情報を出力します。

    ```sql
    SELECT name FROM table WHERE salary > (SELECT AVG(salary) FROM table);
    ```

3. 相関スカラーサブクエリ。例えば、各部門で最も高い給与をもらっている人の情報を出力します。

    ```sql
    SELECT name FROM table a WHERE salary = (SELECT MAX(salary) FROM table b WHERE b.department = a.department);
    ```

4. スカラーサブクエリを通常の関数の引数として使用します。

    ```sql
    SELECT name FROM table WHERE salary = ABS((SELECT MAX(salary) FROM table));
    ```

### WHERE と演算子

SQL の演算子は比較のための一連の関数で、これらの演算子は広く SELECT 文の WHERE 句で使用されます。

#### 算術演算子

算術演算子は通常、左オペランド、演算子、右オペランド（ほとんどの場合）を含む式に現れます。

**+ と -**：それぞれ加算と減算を表し、単項または二項演算子として機能します。単項演算子として使用される場合、例えば +1, -2.5、または -col_name では、値に +1 または -1 を乗じたものとして解釈されます。

したがって、単項演算子 + は変更されていない値を返し、単項演算子 - は値の符号を変更します。

ユーザーは二つの単項演算子を重ねて使用することができます。例えば ++5（正の値を返す）、-+2 または +-2（これらは負の値を返す）ですが、二つの - を連続して使用することはできません。

なぜなら -- はコメントとして解釈されるからです（ユーザーは二つの - を使用することができますが、その場合は二つの - の間にスペースまたは括弧を入れる必要があります。例えば -(-2) または - -2 といった形です。これらの表現は実際には +2 を意味します）。

二項演算子としての + または -、例えば 2+2、3-1.5、または col1 + col2 は、左の値に右の値を加算または減算することを意味します。左の値と右の値はどちらも数値型でなければなりません。

**\* と /**：それぞれ乗算と除算の演算子を表します。両側のオペランドは数値型でなければなりません。

二つの数が乗算される場合、より小さい型のオペランドは必要に応じてより大きい型に昇格されることがあります（例えば SMALLINT が INT や BIGINT に昇格されるなど）。式の結果は次に大きい型に昇格されます。

例えば、TINYINT と INT の乗算の結果は BIGINT 型になります。二つの数が乗算される場合、精度の損失を避けるために、オペランドと式の結果は DOUBLE 型として解釈されます。

式の結果を他の型に変換したい場合は、CAST 関数を使用する必要があります。

**%**：モジュロ演算子。左のオペランドを右のオペランドで割った余りを返します。左のオペランドと右のオペランドはどちらも整数型でなければなりません。

**&、| および ^**：ビット単位の演算子は、二つのオペランドに対するビット単位の AND、OR、XOR 操作の結果を返します。二つのオペランドはどちらも整数型でなければなりません。

ビット単位の演算子の二つのオペランドの型が一致しない場合、より小さい型のオペランドはより大きい型に昇格され、その後に対応するビット単位の操作が行われます。

一つの式に複数の算術演算子が現れることがあります。ユーザーは括弧を使用して関連する算術式を囲むことができます。算術演算子には通常、同じ機能を表す数学関数がありません。

例えば、MOD() 関数は % 演算子の機能を表すものはありません。逆に、数学関数には対応する算術演算子がありません。例えば、POW() 関数には対応する ** 指数演算子がありません。

#### BETWEEN 演算子

WHERE 句では、式が同時に上限と下限と比較されることがあります。式が下限以上で、かつ上限以下の場合、比較の結果は true です。文法は以下の通りです：

```sql
expression BETWEEN lower_bound AND upper_bound
```

データ型：通常、式（expression）の結果は数値型ですが、この演算子は他のデータ型もサポートしています。下限と上限が比較可能な文字列であることを保証する必要がある場合は、cast() 関数を使用できます。

使用説明：操作数が string 型の場合は注意が必要です。上限の長い文字列の開始部分は上限にマッチしないため、その文字列は上限より大きくなります。例えば、'MJ' は 'M' より大きいため、`between 'A' and 'M'` では 'MJ' にマッチしません。

式が正常に機能することを保証するためには、upper(), lower(), substr(), trim() などの関数を使用できます。

例：

```sql
SELECT c1 FROM t1 WHERE month BETWEEN 1 AND 6;
```

#### 比較演算子

比較演算子は、列と列が等しいか、または列を並べ替えるために使用されます。`=`, `!=`, `<=`, `>=` はすべてのデータ型に適用できます。

`<>` は不等号を意味し、`!=` と同じ機能を持っています。IN と BETWEEN 演算子は、等しい、小さい、大きいなどの関係を表す比較をより簡潔に記述するために提供されています。

#### IN 演算子

IN 演算子は VALUE 集合と比較され、その集合の任意の要素と一致する場合は TRUE を返します。

引数と VALUE 集合は比較可能でなければなりません。IN 演算子を使用するすべての式は、OR を使用して接続された等価比較として記述することができますが、IN の構文はよりシンプルで、より正確で、StarRocks による最適化が容易です。

例：

```sql
SELECT * FROM small_table WHERE tiny_column IN (1,2);
```

#### LIKE 演算子

この演算子は文字列との比較に使用されます。"_" は単一の文字にマッチし、"%" は複数の文字にマッチします。引数は完全な文字列にマッチする必要があります。通常、実際の使用法に合わせて "%" を文字列の末尾に置く方が適切です。

例：

```sql
SELECT varchar_column FROM small_table WHERE varchar_column LIKE 'm%';

+----------------+
| varchar_column |
+----------------+
|     milan      |
+----------------+

1 row in set (0.02 sec)
```

```sql
SELECT varchar_column FROM small_table WHERE varchar_column LIKE 'm____';

+----------------+
| varchar_column | 
+----------------+
|    milan       | 
+----------------+

1 row in set (0.01 sec)
```

#### 論理演算子

論理演算子は BOOL 値を返し、単項および多項演算子を含み、各演算子は BOOL 値を返す式を処理します。サポートされている演算子は以下の通りです：

AND: 二項演算子で、左側と右側の引数の計算結果が TRUE の場合、AND 演算子は TRUE を返します。

OR: 二項演算子で、左側と右側の引数の計算結果のいずれかが TRUE の場合、OR 演算子は TRUE を返します。両方の引数が FALSE の場合、OR 演算子は FALSE を返します。

NOT: 単項演算子で、式の結果を反転します。引数が TRUE の場合、この演算子は FALSE を返し、引数が FALSE の場合は TRUE を返します。

例：

```sql
SELECT TRUE AND TRUE;

+-------------------+
| (TRUE) AND (TRUE) | 
+-------------------+
|         1         | 
+-------------------+

1 row in set (0.00 sec)
```

```sql
SELECT TRUE AND FALSE;

+--------------------+
| (TRUE) AND (FALSE) | 
+--------------------+
|         0          | 
+--------------------+

1 row in set (0.01 sec)
```

```sql
SELECT TRUE OR FALSE;

+-------------------+
| (TRUE) OR (FALSE) | 
+-------------------+
|        1          | 
+-------------------+

1 row in set (0.01 sec)
```

```sql
SELECT NOT TRUE;

+----------+
| NOT TRUE | 
+----------+
|     0    | 
+----------+

1 row in set (0.01 sec)
```

#### 正規表現演算子

POSIX 標準の正規表現を使用して、文字列が正規表現にマッチするかどうかを判断します。

"^" は文字列の先頭にマッチし、"$" は文字列の末尾にマッチし、"." は任意の単一文字にマッチし、"*" は 0 個以上の任意の文字にマッチし、"+" は 1 個以上の任意の文字にマッチし、"?" は非貪欲マッチを表します。正規表現は完全な値にマッチする必要があり、文字列の一部にのみマッチするわけではありません。

中間部分にマッチさせたい場合、正規表現の前部分は "^.*" または ".*" と書くことができます。"^" と "$" は通常省略可能です。RLIKE 演算子と REGEXP 演算子は同義語です。

"|" はオプションの演算子で、"|" の両側の正規表現はどちらか一方の条件を満たせばよく、"|" と両側の正規表現は通常括弧で囲む必要があります。

例：

```sql
SELECT varchar_column FROM small_table WHERE varchar_column REGEXP '(mi|MI).*';

+----------------+
| varchar_column | 
+----------------+
|     milan      |       
+----------------+

1 row in set (0.01 sec)
```

```sql
mysql> select varchar_column from small_table where varchar_column regexp 'm.*';

+----------------+
| varchar_column | 
+----------------+
|     milan      |  
+----------------+

1行がセットされました (0.01秒)
```

### エイリアス (alias)

クエリ内でテーブル名、カラム名、またはカラムを含む式の名前を記述する際には、ASを使用してそれらにエイリアスを割り当てることができます。

テーブル名やカラム名が必要な場合、エイリアスを使用してアクセスすることができます。エイリアスは通常、元の名前よりも短く、覚えやすいものです。新しいエイリアスを作成するには、selectリストまたはfromリストにあるテーブル、カラム、式の名前の後にASエイリアス句を追加するだけです。ASキーワードはオプションで、ユーザーは元の名前の後に直接エイリアスを指定することができます。エイリアスまたは他の識別子が[StarRocksの予約キーワード](../keywords.md)と同名の場合は、その名前にバッククォートを付ける必要があります。例えば、`rank`のように。**エイリアスは大文字と小文字を区別します**。

例：

```sql
select tiny_column as name, int_column as sex from big_table;

select sum(tiny_column) as total_count from big_table;

select one.tiny_column, two.int_column from small_table one, big_table two where one.tiny_column = two.tiny_column;
```

