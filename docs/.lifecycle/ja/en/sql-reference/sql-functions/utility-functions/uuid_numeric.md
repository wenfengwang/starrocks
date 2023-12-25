---
displayed_sidebar: English
---

# uuid_numeric

## 説明

LARGEINT 型のランダムな UUID を返します。この関数は `uuid` 関数よりも実行性能が2桁優れています。

## 構文

```Haskell
uuid_numeric();
```

## パラメーター

なし

## 戻り値

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
