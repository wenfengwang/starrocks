---
displayed_sidebar: "Japanese"
---

# uuid

## 説明

VARCHAR型のランダムなUUIDを返します。この関数を2回呼び出すと、2つの異なる数値が生成されます。UUIDは36文字の長さであり、aaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeeeの形式で4つのハイフンで接続された5つの16進数を含んでいます。

## 構文

```Haskell
uuid();
```

## パラメーター

なし

## 戻り値

VARCHAR型の値を返します。

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
