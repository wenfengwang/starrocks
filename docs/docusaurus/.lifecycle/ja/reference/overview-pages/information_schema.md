---
displayed_sidebar: "Japanese"
---

# 情報スキーマ

StarRocksの情報スキーマは、各StarRocksインスタンス内にあるデータベースです。情報スキーマには、StarRocksインスタンスが保持しているすべてのオブジェクトの詳細なメタデータ情報を格納する読み取り専用のシステム定義ビューが複数含まれています。StarRocksの情報スキーマは、SQL-92 ANSI情報スキーマを基にしていますが、StarRocks固有のビューと関数が追加されています。

v3.2.0から、StarRocksの情報スキーマは外部カタログのメタデータを管理することをサポートしています。

## 情報スキーマを介したメタデータの表示

情報スキーマ内のビューの内容をクエリして、StarRocksインスタンス内のメタデータ情報を表示することができます。

以下の例は、ビュー`tables`をクエリして、StarRocks内の`table1`というテーブルのメタデータ情報をチェックしています。

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

## 情報スキーマ内のビュー

StarRocksの情報スキーマには、以下のメタデータビューが含まれています：

| **ビュー**                                                   | **説明**                                                     |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| [be_bvars](../information_schema/be_bvars.md)                                       | `be_bvars`は、bRPCに関する統計情報を提供します。              |
| [be_cloud_native_compactions](../information_schema/be_cloud_native_compactions.md) | `be_cloud_native_compactions`は、共有データクラスタのCN（またはv3.0のBE）で実行されているコンパクショントランザクションに関する情報を提供します。 |
| [be_compactions](../information_schema/be_compactions.md)                           | `be_compactions`は、コンパクションタスクに関する統計情報を提供します。 |
| [character_sets](../information_schema/character_sets.md)                           | `character_sets`は利用可能な文字セットを識別します。          |
| [collations](../information_schema/collations.md)                                   | `collations`は利用可能な照合順序を含みます。                |
| [column_privileges](../information_schema/column_privileges.md)                     | `column_privileges`は、現在有効なロールによってカラムに付与されたすべての権限を識別します。 |
| [columns](../information_schema/columns.md)                                         | `columns`はすべてのテーブルカラム（またはビューカラム）に関する情報を含みます。 |
| [engines](../information_schema/engines.md)                                         | `engines`はストレージエンジンに関する情報を提供します。       |
| [events](../information_schema/events.md)                                           | `events`はEvent Managerのイベントに関する情報を提供します。  |
| [global_variables](../information_schema/global_variables.md)                       | `global_variables`はグローバル変数に関する情報を提供します。  |
| [key_column_usage](../information_schema/key_column_usage.md)                       | `key_column_usage`は、一意キー、主キー、または外部キー制約によって制約されているすべてのカラムを識別します。 |
| [load_tracking_logs](../information_schema/load_tracking_logs.md)                   | `load_tracking_logs`はロードジョブのエラー情報（存在する場合）を提供します。 |
| [loads](../information_schema/loads.md)                                             | `loads`はロードジョブの結果を提供します。現在は、このビューから[Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)と[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)ジョブの結果のみを表示できます。 |
| [materialized_views](../information_schema/materialized_views.md)                   | `materialized_views`は非同期マテリアライズドビューに関する情報を提供します。 |
| [partitions](../information_schema/partitions.md)                                   | `partitions`はテーブルパーティションに関する情報を提供します。 |
| [pipe_files](../information_schema/pipe_files.md)                                   | `pipe_files`は指定されたパイプを介してロードされるデータファイルのステータスを提供します。 |
| [pipes](../information_schema/pipes.md)                                             | `pipes`は現在のまたは指定されたデータベースに格納されているすべてのパイプに関する情報を提供します。 |
| [referential_constraints](../information_schema/referential_constraints.md)         | `referential_constraints`は全ての参照（外部キー）制約を含みます。 |
| [routines](../information_schema/routines.md)                                       | `routines`はすべてのストアドルーチン（ストアドプロシージャとストアドファンクション）に関する情報を含みます。 |
| [schema_privileges](../information_schema/schema_privileges.md)                     | `schema_privileges`はデータベース権限に関する情報を提供します。 |
| [schemata](../information_schema/schemata.md)                                       | `schemata`はデータベースに関する情報を提供します。           |
| [session_variables](../information_schema/session_variables.md)                     | `session_variables`はセッション変数に関する情報を提供します。  |
| [statistics](../information_schema/statistics.md)                                   | `statistics`はテーブルインデックスに関する情報を提供します。  |
| [table_constraints](../information_schema/table_constraints.md)                     | `table_constraints`はどのテーブルに制約があるかを説明します。  |
| [table_privileges](../information_schema/table_privileges.md)                       | `table_privileges`はテーブル権限に関する情報を提供します。    |
| [tables](../information_schema/tables.md)                                           | `tables`はテーブルに関する情報を提供します。                 |
| [tables_config](../information_schema/tables_config.md)                             | `tables_config`はテーブルの構成に関する情報を提供します。     |
| [task_runs](../information_schema/task_runs.md)                                     | `task_runs`は非同期タスクの実行に関する情報を提供します。    |
| [tasks](../information_schema/tasks.md)                                             | `tasks`は非同期タスクに関する情報を提供します。              |
| [triggers](../information_schema/triggers.md)                                       | `triggers`はトリガーに関する情報を提供します。                |
| [user_privileges](../information_schema/user_privileges.md)                         | `user_privileges`はユーザー権限に関する情報を提供します。    |
| [views](../information_schema/views.md)                                             | `views`はすべてのユーザー定義ビューに関する情報を提供します。 |
