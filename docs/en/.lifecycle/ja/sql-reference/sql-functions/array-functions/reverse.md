---
displayed_sidebar: "Japanese"
---

# reverse

## 説明

文字列または配列を逆順にします。文字列または配列の要素の文字を逆の順序で持つ文字列または配列を返します。

## 構文

```Haskell
reverse(param)
```

## パラメータ

`param`: 逆順にする文字列または配列です。VARCHAR、CHAR、またはARRAY型であることができます。

現在、この関数は一次元配列のみをサポートし、配列の要素はDECIMAL型ではない必要があります。この関数は以下のタイプの配列要素をサポートしています：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、DECIMALV2、DATETIME、DATE、およびJSON。**JSONは2.5からサポートされています。**

## 戻り値

戻り値の型は`param`と同じです。

## 例

例1：文字列を逆順にする。

```Plain Text
MySQL > SELECT REVERSE('hello');
+------------------+
| REVERSE('hello') |
+------------------+
| olleh            |
+------------------+
1 row in set (0.00 sec)
```

例2：配列を逆順にする。

```Plain Text
MYSQL> SELECT REVERSE([4,1,5,8]);
+--------------------+
| REVERSE([4,1,5,8]) |
+--------------------+
| [8,5,1,4]          |
+--------------------+
```
