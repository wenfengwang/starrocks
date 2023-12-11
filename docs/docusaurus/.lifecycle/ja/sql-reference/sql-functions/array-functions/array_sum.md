---
displayed_sidebar: "Japanese"
---

# array_sum

## 説明

配列内のすべての要素を合計します。

StarRocks 2.5 から、array_sum() はラムダ式を引数として受け取ることができます。ただし、ラムダ式と直接動作することはできません。これは[array_map()](./array_map.md) から変換された結果で動作する必要があります。

## 構文

```Haskell
array_sum(array(type))
array_sum(lambda_function, arr1,arr2...) = array_sum(array_map(lambda_function, arr1,arr2...))
```

## パラメータ

- `array(type)`: 合計を計算したい配列。配列の要素は次のデータ型をサポートしています：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、および DECIMALV2。
- `lambda_function`: array_sum() でターゲット配列を計算するために使用されるラムダ式。

## 戻り値

数値を返します。

## 例

### ラムダ関数を使用しない場合の array_sum の使用

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

### ラムダ関数を使用した array_sum の使用

```plain text
-- [1,2,3] を [1,2,3] で乗算し、要素を合計します。
select array_sum(array_map(x->x*x,[1,2,3]));
+---------------------------------------------+
| array_sum(array_map(x -> x * x, [1, 2, 3])) |
+---------------------------------------------+
|                                          14 |
+---------------------------------------------+
```

## キーワード

ARRAY_SUM,ARRAY