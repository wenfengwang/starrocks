---
displayed_sidebar: "Japanese"
---

# max_by

## 説明

`y` の最大値に関連付けられた `x` の値を返します。

例えば、`SELECT max_by(subject, exam_result) FROM exam;` は最高の試験結果を持つ科目を返すものです。

この関数は v2.5 からサポートされています。

## 構文

```Haskell
max_by(x,y)
```

## パラメーター

- `x`: 任意のタイプの式。
- `y`: 順序付けできるタイプの式。

## 戻り値

`x` と同じタイプの値を返します。

## 使用上の注意

- `y` はソート可能なタイプでなければなりません。`bitmap` や `hll` のようなソートできないタイプの `y` を使用すると、エラーが返されます。
- `y` が null 値を含む場合、null 値に対応する行は無視されます。
- もし `x` の複数の値が `y` の最大値を持つ場合、この関数は最初に出会った `x` の値を返します。

## 例

1. `exam` テーブルを作成します。

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
   2つの科目 `physics` と `music` が同じ最高得点 `95` を持ち、最初に出会った科目 (`physics`) が返されます。

    ```Plain
    SELECT max_by(subject, exam_result) FROM exam;
    +------------------------------+
    | max_by(subject, exam_result) |
    +------------------------------+
    | physics                      |
    +------------------------------+
    1 row in set (0.01 sec)
    ```