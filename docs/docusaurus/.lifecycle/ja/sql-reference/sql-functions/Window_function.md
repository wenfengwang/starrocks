---
displayed_sidebar: "Japanese"
---

# ウィンドウ関数

## バックグラウンド

ウィンドウ関数は、特別な組み込み関数の特別なクラスです。 集約関数と同様に、複数の入力行に対して計算を行い、単一のデータ値を取得します。 違いは、ウィンドウ関数が「グループ化」メソッドを使用するのではなく、特定のウィンドウ内で入力データを処理する点です。 各ウィンドウのデータは、over()句を使用してソートおよびグループ化できます。 ウィンドウ関数は、グループごとに1つの値を計算するのではなく、**各行ごとに個別の値を計算**します。 この柔軟性により、ユーザーは選択句に追加の列を追加し、さらに結果セットをフィルタリングできます。 ウィンドウ関数は選択リストと句の最外部位置にのみ現れることができます。 ウィンドウ関数はクエリの最後に効果を発揮します。つまり、`join`、`where`、`group by`操作の後に効果を発揮します。 大規模なデータに対してトレンドを分析し、外れ値を計算し、バケット分析を実行するために、ウィンドウ関数はよく使用されます。

## 使用法

ウィンドウ関数の構文：

```SQL
function(args) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
partition_by_clause ::= PARTITION BY expr [, expr ...]
order_by_clause ::= ORDER BY expr [ASC | DESC] [, expr [ASC | DESC] ...]
```

### 関数

現在サポートされている関数は以下の通りです：

* MIN(), MAX(), COUNT(), SUM(), AVG()
* FIRST_VALUE(), LAST_VALUE(), LEAD(), LAG()
* ROW_NUMBER(), RANK(), DENSE_RANK()
* CUME_DIST(), PERCENT_RANK(), QUALIFY()
* NTILE()
* VARIANCE(), VAR_SAMP(), STD(), STDDEV_SAMP(), COVAR_SAMP(), COVAR_POP(), CORR()

### PARTITION BY句

Partition By句はGroup Byと同様です。 指定した1つまたは複数の列によって入力行をグループ化します。 同じ値を持つ行が一緒にグループ化されます。

### ORDER BY句

`Order By`句は基本的に外部の`Order By`と同じです。 入力行の順序を定義します。 `Partition By`が指定されている場合、`Order By`は各パーティショングループ内の順序を定義します。 唯一の違いは、`OVER`句内の`Order By n`（nは正の整数）は操作なしが相当し、外部の`Order By`の`n`はn番目の列で並べ替えを示します。

例：

次の例は、イベントテーブルの`date_and_time`列でソートされた`id`列を選択リストに追加しています。

```SQL
SELECT row_number() OVER (ORDER BY date_and_time) AS id,
    c1, c2, c3, c4
FROM events;
```

### ウィンドウ句

ウィンドウ句は操作のための行の範囲を指定するために使用されます（現在の行を基準に前後の行）。 AVG()、COUNT()、FIRST_VALUE()、LAST_VALUE()、SUM()の以下の構文をサポートしています。 MAX()およびMIN()の場合、ウィンドウ句は`UNBOUNDED PRECEDING`から始まることができます。

構文：

```SQL
ROWS BETWEEN [ { m | UNBOUNDED } PRECEDING | CURRENT ROW] [ AND [CURRENT ROW | { UNBOUNDED | n } FOLLOWING] ]
```

例：

以下のストックデータがあるとします。株式シンボルはJDRで、終値は日次の終値です。

```SQL
create table stock_ticker (
    stock_symbol string,
    closing_price decimal(8,2),
    closing_date timestamp);

-- ...いくつかのデータをロード...

select *
from stock_ticker
order by stock_symbol, closing_date
```

元のデータは以下の通りです：

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

このクエリは、ウィンドウ関数を使用して、移動平均列を生成します。その値は3日間（前日、当日、翌日）の平均株価です。 最初の日は前日の値を持たないため、最後の日は翌日の値を持たないため、これらの値は2日間の平均値を計算します。 ここで、`Partition By`は効果を持たないため、すべてのデータがJDRデータであるためです。 ただし、他の株情報がある場合、`Partition By`により、ウィンドウ関数が各パーティション内で操作されることが保証されます。

