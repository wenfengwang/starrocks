---
displayed_sidebar: Chinese
---

# bitmap_agg

## 機能

複数行の非NULL数値を1行のBITMAP値に結合する、つまり複数行を1行に変換する。

## 文法

```Haskell
BITMAP_AGG(col)
```

## パラメータ

`col`: 結合する数値が含まれる列。サポートされるデータ型はBOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINTで、これらの型に変換可能なVARCHARもサポートされています。

## 戻り値の説明

BITMAP型の値を返します。

## 使用説明

値が0未満または18446744073709551615を超える場合、その値は無視され、Bitmapに結合されません（例3を参照）。

## 例

テーブルを作成し、データをインポートします。

```sql
mysql> CREATE TABLE t1_test (
    c1 int,
    c2 boolean,
    c3 tinyint,
    c4 int,
    c5 bigint,
    c6 largeint
    )
DUPLICATE KEY(c1)
DISTRIBUTED BY HASH(c1);

INSERT INTO t1_test VALUES
    (1, true, 11, 111, 1111, 11111),
    (2, false, 22, 222, 2222, 22222),
    (3, true, 33, 333, 3333, 33333),
    (4, null, null, null, null, null),
    (5, -1, -11, -111, -1111, -11111),
    (6, null, null, null, null, "36893488147419103232");
```

テーブルのデータを照会します。

```PlainText
select * from t1_test order by c1;
+------+------+------+------+-------+----------------------+
| c1   | c2   | c3   | c4   | c5    | c6                   |
+------+------+------+------+-------+----------------------+
|    1 |    1 |   11 |  111 |  1111 | 11111                |
|    2 |    0 |   22 |  222 |  2222 | 22222                |
|    3 |    1 |   33 |  333 |  3333 | 33333                |
|    4 | NULL | NULL | NULL |  NULL | NULL                 |
|    5 |    1 |  -11 | -111 | -1111 | -11111               |
|    6 | NULL | NULL | NULL |  NULL | 36893488147419103232 |
+------+------+------+------+-------+----------------------+
```

例1：`c1`列の数値をBitmapに結合します。

```PlainText
mysql> select bitmap_to_string(bitmap_agg(c1)) from t1_test;
+----------------------------------+
| bitmap_to_string(bitmap_agg(c1)) |
+----------------------------------+
| 1,2,3,4,5,6                      |
+----------------------------------+
```

例2：`c2`列の数値をBitmapに結合し、NULL値を無視します。

```PlainText
mysql> select BITMAP_TO_STRING(BITMAP_AGG(c2)) from t1_test;
+----------------------------------+
| bitmap_to_string(bitmap_agg(c2)) |
+----------------------------------+
| 0,1                              |
+----------------------------------+
```

例3：`c6`列の数値をBitmapに結合し、NULL値と範囲外の値を無視します。

```PlainText
mysql> select bitmap_to_string(bitmap_agg(c6)) from t1_test;
+----------------------------------+
| bitmap_to_string(bitmap_agg(c6)) |
+----------------------------------+
| 11111,22222,33333                |
+----------------------------------+
```

## キーワード

BITMAP_AGG, BITMAP
