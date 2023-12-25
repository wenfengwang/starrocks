---
displayed_sidebar: English
---


# stddev,stddev_pop,std

## 説明

expr 式の母集団標準偏差を返します。v2.5.10 以降、この関数はウィンドウ関数としても使用可能です。

## 構文

```Haskell
STDDEV(expr)
```

## パラメータ

`expr`: 式です。テーブルのカラムである場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、または DECIMAL に評価される必要があります。

## 戻り値

DOUBLE 型の値を返します。

## 例

```plaintext
mysql> SELECT stddev(lo_quantity), stddev_pop(lo_quantity) from lineorder;
+---------------------+-------------------------+
| stddev(lo_quantity) | stddev_pop(lo_quantity) |
+---------------------+-------------------------+
|   14.43100708360797 |       14.43100708360797 |
+---------------------+-------------------------+
```

## キーワード

STDDEV,STDDEV_POP,POP
