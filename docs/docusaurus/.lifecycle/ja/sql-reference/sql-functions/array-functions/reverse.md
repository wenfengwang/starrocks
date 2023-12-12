---
displayed_sidebar: "Japanese"
---

# 逆

## 説明

指定された文字列や配列を逆順にします。指定した文字列や配列の要素を逆の順番で含む文字列や配列を返します。

## 構文

```Haskell
reverse(param)
```

## パラメータ

`param`: 逆にする文字列や配列。VARCHAR、CHAR、またはARRAYの型になります。

現在、この関数は一次元の配列のみをサポートしており、配列の要素はDECIMAL型であってはなりません。この関数は次の種類の配列の要素をサポートしています: BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、DECIMALV2、DATETIME、DATE、およびJSON。**JSONはバージョン2.5からサポートされています。**

## 戻り値

戻り値の型は`param`と同じです。

## 例

例1: 文字列を逆にする。

```Plain Text
MySQL > SELECT REVERSE('hello');
+------------------+
| REVERSE('hello') |
+------------------+
| olleh            |
+------------------+
1 row in set (0.00 sec)
```

例2: 配列を逆にする。

```Plain Text
MYSQL> SELECT REVERSE([4,1,5,8]);
+--------------------+
| REVERSE([4,1,5,8]) |
+--------------------+
| [8,5,1,4]          |
+--------------------+
```