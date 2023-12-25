---
displayed_sidebar: English
---

# ADMIN SET REPLICA STATUS

## 説明

このステートメントは、指定されたレプリカのステータスを設定するために使用されます。

このコマンドは現在、一部のレプリカのステータスをBADまたはOKに手動で設定し、システムによるこれらのレプリカの自動修復を可能にするためだけに使用されます。

:::tip

この操作にはSYSTEMレベルのOPERATE権限が必要です。[GRANT](../account-management/GRANT.md)の説明に従ってこの権限を付与することができます。

:::

## 構文

```sql
ADMIN SET REPLICA STATUS
PROPERTIES ("key" = "value", ...)
```

現在サポートされている属性は以下の通りです:

"tablet_id": 必須。タブレットIDを指定します。

"backend_id": 必須。バックエンドIDを指定します。

"status": 必須。ステータスを指定します。現在、"bad" と "ok" のみがサポートされています。

指定されたレプリカが存在しない、またはそのステータスがbadである場合、そのレプリカは無視されます。

注意:

Badステータスに設定されたレプリカは直ちに削除される可能性がありますので、慎重に操作してください。

## 例

1. BE 10001上のタブレット10003のレプリカステータスをbadに設定します。

    ```sql
    ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");
    ```

2. BE 10001上のタブレット10003のレプリカステータスをokに設定します。

    ```sql
    ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "ok");
    ```
