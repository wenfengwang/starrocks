---
displayed_sidebar: "Japanese"
---

# SELECT（抽出）

## 説明

1つ以上のテーブル、ビュー、またはマテリアライズドビューからデータをクエリします。SELECTステートメントは通常、以下の節から構成されています。

- [WITH](#with)
- [WHEREとオペレータ](#where-and-operators)
- [GROUP BY](#group-by)
- [HAVING](#having)
- [UNION](#union)
- [INTERSECT](#intersect)
- [EXCEPT/MINUS](#exceptminus)
- [ORDER BY](#order-by)
- [LIMIT](#limit)
- [OFFSET](#offset)
- [ジョイン](#join)
- [サブクエリ](#subquery)
- [DISTINCT](#distinct)
- [エイリアス](#alias)

SELECTは独立したステートメントとして機能することも、他のステートメントにネストされた節として機能することもあります。SELECT節の出力は、他のステートメントの入力として使用できます。

StarRocksのクエリステートメントは、基本的にSQL92標準に準拠しています。ここでは、サポートされるSELECTの使用法について簡単に説明します。

> **注意**
>
> StarRocksの内部テーブルからデータをクエリするには、これらのオブジェクトに対するSELECT権限を持っている必要があります。外部データソースのテーブル、ビュー、またはマテリアライズドビューからデータをクエリするには、対応する外部カタログに対するUSAGE権限を持っている必要があります。

### WITH

SELECTステートメントの前に追加できる節で、SELECT内で複数回参照される複雑な式に対するエイリアスを定義します。

CRATE VIEWと同様ですが、節で定義されたテーブルと列名はクエリ終了後に永続化されず、実際のテーブルやVIEWの名前と競合しません。

WITH節を使用する利点は次のとおりです。

・クエリ内の重複を減らすことで、便利でメンテナンスしやすくし、クエリ内の複雑な部分を別々のブロックに抽象化することで、SQLコードの読みやすさを向上させることができます。

例：

```sql
-- 外側のレベルで1つのサブクエリを定義し、UNION ALLクエリの初期段階の一部として内側のレベルで別のサブクエリを定義します。

with t1 as (select 1),t2 as (select 2)
select * from t1 union all select * from t2;
```

### ジョイン

ジョイン操作は、2つ以上のテーブルからデータを結合し、それらの一部の列からなる結果セットを返します。

StarRocksは、セルフジョイン、クロスジョイン、インナージョイン、アウタージョイン、セミジョイン、およびアンチジョインをサポートしています。アウタージョインには、左ジョイン、右ジョイン、およびフルジョインが含まれます。

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

StarRocksはセルフジョインをサポートしており、これは同じテーブルの異なる列が結合されるセルフジョインです。

セルフジョインを識別する特別な構文は実際にはありません。セルフジョインの結合条件の両側の条件は同じテーブルから来ます。

それらにそれぞれ異なるエイリアスを割り当てる必要があります。

例：

```sql
SELECT lhs.id, rhs.parent, lhs.c1, rhs.c2 FROM tree_data lhs, tree_data rhs WHERE lhs.id = rhs.parent;
```

#### クロスジョイン

クロスジョインは多くの結果を生成する可能性があるため、慎重に使用する必要があります。

クロスジョインを使用する必要がある場合でも、フィルタ条件を使用し、返される結果が少なくなるようにする必要があります。例：

```sql
SELECT * FROM t1, t2;

SELECT * FROM t1 CROSS JOIN t2;
```

#### インナージョイン

インナージョインはもっともよく知られ、一般的に使用されるジョインです。似ている2つのテーブルからリクエストされた列の結果を返し、両方のテーブルの列に同じ値が含まれている場合に結合します。

両テーブルの列名が同じ場合は、フルネーム（table_name.column_nameの形式で）を使用するか、列名にエイリアスを付ける必要があります。

例：

次の3つのクエリは同等です。

```sql
SELECT t1.id, c1, c2 FROM t1, t2 WHERE t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 JOIN t2 ON t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id = t2.id;
```

#### アウタージョイン

アウタージョインは左側または右側のテーブル、または両方の行を返します。他のテーブルに一致するデータがない場合は、NULLに設定します。例：

```sql
SELECT * FROM t1 LEFT OUTER JOIN t2 ON t1.id = t2.id;

SELECT * FROM t1 RIGHT OUTER JOIN t2 ON t1.id = t2.id;

SELECT * FROM t1 FULL OUTER JOIN t2 ON t1.id = t2.id;
```

#### 等価および非等価ジョイン

通常、ユーザーは最も等しいジョインを使用し、ジョイン条件のオペレータが等号であることを要求します。

非等価ジョインは、ジョイン条件!=、等号を使用します。非等価ジョインは多くの結果を生成し、計算中にメモリ制限を超過する可能性があります。

慎重に使用してください。非等価ジョインは内部ジョインのみをサポートします。例：

```sql
SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id = t2.id;

SELECT t1.id, c1, c2 FROM t1 INNER JOIN t2 ON t1.id > t2.id;
```

#### 左セミジョイン

左側のセミジョインは、右側のテーブルと一致する行のみを返し、右側のテーブルとのデータの一致の数に関係なく左側の行を返します。

この左側の行は最大で1回返されます。右側のセミジョインも同様に機能しますが、返されるデータは右側のテーブルです。

例：

```sql
SELECT t1.c1, t1.c2, t1.c2 FROM t1 LEFT SEMI JOIN t2 ON t1.id = t2.id;
```

#### アンチジョイン

左アンチジョインは、右側のテーブルと一致しない左側のテーブルの行のみを返します。

右アンチジョインはこの比較を反転し、左側のテーブルと一致しない右側のテーブルの行のみを返します。例：

```sql
SELECT t1.c1, t1.c2, t1.c2 FROM t1 LEFT ANTI JOIN t2 ON t1.id = t2.id;
```

#### 等価ジョインと非等価ジョイン

StarRocksがサポートする様々なジョインは、ジョイン条件によって等価ジョインと非等価ジョインに分類されます。

| **等価ジョイン**            | セルフジョイン、クロスジョイン、インナージョイン、アウタージョイン、セミジョイン、およびアンチジョイン |
| -------------------------- | ------------------------------------------------------------ |
| **非等価ジョイン**           | クロスジョイン、インナージョイン、左セミジョイン、左アンチジョイン、およびアウタージョイン   |

- 等価ジョイン
  
  等価ジョインは、`=`演算子によって2つの結合項目が結合条件として使用されるジョインです。例：`a JOIN b ON a.id = b.id`。

- 非等価ジョイン
  
  非等価ジョインは、`<`、`<=`、`>`、`>=`、または`<>`などの比較演算子によって、2つの結合項目が結合条件として使用されるジョインです。例：`a JOIN b ON a.id < b.id`。非等価ジョインは等価ジョインよりも遅く実行されます。非等価ジョインを使用する際は注意してください。

  次の2つの例では非等価ジョインの実行方法を示しています：

  ```SQL
  SELECT t1.id, c1, c2 
  FROM t1 
  INNER JOIN t2 ON t1.id < t2.id;
    
  SELECT t1.id, c1, c2 
  FROM t1 
  LEFT JOIN t2 ON t1.id > t2.id;
  ```

### ORDER BY（順序付け）

SELECTステートメントのORDER BY節は、1つ以上の列の値を比較して結果セットを並べ替えます。

ORDER BYは時間とリソースを消費する操作であり、結果全体をソートする前にすべての結果を1つのノードに送信する必要があります。ソートには、ORDER BYのないクエリよりも多くのメモリリソースを消費します。そのため、ソートされた結果セットから最初の`N`個の結果のみを必要とする場合は、メモリ使用量とネットワークオーバーヘッドを削減するLIMIT節を使用することができます。LIMIT節が指定されていない場合は、デフォルトで最初の65535個の結果が返されます。

構文：

```sql
ORDER BY <column_name>
```
```sql
      + {T}
      + {T}
    + {T}
  + {T}
```

```sql
      + {T}
      + {T}
    + {T}
  + {T}
```
```plain text
mysql> select 1;

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

OFFSET

オフセット句は、結果セットが最初の数行をスキップして、その後の結果を直接返すようにします。

結果セットのデフォルトは行0から開始するため、オフセット0とオフセットなしでは、同じ結果が返されます。

一般的には、オフセット句は有効にするために ORDER BY 句と LIMIT 句とともに使用する必要があります。

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

注意: ORDER BY なしで offset 構文を使用することは許可されていますが、この時点では offset は意味をなしません。

この場合、limit value のみが取られ、offset value は無視されます。そのため、ORDER BY なしで。

Offset は、結果セット内の最大行数を超え、それでも結果です。ユーザーは ORDER BY を使用することを推奨します。

### UNION

複数のクエリの結果を結合します。

**構文:**

```sql
query_1 UNION [DISTINCT | ALL] query_2
```

- DISTINCT(デフォルト)はユニークな行のみを返します。UNION は UNION DISTINCT と同等です。
- ALL は重複を含むすべての行を結合します。重複を削除するにはメモリを使用するため、UNION ALL を使用したクエリは高速でメモリを消費しません。パフォーマンス向上のため、UNION ALL を使用してください。

> **注意**
>
> 各クエリステートメントは同じ数の列を返し、列は互換性のあるデータ型である必要があります。

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

例2: 2つのテーブルのユニークなIDをすべて返します。次の2つのステートメントは同等です。

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

例3: 2つのテーブルのすべてのユニークなIDの中から最初の3つを返します。次の2つのステートメントは同等です。

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

複数のクエリ結果の共通部分、つまりすべての結果セットに表示される結果を計算します。ALL キーワードはサポートされません。

**構文:**

```SQL
query_1 INTERSECT [DISTINCT] query_2
```

> **注意**
>
> - INTERSECT は INTERSECT DISTINCT と同等です。
> - 各クエリステートメントは同じ数の列を返し、列は互換性のあるデータ型である必要があります。

**例:**

UNION で使用される2つのテーブルを使用します。

両方のテーブルで一意な `(id, price)` の組み合わせを返します。次の2つのステートメントは同等です。

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

左側のクエリに存在しない右側のクエリの一意な結果を返します。EXCEPT は MINUS と同等です。

**構文:**

```SQL
query_1 {EXCEPT | MINUS} [DISTINCT] query_2
```

> **注意**
>
> - EXCEPT は EXCEPT DISTINCT と同等です。ALL キーワードはサポートされていません。
> - 各クエリステートメントは同じ数の列を返し、列は互換性のあるデータ型である必要があります。

**例:**

UNION で使用される2つのテーブルを使用します。

`select1` で見つからない `select2` の `(id, price)` の一意な結果を返します。

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

DISTINCT キーワードは、結果セットを重複させません。例:

```SQL
-- 1つの列からユニークな値を返します。
select distinct tiny_column from big_table limit 2;

-- 複数の列からの値のユニークな組み合わせを返します。
select distinct tiny_column, int_column from big_table limit 2;
```

DISTINCT は集約関数 (通常は count 関数) と一緒に使用でき、count (distinct) は1つまたは複数の列に含まれる異なる組み合わせがいくつ含まれるかを計算します。

```SQL
-- 1つの列からユニークな値の数を数えます。
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
-- 複数の列からの値のユニークな組み合わせを数えます。
select count(distinct tiny_column, int_column) from big_table limit 2;
```

StarRocks は distinct を同時に使用する複数の集約関数をサポートしています。

```SQL
-- 複数の集約関数で別々に唯一の値を数えます。
select count(distinct tiny_column, int_column), count(distinct varchar_column) from big_table;
```

### サブクエリ

サブクエリは、関連性の観点から2つのタイプに分類されます。

- 非相関サブクエリは、外側のクエリとは独立してその結果を取得します。
- 相関サブクエリは、外側のクエリから値を必要とします。

#### 相関サブクエリ

相関サブクエリは[NOT] INおよびEXISTSをサポートしています。

例：

```sql
SELECT x FROM t1 WHERE x [NOT] IN (SELECT y FROM t2);

SELECT * FROM t1 WHERE (x,y) [NOT] IN (SELECT x,y FROM t2 LIMIT 2);

SELECT x FROM t1 WHERE EXISTS (SELECT y FROM t2 WHERE y = 1);
```

バージョン3.0からは、`SELECT... FROM... WHERE... [NOT] IN`のWHERE句で複数のフィールドを指定できます。たとえば、2番目のSELECTステートメントのWHERE句で`WHERE (x,y)`を使用できます。

#### 非相関サブクエリ

非相関サブクエリは[NOT] INおよび[NOT] EXISTSをサポートしています。

例：

```sql
SELECT * FROM t1 WHERE x [NOT] IN (SELECT a FROM t2 WHERE t1.y = t2.b);

SELECT * FROM t1 WHERE [NOT] EXISTS (SELECT a FROM t2 WHERE t1.y = t2.b);
```

サブクエリはスカラー量子クエリもサポートしています。これには無関係なスカラー量子クエリ、関連したスカラー量子クエリ、一般的な関数のパラメーターとしてのスカラー量子クエリがあります。

例：

1. プレディケート=サインを持つ非相関スカラー量子クエリ。たとえば、最高賃金の人物に関する情報を出力します。

    ```sql
    SELECT name FROM table WHERE salary = (SELECT MAX(salary) FROM table);
    ```

2. プレディケート`>`、`<`などを持つ非相関スカラー量子クエリ。たとえば、平均以上に支払われる人々の情報を出力します。

    ```sql
    SELECT name FROM table WHERE salary > (SELECT AVG(salary) FROM table);
    ```

3. 関連したスカラー量子クエリ。たとえば、それぞれの部署における最高給与情報を出力します。

    ```sql
    SELECT name FROM table a WHERE salary = (SELECT MAX(salary) FROM table b WHERE b.Department= a.Department);
    ```

4. スカラー量子クエリは一般的な関数のパラメーターとして使用されます。

    ```sql
    SELECT name FROM table WHERE salary = abs((SELECT MAX(salary) FROM table));
    ```

### WHEREおよび演算子

SQL演算子は比較に使用される一連の関数で、selectステートメントのwhere句で広く使用されます。

#### 算術演算子

算術演算子は通常、左、右、および最も左オペランドを含む式に表示されます

**+および-**: ユニット演算子または2項演算子として使用できます。単位演算子として使用される場合、+1、-2.5または-col_nameなど、その値が+1または-1で乗算されることを意味します。

したがって、セル演算子+は変化なしの値を返し、セル演算子-はその値のシンボルビットを変更します。

ユーザーは+5（正の値を返す）、-+2または+2（負の値を返す）のように2つのセル演算子をオーバーレイすることができますが、ユーザーは2つの連続した-符号を使用することはできません。

なぜなら、--は次の文のコメントとして解釈されるからです（ユーザーが2つの符号を使用できる場合、2つの符号の間に空白または括弧が必要です。たとえば、-(-2)または- -2とします。これにより、実際に+ 2が結果として得られます）。

+または-が2項演算子として使用される場合、たとえば、2+2、3+1.5、またはcol1+col2のように、左の値が右の値から加算または減算されることを意味します。左右の値はどちらも数値型でなければなりません。

**と/**: それぞれ乗算および除算を表します。両側のオペランドはデータ型でなければなりません。2つの数値が乗算されるとき。

必要に応じてより小さいオペランドが昇格される可能性があります（たとえば、SMALLINTからINTまたはBIGINTへ）、および式の結果は次の大きな型へ昇格されます。

たとえば、TINYINTにINTを乗算すると、BIGINT型の結果が生じます。2つの数値が乗算されると、両方のオペランドおよび式の結果は精度の損失を回避するためにDOUBLE型として解釈されます。

ユーザーが式の結果を別のタイプに変換したい場合、CAST関数を使用して変換する必要があります。

**%**: Mod演算子。左側のオペランドを右側のオペランドで割った余りを返します。左右のオペランドはどちらも整数でなければなりません。

**&、|および^**: ビット演算子は、2つのオペランドでのビット毎AND、ビット毎OR、ビット毎XOR演算の結果を返します。両方のオペランドには整数型が必要です。

ビット演算子の2つのオペランドの型が不一致の場合、より小さい型のオペランドがより大きな型のオペランドに昇格され、対応するビット演算が実行されます。

複数の算術演算子が式に表示されることがあり、ユーザーは対応する算術式を括弧で囲むことができます。算術演算子には通常、算術演算子と同じ機能を表す対応する数学関数がありません。

たとえば、%演算子を表すMOD()関数はありません。逆に、数学関数には対応する算術演算子がありません。たとえば、べき乗演算子**を表すべき乗関数POW()はありません。

ユーザーは数学関数を通じてサポートされている算術関数を確認することができます。

#### 間演算子

where句では、式は上限と下限の両方と比較される場合があります。式が下限以上かつ上限以下である場合、比較の結果はtrueになります。

構文：

```sql
expression BETWEEN lower_bound AND upper_bound
```

データ型：通常、式は数値型を評価しますが、他のデータ型もサポートされています。下限および上限の両方が比較可能な文字であることを確認する必要がある場合は、cast()関数を使用できます。

使用方法：オペランドが文字列型の場合、上限から始まる長い文字列は上限と一致せず、上限より大きくなりません。たとえば、「'A'から'M'」間の場合、「MJ」は「'M'」よりも大きくなりません。

式が正しく動作するようにする必要がある場合は、upper()、lower()、substr()、trim()などの関数を使用できます。

例：

```sql
select c1 from t1 where month between 1 and 6;
```

#### 比較演算子

比較演算子は2つの値を比較するために使用されます。`=`、`!=`、`>=`はすべてのデータ型に適用されます。

`<>`および`!=`演算子は等価であり、2つの値が等しくないことを示します。

#### In演算子

In演算子はVALUEコレクションと比較し、コレクション内の要素と一致する場合にTRUEを返します。

パラメーターとVALUEコレクションは比較可能である必要があります。IN演算子を使用したすべての式は、等価な比較がORで接続された形式で書かれることができますが、INの構文はより単純で正確であり、StarRocksが最適化するのにも便利です。

例：

```sql
select * from small_table where tiny_column in (1,2);
```

#### Like演算子

この演算子は文字列を比較するために使用されます。''は一文字に一致し、'%'は複数の文字に一致します。パラメーターは完全に文字列に一致しなければなりません。通常、文字列の末尾に'%'を置くことが有益です。

例：

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

論理演算子はBOOL値を返します。単一および複数の演算子があり、それぞれはBOOL値を返す式のパラメーターを処理します。サポートされている演算子は次のとおりです:

AND: 2項演算子で、AND演算子は左右のパラメーターがともにTRUEとして計算された場合にTRUEを返します。

OR: 2項演算子で、OR演算子は左右のパラメーターのいずれかがTRUEとして計算された場合にTRUEを返します。両方のパラメーターがFALSEの場合、OR演算子はFALSEを返します。

NOT: 単一演算子で、式を反転させた結果です。パラメーターがTRUEの場合、演算子はFALSEを返します。パラメーターがFALSEの場合、演算子はTRUEを返します。

例：

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

正規表現が一致するかどうかを判断します。POSIX標準の正規表現を使用し、'^'は文字列の最初部分に一致し、'$'は文字列の末尾に一致します。

"."は任意の単一の文字に一致し、"*"はゼロ個以上のオプションに一致し、"+"は1個以上のオプションに一致し、"?"は貪欲表現を意味し、などがあります。正規表現は、文字列の一部ではなく、完全な値が一致する必要があります。
```plain text
中央部分を一致させたい場合、正規表現の前部分は'^. 'または'.'. '^'と'$'は通常省略されます。RLIKE演算子とREGEXP演算子は同義語です。

'|'演算子は任意の演算子です。'|'の両側の正規表現は片側条件を満たす必要があります。通常、'|'演算子と両側の正規表現は()で囲む必要があります。

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

### 別名

クエリ内のテーブル、列、または列を含む式の名前を記述する際、それらに別名を付けることができます。別名は通常、元の名前よりも短く、覚えやすいです。

別名が必要な場合、選択リストまたはfromリスト内のテーブル、列、および式の名前の後にAS句を追加するだけです。ASキーワードは任意です。ASを使用せずに元の名前の直後に別名を指定することもできます。

別名または他の識別子が[StarRocksのキーワード](../keywords.md)と同じ名前を持つ場合、バッククォートのペアで名前を囲む必要があります。たとえば、`rank`。

別名は大文字と小文字を区別します。

例：

```sql
select tiny_column as name, int_column as sex from big_table;

select sum(tiny_column) as total_count from big_table;

select one.tiny_column, two.int_column from small_table one, <br/> big_table two where one.tiny_column = two.tiny_column;
```