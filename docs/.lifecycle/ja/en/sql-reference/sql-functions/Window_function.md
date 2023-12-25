---
displayed_sidebar: English
---

# ウィンドウ関数

## 背景

ウィンドウ関数は、組み込み関数の特別なクラスです。集約関数と同様に、複数の入力行に対して計算を行い、1つのデータ値を取得します。違いは、ウィンドウ関数が「group by」メソッドを使用するのではなく、特定のウィンドウ内で入力データを処理することです。各ウィンドウのデータは、`over()`句を使用して並べ替えおよびグループ化できます。ウィンドウ関数は、グループごとに1つの値を計算するのではなく、**行ごとに個別の値を計算します**。この柔軟性により、ユーザーは`select`句に列を追加し、結果セットをさらにフィルター処理できます。ウィンドウ関数は、`select`リストと句の最も外側の位置にのみ表示できます。これは、クエリの最後、つまり、`join`、`where`および`group by`操作が実行された後に有効になります。ウィンドウ関数は、傾向の分析、外れ値の計算、および大規模データのバケット分析の実行によく使用されます。

## 使用方法

ウィンドウ関数の構文:

```SQL
function(args) OVER([partition_by_clause] [order_by_clause] [window_clause])
partition_by_clause ::= PARTITION BY expr [, expr ...]
order_by_clause ::= ORDER BY expr [ASC | DESC] [, expr [ASC | DESC] ...]
```

### 関数

現在サポートされている関数は次のとおりです。

* MIN(), MAX(), COUNT(), SUM(), AVG()
* FIRST_VALUE(), LAST_VALUE(), LEAD(), LAG()
* ROW_NUMBER(), RANK(), DENSE_RANK()
* CUME_DIST(), PERCENT_RANK(), QUALIFY()
* NTILE()
* VARIANCE(), VAR_SAMP(), STD(), STDDEV_SAMP(), COVAR_SAMP(), COVAR_POP(), CORR()

### PARTITION BY 句

`PARTITION BY`句は`GROUP BY`と似ています。入力行を、指定された1つ以上の列に基づいてグループ化します。同じ値を持つ行は一緒にグループ化されます。

### ORDER BY 句

`ORDER BY`句は、基本的に外側の`ORDER BY`と同じです。入力行の順序を定義します。`PARTITION BY`が指定されている場合、`ORDER BY`は各パーティション・グループ内の順序を定義します。唯一の違いは、`OVER`句の`ORDER BY n`（nは正の整数）は操作なしと同等であるのに対し、外側の`ORDER BY`での`n`はn番目の列によるソートを示します。

例:

この例では、`events`テーブルの`date_and_time`列でソートされた1、2、3などの値を持つid列を`select`リストに追加する方法を示しています。

```SQL
SELECT row_number() OVER (ORDER BY date_and_time) AS id,
    c1, c2, c3, c4
FROM events;
```

### Window 句

`window`句は、操作の行の範囲（現在の行に基づく前後の行）を指定するために使用されます。サポートされている構文は、AVG(), COUNT(), FIRST_VALUE(), LAST_VALUE(), SUM()です。MAX()とMIN()の場合、`window`句は`UNBOUNDED PRECEDING`からの開始を指定できます。

構文:

```SQL
ROWS BETWEEN [ { m | UNBOUNDED } PRECEDING | CURRENT ROW] AND [CURRENT ROW | { UNBOUNDED | n } FOLLOWING]
```

例:

次の株式データがあり、株式シンボルがJDR、終値が日次終値であるとします。

```SQL
create table stock_ticker (
    stock_symbol string,
    closing_price decimal(8,2),
    closing_date timestamp);

-- ...データをいくつかロード...

select *
from stock_ticker
order by stock_symbol, closing_date
```

生データは次のように示されました。

```plaintext
+--------------+---------------+---------------------+
| stock_symbol | closing_price | closing_date        |
+--------------+---------------+---------------------+
| JDR          | 12.86         | 2014-10-02 00:00:00 |
| JDR          | 12.89         | 2014-10-03 00:00:00 |
| JDR          | 12.94         | 2014-10-04 00:00:00 |
| JDR          | 12.55         | 2014-10-05 00:00:00 |
| JDR          | 14.03         | 2014-10-06 00:00:00 |
| JDR          | 14.75         | 2014-10-07 00:00:00 |
| JDR          | 13.98         | 2014-10-08 00:00:00 |
+--------------+---------------+---------------------+
```

このクエリでは、ウィンドウ関数を使用して、値が3日間（前日、当日、翌日）の平均株価である`moving_average`列を生成します。最初の日には前日の値がなく、最終日には翌日の値がないため、これら2つの行は2日間の平均値のみを計算します。ここでは`Partition By`はすべてのデータがJDRデータであるため、効果がありません。ただし、他の株式情報がある場合、`Partition By`は各パーティション内でウィンドウ関数が操作されることを保証します。

```SQL
select stock_symbol, closing_date, closing_price,
    avg(closing_price)
        over (partition by stock_symbol
              order by closing_date
              rows between 1 preceding and 1 following
        ) as moving_average
from stock_ticker;
```

次のデータが取得されます。

```plaintext
+--------------+---------------------+---------------+----------------+
| stock_symbol | closing_date        | closing_price | moving_average |
+--------------+---------------------+---------------+----------------+
| JDR          | 2014-10-02 00:00:00 | 12.86         | 12.87          |
| JDR          | 2014-10-03 00:00:00 | 12.89         | 12.89          |
| JDR          | 2014-10-04 00:00:00 | 12.94         | 12.79          |
| JDR          | 2014-10-05 00:00:00 | 12.55         | 13.17          |
| JDR          | 2014-10-06 00:00:00 | 14.03         | 13.77          |
| JDR          | 2014-10-07 00:00:00 | 14.75         | 14.25          |
| JDR          | 2014-10-08 00:00:00 | 13.98         | 14.36          |
+--------------+---------------------+---------------+----------------+
```

## 関数例

ここでは、StarRocksでサポートされているウィンドウ関数について説明します。

### AVG()

構文:

