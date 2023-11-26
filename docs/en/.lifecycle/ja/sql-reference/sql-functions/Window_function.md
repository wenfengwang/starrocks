# ウィンドウ関数

## 背景

ウィンドウ関数は、特別な組み込み関数の一種です。集計関数と同様に、複数の入力行に対して計算を行い、単一のデータ値を取得します。違いは、ウィンドウ関数は「GROUP BY」メソッドではなく、特定のウィンドウ内で入力データを処理することです。各ウィンドウ内のデータは、over()句を使用してソートやグループ化することができます。ウィンドウ関数は、グループごとに1つの値を計算するのではなく、**各行ごとに個別の値を計算**します。この柔軟性により、ユーザーは選択句に追加の列を追加し、結果セットをさらにフィルタリングすることができます。ウィンドウ関数は、クエリの最後、つまり`join`、`where`、`group by`の操作が実行された後に効果を発揮します。ウィンドウ関数は、大規模なデータに対してトレンドの分析、外れ値の計算、バケット分析などを行うためによく使用されます。

## 使用法

ウィンドウ関数の構文:

```SQL
function(args) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
partition_by_clause ::= PARTITION BY expr [, expr ...]
order_by_clause ::= ORDER BY expr [ASC | DESC] [, expr [ASC | DESC] ...]
```

### 関数

現在サポートされている関数は以下のとおりです:

* MIN(), MAX(), COUNT(), SUM(), AVG()
* FIRST_VALUE(), LAST_VALUE(), LEAD(), LAG()
* ROW_NUMBER(), RANK(), DENSE_RANK()
* CUME_DIST(), PERCENT_RANK(), QUALIFY()
* NTILE()
* VARIANCE(), VAR_SAMP(), STD(), STDDEV_SAMP(), COVAR_SAMP(), COVAR_POP(), CORR()

### PARTITION BY句

PARTITION BY句は、GROUP BYと似ています。指定された1つ以上の列によって入力行をグループ化します。同じ値を持つ行は一緒にグループ化されます。

### ORDER BY句

`Order By`句は、基本的には外部の`Order By`と同じです。入力行の順序を定義します。`Partition By`が指定されている場合、`Order By`は各パーティションのグループ内の順序を定義します。唯一の違いは、`OVER`句の`Order By n`（nは正の整数）は何も操作を行わず、外部の`Order By`のnはn番目の列でソートすることを示します。

例:

この例では、イベントテーブルの`date_and_time`列でソートされた1、2、3などの値を持つid列を選択リストに追加しています。

```SQL
SELECT row_number() OVER (ORDER BY date_and_time) AS id,
    c1, c2, c3, c4
FROM events;
```

### Window句

Window句は、操作の範囲（現在の行を基準にした前後の行）を指定するために使用されます。以下の構文をサポートしています: AVG()、COUNT()、FIRST_VALUE()、LAST_VALUE()、SUM()。MAX()とMIN()の場合、Window句は`UNBOUNDED PRECEDING`から開始することができます。

構文:

```SQL
ROWS BETWEEN [ { m | UNBOUNDED } PRECEDING | CURRENT ROW] [ AND [CURRENT ROW | { UNBOUNDED | n } FOLLOWING] ]
```

例:

以下の株価データがあるとします。株式シンボルはJDRで、終値は日次の終値です。

```SQL
create table stock_ticker (
    stock_symbol string,
    closing_price decimal(8,2),
    closing_date timestamp);

-- ...データをロード...

select *
from stock_ticker
order by stock_symbol, closing_date
```

元のデータは以下のように表示されます:

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

このクエリでは、ウィンドウ関数を使用して、移動平均列を生成しています。移動平均列の値は、3日間（前日、当日、翌日）の平均株価です。最初の日は前日の値を持たないため、最後の日は翌日の値を持たないため、これらの2つの行は2日間の平均値のみを計算します。ここでは`Partition By`は効果を持ちませんが、すべてのデータがJDRデータであるためです。ただし、他の株式情報がある場合、`Partition By`は各パーティション内でウィンドウ関数が操作されることを保証します。

