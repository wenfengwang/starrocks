---
displayed_sidebar: "Japanese"
---

# 天井

## 説明

入力の `arg` から最も近い等しいかそれ以上の整数に丸めた値を返します。

## 構文

```Shell
ceiling(arg)
```

## パラメーター

`arg` は DOUBLE データ型をサポートしています。

## 戻り値

BIGINT データ型の値を返します。

## 例

```Plain
mysql> select ceiling(3.14);
+---------------+
| ceiling(3.14) |
+---------------+
|             4 |
+---------------+
1 行が選択されました (0.00 秒)
```