```SQL
select stock_symbol, closing_date, closing_price,
    avg(closing_price)
        over (partition by stock_symbol
              order by closing_date
              rows between 1 preceding and 1 following
        ) as moving_average
from stock_ticker;
```

次のデータが得られます：

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

このセクションでは、StarRocksでサポートされているウィンドウ関数を説明します。

### AVG()

構文：

```SQL
AVG(expr) [OVER (*analytic_clause*)]
```

例：

現在の行とその前後の各行のx平均を計算します。

```SQL
select x, property,
    avg(x)
        over (
            partition by property
            order by x
            rows between 1 preceding and 1 following
        ) as '移動平均'
from int_t
where property in ('odd','even');
```

```plaintext
+----+----------+----------------+
| x  | property | 移動平均       |
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

構文：

```SQL
COUNT(expr) [OVER (analytic_clause)]
```

例：

現在の行から最初の行までのxの発生回数をカウントします。

```SQL
select x, property,
    count(x)
        over (
            partition by property
            order by x
            rows between unbounded preceding and current row
        ) as '累積合計'
from int_t where property in ('odd','even');
```

```plaintext
+----+----------+------------------+
| x  | property | 累積合計        |
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
CUME_DIST() 関数は、パーティション内の値の累積分布を計算し、その相対的な位置を、現在の行の値以下の割合として示します。0から1の範囲であり、パーセンタイルの計算やデータ分布解析に有用です。

構文：

```SQL
CUME_DIST() OVER (partition_by_clause order_by_clause)
```

**この関数は、ORDER BYと併用して、パーティション行を希望の順序に並べ替えます。ORDER BYを使用しない場合、すべての行は等位であり、値 N/N = 1 となります。N はパーティションのサイズです。**

CUME_DIST() は NULL 値を含み、これらを最も低い値として扱います。

次の例は、列 y の累積分布を列 x の各グループ内で示しています。

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

DENSE_RANK() 関数はランキングを表すために使用されます。RANK() とは異なり、DENSE_RANK() には空き番号がありません。例えば、2つの1位がある場合、DENSE_RANK() の3番目の番号は依然として2になりますが、RANK() の3番目の番号は3になります。

構文：

```SQL
DENSE_RANK() OVER(partition_by_clause order_by_clause)
```

次の例は、プロパティ列のグルーピングに従って列 x のランキングを示しています。

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

NTILE() 関数は、パーティション内のソートされた行を指定された `num_buckets` の数だけできるだけ均等に分割し、分割された行をそれぞれのバケツに格納し、1 から `[1, 2, ..., num_buckets]` までのバケツ番号を返します。

バケツのサイズに関して：

* 行数が指定された `num_buckets` で正確に割り切れる場合、すべてのバケツのサイズは同じになります。
* 行数を指定された `num_buckets` で割り切れない場合、2つの異なるサイズのバケツがあります。サイズの差は 1 です。行数が多いバケツが行数が少ないバケツよりも前にリストされます。

構文：

```SQL
NTILE (num_buckets) OVER (partition_by_clause order_by_clause)
```

`num_buckets`: 作成するバケツの数。この値は、定数の正の整数でなければなりません。その最大値は `2^63 - 1` です。

NTILE() 関数ではウィンドウ節は許可されていません。

NTILE() 関数は BIGINT 型のデータを返します。

例：

次の例では、パーティション内のすべての行を 2 つのバケツに分割します。

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

上記の例では、`num_buckets` が `2` の場合：

* No.1 から No.6 の行は最初のパーティションに分類されます。No.1 から No.3 の行は最初のバケツに、No.4 から No.6 の行は 2 番目のバケツに保存されます。
* No.7 から No.9 の行は 2 番目のパーティションに分類されます。No.7 と No.8 の行は最初のバケツに保存され、No.9 の行は 2 番目のバケツに保存されます。
* No.10 の行は 3 番目のパーティションに分類され、最初のバケツに保存されます。

<br/>

### FIRST_VALUE()

FIRST_VALUE() はウィンドウ範囲の最初の値を返します。

構文：

```SQL
FIRST_VALUE(expr [IGNORE NULLS]) OVER(partition_by_clause order_by_clause [window_clause])
```

`IGNORE NULLS` は v2.5.0 からサポートされています。これは、`expr` の NULL 値が計算から除外されるかどうかを決定するために使用されます。デフォルトでは、NULL 値が含まれるため、フィルタリングされた結果の最初の値が NULL の場合、NULL が返されます。IGNORE NULLS を指定すると、フィルタリングされた結果の最初の非 NULL 値が返されます。すべての値が NULL の場合、IGNORE NULLS を指定しても NULL が返されます。

