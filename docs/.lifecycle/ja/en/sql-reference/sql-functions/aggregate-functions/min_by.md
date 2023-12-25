---
displayed_sidebar: English
---

# min_by

## 説明

`y`の最小値に関連付けられた`x`の値を返します。

たとえば、`SELECT min_by(subject, exam_result) FROM exam;`は、試験の点数が最も低い科目を返します。

この関数はv2.5からサポートされています。

## 構文

```Haskell
min_by(x,y)
```

## パラメーター

- `x`: 任意の型の式です。
- `y`: 順序付け可能な型の式です。

## 戻り値

`x`と同じ型の値を返します。

## 使用上の注意

- `y`は並べ替え可能な型である必要があります。`bitmap`や`hll`などの並べ替え不可能な型を`y`に使用すると、エラーが返されます。
- `y`にNULL値が含まれている場合、そのNULL値に対応する行は無視されます。
- `x`の値が複数あり、それらが同じ最小値の`y`を持つ場合、この関数は最初に遭遇した`x`の値を返します。

## 例

1. テーブル`exam`を作成します。

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

3. 最も低いスコアの科目を取得します。
   最も低いスコア`70`の科目`english`が返されます。

    ```Plain
    SELECT min_by(subject, exam_result) FROM exam;
    +------------------------------+
    | min_by(subject, exam_result) |
    +------------------------------+
    | english                      |
    +------------------------------+
    1 row in set (0.01 sec)
    ```
