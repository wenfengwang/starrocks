---
displayed_sidebar: English
---

# percentile_disc

## 説明

`expr`の入力列の離散分布に基づいて百分位数の値を返します。正確なパーセンタイル値が見つからない場合、この関数は最も近い2つの値のうち大きい方の値を返します。

この関数はv2.5以降でサポートされています。

## 構文

```SQL
PERCENTILE_DISC (expr, percentile) 
```

## パラメーター

- `expr`: パーセンタイル値を計算する列。列は、ソート可能な任意のデータ型である必要があります。
- `percentile`: 検索する値のパーセンタイル。これは、0から1までの定数浮動小数点数でなければなりません。たとえば、中央値を求める場合は、このパラメーターを`0.5`に設定します。70パーセンタイルで値を検索する場合は、0.7を指定します。

## 戻り値

戻り値のデータ型は`expr`と同じです。

## 使用上の注意

NULL値は計算で無視されます。

## 例

テーブル`exam`を作成し、このテーブルにデータを挿入します。

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
+-----------+-------------+
| subject   | exam_result |
+-----------+-------------+
| chemistry |          80 |
| chemistry |         100 |
| chemistry |        NULL |
| math      |          60 |
| math      |          70 |
| math      |          85 |
| physics   |          75 |
| physics   |          80 |
| physics   |          85 |
| physics   |          99 |
+-----------+-------------+
```

各科目の中央値を計算します。

```SQL
SELECT subject, PERCENTILE_DISC (exam_result, 0.5)
FROM exam GROUP BY subject;
```

出力

```Plain
+-----------+---------------------------------+
| subject   | PERCENTILE_DISC(exam_result, 0.5) |
+-----------+---------------------------------+
| chemistry |                              100 |
| math      |                               70 |
| physics   |                               85 |
+-----------+---------------------------------+
```
