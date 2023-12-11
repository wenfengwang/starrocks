---
displayed_sidebar: "Japanese"
---

# generate_series

## 説明

`start` と `end` で指定された間隔内の値のシリーズを生成し、オプションの `step` を持ちます。

generate_series() はテーブル関数です。テーブル関数は、各入力行に対して行セットを返すことができます。行セットには、ゼロ個、1個、または複数の行が含まれることができます。各行には1つ以上の列が含まれることができます。

StarRocks で generate_series() を使用するには、入力パラメータが定数である場合は、TABLE キーワードで囲む必要があります。入力パラメータがカラム名のような式である場合、TABLE キーワードは必要ありません(Example 5 を参照)。

この関数は v3.1 からサポートされています。

## 構文

```SQL
generate_series(start, end [,step])
```

## パラメータ

- `start`: シリーズの開始値、必須です。サポートされているデータ型は INT、BIGINT、および LARGEINT です。
- `end`: シリーズの終了値、必須です。サポートされているデータ型は INT、BIGINT、および LARGEINT です。
- `step`: 増分または減分する値、オプションです。サポートされているデータ型は INT、BIGINT、および LARGEINT です。指定されていない場合、デフォルトのステップは 1 です。 `step` は負または正の値のいずれかにすることができますが、ゼロにすることはできません。

これらの3つのパラメータは、データ型が同じでなければなりません。例: `generate_series(INT start, INT end [, INT step])`。

## 戻り値

`start` および `end` の入力パラメータと同じ値のシリーズを返します。

- `step` が正の場合、`start` が `end` よりも大きい場合、0行が返されます。逆に、`step` が負の場合、`start` が `end` よりも小さい場合、0行が返されます。
- `step` が 0 の場合、エラーが返されます。
- この関数は、ヌルについて次のように処理します: 入力パラメータがリテラルヌルの場合、エラーが報告されます。式の結果がヌルである場合、0行が返されます(Example 5 を参照)。

## 例

Example 1: デフォルトのステップ `1` で昇順の範囲 [2,5] 内の値のシーケンスを生成します。

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

Example 2: 指定されたステップ `2` で昇順の範囲 [2,5] 内の値のシーケンスを生成します。

```SQL
MySQL > select * from TABLE(generate_series(2, 5, 2));
+-----------------+
| generate_series |
+-----------------+
|               2 |
|               4 |
+-----------------+
```

Example 3: 指定されたステップ `-1` で降順の範囲 [5,2] 内の値のシーケンスを生成します。

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

Example 4: `step` が負であり、`start` が `end` より小さい場合、0行が返されます。

```SQL
MySQL > select * from TABLE(generate_series(2, 5, -1));
Empty set (0.01 sec)
```

Example 5: generate_series() の入力パラメータとしてテーブル列を使用する場合、generate_series() に `TABLE()` を使用する必要はありません。

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

-- (1,3) と (4,7) の行に対してステップ `1` で複数の行を生成します。
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

-- (5,2) と (9,6) の行に対してステップ `-1` で複数の行を生成します。
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

入力行 `(NULL, 10)` はヌルの値を持ち、この行に対して 0行が返されます。

## キーワード

テーブル関数, シリーズの生成