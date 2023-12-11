---
displayed_sidebar: "Japanese"
---

# array_agg

## 説明

カラムの値（`NULL`を含む）を配列に集約（複数行を1行に）し、必要に応じて特定のカラムで要素を並べ替えます。v3.0から、array_agg()はORDER BYを使用して要素をソートすることがサポートされています。

## 構文

```Haskell
ARRAY_AGG([distinct] col [order by col0 [desc | asc] [nulls first | nulls last] ...])
```

## パラメーター

- `col`: 集約したいカラムです。サポートされるデータ型はBOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、CHAR、DATETIME、DATE、ARRAY（v3.1以降）、MAP（v3.1以降）、STRUCT（v3.1以降）です。

- `col0`: `col`の順序を決定するカラムです。ORDER BYカラムは複数ある場合があります。

- `[desc | asc]`: 要素を昇順（デフォルト）または`col0`の降順で並べ替えるかを指定します。

- `[nulls first | nulls last]`: NULL値を先頭または最後に配置するかを指定します。

## 返り値

`col0`でオプション的にソートされたARRAY型の値を返します。

## 使用上の注意

- 配列内の要素の順序はランダムです。つまり、ORDER BYカラムが指定されていない場合や並べ替えのORDER BYカラムが指定されていない場合、カラムの値の順序と異なる場合があります。
- 返される配列の要素のデータ型は、カラムの値のデータ型と同じです。
- 入力が空でGROUP BYカラムがない場合は、`NULL`を返します。

## 例

次のデータテーブルを例にとります：

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

例1：カラム`a`の値をグループ化し、カラム`pv`の値を`name`で順序付けして配列に集約します。

```plaintext
mysql> select a, array_agg(pv order by name nulls first) from t group by a;
+------+---------------------------------+
| a    | array_agg(pv ORDER BY name ASC) |
+------+---------------------------------+
|    2 | [334]                           |
|   11 | [33]                            |
|    1 | [4,5,3]                         |
+------+---------------------------------+

-- 順序なしで値を集約
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

例2：カラム`pv`の値を`name`で順序付けして配列に集約します。

```plaintext
mysql> select array_agg(pv order by name desc nulls last) from t;
+----------------------------------+
| array_agg(pv ORDER BY name DESC) |
+----------------------------------+
| [3,4,5,33,334]                   |
+----------------------------------+
1 row in set (0.02 sec)

-- 順序なしで値を集約
mysql> select array_agg(pv) from t;
+----------------+
| array_agg(pv)  |
+----------------+
| [3,4,5,33,334] |
+----------------+
1 row in set (0.03 sec)
```

例3：カラム`pv`の値をWHERE句を使用して集約します。`pv`のデータがフィルタ条件に合致しない場合、`NULL`値が返されます。

```plaintext
mysql> select array_agg(pv order by name desc nulls last) from t where a < 0;
+----------------------------------+
| array_agg(pv ORDER BY name DESC) |
+----------------------------------+
| NULL                             |
+----------------------------------+
1 row in set (0.02 sec)

-- 順序なしで値を集約
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