```SQL
AVG(expr) [OVER (*analytic_clause*)]
```

例:

現在の行とその前後の各行のx平均を計算します。

```SQL
select x, property,
    avg(x)
        over (
            partition by property
            order by x
            rows between 1 preceding and 1 following
        ) as 'moving average'
from int_t
where property in ('odd', 'even');
```

```plaintext
+----+----------+----------------+
| x  | property | moving average |
+----+----------+----------------+
| 2  | even     | 3              |
| 4  | even     | 4              |
| 6  | even     | 6              |
| 8  | even     | 8              |
| 10 | even     | 9              |
| 1  | odd      | 2              |
| 3  | odd      | 3              |
| 5  | odd      | 5              |
| 7  | odd      | 7              |
| 9  | odd      | 8              |
+----+----------+----------------+
```

### COUNT()

構文:

```SQL
COUNT(expr) [OVER (analytic_clause)]
```

例:

現在の行から最初の行までのxの出現をカウントします。

```SQL
select x, property,
    count(x)
        over (
            partition by property
            order by x
            rows between unbounded preceding and current row
        ) as 'cumulative count'
from int_t where property in ('odd', 'even');
```

```plaintext
+----+----------+------------------+
| x  | property | cumulative count |
+----+----------+------------------+
| 2  | even     | 1                |
| 4  | even     | 2                |
| 6  | even     | 3                |
| 8  | even     | 4                |
| 10 | even     | 5                |
| 1  | odd      | 1                |
| 3  | odd      | 2                |
| 5  | odd      | 3                |
| 7  | odd      | 4                |
| 9  | odd      | 5                |
+----+----------+------------------+
```

### CUME_DIST()

CUME_DIST()関数は、パーティション内の値の累積分布を計算し、その相対位置を現在の行の値以下の値の割合として示します。0から1の範囲で、パーセンタイル計算とデータ分布分析に役立ちます。

構文:

```SQL
CUME_DIST() OVER (partition_by_clause order_by_clause)
```

**この関数は`ORDER BY`と共に使用して、パーティション行を目的の順序に並べ替える必要があります。`ORDER BY`を使用しない場合、すべての行はピアであり、値はN/N = 1（Nはパーティションサイズ）です。**

CUME_DIST()にはNULL値が含まれ、それらを最小値として扱います。

次の例は、列xの各グループ内の列yの累積分布を示しています。

```SQL
SELECT x, y,
    CUME_DIST()
        OVER (
            PARTITION BY x
            ORDER BY y
        ) AS `cume_dist`
FROM int_t;
```

```plaintext
+---+---+--------------------+
| x | y | cume_dist          |
+---+---+--------------------+
| 1 | 1 | 0.3333333333333333 |
| 1 | 2 |                  1 |
| 1 | 2 |                  1 |
| 2 | 1 | 0.3333333333333333 |
| 2 | 2 | 0.6666666666666666 |
| 2 | 3 |                  1 |
| 3 | 1 | 0.6666666666666666 |
| 3 | 1 | 0.6666666666666666 |
| 3 | 2 |                  1 |
+---+---+--------------------+
```

### DENSE_RANK()

DENSE_RANK() 関数はランキングを表すために使用されます。RANK()と異なり、DENSE_RANK()**は空白の番号を持ちません**。例えば、1位が2つある場合、DENSE_RANK()の次の番号は2ですが、RANK()の次の番号は3になります。

構文：

```SQL
DENSE_RANK() OVER(partition_by_clause order_by_clause)
```

以下の例は、プロパティ列のグループに基づいて列 x のランキングを示しています。

```SQL
SELECT x, y,
    DENSE_RANK()
        OVER (
            PARTITION BY x
            ORDER BY y
        ) AS `rank`
FROM int_t;
```

```plaintext
+---+---+------+
| x | y | rank |
+---+---+------+
| 1 | 1 | 1    |
| 1 | 2 | 2    |
| 1 | 2 | 2    |
| 2 | 1 | 1    |
| 2 | 2 | 2    |
| 2 | 3 | 3    |
| 3 | 1 | 1    |
| 3 | 1 | 1    |
| 3 | 2 | 2    |
+---+---+------+
```

### NTILE()

NTILE() 関数は、パーティション内のソートされた行を指定された数の`num_buckets`にできるだけ均等に分割し、1から始まる各バケットに分割された行を格納し、各行が属するバケット番号を`[1, 2, ..., num_buckets]`として返します。

バケットのサイズについて：

* 行数が指定された`num_buckets`の数で正確に割り切れる場合、すべてのバケットは同じサイズになります。
* 行数が指定された`num_buckets`の数で正確に割り切れない場合、2つの異なるサイズのバケットが存在します。サイズの差は1です。行数が多いバケットが、行数が少ないバケットよりも前にリストされます。

構文：

```SQL
NTILE(num_buckets) OVER(partition_by_clause order_by_clause)
```

`num_buckets`: 作成されるバケットの数。この値は、最大`2^63 - 1`の定数正整数でなければなりません。

ウィンドウ句はNTILE()関数では使用できません。

NTILE()関数はBIGINT型のデータを返します。

例：

以下の例では、パーティション内のすべての行を2つのバケットに分割しています。

```sql
SELECT id, x, y,
    NTILE(2)
        OVER (
            PARTITION BY x
            ORDER BY y
        ) AS bucket_id
FROM t1;
```

```plaintext
+------+------+------+-----------+
| id   | x    | y    | bucket_id |
+------+------+------+-----------+
|    1 |    1 |   11 |         1 |
|    2 |    1 |   11 |         1 |
|    3 |    1 |   22 |         1 |
|    4 |    1 |   33 |         2 |
|    5 |    1 |   44 |         2 |
|    6 |    1 |   55 |         2 |
|    7 |    2 |   66 |         1 |
|    8 |    2 |   77 |         1 |
|    9 |    2 |   88 |         2 |
|   10 |    3 |   99 |         1 |
+------+------+------+-----------+
```

上記の例に示されているように、`num_buckets`が`2`の場合：

