---
displayed_sidebar: English
---

# ADMIN CANCEL REPAIR

## 説明

このステートメントは、指定されたテーブルやパーティションの高優先度での修復をキャンセルするために使用されます。

:::tip

この操作にはSYSTEMレベルのOPERATE権限が必要です。この権限を付与するには、[GRANT](../account-management/GRANT.md)の説明に従ってください。

:::

## 構文

```sql
ADMIN CANCEL REPAIR TABLE table_name[ PARTITION (p1,...)]
```

注記
>
> このステートメントは、システムが指定されたテーブルやパーティションのシャーディングレプリカを高優先度で修復しなくなることのみを示しています。これらのコピーはデフォルトのスケジューリングによって修復されます。

## 例

1. 高優先度修復のキャンセル

    ```sql
    ADMIN CANCEL REPAIR TABLE tbl PARTITION(p1);
    ```
