---
displayed_sidebar: "Japanese"
---

# ADMIN CHECK TABLET

## Description（説明）

このステートメントは、一連のタブレットをチェックするために使用されます。

構文：

```sql
ADMIN CHECK TABLE (tablet_id1, tablet_id2, ...)
PROPERTIES("type" = "...")
```

注：

1. タブレット ID および PROPERTIES 内の "type" プロパティを指定する必要があります。

2. 現在、"type" は次ののみサポートされています：

   一貫性: タブレットのレプリカの一貫性をチェックします。このコマンドは非同期です。送信した後、StarRocks は対応するタブレット間の一貫性をチェックし始めます。最終結果は "/statistic" の SHOW PROC 結果内の InconsistentTabletNum 列に表示されます。

## Examples（例）

1. 指定されたタブレットのレプリカの一貫性をチェックします

    ```sql
    ADMIN CHECK TABLET (10000, 10001)
    PROPERTIES("type" = "consistency");
    ```