例：

以下のデータがあるとします：

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

FIRST_VALUE() を使用して、国別のグループ化に基づいて各グループ内の最初の挨拶値を返します。

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
| USA     | John    | Hello     |
| USA     | Pete    | Hello     |
+---------+---------+-----------+
```

### LAG()

`offset` 行前の現在の行の値を返します。この関数はしばしば行間の値の比較やデータのフィルタリングに使用されます。

`LAG()` は以下のタイプのデータをクエリするために使用可能です：

* 数値：TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL
* 文字列：CHAR、VARCHAR
* 日付：DATE、DATETIME
* StarRocks v2.5 からは、BITMAP および HLL がサポートされています。

構文：

```SQL
LAG(expr [IGNORE NULLS] [, offset[, default]])
OVER([<partition_by_clause>] [<order_by_clause>])
```

パラメータ：

* `expr`：計算したいフィールドです。
* `offset`：オフセットです。これは **正の整数** でなければなりません。このパラメータが指定されていない場合、デフォルトで 1 になります。
* `default`：一致する行が見つからない場合に返されるデフォルト値です。このパラメータが指定されていない場合、デフォルトで NULL になります。`default` は `expr` と互換性のある型の任意の式をサポートします。
* `IGNORE NULLS`はv3.0からサポートされています。これは`expr`のNULL値が結果に含まれるかどうかを決定するために使用されます。デフォルトでは、NULL値は`offset`行がカウントされるときに含まれます。つまり、NULLが返されるのは、対象の行の値がNULLの場合です。 例1を参照してください。 `IGNORE NULLS`を指定すると、`offset`行がカウントされるときにNULL値が無視され、システムは引き続き`offset`の非NULL値を検索します。 `offset`の非NULL値が見つからない場合、NULLまたは（指定されている場合）`default`が返されます。例2を参照してください。

例1：IGNORE NULLSが指定されていない

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

このテーブルからデータをクエリします。ここで `offset` は2、つまり前の2つの行をトラバースします。 `default` は0で、一致する行が見つからない場合に0が返されます。

出力：

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

最初の2行の場合、前の2行は存在せず、デフォルト値0が返されます。 

行3のNULLの場合、2つ前の値はNULLであり、NULLが返されます。

例2：IGNORE NULLSが指定されている

前述のテーブルとパラメータ設定を使用します。

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

行1から4の場合、システムはそれぞれの前の行で2つの非NULL値を見つけることができず、デフォルト値0が返されます。

行7の値6の場合、2つ前の値はNULLですが、IGNORE NULLSが指定されているためNULLが無視されます。システムは引き続き非NULL値を検索し、行4の2が返されます。
```
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

    7 ~ 10行目では、後続の行に2つの非NULL値を見つけることができないため、デフォルト値0が返されます。

    最初の行では、2つの行先の値がNULLであり、NULLはIGNORE NULLSが指定されているため無視されます。システムは引き続き2番目の非NULL値を検索し、4行目の2が返されます。

### MAX()

現在のウィンドウで指定された行の最大値を返します。

構文

```SQL
MAX(expr) [OVER (analytic_clause)]
```

例：

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

StarRocks 2.4以降では、`rows between n preceding and n following` という行範囲を指定できるようになり、現在の行の前のn行と現在の行の後のn行を取得できます。

例のステートメント：

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

現在のウィンドウで指定された行の最小値を返します。

構文：

```SQL
MIN(expr) [OVER (analytic_clause)]
```

例：

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

StarRocks 2.4以降では、`rows between n preceding and n following` という行範囲を指定できるようになり、現在の行の前のn行と現在の行の後のn行を取得できます。

例のステートメント：

```sql
select x, property,
    min(x)
    over (
          order by property, x desc
          rows between 3 preceding and 2 following) as 'local minimum'
from int_t
where property in ('prime','square');
```
`<column_list>`: データを取得したい列。

`<data_source>`: データソースは一般的にはテーブルです。

`<window_function>`: `QUALIFY`句の後には、ROW_NUMBER()、RANK()、DENSE_RANK()を含むウィンドウ関数しか続けることができません。

**例:**

