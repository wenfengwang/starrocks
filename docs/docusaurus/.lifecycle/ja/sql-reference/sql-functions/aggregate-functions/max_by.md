---
displayed_sidebar: "Japanese"
---

# max_by

## 説明

`y` の最大値に関連付けられた `x` の値を返します。

例えば、`SELECT max_by(subject, exam_result) FROM exam;` は最高の試験のスコアを持つ科目を返すものです。

この関数は v2.5 からサポートされています。

## 構文

```Haskell
max_by(x,y)
```

## パラメーター

- `x`: 任意の型の式。
- `y`: 順序付けられる型の式。

## 返り値

`x` と同じ型の値を返します。

## 使用上の注意

- `y` はソート可能な型でなければなりません。`bitmap` や `hll` のようなソート不可能な型の `y` を使用すると、エラーが返されます。
- `y` に null 値が含まれる場合、null 値に対応する行は無視されます。
- `x` の複数の値が同じ最大値の `y` を持つ場合、この関数は最初に出会った `x` の値を返します。

## 例

1. `exam` テーブルを作成します。

    ```SQL
    CREATE TABLE exam (
        subject_id INT,
        subject STRING,
        exam_result INT
    ) DISTRIBUTED BY HASH(`subject_id`);
    ```

2. このテーブルに値を挿入し、データをクエリします。

    ```SQL
    insert into exam values
    (1,'math',90),
    (2,'english',70),
    (3,'physics',95),
    (4,'chemistry',85),
    (5,'music',95),
    (6,'biology',null);

    select * from exam order by subject_id;
    +------------+-----------+-------------+
    | subject_id | subject   | exam_result |
    +------------+-----------+-------------+
    |          1 | math      |          90 |
    |          2 | english   |          70 |
    |          3 | physics   |          95 |
    |          4 | chemistry |          85 |
    |          5 | music     |          95 |
    |          6 | biology   |        null |
    +------------+-----------+-------------+
    6 rows in set (0.03 sec)
    ```

3. 最高スコアを持つ科目を取得します。
   `physics` と `music` の2つの科目が同じ最高スコアである `95` を持ち、最初に出会った科目 (`physics`) が返されます。

    ```Plain
    SELECT max_by(subject, exam_result) FROM exam;
    +------------------------------+
    | max_by(subject, exam_result) |
    +------------------------------+
    | physics                      |
    +------------------------------+
    1 row in set (0.01 sec)
    ```