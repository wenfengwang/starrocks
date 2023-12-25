---
displayed_sidebar: English
---

# ADMIN CHECK TABLET

## 説明

このステートメントは、タブレットのグループをチェックするために使用されます。

:::tip

この操作には、SYSTEM レベルの OPERATE 権限が必要です。[GRANT](../account-management/GRANT.md)の指示に従って、この権限を付与することができます。

:::

## 構文

```sql
ADMIN CHECK TABLET (tablet_id1, tablet_id2, ...)
PROPERTIES("type" = "...")
```

注記：

1. tablet_id と PROPERTIES の "type" プロパティは指定する必要があります。

2. 現在、"type" は以下のみをサポートしています:

   Consistency: タブレットのレプリカの一貫性をチェックします。このコマンドは非同期です。送信後、StarRocks は対応するタブレット間の一貫性のチェックを開始します。最終結果は SHOW PROC "/statistic" の結果にある InconsistentTabletNum 列で表示されます。

## 例

1. 指定されたタブレットのグループのレプリカの一貫性をチェックする

    ```sql
    ADMIN CHECK TABLET (10000, 10001)
    PROPERTIES("type" = "consistency");
    ```
