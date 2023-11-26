---
displayed_sidebar: "Japanese"
---

# ceil, dceil

## 説明

入力の `arg` を最も近い等しいか大きい整数に丸めた値を返します。

## 構文

```Shell
ceil(arg)
```

## パラメーター

`arg` は DOUBLE データ型をサポートしています。

## 戻り値

BIGINT データ型の値を返します。

## 例

```Plain
mysql> select ceil(3.14);
+------------+
| ceil(3.14) |
+------------+
|          4 |
+------------+
1 行が返されました (0.15 秒)
```
