---
displayed_sidebar: "Japanese"
---

# 関数の削除

## 説明

カスタム関数を削除します。関数はその名前とパラメータタイプが一致している場合のみ削除できます。

カスタム関数の所有者だけが関数を削除する権限を持っています。

### 構文

```sql
DROP FUNCTION 関数名(arg_type [, ...])
```

### パラメータ

`関数名`: 削除する関数の名前。

`arg_type`: 削除する関数の引数の型。

## 例

1. 関数を削除する。

    ```sql
    DROP FUNCTION my_add(INT, INT)
    ```