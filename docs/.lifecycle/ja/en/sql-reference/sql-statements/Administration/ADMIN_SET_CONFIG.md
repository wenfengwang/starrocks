---
displayed_sidebar: English
---

# ADMIN SET CONFIG

## 説明

このステートメントは、クラスタの設定項目を設定するために使用されます（現在、このコマンドを使用して設定できるのはFEの動的設定項目のみです）。これらの設定項目は、[ADMIN SHOW FRONTEND CONFIG](ADMIN_SET_CONFIG.md) コマンドを使用して表示できます。

設定は、FEが再起動した後に `fe.conf` ファイルのデフォルト値にリセットされます。そのため、変更が失われないように、`fe.conf` ファイル内の設定項目も変更することをお勧めします。

:::tip

この操作には、SYSTEMレベルのOPERATE権限が必要です。[GRANT](../account-management/GRANT.md) の指示に従って、この権限を付与することができます。

:::

## 構文

```sql
ADMIN SET FRONTEND CONFIG ("key" = "value")
```

## 例

1. `disable_balance` を `true` に設定します。

    ```sql
    ADMIN SET FRONTEND CONFIG ("disable_balance" = "true");
    ```
