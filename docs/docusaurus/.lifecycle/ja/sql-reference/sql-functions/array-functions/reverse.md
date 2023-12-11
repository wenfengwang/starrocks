```yaml
---
displayed_sidebar: "English"
---

# reverse

## Description

文字列または配列を反転します。文字列または配列の要素は逆の順序であり、文字列または配列で返されます。

## Syntax

```Haskell
reverse(param)
```

## Parameters

`param`: 反転する文字列または配列。VARCHAR、CHAR、またはARRAYタイプであることができます。

現時点では、この関数は1次元配列のみをサポートし、配列要素はDECIMALタイプであってはいけません。この関数は次の種類の配列要素をサポートしています：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、DECIMALV2、DATETIME、DATE、およびJSON。**JSONはバージョン2.5からサポートされています。**

## Return value

戻り値の型は`param`と同じです。

## Examples

例1：文字列を反転する。

```Plain Text
MySQL > SELECT REVERSE('hello');
+------------------+
| REVERSE('hello') |
+------------------+
| olleh            |
+------------------+
1 row in set (0.00 sec)
```

例2：配列を反転する。

```Plain Text
MYSQL> SELECT REVERSE([4,1,5,8]);
+--------------------+
| REVERSE([4,1,5,8]) |
+--------------------+
| [8,5,1,4]          |
+--------------------+
```