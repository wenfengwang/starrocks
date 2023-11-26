---
displayed_sidebar: "Japanese"
---

# percentile_disc

## 説明

入力列 `expr` の離散分布に基づいてパーセンタイル値を返します。正確なパーセンタイル値が見つからない場合、この関数は2つの最も近い値のうち大きい方を返します。

この関数はv2.5以降でサポートされています。

## 構文

```SQL
PERCENTILE_DISC (expr, percentile) 
```

## パラメーター

- `expr`: パーセンタイル値を計算したい列です。列はソート可能な任意のデータ型である必要があります。
- `percentile`: 見つけたい値のパーセンタイルです。0から1までの定数の浮動小数点数である必要があります。たとえば、中央値を見つけたい場合は、このパラメーターを `0.5` に設定します。70パーセンタイルの値を見つけたい場合は、`0.7` を指定します。

## 戻り値

戻り値のデータ型は `expr` と同じです。

## 使用上の注意

NULL値は計算から除外されます。

## 例

`exam` テーブルを作成し、データを挿入します。

```sql
CREATE TABLE exam (
    subject STRING,
    exam_result INT
) 
DISTRIBUTED BY HASH(`subject`);

insert into exam values
('chemistry',80),
('chemistry',100),
('chemistry',null),
('math',60),
('math',70),
('math',85),
('physics',75),
('physics',80),
('physics',85),
('physics',99);
```

```Plain
select * from exam order by Subject;
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

各科目の中央値を計算します。

```SQL
SELECT Subject, PERCENTILE_DISC (Score, 0.5)
FROM exam group by Subject;
```

出力結果

```Plain
+-----------+-----------------------------+
| Subject   | percentile_disc(Score, 0.5) |
+-----------+-----------------------------+
| chemistry |                         100 |
| math      |                          70 |
| physics   |                          85 |
+-----------+-----------------------------+
```
