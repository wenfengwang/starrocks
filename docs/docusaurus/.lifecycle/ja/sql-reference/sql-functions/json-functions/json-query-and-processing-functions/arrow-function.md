---
displayed_sidebar: "Japanese"
---

# アロー関数

## 説明

JSONオブジェクト内の`json_path`式によって位置が特定される要素を問い合わせ、JSON値を返します。アロー関数`->`は[json_query](json_query.md)関数よりもコンパクトで使用が容易です。

## 構文

```Haskell
json_object_expr -> json_path
```

## パラメーター

- `json_object_expr`: JSONオブジェクトを表す式。このオブジェクトはJSON列、またはPARSE_JSONなどのJSON構築関数によって生成されるJSONオブジェクトである。

- `json_path`: JSONオブジェクト内の要素へのパスを表す式。このパラメーターの値は文字列である。StarRocksでサポートされているJSONパス構文の詳細については、[JSON関数および演算子の概要](../overview-of-json-functions-and-operators.md)を参照してください。

## 戻り値

JSON値を返します。

> 要素が存在しない場合、アロー関数は`NULL`というSQL値を返します。

## 例

例1: 指定されたJSONオブジェクト内の`'$.a.b'`式によって位置が特定される要素を問い合わせます。

```plaintext
mysql> SELECT parse_json('{"a": {"b": 1}}') -> '$.a.b';

       -> 1
```

例2: ネストされたアロー関数を使用して要素を問い合わせます。他のアロー関数がネストされたアロー関数内で、ネストされたアロー関数によって返された結果に基づいて要素を問い合わせます。

> この例では、ルート要素$が`json_path`式から省略されています。

```plaintext
mysql> SELECT parse_json('{"a": {"b": 1}}')->'a'->'b';

       -> 1
```

例3: 指定されたJSONオブジェクト内の`'a'`式によって位置が特定される要素を問い合わせます。

> この例では、ルート要素$が`json_path`式から省略されています。

```plaintext
mysql> SELECT parse_json('{"a": "b"}') -> 'a';

       -> "b"
```