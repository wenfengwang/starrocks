---
displayed_sidebar: English
---

# array_remove

## 説明

配列から要素を削除します。

## 構文

```Haskell
array_remove(any_array, any_element)
```

## パラメーター

- `any_array`: 検索される配列。
- `any_element`: 配列内の要素に一致する式。

## 戻り値

指定された要素を削除した配列を返します。

## 例

```plaintext
mysql> SELECT array_remove([1,2,3,NULL,3], 3);

+---------------------------------+

| array_remove([1,2,3,NULL,3], 3) |

+---------------------------------+

| [1,2,NULL]                      |

+---------------------------------+

1行がセットされました (0.01秒)
```

## キーワード

ARRAY_REMOVE, ARRAY
