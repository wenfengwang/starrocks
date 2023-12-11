```markdown
---
displayed_sidebar: "Japanese"
---

# インデックスの作成

## 説明

このステートメントはインデックスを作成するために使用されます。

構文:

```sql
CREATE INDEX インデックス名 ON テーブル名 (列 [, ...],) [BITMAPを使用] [COMMENT 'バラバラ']
```

注:

1. 現在のバージョンでは、ビットマップインデックスのみがサポートされています。
2. 単一の列にのみBITMAPインデックスを作成できます。

## 例

1. `table1` の `siteid` にビットマップインデックスを作成します。

    ```sql
    CREATE INDEX インデックス名 ON table1 (siteid) BITMAPを使用 COMMENT 'バラバラ';
    ```