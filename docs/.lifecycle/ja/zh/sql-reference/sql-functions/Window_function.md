---
displayed_sidebar: Chinese
---

# 使用ウィンドウ関数でデータを整理・フィルタリングする

本文では、StarRocks のウィンドウ関数の使用方法について説明します。

ウィンドウ関数は StarRocks に組み込まれた特別な関数です。集約関数と同様に、ウィンドウ関数は複数行のデータを計算して一つのデータ値を得ます。しかし、ウィンドウ関数は Over() 句を使用して**現在のウィンドウ**内のデータをソートおよびグループ化し、**結果セットの各行に対して**個別の値を計算します。これは、Group By でグループ化された各グループに対して一つの値を計算するのではなく、SELECT 句に追加の列を加えて結果セットを再編成およびフィルタリングする柔軟な方法を可能にします。

ウィンドウ関数は金融や科学計算の分野でよく使用され、トレンド分析、外れ値の計算、大量データのバケット分析などに利用されます。

現在 StarRocks でサポートされているウィンドウ関数には以下のものがあります：

* MIN(), MAX(), COUNT(), SUM(), AVG()
* FIRST_VALUE(), LAST_VALUE(), LEAD(), LAG()
* ROW_NUMBER(), RANK(), DENSE_RANK(), QUALIFY()
* NTILE()
* VARIANCE(), VAR_SAMP(), STD(), STDDEV_SAMP(), COVAR_SAMP(), COVAR_POP(), CORR()

## ウィンドウ関数の構文とパラメータ

構文：

```SQL
FUNCTION(args) OVER([partition_by_clause] [order_by_clause] [window_clause])
partition_by_clause ::= PARTITION BY expr [, expr ...]
order_by_clause ::= ORDER BY expr [ASC | DESC] [, expr [ASC | DESC] ...]
```

> 注意：ウィンドウ関数は SELECT リストと最も外側の Order By 句でのみ使用できます。クエリプロセス中、ウィンドウ関数は最後に実行されます。つまり、Join、Where、Group By などの操作が完了した後に実行されます。

パラメータ：

* **partition_by_clause**：Partition By 句。この句は入力行を指定された一つまたは複数の列でグループ化し、同じ値の行を一つのグループに分けます。
* **order_by_clause**：Order By 句。外側の Order By と同様に、Order By 句は入力行の並び順を定義します。Partition By が指定されている場合、Order By は各 Partition グループ内の順序を定義します。外側の Order By と唯一異なる点は、OVER() 句内の `Order By n`（n は正の整数）は何も操作を行わないことですが、外側の `Order By n` は第 `n` 列によるソートを意味します。

    以下の例は、SELECT リストに `id` 列を追加し、その値が `events` テーブルの `date_and_time` 列によってソートされた `1`, `2`, `3`... という順序であることを示しています。

    ```SQL
    SELECT row_number() OVER (ORDER BY date_and_time) AS id,
        c1, c2, c3, c4
    FROM events;
    ```

* **window_clause**：Window 句は、ウィンドウ関数に計算範囲を指定するために使用されます。現在の行を基準に、前後の数行をウィンドウ関数の計算対象とします。Window 句でサポートされる関数には `AVG()`, `COUNT()`, `FIRST_VALUE()`, `LAST_VALUE()`, `SUM()` があります。`MAX()` と `MIN()` については、Window 句では UNBOUNDED、PRECEDING キーワードを使用して開始範囲を指定できます。

    Window 句の構文：

    ```SQL
    ROWS BETWEEN [ { m | UNBOUNDED } PRECEDING | CURRENT ROW] [ AND [CURRENT ROW | { UNBOUNDED | n } FOLLOWING] ]
    ```

    > 注意：Window 句は Order By 句の内側になければなりません。

## AVG() ウィンドウ関数の使用

`AVG()` 関数は、特定のウィンドウ内で選択されたフィールドの平均値を計算するために使用されます。

構文：

```SQL
AVG( expr ) [OVER (*analytic_clause*)]
```

以下の例は、次のような株式データを模擬しています。株式コードは `JDR` で、`closing price` は毎日の終値を表しています。

