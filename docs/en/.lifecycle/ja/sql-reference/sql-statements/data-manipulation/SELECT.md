---
displayed_sidebar: "Japanese"
---

# SELECT

## 説明

1つ以上のテーブル、ビュー、またはマテリアライズドビューからデータをクエリします。SELECTステートメントは通常、次の節で構成されます。

- [WITH](#with)
- [WHEREおよび演算子](#where-and-operators)
- [GROUP BY](#group-by)
- [HAVING](#having)
- [UNION](#union)
- [INTERSECT](#intersect)
- [EXCEPT/MINUS](#exceptminus)
- [ORDER BY](#order-by)
- [LIMIT](#limit)
- [OFFSET](#offset)
- [結合](#join)
- [サブクエリ](#subquery)
- [DISTINCT](#distinct)
- [エイリアス](#alias)

SELECTは独立したステートメントとして動作することも、他のステートメントにネストされた節として動作することもあります。SELECT節の出力は他のステートメントの入力として使用できます。

StarRocksのクエリステートメントは基本的にSQL92の標準に準拠しています。以下に、サポートされているSELECTの使用方法の概要を示します。

> **注意**
>
> StarRocksの内部テーブルからテーブル、ビュー、またはマテリアライズドビューのデータをクエリするには、これらのオブジェクトに対してSELECT権限を持っている必要があります。外部データソースのテーブル、ビュー、またはマテリアライズドビューからデータをクエリするには、対応する外部カタログに対してUSAGE権限を持っている必要があります。

### WITH

SELECTステートメントの前に追加できる節で、SELECT内で複数回参照される複雑な式のエイリアスを定義します。

CRATE VIEWと似ていますが、節で定義されたテーブル名と列名はクエリの終了後には継続されず、実際のテーブルまたはビューの名前と競合しません。

WITH節を使用する利点は次のとおりです。

- クエリ内の重複を減らし、便利でメンテナンスしやすくします。
- クエリの最も複雑な部分を別のブロックに抽象化することで、SQLコードを読みやすく理解しやすくします。

例:

```sql
-- 外部レベルで1つのサブクエリを定義し、UNION ALLクエリの初期ステージの一部として内部レベルでもう1つのサブクエリを定義します。

with t1 as (select 1),t2 as (select 2)
select * from t1 union all select * from t2;
```

### 結合

結合操作は、2つ以上のテーブルからデータを結合し、それらの一部の列から結果セットを返します。

StarRocksは、セルフ結合、クロス結合、インナー結合、アウター結合、セミ結合、アンチ結合をサポートしています。アウター結合には、左結合、右結合、フル結合が含まれます。

構文:

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

#### セルフ結合

StarRocksはセルフ結合をサポートしており、セルフ結合とセルフ結合の両方をサポートしています。たとえば、同じテーブルの異なる列を結合します。

セルフ結合を識別するための特別な構文は実際にはありません。セルフ結合の結合の両側の条件は同じテーブルから来ます。

それらに異なるエイリアスを割り当てる必要があります。

例:

```sql
SELECT lhs.id, rhs.parent, lhs.c1, rhs.c2 FROM tree_data lhs, tree_data rhs WHERE lhs.id = rhs.parent;
```

#### クロス結合

クロス結合は多くの結果を生成するため、クロス結合は注意して使用する必要があります。

クロス結合を使用する必要がある場合でも、フィルタ条件を使用して返される結果が少なくなるようにする必要があります。例:

```sql
SELECT * FROM t1, t2;

SELECT * FROM t1 CROSS JOIN t2;
```

#### インナー結合

インナー結合は最もよく知られていて一般的に使用される結合です。2つの類似したテーブルからリクエストされた列の結果を返し、両方のテーブルの列に同じ値が含まれている場合に結合します。

両方のテーブルの列名が同じ場合は、完全な名前（テーブル名.列名の形式）を使用するか、列名にエイリアスを付ける必要があります。

例:

次の3つのクエリは同等です。

```sql
SELECT t1.id, c1, c2 FROM t1, t2 WHERE t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 JOIN t2 ON t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id = t2.id;
```

#### アウター結合

アウター結合は、左または右のテーブルまたは両方のすべての行を返します。他のテーブルに一致するデータがない場合は、NULLに設定します。例:

```sql
SELECT * FROM t1 LEFT OUTER JOIN t2 ON t1.id = t2.id;

SELECT * FROM t1 RIGHT OUTER JOIN t2 ON t1.id = t2.id;

SELECT * FROM t1 FULL OUTER JOIN t2 ON t1.id = t2.id;
```

#### 等価および非等価結合

通常、ユーザーは最も等しい結合を使用し、結合条件の演算子が等号であることを要求します。

非等価結合は、結合条件!=、等号を使用します。非等価結合は多くの結果を生成し、計算中にメモリ制限を超える可能性があります。

注意して使用してください。非等価結合は内部結合のみをサポートしています。例:

```sql
SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id > t2.id;
```

#### セミ結合

左セミ結合は、左テーブルの行のみを返します。右テーブルのデータと一致する行がいくつあっても、左テーブルのこの行は最大1回返されます。右セミ結合は同様に動作しますが、返されるデータは右テーブルです。

例:

```sql
SELECT t1.c1, t1.c2, t1.c2 FROM t1 LEFT SEMI JOIN t2 ON t1.id = t2.id;
```

#### アンチ結合

左アンチ結合は、右テーブルと一致しない左テーブルの行のみを返します。

右アンチ結合は、これとは逆の比較を行い、左テーブルと一致しない右テーブルの行のみを返します。例:

```sql
SELECT t1.c1, t1.c2, t1.c2 FROM t1 LEFT ANTI JOIN t2 ON t1.id = t2.id;
```

#### 等結合と非等結合

StarRocksでサポートされているさまざまな結合は、結合で指定された結合条件に応じて等結合と非等結合に分類できます。

| **等結合**         | セルフ結合、クロス結合、インナー結合、アウター結合、セミ結合、アンチ結合 |
| -------------------------- | ------------------------------------------------------------ |
| **非等結合** | クロス結合、インナー結合、左セミ結合、左アンチ結合、アウター結合   |

- 等結合

  等結合は、結合条件に`=`演算子を使用する結合条件を使用します。例: `a JOIN b ON a.id = b.id`。

- 非等結合

  非等結合は、`<`、`<=`、`>`、`>=`、または`<>`などの比較演算子を使用して2つの結合項目を結合する結合条件を使用します。例: `a JOIN b ON a.id < b.id`。非等結合は等結合よりも遅く実行されます。非等結合を使用する場合は注意して使用することをお勧めします。

  次の2つの例は、非等結合の実行方法を示しています:

  ```SQL
  SELECT t1.id, c1, c2 
  FROM t1 
  INNER JOIN t2 ON t1.id < t2.id;
    
  SELECT t1.id, c1, c2 
  FROM t1 
  LEFT JOIN t2 ON t1.id > t2.id;
  ```

### ORDER BY

SELECTステートメントのORDER BY節は、1つまたは複数の列の値を比較して結果セットをソートします。

ORDER BYは時間とリソースを消費する操作です。ソートするためには、すべての結果を1つのノードに送信してマージする必要があります。ソートは、ORDER BYを使用しないクエリよりもメモリリソースを消費します。

したがって、ソートされた結果セットから最初のN個の結果のみが必要な場合は、メモリ使用量とネットワークオーバーヘッドを削減するためにLIMIT節を使用できます。LIMIT節が指定されていない場合、デフォルトで最初の65535個の結果が返されます。

構文:

```sql
ORDER BY <column_name> 
    [ASC | DESC]
    [NULLS FIRST | NULLS LAST]
```

`ASC`は結果を昇順で返すことを指定します。`DESC`は結果を降順で返すことを指定します。順序が指定されていない場合、ASC（昇順）がデフォルトです。例:

```sql
select * from big_table order by tiny_column, short_column desc;
```

NULL値のソート順: `NULLS FIRST`はNULL値を非NULL値の前に返すことを示します。`NULLS LAST`はNULL値を非NULL値の後に返すことを示します。

例:

```sql
select  *  from  sales_record  order by  employee_id  nulls first;
```

### GROUP BY

GROUP BY節は、COUNT()、SUM()、AVG()、MIN()、MAX()などの集計関数と一緒によく使用されます。

GROUP BYで指定された列は集計操作に参加しません。GROUP BY節は、集計関数によって生成された結果をフィルタリングするためにHAVING節と組み合わせて使用できます。

例:

```sql
select tiny_column, sum(short_column)
from small_table 
group by tiny_column;
```

```plain text
+-------------+---------------------+
| tiny_column |  sum('short_column')|
+-------------+---------------------+
|      1      |        2            |
|      2      |        1            |
+-------------+---------------------+
```

#### 構文

  ```sql
  SELECT ...
  FROM ...
  [ ... ]
  GROUP BY [
      , ... |
      GROUPING SETS [, ...] (  groupSet [ , groupSet [ , ... ] ] ) |
      ROLLUP(expr  [ , expr [ , ... ] ]) |
      expr  [ , expr [ , ... ] ] WITH ROLLUP |
      CUBE(expr  [ , expr [ , ... ] ]) |
      expr  [ , expr [ , ... ] ] WITH CUBE
      ]
  [ ... ]
  ```

#### パラメータ

  `groupSet`は、セレクトリスト内の列、エイリアス、または式から構成されるセットを表します。 `groupSet ::= { ( expr  [ , expr [ , ... ] ] )}`

  `expr`は、セレクトリスト内の列、エイリアス、または式を示します。

#### 注意

StarRocksはPostgreSQLのような構文をサポートしています。構文の例は次のとおりです。

  ```sql
  SELECT a, b, SUM( c ) FROM tab1 GROUP BY GROUPING SETS ( (a, b), (a), (b), ( ) );
  SELECT a, b,c, SUM( d ) FROM tab1 GROUP BY ROLLUP(a,b,c)
  SELECT a, b,c, SUM( d ) FROM tab1 GROUP BY CUBE(a,b,c)
  ```

  `ROLLUP(a,b,c)`は次の`GROUPING SETS`ステートメントと同等です。

  ```sql
  GROUPING SETS (
  (a,b,c),
  ( a, b ),
  ( a),
  ( )
  )
  ```

  `CUBE ( a, b, c )`は次の`GROUPING SETS`ステートメントと同等です。

  ```sql
  GROUPING SETS (
  ( a, b, c ),
  ( a, b ),
  ( a,    c ),
  ( a       ),
  (    b, c ),
  (    b    ),
  (       c ),
  (         )
  )
  ```

#### 例

  次は実際のデータの例です。

  ```plain text
  SELECT * FROM t;
  +------+------+------+
  | k1   | k2   | k3   |
  +------+------+------+
  | a    | A    |    1 |
  | a    | A    |    2 |
  | a    | B    |    1 |
  | a    | B    |    3 |
  | b    | A    |    1 |
  | b    | A    |    4 |
  | b    | B    |    1 |
  | b    | B    |    5 |
  +------+------+------+
  8 rows in set (0.01 sec)

  SELECT k1, k2, SUM(k3) FROM t GROUP BY GROUPING SETS ( (k1, k2), (k2), (k1), ( ) );
  +------+------+-----------+
  | k1   | k2   | sum(`k3`) |
  +------+------+-----------+
  | b    | B    |         6 |
  | a    | B    |         4 |
  | a    | A    |         3 |
  | b    | A    |         5 |
  | NULL | B    |        10 |
  | NULL | A    |         8 |
  | a    | NULL |         7 |
  | b    | NULL |        11 |
  | NULL | NULL |        18 |
  +------+------+-----------+
  9 rows in set (0.06 sec)

  > SELECT k1, k2, GROUPING_ID(k1,k2), SUM(k3) FROM t GROUP BY GROUPING SETS ((k1, k2), (k1), (k2), ());
  +------+------+---------------+----------------+
  | k1   | k2   | grouping_id(k1,k2) | sum(`k3`) |
  +------+------+---------------+----------------+
  | a    | A    |             0 |              3 |
  | a    | B    |             0 |              4 |
  | a    | NULL |             1 |              7 |
  | b    | A    |             0 |              5 |
  | b    | B    |             0 |              6 |
  | b    | NULL |             1 |             11 |
  | NULL | A    |             2 |              8 |
  | NULL | B    |             2 |             10 |
  | NULL | NULL |             3 |             18 |
  +------+------+---------------+----------------+
  9 rows in set (0.02 sec)
  ```

GROUP BY `GROUPING SETS` ｜ `CUBE` ｜ `ROLLUP`は、GROUP BY節の複数のセットのグループの集計を実現するための拡張です。結果は、複数の対応するGROUP BY節のUNION操作と等価です。

GROUP BY節は、GROUP BY GROUPING SETSに1つの要素しか含まれていない特別な場合です。たとえば、GROUPING SETSステートメント:

  ```sql
  SELECT a, b, SUM( c ) FROM tab1 GROUP BY GROUPING SETS ( (a, b), (a), (b), ( ) );
  ```

  クエリ結果は次のと等価です:

  ```sql
  SELECT a, b, SUM( c ) FROM tab1 GROUP BY a, b
  UNION
  SELECT a, null, SUM( c ) FROM tab1 GROUP BY a
  UNION
  SELECT null, b, SUM( c ) FROM tab1 GROUP BY b
  UNION
  SELECT null, null, SUM( c ) FROM tab1
  ```

  `GROUPING(expr)`は、列が集計列であるかどうかを示します。集計列の場合は0、それ以外の場合は1です。

  `GROUPING_ID(expr  [ , expr [ , ... ] ])`はGROUPINGに似ています。GROUPING_IDは、指定された列の順序に従って列リストのビットマップ値を計算し、各ビットはGROUPINGの値です。
  
  GROUPING_ID()関数はビットベクトルの10進数値を返します。

### HAVING

HAVING節は、テーブルの行データをフィルタリングするのではなく、集計関数の結果をフィルタリングします。

通常、HAVINGは集計関数（COUNT()、SUM()、AVG()、MIN()、MAX()など）とGROUP BY節と組み合わせて使用されます。

例:

```sql
select tiny_column, sum(short_column) 
from small_table 
group by tiny_column 
having sum(short_column) = 1;
```

```plain text
+-------------+---------------------+
|tiny_column  | sum('short_column') |
+-------------+---------------------+
|         2   |        1            |
+-------------+---------------------+

1 row in set (0.07 sec)
```

```sql
select tiny_column, sum(short_column) 
from small_table 
group by tiny_column 
having tiny_column > 1;
```

```plain text
+-------------+---------------------+
|tiny_column  | sum('short_column') |
+-------------+---------------------+
|      2      |          1          |
+-------------+---------------------+

1 row in set (0.07 sec)
```

### LIMIT

LIMIT節は、返される行の最大数を制限するために使用されます。返される行の最大数を設定することで、StarRocksはメモリ使用量を最適化することができます。

この節は主に次のシナリオで使用されます。

- 上位N件のクエリ結果を返します。
- 下のテーブルに含まれるデータが多すぎるか、WHERE節があまりにも多くのデータをフィルタリングしないため、クエリ結果セットのサイズを制限する必要がある場合。

使用方法: LIMIT節の値は数値リテラル定数である必要があります。

例:

```plain text
mysql> select tiny_column from small_table limit 1;

+-------------+
|tiny_column  |
+-------------+
|     1       |
+-------------+

1 row in set (0.02 sec)
```

```plain text
mysql> select tiny_column from small_table limit 10000;

+-------------+
|tiny_column  |
+-------------+
|      1      |
|      2      |
+-------------+

2 rows in set (0.01 sec)
```

### OFFSET

OFFSET節は、結果セットが最初のいくつかの行をスキップし、直後の結果を直接返します。

結果セットはデフォルトで行0から開始するため、offset 0とno offsetは同じ結果を返します。

一般的には、OFFSET節はORDER BYおよびLIMIT節と組み合わせて使用する必要があります。

例:

```plain text
mysql> select varchar_column from big_table order by varchar_column limit 3;

+----------------+
| varchar_column | 
+----------------+
|    beijing     | 
|    chongqing   | 
|    tianjin     | 
+----------------+

3 rows in set (0.02 sec)
```

```plain text
mysql> select varchar_column from big_table order by varchar_column limit 1 offset 0;

+----------------+
|varchar_column  |
+----------------+
|     beijing    |
+----------------+

1 row in set (0.01 sec)
```

```plain text
mysql> select varchar_column from big_table order by varchar_column limit 1 offset 1;

+----------------+
|varchar_column  |
+----------------+
|    chongqing   | 
+----------------+

1 row in set (0.01 sec)
```

```plain text
mysql> select varchar_column from big_table order by varchar_column limit 1 offset 2;

+----------------+
|varchar_column  |
+----------------+
|     tianjin    |     
+----------------+

1 row in set (0.02 sec)
```

注意: ORDER BYなしでoffset構文を使用することも許可されていますが、この場合、offsetは意味を持ちません。

この場合、limitの値のみが取られ、offsetの値は無視されます。したがって、ORDER BYなしでoffset構文を使用することはお勧めしません。

offsetが結果セットの最大行数を超えている場合でも、結果が返されます。offsetをorder byと一緒に使用することをお勧めします。

### UNION

複数のクエリの結果を結合します。

**構文:**

```sql
query_1 UNION [DISTINCT | ALL] query_2
```

- DISTINCT（デフォルト）は重複する行のみを返します。UNIONはUNION DISTINCTと同等です。
- ALLは重複を含むすべての行を結合します。重複の削除はメモリを多く消費するため、UNION ALLを使用したクエリの方が高速でメモリ消費量が少なくなります。パフォーマンスを向上させるために、UNION ALLを使用してください。

> **注意**
>
> 各クエリステートメントは同じ数の列を返し、列は互換性のあるデータ型である必要があります。

**例:**

テーブル`select1`と`select2`を作成します。

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

例1: 両方のテーブルに存在するすべてのID、重複を含む。

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
11 rows in set (0.02 sec)
```

例2: 両方のテーブルに存在する一意のIDのみを返します。次の2つのステートメントは同等です。

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
6 rows in set (0.01 sec)

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
5 rows in set (0.02 sec)
```

例3: 両方のテーブルの一意のIDのうち、最初の3つを返します。次の2つのステートメントは同等です。

```SQL
mysql> (select id from select1) union distinct (select id from select2)
order by id
limit 3;
++------+
| id   |
+------+
|    1 |
|    2 |
|    3 |
+------+
4 rows in set (0.11 sec)

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
3 rows in set (0.01 sec)
```

### **INTERSECT**

複数のクエリの結果の共通部分、つまりすべての結果セットに表示される結果を計算します。この節は結果セットの一意の行のみを返します。ALLキーワードはサポートされていません。

**構文:**

```SQL
query_1 INTERSECT [DISTINCT] query_2
```

> **注意**
>
> - INTERSECTはINTERSECT DISTINCTと同等です。
> - 各クエリステートメントは同じ数の列を返し、列は互換性のあるデータ型である必要があります。

**例:**

UNIONの2つのテーブルを使用します。

両方のテーブルで共通の`(id, price)`の組み合わせを返します。次の2つのステートメントは同等です。

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

左側のクエリの結果に存在しない右側のクエリの結果を返します。EXCEPTはMINUSと同等です。

**構文:**

```SQL
query_1 {EXCEPT | MINUS} [DISTINCT] query_2
```

> **注意**
>
> - EXCEPTはEXCEPT DISTINCTと同等です。ALLキーワードはサポートされていません。
> - 各クエリステートメントは同じ数の列を返し、列は互換性のあるデータ型である必要があります。

**例:**

UNIONの2つのテーブルを使用します。

`select1`の中で`select2`で見つからない一意の`(id, price)`の組み合わせを返します。

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

DISTINCTキーワードは、結果セットの重複を削除します。例:

```SQL
-- 1つの列から一意の値を返します。
select distinct tiny_column from big_table limit 2;

-- 複数の列から一意の値の組み合わせを返します。
select distinct tiny_column, int_column from big_table limit 2;
```

DISTINCTは集計関数（通常はカウント関数）と組み合わせて使用することもできます。count(distinct)は、1つまたは複数の列に含まれる異なる組み合わせの数を計算するために使用されます。

```SQL
-- 1つの列から一意の値の数をカウントします。
select count(distinct tiny_column) from small_table;
```

```plain text
+-------------------------------+
| count(DISTINCT 'tiny_column') |
+-------------------------------+
|             2                 |
+-------------------------------+
1 row in set (0.06 sec)
```

```SQL
-- 複数の列から一意の値の組み合わせの数をカウントします。
select count(distinct tiny_column, int_column) from big_table limit 2;
```

StarRocksは複数の集計関数を同時にdistinctを使用してサポートしています。

```SQL
-- 複数の集計関数を個別に一意の値としてカウントします。
select count(distinct tiny_column, int_column), count(distinct varchar_column) from big_table;
```

### サブクエリ

サブクエリは、関連性に基づいて2つのタイプに分類されます。

- 非関連サブクエリは、外部クエリとは独立して結果を取得します。
- 関連サブクエリは、外部クエリから値を必要とします。

#### 非関連サブクエリ

非関連サブクエリは[NOT] INおよびEXISTSをサポートしています。

例:

```sql
SELECT x FROM t1 WHERE x [NOT] IN (SELECT y FROM t2);

SELECT * FROM t1 WHERE (x,y) [NOT] IN (SELECT x,y FROM t2 LIMIT 2);

SELECT x FROM t1 WHERE EXISTS (SELECT y FROM t2 WHERE y = 1);
```

v3.0以降、`SELECT... FROM... WHERE... [NOT] IN`のWHERE節で複数のフィールドを指定できます。たとえば、2番目のSELECTステートメントの`WHERE (x,y)`です。

#### 関連サブクエリ

関連サブクエリは[NOT] INおよび[NOT] EXISTSをサポートしています。

例:

```sql
SELECT * FROM t1 WHERE x [NOT] IN (SELECT a FROM t2 WHERE t1.y = t2.b);

SELECT * FROM t1 WHERE [NOT] EXISTS (SELECT a FROM t2 WHERE t1.y = t2.b);
```

サブクエリはスカラー量子クエリもサポートしています。一般的な関数のパラメータとしての関連スカラー量子クエリ、関連スカラー量子クエリ、および一般関数のパラメータとしてのスカラー量子クエリに分類できます。

例:

1. プレディケート=符号を持つ関連しないスカラー量子クエリ。たとえば、最高賃金を受け取る人の情報を出力します。

    ```sql
    SELECT name FROM table WHERE salary = (SELECT MAX(salary) FROM table);
    ```

2. プレディケート`>`,`<`などを持つ関連しないスカラー量子クエリ。たとえば、平均以上の給与を受け取る人の情報を出力します。

    ```sql
    SELECT name FROM table WHERE salary > (SELECT AVG(salary) FROM table);
    ```

3. 関連するスカラー量子クエリ。例えば、各部門の最高給与情報を出力します。

    ```sql
    SELECT name FROM table a WHERE salary = (SELECT MAX(salary) FROM table b WHERE b.Department= a.Department);
    ```

4. スカラー量子クエリは通常の関数のパラメータとして使用されます。

    ```sql
    SELECT name FROM table WHERE salary = abs((SELECT MAX(salary) FROM table));
    ```

### WHEREと演算子

SQLの演算子は比較に使用される一連の関数であり、select文のwhere句で広く使用されます。

#### 算術演算子

算術演算子は通常、左、右、および最も頻繁に左のオペランドを含む式に表示されます。

**+および-**: ユニット演算子または2項演算子として使用できます。ユニット演算子として使用される場合、+1、-2.5、または-col_ nameなど、値が+1または-1で乗算されることを意味します。

したがって、セル演算子+は変更されていない値を返し、セル演算子-はその値の符号ビットを変更します。

ユーザーは+5（正の値を返す）、-+2または+2（負の値を返す）など、2つのセル演算子を重ねることができますが、2つの連続した-記号を使用することはできません。

なぜなら--は次の文でコメントとして解釈されるからです（ユーザーが2つの記号を使用できる場合、2つの記号の間にスペースまたは括弧が必要です。たとえば、-（-2）または- -2は実際には+ 2の結果をもたらします）。

+または-が2項演算子（2+2、3+1.5、またはcol1+col2など）の場合、左の値が右の値に加算または減算されることを意味します。左右の値はいずれも数値型でなければなりません。

**および/**: それぞれ乗算と除算を表します。両側のオペランドはデータ型である必要があります。2つの数値が乗算される場合、必要に応じてより小さいオペランドが昇格する場合があります（たとえば、SMALLINTからINTまたはBIGINTへ）。

および式の結果は、次の大きな型に昇格されます。

たとえば、TINYINTをINTで乗算すると、BIGINT型の結果が生成されます。2つの数値が乗算される場合、両方のオペランドと式の結果は、精度の損失を避けるためにDOUBLE型として解釈されます。

ユーザーが式の結果を別の型に変換したい場合は、CAST関数を使用して変換する必要があります。

**%**: モジュラ演算子。左オペランドを右オペランドで割った余りを返します。左右のオペランドは整数でなければなりません。

**&、|および^**: ビット演算子は、2つのオペランドのビットごとのAND、ビットごとのOR、ビットごとのXOR演算の結果を返します。両方のオペランドは整数型である必要があります。

ビット演算子の2つのオペランドの型が一致しない場合、より小さい型のオペランドがより大きな型のオペランドに昇格し、対応するビット演算が実行されます。

複数の算術演算子が式に表示される場合、ユーザーは対応する算術式を括弧で囲むことができます。算術演算子に対応する数学関数は通常、算術演算子と同じ機能を表現するための数学関数を持っていません。

たとえば、%演算子を表すMOD（）関数はありません。逆に、数学関数には対応する算術演算子がありません。たとえば、べき乗関数POW（）には対応する**指数演算子がありません。

ユーザーは、数学関数をサポートしている算術関数を数学関数のセクションで確認することができます。

#### Between演算子

where句では、式を上限と下限の両方と比較することができます。式が下限以上であり、上限以下であれば、比較の結果はtrueです。

構文:

```sql
expression BETWEEN lower_bound AND upper_bound
```

データ型: 通常、式は数値型を評価することができますが、他のデータ型もサポートされています。下限と上限の両方が比較可能な文字列であることを確認する必要がある場合は、cast（）関数を使用できます。

使用方法: オペランドが文字列型の場合、大文字で始まる長い文字列は上限と一致せず、上限よりも大きいです。たとえば、「between'A'and'M'は'MJ'に一致しません。

式が正しく機能することを確認する必要がある場合は、upper（）、lower（）、substr（）、trim（）などの関数を使用できます。

例:

```sql
select c1 from t1 where month between 1 and 6;
```

#### 比較演算子

比較演算子は2つの値を比較するために使用されます。`=`、`！= `、`> =`はすべてのデータ型に適用されます。

`<>`演算子と`！= `演算子は同等であり、2つの値が等しくないことを示します。

#### In演算子

In演算子はVALUEコレクションと比較し、コレクションの要素のいずれかに一致する場合にTRUEを返します。

パラメータとVALUEコレクションは比較可能でなければなりません。IN演算子を使用するすべての式は、等価な比較で接続されたORとして書くことができますが、INの構文はよりシンプルで正確であり、StarRocksが最適化しやすくなっています。

例:

```sql
select * from small_table where tiny_column in (1,2);
```

#### Like演算子

この演算子は文字列と比較するために使用されます。''は1文字に一致し、'%'は複数の文字に一致します。パラメータは完全な文字列と一致する必要があります。通常、文字列の末尾に'%'を配置することがより実用的です。

例:

```plain text
mysql> select varchar_column from small_table where varchar_column like 'm%';

+----------------+
|varchar_column  |
+----------------+
|     milan      |
+----------------+

1 row in set (0.02 sec)
```

```plain
mysql> select varchar_column from small_table where varchar_column like 'm____';

+----------------+
| varchar_column | 
+----------------+
|    milan       | 
+----------------+

1 row in set (0.01 sec)
```

#### 論理演算子

論理演算子はBOOL値を返し、ユニット演算子と複数の演算子を含みます。それぞれの演算子はBOOL値を返す式を処理するパラメータを扱います。サポートされる演算子は次のとおりです。

AND: 2項演算子で、AND演算子は左側と右側のパラメータがともにTRUEとして計算された場合にTRUEを返します。

OR: 2項演算子で、左側と右側のパラメータのいずれかがTRUEとして計算された場合にTRUEを返します。両方のパラメータがFALSEの場合、OR演算子はFALSEを返します。

NOT: ユニット演算子で、式を反転した結果を返します。パラメータがTRUEの場合、演算子はFALSEを返します。パラメータがFALSEの場合、演算子はTRUEを返します。

例:

```plain text
mysql> select true and true;

+-------------------+
| (TRUE) AND (TRUE) | 
+-------------------+
|         1         | 
+-------------------+

1 row in set (0.00 sec)
```

```plain text
mysql> select true and false;

+--------------------+
| (TRUE) AND (FALSE) | 
+--------------------+
|         0          | 
+--------------------+

1 row in set (0.01 sec)
```

```plain text
mysql> select true or false;

+-------------------+
| (TRUE) OR (FALSE) | 
+-------------------+
|        1          | 
+-------------------+

1 row in set (0.01 sec)
```

```plain text
mysql> select not true;

+----------+
| NOT TRUE | 
+----------+
|     0    | 
+----------+

1 row in set (0.01 sec)
```

#### 正規表現演算子

正規表現が一致するかどうかを判断します。POSIX標準の正規表現を使用し、'^'は文字列の最初の部分に一致し、'$'は文字列の末尾に一致します。

"."は任意の1文字に一致し、"*"は0回以上のオプションに一致し、"+"は1回以上のオプションに一致し、"?"は貪欲表現を意味し、などです。正規表現は完全な値と一致する必要があり、文字列の一部だけではありません。

中間部分を一致させたい場合、正規表現の前半部分は'^.'または'.'と書くことができます。通常、'^'と'$'は省略されます。RLIKE演算子とREGEXP演算子は同義です。

'|'演算子はオプションの演算子です。'|'の両側の正規表現は、一方の条件を満たす必要があります。'|'演算子と両側の正規表現は通常、()で囲む必要があります。

例:

```plain text
mysql> select varchar_column from small_table where varchar_column regexp '(mi|MI).*';

+----------------+
| varchar_column | 
+----------------+
|     milan      |       
+----------------+

1 row in set (0.01 sec)
```

```plain text
mysql> select varchar_column from small_table where varchar_column regexp 'm.*';

+----------------+
| varchar_column | 
+----------------+
|     milan      |  
+----------------+

1 row in set (0.01 sec)
```

### 別名

クエリ内のテーブル、列、または列を含む式の名前を書く場合、それらに別名を割り当てることができます。別名は通常、元の名前よりも短く、覚えやすいです。

別名が必要な場合は、selectリストまたはfromリストのテーブル、列、および式の名前の後にAS句を追加するだけです。ASキーワードはオプションです。ASを使用せずに元の名前の直後に直接別名を指定することもできます。

別名またはその他の識別子が内部の[StarRocksキーワード](../keywords.md)と同じ名前である場合は、名前をバッククォートで囲む必要があります。たとえば、`rank`です。

別名は大文字と小文字を区別します。

例:

```sql
select tiny_column as name, int_column as sex from big_table;

select sum(tiny_column) as total_count from big_table;

select one.tiny_column, two.int_column from small_table one, <br/> big_table two where one.tiny_column = two.tiny_column;
```
