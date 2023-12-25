---
displayed_sidebar: Chinese
---

# ADMIN SET CONFIG

## 機能

このステートメントは、クラスタの設定項目を設定するために使用されます（現在はFEの設定項目のみ設定可能です）。

設定可能な項目は、[ADMIN SHOW FRONTEND CONFIG](ADMIN_SHOW_CONFIG.md) コマンドで確認できます。

設定後の項目は、FEが再起動した後に **fe.conf** ファイルの設定またはデフォルト値にリセットされます。設定を永続的に有効にするには、設定後に **fe.conf** ファイルも同時に変更することをお勧めします。これにより、再起動後に変更が無効になるのを防ぎます。

:::tip

この操作にはSYSTEMレベルのOPERATE権限が必要です。ユーザーに権限を付与するには [GRANT](../account-management/GRANT.md) を参照してください。

:::

## 文法

```sql
ADMIN SET FRONTEND CONFIG ("key" = "value")
```

## 例

`tablet_sched_disable_balance` を `true` に設定します。

```sql
 ADMIN SET FRONTEND CONFIG ("tablet_sched_disable_balance" = "true");
```
