---
displayed_sidebar: "Japanese"
---

# ADMIN SET CONFIG

## 説明

このステートメントは、クラスタの設定項目を設定するために使用されます（現在、このコマンドを使用してFEダイナミック設定項目のみを設定できます）。これらの設定項目は、[ADMIN SHOW FRONTEND CONFIG](ADMIN_SET_CONFIG.md)コマンドを使用して表示することができます。

設定は、FEの再起動後に`fe.conf`ファイルのデフォルト値に復元されます。そのため、変更の損失を防ぐために、`fe.conf`の設定項目も変更することをおすすめします。

## 構文

```sql
ADMIN SET FRONTEND CONFIG ("key" = "value")
```

## 例

1. `disable_balance`を`true`に設定します。

    ```sql
    ADMIN SET FRONTEND CONFIG ("disable_balance" = "true");
    ```
