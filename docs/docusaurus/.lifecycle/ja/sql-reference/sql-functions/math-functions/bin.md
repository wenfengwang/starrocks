---
displayed_sidebar: "Japanese"
---

# bin（バイナリ）

## 説明

入力された `arg` をバイナリに変換します。

## 構文

```Shell
bin(arg)
```

## パラメータ

`arg`: バイナリに変換したい入力です。BIGINTデータ型をサポートしています。

## 返り値

VARCHARデータ型の値を返します。

## 例

```Plain
mysql> select bin(3);
+--------+
| bin(3) |
+--------+
| 11     |
+--------+
1 行が返されました（0.02 秒）
```