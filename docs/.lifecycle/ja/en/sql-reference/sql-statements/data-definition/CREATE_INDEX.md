---
displayed_sidebar: English
---

# CREATE INDEX

## 説明

このステートメントは、インデックスを作成するために使用されます。

:::tip

この操作には、対象テーブルに対するALTER権限が必要です。この権限を付与するには、[GRANT](../account-management/GRANT.md)の指示に従ってください。

:::

## 構文

```sql
CREATE INDEX index_name ON table_name (column [, ...],) [USING BITMAP] [COMMENT 'balabala']
```

注記：

1. 現在のバージョンではビットマップインデックスのみをサポートしています。
2. BITMAPインデックスは単一の列にのみ作成可能です。

## 例

1. `table1`の`siteid`にビットマップインデックスを作成します。

    ```sql
    CREATE INDEX index_name ON table1 (siteid) USING BITMAP COMMENT 'balabala';
    ```
