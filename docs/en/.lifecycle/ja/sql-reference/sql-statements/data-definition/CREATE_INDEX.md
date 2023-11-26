---
displayed_sidebar: "Japanese"
---

# CREATE INDEX（インデックスの作成）

## 説明

このステートメントは、インデックスを作成するために使用されます。

構文:

```sql
CREATE INDEX インデックス名 ON テーブル名 (カラム [, ...],) [USING BITMAP] [COMMENT 'balabala']
```

注意:

1. 現在のバージョンでは、ビットマップインデックスのみをサポートしています。
2. シングルカラムのみでBITMAPインデックスを作成します。

## 例

1. `table1` の `siteid` に対してビットマップインデックスを作成します。

    ```sql
    CREATE INDEX インデックス名 ON table1 (siteid) USING BITMAP COMMENT 'balabala';
    ```
