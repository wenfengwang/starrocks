---
displayed_sidebar: English
---

# DROP FUNCTION

## 説明

カスタム関数を削除します。関数は、その名前とパラメータータイプが一致している場合にのみ削除できます。

カスタム関数の所有者だけが関数を削除する権限を持っています。

### 構文

```sql
DROP FUNCTION function_name(arg_type [, ...])
```

### パラメーター

`function_name`: 削除される関数の名前です。

`arg_type`: 削除される関数の引数の型です。

## 例

1. 関数を削除する。

    ```sql
    DROP FUNCTION my_add(INT, INT)
    ```
