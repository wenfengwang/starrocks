---
displayed_sidebar: "Japanese"
---

# ceil, dceil

## 説明

入力 `arg` を最も近い整数またはそれ以上の整数に丸めて返します。

## 構文

```Shell
ceil(arg)
```

## パラメータ

`arg` はDOUBLEデータ型をサポートしています。

## 戻り値

BIGINTデータ型の値を返します。

## 例

```Plain
mysql> select ceil(3.14);
+------------+
| ceil(3.14) |
+------------+
|          4 |
+------------+
1 row in set (0.15 sec)
```