---
displayed_sidebar: Chinese
---

# percentile_cont

## 機能

正確なパーセンタイルを計算します。この関数は連続分布モデルを使用し、パーセンタイルに完全に一致する値が見つからない場合は、隣接する2つの値の線形補間を返します。

この関数はバージョン2.4からサポートされています。

## 構文

```SQL
PERCENTILE_CONT (expr, percentile) 
```

## パラメータ説明

- `expr`: パーセンタイルを計算する列で、列の値は数値型、DATEまたはDATETIME型でなければなりません。物理学(physics)のスコアの中央値を計算する場合は、`expr`には物理学のスコアを含む列を指定します。

- `percentile`: 指定されたパーセンタイルで、0と1の間の浮動小数点定数です。中央値を計算する場合は0.5に設定します。

## 戻り値の説明

指定されたパーセンタイルに対応する値を返します。パーセンタイルに完全に一致する値が見つからない場合は、隣接する2つの数値の線形補間を返します。

戻り値の型は`expr`のデータ型と同じです。

## 使用説明

この関数はNull値を無視します。

## 例

`exam`という表があり、そのデータは以下の通りです。

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

各科目のスコアの中央値を計算し、Null値は無視します。

クエリ文：

```SQL
SELECT Subject, PERCENTILE_CONT (Score, 0.5)  FROM exam group by Subject;
```

結果を返す：

```Plain
+-----------+-----------------------------+
| Subject   | percentile_cont(Score, 0.5) |
+-----------+-----------------------------+
| chemistry |                          90 |
| math      |                          70 |
| physics   |                        82.5 |
+-----------+-----------------------------+
```
