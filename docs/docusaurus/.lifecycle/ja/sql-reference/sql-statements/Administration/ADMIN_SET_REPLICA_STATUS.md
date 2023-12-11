---
displayed_sidebar: "Japanese"
---

# 管理者 セット レプリカ ステータス

## 説明

このステートメントは、指定されたレプリカのステータスを設定するために使用されます。

現在、このコマンドは手動でいくつかのレプリカのステータスをBADまたはOKに設定するためにのみ使用され、システムにこれらのレプリカを自動的に修復させます。

構文:

```sql
ADMIN SET REPLICA STATUS
PROPERTIES ("key" = "value", ...)
```

現在サポートされている次の属性があります:

"tablet_id": 必須。Tablet ID を指定します。

"backend_id": 必須。Backend ID を指定します。

"status": 必須。ステータスを指定します。現在、"bad" または "ok" のみがサポートされています。

指定されたレプリカが存在しないか、そのステータスが悪い場合、そのレプリカは無視されます。

注意:

悪いステータスに設定されたレプリカは即座に削除される可能性がありますので、ご注意ください。

## 例

1. BE 10001 のテーブル 10003 のレプリカのステータスをbadに設定します。

    ```sql
    ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");
    ```

2. BE 10001 のテーブル 10003 のレプリカのステータスをokに設定します。

    ```sql
    ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "ok");
    ```