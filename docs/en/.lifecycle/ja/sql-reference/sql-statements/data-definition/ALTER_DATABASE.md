---
displayed_sidebar: "Japanese"
---

# ALTER DATABASE（データベースの変更）

## 説明

指定されたデータベースのプロパティを設定します。（管理者のみ）

1. データベースのデータクォータをB/K/KB/M/MB/G/GB/T/TB/P/PB単位で設定します。

    ```sql
    ALTER DATABASE データベース名 SET DATA QUOTA クォータ;
    ```

2. データベースの名前を変更します。

    ```sql
    ALTER DATABASE データベース名 RENAME 新しいデータベース名;
    ```

3. データベースのレプリカクォータを設定します。

    ```sql
    ALTER DATABASE データベース名 SET REPLICA QUOTA クォータ;
    ```

注意：

```plain text
データベースの名前を変更した後は、必要に応じてREVOKEコマンドとGRANTコマンドを使用して対応するユーザーの権限を変更してください。
データベースのデフォルトのデータクォータとデフォルトのレプリカクォータは2^63-1です。
```

## 例

1. 指定されたデータベースのデータクォータを設定します。

    ```SQL
    ALTER DATABASE example_db SET DATA QUOTA 10995116277760;
    -- 上記の単位はバイトで、以下の文と同じです。
    ALTER DATABASE example_db SET DATA QUOTA 10T;
    ALTER DATABASE example_db SET DATA QUOTA 100G;
    ALTER DATABASE example_db SET DATA QUOTA 200M;
    ```

2. データベース example_db を example_db2 に名前を変更します。

    ```SQL
    ALTER DATABASE example_db RENAME example_db2;
    ```

3. 指定されたデータベースのレプリカクォータを設定します。

    ```SQL
    ALTER DATABASE example_db SET REPLICA QUOTA 102400;
    ```

## 参照

- [CREATE DATABASE（データベースの作成）](CREATE_DATABASE.md)
- [USE（データベースの使用）](../data-definition/USE.md)
- [SHOW DATABASES（データベースの表示）](../data-manipulation/SHOW_DATABASES.md)
- [DESC（テーブルの詳細情報の表示）](../Utility/DESCRIBE.md)
- [DROP DATABASE（データベースの削除）](../data-definition/DROP_DATABASE.md)