```SQL
select stock_symbol, closing_date, closing_price,
    avg(closing_price)
        over (partition by stock_symbol
              order by closing_date
              rows between 1 preceding and 1 following
        ) as moving_average
from stock_ticker;
```

以下のデータが得られます:

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

## 関数の例

このセクションでは、StarRocksでサポートされているウィンドウ関数について説明します。

### AVG()

構文:

```SQL
AVG(expr) [OVER (*analytic_clause*)]
```

例:

現在の行とその前後の各行のxの平均を計算します。

```SQL
select x, property,
    avg(x)
        over (
            partition by property
            order by x
            rows between 1 preceding and 1 following
        ) as 'moving average'
from int_t
where property in ('odd','even');
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

現在の行から最初の行までのxの出現回数をカウントします。

```SQL
select x, property,
    count(x)
        over (
            partition by property
            order by x
            rows between unbounded preceding and current row
        ) as 'cumulative total'
from int_t where property in ('odd','even');
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

CUME_DIST()関数は、パーティション内の値の累積分布を計算し、現在の行の値をパーセントで示します。0から1までの範囲で、パーセンタイルの計算やデータ分布の分析に役立ちます。

構文:

```SQL
CUME_DIST() OVER (partition_by_clause order_by_clause)
```

**この関数は、パーティションの行を所望の順序に並べ替えるためにORDER BYと共に使用する必要があります。ORDER BYが指定されていない場合、すべての行は同等であり、値N/N = 1（Nはパーティションのサイズ）が返されます。**

CUME_DIST()はNULL値を含み、それらを最小値として扱います。

以下の例は、列yの累積分布をxのグループごとに示しています。

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
| 2 | 2 | 0.6666666666666667 |
| 2 | 3 |                  1 |
| 3 | 1 | 0.6666666666666667 |
| 3 | 1 | 0.6666666666666667 |
| 3 | 2 |                  1 |
+---+---+--------------------+
```

### DENSE_RANK()

DENSE_RANK()関数は、ランキングを表すために使用されます。RANK()とは異なり、DENSE_RANK()**は空き番号を持ちません**。たとえば、2つのタイの1がある場合、DENSE_RANK()の3番目の数値は2のままですが、RANK()の3番目の数値は3になります。

構文:

```SQL
DENSE_RANK() OVER(partition_by_clause order_by_clause)
```

以下の例は、プロパティ列のグループ化に基づいて列xのランキングを示しています。

```SQL
select x, y,
    dense_rank()
        over (
            partition by x
            order by y
        ) as `rank`
from int_t;
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

NTILE()関数は、パーティション内のソートされた行を指定された`num_buckets`の数で均等に分割し、各バケットに分割された行を格納し、1から`num_buckets`までのバケット番号を返します。

バケットのサイズについて:

* 行数が指定された`num_buckets`で正確に分割できる場合、すべてのバケットのサイズは同じになります。
* 行数が指定された`num_buckets`で正確に分割できない場合、2つの異なるサイズのバケットがあります。サイズの差は1です。行数が多いバケットが行数が少ないバケットよりも前に表示されます。

構文:

```SQL
NTILE (num_buckets) OVER (partition_by_clause order_by_clause)
```

`num_buckets`: 作成するバケットの数。値は定数の正の整数でなければなりません。最大値は`2^63 - 1`です。

NTILE()関数ではWindow句は使用できません。

NTILE()関数はBIGINT型のデータを返します。

例:

以下の例では、パーティション内のすべての行を2つのバケットに分割します。

