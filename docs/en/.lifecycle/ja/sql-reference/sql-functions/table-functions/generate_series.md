---
displayed_sidebar: "Japanese"
---

# generate_series

## 説明

`start` と `end` で指定された間隔内の値のシリーズを生成し、オプションの `step` を持つ。

generate_series() はテーブル関数です。テーブル関数は、各入力行に対して行セットを返すことができます。行セットには、ゼロ行、1行、または複数行が含まれることができます。各行には、1つ以上の列が含まれることができます。

StarRocks で generate_series() を使用するには、入力パラメータが定数の場合は TABLE キーワードで囲む必要があります。入力パラメータが列名などの式である場合は、TABLE キーワードは必要ありません（例5を参照）。

この関数は v3.1 からサポートされています。

## 構文

```SQL
generate_series(start, end [,step])
```

## パラメータ

- `start`: シリーズの開始値、必須。サポートされるデータ型は INT、BIGINT、LARGEINT です。
- `end`: シリーズの終了値、必須。サポートされるデータ型は INT、BIGINT、LARGEINT です。
- `step`: 増分または減分する値、オプション。サポートされるデータ型は INT、BIGINT、LARGEINT です。指定しない場合、デフォルトのステップは 1 です。`step` は負の値または正の値のいずれかにすることができますが、0 にすることはできません。

3つのパラメータは同じデータ型でなければなりません。例えば、`generate_series(INT start, INT end [, INT step])` のようになります。

## 戻り値

入力パラメータ `start` と `end` と同じ値のシリーズを返します。

- `step` が正の場合、`start` が `end` より大きい場合はゼロ行が返されます。逆に、`step` が負の場合、`start` が `end` より小さい場合はゼロ行が返されます。
- `step` が 0 の場合はエラーが返されます。
- この関数は null を以下のように扱います。入力パラメータのいずれかがリテラルの null の場合、エラーが報告されます。入力パラメータが式であり、式の結果が null の場合は 0 行が返されます（例5を参照）。

## 例

例1: デフォルトのステップ `1` を使用して、範囲 [2,5] 内の値のシーケンスを昇順で生成する。

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

例2: 指定したステップ `2` を使用して、範囲 [2,5] 内の値のシーケンスを昇順で生成する。

```SQL
MySQL > select * from TABLE(generate_series(2, 5, 2));
+-----------------+
| generate_series |
+-----------------+
|               2 |
|               4 |
+-----------------+
```

例3: 指定したステップ `-1` を使用して、範囲 [5,2] 内の値のシーケンスを降順で生成する。

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

例4: `step` が負であり、`start` が `end` より小さい場合はゼロ行が返されます。

```SQL
MySQL > select * from TABLE(generate_series(2, 5, -1));
Empty set (0.01 sec)
```

例5: generate_series() の入力パラメータとしてテーブルの列を使用する。この場合、generate_series() に `TABLE()` を使用する必要はありません。

```SQL
CREATE TABLE t_numbers(start INT, end INT)
DUPLICATE KEY (start)
DISTRIBUTED BY HASH(start) BUCKETS 1;

INSERT INTO t_numbers VALUES
(1, 3),
(5, 2),
(NULL, 10),
(4, 7),
(9,6);

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

-- ステップ 1 で行 (1,3) と (4,7) の複数行を生成する。
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

-- ステップ -1 で行 (5,2) と (9,6) の複数行を生成する。
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

入力行 `(NULL, 10)` は NULL の値を持っており、この行に対してはゼロ行が返されます。

## キーワード

テーブル関数, generate series