```SQL
CREATE TABLE stock_ticker (
    stock_symbol  STRING,
    closing_price DECIMAL(8,2),
    closing_date  DATETIME
)
DUPLICATE KEY(stock_symbol)
COMMENT "OLAP"
DISTRIBUTED BY HASH(closing_date);

INSERT INTO stock_ticker VALUES 
    ("JDR", 12.86, "2014-10-02 00:00:00"), 
    ("JDR", 12.89, "2014-10-03 00:00:00"), 
    ("JDR", 12.94, "2014-10-04 00:00:00"), 
    ("JDR", 12.55, "2014-10-05 00:00:00"), 
    ("JDR", 14.03, "2014-10-06 00:00:00"), 
    ("JDR", 14.75, "2014-10-07 00:00:00"), 
    ("JDR", 13.98, "2014-10-08 00:00:00")
;
```

以下の例では `AVG()` 関数を使用して、その株式の毎日の終値と前後一日の終値の平均値を計算しています。

```SQL
select stock_symbol, closing_date, closing_price,
    avg(closing_price)
        over (partition by stock_symbol
              order by closing_date
              rows between 1 preceding and 1 following
        ) as moving_average
from stock_ticker;
```

結果：

```Plain Text
+--------------+---------------------+---------------+----------------+
| stock_symbol | closing_date        | closing_price | moving_average |
+--------------+---------------------+---------------+----------------+
| JDR          | 2014-10-02 00:00:00 |         12.86 |    12.87500000 |
| JDR          | 2014-10-03 00:00:00 |         12.89 |    12.89666667 |
| JDR          | 2014-10-04 00:00:00 |         12.94 |    12.79333333 |
| JDR          | 2014-10-05 00:00:00 |         12.55 |    13.17333333 |
| JDR          | 2014-10-06 00:00:00 |         14.03 |    13.77666667 |
| JDR          | 2014-10-07 00:00:00 |         14.75 |    14.25333333 |
| JDR          | 2014-10-08 00:00:00 |         13.98 |    14.36500000 |
+--------------+---------------------+---------------+----------------+
```

<br/>

## COUNT() ウィンドウ関数の使用

`COUNT()` 関数は、特定のウィンドウ内で条件を満たす行の数を返します。

構文：

```SQL
COUNT(expr) [OVER (analytic_clause)]
```

以下の例では `COUNT()` を使用して、**現在の行から最初の行まで**の `property` 列のデータが出現する回数を計算しています。

```SQL
select x, property,
    count(x)
        over (
            partition by property
            order by x
            rows between unbounded preceding and current row
        ) as cumulative_count
from int_t where property in ('odd','even');
```

結果：

```Plain Text
+----+----------+------------------+
| x  | property | cumulative_count |
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

<br/>

## DENSE_RANK() ウィンドウ関数の使用

`DENSE_RANK()` 関数は、特定のウィンドウ内のデータにランクを付けるために使用されます。関数内で同じランクが出現した場合、次の行のランクはその同じランク数に 1 を加えたものになります。したがって、`DENSE_RANK()` が返す順番は**連続した数字**です。一方、`RANK()` が返す順番は**不連続な数字**の可能性があります。

構文：

```SQL
DENSE_RANK() OVER(partition_by_clause order_by_clause)
```

以下の例では `DENSE_RANK()` を使用して `x` 列にランクを付けています。

```SQL
select x, y,
    dense_rank()
        over (
            partition by x
            order by y
        ) as rank
from int_t;
```

結果：

```Plain Text
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

<br/>

## FIRST_VALUE() ウィンドウ関数の使用

`FIRST_VALUE()` 関数は、ウィンドウ範囲内の**最初の**値を返します。

構文：

```SQL
FIRST_VALUE(expr [IGNORE NULLS]) OVER(partition_by_clause order_by_clause [window_clause])
```

バージョン 2.5 から `IGNORE NULLS` がサポートされており、計算結果に NULL 値を含めるかどうかを指定できます。`IGNORE NULLS` を指定しない場合、デフォルトでは NULL 値が含まれます。例えば、最初の値が NULL であれば NULL が返されます。`IGNORE NULLS` を指定した場合、最初の非 NULL 値が返されます。すべての値が NULL の場合は、`IGNORE NULLS` を指定しても NULL が返されます。

以下の例では次のデータを使用しています：

```SQL
select name, country, greeting
from mail_merge;
```

