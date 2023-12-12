---
displayed_sidebar: "Japanese"
---

# array_agg

## 説明

列の値（`NULL`を含む）を配列に集約し（複数行を1行に）、必要に応じて特定の列で要素を順に並べ替えます。 v3.0から、array_agg()はORDER BYを使用して要素をソートすることができます。

## 構文

```Haskell
ARRAY_AGG([distinct] col [order by col0 [desc | asc] [nulls first | nulls last] ...])
```

## パラメータ

- `col`: 集約したい列。サポートされるデータ型はBOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、CHAR、DATETIME、DATE、ARRAY（v3.1以降）、MAP（v3.1以降）、STRUCT（v3.1以降）です。

- `col0`: `col`の順序を決定する列。ORDER BY列は複数あるかもしれません。

- `[desc | asc]`: `col0`の要素を昇順（デフォルト）または降順でソートするかどうかを指定します。

- `[nulls first | nulls last]`: NULLの値を最初に配置するか最後に配置するかを指定します。

## 戻り値

`col0`でオプションの並び替えがされた、ARRAY型の値を返します。

## 使用上の注意

- 配列内の要素の順序はランダムです。つまり、ORDER BY列が指定されていない場合や、並べ替えされたORDER BY列が指定されていない場合、列の値の順序と異なる場合があります。
- 返された配列の要素のデータ型は、列の値のデータ型と同じです。
- 入力が空でグループ化されていない場合、`NULL`が返されます。

## 例

次のデータテーブルを例に取ります：

```plaintext
mysql> select * from t;
+------+------+------+
| a    | name | pv   |
+------+------+------+
|   11 |      |   33 |
|    2 | NULL |  334 |
|    1 | fzh  |    3 |
|    1 | fff  |    4 |
|    1 | fff  |    5 |
+------+------+------+
```

例1: 列 `a` の値をグループ化し、列 `pv` の値を`name`で順に並べ替えて配列に集約します。

```plaintext
mysql> select a, array_agg(pv order by name nulls first) from t group by a;
+------+---------------------------------+
| a    | array_agg(pv ORDER BY name ASC) |
+------+---------------------------------+
|    2 | [334]                           |
|   11 | [33]                            |
|    1 | [4,5,3]                         |
+------+---------------------------------+

-- 並び替えなしで値を集約します。
mysql> select a, array_agg(pv) from t group by a;
+------+---------------+
| a    | array_agg(pv) |
+------+---------------+
|   11 | [33]          |
|    2 | [334]         |
|    1 | [3,4,5]       |
+------+---------------+
3 rows in set (0.03 sec)
```

例2: 列 `pv` の値を`name`で並べ替えて配列に集約します。

```plaintext
mysql> select array_agg(pv order by name desc nulls last) from t;
+----------------------------------+
| array_agg(pv ORDER BY name DESC) |
+----------------------------------+
| [3,4,5,33,334]                   |
+----------------------------------+
1 row in set (0.02 sec)

-- 並び替えなしで値を集約します。
mysql> select array_agg(pv) from t;
+----------------+
| array_agg(pv)  |
+----------------+
| [3,4,5,33,334] |
+----------------+
1 row in set (0.03 sec)
```

例3: `WHERE`句を使用して列 `pv` の値を集約します。`pv`でフィルタ条件に一致するデータがない場合、`NULL`が返されます。

```plaintext
mysql> select array_agg(pv order by name desc nulls last) from t where a < 0;
+----------------------------------+
| array_agg(pv ORDER BY name DESC) |
+----------------------------------+
| NULL                             |
+----------------------------------+
1 row in set (0.02 sec)

-- 並び替えなしで値を集約します。
mysql> select array_agg(pv) from t where a < 0;
+---------------+
| array_agg(pv) |
+---------------+
| NULL          |
+---------------+
1 row in set (0.03 sec)
```

## キーワード

ARRAY_AGG, ARRAY