* No.1からNo.6までの行は最初のパーティションに分類され、No.1からNo.3までの行は最初のバケットに、No.4からNo.6までの行は2番目のバケットに格納されました。
* No.7からNo.9までの行は2番目のパーティションに分類され、No.7とNo.8の行は最初のバケットに、No.9の行は2番目のバケットに格納されました。
* No.10の行は3番目のパーティションに分類され、最初のバケットに格納されました。

<br/>

### FIRST_VALUE()

FIRST_VALUE()はウィンドウ範囲の**最初の**値を返します。

構文：

```SQL
FIRST_VALUE(expr [IGNORE NULLS]) OVER(partition_by_clause order_by_clause [window_clause])
```

`IGNORE NULLS`はv2.5.0からサポートされています。これは、`expr`のNULL値を計算から除外するかどうかを決定するために使用されます。デフォルトでは、NULL値は含まれており、フィルタリングされた結果の最初の値がNULLの場合はNULLが返されます。IGNORE NULLSを指定すると、フィルタリングされた結果の最初の非NULL値が返されます。すべての値がNULLの場合は、IGNORE NULLSを指定してもNULLが返されます。

例：

以下のデータがあります。

```SQL
SELECT name, country, greeting
FROM mail_merge;
```

```plaintext
+---------+---------+--------------+
| name    | country | greeting     |
+---------+---------+--------------+
| Pete    | USA     | Hello        |
| John    | USA     | Hi           |
| Boris   | Germany | Guten tag    |
| Michael | Germany | Guten morgen |
| Bjorn   | Sweden  | Hej          |
| Mats    | Sweden  | Tja          |
+---------+---------+--------------+
```

FIRST_VALUE()を使用して、国に基づいたグループごとの最初の挨拶値を返します。

```SQL
SELECT country, name,
    FIRST_VALUE(greeting)
        OVER (
            PARTITION BY country
            ORDER BY name, greeting
        ) AS greeting
FROM mail_merge;
```

```plaintext
+---------+---------+-----------+
| country | name    | greeting  |
+---------+---------+-----------+
| Germany | Boris   | Guten tag |
| Germany | Michael | Guten tag |
| Sweden  | Bjorn   | Hej       |
| Sweden  | Mats    | Hej       |
| USA     | John    | Hi        |
| USA     | Pete    | Hi        |
+---------+---------+-----------+
```

### LAG()

LAG()は、現在の行から`offset`行遅れた行の値を返します。この関数は、行間の値を比較し、データをフィルタリングするためによく使用されます。

`LAG()`は、以下のタイプのデータのクエリに使用できます：

* 数値：TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL
* 文字列：CHAR、VARCHAR
* 日付：DATE、DATETIME
* BITMAPとHLLはStarRocks v2.5からサポートされています。

構文：

```SQL
LAG(expr [IGNORE NULLS] [, offset[, default]])
OVER([<partition_by_clause>] [<order_by_clause>])
```

パラメータ：

* `expr`：計算するフィールド。
* `offset`：オフセット。これは**正の整数**でなければなりません。このパラメータが指定されていない場合、デフォルトは1です。
* `default`：一致する行が見つからない場合に返されるデフォルト値。このパラメータが指定されていない場合、デフォルトはNULLです。`default`は、`expr`と互換性のある任意の式をサポートします。
* `IGNORE NULLS`はv3.0からサポートされています。これは、`expr`のNULL値が結果に含まれるかどうかを決定するために使用されます。デフォルトでは、`offset`行を数えるときにNULL値が含まれます。つまり、目的の行の値がNULLの場合はNULLが返されます。IGNORE NULLSを指定すると、`offset`行を数えるときにNULL値は無視され、システムは`offset`の非NULL値を探し続けます。`offset`の非NULL値が見つからない場合は、NULLまたは`default`（指定されている場合）が返されます。

例1：IGNORE NULLSが指定されていない場合

テーブルを作成し、値を挿入します：

```SQL
CREATE TABLE test_tbl (col_1 INT, col_2 INT)
DISTRIBUTED BY HASH(col_1);

INSERT INTO test_tbl VALUES 
    (1, NULL),
    (2, 4),
    (3, NULL),
    (4, 2),
    (5, NULL),
    (6, 7),
    (7, 6),
    (8, 5),
    (9, NULL),
    (10, NULL);
```

このテーブルからデータをクエリします。ここで`offset`は2です。つまり、前の2行をたどることを意味します。`default`は0で、一致する行が見つからない場合に0が返されます。

出力：

```plaintext
SELECT col_1, col_2, LAG(col_2, 2, 0) OVER (ORDER BY col_1) 
FROM test_tbl ORDER BY col_1;
+-------+-------+---------------------------------------------+
| col_1 | col_2 | LAG(col_2, 2, 0) OVER (ORDER BY col_1 ASC) |
+-------+-------+---------------------------------------------+
|     1 |  NULL |                                           0 |
|     2 |     4 |                                           0 |
|     3 |  NULL |                                        NULL |
|     4 |     2 |                                           4 |
|     5 |  NULL |                                        NULL |
|     6 |     7 |                                           2 |
|     7 |     6 |                                        NULL |
|     8 |     5 |                                           7 |
|     9 |  NULL |                                           6 |
|    10 |  NULL |                                           5 |
+-------+-------+---------------------------------------------+
```

最初の2行については、前の2行が存在しないため、デフォルト値の0が返されます。

行3のNULLについては、2行前の値がNULLであり、NULL値が許可されているためNULLが返されます。

例2：IGNORE NULLSが指定されている場合

上記のテーブルとパラメータ設定を使用します。

```SQL
SELECT col_1, col_2, LAG(col_2 IGNORE NULLS, 2, 0) OVER (ORDER BY col_1) 
FROM test_tbl ORDER BY col_1;
+-------+-------+------------------------------------------------+
| col_1 | col_2 | LAG(col_2 IGNORE NULLS, 2, 0) OVER (ORDER BY col_1 ASC) |
+-------+-------+------------------------------------------------+
|     1 |  NULL |                                           0 |
|     2 |     4 |                                           0 |
|     3 |  NULL |                                           0 |
|     4 |     2 |                                           0 |
|     5 |  NULL |                                           4 |
|     6 |     7 |                                           4 |
|     7 |     6 |                                           2 |
|     8 |     5 |                                           7 |
|     9 |  NULL |                                           6 |
|    10 |  NULL |                                           6 |
+-------+-------+---------------------------------------------+
```