```Plain Text
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

以下の例では `FIRST_VALUE()` 関数を使用して、`country` 列でグループ化された各グループの最初の `greeting` 値を返しています。

```SQL
select country, name,
    first_value(greeting)
        over (
            partition by country
            order by name, greeting
        ) as greeting
from mail_merge;
```

結果：

```Plain Text
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

<br/>

## LAG() ウィンドウ関数の使用

`LAG()` 関数は、現在の行の**前**にある行の値を計算するために使用されます。この関数は行間の差を直接比較したり、データフィルタリングを行うために使うことができます。

`LAG()` 関数は以下のデータタイプのクエリに対応しています：

* 数値タイプ：TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL
* 文字列タイプ：CHAR、VARCHAR
* 日付タイプ：DATE、DATETIME
* バージョン 2.5 から、`LAG()` 関数は BITMAP および HLL データタイプのクエリにも対応しています。

**構文**

```SQL
LAG(expr [IGNORE NULLS] [, offset[, default]])
OVER([<partition_by_clause>] [<order_by_clause>])
```

**パラメータ説明**

* `expr`: 計算対象のフィールド。
* `offset`: オフセット量で、前方に検索する行数を表し、**正の整数**でなければなりません。指定されていない場合は、デフォルトで 1 として扱われます。
* `default`: 条件に合致する行が見つからない場合に返されるデフォルト値。`default` が指定されていない場合、デフォルトで NULL が返されます。`default` のデータタイプは `expr` と互換性がなければなりません。

* `IGNORE NULLS`：バージョン3.0から、`LAG()`は`IGNORE NULLS`をサポートしています。つまり、計算結果でNULL値を無視するかどうかです。`IGNORE NULLS`を指定しない場合、デフォルトの戻り値にはNULL値が含まれます。例えば、指定した現在行の前の`offset`行の値がNULLの場合、NULLを返します（例1を参照）。`IGNORE NULLS`を指定した場合、`offset`行前を遍歷する際にNULLの行を無視し、非NULL値をさらに前に遍歷します。`IGNORE NULLS`を指定したが、現在行の前にoffset個の非NULL値が存在しない場合、NULLまたは`default`（指定されている場合）を返します（例2を参照）。

**例**

例1：`IGNORE NULLS`未指定のlag

テーブル作成とデータ挿入：

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

データをクエリし、`offset`を2に指定し、前方に2行を探します。`default`を0に指定し、条件に合う行がない場合は0を返します。

結果：

```SQL
SELECT col_1, col_2, LAG(col_2,2,0) OVER (ORDER BY col_1) 
FROM test_tbl ORDER BY col_1;
+-------+-------+---------------------------------------------+
| col_1 | col_2 | LAG(col_2, 2, 0) OVER (ORDER BY col_1 ASC)  |
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

最初の2行については、前方に2つの非NULL値が存在しないため、デフォルト値0が返されます。

3行目のデータNULLについては、前方2行の値がNULLであり、`IGNORE NULLS`が指定されていないため、NULLを含む結果を返すことが許されているため、NULLが返されます。

例2：`IGNORE NULLS`指定のlag

上記のテーブルを使用します。

```SQL
SELECT col_1, col_2, LAG(col_2 IGNORE NULLS,2,0) OVER (ORDER BY col_1) 
FROM test_tbl ORDER BY col_1;
+-------+-------+---------------------------------------------+
| col_1 | col_2 | LAG(col_2, 2, 0) OVER (ORDER BY col_1 ASC)  |
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

1〜4行目については、現在行の前に2つの非NULL値が存在しないため、デフォルト値0が返されます。

7行目のデータ6については、前方2行の値がNULLであるため、`IGNORE NULLS`が指定されている場合、この行を無視してさらに前方を遍歷し、結果として4行目の2が返されます。

<br/>

## `LAST_VALUE()`ウィンドウ関数の使用

`LAST_VALUE()`はウィンドウ範囲内の**最後の**値を返します。`FIRST_VALUE()`とは逆です。

構文：

```SQL
LAST_VALUE(expr [IGNORE NULLS]) OVER(partition_by_clause order_by_clause [window_clause])
```

バージョン2.5から`IGNORE NULLS`をサポートしています。つまり、計算結果でNULL値を無視するかどうかです。`IGNORE NULLS`を指定しない場合、デフォルトではNULL値が含まれます。例えば、最後の値がNULLの場合はNULLを返します。`IGNORE NULLS`を指定した場合、最後の非NULL値を返します。すべての値がNULLの場合は、`IGNORE NULLS`を指定してもNULLが返されます。

