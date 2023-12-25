---
displayed_sidebar: English
---

# array_sum

## 説明

配列内のすべての要素を合計します。

StarRocks 2.5 から、array_sum() はラムダ式を引数として受け取ることができます。ただし、ラムダ式を直接操作することはできません。[array_map()](./array_map.md)から変換された結果に対して動作する必要があります。

## 構文

```Haskell
array_sum(array(type))
array_sum(lambda_function, arr1, arr2...) = array_sum(array_map(lambda_function, arr1, arr2...))
```

## パラメーター

- `array(type)`: 合計を計算する配列。配列要素は、BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMALV2 のデータ型をサポートします。
- `lambda_function`: array_sum() のターゲット配列を計算するために使用されるラムダ式。

## 戻り値

数値を返します。

## 例

### ラムダ関数なしでarray_sumを使用する

```plain text
mysql> select array_sum([11, 11, 12]);
+-----------------------+
| array_sum([11,11,12]) |
+-----------------------+
| 34                    |
+-----------------------+

mysql> select array_sum([11.33, 11.11, 12.324]);
+---------------------------------+
| array_sum([11.33,11.11,12.324]) |
+---------------------------------+
| 34.764                          |
+---------------------------------+
```

### ラムダ関数でarray_sumを使用する

```plain text
-- [1,2,3] と [1,2,3] を掛け合わせて要素を合計します。
select array_sum(array_map(x -> x * x, [1, 2, 3]));
+---------------------------------------------+
| array_sum(array_map(x -> x * x, [1, 2, 3])) |
+---------------------------------------------+
|                                          14 |
+---------------------------------------------+
```

## キーワード

ARRAY_SUM, ARRAY
