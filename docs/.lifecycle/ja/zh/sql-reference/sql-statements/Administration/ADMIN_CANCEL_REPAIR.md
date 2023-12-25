---
displayed_sidebar: Chinese
---

# ADMIN CANCEL REPAIR

## 機能

このステートメントは、指定されたテーブルまたはパーティションの高優先度での修復をキャンセルするために使用されます。

:::tip

この操作にはSYSTEMレベルのOPERATE権限が必要です。ユーザーに権限を付与するには [GRANT](../account-management/GRANT.md) を参照してください。

:::

## 文法

```sql
ADMIN CANCEL REPAIR TABLE table_name[ PARTITION (p1,...)]
```

説明：このステートメントは、システムが指定されたテーブルまたはパーティションのシャードレプリカを高優先度で修復しないことを意味します。システムは引き続きデフォルトのスケジュールでレプリカを修復します。

## 例

高優先度の修復をキャンセルします。

```sql
ADMIN CANCEL REPAIR TABLE tbl PARTITION(p1);
```