以下の例では`LAST_VALUE()`関数を使用し、`country`列でグループ化し、各グループの最後の`greeting`値を返します。

```SQL
SELECT country, name,
    LAST_VALUE(greeting)
        OVER (
            PARTITION BY country
            ORDER BY name, greeting
        ) AS greeting
FROM mail_merge;
```

結果：

```Plain Text
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

<br/>

## `LEAD()`ウィンドウ関数の使用

現在の行**以降**の特定の行の値を計算するために使用されます。この関数は、行間の差を直接比較したり、データをフィルタリングするために使用できます。

`LEAD()`がサポートするデータ型は[LAG](#lagウィンドウ関数の使用)と同じです。

構文：

```Haskell
LEAD(expr [IGNORE NULLS] [, offset[, default]])
OVER([<partition_by_clause>] [<order_by_clause>])
```

パラメータ説明：

* `expr`: 計算対象のフィールド。
* `offset`: オフセット量で、後方に検索する行数を示し、**正の整数**である必要があります。指定されていない場合は、デフォルトで1として扱われます。
* `default`: 条件に合う行が見つからない場合に返されるデフォルト値。`default`が指定されていない場合、デフォルトでNULLが返されます。`default`のデータ型は`expr`と互換性がなければなりません。
* `IGNORE NULLS`：バージョン3.0から、`LEAD()`は`IGNORE NULLS`をサポートしています。つまり、計算結果でNULL値を無視するかどうかです。`IGNORE NULLS`を指定しない場合、デフォルトの戻り値にはNULL値が含まれます。例えば、指定した現在行の後の`offset`行の値がNULLの場合、NULLを返します（例1を参照）。`IGNORE NULLS`を指定した場合、`offset`行後を遍歷する際にNULLの行を無視し、非NULL値をさらに後方に遍歷します。`IGNORE NULLS`を指定したが、現在行の後にoffset個の非NULL値が存在しない場合、NULLまたは`default`（指定されている場合）を返します（例2を参照）。

例1：`IGNORE NULLS`未指定のlead

テーブル作成とデータ挿入：

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

データをクエリし、`offset`を2に指定し、後方に2行を探します。`default`を0に指定し、条件に合う行がない場合は0を返します。

結果：

```SQL
SELECT col_1, col_2, LEAD(col_2,2,0) OVER (ORDER BY col_1) 
FROM test_tbl ORDER BY col_1;
+-------+-------+----------------------------------------------+
| col_1 | col_2 | LEAD(col_2, 2, 0) OVER (ORDER BY col_1 ASC)  |
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

1行目のデータNULLについては、後方に2行遍歷した結果がNULLであるため、`IGNORE NULLS`が指定されていない場合、NULLを含む結果を返すことが許されているため、NULLが返されます。

最後の2行については、後方に2つの非NULL値が存在しないため、デフォルト値0が返されます。

例2：`IGNORE NULLS`指定のlead

上記のテーブルを使用します。

```SQL
SELECT col_1, col_2, LEAD(col_2 IGNORE NULLS,2,0) OVER (ORDER BY col_1) 
FROM test_tbl ORDER BY col_1;
+-------+-------+----------------------------------------------+
| col_1 | col_2 | LEAD(col_2, 2, 0) OVER (ORDER BY col_1 ASC)  |
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

7〜10行目については、後方に2つの非NULL値が存在しないため、デフォルト値0が返されます。

1行目のデータNULLについては、後方に2行遍歷した結果がNULLであるため、`IGNORE NULLS`が指定されている場合、この行を無視してさらに後方を遍歷し、結果として4行目の2が返されます。

<br />

## `MAX()`ウィンドウ関数の使用

`MAX()`関数は、現在のウィンドウで指定された行数内のデータの最大値を返します。

構文：

```SQL
MAX(expr) OVER (analytic_clause)
```

以下の例では、**最初の行から現在の行の後の1行まで**の最大値を計算します。

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

結果：

```Plain Text
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

バージョン2.4から、この関数は`ROWS BETWEEN n PRECEDING AND n FOLLOWING`を設定することをサポートしています。つまり、現在の行の前n行と後n行の最大値を計算することができます。例えば、現在の行の前3行と後2行の最大値を計算する場合、以下のように書けます：

