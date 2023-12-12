---
displayed_sidebar: "Japanese"
---

# array_unique_agg

## 説明

配列列内の重複しない値（`NULL`を含む）を配列に集約します（複数の行を1行に）。

## 構文

```Haskell
ARRAY_UNIQUE_AGG(col)
```

## パラメータ

- `col`: 集約したい列。サポートされているデータ型はARRAYです。

## 戻り値

ARRAY型の値を返します。

## 使用上の注意

- 配列内の要素の順序はランダムです。
- 戻される配列内の要素のデータ型は、入力列の要素のデータ型と同じです。
- 入力が空でgroup-by列がない場合、`NULL`が返されます。

## 例

次のデータテーブルを例にとってください：

```plaintext
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

例 1: 列 `a` の値をグループ化し、列 `b` 内の重複しない値を配列に集約します。

```plaintext
mysql> select a, array_unique_agg(b) from array_unique_agg_example group by a;
+------+---------------------+
| a    | array_unique_agg(b) |
+------+---------------------+
|    1 | [4,1,2,3]           |
|    2 | [4,1,2,3,null]      |
+------+---------------------+
```

例 2: 列 `b` の値をWHERE句を使用して集約します。フィルタ条件を満たすデータがない場合、`NULL`が返されます。

```plaintext
mysql> select array_unique_agg(b) from array_unique_agg_example where a < 0;
+---------------------+
| array_unique_agg(b) |
+---------------------+
| NULL                |
+---------------------+
```

## キーワード

ARRAY_UNIQUE_AGG, ARRAY