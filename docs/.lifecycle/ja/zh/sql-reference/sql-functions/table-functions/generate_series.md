---
displayed_sidebar: Chinese
---

# generate_series

## 機能

`start` から `end` までの数値を生成し、ステップは `step` で、デフォルトの `step` は 1 です。

generate_series はテーブル関数です。テーブル関数は、各入力行に対して行の集合を返します。返される集合には、0行、1行、または複数行が含まれることがあります。各行には1つ以上の列が含まれることがあります。

StarRocks で generate_series() を呼び出す場合、入力パラメータが数値定数の場合は `TABLE()` キーワードで generate_series() を囲む必要があります。入力パラメータが式である場合、例えば列名の場合は `TABLE()` キーワードで囲む必要はありません。詳細は[例](#例)を参照してください。

この関数はバージョン 3.1 からサポートされています。

## 文法

```SQL
generate_series(start, end [, step])
```

## パラメータ説明

- `start`：開始値、必須。INT、BIGINT、LARGEINT 型をサポート。
- `end`：終了値、必須。INT、BIGINT、LARGEINT 型をサポート。
- `step`：数値が増加または減少するステップ、オプション。INT、BIGINT、LARGEINT 型をサポート。指定しない場合のデフォルト値は 1 です。`step` の値は 0 にはできません。そうするとエラーが発生します。

3つのパラメータは同じ型でなければなりません。例えば `generate_series(INT start, INT end [, INT step])` です。

## 戻り値の説明

一連の数値を返し、戻り値の型は `start` と `end` の型と同じです。

- `step` が正の数の場合、`start` が `end` より大きい場合は行を返しません。逆に、`step` が負の数の場合、`start` が `end` より小さい場合は行を返しません。
- `step` が 0 の場合はエラーが返されます。
- NULL 値の処理：入力パラメータがリテラル null の場合はエラーが返されます。入力パラメータが式で、その結果が NULL の場合は行を返しません。例 5 を参照してください。

## 例

例 1：デフォルトのステップを使用して、2 から 5 までの整数を昇順で返します。

```SQL
MySQL > select * from TABLE(generate_series(2, 5));
+-----------------+
| generate_series |
+-----------------+
|               2 |
|               3 |
|               4 |
|               5 |
+-----------------+
```

例 2：ステップ 2 を使用して、2 と 5 の間の整数を昇順で返します。

```SQL
MySQL > select * from TABLE(generate_series(2, 5, 2));
+-----------------+
| generate_series |
+-----------------+
|               2 |
|               4 |
+-----------------+
```

例 3：ステップ -1 を使用して、5 と 2 の間の数値を降順で返します。

```SQL
MySQL > select * from TABLE(generate_series(5, 2, -1));
+-----------------+
| generate_series |
+-----------------+
|               5 |
|               4 |
|               3 |
|               2 |
+-----------------+
```

例 4：ステップが負の数で、開始値が終了値より小さい場合、行を返しません。

```SQL
MySQL > select * from TABLE(generate_series(2, 5, -1));
Empty set (0.01 sec)
```

例 5：テーブルの列を generate_series() の入力パラメータとして使用します。この場合、`TABLE()` を使用する**必要はありません**。入力行に NULL が含まれている場合、その行に対しては行を返しません。例えば、以下の表の `(NULL, 10)` です。

```SQL
CREATE TABLE t_numbers(start INT, end INT)
DUPLICATE KEY (start)
DISTRIBUTED BY HASH(start) BUCKETS 1;

INSERT INTO t_numbers VALUES
(1, 3),
(5, 2),
(NULL, 10),
(4, 7),
(9, 6);

SELECT * FROM t_numbers;
+-------+------+
| start | end  |
+-------+------+
|  NULL |   10 |
|     1 |    3 |
|     4 |    7 |
|     5 |    2 |
|     9 |    6 |
+-------+------+

-- デフォルトのステップは 1 で、データ行 (1, 3) と (4, 7) に対して昇順で複数行を返します。
SELECT * FROM t_numbers, generate_series(t_numbers.start, t_numbers.end);
+-------+------+-----------------+
| start | end  | generate_series |
+-------+------+-----------------+
|     1 |    3 |               1 |
|     1 |    3 |               2 |
|     1 |    3 |               3 |
|     4 |    7 |               4 |
|     4 |    7 |               5 |
|     4 |    7 |               6 |
|     4 |    7 |               7 |
+-------+------+-----------------+

-- ステップ -1 で、データ行 (5, 2) と (9, 6) に対して降順で複数行を返します。
SELECT * FROM t_numbers, generate_series(t_numbers.start, t_numbers.end, -1);
+-------+------+-----------------+
| start | end  | generate_series |
+-------+------+-----------------+
|     5 |    2 |               5 |
|     5 |    2 |               4 |
|     5 |    2 |               3 |
|     5 |    2 |               2 |
|     9 |    6 |               9 |
|     9 |    6 |               8 |
|     9 |    6 |               7 |
|     9 |    6 |               6 |
+-------+------+-----------------+
```

## キーワード

テーブル関数, table function, generate series
