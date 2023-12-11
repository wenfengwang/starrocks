---
displayed_sidebar: "Japanese"
---

# GROUP BY（グループ化）

## 説明

GROUP BY `GROUPING SETS` ｜ `CUBE` ｜ `ROLLUP` は、GROUP BY 句の拡張機能です。GROUP BY 句で複数のセットのグループを集計することができます。その結果は、複数の対応するGROUP BY句のUNION演算と同等です。

GROUP BY句は、GROUP BY GROUPING SETSを含む特別な場合です。たとえば、GROUPING SETSステートメント：

```sql
SELECT a, b, SUM( c ) FROM tab1 GROUP BY GROUPING SETS ( (a, b), (a), (b), ( ) );
```

上記のクエリ結果は、以下と等価です：

```sql
SELECT a, b, SUM( c ) FROM tab1 GROUP BY a, b
UNION
SELECT a, null, SUM( c ) FROM tab1 GROUP BY a
UNION
SELECT null, b, SUM( c ) FROM tab1 GROUP BY b
UNION
SELECT null, null, SUM( c ) FROM tab1
```

`GROUPING(expr)`は、列が集計列であるかどうかを示します。列が集計列であれば0、そうでなければ1となります。

`GROUPING_ID(expr  [ , expr [ , ... ] ])` はGROUPINGと似ていますが、指定された列の順序に従って列リストのビットマップ値を計算し、各ビットがGROUPINGの値となります。
 GROUPING_ID() 関数は、ビットベクトルの10進値を返します。

### 構文

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

### パラメータ

`groupSet`は、選択リスト内の列、エイリアス、または式で構成されるセットを表します。 `groupSet ::= { ( expr  [ , expr [ , ... ] ] )}`

`expr`は、選択リスト内の列、エイリアス、または式を示します。

### 注意

StarrocksはPostgreSQLのような構文をサポートしています。構文の例は以下の通りです：

```sql
SELECT a, b, SUM( c ) FROM tab1 GROUP BY GROUPING SETS ( (a, b), (a), (b), ( ) );
SELECT a, b,c, SUM( d ) FROM tab1 GROUP BY ROLLUP(a,b,c)
SELECT a, b,c, SUM( d ) FROM tab1 GROUP BY CUBE(a,b,c)
```

`ROLLUP(a,b,c)` は以下の`GROUPING SETS`ステートメントと等価です：

```sql
GROUPING SETS (
(a,b,c),
( a, b ),
( a),
( )
)
```

`CUBE ( a, b, c )` は以下の`GROUPING SETS`ステートメントと等価です：

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

## 例

以下は実際のデータの例です：

```plain text
> SELECT * FROM t;
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

> SELECT k1, k2, SUM(k3) FROM t GROUP BY GROUPING SETS ( (k1, k2), (k2), (k1), ( ) );
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