行1から4については、それぞれの前の行で2つの非NULL値を見つけることができなかったため、デフォルト値0が返されます。

行7の値6については、2行前がNULLであり、IGNORE NULLSが指定されているため、NULLは無視されます。システムは非NULL値を探し続け、行4の値2が返されます。

### LAST_VALUE()

LAST_VALUE()はウィンドウ範囲の**最後の**値を返します。これはFIRST_VALUE()の逆です。

構文：

```SQL
LAST_VALUE(expr [IGNORE NULLS]) OVER(partition_by_clause order_by_clause [window_clause])
```

`IGNORE NULLS`はv2.5.0からサポートされています。これは、`expr`のNULL値を計算から除外するかどうかを決定するために使用されます。デフォルトでは、NULL値が含まれます。つまり、フィルタリングされた結果の最後の値がNULLである場合、NULLが返されます。IGNORE NULLSを指定すると、フィルタリングされた結果の最後の非NULL値が返されます。すべての値がNULLである場合、IGNORE NULLSを指定してもNULLが返されます。

例のデータを使用します：

```SQL
SELECT country, name,
    last_value(greeting)
        OVER (
            PARTITION BY country
            ORDER BY name, greeting
        ) AS greeting
FROM mail_merge;
```

```plaintext
+---------+---------+--------------+
| country | name    | greeting     |
+---------+---------+--------------+
| Germany | Boris   | Guten Morgen |
| Germany | Michael | Guten Morgen |
| Sweden  | Bjorn   | Tja          |
| Sweden  | Mats    | Tja          |
| USA     | John    | Hello        |
| USA     | Pete    | Hello        |
+---------+---------+--------------+
```

### LEAD()

LEAD()は、現在の行から`offset`行先の行の値を返します。この関数は、行間の値を比較したり、データをフィルタリングしたりするためによく使用されます。

