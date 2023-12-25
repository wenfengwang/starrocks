---
displayed_sidebar: Chinese
---

# Lambda 表現式

Lambda 表現式（Lambda expression）は匿名関数であり、高階関数の引数として渡すことができますが、単独で使用することはできません。Lambda 表現式を使用すると、コードをよりシンプルでコンパクト、拡張可能にすることができます。

Lambda 表現式は `->` 演算子を使用して表され、「goes to」と読みます。演算子の左側が入力パラメータで、右側が表現式です。

バージョン 2.5 から、StarRocks は以下の高階関数（higher-order function）で Lambda 表現式を使用することをサポートしています：[array_map()](./array-functions/array_map.md)、[array_filter()](./array-functions/array_filter.md)、[array_sum()](./array-functions/array_sum.md)、[array_sortby()](./array-functions/array_sortby.md)。

## 文法

```Haskell
parameter -> expression
```

## パラメータ説明

`parameter`: Lambda 表現式が受け取るパラメータで、0個、1個、または複数のパラメータを受け取ることができます。パラメータが2個以上の場合は、括弧で囲む必要があります。

`expression`: Lambda 表現式で、スカラー関数を内包することができます。表現式は入力パラメータをサポートする有効な表現式でなければなりません。

## 戻り値の説明

戻り値の型は `expression` の結果の型によって決まります。

## 注意事項

ほとんどのスカラー関数は Lambda 表現式内で使用することができますが、以下の例外があります：

- サブクエリはサポートされていません。例：`x -> 5 + (SELECT 3)`。
- 集約関数はサポートされていません。例：`x -> min(y)`。
- ウィンドウ関数はサポートされていません。
- テーブル関数はサポートされていません。
- 関連列（correlated column）はサポートされていません。

## 例

いくつかの Lambda 表現式の簡単な例：

```SQL
-- パラメータを受け取らず、3を直接返します。
() -> 3    
-- 数値型のパラメータを1つ受け取り、2を加えた結果を返します。
x -> x + 2 
-- 2つのパラメータを受け取り、それらの合計を返します。
(x, y) -> x + y 
-- Lambda 表現式内で関数を使用。
x -> COALESCE(x, 0)
x -> day(x)
x -> split(x, ",")
x -> if(x > 0, "positive", "negative")
```

高階関数での Lambda 表現式の使用例：

```Haskell
select array_map((x, y, z) -> x + y, [1], [2], [4]);
+----------------------------------------------+
| array_map((x, y, z) -> x + y, [1], [2], [4]) |
+----------------------------------------------+
| [3]                                          |
+----------------------------------------------+
1行がセットされました (0.01秒)
```
