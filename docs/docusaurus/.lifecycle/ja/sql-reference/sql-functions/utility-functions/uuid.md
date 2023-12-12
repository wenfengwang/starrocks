---
displayed_sidebar: "Japanese"
---

# uuid

## 説明

VARCHARタイプのランダムなUUIDを返します。この関数の2回の呼び出しでは、2つの異なる番号が生成される場合があります。UUIDは36文字の長さです。 aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee形式で4つのハイフンで接続された5つの16進数を含んでいます。

## 構文

```Haskell
uuid();
```

## パラメーター

なし

## 戻り値

VARCHARタイプの値を返します。

## 例

```Plain Text
mysql> select uuid();
+--------------------------------------+
| uuid()                               |
+--------------------------------------+
| 74a2ed19-9d21-4a99-a67b-aa5545f26454 |
+--------------------------------------+
1 row in set (0.01 sec)
```