```sql
select id, x, y,
    ntile(2)
        over (
            partition by x
            order by y
        ) as bucket_id
from t1;
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

上記の例では、`num_buckets`が`2`の場合:

* 1から6の行は最初のパーティションに分類されます。1から3の行は最初のバケットに格納され、4から6の行は2番目のバケットに格納されます。
* 7から9の行は2番目のパーティションに分類されます。7と8の行は最初のバケットに格納され、9の行は2番目のバケットに格納されます。
* 10の行は3番目のパーティションに分類され、最初のバケットに格納されます。

<br/>

### FIRST_VALUE()

FIRST_VALUE()は、ウィンドウ範囲の**最初の**値を返します。

構文:

```SQL
FIRST_VALUE(expr [IGNORE NULLS]) OVER(partition_by_clause order_by_clause [window_clause])
```

`IGNORE NULLS`はv2.5.0からサポートされています。NULL値を`expr`の計算から除外するかどうかを決定するために使用されます。デフォルトでは、NULL値は含まれ、フィルタリングされた結果の最初の値がNULLの場合はNULLが返されます。IGNORE NULLSを指定すると、フィルタリングされた結果の最初の非NULL値が返されます。すべての値がNULLの場合、IGNORE NULLSを指定していてもNULLが返されます。

例:

以下のデータがあるとします:

```SQL
 select name, country, greeting
 from mail_merge;
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

FIRST_VALUE()を使用して、国ごとのグループ化に基づいて最初の挨拶値を返します。

```SQL
select country, name,
    first_value(greeting)
        over (
            partition by country
            order by name, greeting
        ) as greeting
from mail_merge;
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

`offset`行前の現在の行の値を返します。この関数は、行間の値の比較やデータのフィルタリングによく使用されます。

`lag()`でクエリできるデータの型は、[lag()](#lag)でサポートされているものと同じです。

構文:

```SQL
LAG(expr [IGNORE NULLS] [, offset[, default]])
OVER([<partition_by_clause>] [<order_by_clause>])
```

パラメータ:

* `expr`: 計算したいフィールドです。
* `offset`: オフセットです。**正の整数**でなければなりません。このパラメータが指定されていない場合、デフォルトは1です。
* `default`: マッチする行が見つからない場合に返されるデフォルト値です。このパラメータが指定されていない場合、デフォルトはNULLです。`default`は、`expr`と互換性のある型の任意の式をサポートします。
* `IGNORE NULLS`はv3.0からサポートされています。NULL値を`expr`の計算に含めるかどうかを決定するために使用されます。デフォルトでは、NULL値は`offset`行が数えられるときに含まれます。つまり、宛先行の値がNULLの場合はNULLが返されます。IGNORE NULLSを指定すると、`offset`行が数えられるときにNULL値は無視され、`offset`個の非NULL値の検索が続行されます。`offset`個の非NULL値が見つからない場合、NULLまたは（指定されている場合）`default`が返されます。

例 1: IGNORE NULLSが指定されていない場合

テーブルを作成し、値を挿入します:

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

このテーブルからデータをクエリし、`offset`が2であるため、前の2行をたどります。`default`は0で、一致する行が見つからない場合は0が返されます。

出力:

```plaintext
SELECT col_1, col_2, LAG(col_2,2,0) OVER (ORDER BY col_1) 
FROM test_tbl ORDER BY col_1;
+-------+-------+---------------------------------------------+
| col_1 | col_2 | lag(col_2, 2, 0) OVER (ORDER BY col_1 ASC ) |
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

最初の2行では、前の2行が存在せず、デフォルト値0が返されます。

NULLの場合、3行目では前の2行の値はNULLであり、NULLが許可されているため、NULLが返されます。

例 2: IGNORE NULLSが指定されている場合

前のテーブルとパラメータ設定を使用します。

