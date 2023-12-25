---
displayed_sidebar: English
---

# ceil、dceil

## 説明

入力された `arg` を最も近い等しいかそれ以上の整数に丸めた値を返します。

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
1行がセットされました (0.15秒)
```
