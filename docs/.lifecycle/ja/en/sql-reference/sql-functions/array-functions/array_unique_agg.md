---
displayed_sidebar: English
---

# array_unique_agg

## 説明

配列カラム内の異なる値（`NULL`を含む）を配列に集約します（複数行から1行へ）。

## 構文

```Haskell
ARRAY_UNIQUE_AGG(col)
```

## パラメーター

- `col`: 集約したい値が含まれるカラム。サポートされるデータ型はARRAYです。

## 戻り値

ARRAY型の値を返します。

## 使用上の注意

- 配列内の要素の順序はランダムです。
- 返される配列の要素のデータ型は、入力カラムの要素のデータ型と同じです。
- 入力が空でgroup-byカラムがない場合は`NULL`を返します。

## 例

以下のデータテーブルを例にします。

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

例 1: カラム`a`の値をグループ化し、カラム`b`の異なる値を配列に集約します。

```plaintext
mysql> select a, array_unique_agg(b) from array_unique_agg_example group by a;
+------+---------------------+
| a    | array_unique_agg(b) |
+------+---------------------+
|    1 | [1,2,3,4]           |
|    2 | [1,2,3,4,null]      |
+------+---------------------+
```

例 2: WHERE句を使用してカラム`b`の値を集約します。フィルタ条件に合致するデータがなければ`NULL`が返されます。

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
