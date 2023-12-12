---
displayed_sidebar: "Japanese"
---

# 管理セット レプリカの状態

## 説明

このステートメントは、指定されたレプリカの状態を設定するために使用されます。

このコマンドは現在、手動で一部のレプリカの状態をBADまたはOKに設定し、システムがこれらのレプリカを自動的に修復することを許可するためにのみ使用されます。

構文:

```sql
ADMIN SET REPLICA STATUS
PROPERTIES ("key" = "value", ...)
```

現在サポートされている属性は次のとおりです:

"table_id'：必須。Tablet IDを指定します。

"backend_id"：必須。Backend IDを指定します。

"status"：必須。ステータスを指定します。現在、"bad"と"ok"のみがサポートされています。

指定されたレプリカが存在しないか、そのステータスが悪い場合、レプリカは無視されます。

注:

悪い状態に設定されたレプリカはすぐに削除される場合がありますので、注意して操作してください。

## 例

1. BE 10001のテーブル10003のレプリカの状態をbadに設定します。

    ```sql
    ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");
    ```

2. BE 10001のテーブル10003のレプリカの状態をokに設定します。

    ```sql
    ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "ok");
    ```