---
displayed_sidebar: Chinese
---

# reverse

## 機能

文字列または配列を反転させ、返される文字列または配列の順序が元の文字列または配列の順序と逆になります。

## 文法

```Haskell
reverse(param)
```

## パラメータ説明

`param`：反転させる必要がある文字列または配列で、現在は一次元配列のみをサポートし、配列の要素のデータ型が `DECIMAL` であることは許可されていません。`param` がサポートするデータ型は VARCHAR、CHAR、ARRAY です。

配列の要素は以下の型をサポートしています：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、DECIMALV2、DATETIME、DATE、JSON。**バージョン 2.5 から、この関数は JSON 型の配列要素をサポートします。**

## 戻り値の説明

返されるデータ型は `param` と一致します。

## 例

文字列を反転させる。

```Plain Text
MySQL > SELECT REVERSE('hello');
+------------------+
| REVERSE('hello') |
+------------------+
| olleh            |
+------------------+
```

配列を反転させる。

```Plain Text
MYSQL> SELECT REVERSE([4,1,5,8]);
+--------------------+
| REVERSE([4,1,5,8]) |
+--------------------+
| [8,5,1,4]          |
+--------------------+
```