```SQL
SELECT col_1, col_2, LAG(col_2 IGNORE NULLS,2,0) OVER (ORDER BY col_1) 
FROM test_tbl ORDER BY col_1;
+-------+-------+---------------------------------------------+
| col_1 | col_2 | lag(col_2, 2, 0) OVER (ORDER BY col_1 ASC ) |
+-------+-------+---------------------------------------------+
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

1から4の行では、前の2行の非NULL値を各行の前に見つけることができず、デフォルト値0が返されます。

7の値では、2行前の値はNULLであり、NULLが無視されるため、非NULL値の検索が続行され、4行目の2が返されます。

### LAST_VALUE()

LAST_VALUE()は、ウィンドウ範囲の**最後の**値を返します。FIRST_VALUE()の逆です。

構文:

```SQL
LAST_VALUE(expr [IGNORE NULLS]) OVER(partition_by_clause order_by_clause [window_clause])
```

`IGNORE NULLS`はv2.5.0からサポートされています。NULL値を`expr`の計算から除外するかどうかを決定するために使用されます。デフォルトでは、NULL値は含まれ、フィルタリングされた結果の最後の値がNULLの場合はNULLが返されます。IGNORE NULLSを指定すると、フィルタリングされた結果の最後の非NULL値が返されます。すべての値がNULLの場合、IGNORE NULLSを指定していてもNULLが返されます。

上記の例のデータを使用します:

```SQL
select country, name,
    last_value(greeting)
        over (
            partition by country
            order by name, greeting
        ) as greeting
from mail_merge;
```

```plaintext
+---------+---------+--------------+
| country | name    | greeting     |
+---------+---------+--------------+
| Germany | Boris   | Guten morgen |
| Germany | Michael | Guten morgen |
| Sweden  | Bjorn   | Tja          |
| Sweden  | Mats    | Tja          |
| USA     | John    | Hello        |
| USA     | Pete    | Hello        |
+---------+---------+--------------+
```

### LEAD()

`offset`行後の現在の行の値を返します。この関数は、行間の値の比較やデータのフィルタリングによく使用されます。

`lead()`でクエリできるデータの型は、[lag()](#lag)でサポートされているものと同じです。

構文

```sql
LEAD(expr [IGNORE NULLS] [, offset[, default]])
OVER([<partition_by_clause>] [<order_by_clause>])
```

パラメータ:

* `expr`: 計算したいフィールドです。
* `offset`: オフセットです。正の整数でなければなりません。このパラメータが指定されていない場合、デフォルトは1です。
* `default`: マッチする行が見つからなかった場合に返されるデフォルト値です。このパラメータが指定されていない場合、デフォルトはNULLです。`default`は、`expr`と互換性のある型の任意の式をサポートしています。
* `IGNORE NULLS`はv3.0からサポートされています。これは、`expr`のNULL値が結果に含まれるかどうかを決定するために使用されます。デフォルトでは、NULL値は`offset`行が数えられるときに含まれます。つまり、宛先行の値がNULLの場合、NULLが返されます。例1を参照してください。`IGNORE NULLS`を指定すると、`offset`行が数えられるときにNULL値が無視され、システムは引き続き`offset`個の非NULL値を検索します。`offset`個の非NULL値が見つからない場合、NULLまたは指定された`default`が返されます。例2を参照してください。

例1: IGNORE NULLSが指定されていない場合

テーブルを作成し、値を挿入します:

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

このテーブルからデータをクエリし、`offset`が2である場合、後続の2行をトラバースします。マッチする行が見つからない場合、デフォルト値として0が返されます。

出力:

```plaintext
SELECT col_1, col_2, LEAD(col_2,2,0) OVER (ORDER BY col_1) 
FROM test_tbl ORDER BY col_1;
+-------+-------+----------------------------------------------+
| col_1 | col_2 | lead(col_2, 2, 0) OVER (ORDER BY col_1 ASC ) |
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

最初の行では、2つ先の値はNULLであり、NULLが返されます。最後の2行では、後続の2つの行が存在せず、デフォルト値の0が返されます。

例2: IGNORE NULLSが指定されている場合

前のテーブルとパラメータ設定を使用します。

