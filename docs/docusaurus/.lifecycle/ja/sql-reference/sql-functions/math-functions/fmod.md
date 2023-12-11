---
displayed_sidebar: "Japanese"
---

# fmod（浮動小数点数の剰余）

## 説明

除算（ `dividend`/`divisor` ）の浮動小数点数の剰余を返します。これはモジュロ関数です。

## 構文

```SQL
fmod(dividend,devisor);
```

## パラメータ

- `dividend`：DOUBLEまたはFLOATがサポートされています。

- `devisor`：DOUBLEまたはFLOATがサポートされています。

> **注意**
>
> `devisor`のデータ型は`dividend`のデータ型と同じである必要があります。そうでない場合、StarRocksは暗黙の型変換を実行してデータ型を変換します。

## 返り値

出力のデータ型と符号は、`dividend`のデータ型と符号と同じである必要があります。`divisor`が`0`の場合、`NULL`が返されます。

## 例

```Plaintext
mysql> select fmod(3.14,3.14);
+------------------+
| fmod(3.14, 3.14) |
+------------------+
|                0 |
+------------------+

mysql> select fmod(11.5,3);
+---------------+
| fmod(11.5, 3) |
+---------------+
|           2.5 |
+---------------+

mysql> select fmod(3,6);
+------------+
| fmod(3, 6) |
+------------+
|          3 |
+------------+

mysql> select fmod(3,0);
+------------+
| fmod(3, 0) |
+------------+
|       NULL |
+------------+
```