```SQL
SELECT x, property,
    MAX(x)
        OVER (
            ORDER BY property, x
            ROWS BETWEEN 3 PRECEDING AND 2 FOLLOWING
        ) AS 'local maximum'
FROM int_t
WHERE property IN ('prime', 'square');
```

## `MIN()`ウィンドウ関数の使用

`MIN()`関数は、現在のウィンドウで指定された行数内のデータの最小値を返します。

構文：

```SQL
MIN(expr) OVER (analytic_clause)
```

以下の例では、**最初の行から現在の行の後の1行まで**の最小値を計算します。

```SQL
SELECT x, property,
    MIN(x)
        OVER (
            ORDER BY property, x DESC
            rows between unbounded preceding and 1 following
        ) as 'local minimum'
from int_t
where property in ('prime','square');
```

返却結果：

```Plain Text
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

バージョン2.4から、この関数は `rows between n preceding and n following` を設定することをサポートし、つまり現在の行の前n行と後ろn行の最小値を計算することができます。例えば、現在の行の前3行と後ろ2行の最小値を計算する場合、以下のように記述します：

```SQL
select x, property,
    min(x)
    over (
          order by property, x desc
          rows between 3 preceding and 2 following) as 'local minimum'
from int_t
where property in ('prime','square');
```

## NTILE() ウィンドウ関数の使用

`NTILE()` 関数は、分割されたデータを指定された数（`num_buckets`）のバケットに**できるだけ均等に**割り当て、各行が属するバケット番号を返します。バケット番号は `1` から `num_buckets` までです。`NTILE()` の戻り値の型は BIGINT です。

> 説明
>
> * 分割された行数が `num_buckets` で割り切れない場合、2つの異なるバケットサイズが存在し、その差は1です。大きいバケットが小さいバケットの前にあります。
> * 分割された行数が `num_buckets` で割り切れる場合、すべてのバケットのサイズは同じです。

構文：

```~SQL
NTILE (num_buckets) OVER (partition_by_clause order_by_clause)
```~

ここで、`num_buckets` はバケットの数で、定数の正の整数でなければならず、最大値は BIGINT の最大値、すなわち `2^63 - 1` です。

> 注意
> `NTILE()` 関数はウィンドウ句を使用できません。

以下の例では `NTILE()` 関数を使用して、現在のウィンドウ内のデータを `2` つのバケットに分割し、分割結果を `bucket_id` 列で表示します。

```~sql
select id, x, y,
    ntile(2)
        over (
            partition by x
            order by y
        ) as bucket_id
from t1;
```~

返却：

```~Plain Text
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
```~

上記の例のように、`num_buckets` が `2` の場合：

* 1行目から6行目は1つの分割で、1行目から3行目は最初のバケットに、4行目から6行目は2番目のバケットにあります。
* 7行目から9行目は1つの分割で、7行目と8行目は最初のバケットに、9行目は2番目のバケットにあります。
* 10行目は1つの分割で、最初のバケットにあります。

<br/>

## RANK() ウィンドウ関数の使用

`RANK()` 関数は、現在のウィンドウ内のデータにランクを付け、分割内の各行に対するランクを返します。ランクは、前の行のランクに1を加えたものです。`DENSE_RANK()` と異なり、`RANK()` が返す番号は**連続していない数字の可能性があり**、`DENSE_RANK()` が返す番号は**連続した数字です**。

構文：

```SQL
RANK() OVER(partition_by_clause order_by_clause)
```

以下の例では `x` 列にランクを付けます。

```SQL
select x, y, 
    rank() over(
        partition by x 
        order by y
    ) as `rank`
from int_t;
```

返却：

```Plain Text
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

<br/>

## ROW_NUMBER() ウィンドウ関数の使用

`ROW_NUMBER()` 関数は、各パーティションの各行に `1` から始まる連続した整数を返します。`RANK()` や `DENSE_RANK()` と異なり、`ROW_NUMBER()` が返す値は**重複することも空白があることもなく**、**連続して増加**します。

構文：

```SQL
ROW_NUMBER() OVER(partition_by_clause order_by_clause)
```

以下の例では `ROW_NUMBER()` を使用して、`x` 列をパーティションとして分割したデータに `rank` を指定します。