```SQL
SELECT col_1, col_2, LEAD(col_2 IGNORE NULLS,2,0) OVER (ORDER BY col_1) 
FROM test_tbl ORDER BY col_1;
+-------+-------+----------------------------------------------+
| col_1 | col_2 | lead(col_2, 2, 0) OVER (ORDER BY col_1 ASC ) |
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

行7から10では、システムは後続の2つの非NULL値を見つけることができず、デフォルト値の0が返されます。

最初の行では、2つ先の値はNULLですが、IGNORE NULLSが指定されているため、NULLは無視されます。システムは引き続き2番目の非NULL値を検索し、4行目の2が返されます。

### MAX()

現在のウィンドウ内の指定された行の最大値を返します。

構文

```SQL
MAX(expr) [OVER (analytic_clause)]
```

例:

現在の行から次の行までの行の最大値を計算します。

```SQL
select x, property,
    max(x)
        over (
            order by property, x
            rows between unbounded preceding and 1 following
        ) as 'local maximum'
from int_t
where property in ('prime','square');
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

StarRocks 2.4以降、`rows between n preceding and n following`という行範囲を指定できるようになりました。これにより、現在の行の前の`n`行と現在の行の後の`n`行をキャプチャできます。

例文:

```sql
select x, property,
    max(x)
        over (
            order by property, x
            rows between 3 preceding and 2 following) as 'local maximum'
from int_t
where property in ('prime','square');
```

### MIN()

現在のウィンドウ内の指定された行の最小値を返します。

構文:

```SQL
MIN(expr) [OVER (analytic_clause)]
```

例:

現在の行から次の行までの行の最小値を計算します。

```SQL
select x, property,
    min(x)
        over (
            order by property, x desc
            rows between unbounded preceding and 1 following
        ) as 'local minimum'
from int_t
where property in ('prime','square');
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

StarRocks 2.4以降、`rows between n preceding and n following`という行範囲を指定できるようになりました。これにより、現在の行の前の`n`行と現在の行の後の`n`行をキャプチャできます。

例文:

```sql
select x, property,
    min(x)
    over (
          order by property, x desc
          rows between 3 preceding and 2 following) as 'local minimum'
from int_t
where property in ('prime','square');
```

### PERCENT_RANK()

PERCENT_RANK()関数は、結果セット内の行の相対ランクをパーセンテージで計算します。最も高い値を除く、パーティション値より小さい値のパーセンテージを返します。返される値の範囲は0から1です。この関数は、パーセンタイルの計算やデータ分布の分析に役立ちます。

PERCENT_RANK()関数は、次の式を使用して計算されます。ここで、rankは行のランクを表し、rowsはパーティションの行数を表します。

```plaintext
(rank - 1) / (rows - 1)
```

構文:

```SQL
PERCENT_RANK() OVER (partition_by_clause order_by_clause)
```

**この関数は、パーティション行を所望の順序に並べ替えるためにORDER BYとともに使用する必要があります。ORDER BYがない場合、すべての行が同等であり、値は(1 - 1)/(N - 1) = 0となります。**

以下の例は、列yの相対ランクを列xの各グループ内で示しています。

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

RANK()関数は、ランキングを表します。DENSE_RANK()とは異なり、RANK()は**空き番号**として表示されます。たとえば、2つのタイの1が現れる場合、RANK()の3番目の数値は2ではなく3になります。

構文:

```SQL
RANK() OVER(partition_by_clause order_by_clause)
```

例:

列xに基づいてランキングを行います。

```SQL
select x, y, rank() over(partition by x order by y) as `rank`
from int_t;
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

パーティションごとに1から連続的に増加する整数を返します。RANK()やDENSE_RANK()とは異なり、ROW_NUMBER()によって返される値は繰り返されず、ギャップもありません。

構文:

```SQL
ROW_NUMBER() OVER(partition_by_clause order_by_clause)
```

例:

```SQL
select x, y, row_number() over(partition by x order by y) as `rank`
from int_t;
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

### QUALIFY()

QUALIFY句は、SELECT文の結果をフィルタリングするために使用されます。SELECT文では、QUALIFY句を使用して列に条件を適用して結果をフィルタリングできます。QUALIFYは、集計関数のHAVING句に類似しています。この関数はv2.5からサポートされています。

QUALIFYを使用する前のSELECT文は次のようになります:

```SQL
SELECT *
FROM (SELECT DATE,
             PROVINCE_CODE,
             TOTAL_SCORE,
             ROW_NUMBER() OVER(PARTITION BY PROVINCE_CODE ORDER BY TOTAL_SCORE) AS SCORE_ROWNUMBER
      FROM example_table) T1
