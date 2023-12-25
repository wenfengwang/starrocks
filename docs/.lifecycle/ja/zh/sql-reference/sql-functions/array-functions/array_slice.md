---
displayed_sidebar: Chinese
---

# array_slice

## 機能

配列の一部を返します。`offset` で指定された位置から `input` 配列の `length` の長さの部分配列を取得します。

## 文法

```Haskell
array_slice(input, offset, length)
```

## パラメータ説明

* `input`：入力配列。型は ARRAY。配列の要素は以下のデータ型が可能です：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMALV2、VARCHAR、DATETIME、DATE、JSON。**バージョン 2.5 から、この関数は JSON 型の配列要素をサポートします。**
* `offset`：結果の部分配列の開始オフセット（**1 から始まります**）。サポートされるデータ型は BIGINT です。
* `length`：結果の部分配列の長さ、つまり要素の数。サポートされるデータ型は BIGINT です。

## 戻り値の説明

戻り値のデータ型は ARRAY（要素の型は入力 `input` と同じです）。

## 注意点

* オフセットは 1 から始まります。
* 指定された長さが実際に取得可能な長さを超える場合、条件に合うすべての要素を返します。例四を参照してください。

## 例

**例一**

```plain text
mysql> select array_slice([1,2,4,5,6], 3, 2) as res;
+-------+
| res   |
+-------+
| [4,5] |
+-------+
```

**例二**

```plain text
mysql> select array_slice(["sql","storage","query","execute"], 1, 2) as res;
+-------------------+
| res               |
+-------------------+
| ["sql","storage"] |
+-------------------+
```

**例三**

```plain text
mysql> select array_slice([57.73,97.32,128.55,null,324.2], 3, 3) as res;
+---------------------+
| res                 |
+---------------------+
| [128.55,null,324.2] |
+---------------------+
```

**例四**

```plain text
mysql> select array_slice([57.73,97.32,128.55,null,324.2], 3, 5) as res;
+---------------------+
| res                 |
+---------------------+
| [128.55,null,324.2] |
+---------------------+
```

上記の例では、指定された長さが 5 ですが、条件に合うのは 3 つの要素のみなので、全ての 3 つの要素を返します。
