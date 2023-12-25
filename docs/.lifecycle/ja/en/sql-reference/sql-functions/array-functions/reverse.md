---
displayed_sidebar: English
---

# reverse

## 説明

文字列または配列を反転します。文字列の文字または配列の要素を逆順にした文字列または配列を返します。

## 構文

```Haskell
reverse(param)
```

## パラメーター

`param`: 反転させる文字列または配列。VARCHAR、CHAR、またはARRAY型が可能です。

現在、この関数は一次元配列のみをサポートし、配列要素はDECIMAL型にはできません。この関数は以下の配列要素の型をサポートしています: BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、DECIMALV2、DATETIME、DATE、およびJSON。**JSONはバージョン2.5からサポートされています。**

## 戻り値

戻り値の型は`param`と同じです。

## 例

例 1: 文字列を反転します。

```Plain Text
MySQL > SELECT REVERSE('hello');
+------------------+
| REVERSE('hello') |
+------------------+
| olleh            |
+------------------+
1 row in set (0.00 sec)
```

例 2: 配列を反転します。

```Plain Text
MYSQL> SELECT REVERSE([4,1,5,8]);
+--------------------+
| REVERSE([4,1,5,8]) |
+--------------------+
| [8,5,1,4]          |
+--------------------+
```
