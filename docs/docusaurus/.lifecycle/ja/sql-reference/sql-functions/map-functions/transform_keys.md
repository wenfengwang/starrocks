---
displayed_sidebar: "Japanese"
---

# transform_keys（キーを変換する）

## 説明

[ラムダ式](../Lambda_expression.md) を使用して、マップ内の各エントリに新しいキーを生成します。

この関数はv3.1以降でサポートされています。

## 構文

```Haskell
MAP transform_keys(lambda_func, any_map)
```

`lambda_func` は `any_map` の後に置かれることもあります:

```Haskell
MAP transform_keys(any_map, lambda_func)
```

## パラメータ

- `any_map` : マップ。

- `lambda_func` : `any_map` に適用したいラムダ式。

## 戻り値

キーのデータ型はラムダ式の結果によって決定され、値のデータ型は `any_map` の値と同じです。

入力パラメータのいずれかがNULLの場合、NULLが返されます。

元のマップのキーまたは値がNULLの場合、NULLは通常の値として処理されます。

ラムダ式には2つのパラメータが必要です。1つ目のパラメータはキーを表し、2つ目のパラメータは値を表します。

## 例

次の例では、[map_from_arrays](map_from_arrays.md) を使用して、マップの値 `{1:"ab",3:"cdd",2:null,null:"abc"}` を生成します。そして、ラムダ式を各キーに適用して、キーを1ずつ増やします。

```SQL
mysql> select transform_keys((k,v)->(k+1), col_map) from (select map_from_arrays([1,3,null,2,null],['ab','cdd',null,null,'abc']) as col_map)A;
+------------------------------------------+
| transform_keys((k, v) -> k + 1, col_map) |
+------------------------------------------+
| {2:"ab",4:"cdd",3:null,null:"abc"}       |
+------------------------------------------+
```