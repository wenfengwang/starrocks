---
displayed_sidebar: "Japanese"
---

# ADMIN SET CONFIG（管理設定の構成）

## 説明

このステートメントは、クラスターの構成項目を設定するために使用されます（現在、このコマンドを使用して FE 動的構成項目のみを設定できます）。これらの構成項目は、[ADMIN SHOW FRONTEND CONFIG](ADMIN_SET_CONFIG.md) コマンドを使用して表示することができます。

構成は、FE が再起動すると `fe.conf` ファイルでデフォルト値に復元されます。そのため、変更が失われないように `fe.conf` 内の構成項目も変更することをお勧めします。

## 構文

```sql
ADMIN SET FRONTEND CONFIG ("key" = "value")
```

## 例

1. `disable_balance` を `true` に設定します。

    ```sql
    ADMIN SET FRONTEND CONFIG ("disable_balance" = "true");
    ```