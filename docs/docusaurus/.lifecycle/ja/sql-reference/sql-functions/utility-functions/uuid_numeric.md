---
displayed_sidebar: "Japanese"
---

# uuid_numeric

## 説明

LARGEINTタイプのランダムUUIDを返します。この関数は、`uuid`関数よりも実行パフォーマンスが2桁優れています。

## 構文

```Haskell
uuid_numeric();
```

## パラメータ

なし

## 戻り値

LARGEINTタイプの値を返します。

## 例

```Plain Text
MySQL > select uuid_numeric();
+--------------------------+
| uuid_numeric()           |
+--------------------------+
| 558712445286367898661205 |
+--------------------------+
1行がセットされました (0.00秒)
```