---
displayed_sidebar: "Japanese"
---

# 天井値を求める

## 説明

入力`arg`から、最も近い等しいか大きな整数に丸めた値を返します。

## 構文

```Shell
ceiling(arg)
```

## パラメーター

`arg`はDOUBLEデータ型をサポートしています。

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
1行 in set (0.00 sec)
```