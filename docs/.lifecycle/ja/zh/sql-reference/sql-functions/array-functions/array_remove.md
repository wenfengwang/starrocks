---
displayed_sidebar: Chinese
---

# array_remove

## 機能

配列から特定の要素を削除します。

## 文法

```Haskell
array_remove(any_array, any_element)
```

## パラメータ説明

* any_array: 対象の配列
* any_element: 削除する必要がある要素

## 戻り値の説明

削除された要素を除いた配列を返します。

## 例

```plain text
mysql> select array_remove([1,2,3,null,3], 3);
+---------------------------------+
| array_remove([1,2,3,NULL,3], 3) |
+---------------------------------+
| [1,2,null]                      |
+---------------------------------+
1行がセットされました (0.01秒)
```
