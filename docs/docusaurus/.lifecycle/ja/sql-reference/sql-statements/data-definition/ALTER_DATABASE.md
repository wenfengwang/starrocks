---
displayed_sidebar: "Japanese"
---

# ALTER DATABASE（データベース変更）

## 説明

指定されたデータベースのプロパティを構成します（管理者専用）。

1. B/K/KB/M/MB/G/GB/T/TB/P/PB 単位でデータベースのデータクォータを設定します。

    ```sql
    ALTER DATABASE db_name SET DATA QUOTA quota;
    ```

2. データベース名を変更します

    ```sql
    ALTER DATABASE db_name RENAME new_db_name;
    ```

3. データベースのレプリカクォータを設定します

    ```sql
    ALTER DATABASE db_name SET REPLICA QUOTA quota;
    ```

注：

```plain text
データベース名を変更した後は、必要に応じてREVOKEおよびGRANTコマンドを使用して対応するユーザー権限を変更してください。
データベースのデフォルトデータクォータおよびデフォルトレプリカクォータは2^63-1です。
```

## 例

1. 指定されたデータベースのデータクォータを設定する

    ```SQL
    ALTER DATABASE example_db SET DATA QUOTA 10995116277760;
    -- 上記の単位はバイトであり、以下のステートメントと同等です。
    ALTER DATABASE example_db SET DATA QUOTA 10T;
    ALTER DATABASE example_db SET DATA QUOTA 100G;
    ALTER DATABASE example_db SET DATA QUOTA 200M;
    ```

2. データベースexample_dbをexample_db2に名前を変更する

    ```SQL
    ALTER DATABASE example_db RENAME example_db2;
    ```

3. 指定されたデータベースのレプリカクォータを設定する

    ```SQL
    ALTER DATABASE example_db SET REPLICA QUOTA 102400;
    ```

## 参照

- [CREATE DATABASE（データベースの作成）](CREATE_DATABASE.md)
- [USE（使用）](../data-definition/USE.md)
- [SHOW DATABASES（データベースの表示）](../data-manipulation/SHOW_DATABASES.md)
- [DESC（DESCRIBE）](../Utility/DESCRIBE.md)
- [DROP DATABASE（データベースの削除）](../data-definition/DROP_DATABASE.md)