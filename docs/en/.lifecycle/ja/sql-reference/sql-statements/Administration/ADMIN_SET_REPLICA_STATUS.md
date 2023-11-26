---
displayed_sidebar: "Japanese"
---

# ADMIN SET REPLICA STATUS（レプリカのステータスを設定する）

## 説明

このステートメントは、指定されたレプリカのステータスを設定するために使用されます。

このコマンドは現在、一部のレプリカのステータスを手動でBADまたはOKに設定し、システムがこれらのレプリカを自動的に修復することを許可するためにのみ使用されます。

構文：

```sql
ADMIN SET REPLICA STATUS
PROPERTIES ("key" = "value", ...)
```

現在サポートされている属性は次のとおりです：

"table_id"：必須。Tablet IDを指定します。

"backend_id"：必須。Backend IDを指定します。

"status"：必須。ステータスを指定します。現在、"bad"と"ok"のみがサポートされています。

指定されたレプリカが存在しないか、そのステータスが悪い場合、レプリカは無視されます。

注意：

Badステータスに設定されたレプリカはすぐに削除される場合がありますので、注意してください。

## 例

1. BE 10001のテーブル10003のレプリカステータスをbadに設定します。

    ```sql
    ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");
    ```

2. BE 10001のテーブル10003のレプリカステータスをokに設定します。

    ```sql
    ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "ok");
    ```
