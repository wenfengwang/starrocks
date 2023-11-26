---
displayed_sidebar: "Japanese"
---

# array_append

## 説明

配列の末尾に新しい要素を追加します。配列を返します。

## 構文

```Haskell
array_append(any_array, any_element)
```

## 例

```plain text
mysql> select array_append([1, 2], 3);
+------------------------+
| array_append([1,2], 3) |
+------------------------+
| [1,2,3]                |
+------------------------+
1 行が返されました (0.00 秒)

```

配列にNULLを追加することもできます。

```plain text
mysql> select array_append([1, 2], NULL);
+---------------------------+
| array_append([1,2], NULL) |
+---------------------------+
| [1,2,NULL]                |
+---------------------------+
1 行が返されました (0.01 秒)

```

## キーワード

ARRAY_APPEND,ARRAY
