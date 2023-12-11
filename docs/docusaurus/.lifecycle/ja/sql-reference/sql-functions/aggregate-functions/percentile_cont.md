---
displayed_sidebar: "Japanese"
---

# percentile_cont

## 説明

`expr` のパーセンタイル値を線形補間で計算します。

## 構文

```Haskell
PERCENTILE_CONT (expr, percentile) 
```

## パラメータ

- `expr`: 値を順序付けるための式。数値データ型、DATE、またはDATETIMEでなければなりません。たとえば、物理学の中央値スコアを見つけたい場合、物理スコアを含む列を指定します。

- `percentile`: 見つけたい値のパーセンタイル。0から1までの定数の浮動小数点数です。たとえば、中央値を見つけたい場合は、このパラメータを`0.5`に設定します。

## 戻り値

指定されたパーセンタイルにある値を返します。望ましいパーセンタイルにちょうど入力値がない場合、2つの隣接する入力値の線形補間で結果が計算されます。

データ型は`expr`と同じです。

## 使用上の注意

この関数はNULLを無視します。

## 例

次のデータが含まれている `exam` という名前のテーブルがあるとします。

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

NULLを無視して各科目の中央値スコアを計算します。

クエリ:

```SQL
SELECT Subject, PERCENTILE_CONT (Score, 0.5)  FROM exam group by Subject;
```

結果:

```Plain
+-----------+-----------------------------+
| Subject   | percentile_cont(Score, 0.5) |
+-----------+-----------------------------+
| chemistry |                          90 |
| math      |                          70 |
| physics   |                        82.5 |
+-----------+-----------------------------+
```