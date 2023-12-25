---
displayed_sidebar: Chinese
---

# array_sum

## 機能

配列内のすべての要素の合計を求めます。

バージョン2.5から、array_sum()はlambda式を入力パラメータとして受け入れ、高階関数として使用することができます。array_sum()は直接lambdaをサポートしていませんが、array_map()で変換された配列に対してarray_sum()操作を行う必要があります。

Lambda式の詳細については、[Lambda expression](../Lambda_expression.md)を参照してください。

## 文法

```Haskell
array_sum(array(type))
array_sum(lambda_function, arr1, arr2...) = array_sum(array_map(lambda_function, arr1, arr2...))
```

## パラメータ説明

- `array(type)`：合計を求めるarray配列。配列の要素は以下のタイプをサポートしています：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMALV2。

- `lambda_function`：Lambda式。この式に基づいて配列の合計を求めます。

## 戻り値の説明

数値型の値を返します。

## 例

### Lambda式を使用しない場合

```plain text

select array_sum([11, 11, 12]);
+-----------------------+
| array_sum([11, 11, 12]) |
+-----------------------+
| 34                    |
+-----------------------+

select array_sum([11.33, 11.11, 12.324]);
+---------------------------------+
| array_sum([11.33, 11.11, 12.324]) |
+---------------------------------+
| 34.764                          |
+---------------------------------+
```

### Lambda式を使用する場合

```plain text
-- 最初に配列内の各要素を乗算し、その後で合計します。
select array_sum(array_map(x -> x * x, [1, 2, 3]));
+---------------------------------------------+
| array_sum(array_map(x -> x * x, [1, 2, 3])) |
+---------------------------------------------+
|                                          14 |
+---------------------------------------------+
```
