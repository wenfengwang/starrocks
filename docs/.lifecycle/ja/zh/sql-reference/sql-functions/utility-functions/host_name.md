---
displayed_sidebar: Chinese
---

# host_name

## 機能

計算が行われるノードのホスト名を取得します。この関数はバージョン2.5からサポートされています。

## 文法

```Haskell
host_name();
```

## パラメータ説明

なし

## 戻り値の説明

戻り値のデータ型は VARCHAR です。

## 例

```Plain Text
mysql> select host_name();
+-------------+
| host_name() |
+-------------+
| sandbox-sql |
+-------------+
1行がセットされました (0.01秒)
```
