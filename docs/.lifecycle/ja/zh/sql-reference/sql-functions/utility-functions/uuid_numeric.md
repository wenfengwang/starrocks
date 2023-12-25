---
displayed_sidebar: Chinese
---

# uuid_numeric

## 機能

数値型のランダムな UUID 値を返します。`uuid`関数と比較して、この関数の実行性能は約2桁向上します。

## 文法

```Haskell
uuid_numeric();
```

## パラメータ説明

なし

## 戻り値の説明

LARGEINT 型の値を返します。

## 例

```Plain Text
MySQL > select uuid_numeric();
+--------------------------+
| uuid_numeric()           |
+--------------------------+
| 558712445286367898661205 |
+--------------------------+
1行がセットされました (0.00 秒)
```
