---
displayed_sidebar: "Japanese"
---

# generate_series

## 説明

`start`、`end`で指定した区間内の値のシリーズを、オプションの`step`で生成します。

generate_series() はテーブル関数です。テーブル関数はそれぞれの入力行に対して行セットを返すことができます。行セットには、0行、1行、または複数行が含まれることがあります。各行には1つ以上の列が含まれることがあります。

StarRocksでgenerate_series()を使用するには、入力パラメータが定数の場合は、TABLEキーワードで囲む必要があります。列名などの式の場合は、TABLEキーワードは不要です（Example 5を参照）。

この関数はv3.1からサポートされています。

## 構文

```SQL
generate_series(start, end [,step])
```

## パラメータ

- `start`: シリーズの開始値、必須。サポートされるデータ型はINT、BIGINT、LARGEINTです。
- `end`: シリーズの終了値、必須。サポートされるデータ型はINT、BIGINT、LARGEINTです。
- `step`: 増分または減分する値、オプション。サポートされるデータ型はINT、BIGINT、LARGEINTです。指定しない場合、デフォルトのステップは1です。`step`は負の値でも正の値でも構いませんが、0にすることはできません。

3つのパラメータは同じデータ型でなければなりません。例えば、`generate_series(INT start, INT end [, INT step])`です。

## 返り値

入力パラメータ`start`と`end`と同じ値のシリーズを返します。

- `step`が正の場合、`start`が`end`よりも大きい場合に0行が返されます。逆に、`step`が負の場合、`start`が`end`よりも小さい場合に0行が返されます。
- `step`が0の場合はエラーが返されます。
- この関数は、nullに対して以下のように処理します: 入力パラメータがリテラルのnullである場合はエラーが報告されます。式の結果がnullである場合は0行が返されます（Example 5を参照）。

## 例

Example 1: デフォルトのステップ`1`で範囲[2,5]内の値のシーケンスを昇順で生成します。

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

Example 2: 指定されたステップ`2`で範囲[2,5]内の値のシーケンスを昇順で生成します。

```SQL
MySQL > select * from TABLE(generate_series(2, 5, 2));
+-----------------+
| generate_series |
+-----------------+
|               2 |
|               4 |
+-----------------+
```

Example 3: 指定されたステップ`-1`で範囲[5,2]内の値のシーケンスを降順で生成します。

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

Example 4: `step`が負で`start`が`end`よりも小さい場合は0行が返されます。

```SQL
MySQL > select * from TABLE(generate_series(2, 5, -1));
Empty set (0.01 sec)
```

Example 5: generate_series()の入力パラメータとしてテーブルの列を使用する場合、`TABLE()`をgenerate_series()と共に使用する必要はありません。

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

-- 行(1,3)と行(4,7)に対してステップ1で複数の行を生成します。
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

-- 行(5,2)と行(9,6)に対してステップ-1で複数の行を生成します。
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

入力行`(NULL, 10)`はNULL値を持っており、この行に対して0行が返されます。

## キーワード

テーブル関数, シリーズを生成