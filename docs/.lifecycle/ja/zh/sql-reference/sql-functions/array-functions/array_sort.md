---
displayed_sidebar: Chinese
---

# array_sort

## 機能

配列内の要素を昇順に並べ替えます。

## 文法

```Haskell
ARRAY_SORT(array)
```

## パラメータ説明

`array`：ソートする必要がある配列。サポートされているデータタイプは ARRAY です。

配列の要素は以下のデータタイプが可能です：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、DECIMALV2、DATETIME、DATE、JSON。**バージョン 2.5 から、この関数は JSON 型の配列要素をサポートしています。**

## 戻り値の説明

戻り値のデータタイプは ARRAY です。

## 注意点

* 昇順のみをサポートします。
* `null` は先頭に来ます。
* 降順で並べ替える必要がある場合は、ソート後の結果に `reverse` 関数を呼び出すことができます。
* 戻り値の配列の要素のタイプは、パラメータ `array` の要素のタイプと一致します。

## 例

以下の例では、次のデータテーブルを使用して説明します。

```Plain Text
mysql> select * from test;
+------+--------------+
| c1   | c2           |
+------+--------------+
|    1 | [4,3,null,1] |
|    2 | NULL         |
|    3 | [null]       |
|    4 | [8,5,1,4]    |
+------+--------------+
```

`c2` 列の配列内の要素を昇順で並べ替えます。

```Plain Text
mysql> select c1, array_sort(c2) from test;
+------+------------------+
| c1   | array_sort(`c2`) |
+------+------------------+
|    1 | [null,1,3,4]     |
|    2 | NULL             |
|    3 | [null]           |
|    4 | [1,4,5,8]        |
+------+------------------+
```
