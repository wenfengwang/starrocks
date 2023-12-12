---
displayed_sidebar: "Japanese"
---

# SELECT（選択）

## 説明

1つまたは複数のテーブル、ビュー、またはマテリアライズド・ビューからデータをクエリします。SELECT 文は一般的に次の節で構成されます。

- [WITH](#with)
- [WHERE および演算子](#where-and-operators)
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
- [別名](#alias)

SELECT 文は独立した文として機能することも、他の文にネストされた節として機能することもできます。SELECT 節の出力は他の文の入力として使用できます。

StarRocks のクエリ文は基本的に SQL92 標準に準拠しています。以下にサポートされるSELECTの使用方法について簡単に説明します。

> **注記**
>
> StarRocks の内蔵テーブルでテーブル、ビュー、またはマテリアライズド・ビューからデータをクエリするには、これらのオブジェクトに対して SELECT 権限が必要です。外部データソースでのテーブル、ビュー、またはマテリアライズド・ビューからデータをクエリするには、対応する外部カタログに対して USAGE 権限が必要です。

### WITH（ウィズ）

SELECT 文の前に追加できる節で、SELECT 内で複数回参照される複雑な式のエイリアスを定義します。

CRATE VIEW と同様ですが、節で定義されたテーブル名と列名はクエリの終了後には永続化されず、実際のテーブルや VIEW の名前と競合しません。

WITH 節を使用する利点は次のとおりです：

- クエリ内の重複を減らし、便利でメンテナンスしやすくします。
- クエリの最も複雑な部分を別々のブロックに抽象化することで、SQL コードを読み取りやすく理解しやすくします。

例：

```sql
-- 外側の段階で1つのサブクエリを定義し、UNION ALL クエリの初期段階としてもう1つのサブクエリを定義します。

with t1 as (select 1),t2 as (select 2)
select * from t1 union all select * from t2;
```

### 結合

結合操作は、2つ以上のテーブルからデータを結合して、その中からいくつかの列の結果セットを返します。

StarRocks は、セルフ結合、クロス結合、インナー結合、アウター結合、セミ結合、アンチ結合をサポートしています。アウター結合には、左結合、右結合、フル結合が含まれます。

構文：

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

StarRocks はセルフ結合をサポートしており、これは自己結合とセルフジョインです。たとえば、同じテーブルの異なる列が結合されます。

実際にセルフ結合を識別する特別な構文はありません。セルフ結合の結合条件の両側の条件は同じテーブルから取得されます。

これらに異なるエイリアスを割り当てる必要があります。

例：

```sql
SELECT lhs.id, rhs.parent, lhs.c1, rhs.c2 FROM tree_data lhs, tree_data rhs WHERE lhs.id = rhs.parent;
```

#### クロス結合

クロス結合は多くの結果を生成できるため、慎重に使用する必要があります。

クロス結合を使用する必要がある場合でも、フィルター条件を使用し、返される結果が少なくなるようにする必要があります。例：

```sql
SELECT * FROM t1, t2;

SELECT * FROM t1 CROSS JOIN t2;
```

#### インナー結合

インナー結合は最もよく知られており、一般的に使用される結合です。2つの類似したテーブルからリクエストされた列の結果を返し、両方のテーブルの列に同じ値が含まれている場合に結合されます。

両方のテーブルの列名が同じ場合は、完全名（`テーブル名.列名` の形式）を使用するか、列名にエイリアスを付ける必要があります。

例：

次の3つのクエリは等価です。

```sql
SELECT t1.id, c1, c2 FROM t1, t2 WHERE t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 JOIN t2 ON t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id = t2.id;
```

#### アウター結合

アウター結合は、左のテーブルまたは右のテーブル、または両方のすべての行を返します。他のテーブルに一致するデータがない場合は、それを NULL に設定します。例：

```sql
SELECT * FROM t1 LEFT OUTER JOIN t2 ON t1.id = t2.id;

SELECT * FROM t1 RIGHT OUTER JOIN t2 ON t1.id = t2.id;

SELECT * FROM t1 FULL OUTER JOIN t2 ON t1.id = t2.id;
```

#### 等価および不等価結合

通常、ユーザーは最も等しい結合を使用し、結合条件の演算子は等号を必要とします。

不等結合は、結合条件が `!=`、等号以外の演算子であることを使用できます。不等結合は多くの結果を生成し、計算中にメモリ制限を超える可能性があります。

慎重に使用してください。不等結合はインナー結合のみをサポートします。例：

```sql
SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id > t2.id;
```

#### セミ結合

左セミ結合は、右テーブルで一致する行数に関係なく、左テーブルの行のみを返します。

左テーブルのこの行は、最大で1回返されます。右セミ結合は同様に機能しますが、返されるデータは右テーブルになります。

例：

```sql
SELECT t1.c1, t1.c2, t1.c2 FROM t1 LEFT SEMI JOIN t2 ON t1.id = t2.id;
```

#### アンチ結合

左アンチ結合では、右テーブルと一致しない左テーブルの行のみが返されます。

右アンチ結合では、この比較を逆にして、左テーブルと一致しない右テーブルの行のみが返されます。例：

```sql
SELECT t1.c1, t1.c2, t1.c2 FROM t1 LEFT ANTI JOIN t2 ON t1.id = t2.id;
```

#### 等結合と不等結合

StarRocks でサポートされる様々な結合は、結合条件によって等結合と不等結合に分類することができます。

| **等****結合**         | セルフ結合、クロス結合、インナー結合、アウター結合、セミ結合、アンチ結合 |
| ------------------------ | ------------------------------------------------------------ |
| **不等****結合** | クロス結合、インナー結合、左セミ結合、左アンチ結合、アウター結合   |

- 等結合
  
  等結合は、`=` 演算子で2つの結合アイテムが結合される結合条件を使用します。例：`a JOIN b ON a.id = b.id`。

- 不等結合
  
  不等結合は、`<`、`<=`、`>`、`>=`、または `<>` のような比較演算子によって2つの結合アイテムが結合される結合条件を使用します。例：`a JOIN b ON a.id < b.id`。不等結合は等結合より実行速度が遅くなります。不等結合を使用する際は注意が必要です。

  次の2つの例は、不等結合の実行方法を示しています：

  ```SQL
  SELECT t1.id, c1, c2 
  FROM t1 
  INNER JOIN t2 ON t1.id < t2.id;
    
  SELECT t1.id, c1, c2 
  FROM t1 
  LEFT JOIN t2 ON t1.id > t2.id;
  ```

### ORDER BY（並び替え）

SELECT 文の ORDER BY 節は、1つ以上の列の値を比較して結果セットをソートします。

ORDER BY は、結果をソートするため、すべての結果を一つのノードに送信してマージする必要があるため、時間とリソースを消費する操作です。ソーティングは、ORDER BY がないクエリよりも多くのメモリリソースを消費します。

したがって、ソートされた結果セットから最初の `N` 個の結果のみが必要な場合は、メモリ使用量とネットワークオーバーヘッドを減らす LIMIT 節を使用できます。LIMIT 節が指定されていない場合、デフォルトで最初の 65535 個の結果を返します。

構文：

```sql
ORDER BY <column_name>
```
    [ASC | DESC]
    [NULLS FIRST | NULLS LAST]
```

`ASC`は、結果が昇順で返されることを指定します。`DESC`は、結果が降順で返されることを指定します。順序が指定されていない場合、ASC（昇順）がデフォルトです。例：

```sql
select * from big_table order by tiny_column, short_column desc;
```

NULL値の並べ替え：`NULLS FIRST`は、NULL値を非NULL値の前に返すことを示します。`NULLS LAST`は、NULL値を非NULL値の後に返すことを示します。

例：

```sql
select  *  from  sales_record  order by  employee_id  nulls first;
```

### GROUP BY

GROUP BY句は、通常、COUNT()、SUM()、AVG()、MIN()、MAX()などの集計関数と一緒に使用されます。

GROUP BYで指定された列は、集計操作には参加しません。GROUP BY句は、HAVING句と組み合わせて、集計関数によって生成された結果をフィルタリングすることができます。

例：

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

  `groupSet`は、SELECTリスト内の列、エイリアス、または式で構成されるセットを表します。 `groupSet ::= { ( expr  [ , expr [ , ... ] ] )}`

  `expr` は、SELECTリスト内の列、エイリアス、または式を示します。

#### 注意

StarRocksはPostgreSQLのような構文をサポートしています。構文の例は以下の通りです：

  ```sql
  SELECT a, b, SUM( c ) FROM tab1 GROUP BY GROUPING SETS ( (a, b), (a), (b), ( ) );
  SELECT a, b,c, SUM( d ) FROM tab1 GROUP BY ROLLUP(a,b,c)
  SELECT a, b,c, SUM( d ) FROM tab1 GROUP BY CUBE(a,b,c)
  ```

  `ROLLUP(a,b,c)`は、次の`GROUPING SETS`ステートメントと同等です：

  ```sql
  GROUPING SETS (
  (a,b,c),
  ( a, b ),
  ( a),
  ( )
  )
  ```

  `CUBE ( a, b, c )`は、次の`GROUPING SETS`ステートメントと同等です：

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

  次は実際のデータの例です：

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

GROUP BY `GROUPING SETS` ｜ `CUBE` ｜ `ROLLUP`は、GROUP BY句の拡張です。GROUP BY句で複数のセットのグループの集計を実現することができます。結果は、複数の対応するGROUP BY句のUNION演算と等価です。

GROUP BY句は、GROUP BY GROUPING SETSを含む特殊な場合です。例えば、GROUPING SETSステートメント：

  ```sql
  SELECT a, b, SUM( c ) FROM tab1 GROUP BY GROUPING SETS ( (a, b), (a), (b), ( ) );
  ```

  クエリの結果は、次のと同等です：

  ```sql
  SELECT a, b, SUM( c ) FROM tab1 GROUP BY a, b
  UNION
  SELECT a, null, SUM( c ) FROM tab1 GROUP BY a
  UNION
  SELECT null, b, SUM( c ) FROM tab1 GROUP BY b
  UNION
  SELECT null, null, SUM( c ) FROM tab1
  ```

  `GROUPING(expr)`は、列が集計列かどうかを示します。集計列の場合は0、それ以外の場合は1です。

  `GROUPING_ID(expr  [ , expr [ , ... ] ])`は、GROUPINGと類似しています。GROUPING_IDは、指定された列の順序に従ってビットマップ値を計算し、各ビットはGROUPINGの値です。
  
  GROUPING_ID()関数は、ビットベクトルの10進値を返します。

### HAVING

HAVING句は、テーブル内の行データをフィルタリングするのではなく、集計関数の結果をフィルタリングします。

一般的には、HAVING句は、COUNT()、SUM()、AVG()、MIN()、MAX()などの集計関数とGROUP BY句と一緒に使用されます。

例：

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

LIMIT句は、返される行の最大数を制限するために使用されます。返される行の最大数を設定することで、StarRocksがメモリ使用量を最適化するのに役立ちます。

この句は、主に以下のシナリオで使用されます：

トップ-Nクエリの結果を返します。

以下のテーブル内に含まれるものを考えてみてください。

テーブル内のデータが多すぎるか、where句があまり多くのデータをフィルタリングしない場合、クエリ結果セットのサイズを制限する必要がある場合。

使用上の注意：LIMIT句の値は、数値リテラル定数でなければなりません。

例：

```plain text
mysql> select tiny_column from small_table limit 1;

+-------------+
|tiny_column  |
+-------------+
```plaintext
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

OFFSET句は、結果セットが最初の数行をスキップして、直接その後の結果を返すようにします。

結果セットはデフォルトで0行目から開始するため、オフセット0とオフセットなしで同じ結果を返します。

一般的に、OFFSET句はORDER BY句とLIMIT句と一緒に使用する必要があります。

例：

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

注意：オフセットを使用せずに順序付ける構文を使用することができますが、この場合、オフセットに意味はありません。

この場合、limitの値のみが取得され、オフセットの値は無視されます。 そのため、order byがない。

オフセットは結果セットの最大行数を超えても結果となります。 ユーザーはorder byと一緒にoffsetを使用することをお勧めします。

### UNION

複数のクエリの結果を結合します。

**構文:**

```sql
query_1 UNION [DISTINCT | ALL] query_2
```

- DISTINCT（デフォルト）はユニークな行のみを返します。 UNIONはUNION DISTINCTと同等です。
- ALLは重複を含むすべての行を結合します。デデュープはメモリを消費するため、UNION ALLを使用したクエリは高速でメモリをより少なく消費します。パフォーマンスを向上させるためには、UNION ALLを使用してください。

> **注記**
>
> 各クエリ文は同じ数の列を返し、列は互換性のあるデータ型でなければなりません。

**例:**

`select1`と`select2`のテーブルを作成します。

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

例1: 重複を含む2つのテーブルのすべてのIDを返します。

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

例2: 2つのテーブルのユニークなIDをすべて返します。以下の2つの文は同等です。

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

例3: 2つのテーブルのすべての一意なIDの最初の3つを返します。以下の2つの文は同等です。

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

複数のクエリ結果の積集合、つまりすべての結果セットで表示される結果を計算します。この句は結果セットの中でのユニークな行のみを返します。ALLキーワードはサポートされていません。

**構文:**

```SQL
query_1 INTERSECT [DISTINCT] query_2
```

> **注記**
>
> - INTERSECTはINTERSECT DISTINCTと同等です。
> - 各クエリ文は同じ数の列を返し、列は互換性のあるデータ型でなければなりません。

**例:**

UNIONの2つのテーブルが使用されます。

両方のテーブルで共通する`(id, price)`の一意な組合せを返します。以下の2つの文は同等です。

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

左側のクエリに存在しない右側のクエリの一意の結果を返します。EXCEPTはMINUSと同等です。

**構文:**

```SQL
query_1 {EXCEPT | MINUS} [DISTINCT] query_2
```

> **注記**
>
> - EXCEPTはEXCEPT DISTINCTと同等です。 ALLキーワードはサポートされていません。
> - 各クエリ文は同じ数の列を返し、列は互換性のあるデータ型でなければなりません。

**例:**

UNIONの2つのテーブルが使用されます。

`select1`の中に`select2`で見つからない一意な`(id, price)`の組合せを返します。

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

DISTINCTキーワードは、結果セットの重複を除去します。例：

```SQL
-- 1つの列から一意な値を返します。
select distinct tiny_column from big_table limit 2;

-- 複数の列から一意な値の組合せを返します。
select distinct tiny_column, int_column from big_table limit 2;
```

DISTINCTは集約関数（通常はカウント関数）と一緒に使用でき、count (distinct)は1つまたは複数の列に含まれる異なる組合せの数を計算するために使用されます。

```SQL
-- 1つの列から一意な値の数を数えます。
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
-- 複数の列から一意な値の組合せの数を数えます。
select count(distinct tiny_column, int_column) from big_table limit 2;
```

StarRocksは一度に複数の集計関数でdistinctを使うことをサポートします。

```SQL
-- 複数の集計関数を分けて一意な値を数えます。
select count(distinct tiny_column, int_column), count(distinct varchar_column) from big_table;
```

### サブクエリ

関連性に応じて、サブクエリは2つのタイプに分類されます。

- 非相関サブクエリは、外部問い合わせとは独立してその結果を取得します。
- 相関サブクエリは、外部問い合わせから値を必要とします。
#### 非相関サブクエリ

非相関サブクエリは[NOT] INとEXISTSをサポートしています。

例:

```sql
SELECT x FROM t1 WHERE x [NOT] IN (SELECT y FROM t2);

SELECT * FROM t1 WHERE (x,y) [NOT] IN (SELECT x,y FROM t2 LIMIT 2);

SELECT x FROM t1 WHERE EXISTS (SELECT y FROM t2 WHERE y = 1);
```

v3.0以降、`SELECT... FROM... WHERE... [NOT] IN`のWHERE句で複数のフィールドを指定できます。たとえば、2番目のSELECT文のWHERE句に`(x,y)`を指定しています。

#### 相関サブクエリ

関連するサブクエリは[NOT] INと[NOT] EXISTSをサポートしています。

例:

```sql
SELECT * FROM t1 WHERE x [NOT] IN (SELECT a FROM t2 WHERE t1.y = t2.b);

SELECT * FROM t1 WHERE [NOT] EXISTS (SELECT a FROM t2 WHERE t1.y = t2.b);
```

サブクエリはスカラー量子クエリもサポートしています。それは無関係なスカラー量子クエリ、関連するスカラー量子クエリ、および一般関数のパラメーターとしてのスカラー量子クエリに分類できます。

例:

1. 述語=記号を使用した非相関スカラー量子クエリ。たとえば、最高賃金をもらっている人の情報を出力します。

    ```sql
    SELECT name FROM table WHERE salary = (SELECT MAX(salary) FROM table);
    ```

2. 要素`>, <`などを持つ非相関スカラー量子クエリ。たとえば、平均以上の給料をもらっている人の情報を出力します。

    ```sql
    SELECT name FROM table WHERE salary > (SELECT AVG(salary) FROM table);
    ```

3. 関連するスカラー量子クエリ。たとえば、各部署の最高給料情報を出力します。

    ```sql
    SELECT name FROM table a WHERE salary = (SELECT MAX(salary) FROM table b WHERE b.Department= a.Department);
    ```

4. スカラー量子クエリが一般関数のパラメーターとして使用されます。

    ```sql
    SELECT name FROM table WHERE salary = abs((SELECT MAX(salary) FROM table));
    ```

### WHEREと演算子

SQL演算子は比較に使用される関数のシリーズであり、SELECT文のWHERE句で広く使用されています。

#### 算術演算子

算術演算子は通常、左、右、および最も頻繁に左のオペランドを含む式に現れます。

**+および-**: 単位としてまたは2項演算子として使用できます。単位演算子として使用される場合、たとえば、+1、-2.5、または-col_ nameなどは、値が+1または-1で乗算されることを意味します。

したがって、単位演算子+は変化しない値を返し、単位演算子-はその値の記号ビットを変更します。

ユーザーは+5（正の値を返す）、-+2または+2（負の値を返す）などの2つの単位演算子を重ねることができますが、ユーザーは2つの連続した-記号を使用することはできません。

なぜなら--は以下の文でコメントとして解釈されるからです（ユーザーが2つの-記号を使用できるようにするには、2つの-記号の間にスペースまたは括弧が必要です。たとえば、-(-2)または- -2としなければならず、その結果は実際に+ 2になります）。

+または-が2項演算子の場合、たとえば、2+2、3+1.5、またはcol1+col2は、左の値が右の値から加算または減算されることを意味します。左と右の値はどちらも数値型でなければなりません。

**and/**: それぞれ乗算および除算を表します。両側のオペランドはデータ型でなければなりません。2つの数値が乗算される場合、必要に応じてより小さいオペランドは昇格される場合があります（たとえば、SMALLINTからINTまたはBIGINT）。および式の結果は次の大きな型に昇格されます。

たとえば、TINYINTにINTを乗じるとBIGINT型の結果が得られます。2つの数値が乗算される場合、両方のオペランドおよび式の結果は精度の損失を避けるためにDOUBLE型として解釈されます。

ユーザーが式の結果を別の型に変換したい場合、CAST関数を使用して変換する必要があります。

**%**: モジュラ演算子。左のオペランドを右のオペランドで除した余りを返します。左右のオペランドはどちらも整数でなければなりません。

**&、|および^**: ビット演算子は2つのオペランドのビット単位のAND、ビット単位のOR、ビット単位のXOR演算の結果を返します。両方のオペランドには整数型が必要です。

ビット演算子の2つのオペランドの型が一貫していない場合、より小さいタイプのオペランドがより大きいタイプのオペランドに昇格され、対応するビット演算が実行されます。

多くの算術演算子が式に現れることができ、ユーザーは対応する算術式を括弧で括ることができます。算術演算子は通常、算術演算子としての同じ機能を表現するための対応する数学関数を持っていません。

たとえば、%演算子を表すMOD()関数はありません。逆に、数学関数には対応する算術演算子がありません。たとえば、累乗関数POW()には対応する**累乗演算子がありません。

ユーザーは、数学関数がサポートする算術関数をMathematical Functionsセクションで確認できます。

#### Between演算子

WHERE句では、式は上限および下限と比較されるかもしれません。式が下限以上でありかつ上限以下であれば、比較の結果は真になります。

構文:

```sql
expression BETWEEN lower_bound AND upper_bound
```

データタイプ: 通常、式は数値型に評価されますが、他のデータ型もサポートされています。下限と上限の両方が比較可能な文字列であることを保証する必要がある場合、cast()関数を使用できます。

使用上の注意: オペランドが文字列型である場合、上限から始まる長い文字列は上限以上のものは上限と一致せず、上限を超えています。 たとえば、"between'A'and'M' は'MJ'とは一致しません。

式が正しく機能することを確認する必要がある場合、upper()、lower()、substr()、trim()などの関数を使用できます。

例:

```sql
select c1 from t1 where month between 1 and 6;
```

#### 比較演算子

比較演算子は2つの値を比較するために使用されます。`=`, `!=`, `>=`はすべてのデータ型に適用されます。

`<>`および`!=`演算子は等価であり、2つの値が等しくないことを示します。

#### In演算子

In演算子はVALUEコレクションに比較し、コレクション内の要素と一致する場合はTRUEを返します。

パラメータとVALUEコレクションは比較可能なものでなければなりません。IN演算子を使用するすべての式は、等価な比較とORで接続されたものとして記述できますが、INの構文はより簡潔で、精度が高く、StarRocksが最適化しやすいです。

例:

```sql
select * from small_table where tiny_column in (1,2);
```

#### Like演算子

この演算子は文字列との比較に使用されます。''は1文字に一致し、'%'は複数の文字に一致します。パラメータは完全に文字列に一致しなければなりません。通常、文字列の末尾に'%'を置くことがより実用的です。

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

論理演算子はBOOL値を返します。ユニットおよび複数の演算子が含まれ、それぞれがBOOL値を返す式を扱います。サポートされている演算子は以下のとおりです:

AND: 2項演算子で、AND演算子は左右のパラメータがともにTRUEとして計算された場合にTRUEを返します。

OR: 2項演算子で、OR演算子は左右のパラメータのいずれかがTRUEとして計算された場合にTRUEを返します。両方のパラメータがFALSEの場合、OR演算子はFALSEを返します。

NOT: ユニット演算子は式の反転結果を返します。パラメータがTRUEの場合、演算子はFALSEを返し、パラメータがFALSEの場合、演算子はTRUEを返します。

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

正規表現が一致するかどうかを判定します。POSIX標準正規表現を使用し、'^'は文字列の先頭部分に一致し、'$'は文字列の末尾に一致します。

"."は任意の1文字に一致し、"*"は0個以上のオプションに一致し、"+"は1個以上のオプションに一致し、「?」は貪欲表現を意味し、その他も同様です。正規表現は完全な値に一致する必要があり、部分的な文字列ではなくなります。
中央部を一致させたい場合、正規表現の前部分を'^. 'または'.'として書くことができます。'^'および'$'は通常省略されます。RLIKE演算子とREGEXP演算子は同義です。

'|'演算子はオプションの演算子です。'|'の両側にある正規表現は一方の条件を満たすだけでよく、通常は両側の'|'演算子と正規表現を()で囲む必要があります。

例：

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

### エイリアス

クエリ内でテーブル、列、または列を含む式の名前を記述する際、それらにエイリアスを割り当てることができます。エイリアスは通常、元の名前よりも短く、覚えやすいです。

エイリアスが必要な場合は、セレクトリストまたはフロムリスト内のテーブル、列、および式の名前の後にAS句を追加するだけです。ASキーワードはオプションです。また、ASを使用せずに元の名前の直後にエイリアスを直接指定することもできます。

エイリアスまたはその他の識別子が[StarRocksキーワード](../keywords.md)と同じ名前の場合、バッククォートのペアで名前を括る必要があります。例： `rank`。

エイリアスは大文字と小文字を区別します。

例：

```sql
select tiny_column as name, int_column as sex from big_table;

select sum(tiny_column) as total_count from big_table;

select one.tiny_column, two.int_column from small_table one, <br/> big_table two where one.tiny_column = two.tiny_column;
```