WHERE T1.SCORE_ROWNUMBER = 1;
```

QUALIFYを使用すると、次のように短縮されます:

```SQL
SELECT DATE, PROVINCE_CODE, TOTAL_SCORE
FROM example_table 
QUALIFY ROW_NUMBER() OVER(PARTITION BY PROVINCE_CODE ORDER BY TOTAL_SCORE) = 1;
```

QUALIFYは、ROW_NUMBER()、RANK()、DENSE_RANK()の3つのウィンドウ関数のみをサポートしています。

**構文:**

```SQL
SELECT <column_list>
FROM <data_source>
[GROUP BY ...]
[HAVING ...]
QUALIFY <window_function>
[ ... ]
```

**パラメータ:**

`<column_list>`: データを取得するための列。

`<data_source>`: データソースは通常テーブルです。

`<window_function>`: `QUALIFY`句の後には、ROW_NUMBER()、RANK()、DENSE_RANK()を含むウィンドウ関数のみが続くことができます。

**例:**

```SQL
-- テーブルを作成します。
CREATE TABLE sales_record (
   city_id INT,
   item STRING,
   sales INT
) DISTRIBUTED BY HASH(`city_id`);

-- テーブルにデータを挿入します。
insert into sales_record values
(1,'fruit',95),
(2,'drinks',70),
(3,'fruit',87),
(4,'drinks',98);

-- テーブルからデータをクエリします。
select * from sales_record order by city_id;
+---------+--------+-------+
| city_id | item   | sales |
+---------+--------+-------+
|       1 | fruit  |    95 |
|       2 | drinks |    70 |
|       3 | fruit  |    87 |
|       4 | drinks |    98 |
+---------+--------+-------+
```

例1: テーブルから行番号が1より大きいレコードを取得します。

```SQL
SELECT city_id, item, sales
FROM sales_record
QUALIFY row_number() OVER (ORDER BY city_id) > 1;
+---------+--------+-------+
| city_id | item   | sales |
+---------+--------+-------+
|       2 | drinks |    70 |
|       3 | fruit  |    87 |
|       4 | drinks |    98 |
+---------+--------+-------+
```

例2: テーブルから各パーティションの行番号が1のレコードを取得します。テーブルは`item`によって2つのパーティションに分割され、各パーティションの最初の行が返されます。

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

例3: テーブルから各パーティションの最大売上を持つレコードを取得します。テーブルは`item`によって2つのパーティションに分割され、各パーティションの最大売上の行が返されます。

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

**使用上の注意:**

QUALIFYを含むクエリのクローズは、次の順序で評価されます。

> 1. From
> 2. Where
> 3. Group by
> 4. Having
> 5. Window
> 6. QUALIFY
> 7. Distinct
> 8. Order by
> 9. Limit

### SUM()

構文:

```SQL
SUM(expr) [OVER (analytic_clause)]
```

例:

propertyでグループ化し、グループ内の**現在の行、前の行、後続の行**の合計を計算します。

```SQL
select x, property,
    sum(x)
        over (
            partition by property
            order by x
            rows between 1 preceding and 1 following
        ) as 'moving total'
from int_t where property in ('odd','even');
```

```plaintext
+----+----------+--------------+
| x  | property | moving total |
+----+----------+--------------+
| 2  | even     | 6            |
| 4  | even     | 12           |
| 6  | even     | 18           |
| 8  | even     | 24           |
| 10 | even     | 18           |
| 1  | odd      | 4            |
| 3  | odd      | 9            |
| 5  | odd      | 15           |
| 7  | odd      | 21           |
+----+----------+--------------+
```

### VARIANCE, VAR_POP, VARIANCE_POP

式の母集団分散を返します。VAR_POPおよびVARIANCE_POPはVARIANCEのエイリアスです。これらの関数はv2.5.10以降のウィンドウ関数として使用できます。

**構文:**

```SQL
VARIANCE(expr) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
>
> 2.5.13、3.0.7、3.1.4以降、このウィンドウ関数はORDER BYおよびWindow句をサポートしています。

