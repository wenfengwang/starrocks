---
displayed_sidebar: English
---

# SELECT

## 説明

1つ以上のテーブル、ビュー、またはマテリアライズドビューからデータをクエリします。SELECT文は、通常、以下の句で構成されます：

- [WITH](#with)
- [WHEREと演算子](#where-and-operators)
- [GROUP BY](#group-by)
- [HAVING](#having)
- [UNION](#union)
- [INTERSECT](#intersect)
- [EXCEPT/MINUS](#exceptminus)
- [ORDER BY](#order-by)
- [LIMIT](#limit)
- [OFFSET](#offset)
- [JOIN](#join)
- [サブクエリ](#subquery)
- [DISTINCT](#distinct)
- [エイリアス](#alias)

SELECTは、独立したステートメントとしても、他のステートメントにネストされた句としても機能します。SELECT句の出力は、他のステートメントの入力として使用できます。

StarRocksのクエリステートメントは、基本的にSQL92標準に準拠しています。ここでは、サポートされているSELECTの使用法について簡単に説明します。

> **注記**
>
> StarRocks内部テーブルのテーブル、ビュー、またはマテリアライズドビューからデータをクエリするには、これらのオブジェクトに対するSELECT権限が必要です。外部データソースのテーブル、ビュー、またはマテリアライズドビューからデータをクエリするには、対応する外部カタログに対するUSAGE権限が必要です。

### WITH

SELECT文の前に追加できる句で、SELECT内で複数回参照される複雑な式のエイリアスを定義します。

CREATE VIEWに似ていますが、句で定義されたテーブル名とカラム名は、クエリ終了後には持続せず、実際のテーブルやVIEWの名前と競合しません。

WITH句を使用する利点は以下の通りです：

便利でメンテナンスが容易で、クエリ内の重複を減らします。

クエリの最も複雑な部分を個別のブロックに抽象化することで、SQLコードが読みやすく、理解しやすくなります。

例：

```sql
-- Define one subquery at the outer level, and another at the inner level as part of the
-- initial stage of the UNION ALL query.

WITH t1 AS (SELECT 1), t2 AS (SELECT 2)
SELECT * FROM t1 UNION ALL SELECT * FROM t2;
```

### JOIN

結合操作は、2つ以上のテーブルからデータを組み合わせて、それらのいくつかの列からなる結果セットを返します。

StarRocksは、セルフジョイン、クロスジョイン、インナージョイン、アウタージョイン、セミジョイン、およびアンチジョインをサポートしています。アウタージョインには、レフトジョイン、ライトジョイン、フルジョインが含まれます。

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

#### セルフジョイン

StarRocksはセルフジョインをサポートしており、これは同じテーブルの異なる列を結合することです。

セルフジョインには特別な構文はありません。セルフジョインの結合条件の両側は同じテーブルから来ます。

異なるエイリアスを割り当てる必要があります。

例：

```sql
SELECT lhs.id, rhs.parent, lhs.c1, rhs.c2 FROM tree_data lhs JOIN tree_data rhs ON lhs.id = rhs.parent;
```

#### クロスジョイン

クロスジョインは多くの結果を生成する可能性があるため、注意して使用する必要があります。

クロスジョインを使用する場合は、フィルター条件を使用し、返される結果が少なくなるようにする必要があります。例：

```sql
SELECT * FROM t1, t2;

SELECT * FROM t1 CROSS JOIN t2;
```

#### インナージョイン

インナージョインは最もよく知られている結合で、一般的に使用されます。2つの類似したテーブルから要求された列の結果を返し、両方のテーブルの列に同じ値が含まれている場合に結合されます。

両方のテーブルの列名が同じ場合は、フルネーム（table_name.column_nameの形式）を使用するか、列名にエイリアスを付ける必要があります。

例：

以下の3つのクエリは同等です。

```sql
SELECT t1.id, c1, c2 FROM t1, t2 WHERE t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 JOIN t2 ON t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id = t2.id;
```

#### アウタージョイン

アウタージョインは、左または右のテーブル、または両方のすべての行を返します。もう一方のテーブルに一致するデータがない場合は、NULLに設定されます。例：

```sql
SELECT * FROM t1 LEFT OUTER JOIN t2 ON t1.id = t2.id;

SELECT * FROM t1 RIGHT OUTER JOIN t2 ON t1.id = t2.id;

SELECT * FROM t1 FULL OUTER JOIN t2 ON t1.id = t2.id;
```

#### 等価結合と非等価結合

通常、ユーザーは等価結合を使用しますが、これには結合条件の演算子が等号である必要があります。

非等価結合は、結合条件で`!=`、`<`、`>`などの等号以外の演算子を使用できます。非等価結合は多くの結果を生成し、計算中にメモリ制限を超える可能性があります。

注意して使用してください。非等価結合はインナージョインのみをサポートします。例：

```sql
SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id > t2.id;
```

#### セミジョイン

左セミジョインは、右のテーブルのデータと一致する左のテーブルの行のみを返します。右のテーブルのデータと一致する行数に関わらず、左のテーブルのこの行は最大で1回のみ返されます。右セミジョインも同様に機能しますが、返されるデータは右のテーブルです。

例：

```sql
SELECT t1.c1, t1.c2 FROM t1 LEFT SEMI JOIN t2 ON t1.id = t2.id;
```

#### アンチジョイン

左アンチジョインは、右のテーブルと一致しない左のテーブルの行のみを返します。

右アンチジョインはこの比較を逆にし、左のテーブルと一致しない右のテーブルの行のみを返します。例：

```sql
SELECT t1.c1, t1.c2 FROM t1 LEFT ANTI JOIN t2 ON t1.id = t2.id;
```

#### 等結合と非等結合

StarRocksがサポートする様々な結合は、結合条件に応じて等結合と非等結合に分類されます。

| **等結合**             | セルフジョイン、クロスジョイン、インナージョイン、アウタージョイン、セミジョイン、アンチジョイン |
| -------------------------- | ------------------------------------------------------------ |
| **非等結合** | クロスジョイン、インナージョイン、左セミジョイン、左アンチジョイン、アウタージョイン   |

- 等結合
  
  等結合では、`=` 演算子で結合される2つの結合項目の結合条件が使用されます。例：`a JOIN b ON a.id = b.id`。

- 非等結合
  
  非等結合では、`<`、`<=`、`>`、`>=`、`<>`などの比較演算子で結合される2つの結合項目の結合条件が使用されます。例：`a JOIN b ON a.id < b.id`。非等結合は等結合よりも実行速度が遅いため、非等結合を使用する際は注意が必要です。

  次の2つの例は、非等結合の実行方法を示しています：

  ```sql
  SELECT t1.id, c1, c2 
  FROM t1 
  INNER JOIN t2 ON t1.id < t2.id;
    
  SELECT t1.id, c1, c2 
  FROM t1 
  LEFT JOIN t2 ON t1.id > t2.id;
  ```

### ORDER BY

SELECT文のORDER BY句は、1つ以上の列の値を比較して結果セットを並べ替えます。

ORDER BYは、結果を並べ替える前にすべての結果を1つのノードに送信してマージする必要があるため、時間とリソースを消費する操作です。並べ替えは、ORDER BYなしのクエリよりも多くのメモリリソースを消費します。

したがって、ソートされた結果セットから最初の `N` 件の結果のみが必要な場合は、LIMIT 句を使用してメモリ使用量とネットワークオーバーヘッドを削減できます。LIMIT 句が指定されていない場合、デフォルトで最初の 65535 件の結果が返されます。

構文：

```sql
ORDER BY <column_name> 
    [ASC | DESC]
    [NULLS FIRST | NULLS LAST]
```

`ASC` は結果を昇順で返すことを指定します。`DESC` は結果を降順で返すことを指定します。順序が指定されていない場合、デフォルトは ASC（昇順）です。例：

```sql
SELECT * FROM big_table ORDER BY tiny_column, short_column DESC;
```

NULL 値のソート順：`NULLS FIRST` は NULL 値を非 NULL 値の前に返すべきことを示します。`NULLS LAST` は NULL 値を非 NULL 値の後に返すべきことを示します。

例：

```sql
SELECT * FROM sales_record ORDER BY employee_id NULLS FIRST;
```

### GROUP BY

GROUP BY 句は、COUNT()、SUM()、AVG()、MIN()、MAX() などの集約関数と共によく使用されます。

GROUP BY で指定された列は集約操作には参加しません。GROUP BY 句は Having 句と組み合わせて使用し、集約関数によって生成された結果をフィルタリングすることができます。

例：

```sql
SELECT tiny_column, SUM(short_column)
FROM small_table 
GROUP BY tiny_column;
```

```plain text
+-------------+---------------------+
| tiny_column | sum(short_column)   |
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
      ... |
      GROUPING SETS ( groupSet [ , groupSet ... ] ) |
      ROLLUP(expr [ , expr ... ]) |
      expr [ , expr ... ] WITH ROLLUP |
      CUBE(expr [ , expr ... ]) |
      expr [ , expr ... ] WITH CUBE
  ]
  [ ... ]
  ```

#### パラメーター

  `groupSet` は、SELECT リスト内の列、エイリアス、または式から構成されるセットを表します。`groupSet ::= { ( expr [ , expr ... ] ) }`

  `expr` は SELECT リスト内の列、エイリアス、または式を指します。

#### 注意

StarRocks は PostgreSQL のような構文をサポートしています。以下に構文の例を示します：

  ```sql
  SELECT a, b, SUM(c) FROM tab1 GROUP BY GROUPING SETS ((a, b), (a), (b), ());
  SELECT a, b, c, SUM(d) FROM tab1 GROUP BY ROLLUP(a, b, c);
  SELECT a, b, c, SUM(d) FROM tab1 GROUP BY CUBE(a, b, c);
  ```

  `ROLLUP(a, b, c)` は以下の `GROUPING SETS` ステートメントと同等です：

  ```sql
  GROUPING SETS (
    (a, b, c),
    (a, b),
    (a),
    ()
  )
  ```

  `CUBE(a, b, c)` は以下の `GROUPING SETS` ステートメントと同等です：

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

#### 例

  以下は実際のデータの例です：

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

  SELECT k1, k2, SUM(k3) FROM t GROUP BY GROUPING SETS ((k1, k2), (k2), (k1), ());
  +------+------+-----------+
  | k1   | k2   | SUM(k3)   |
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

  > SELECT k1, k2, GROUPING_ID(k1, k2), SUM(k3) FROM t GROUP BY GROUPING SETS ((k1, k2), (k1), (k2), ());
  +------+------+-------------------+-----------+
  | k1   | k2   | GROUPING_ID(k1, k2) | SUM(k3) |
  +------+------+-------------------+-----------+
  | a    | A    |                 0 |         3 |
  | a    | B    |                 0 |         4 |
  | a    | NULL |                 1 |         7 |
  | b    | A    |                 0 |         5 |
  | b    | B    |                 0 |         6 |
  | b    | NULL |                 1 |        11 |
  | NULL | A    |                 2 |         8 |
  | NULL | B    |                 2 |        10 |
  | NULL | NULL |                 3 |        18 |
  +------+------+-------------------+-----------+
  9 rows in set (0.02 sec)
  ```

GROUP BY `GROUPING SETS` | `CUBE` | `ROLLUP` は GROUP BY 句の拡張であり、GROUP BY 句で複数のセットのグループを集約することができます。結果は、対応する複数の GROUP BY 句の UNION 操作と同等です。

GROUP BY 句は、1 つの要素のみを含む GROUP BY `GROUPING SETS` の特別なケースです。例えば、以下の GROUPING SETS ステートメント：

  ```sql
  SELECT a, b, SUM(c) FROM tab1 GROUP BY GROUPING SETS ((a, b), (a), (b), ());
  ```

  このクエリ結果は以下と同等です：

  ```sql
  SELECT a, b, SUM(c) FROM tab1 GROUP BY a, b
  UNION
  SELECT a, NULL, SUM(c) FROM tab1 GROUP BY a
  UNION
  SELECT NULL, b, SUM(c) FROM tab1 GROUP BY b
  UNION
  SELECT NULL, NULL, SUM(c) FROM tab1
  ```

  `GROUPING(expr)` は列が集約列かどうかを示します。集約列であれば 0、そうでなければ 1 です。

  `GROUPING_ID(expr [, expr ...])` は GROUPING に似ています。GROUPING_ID は指定された列の順序に従って列リストのビットマップ値を計算し、各ビットは GROUPING の値です。
  
  GROUPING_ID() 関数はビットベクトルの10進値を返します。

### HAVING

HAVING 句はテーブル内の行データをフィルタリングするのではなく、集約関数の結果をフィルタリングします。

一般的に、HAVING は集約関数（COUNT()、SUM()、AVG()、MIN()、MAX() など）と GROUP BY 句と共に使用されます。

例：

```sql
SELECT tiny_column, SUM(short_column) 
FROM small_table 
GROUP BY tiny_column 
HAVING SUM(short_column) = 1;
```

```plain text
+-------------+---------------------+
| tiny_column | sum(short_column)   |
+-------------+---------------------+
|         2   |        1            |
+-------------+---------------------+

1 row in set (0.07 sec)
```

```sql
SELECT tiny_column, SUM(short_column) 
FROM small_table 
GROUP BY tiny_column 
HAVING tiny_column > 1;
```

```plain text
+-------------+---------------------+
| tiny_column | sum(short_column)   |
+-------------+---------------------+
|      2      |        1            |
+-------------+---------------------+

1 row in set (0.07 sec)
```

### LIMIT

LIMIT 句は返される行数の最大数を制限するために使用されます。返される最大行数を設定することで、StarRocks はメモリ使用量を最適化するのに役立ちます。

この句は主に以下のシナリオで使用されます：

トップNクエリの結果を返します。

以下の表に含まれる内容を考えてみてください。

クエリ結果セットのサイズは、テーブル内のデータ量が多いため、または WHERE 句でフィルタリングされるデータが多すぎるために制限する必要があります。

使用指示：LIMIT 句の値は数値リテラル定数でなければなりません。

例：

```plain text
mysql> SELECT tiny_column FROM small_table LIMIT 1;

+-------------+
| tiny_column |
+-------------+
|     1       |
+-------------+

1 row in set (0.02 sec)
```

```plain text
mysql> SELECT tiny_column FROM small_table LIMIT 10000;

+-------------+
| tiny_column |
+-------------+
|      1      |
|      2      |
+-------------+

2 rows in set (0.01 sec)
```

### OFFSET

OFFSET 句を使用すると、結果セットは最初のいくつかの行をスキップして、直後の結果を直接返します。

結果セットはデフォルトで行0から始まるため、OFFSET 0 と OFFSET なしでは同じ結果が返されます。

一般的に、OFFSET 句は ORDER BY 句と LIMIT 句と組み合わせて使用する必要があります。

例：

```plain text
mysql> SELECT varchar_column FROM big_table ORDER BY varchar_column LIMIT 3;

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
mysql> SELECT varchar_column FROM big_table ORDER BY varchar_column LIMIT 1 OFFSET 0;

+----------------+
| varchar_column |
+----------------+
|     beijing    |
+----------------+

1 row in set (0.01 sec)
```

```plain text
mysql> SELECT varchar_column FROM big_table ORDER BY varchar_column LIMIT 1 OFFSET 1;

+----------------+
| varchar_column |
+----------------+
|    chongqing   |
+----------------+

1 row in set (0.01 sec)
```
|    chongqing   | 
+----------------+

1行のセット (0.01秒)
```

```plain text
mysql> select varchar_column from big_table order by varchar_column limit 1 offset 2;

+----------------+
|varchar_column  |
+----------------+
|     tianjin    |     
+----------------+

1行のセット (0.02秒)
```

注: order byを使用せずにoffset構文を使用することは許可されていますが、この場合offsetは意味をなしません。

このケースでは、limit値のみが考慮され、offset値は無視されます。したがって、order byなしで。

offsetが結果セットの最大行数を超えても、結果は得られます。ユーザーはoffsetをorder byと共に使用することを推奨します。

### UNION

複数のクエリの結果を結合します。

**構文:**

```sql
query_1 UNION [DISTINCT | ALL] query_2
```

- DISTINCT（デフォルト）はユニークな行のみを返します。UNIONはUNION DISTINCTと同等です。
- ALLは重複を含む全ての行を結合します。重複排除はメモリ集約的なため、UNION ALLを使用したクエリはより高速でメモリ消費も少なくなります。より良いパフォーマンスのためにはUNION ALLを使用してください。

> **注記**
>
> 各クエリ文は同じ数の列を返す必要があり、列のデータ型は互換性がある必要があります。

**例:**

`select1` と `select2` のテーブルを作成します。

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

例1: 2つのテーブルの全IDを重複を含めて返します。

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
11行のセット (0.02秒)
```

例2: 2つのテーブルの全ユニークなIDを返します。以下の2つのステートメントは同等です。

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
6行のセット (0.01秒)

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
5行のセット (0.02秒)
```

例3: 2つのテーブルの全ユニークなIDの中から最初の3つのIDを返します。以下の2つのステートメントは同等です。

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
4行のセット (0.11秒)

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
3行のセット (0.01秒)
```

### **INTERSECT**

複数のクエリの結果の交差部分、つまり全ての結果セットに現れる結果を計算します。この節は結果セットの中でユニークな行のみを返します。ALLキーワードはサポートされていません。

**構文:**

```SQL
query_1 INTERSECT [DISTINCT] query_2
```

> **注記**
>
> - INTERSECTはINTERSECT DISTINCTと同等です。
> - 各クエリ文は同じ数の列を返す必要があり、列のデータ型は互換性がある必要があります。

**例:**

UNIONで使用される2つのテーブルを使用します。

両方のテーブルに共通する`(id, price)`のユニークな組み合わせを返します。以下の2つのステートメントは同等です。

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

左側のクエリに存在し、右側のクエリには存在しないユニークな結果を返します。EXCEPTはMINUSと同等です。

**構文:**

```SQL
query_1 {EXCEPT | MINUS} [DISTINCT] query_2
```

> **注記**
>
> - EXCEPTはEXCEPT DISTINCTと同等です。ALLキーワードはサポートされていません。
> - 各クエリ文は同じ数の列を返す必要があり、列のデータ型は互換性がある必要があります。

**例:**

UNIONで使用される2つのテーブルを使用します。

`select1`に存在し`select2`には存在しない`(id, price)`のユニークな組み合わせを返します。

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

DISTINCTキーワードは結果セットの重複を排除します。例:

```SQL
-- 一つの列からユニークな値を返します。
select distinct tiny_column from big_table limit 2;

-- 複数の列からユニークな値の組み合わせを返します。
select distinct tiny_column, int_column from big_table limit 2;
```

DISTINCTは集計関数（通常はcount関数）と共に使用でき、count(distinct)は一つまたは複数の列に含まれる異なる組み合わせの数を計算するために使用されます。

```SQL
-- 一つの列からユニークな値を数えます。
select count(distinct tiny_column) from small_table;
```

```plain text
+-------------------------------+
| count(DISTINCT tiny_column) |
+-------------------------------+
|             2                 |
+-------------------------------+
1行のセット (0.06秒)
```

```SQL
-- 複数の列からユニークな値の組み合わせを数えます。
select count(distinct tiny_column, int_column) from big_table limit 2;
```

StarRocksはdistinctを使用する複数の集計関数を同時にサポートしています。

```SQL
-- 複数の集計関数からユニークな値を個別に数えます。
select count(distinct tiny_column, int_column), count(distinct varchar_column) from big_table;
```

### サブクエリ

サブクエリは関連性の観点から2つのタイプに分類されます:

- 非相関サブクエリは外部クエリとは独立して結果を取得します。
- 相関サブクエリは外部クエリからの値が必要です。

#### 非相関サブクエリ

非相関サブクエリは[NOT] INとEXISTSをサポートします。

例:

```sql
SELECT x FROM t1 WHERE x [NOT] IN (SELECT y FROM t2);

SELECT * FROM t1 WHERE (x,y) [NOT] IN (SELECT x,y FROM t2 LIMIT 2);

SELECT x FROM t1 WHERE EXISTS (SELECT y FROM t2 WHERE y = 1);
```

v3.0以降、`SELECT... FROM... WHERE... [NOT] IN`のWHERE句で複数のフィールドを指定できます。例えば、2番目のSELECT文の`WHERE (x,y)`です。

#### 相関サブクエリ

相関サブクエリは[NOT] INと[NOT] EXISTSをサポートします。

例:

```sql
SELECT * FROM t1 WHERE x [NOT] IN (SELECT a FROM t2 WHERE t1.y = t2.b);

SELECT * FROM t1 WHERE [NOT] EXISTS (SELECT a FROM t2 WHERE t1.y = t2.b);
```

サブクエリはスカラー量子クエリもサポートしています。これは無関係なスカラー量子クエリ、関連するスカラー量子クエリ、一般関数のパラメータとしてのスカラー量子クエリに分けられます。

例:

1. 述語=記号を持つ非相関スカラー量子クエリ。例えば、最高給与の人に関する情報を出力します。

    ```sql
    SELECT name FROM table WHERE salary = (SELECT MAX(salary) FROM table);
    ```

2. 述語`>`、`<`などを持つ非相関スカラー量子クエリ。例えば、平均よりも多くの給与を得ている人に関する情報を出力します。

    ```sql
    SELECT name FROM table WHERE salary > (SELECT AVG(salary) FROM table);
    ```

3. 関連するスカラー量子クエリ。例えば、各部門で最高の給与情報を出力します。

    ```sql
    SELECT name FROM table a WHERE salary = (SELECT MAX(salary) FROM table b WHERE b.Department = a.Department);
    ```

4. スカラー量子クエリは一般関数のパラメータとして使用されます。

    ```sql
    SELECT name FROM table WHERE salary = abs((SELECT MAX(salary) FROM table));
    ```

### Whereと演算子

SQL演算子は比較に使用される一連の関数で、SELECT文のWHERE句で広く使用されています。

#### 算術演算子

算術演算子は通常、左オペランド、右オペランド、そして大抵の場合は左オペランドを含む式に現れます

**+ と -**: 単項または二項演算子として使用できます。単項演算子として使用される場合、例えば +1、-2.5、-col_name は、値が +1 または -1 で乗算されることを意味します。

そのため、単項演算子 + は値を変更せずに返し、単項演算子 - はその値の符号ビットを変更します。

ユーザーは二つの単項演算子を重ねて使用することができます。例えば +5（正の値を返す）、-+2 または +-2（負の値を返す）ですが、連続する二つの - 符号を使用することはできません。

なぜなら -- は次のステートメントでコメントとして解釈されるからです（ユーザーが二つの符号を使用する場合、二つの符号の間にスペースまたは括弧が必要です。例えば -(-2) や --2 は実際には +2 となります）。

+ または - が二項演算子である場合、例えば 2+2、3+1.5、col1+col2 といった場合、左の値が右の値に加算または減算されることを意味します。左右の値はともに数値型でなければなりません。

**× と ÷**: それぞれ乗算と除算を表します。両側のオペランドはデータ型でなければなりません。二つの数値が乗算されるとき、

必要に応じて小さいオペランドは昇格されることがあります（例えば、SMALLINT から INT または BIGINT へ）、そして式の結果はより大きな型へ昇格されます。

例えば、TINYINT と INT の乗算は BIGINT 型の結果を生成します。二つの数値が乗算されると、精度の損失を避けるためにオペランドと式の結果は両方とも DOUBLE 型として解釈されます。

ユーザーが式の結果を別の型に変換したい場合は、CAST 関数を使用して型変換する必要があります。

**%**: 剰余演算子。左オペランドを右オペランドで割った余りを返します。左右のオペランドは整数でなければなりません。

**&、| と ^**: ビット単位の演算子は、二つのオペランドに対するビット単位の AND、OR、XOR 演算の結果を返します。どちらのオペランドも整数型である必要があります。

ビット単位の演算子の二つのオペランドの型が一致しない場合、小さい型のオペランドは大きい型のオペランドに昇格され、対応するビット単位の演算が行われます。

複数の算術演算子が式に現れることがあり、ユーザーは対応する算術式を括弧で囲むことができます。算術演算子には通常、算術演算子と同じ機能を表す数学関数がありません。

例えば、% 演算子を表す MOD() 関数はありません。逆に、数学関数には対応する算術演算子がありません。例えば、べき乗関数 POW() には対応する ** 指数演算子がありません。

 ユーザーは、数学関数セクションを通じて、サポートされている算術関数を確認することができます。

#### BETWEEN 演算子

WHERE 句では、式を上限と下限の両方と比較することができます。式が下限以上かつ上限以下であれば、比較の結果は真です。

構文：

```sql
expression BETWEEN lower_bound AND upper_bound
```

データ型: 通常、式は数値型に評価されますが、他のデータ型もサポートされています。下限と上限が比較可能な文字であることを確実にする必要がある場合は、cast() 関数を使用できます。

 使用説明: オペランドが文字列型の場合、上限で始まる長い文字列は上限と一致せず、上限よりも大きいとみなされることに注意してください。例えば、"between 'A' and 'M'" は 'MJ' とは一致しません。

 式が正しく機能することを確実にするためには、upper()、lower()、substr()、trim() などの関数を使用することができます。

例：

```sql
select c1 from t1 where month BETWEEN 1 AND 6;
```

#### 比較演算子

比較演算子は二つの値を比較するために使用されます。`=`, `!=`, `>=` は全てのデータ型に適用されます。

`<>` 演算子と `!=` 演算子は同等で、二つの値が等しくないことを示します。

#### IN 演算子

IN 演算子は値のコレクションと比較し、コレクション内のいずれかの要素と一致する場合に TRUE を返します。

パラメータと値のコレクションは比較可能でなければなりません。IN 演算子を使用する全ての式は、OR で接続された等価な比較として書き換えることができますが、IN の構文はよりシンプルで、より明確で、StarRocks による最適化が容易です。

例：

```sql
select * from small_table where tiny_column IN (1,2);
```

#### LIKE 演算子

この演算子は文字列との比較に使用されます。'_' は一文字にマッチし、'%' は複数の文字にマッチします。パラメータは完全な文字列と一致する必要があります。通常、文字列の末尾に '%' を置くことが実用的です。

例：

```plain text
mysql> select varchar_column from small_table where varchar_column LIKE 'm%';

+----------------+
| varchar_column |
+----------------+
|     milan      |
+----------------+

1 row in set (0.02 sec)
```

```plain
mysql> select varchar_column from small_table where varchar_column LIKE 'm____';

+----------------+
| varchar_column |
+----------------+
|    milan       |
+----------------+

1 row in set (0.01 sec)
```

#### 論理演算子

論理演算子は BOOL 値を返し、単項および複数の演算子を含み、それぞれが BOOL 値を返す式のパラメータを処理します。サポートされている演算子は以下の通りです。

AND: 二項演算子で、AND 演算子は左右のパラメータが両方 TRUE と評価された場合に TRUE を返します。

OR: 二項演算子で、左右のパラメータのどちらかが TRUE と評価された場合に TRUE を返します。両方のパラメータが FALSE の場合、OR 演算子は FALSE を返します。

NOT: 単項演算子で、式の結果を反転します。パラメータが TRUE の場合、演算子は FALSE を返し、パラメータが FALSE の場合、演算子は TRUE を返します。

例：

```plain text
mysql> select TRUE AND TRUE;

+-------------------+
| (TRUE) AND (TRUE) |
+-------------------+
|         1         |
+-------------------+

1 row in set (0.00 sec)
```

```plain text
mysql> select TRUE AND FALSE;

+--------------------+
| (TRUE) AND (FALSE) |
+--------------------+
|         0          |
+--------------------+

1 row in set (0.01 sec)
```

```plain text
mysql> select TRUE OR FALSE;

+-------------------+
| (TRUE) OR (FALSE) |
+-------------------+
|        1          |
+-------------------+

1 row in set (0.01 sec)
```

```plain text
mysql> select NOT TRUE;

+----------+
| NOT TRUE |
+----------+
|     0    |
+----------+

1 row in set (0.01 sec)
```

#### 正規表現演算子

正規表現がマッチするかどうかを判定します。POSIX 標準の正規表現を使用すると、'^' は文字列の最初の部分にマッチし、'$' は文字列の末尾にマッチします。

"." は任意の一文字にマッチし、"*" はゼロ個以上の任意の文字にマッチし、"+" は一個以上の任意の文字にマッチし、"?" は最短一致を意味します。正規表現は文字列の一部ではなく、完全な値にマッチする必要があります。

中間部分にマッチさせたい場合は、正規表現の前部分を '^.*' または '.*' と書くことができます。通常、'^' と '$' は省略されます。RLIKE 演算子と REGEXP 演算子は同義です。

'|' 演算子は選択演算子で、'|' の両側の正規表現はどちらか一方の条件を満たせばよいです。'|' 演算子と両側の正規表現は通常、() で囲む必要があります。

例：

```plain text
mysql> select varchar_column from small_table where varchar_column REGEXP '(mi|MI).*';

+----------------+
| varchar_column |
+----------------+
|     milan      |
+----------------+

1 row in set (0.01 sec)
```

```plain text
mysql> select varchar_column from small_table where varchar_column REGEXP 'm.*';

+----------------+
| varchar_column |
+----------------+
|     milan      |
+----------------+

1 row in set (0.01 sec)
```

### エイリアス


クエリ内でテーブル名、カラム名、またはカラムを含む式の名前を記述する際、それらにエイリアスを割り当てることができます。エイリアスは通常、元の名前よりも短く、覚えやすいものです。

エイリアスが必要な場合、selectリストやfromリスト内のテーブル名、カラム名、式名の後にAS句を追加するだけです。ASキーワードは省略可能です。また、ASを使用せずに、元の名前の直後にエイリアスを指定することもできます。

エイリアスや他の識別子が[StarRocksの予約語](../keywords.md)と同じ名前の場合、その名前をバッククォートで囲む必要があります。例えば、`rank`のように。

エイリアスは大文字と小文字を区別します。

例：

```sql
select tiny_column as name, int_column as sex from big_table;

select sum(tiny_column) as total_count from big_table;

select one.tiny_column, two.int_column from small_table one, <br/> big_table two where one.tiny_column = two.tiny_column;
```
