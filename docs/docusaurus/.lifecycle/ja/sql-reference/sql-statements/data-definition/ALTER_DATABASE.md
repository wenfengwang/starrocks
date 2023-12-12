---
displayed_sidebar: "Japanese"
---

# ALTER DATABASE（データベースの変更）

## 説明

指定されたデータベースのプロパティを設定します。（管理者のみ）

1. B/K/KB/M/MB/G/GB/T/TB/P/PB 単位でデータベースのデータ割り当てを設定します。

    ```sql
    ALTER DATABASE db_name SET DATA QUOTA quota;
    ```

2. データベースの名前を変更します

    ```sql
    ALTER DATABASE db_name RENAME new_db_name;
    ```

3. データベースのレプリカの割り当てを設定します

    ```sql
    ALTER DATABASE db_name SET REPLICA QUOTA quota;
    ```

注意：

```plain text
データベースの名前を変更したら、必要に応じて対応するユーザーの権限を変更するために REVOKE および GRANT コマンドを使用します。
データベースのデフォルトのデータ割り当ておよびデフォルトのレプリカ割り当ては 2^63-1 です。
```

## 例

1. 指定されたデータベースのデータ割り当てを設定する

    ```SQL
    ALTER DATABASE example_db SET DATA QUOTA 10995116277760;
    -- 上記の単位はバイトで、次のステートメントと同等です。
    ALTER DATABASE example_db SET DATA QUOTA 10T;
    ALTER DATABASE example_db SET DATA QUOTA 100G;
    ALTER DATABASE example_db SET DATA QUOTA 200M;
    ```

2. データベース example_db を example_db2 に名前を変更する

    ```SQL
    ALTER DATABASE example_db RENAME example_db2;
    ```

3. 指定されたデータベースのレプリカ割り当てを設定する

    ```SQL
    ALTER DATABASE example_db SET REPLICA QUOTA 102400;
    ```

## 参照

- [CREATE DATABASE](CREATE_DATABASE.md)
- [USE](../data-definition/USE.md)
- [SHOW DATABASES](../data-manipulation/SHOW_DATABASES.md)
- [DESC](../Utility/DESCRIBE.md)
- [DROP DATABASE](../data-definition/DROP_DATABASE.md)