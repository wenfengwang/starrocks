---
displayed_sidebar: Chinese
---

# max_by

## 機能

`y` の最大値に関連付けられた `x` の値を返します。例えば `SELECT max_by(subject, exam_result) FROM exam;` は、`exam` テーブルの中で最も高い試験結果を持つ科目を返します。この関数はバージョン 2.5 からサポートされています。

## 文法

```Haskell
max_by(x,y)
```

## パラメータ説明

- `x`: 任意の型の式です。

- `y`: ソート可能なある型の式です。

## 戻り値の説明

戻り値の型は `x` と同じです。

## 使用説明

- `y` はソート可能なデータ型でなければなりません。`y` がソート不可能な場合、例えば BITMAP や HLL のような型であれば、エラーが返されます。

- `y` の値に NULL が含まれている場合、NULL に対応する行は計算に含まれません。

- 複数の行で最大値が存在する場合、最初に出現した `x` の値を返します。

## 例

1. `exam` テーブルを作成します。

    ```SQL
    CREATE TABLE exam (
        subject_id INT,
        subject STRING,
        exam_result INT
    ) DISTRIBUTED BY HASH(`subject_id`);
    ```

2. テーブルにデータを挿入し、テーブルのデータを照会します。

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

3. 得点が最も高い1つの科目を返します。

   `physics` と `music` の2つの科目が最高得点の95を持っていることがわかります。最初に出現した科目 `physics` を返します。

    ```Plain
    SELECT max_by(subject, exam_result) FROM exam;
    +------------------------------+
    | max_by(subject, exam_result) |
    +------------------------------+
    | physics                      |
    +------------------------------+
    1 row in set (0.02 sec)
    ```
