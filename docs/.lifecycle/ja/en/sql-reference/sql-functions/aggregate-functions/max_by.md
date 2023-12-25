---
displayed_sidebar: English
---

# max_by

## 説明

`y`の最大値に関連付けられた`x`の値を返します。

たとえば、`SELECT max_by(subject, exam_result) FROM exam;`は、試験の最高得点を持つ科目を返します。

この関数はv2.5からサポートされています。

## 構文

```Haskell
max_by(x,y)
```

## パラメーター

- `x`: 任意の型の式。
- `y`: 順序付けが可能な型の式。

## 戻り値

`x`と同じ型の値を返します。

## 使用上の注意

- `y`は並べ替え可能な型である必要があります。`bitmap`や`hll`などの並べ替え不可能な型を`y`に使用すると、エラーが返されます。
- `y`にNULL値が含まれている場合、そのNULL値に対応する行は無視されます。
- `x`の値が複数あり、それらが`y`の最大値を持つ場合、この関数は最初に遭遇した`x`の値を返します。

## 例

1. `exam`テーブルを作成します。

    ```SQL
    CREATE TABLE exam (
        subject_id INT,
        subject STRING,
        exam_result INT
    ) DISTRIBUTED BY HASH(`subject_id`);
    ```

2. このテーブルに値を挿入し、このテーブルからデータをクエリします。

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

3. 最高得点を持つ科目を取得します。
   `physics`と`music`の2科目が最高得点95を持っており、最初に遭遇した科目（`physics`）が返されます。

    ```Plain
    SELECT max_by(subject, exam_result) FROM exam;
    +------------------------------+
    | max_by(subject, exam_result) |
    +------------------------------+
    | physics                      |
    +------------------------------+
    1 row in set (0.01 sec)
    ```
