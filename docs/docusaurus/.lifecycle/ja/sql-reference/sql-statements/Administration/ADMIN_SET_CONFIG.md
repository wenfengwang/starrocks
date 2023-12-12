---
displayed_sidebar: "Japanese"
---

# ADMIN SET CONFIG（管理セット構成）

## 説明

この文は、クラスタの構成項目を設定するために使用されます（現在、このコマンドを使用してFEダイナミック構成項目のみを設定できます）。これらの構成項目は、[ADMIN SHOW FRONTEND CONFIG](ADMIN_SET_CONFIG.md)コマンドを使用して表示できます。

構成はFEの再起動後に`fe.conf`ファイルのデフォルト値に復元されます。したがって、変更が失われないように、`fe.conf`内の構成項目も変更することをお勧めします。

## 構文

```sql
ADMIN SET FRONTEND CONFIG（"key" = "value"）
```

## 例

1. `disable_balance`を`true`に設定します。

    ```sql
    ADMIN SET FRONTEND CONFIG（"disable_balance" = "true"）;
    ```