```yaml
---
displayed_sidebar: "Japanese"
---

# array_slice

## 説明

配列のスライスを返します。この関数は、`offset` で指定された位置から `input` の `length` 要素を取り出します。

## 構文

```Haskell
array_slice(input, offset, length)
```

## パラメータ

- `input`: 取り出したい配列。この関数は次の種類の配列要素をサポートしています: BOOLEAN, TINYINT, SMALLINT, INT, BIGINT, LARGEINT, FLOAT, DOUBLE, VARCHAR, DECIMALV2, DATETIME, DATE, 及び JSON。**JSON は 2.5 からサポートされています。**

- `offset`: 要素を取り出す位置。有効な値は `1` からです。BIGINT 値である必要があります。

- `length`: 取り出したいスライスの長さ。BIGINT 値である必要があります。

## 戻り値

`input` パラメータで指定された配列と同じデータ型を持つ配列を返します。

## 使用上の注意

- オフセットは 1 から開始します。
- 指定された長さが取り出せる要素の実際の数を超える場合、一致するすべての要素が返されます。Example 4 を参照してください。

## 例

例 1: 3 番目の要素から 2 つの要素を取り出す。

```Plain
mysql> select array_slice([1,2,4,5,6], 3, 2) as res;
+-------+
| res   |
+-------+
| [4,5] |
+-------+
```

例 2: 最初の要素から 2 つの要素を取り出す。

```Plain
mysql> select array_slice(["sql","storage","query","execute"], 1, 2) as res;
+-------------------+
| res               |
+-------------------+
| ["sql","storage"] |
+-------------------+
```

例 3: Null の要素は通常の値として扱われます。

```Plain
mysql> select array_slice([57.73,97.32,128.55,null,324.2], 3, 3) as res;
+---------------------+
| res                 |
+---------------------+
| [128.55,null,324.2] |
+---------------------+
```

例 4: 3 番目の要素から 5 つの要素を取り出す。

この関数は 5 つの要素を取り出そうとしていますが、3 番目の要素からは 3 つの要素しかありません。その結果、それら 3 つの要素がすべて返されます。

```Plain
mysql> select array_slice([57.73,97.32,128.55,null,324.2], 3, 5) as res;
+---------------------+
| res                 |
+---------------------+
| [128.55,null,324.2] |
+---------------------+
```