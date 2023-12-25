---
displayed_sidebar: Chinese
---

# array_append

## 機能

配列の末尾に新しい要素を追加します。ARRAY 型の値を返します。

## 文法

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
1行がセットされました (0.00 秒)

```

配列に NULL を追加することもできます。

```plain text
mysql> select array_append([1, 2], NULL);
+---------------------------+
| array_append([1,2], NULL) |
+---------------------------+
| [1,2,NULL]                |
+---------------------------+
1行がセットされました (0.01 秒)

```
