---
displayed_sidebar: English
---

# ALTER DATABASE（データベースの変更）

## 説明

指定されたデータベースのプロパティを設定します。

:::tip

この操作には、対象データベースに対するALTER権限が必要です。この権限を付与するには、[GRANT](../account-management/GRANT.md)の説明に従ってください。

:::

1. データベースのデータクォータをB/K/KB/M/MB/G/GB/T/TB/P/PBの単位で設定します。

    ```sql
    ALTER DATABASE db_name SET DATA QUOTA quota;
    ```

2. データベースの名前を変更

    ```sql
    ALTER DATABASE db_name RENAME TO new_db_name;
    ```

3. データベースのレプリカクォータを設定

    ```sql
    ALTER DATABASE db_name SET REPLICA QUOTA quota;
    ```

注記：

```plain text
データベースの名前を変更した後、必要に応じてREVOKEおよびGRANTコマンドを使用して、対応するユーザー権限を変更してください。
データベースのデフォルトデータクォータおよびデフォルトレプリカクォータは2^63-1です。
```

## 例

1. 指定されたデータベースのデータクォータを設定

    ```SQL
    ALTER DATABASE example_db SET DATA QUOTA 10995116277760;
    -- 上記の単位はバイトで、以下のステートメントと同等です。
    ALTER DATABASE example_db SET DATA QUOTA 10T;
    ALTER DATABASE example_db SET DATA QUOTA 100G;
    ALTER DATABASE example_db SET DATA QUOTA 200M;
    ```

2. データベースexample_dbの名前をexample_db2に変更

    ```SQL
    ALTER DATABASE example_db RENAME TO example_db2;
    ```

3. 指定されたデータベースのレプリカクォータを設定

    ```SQL
    ALTER DATABASE example_db SET REPLICA QUOTA 102400;
    ```

## 参照

- [CREATE DATABASE（データベースの作成）](CREATE_DATABASE.md)
- [USE（使用）](../data-definition/USE.md)
- [SHOW DATABASES（データベースの表示）](../data-manipulation/SHOW_DATABASES.md)
- [DESC（説明）](../Utility/DESCRIBE.md)
- [DROP DATABASE（データベースの削除）](../data-definition/DROP_DATABASE.md)
