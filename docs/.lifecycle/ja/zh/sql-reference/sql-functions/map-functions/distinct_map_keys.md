---
displayed_sidebar: Chinese
---

# distinct_map_keys

## 機能

Map内の重複するKeyを削除します。概念的には、Map内のKeyは重複することができず、Valueは重複しても構いません。Keyが同じであるキーバリューペアについて、この関数は最後に出現したキーバリューペアのみを保持します（LAST WIN原則）。例えば、`SELECT distinct_map_keys(MAP{1:3,1:'4'});` は `{1:"4"}` を返します。

この関数は、外部MAPデータをクエリする際に、Map内のKeyの重複を排除するために使用できます。StarRocksのネイティブテーブルは、Map内の重複するKeyを自動的に排除します。

この関数はバージョン3.1からサポートされています。

## 文法

```Haskell
distinct_map_keys(any_map)
```

## パラメータ説明

`any_map`: Map式。

## 戻り値の説明

重複しないKeyを持つ新しいMapを返します。

入力値がNULLの場合は、NULLを返します。

## 例

例1：基本的な使用法。

```plain
SELECT distinct_map_keys(MAP{"a":1,"a":2});
+-------------------------------------+
| distinct_map_keys(MAP{'a':1,'a':2}) |
+-------------------------------------+
| {"a":2}                             |
+-------------------------------------+
```

例2：外部テーブルからMapデータをクエリし、`col_map`列の重複するKeyを削除します。`unique`列は結果を返します。

```plain
SELECT distinct_map_keys(col_map) AS unique, col_map FROM external_table;
+---------------+---------------+
|      unique   | col_map       |
+---------------+---------------+
|       {"c":2} | {"c":1,"c":2} |
|           NULL|          NULL |
| {"e":4,"d":5} | {"e":4,"d":5} |
+---------------+---------------+
3 rows in set (0.05 sec)
```
