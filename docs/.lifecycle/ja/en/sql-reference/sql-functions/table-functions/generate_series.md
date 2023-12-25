---
displayed_sidebar: English
---

# generate_series

## 説明

`start` と `end` で指定された間隔内で一連の値を生成し、オプションの `step` を使用します。

`generate_series()` はテーブル関数です。テーブル関数は、入力行ごとに行セットを返すことができます。行セットには、0行、1行、または複数の行を含むことができます。各行には、1つ以上の列を含むことができます。

StarRocksで `generate_series()` を使用するには、入力パラメータが定数の場合、`TABLE` キーワードで囲む必要があります。入力パラメータが式である場合、例えば列名など、`TABLE` キーワードは必要ありません（例5を参照）。

この関数はv3.1からサポートされています。

## 構文

```SQL
generate_series(start, end [, step])
```

## パラメータ

- `start`: シリーズの開始値、必須。サポートされるデータ型はINT、BIGINT、LARGEINTです。
- `end`: シリーズの終了値、必須。サポートされるデータ型はINT、BIGINT、LARGEINTです。
- `step`: インクリメントまたはデクリメントする値、オプション。サポートされるデータ型はINT、BIGINT、LARGEINTです。指定されない場合、デフォルトのステップは1です。`step`は負または正のいずれかにできますが、0にはできません。

3つのパラメータは同じデータ型でなければなりません。例えば、`generate_series(INT start, INT end [, INT step])`。
v3.3から名前付き引数がサポートされており、すべてのパラメータは名前付きで入力されます。例：`generate_series(start=>3, end=>7, step=>2)`。

## 戻り値

入力パラメータ `start` と `end` と同じ型の値のシリーズを返します。

- `step` が正の場合、`start` が `end` より大きい場合は0行が返されます。逆に、`step` が負の場合、`start` が `end` より小さい場合は0行が返されます。
- `step` が0の場合はエラーが返されます。
- この関数はnullを次のように扱います：入力パラメータがリテラルnullの場合、エラーが報告されます。入力パラメータが式で、その結果がnullの場合、0行が返されます（例5を参照）。

## 例

例1: デフォルトステップ `1` を使用して、範囲 [2,5] 内の値のシーケンスを昇順で生成します。

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

例2: 指定されたステップ `2` を使用して、範囲 [2,5] 内の値のシーケンスを昇順で生成します。

```SQL
MySQL > select * from TABLE(generate_series(2, 5, 2));
+-----------------+
| generate_series |
+-----------------+
|               2 |
|               4 |
+-----------------+
```

例3: 指定されたステップ `-1` を使用して、範囲 [5,2] 内の値のシーケンスを降順で生成します。

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

例4: `step` が負で `start` が `end` より小さい場合、0行が返されます。

```SQL
MySQL > select * from TABLE(generate_series(2, 5, -1));
Empty set (0.01 sec)
```

例5: テーブルの列を `generate_series()` の入力パラメータとして使用します。このユースケースでは、`generate_series()` と `TABLE()` を使用する必要はありません。

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

-- ステップ1で行 (1,3) と (4,7) に対して複数の行を生成します。
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

-- ステップ-1で行 (5,2) と (9,6) に対して複数の行を生成します。
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
例6: 名前付き引数を使用して、範囲 [2,5] 内の値のシーケンスを指定されたステップ `2` で昇順で生成します。

```SQL
MySQL > select * from TABLE(generate_series(start=>2, end=>5, step=>2));
+-----------------+
| generate_series |
+-----------------+
|               2 |
|               4 |
+-----------------+
```

入力行 `(NULL, 10)` にはNULL値があり、この行に対しては0行が返されます。

## キーワード

テーブル関数、シリーズ生成
