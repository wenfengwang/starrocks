---
displayed_sidebar: Chinese
---

# like

## 機能

文字列 `expr` が指定されたパターン `pattern` に**あいまいマッチ**するかどうかを判断し、マッチすれば 1 を、しなければ 0 を返します。LIKE は通常 `%` や `_` と組み合わせて使用され、`%` は 0 個以上の任意の文字を、`_` は任意の単一文字を表します。

## 文法

```Haskell
BOOLEAN like(VARCHAR expr, VARCHAR pattern);
```

## パラメータ説明

`expr`: 対象の文字列で、サポートされるデータ型は VARCHAR です。

`pattern`: マッチさせる必要がある文字列のパターンで、サポートされるデータ型は VARCHAR です。

## 戻り値の説明

戻り値のデータ型は BOOLEAN です。

## 例

```Plain Text

mysql> select like("star","star");
+----------------------+
| like('star', 'star') |
+----------------------+
|                    1 |
+----------------------+


mysql> select like("starrocks","star%");
+----------------------+
| like('starrocks', 'star%') |
+----------------------+
|                    1 |
+----------------------+
1 row in set (0.00 sec)
```