```SQL
select x, y, 
    row_number() over(
        partition by x 
        order by y
    ) as `rank`
from int_t;
```

返却：

```Plain Text
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

## QUALIFY ウィンドウ関数の使用

QUALIFY 句はウィンドウ関数の結果をフィルタリングするために使用されます。SELECT 文で、QUALIFY を使用してフィルタ条件を設定し、条件に合うレコードを複数のレコードから選択します。QUALIFY は集約関数の HAVING 句と同様の機能を持ちます。この関数はバージョン2.5からサポートされています。

QUALIFY はより簡潔なデータ選択方法を提供します。例えば、QUALIFY を使用しない場合、フィルタリング文は複雑になります：

```SQL
SELECT *
FROM (SELECT DATE,
             PROVINCE_CODE,
             TOTAL_SCORE,
             ROW_NUMBER() OVER(PARTITION BY PROVINCE_CODE ORDER BY TOTAL_SCORE) AS SCORE_ROWNUMBER
      FROM example_table) T1
WHERE T1.SCORE_ROWNUMBER = 1;
```

QUALIFY を使用すると、文は次のように簡単になります：

```SQL
SELECT DATE, PROVINCE_CODE, TOTAL_SCORE
FROM example_table 
QUALIFY ROW_NUMBER() OVER(PARTITION BY PROVINCE_CODE ORDER BY TOTAL_SCORE) = 1;
```

**現在 QUALIFY は以下のウィンドウ関数のみをサポートしています：ROW_NUMBER(), RANK(), DENSE_RANK()。**

**構文：**

```SQL
SELECT <column_list>
FROM <data_source>
[GROUP BY ...]
[HAVING ...]
QUALIFY <window_function>
[ ... ]
```

**パラメータ：**

* `<column_list>`: 取得するデータの列、複数の列はコンマで区切ります。
* `<data_source>`: データソース、通常はテーブルです。
* `<window_function>`: データをフィルタリングするためのウィンドウ関数。現在は ROW_NUMBER(), RANK(), DENSE_RANK() のみをサポートしています。

**例：**

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

-- テーブルのデータを照会する。
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

例1：分割なしで、行番号が1より大きいレコードを取得します。

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

例2：`item` でテーブルを2つの分割に分け、各分割で行番号が`1`のレコードを取得します。

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
```

例3：`item` でテーブルを2つの分割に分け、rank() を使用して各分割で `sales` の売上が第1位のレコードを取得します。

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

**注意事項：**

QUALIFY を含むクエリ文では、以下の順序で子句が実行されます：

> 1. From
> 2. Where
> 3. Group by
> 4. Having
> 5. Window
> 6. QUALIFY
> 7. Distinct
> 8. Order by
> 9. Limit

<br/>

## SUM() ウィンドウ関数の使用

`SUM()` 関数は特定のウィンドウ内の指定された行の合計を計算します。

構文：

```SQL
SUM(expr) [OVER (analytic_clause)]
```

以下の例では、`property` 列でグループ化されたデータに対して、**現在の行および前後の各1行**の `x` 列の合計を計算します。

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

返却：

