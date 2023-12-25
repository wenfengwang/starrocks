---
displayed_sidebar: English
---

# percentile_cont

## 説明

線形補間を用いて`expr`の百分位数値を計算します。

## 構文

```Haskell
PERCENTILE_CONT (expr, percentile) 
```

## パラメーター

- `expr`: 値を順序付けるための式。数値データ型、DATE、またはDATETIMEでなければなりません。例えば、物理の中央値スコアを見つけたい場合は、物理のスコアが含まれている列を指定します。

- `percentile`: 見つけたい値のパーセンタイルです。0から1までの定数浮動小数点数である必要があります。例えば、中央値を見つけたい場合は、このパラメーターを`0.5`に設定します。

## 戻り値

指定されたパーセンタイルにある値を返します。目的のパーセンタイルに正確に一致する入力値がない場合、結果は最も近い2つの入力値の線形補間を使用して計算されます。

データ型は`expr`と同じです。

## 使用上の注意

この関数はNULLを無視します。

## 例

以下のデータを持つ`exam`という名前のテーブルがあるとします。

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

NULLを無視して各科目のスコアの中央値を計算します。

クエリ：

```SQL
SELECT Subject, PERCENTILE_CONT (Score, 0.5) FROM exam GROUP BY Subject;
```

結果：

```Plain
+-----------+-----------------------------+
| Subject   | percentile_cont(Score, 0.5) |
+-----------+-----------------------------+
| chemistry |                          90 |
| math      |                          70 |
| physics   |                        82.5 |
+-----------+-----------------------------+
```
