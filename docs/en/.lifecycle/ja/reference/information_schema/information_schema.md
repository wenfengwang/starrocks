---
displayed_sidebar: "Japanese"
---

# インフォメーションスキーマ

StarRocksのインフォメーションスキーマは、各StarRocksインスタンス内のデータベースです。インフォメーションスキーマには、StarRocksインスタンスが保持するすべてのオブジェクトの詳細なメタデータ情報を格納するいくつかの読み取り専用のシステム定義ビューが含まれています。StarRocksのインフォメーションスキーマは、SQL-92 ANSIインフォメーションスキーマを基にしていますが、StarRocks固有のビューと関数が追加されています。

## インフォメーションスキーマを介してメタデータを表示する

インフォメーションスキーマのビューの内容をクエリして、StarRocksインスタンス内のメタデータ情報を表示することができます。

以下の例では、ビュー`tables`をクエリして、StarRocksの`table1`というテーブルのメタデータ情報をチェックしています。

```Plain
MySQL > SELECT * FROM information_schema.tables WHERE TABLE_NAME like 'table1'\G
*************************** 1. row ***************************
  TABLE_CATALOG: def
   TABLE_SCHEMA: test_db
     TABLE_NAME: table1
     TABLE_TYPE: BASE TABLE
         ENGINE: StarRocks
        VERSION: NULL
     ROW_FORMAT: 
     TABLE_ROWS: 4
 AVG_ROW_LENGTH: 1657
    DATA_LENGTH: 6630
MAX_DATA_LENGTH: NULL
   INDEX_LENGTH: NULL
      DATA_FREE: NULL
 AUTO_INCREMENT: NULL
    CREATE_TIME: 2023-06-13 11:37:00
    UPDATE_TIME: 2023-06-13 11:38:06
     CHECK_TIME: NULL
TABLE_COLLATION: utf8_general_ci
       CHECKSUM: NULL
 CREATE_OPTIONS: 
  TABLE_COMMENT: 
1 row in set (0.01 sec)
```

## インフォメーションスキーマのビュー

StarRocksのインフォメーションスキーマには、以下のメタデータビューが含まれています：

| **ビュー**                                                    | **説明**                                                      |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| [be_bvars](./be_bvars.md)                                       | `be_bvars`はbRPCに関する統計情報を提供します。  |
| [be_cloud_native_compactions](./be_cloud_native_compactions.md) | `be_cloud_native_compactions`は、共有データクラスタのCN（またはv3.0のBE）で実行されているコンパクショントランザクションに関する情報を提供します。 |
| [be_compactions](./be_compactions.md)                           | `be_compactions`はコンパクションタスクの統計情報を提供します。 |
| [character_sets](./character_sets.md)                           | `character_sets`は利用可能な文字セットを識別します。    |
| [collations](./collations.md)                                   | `collations`は利用可能な照合順序を含みます。              |
| [column_privileges](./column_privileges.md)                     | `column_privileges`は、現在有効なロールによって現在有効なロールによってカラムに付与されたすべての権限を識別します。 |
| [columns](./columns.md)                                         | `columns`はすべてのテーブル列（またはビュー列）に関する情報を含みます。 |
| [engines](./engines.md)                                         | `engines`はストレージエンジンに関する情報を提供します。        |
| [events](./events.md)                                           | `events`はイベントマネージャのイベントに関する情報を提供します。    |
| [global_variables](./global_variables.md)                       | `global_variables`はグローバル変数に関する情報を提供します。 |
| [key_column_usage](./key_column_usage.md)                       | `key_column_usage`は一意の主キーまたは外部キー制約によって制約されているすべての列を識別します。 |
| [load_tracking_logs](./load_tracking_logs.md)                   | `load_tracking_logs`はロードジョブのエラー情報（ある場合）を提供します。 |
| [loads](./loads.md)                                             | `loads`はロードジョブの結果を提供します。現在、このビューからは[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)と[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)ジョブの結果のみ表示できます。 |
| [materialized_views](./materialized_views.md)                   | `materialized_views`は非同期マテリアライズドビューに関する情報を提供します。 |
| [partitions](./partitions.md)                                   | `partitions`はテーブルパーティションに関する情報を提供します。    |
| [pipe_files](./pipe_files.md)                                   | `pipe_files`は指定されたパイプを介してロードされるデータファイルのステータスを提供します。 |
| [pipes](./pipes.md)                                             | `pipes`は現在のまたは指定されたデータベースに格納されているすべてのパイプに関する情報を提供します。 |
| [referential_constraints](./referential_constraints.md)         | `referential_constraints`はすべての参照（外部キー）制約を含みます。 |
| [routines](./routines.md)                                       | `routines`はすべてのストアドルーチン（ストアドプロシージャとストアド関数）を含みます。 |
| [schema_privileges](./schema_privileges.md)                     | `schema_privileges`はデータベースの権限に関する情報を提供します。 |
| [schemata](./schemata.md)                                       | `schemata`はデータベースに関する情報を提供します。             |
| [session_variables](./session_variables.md)                     | `session_variables`はセッション変数に関する情報を提供します。 |
| [statistics](./statistics.md)                                   | `statistics`はテーブルのインデックスに関する情報を提供します。       |
| [table_constraints](./table_constraints.md)                     | `table_constraints`は制約を持つテーブルを説明します。 |
| [table_privileges](./table_privileges.md)                       | `table_privileges`はテーブルの権限に関する情報を提供します。 |
| [tables](./tables.md)                                           | `tables`はテーブルに関する情報を提供します。                  |
| [tables_config](./tables_config.md)                             | `tables_config`はテーブルの設定に関する情報を提供します。 |
| [task_runs](./task_runs.md)                                     | `task_runs`は非同期タスクの実行に関する情報を提供します。 |
| [tasks](./tasks.md)                                             | `tasks`は非同期タスクに関する情報を提供します。       |
| [triggers](./triggers.md)                                       | `triggers`はトリガーに関する情報を提供します。              |
| [user_privileges](./user_privileges.md)                         | `user_privileges`はユーザーの権限に関する情報を提供します。 |
| [views](./views.md)                                             | `views`はすべてのユーザー定義ビューに関する情報を提供します。   |
