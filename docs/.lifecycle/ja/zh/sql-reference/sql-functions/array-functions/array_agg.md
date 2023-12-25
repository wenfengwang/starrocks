---
displayed_sidebar: Chinese
---

# array_agg

## 機能

一列の値（null値を含む）を配列に連結する（複数行を一行に変換）。バージョン3.0以前では、この関数は配列内の要素の順序を保証しませんでした。バージョン3.0から、array_agg() は ORDER BY を使用して配列内の要素をソートすることをサポートしています。

## 文法

```Haskell
ARRAY_AGG(col [order by col0 [desc | asc] [nulls first | nulls last] ...])
```

## パラメータ説明

- `col`：値を連結する列。サポートされるデータ型は BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、CHAR、DATETIME、DATE、ARRAY（3.1以降）、MAP（3.1以降）、STRUCT（3.1以降）です。

- `col0`: ソート列で、`col` の要素の順序を決定します。複数のソート列を指定できます。

- `[desc | asc]`: 配列の要素をソートする際に、`col0` に基づいて昇順または降順で並べます。デフォルトは昇順です。

- `[nulls first | nulls last]`: null値を要素の最前面または最後面に配置します。

## 戻り値の説明

戻り値のデータ型は ARRAY です。

## 注意事項

- ORDER BY を指定しない場合、配列内の要素の順序はランダムであり、元の列の値の順序と同じであることは保証されません。
- 戻り値の配列内の要素の型は `col` の型と一致します。
- 条件を満たす入力値がない場合、NULL を返します。

## 例

全行を変換する際に、条件を満たすデータがない場合、集約結果は `NULL` になります。

```Plain Text
mysql> select array_agg(c2) from test where c1>4;
+-----------------+
| array_agg(`c2`) |
+-----------------+
| NULL            |
+-----------------+
```

## 例

以下の例では、次のデータテーブルを使用して説明します：

```Plain_Text
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

例1: `a` 列でグループ化し、`pv` 列の値を配列に連結し、`name` の昇順で配列の要素をソートし、null値を最前面に配置します。

```Plain_Text
mysql> select a, array_agg(pv order by name nulls first) from t group by a;
+------+---------------------------------+
| a    | array_agg(pv ORDER BY name ASC) |
+------+---------------------------------+
|    2 | [334]                           |
|   11 | [33]                            |
|    1 | [4,5,3]                         |
+------+---------------------------------+

-- ソートなし。
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

例2: `pv` 列の値を配列に連結し、`name` の降順で配列の要素をソートし、null値を最後面に配置します。

```Plain_Text
mysql> select array_agg(pv order by name desc nulls last) from t;
+----------------------------------+
| array_agg(pv ORDER BY name DESC) |
+----------------------------------+
| [3,4,5,33,334]                   |
+----------------------------------+
1 row in set (0.02 sec)

-- ソートなし。
mysql> select array_agg(pv) from t;
+----------------+
| array_agg(pv)  |
+----------------+
| [3,4,5,33,334] |
+----------------+
1 row in set (0.03 sec)
```

例3: `pv` 列の値を配列に連結し、where句を使用して `pv` の値をフィルタリングします。条件を満たす値がない場合、NULL を返します。

```Plain_Text
mysql> select array_agg(pv order by name desc nulls last) from t where a < 0;
+----------------------------------+
| array_agg(pv ORDER BY name DESC) |
+----------------------------------+
| NULL                             |
+----------------------------------+
1 row in set (0.02 sec)

-- ソートなしで値を集約。
mysql> select array_agg(pv) from t where a < 0;
+---------------+
| array_agg(pv) |
+---------------+
| NULL          |
+---------------+
1 row in set (0.03 sec)
```

## キーワード

ARRAY_AGG、ARRAY
