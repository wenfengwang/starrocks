---
displayed_sidebar: English
---

# ceiling

## 説明

入力`arg`から最も近い、もしくはそれ以上の整数に丸められた値を返します。

## 構文

```Shell
ceiling(arg)
```

## パラメーター

`arg` はDOUBLEデータ型をサポートしています。

## 戻り値

BIGINTデータ型の値を返します。

## 例

```Plain
mysql> select ceiling(3.14);
+---------------+
| ceiling(3.14) |
+---------------+
|             4 |
+---------------+
1行がセットされました (0.00 秒)
```
