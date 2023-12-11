---
displayed_sidebar: "Japanese"
---

# min_by

## 説明

`y` の最小値に関連する `x` の値を返します。

例えば、`SELECT min_by(subject, exam_result) FROM exam;` とすると、最低点の科目を返します。

この関数はv2.5からサポートされています。

## 構文

```Haskell
min_by(x,y)
```

## パラメータ

- `x`: 任意の型の式。
- `y`: 順序付け可能な型の式。

## 戻り値

`x` と同じ型の値を返します。

## 使用上の注意

- `y` はソート可能な型でなければなりません。`bitmap` や `hll` のようなソート不可能な型の `y` を使用すると、エラーが返されます。
- `y` が null 値を含む場合、null 値に対応する行は無視されます。
- `x` の複数の値が同じ `y` の最小値を持つ場合、この関数は最初に出会った `x` の値を返します。

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

3. 最低点の科目を取得します。
   最低点 `70` を持つ科目 `english` が返されます。

    ```Plain
    SELECT min_by(subject, exam_result) FROM exam;
    +------------------------------+
    | min_by(subject, exam_result) |
    +------------------------------+
    | english                      |
    +------------------------------+
    1 row in set (0.01 sec)
    ```