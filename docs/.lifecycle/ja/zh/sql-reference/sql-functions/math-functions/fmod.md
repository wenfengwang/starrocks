---
displayed_sidebar: Chinese
---

# fmod

## 機能

浮動小数点数の剰余を返す取模関数です。

以下の構文に従って、`dividend` を `divisor` で割った後の浮動小数点数の剰余を返します。

## 構文

```Haskell
fmod(dividend, divisor);
```

## 引数説明

`dividend`: 被除数で、サポートされるデータ型は DOUBLE、FLOAT です。

`divisor`: 除数で、サポートされるデータ型は DOUBLE、FLOAT です。

> `dividend` と `divisor` のデータ型は一致している必要があります。一致していない場合は、暗黙の型変換が行われます。

## 戻り値の説明

戻り値のデータ型と符号は `dividend` と同じです。`divisor` が 0 の場合は、NULL を返します。

## 例

```Plain
mysql> select fmod(3.14, 3.14);
+------------------+
| fmod(3.14, 3.14) |
+------------------+
|                0 |
+------------------+

mysql> select fmod(11.5, 3);
+---------------+
| fmod(11.5, 3) |
+---------------+
|           2.5 |
+---------------+

mysql> select fmod(3, 6);
+------------+
| fmod(3, 6) |
+------------+
|          3 |
+------------+

mysql> select fmod(3, 0);
+------------+
| fmod(3, 0) |
+------------+
|       NULL |
+------------+
```
