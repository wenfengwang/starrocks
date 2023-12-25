---
displayed_sidebar: Chinese
---

# ALTER DATABASE（データベースの変更）

## 機能

このステートメントは、指定されたデータベースの属性を設定するために使用されます。

:::tip

この操作には、該当するデータベースのALTER権限が必要です。ユーザーに権限を付与するには、[GRANT](../account-management/GRANT.md) を参照してください。

:::

## 文法

### データベースのデータ量クォータを設定

単位は B/K/KB/M/MB/G/GB/T/TB/P/PB です。

```sql
ALTER DATABASE <db_name> SET DATA QUOTA <quota>
```

### データベースの名前を変更

```sql
ALTER DATABASE <db_name> RENAME TO <new_db_name>
```

### データベースのレプリカ数クォータを設定

```sql
ALTER DATABASE <db_name> SET REPLICA QUOTA <quota>
```

説明：

```plain text
データベースの名前を変更した後、必要に応じて、REVOKE および GRANT コマンドを使用して、関連するユーザー権限を変更してください。
データベースのデフォルトのデータ量クォータとデフォルトのレプリカ数クォータは両方とも 2^63 - 1 です。
```

## 例

1. 指定したデータベースのデータ量クォータを設定

    ```SQL
    ALTER DATABASE example_db SET DATA QUOTA 10995116277760;
    -- 上記の単位はバイトで、以下のステートメントと同等です
    ALTER DATABASE example_db SET DATA QUOTA 10T;
    ALTER DATABASE example_db SET DATA QUOTA 100G;
    ALTER DATABASE example_db SET DATA QUOTA 200M;
    ```

2. データベース example_db を example_db2 に名前を変更

    ```SQL
    ALTER DATABASE example_db RENAME TO example_db2;
    ```

3. 指定したデータベースのレプリカ数クォータを設定

    ```SQL
    ALTER DATABASE example_db SET REPLICA QUOTA 102400;
    ```

## 関連参照

- [CREATE DATABASE（データベースの作成）](CREATE_DATABASE.md)
- [USE（使用）](../data-definition/USE.md)
- [SHOW DATABASES（データベースを表示）](../data-manipulation/SHOW_DATABASES.md)
- [DESC（説明）](../Utility/DESCRIBE.md)
- [DROP DATABASE（データベースの削除）](../data-definition/DROP_DATABASE.md)
