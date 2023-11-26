---
displayed_sidebar: "Japanese"
---

# uuid_numeric

## 説明

LARGEINT型のランダムなUUIDを返します。この関数は、`uuid`関数よりも実行パフォーマンスが2桁優れています。

## 構文

```Haskell
uuid_numeric();
```

## パラメーター

なし

## 戻り値

LARGEINT型の値を返します。

## 例

```Plain Text
MySQL > select uuid_numeric();
+--------------------------+
| uuid_numeric()           |
+--------------------------+
| 558712445286367898661205 |
+--------------------------+
1 行が返されました (0.00 秒)
```
