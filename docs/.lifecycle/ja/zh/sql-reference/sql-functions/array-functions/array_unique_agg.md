---
displayed_sidebar: Chinese
---

# array_unique_agg

## 機能

一列の中の distinct 値（null 含む）を配列に連結する（複数行を一行に変換）。

## 文法

```Haskell
ARRAY_UNIQUE_AGG(col)
```

## パラメータ説明

- `col`：値を連結する必要がある列。サポートされるデータタイプは ARRAY。

## 戻り値の説明

戻り値のデータタイプは ARRAY。

## 注意事項

- 配列内の要素の順序はランダムです。
- 戻り値の配列内の要素のタイプは `col` の要素のタイプと一致します。
- 条件に合う入力値がない場合、NULL を返します。

## 例

以下の例では、次のデータテーブルを使用して説明します：

```Plain_Text
mysql> select * from array_unique_agg_example;
+------+--------------+
| a    | b            |
+------+--------------+
|    2 | [1,null,2,4] |
|    2 | [1,null,3]   |
|    1 | [1,1,2,3]    |
|    1 | [2,3,4]      |
+------+--------------+
```

例1: `a` 列でグループ化し、`b` 列の値を配列に連結します。

```Plain_Text
mysql> select a, array_unique_agg(b) from array_unique_agg_example group by a;
+------+---------------------+
| a    | array_unique_agg(b) |
+------+---------------------+
|    1 | [4,1,2,3]           |
|    2 | [4,1,2,3,null]      |
+------+---------------------+
```

例2: `b` 列の値を配列に連結し、where 句でフィルタリングします。条件に合う値がない場合、NULL を返します。

```plaintext
select array_unique_agg(b) from array_unique_agg_example where a < 2;
+---------------------+
| array_unique_agg(b) |
+---------------------+
| [4,1,2,3]           |
+---------------------+

mysql> select array_unique_agg(b) from array_unique_agg_example where a < 0;
+---------------------+
| array_unique_agg(b) |
+---------------------+
| NULL                |
+---------------------+
```

## キーワード

ARRAY_UNIQUE_AGG, ARRAY
