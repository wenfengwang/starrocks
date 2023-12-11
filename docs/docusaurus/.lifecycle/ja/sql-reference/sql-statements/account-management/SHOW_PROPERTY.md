---
displayed_sidebar: "Japanese"
---

# プロパティを表示する

## 説明

このステートメントはユーザーのプロパティを表示するために使用されます。

構文:

```sql
SHOW PROPERTY [FOR ユーザー] [LIKE キー]
```

## 例

1. jackユーザーのプロパティを表示する

    ```sql
    SHOW PROPERTY FOR 'jack'
    ```

2. jackユーザーがインポートしたクラスターに関連するプロパティを表示する

    ```sql
    SHOW PROPERTY FOR 'jack' LIKE '%load_cluster%'
    ```