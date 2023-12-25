---
displayed_sidebar: English
---

# bin

## 説明

入力された `arg` をバイナリに変換します。

## 構文

```Shell
bin(arg)
```

## パラメータ

`arg`: バイナリに変換したい入力です。BIGINT データ型に対応しています。

## 戻り値

VARCHAR データ型の値を返します。

## 例

```Plain
mysql> select bin(3);
+--------+
| bin(3) |
+--------+
| 11     |
+--------+
1 row in set (0.02 sec)
```
