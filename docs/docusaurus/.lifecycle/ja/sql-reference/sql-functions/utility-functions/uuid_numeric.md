---
displayed_sidebar: "Japanese"
---

# uuid_numeric

## 説明

LARGEINT型のランダムUUIDを返します。この関数は、`uuid`関数よりも実行パフォーマンスが2桁優れています。

## 構文

```Haskell
uuid_numeric();
```

## パラメータ

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
1 行が設定されました (0.00 秒)
```