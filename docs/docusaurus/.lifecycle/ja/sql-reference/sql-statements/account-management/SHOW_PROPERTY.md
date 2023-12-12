```yaml
---
displayed_sidebar: "Japanese"
---

# プロパティを表示

## 説明

このステートメントは、ユーザーのプロパティを表示するために使用されます。

構文:

```sql
SHOW PROPERTY [FOR ユーザー] [LIKE キー]
```

## 例

1. jackユーザーのプロパティを表示する

    ```sql
    SHOW PROPERTY FOR 'jack'
    ```

2. Jackユーザーがインポートしたクラスタに関連するプロパティを表示する

    ```sql
    SHOW PROPERTY FOR 'jack' LIKE '%load_cluster%'
    ```
```