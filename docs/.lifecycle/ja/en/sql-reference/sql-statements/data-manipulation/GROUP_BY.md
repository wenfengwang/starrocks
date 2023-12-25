---
displayed_sidebar: English
---

# GROUP BY

## 説明

GROUP BY `GROUPING SETS`、`CUBE`、`ROLLUP` は GROUP BY 句の拡張です。これにより、GROUP BY 句で複数のセットのグループを集約することができます。結果は、複数の対応する GROUP BY 句の UNION 操作と同等です。

GROUP BY 句は、要素が1つだけ含まれる GROUP BY `GROUPING SETS` の特別なケースです。例えば、GROUPING SETS ステートメントは以下のようになります。

  ```sql
  SELECT a, b, SUM(c) FROM tab1 GROUP BY GROUPING SETS ((a, b), (a), (b), ());
  ```

  このクエリ結果は以下と同等です。

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

  `GROUPING_ID(expr [, expr [, ...]])` は GROUPING に似ています。GROUPING_ID は指定された列の順序に従って列リストのビットマップ値を計算し、各ビットは GROUPING の値です。
  
  GROUPING_ID() 関数はビットベクトルの10進数値を返します。

### 構文

  ```sql
  SELECT ...
  FROM ...
  [ ... ]
  GROUP BY [
      ... |
      GROUPING SETS [, ...] (groupSet [, groupSet [, ...]]) |
      ROLLUP(expr [, expr [, ...]]) |
      expr [, expr [, ...]] WITH ROLLUP |
      CUBE(expr [, expr [, ...]]) |
      expr [, expr [, ...]] WITH CUBE
      ]
  [ ... ]
  ```

### パラメーター

  `groupSet` は、SELECT リスト内の列、エイリアス、または式から構成されるセットを表します。`groupSet ::= { (expr [, expr [, ...]]) }`

  `expr` は SELECT リスト内の列、エイリアス、または式を指します。

### 注意

  StarRocks は PostgreSQL のような構文をサポートしています。構文の例は以下の通りです。

  ```sql
  SELECT a, b, SUM(c) FROM tab1 GROUP BY GROUPING SETS ((a, b), (a), (b), ());
  SELECT a, b, c, SUM(d) FROM tab1 GROUP BY ROLLUP(a, b, c);
  SELECT a, b, c, SUM(d) FROM tab1 GROUP BY CUBE(a, b, c);
  ```

  `ROLLUP(a, b, c)` は以下の `GROUPING SETS` ステートメントと同等です。

  ```sql
  GROUPING SETS (
  (a, b, c),
  (a, b),
  (a),
  ()
  )
  ```

  `CUBE(a, b, c)` は以下の `GROUPING SETS` ステートメントと同等です。

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

  以下は実際のデータの例です。

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

  > SELECT k1, k2, SUM(k3) FROM t GROUP BY GROUPING SETS ((k1, k2), (k2), (k1), ());
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