```Plain Text
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

## VARIANCE, VAR_POP, VARIANCE_POP ウィンドウ関数の使用

VARIANCE() ウィンドウ関数は、式の総変動を計算します。VAR_POP と VARIANCE_POP は VARIANCE の別名です。

**構文：**

```SQL
VARIANCE(expr) [OVER (analytic_clause)]
VAR_POP(expr) [OVER (analytic_clause)]
VARIANCE_POP(expr) [OVER (analytic_clause)]
VARIANCE(expr) OVER([partition_by_clause] [order_by_clause] [window_clause])
```

> 注意
>
> バージョン2.5.13、3.0.7、3.1.4から、このウィンドウ関数はORDER BYおよびWindow句をサポートしています。

**パラメータ説明：**

表現 `expr` が列値の場合、以下のデータ型をサポートしています: TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL

**例：**

テーブル `agg` に以下のデータがあるとします:

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

VARIANCE() ウィンドウ関数を使用します。

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

## VAR_SAMP, VARIANCE_SAMP ウィンドウ関数の使用

VAR_SAMP() ウィンドウ関数は、式の標本分散を計算するために使用されます。

**構文：**

```sql
VAR_SAMP(expr) OVER([partition_by_clause] [order_by_clause] [window_clause])
```

> 注意
>
> バージョン2.5.13、3.0.7、3.1.4から、このウィンドウ関数はORDER BYおよびWindow句をサポートしています。

**パラメータ説明：**

表現 `expr` が列値の場合、以下のデータ型をサポートしています: TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL

**例：**

テーブル `agg` に以下のデータがあるとします:

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

## STD, STDDEV, STDDEV_POP ウィンドウ関数の使用

STD() ウィンドウ関数は、式の母集団標準偏差を計算するために使用されます。

**構文：**

```sql
STD(expr) OVER([partition_by_clause] [order_by_clause] [window_clause])
```

> 注意
>
> バージョン2.5.13、3.0.7、3.1.4から、このウィンドウ関数はORDER BYおよびWindow句をサポートしています。

**パラメータ説明：**

表現 `expr` が列値の場合、以下のデータ型をサポートしています: TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL

**例：**

テーブル `agg` に以下のデータがあるとします:

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

## STDDEV_SAMP ウィンドウ関数の使用

STDDEV_SAMP() ウィンドウ関数は、式の標本標準偏差を計算するために使用されます。

**構文：**

```sql
STDDEV_SAMP(expr) OVER([partition_by_clause] [order_by_clause] [window_clause])
```

> 注意
>
> バージョン2.5.13、3.0.7、3.1.4から、このウィンドウ関数はORDER BYおよびWindow句をサポートしています。

**パラメータ説明：**

表現 `expr` が列値の場合、以下のデータ型をサポートしています: TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL

**例：**

テーブル `agg` に以下のデータがあるとします:

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

## COVAR_SAMP ウィンドウ関数の使用

COVAR_SAMP() ウィンドウ関数は、式の標本共分散を計算するために使用されます。

**構文：**

```sql
COVAR_SAMP(expr1, expr2) OVER([partition_by_clause] [order_by_clause] [window_clause])
```

> 注意
>
> バージョン2.5.13、3.0.7、3.1.4から、このウィンドウ関数はORDER BYおよびWindow句をサポートしています。

**パラメータ説明：**

表現 `expr` が列値の場合、以下のデータ型をサポートしています: TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL

**例：**

テーブル `agg` に以下のデータがあるとします:

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

## COVAR_POP ウィンドウ関数の使用

COVAR_POP() ウィンドウ関数は、式の母集団共分散を計算するために使用されます。

**構文：**

```sql
COVAR_POP(expr1, expr2) OVER([partition_by_clause] [order_by_clause] [window_clause])
```

> 注意
>
> バージョン2.5.13、3.0.7、3.1.4から、このウィンドウ関数はORDER BYおよびWindow句をサポートしています。

**パラメータ説明：**

表現 `expr` が列値の場合、以下のデータ型をサポートしています: TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL

**例：**

テーブル `agg` に以下のデータがあるとします:

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

## CORR ウィンドウ関数の使用

CORR() ウィンドウ関数は、式の相関係数を計算するために使用されます。

**構文：**

```sql
CORR(expr1, expr2) OVER([partition_by_clause] [order_by_clause] [window_clause])
```

> 注意
>
> バージョン2.5.13、3.0.7、3.1.4から、このウィンドウ関数はORDER BYおよびWindow句をサポートしています。

**パラメータ説明：**

表現 `expr` が列値の場合、以下のデータ型をサポートしています: TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL

**例：**

テーブル `agg` に以下のデータがあるとします:

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

CORR() ウィンドウ関数を使用します。

```plaintext
mysql> select CORR(k, v) over (partition by no) FROM agg;
+-------------------------------------+
| corr(k, v) OVER (PARTITION BY no ) |
+-------------------------------------+
|                                NULL |
|                                  1 |
|                                  1 |
|                                  1 |
|                                  1 |
+-------------------------------------+

mysql> select CORR(k,v) over (
    partition by no
    order by k
    rows between unbounded preceding and 1 following) AS window_test
FROM agg order by no,k;
+-------------------+
| window_test       |
+-------------------+
|              NULL |
|                 1 |
|                 1 |
|                 1 |
|                 1 |
+-------------------+
列値としての `expr` 式が与えられた場合、以下のデータ型がサポートされます: TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL

**例：**

`agg` テーブルに以下のデータがあるとします:

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

CORR() ウィンドウ関数を使用します。

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