**パラメータ:**

`expr`がテーブルの列である場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、またはDECIMALに評価される必要があります。

**例:**

テーブル`agg`に次のデータがあるとします:

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

VARIANCE()関数を使用します。

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

### VAR_SAMP, VARIANCE_SAMP

式の標本分散を返します。これらの関数はv2.5.10以降のウィンドウ関数として使用できます。

**構文:**

```sql
VAR_SAMP(expr) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
>
> 2.5.13、3.0.7、3.1.4以降、このウィンドウ関数はORDER BYおよびWindow句をサポートしています。

**パラメータ:**

`expr`がテーブルの列である場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、またはDECIMALに評価される必要があります。

**例:**

テーブル`agg`に次のデータがあるとします:

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

VAR_SAMP()ウィンドウ関数を使用します。

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

### STD, STDDEV, STDDEV_POP

式の標準偏差を返します。これらの関数はv2.5.10以降のウィンドウ関数として使用できます。

**構文:**

```sql
STD(expr) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
>
> 2.5.13、3.0.7、3.1.4以降、このウィンドウ関数はORDER BYおよびWindow句をサポートしています。

**パラメータ:**

`expr`がテーブルの列である場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、またはDECIMALに評価される必要があります。

**例:**

テーブル`agg`に次のデータがあるとします:

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

STD()ウィンドウ関数を使用します。

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

式の標本標準偏差を返します。この関数はv2.5.10以降のウィンドウ関数として使用できます。
**構文:**

```sql
STDDEV_SAMP(expr) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
>
> 2.5.13、3.0.7、3.1.4以降、このウィンドウ関数はORDER BYおよびWindow句をサポートしています。

**パラメータ:**

`expr`がテーブルの列である場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、またはDECIMALに評価される必要があります。

**例:**

テーブル`agg`に次のデータがあるとします:

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

STDDEV_SAMP()ウィンドウ関数を使用します。

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

2つの式のサンプル共分散を返します。この関数はv2.5.10からサポートされています。また、集計関数でもあります。

**構文:**

```sql
COVAR_SAMP(expr1,expr2) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
>
> 2.5.13、3.0.7、3.1.4以降、このウィンドウ関数はORDER BYおよびWindow句をサポートしています。

**パラメータ:**

`expr`がテーブルの列である場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、またはDECIMALに評価される必要があります。

**例:**

テーブル`agg`に次のデータがあるとします:

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

COVAR_SAMP()ウィンドウ関数を使用します。

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

2つの式の母集団共分散を返します。この関数はv2.5.10からサポートされています。また、集計関数でもあります。

**構文:**

```sql
COVAR_POP(expr1, expr2) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
>
> 2.5.13、3.0.7、3.1.4以降、このウィンドウ関数はORDER BYおよびWindow句をサポートしています。

**パラメータ:**

`expr`がテーブルの列である場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、またはDECIMALに評価される必要があります。

**例:**

テーブル`agg`に次のデータがあるとします:

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

COVAR_POP()ウィンドウ関数を使用します。

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

2つの式のピアソン相関係数を返します。この関数はv2.5.10からサポートされています。また、集計関数でもあります。

**構文:**

```sql
CORR(expr1, expr2) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
>
> 2.5.13、3.0.7、3.1.4以降、このウィンドウ関数はORDER BYおよびWindow句をサポートしています。

**パラメータ:**

`expr`がテーブルの列である場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、またはDECIMALに評価される必要があります。

**例:**

テーブル`agg`に次のデータがあるとします:

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

CORR()ウィンドウ関数を使用します。

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
