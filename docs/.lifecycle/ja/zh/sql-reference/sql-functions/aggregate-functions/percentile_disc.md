---
displayed_sidebar: Chinese
---

# percentile_disc

## 機能

百分位数を計算します。percentile_contと異なり、この関数は百分位に完全に一致する値が見つからない場合、隣接する2つの値のうち大きい方をデフォルトで返します。

この関数はバージョン2.5からサポートされています。

## 文法

```SQL
PERCENTILE_DISC (expr, percentile) 
```

## パラメータ説明

- `expr`: 百分位数を計算する列で、任意のソート可能な型の列値をサポートします。

- `percentile`: 指定された百分位で、0と1の間の浮動小数点定数です。中央値を計算する場合は0.5に設定します。

## 戻り値の説明

指定された百分位に対応する値を返します。百分位に完全に一致する値が見つからない場合は、隣接する2つの数値のうち大きい方を返します。

戻り値の型は`expr`のデータ型と同じです。

## 使用説明

この関数はNull値を無視します。

## 例

`exam`テーブルを作成し、データを挿入します。

```sql
CREATE TABLE exam (
    subject STRING,
    exam_result INT
) 
DISTRIBUTED BY HASH(`subject`);

INSERT INTO exam VALUES
('chemistry',80),
('chemistry',100),
('chemistry',NULL),
('math',60),
('math',70),
('math',85),
('physics',75),
('physics',80),
('physics',85),
('physics',99);
```

```Plain
SELECT * FROM exam ORDER BY subject;
+-----------+-------+
| Subject   | Score |
+-----------+-------+
| chemistry |    80 |
| chemistry |   100 |
| chemistry |  NULL |
| math      |    60 |
| math      |    70 |
| math      |    85 |
| physics   |    75 |
| physics   |    80 |
| physics   |    85 |
| physics   |    99 |
+-----------+-------+
```

各科目のスコアの中央値を計算し、Null値は無視します。

クエリ文：

```SQL
SELECT subject, PERCENTILE_DISC (Score, 0.5)
FROM exam GROUP BY subject;
```

結果を返す：

```Plain
+-----------+-----------------------------+
| Subject   | percentile_disc(Score, 0.5) |
+-----------+-----------------------------+
| chemistry |                         100 |
| math      |                          70 |
| physics   |                          85 |
+-----------+-----------------------------+
```
