---
displayed_sidebar: "Japanese"
---

# reverse

## 説明

文字列または配列を逆にします。文字列または配列要素の文字を逆の順番に持つ文字列または配列を返します。

## 構文

```Haskell
reverse(param)
```

## パラメーター

`param`: 逆にする文字列または配列。VARCHAR、CHAR、またはARRAY型であることができます。

現在、この関数は1次元配列のみをサポートし、配列要素はDECIMAL型であってはなりません。この関数は次の種類の配列要素をサポートしています: BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、VARCHAR、DECIMALV2、DATETIME、DATE、そしてJSON。**JSONは2.5からサポートされています。**

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
1行が選択されました (0.00 秒)
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