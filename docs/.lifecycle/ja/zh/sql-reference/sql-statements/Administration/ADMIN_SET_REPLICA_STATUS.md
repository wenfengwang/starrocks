---
displayed_sidebar: Chinese
---

# ADMIN SET REPLICA STATUS

## 機能

このステートメントは、指定されたレプリカの状態を設定するために使用されます。

このコマンドは現在、特定のレプリカの状態を手動で bad または ok に設定し、システムがこれらのレプリカを自動的に修復できるようにするためにのみ使用されます。

:::tip

この操作には SYSTEM レベルの OPERATE 権限が必要です。ユーザーに権限を付与するには [GRANT](../account-management/GRANT.md) を参照してください。

:::

## 文法

```sql
ADMIN SET REPLICA STATUS
PROPERTIES ("key" = "value", ...)
```

現在、以下のプロパティがサポートされています：

**tablet_id**：必須。Tablet Id を指定します。

**backend_id**：必須。Backend Id を指定します。

**status**：必須。状態を指定します。現在、"bad" または "ok" のみがサポートされています。

指定されたレプリカが存在しない場合、または状態が既に bad である場合は、無視されます。

**注意：**
bad 状態に設定されたレプリカは直ちに削除される可能性があるため、慎重に操作してください。

## 例

1. BE 10001 上の tablet 10003 のレプリカの状態を bad に設定します。

    ```sql
    ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");
    ```

2. BE 10001 上の tablet 10003 のレプリカの状態を ok に設定します。

    ```sql
    ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "ok");
    ```