`lead()`によって照会できるデータ型は、[lag()](#lag)でサポートされているものと同じです。

構文：

```sql
LEAD(expr [IGNORE NULLS] [, offset[, default]])
OVER([<partition_by_clause>] [<order_by_clause>])
```

パラメーター：

* `expr`: 計算対象のフィールド。
* `offset`: オフセットで、正の整数でなければなりません。このパラメーターが指定されていない場合、デフォルトは1です。
* `default`: 一致する行が見つからない場合に返されるデフォルト値です。このパラメーターが指定されていない場合、デフォルトはNULLです。`default`は、`expr`と互換性のある型の任意の式をサポートします。
* `IGNORE NULLS`はv3.0からサポートされています。これは、`expr`のNULL値を結果に含めるかどうかを決定するために使用されます。デフォルトでは、`offset`行を数えるときにNULL値が含まれます。つまり、目的の行の値がNULLである場合、NULLが返されます。IGNORE NULLSを指定すると、`offset`行を数えるときにNULL値は無視され、システムは`offset`の非NULL値を探し続けます。`offset`の非NULL値が見つからない場合、NULLまたは`default`(指定されている場合)が返されます。

例1：IGNORE NULLSが指定されていない場合

テーブルを作成し、値を挿入します：

```SQL
CREATE TABLE test_tbl (col_1 INT, col_2 INT)
DISTRIBUTED BY HASH(col_1);

INSERT INTO test_tbl VALUES 
    (1, NULL),
    (2, 4),
    (3, NULL),
    (4, 2),
    (5, NULL),
    (6, 7),
    (7, 6),
    (8, 5),
    (9, NULL),
    (10, NULL);
```

このテーブルからデータをクエリします。ここで`offset`は2で、2行先をトラバースすることを意味します。`default`は0で、一致する行が見つからない場合に0が返されます。

出力：

```plaintext
SELECT col_1, col_2, LEAD(col_2,2,0) OVER (ORDER BY col_1) 
FROM test_tbl ORDER BY col_1;
+-------+-------+----------------------------------------------+
| col_1 | col_2 | LEAD(col_2, 2, 0) OVER (ORDER BY col_1 ASC) |
+-------+-------+----------------------------------------------+
|     1 |  NULL |                                         NULL |
|     2 |     4 |                                            2 |
|     3 |  NULL |                                         NULL |
|     4 |     2 |                                            7 |
|     5 |  NULL |                                            6 |
|     6 |     7 |                                            5 |
|     7 |     6 |                                         NULL |
|     8 |     5 |                                         NULL |
|     9 |  NULL |                                            0 |
|    10 |  NULL |                                            0 |
+-------+-------+----------------------------------------------+
```

最初の行では、2行先の値がNULLであり、NULL値が許可されているため、NULLが返されます。

最後の2行では、2行先が存在しないため、デフォルト値0が返されます。

例2：IGNORE NULLSが指定されている場合

上記のテーブルとパラメータ設定を使用します：

```SQL
SELECT col_1, col_2, LEAD(col_2 IGNORE NULLS,2,0) OVER (ORDER BY col_1) 
FROM test_tbl ORDER BY col_1;
+-------+-------+----------------------------------------------+
| col_1 | col_2 | LEAD(col_2 IGNORE NULLS, 2, 0) OVER (ORDER BY col_1 ASC) |
+-------+-------+----------------------------------------------+
|     1 |  NULL |                                            2 |
|     2 |     4 |                                            7 |
|     3 |  NULL |                                            7 |
|     4 |     2 |                                            6 |
|     5 |  NULL |                                            6 |
|     6 |     7 |                                            5 |
|     7 |     6 |                                            0 |
|     8 |     5 |                                            0 |
|     9 |  NULL |                                            0 |
|    10 |  NULL |                                            0 |
+-------+-------+----------------------------------------------+
```

行7から10については、システムは2行先の非NULL値を見つけることができず、デフォルト値0が返されます。

最初の行では、2行先の値がNULLであり、IGNORE NULLSが指定されているため、NULLは無視されます。システムは非NULL値を探し続け、行4の値2が返されます。

### MAX()

現在のウィンドウ内の指定された行の最大値を返します。

構文：

```SQL
MAX(expr) OVER (analytic_clause)
```

例：

最初の行から現在の行の後の行までの行の最大値を計算します。

```SQL
SELECT x, property,
    MAX(x)
        OVER (
            ORDER BY property, x
            ROWS BETWEEN UNBOUNDED PRECEDING AND 1 FOLLOWING
        ) AS 'local maximum'
FROM int_t
WHERE property IN ('prime', 'square');
```

```plaintext
+---+----------+---------------+
| x | property | local maximum |
+---+----------+---------------+
| 2 | prime    | 3             |
| 3 | prime    | 5             |
| 5 | prime    | 7             |
| 7 | prime    | 7             |
| 1 | square   | 7             |
| 4 | square   | 9             |
| 9 | square   | 9             |
+---+----------+---------------+
```

StarRocks 2.4以降では、行範囲を`ROWS BETWEEN n PRECEDING AND n FOLLOWING`として指定できるようになり、現在の行の前の`n`行と後の`n`行をキャプチャできます。

ステートメントの例：

```sql
SELECT x, property,
    MAX(x)
        OVER (
            ORDER BY property, x
            ROWS BETWEEN 3 PRECEDING AND 2 FOLLOWING
        ) AS 'local maximum'
FROM int_t
WHERE property IN ('prime', 'square');
```

### MIN()

現在のウィンドウ内の指定された行の最小値を返します。

構文：

```SQL
MIN(expr) OVER (analytic_clause)
```

例：

最初の行から現在の行の後の行までの行の最小値を計算します。

```SQL
SELECT x, property,
    MIN(x)
        OVER (
            ORDER BY property, x DESC
            ROWS BETWEEN UNBOUNDED PRECEDING AND 1 FOLLOWING
        ) AS 'local minimum'
FROM int_t
WHERE property IN ('prime', 'square');
```

```plaintext
+---+----------+---------------+
| x | property | local minimum |
+---+----------+---------------+
| 7 | prime    | 5             |
| 5 | prime    | 3             |
| 3 | prime    | 2             |
| 2 | prime    | 2             |
| 9 | square   | 2             |
| 4 | square   | 1             |
| 1 | square   | 1             |
+---+----------+---------------+
```

StarRocks 2.4以降では、行範囲を`ROWS BETWEEN n PRECEDING AND n FOLLOWING`として指定できるようになり、現在の行の前の`n`行と後の`n`行をキャプチャできます。

ステートメントの例：

```sql
SELECT x, property,
    MIN(x)
    OVER (
          ORDER BY property, x DESC
          ROWS BETWEEN 3 PRECEDING AND 2 FOLLOWING
        ) AS 'local minimum'
FROM int_t
WHERE property IN ('prime', 'square');
```

### PERCENT_RANK()

PERCENT_RANK() 関数は、結果セット内の行の相対ランクをパーセンテージで計算します。現在の行の値より小さいパーティション値の割合（最大値を除く）を返します。戻り値の範囲は 0 から 1 です。この関数は、パーセンタイルの計算やデータ分布の分析に役立ちます。

PERCENT_RANK() 関数は、次の式を使用して計算されます。ここで rank は行のランクを表し、rows はパーティション内の行数を表します。

```plaintext
(rank - 1) / (rows - 1)
```

構文：

```SQL
PERCENT_RANK() OVER (PARTITION BY partition_by_clause ORDER BY order_by_clause)
```

**この関数は ORDER BY と共に使用して、パーティション内の行を所望の順序に並べ替える必要があります。ORDER BY がない場合、すべての行は同等と見なされ、値は (1 - 1)/(N - 1) = 0 となります。ここで N はパーティションのサイズです。**

次の例は、列 x の各グループ内での列 y の相対ランクを示しています。

```SQL
SELECT x, y,
    PERCENT_RANK()
        OVER (
            PARTITION BY x
            ORDER BY y
        ) AS `percent_rank`
FROM int_t;
```

```plaintext
+---+---+--------------+
| x | y | percent_rank |
+---+---+--------------+
| 1 | 1 |            0 |
| 1 | 2 |          0.5 |
| 1 | 2 |          0.5 |
| 2 | 1 |            0 |
| 2 | 2 |          0.5 |
| 2 | 3 |            1 |
| 3 | 1 |            0 |
| 3 | 1 |            0 |
| 3 | 2 |            1 |
+---+---+--------------+
```

### RANK()

RANK() 関数はランキングを表すために使用されます。DENSE_RANK() と異なり、RANK() は**空番**として現れます。たとえば、同点の 1 が 2 つある場合、RANK() の次の数値は 2 ではなく 3 になります。

構文：

```SQL
RANK() OVER (PARTITION BY partition_by_clause ORDER BY order_by_clause)
```

例：

列 x によるランキング：

```SQL
SELECT x, y, RANK() OVER (PARTITION BY x ORDER BY y) AS `rank`
FROM int_t;
```

```plaintext
+---+---+------+
| x | y | rank |
+---+---+------+
| 1 | 1 | 1    |
| 1 | 2 | 2    |
| 1 | 2 | 2    |
| 2 | 1 | 1    |
| 2 | 2 | 2    |
| 2 | 3 | 3    |
| 3 | 1 | 1    |
| 3 | 1 | 1    |
| 3 | 2 | 3    |
+---+---+------+
```

### ROW_NUMBER()

ROW_NUMBER() は、パーティションの各行に対して 1 から始まる連続する整数を返します。RANK() や DENSE_RANK() と異なり、ROW_NUMBER() によって返される値は**繰り返されず、間隔もなく**、**連続してインクリメントされます**。

構文：

```SQL
ROW_NUMBER() OVER (PARTITION BY partition_by_clause ORDER BY order_by_clause)
```

例：

```SQL
SELECT x, y, ROW_NUMBER() OVER (PARTITION BY x ORDER BY y) AS `rank`
FROM int_t;
```

```plaintext
+---+---+------+
| x | y | rank |
+---+---+------+
| 1 | 1 | 1    |
| 1 | 2 | 2    |
| 1 | 2 | 3    |
| 2 | 1 | 1    |
| 2 | 2 | 2    |
| 2 | 3 | 3    |
| 3 | 1 | 1    |
| 3 | 1 | 2    |
| 3 | 2 | 3    |
+---+---+------+
```

### QUALIFY

QUALIFY 句はウィンドウ関数の結果をフィルタリングするために使用されます。SELECT ステートメントで QUALIFY 句を使用して列に条件を適用し、結果をフィルタリングすることができます。QUALIFY は集約関数の HAVING 句に類似しています。この機能は v2.5 からサポートされています。

QUALIFY は SELECT ステートメントの記述を簡素化します。

QUALIFY を使用する前の SELECT ステートメントは以下のようになります：

```SQL
SELECT *
FROM (SELECT DATE,
             PROVINCE_CODE,
             TOTAL_SCORE,
             ROW_NUMBER() OVER (PARTITION BY PROVINCE_CODE ORDER BY TOTAL_SCORE) AS SCORE_ROWNUMBER
      FROM example_table) T1
WHERE T1.SCORE_ROWNUMBER = 1;
```

QUALIFY を使用した後のステートメントは以下のように短縮されます：

```SQL
SELECT DATE, PROVINCE_CODE, TOTAL_SCORE
FROM example_table 
QUALIFY ROW_NUMBER() OVER (PARTITION BY PROVINCE_CODE ORDER BY TOTAL_SCORE) = 1;
```

QUALIFY は、ROW_NUMBER()、RANK()、DENSE_RANK() の 3 つのウィンドウ関数のみをサポートします。

**構文：**

```SQL
SELECT <column_list>
FROM <data_source>
[GROUP BY ...]
[HAVING ...]
QUALIFY <window_function>
[ ... ]
```

**パラメーター：**

`<column_list>`: データを取得したい列。

`<data_source>`: データソースは通常テーブルです。

`<window_function>`: QUALIFY 句にはウィンドウ関数のみが続けられます。ROW_NUMBER()、RANK()、DENSE_RANK() を含みます。

**例：**

```SQL
-- テーブルを作成します。
CREATE TABLE sales_record (
   city_id INT,
   item STRING,
   sales INT
) DISTRIBUTED BY HASH(city_id);

-- テーブルにデータを挿入します。
INSERT INTO sales_record VALUES
(1, 'fruit', 95),
(2, 'drinks', 70),
(3, 'fruit', 87),
(4, 'drinks', 98);

-- テーブルからデータをクエリします。
SELECT * FROM sales_record ORDER BY city_id;
+---------+--------+-------+
| city_id | item   | sales |
+---------+--------+-------+
|       1 | fruit  |    95 |
|       2 | drinks |    70 |
|       3 | fruit  |    87 |
|       4 | drinks |    98 |
+---------+--------+-------+
```

例 1: 行番号が 1 より大きいレコードをテーブルから取得します。

```SQL
SELECT city_id, item, sales
FROM sales_record
QUALIFY ROW_NUMBER() OVER (ORDER BY city_id) > 1;
+---------+--------+-------+
| city_id | item   | sales |
+---------+--------+-------+
|       2 | drinks |    70 |
|       3 | fruit  |    87 |
|       4 | drinks |    98 |
+---------+--------+-------+
```

例 2: テーブルの各パーティションから行番号が 1 のレコードを取得します。テーブルは `item` によって 2 つのパーティションに分けられ、各パーティションの最初の行が返されます。

```SQL
SELECT city_id, item, sales
FROM sales_record 
QUALIFY ROW_NUMBER() OVER (PARTITION BY item ORDER BY city_id) = 1
ORDER BY city_id;
+---------+--------+-------+
| city_id | item   | sales |
+---------+--------+-------+
|       1 | fruit  |    95 |
|       2 | drinks |    70 |
+---------+--------+-------+
2 rows in set (0.01 sec)
```

例 3: 各パーティションから売上ランクが 1 位のレコードを取得します。テーブルは `item` によって 2 つのパーティションに分けられ、各パーティションで売上が最も高い行が返されます。

```SQL
SELECT city_id, item, sales
FROM sales_record
QUALIFY RANK() OVER (PARTITION BY item ORDER BY sales DESC) = 1
ORDER BY city_id;
+---------+--------+-------+
| city_id | item   | sales |
+---------+--------+-------+
|       1 | fruit  |    95 |
|       4 | drinks |    98 |
+---------+--------+-------+
```

**使用上の注意：**

QUALIFY を含むクエリ内の句の実行順序は以下の順に評価されます：

> 1. FROM
> 2. WHERE
> 3. GROUP BY
> 4. HAVING
> 5. WINDOW
> 6. QUALIFY
> 7. DISTINCT
> 8. ORDER BY
> 9. LIMIT

### SUM()

構文：

```SQL
SUM(expr) [OVER (analytic_clause)]
```

例：

プロパティによるグループ化し、グループ内の**現在、前、および後の行の合計を計算します**。

```SQL
SELECT x, property,
    SUM(x)
        OVER (
            PARTITION BY property
            ORDER BY x
            ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
        ) AS 'moving_total'
FROM int_t WHERE property IN ('odd', 'even');
```

```plaintext
+----+----------+--------------+
| x  | property | moving_total |
+----+----------+--------------+
|  2 | even     |            6 |
|  4 | even     |           12 |
|  6 | even     |           18 |
|  8 | even     |           24 |
| 10 | even     |           18 |
|  1 | odd      |            4 |
|  3 | odd      |            9 |
|  5 | odd      |           15 |
|  7 | odd      |           21 |
+----+----------+--------------+
```

### VARIANCE, VAR_POP, VARIANCE_POP

式の母集団分散を返します。VAR_POP と VARIANCE_POP は VARIANCE の別名です。これらの関数は、v2.5.10 以降、ウィンドウ関数として使用できます。

**構文：**

```SQL
VARIANCE(expr) OVER ([partition_by_clause] [order_by_clause] [window_clause])
```

> **注記**
>
> 2.5.13、3.0.7、3.1.4 以降、このウィンドウ関数は ORDER BY 句とウィンドウ句をサポートしています。

**パラメーター：**

`expr` がテーブルの列である場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、または DECIMAL に評価される必要があります。

**例：**

テーブル `agg` に次のデータがあるとします：

```plaintext
mysql> SELECT * FROM agg;
+------+-------+-------+
| no   | k     | v     |
+------+-------+-------+
|    1 | 10.00 |  NULL |
|    2 | 10.00 | 11.00 |
|    2 | 20.00 | 22.00 |
|    2 | 25.00 |  NULL |
|    2 | 30.00 | 35.00 |
+------+-------+-------+
```
VARIANCE() 関数を使用します。

```plaintext
mysql> select variance(k) over (partition by no) FROM agg;
+-------------------------------------+
| variance(k) OVER (PARTITION BY no ) |
+-------------------------------------+
|                                   0 |
|                             54.6875 |
|                             54.6875 |
|                             54.6875 |
|                             54.6875 |
+-------------------------------------+

mysql> select variance(k) over(
    partition by no
    order by k
    rows between unbounded preceding and 1 following) AS window_test
FROM agg order by no,k;
+-------------------+
| window_test       |
+-------------------+
|                 0 |
|                25 |
| 38.88888888888889 |
|           54.6875 |
|           54.6875 |
+-------------------+
```

### VAR_SAMP、VARIANCE_SAMP

式の標本分散を返します。これらの関数は、v2.5.10以降、ウィンドウ関数として使用できます。

**構文：**

```sql
VAR_SAMP(expr) OVER([partition_by_clause] [order_by_clause] [window_clause])
```

> **注記**
>
> 2.5.13、3.0.7、3.1.4 以降、このウィンドウ関数は ORDER BY 句と Window 句をサポートします。

**パラメーター：**

`expr` がテーブル列の場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、または DECIMAL に評価される必要があります。

**例：**

テーブル `agg` に次のデータがあるとします。

```plaintext
mysql> select * from agg;
+------+-------+-------+
| no   | k     | v     |
+------+-------+-------+
|    1 | 10.00 |  NULL |
|    2 | 10.00 | 11.00 |
|    2 | 20.00 | 22.00 |
|    2 | 25.00 |  NULL |
|    2 | 30.00 | 35.00 |
+------+-------+-------+
```

VAR_SAMP() ウィンドウ関数を使用します。

```plaintext
mysql> select VAR_SAMP(k) over (partition by no) FROM agg;
+-------------------------------------+
| var_samp(k) OVER (PARTITION BY no ) |
+-------------------------------------+
|                                   0 |
|                   72.91666666666667 |
|                   72.91666666666667 |
|                   72.91666666666667 |
|                   72.91666666666667 |
+-------------------------------------+

mysql> select VAR_SAMP(k) over(
    partition by no
    order by k
    rows between unbounded preceding and 1 following) AS window_test
FROM agg order by no,k;
+--------------------+
| window_test        |
+--------------------+
|                  0 |
|                 50 |
| 58.333333333333336 |
|  72.91666666666667 |
|  72.91666666666667 |
+--------------------+
```

### STD、STDDEV、STDDEV_POP

式の標準偏差を返します。これらの関数は、v2.5.10以降、ウィンドウ関数として使用できます。

**構文：**

```sql
STD(expr) OVER([partition_by_clause] [order_by_clause] [window_clause])
```

> **注記**
>
> 2.5.13、3.0.7、3.1.4 以降、このウィンドウ関数は ORDER BY 句と Window 句をサポートします。

**パラメーター：**

`expr` がテーブル列の場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、または DECIMAL に評価される必要があります。

**例：**

テーブル `agg` に次のデータがあるとします。

```plaintext
mysql> select * from agg;
+------+-------+-------+
| no   | k     | v     |
+------+-------+-------+
|    1 | 10.00 |  NULL |
|    2 | 10.00 | 11.00 |
|    2 | 20.00 | 22.00 |
|    2 | 25.00 |  NULL |
|    2 | 30.00 | 35.00 |
+------+-------+-------+
```

STD() ウィンドウ関数を使用します。

```plaintext
mysql> select STD(k) over (partition by no) FROM agg;
+--------------------------------+
| std(k) OVER (PARTITION BY no ) |
+--------------------------------+
|                              0 |
|               7.39509972887452 |
|               7.39509972887452 |
|               7.39509972887452 |
|               7.39509972887452 |
+--------------------------------+

mysql> select std(k) over (
    partition by no
    order by k
    rows between unbounded preceding and 1 following) AS window_test
FROM agg order by no,k;
+-------------------+
| window_test       |
+-------------------+
|                 0 |
|                 5 |
| 6.236095644623236 |
|  7.39509972887452 |
|  7.39509972887452 |
+-------------------+
```

### STDDEV_SAMP

式の標本標準偏差を返します。この関数は v2.5.10 以降、ウィンドウ関数として使用できます。

**構文：**

```sql
STDDEV_SAMP(expr) OVER([partition_by_clause] [order_by_clause] [window_clause])
```

> **注記**
>
> 2.5.13、3.0.7、3.1.4 以降、このウィンドウ関数は ORDER BY 句と Window 句をサポートします。

**パラメーター：**

`expr` がテーブル列の場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、または DECIMAL に評価される必要があります。

**例：**

テーブル `agg` に次のデータがあるとします。

```plaintext
mysql> select * from agg;
+------+-------+-------+
| no   | k     | v     |
+------+-------+-------+
|    1 | 10.00 |  NULL |
|    2 | 10.00 | 11.00 |
|    2 | 20.00 | 22.00 |
|    2 | 25.00 |  NULL |
|    2 | 30.00 | 35.00 |
+------+-------+-------+
```

STDDEV_SAMP() ウィンドウ関数を使用します。

```plaintext
mysql> select STDDEV_SAMP(k) over (partition by no) FROM agg;
+----------------------------------------+
| stddev_samp(k) OVER (PARTITION BY no ) |
+----------------------------------------+
|                                      0 |
|                      8.539125638299666 |
|                      8.539125638299666 |
|                      8.539125638299666 |
|                      8.539125638299666 |
+----------------------------------------+

mysql> select STDDEV_SAMP(k) over (
    partition by no
    order by k
    rows between unbounded preceding and 1 following) AS window_test
FROM agg order by no,k;
+--------------------+
| window_test        |
+--------------------+
|                  0 |
| 7.0710678118654755 |
|  7.637626158259733 |
|  8.539125638299666 |
|  8.539125638299666 |
+--------------------+
```

### COVAR_SAMP

2 つの式の標本共分散を返します。この関数は v2.5.10 からサポートされています。これも集計関数です。

**構文：**

```sql
COVAR_SAMP(expr1, expr2) OVER([partition_by_clause] [order_by_clause] [window_clause])
```

> **注記**
>
> 2.5.13、3.0.7、3.1.4 以降、このウィンドウ関数は ORDER BY 句と Window 句をサポートします。

**パラメーター：**

`expr` がテーブル列の場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、または DECIMAL に評価される必要があります。

**例：**

テーブル `agg` に次のデータがあるとします。

```plaintext
mysql> select * from agg;
+------+-------+-------+
| no   | k     | v     |
+------+-------+-------+
|    1 | 10.00 |  NULL |
|    2 | 10.00 | 11.00 |
|    2 | 20.00 | 22.00 |
|    2 | 25.00 |  NULL |
|    2 | 30.00 | 35.00 |
+------+-------+-------+
```

COVAR_SAMP() ウィンドウ関数を使用します。

```plaintext
mysql> select COVAR_SAMP(k, v) over (partition by no) FROM agg;
+------------------------------------------+
| covar_samp(k, v) OVER (PARTITION BY no ) |
+------------------------------------------+
|                                     NULL |
|                       119.99999999999999 |
|                       119.99999999999999 |
|                       119.99999999999999 |
|                       119.99999999999999 |
+------------------------------------------+

mysql> select COVAR_SAMP(k,v) over (
    partition by no
    order by k
    rows between unbounded preceding and 1 following) AS window_test
FROM agg order by no,k;
+--------------------+
| window_test        |
+--------------------+
|               NULL |
|                 55 |
|                 55 |
| 119.99999999999999 |
| 119.99999999999999 |
+--------------------+
```

### COVAR_POP

2 つの式の母集団共分散を返します。この関数は v2.5.10 からサポートされています。これも集計関数です。

**構文：**

```sql
COVAR_POP(expr1, expr2) OVER([partition_by_clause] [order_by_clause] [window_clause])
```

> **注記**
>
> 2.5.13、3.0.7、3.1.4 以降、このウィンドウ関数は ORDER BY 句と Window 句をサポートします。

**パラメーター：**

`expr` がテーブル列の場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、または DECIMAL に評価される必要があります。

**例：**

テーブル `agg` に次のデータがあるとします。

```plaintext
mysql> select * from agg;
+------+-------+-------+
| no   | k     | v     |
+------+-------+-------+
|    1 | 10.00 |  NULL |
|    2 | 10.00 | 11.00 |
|    2 | 20.00 | 22.00 |
|    2 | 25.00 |  NULL |
|    2 | 30.00 | 35.00 |
+------+-------+-------+
```

COVAR_POP() ウィンドウ関数を使用します。

```plaintext
mysql> select COVAR_POP(k, v) over (partition by no) FROM agg;
+-----------------------------------------+
| covar_pop(k, v) OVER (PARTITION BY no ) |
+-----------------------------------------+
|                                    NULL |
|                       79.99999999999999 |
|                       79.99999999999999 |
|                       79.99999999999999 |
|                       79.99999999999999 |
+-----------------------------------------+

mysql> select COVAR_POP(k,v) over (
    partition by no
    order by k
    rows between unbounded preceding and 1 following) AS window_test
FROM agg order by no,k;
+-------------------+
| window_test       |
+-------------------+
|              NULL |
|              27.5 |
|              27.5 |
| 79.99999999999999 |
| 79.99999999999999 |
+-------------------+
```

### CORR

2 つの式間のピアソン相関係数を返します。この関数は v2.5.10 からサポートされています。これも集計関数です。

**構文：**

```sql
CORR(expr1, expr2) OVER([partition_by_clause] [order_by_clause] [window_clause])
```
CORR(expr1, expr2) OVER([partition_by_clause] [order_by_clause] [window_clause])
```

> **注記**
>
> 2.5.13、3.0.7、3.1.4 以降、このウィンドウ関数は ORDER BY 句と WINDOW 句をサポートしています。

**パラメーター：**

`expr` がテーブルのカラムである場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、または DECIMAL に評価される必要があります。

**例：**

テーブル `agg` に次のデータがあるとします。

```plaintext
mysql> select * from agg;
+------+-------+-------+
| no   | k     | v     |
+------+-------+-------+
|    1 | 10.00 |  NULL |
|    2 | 10.00 | 11.00 |
|    2 | 20.00 | 22.00 |
|    2 | 25.00 |  NULL |
|    2 | 30.00 | 35.00 |
+------+-------+-------+
```

ウィンドウ関数 CORR() を使用します。

```plaintext
mysql> select CORR(k, v) over (partition by no) FROM agg;
+------------------------------------+
| corr(k, v) OVER (PARTITION BY no ) |
+------------------------------------+
|                               NULL |
|                 0.9988445981121532 |
|                 0.9988445981121532 |
|                 0.9988445981121532 |
|                 0.9988445981121532 |
+------------------------------------+

mysql> select CORR(k,v) over (
    partition by no
    order by k
    rows between unbounded preceding and 1 following) AS window_test
FROM agg order by no,k;
+--------------------+
| window_test        |
+--------------------+
|               NULL |
|                  1 |
|                  1 |
| 0.9988445981121532 |
| 0.9988445981121532 |
+--------------------+
```