```SQL
-- テーブルを作成する。
CREATE TABLE sales_record (
   city_id INT,
   item STRING,
   sales INT
) DISTRIBUTED BY HASH(`city_id`);

-- テーブルにデータを挿入する。
insert into sales_record values
(1,'fruit',95),
(2,'drinks',70),
(3,'fruit',87),
(4,'drinks',98);

-- テーブルからデータをクエリする。
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

例1: 行番号が1より大きいレコードをテーブルから取得する。

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

例2: テーブルの各パーティションから行番号が1のレコードを取得します。テーブルは`item`によって2つのパーティションに分割され、各パーティションの最初の行が返されます。

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

例3: テーブルの各パーティションから売上ランク1位のレコードを取得します。テーブルは`item`によって2つのパーティションに分割され、各パーティションの売上額が最も高い行が返されます。

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

`QUALIFY`を含むクエリの句の実行順序は、次の順序で評価されます:

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

プロパティでグループ化し、グループ内の**現在、前、および次の行の合計**を計算します。

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

式の標本分散を返します。VAR_POPおよびVARIANCE_POPはVARIANCEの別名です。これらの関数はv2.5.10以降、ウィンドウ関数として使用できます。

**構文:**

```SQL
VARIANCE(expr) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注記**
>
> 2.5.13、3.0.7、3.1.4以降では、このウィンドウ関数はORDER BYおよびWindow句をサポートしています。

**パラメータ:**

`expr`がテーブルの列である場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、またはDECIMALに評価される必要があります。

**例:**

テーブル`agg`が次のデータを持っているとします:

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

式の標本分散を返します。VAR_SAMP()関数はウィンドウ関数として使用できます。v2.5.10以降から使用できます。

**構文:**

```sql
VAR_SAMP(expr) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注記**
>
> 2.5.13、3.0.7、3.1.4以降では、このウィンドウ関数はORDER BYおよびWindow句をサポートしています。

**パラメータ:**

`expr`がテーブルの列である場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、またはDECIMALに評価される必要があります。

**例:**

テーブル`agg`が次のデータを持っているとします:

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
```plaintext
+ {T}
+ {T}
  + {T}
    + {T}
      + {T}
        + {T}
          + {T}
  + {T}

パラメータ：
exprがテーブルの列の場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、またはDECIMALで評価する必要があります。

例：
テーブルaggに次のデータがあるとします：

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

STDDEV_SAMP

式の標本標準偏差を返します。この関数はv2.5.10以降のウィンドウ関数として使用できます。

構文：

```sql
STDDEV_SAMP(expr) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
>
> 2.5.13、3.0.7、3.1.4以降、このウィンドウ関数はORDER BYおよびWindow句をサポートしています。

パラメータ：
exprがテーブルの列の場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、またはDECIMALで評価する必要があります。

例：
テーブルaggに次のデータがあるとします：

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

COVAR_SAMP

2つの式の標本共分散を返します。この関数はv2.5.10からサポートされています。また、集約関数でもあります。

構文：

```sql
COVAR_SAMP(expr1,expr2) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
>
> 2.5.13、3.0.7、3.1.4以降、このウィンドウ関数はORDER BYおよびWindow句をサポートしています。

パラメータ：
exprがテーブルの列の場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、またはDECIMALで評価する必要があります。

例：
テーブルaggに次のデータがあるとします：

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

COVAR_POP

2つの式の母集団共分散を返します。この関数はv2.5.10からサポートされています。また、集約関数でもあります。

構文：

```sql
COVAR_POP(expr1, expr2) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
>
> 2.5.13、3.0.7、3.1.4以降、このウィンドウ関数はORDER BYおよびWindow句をサポートしています。

パラメータ：
exprがテーブルの列の場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、またはDECIMALで評価する必要があります。

例：
テーブルaggに次のデータがあるとします：

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

CORR

2つの式間のピアソン相関係数を返します。この関数はv2.5.10からサポートされています。また、集約関数でもあります。

構文：

```sql
CORR(expr1, expr2) OVER([partition_by_clause] [order_by_clause] [order_by_clause window_clause])
```

> **注意**
>
> 2.5.13、3.0.7、3.1.4以降、このウィンドウ関数はORDER BYおよびWindow句をサポートしています。

パラメータ：
exprがテーブルの列の場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、またはDECIMALで評価する必要があります。
```
**例：**

テーブル`agg`に次のデータがあるとします：

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
```