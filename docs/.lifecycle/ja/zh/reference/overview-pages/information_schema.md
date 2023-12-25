---
displayed_sidebar: Chinese
---

# Information Schema

Information Schema は StarRocks インスタンス内のデータベースです。このデータベースには、StarRocks インスタンス内のすべてのオブジェクトに関する多くのメタデータ情報を格納している、システム定義のビューがいくつか含まれています。

バージョン3.2.0以降、Information Schema は External Catalog 内のメタデータ情報の管理をサポートしています。

## Information Schema を通じてメタデータ情報を見る

Information Schema のビューをクエリすることで、StarRocks インスタンス内のメタデータ情報を確認できます。

以下の例では、ビュー `tables` をクエリして、StarRocks にある `table1` という名前のテーブルに関連するメタデータ情報を見ています。

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

## Information Schema のビュー

StarRocks Information Schema には以下のビューが含まれています:

| **ビュー名**                                                  | **説明**                                                     |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| [be_bvars](../information_schema/be_bvars.md)                                       | `be_bvars` は bRPC の統計情報を提供します。                        |
| [be_cloud_native_compactions](../information_schema/be_cloud_native_compactions.md) | `be_cloud_native_compactions` は、統合ストレージクラスターの CN（またはバージョン3.0 の BE）で実行される Compaction トランザクションに関する情報を提供します。 |
| [be_compactions](../information_schema/be_compactions.md)                           | `be_compactions` は Compaction タスクの統計情報を提供します。        |
| [character_sets](../information_schema/character_sets.md)                           | `character_sets` は使用可能な文字セットを識別するために使用されます。                      |
| [collations](../information_schema/collations.md)                                   | `collations` は使用可能な照合順序を含んでいます。                            |
| [column_privileges](../information_schema/column_privileges.md)                     | `column_privileges` は、現在有効なロールに付与された、または現在有効なロールによって付与されたすべての列の権限を識別するために使用されます。 |
| [columns](../information_schema/columns.md)                                         | `columns` は、すべてのテーブル（またはビュー）の列に関する情報を含んでいます。               |
| [engines](../information_schema/engines.md)                                         | `engines` はストレージエンジンに関する情報を提供します。                           |
| [events](../information_schema/events.md)                                           | `events` は Event Manager のイベントに関する情報を提供します。                 |
| [global_variables](../information_schema/global_variables.md)                       | `global_variables` はグローバル変数に関する情報を提供します。                  |
| [key_column_usage](../information_schema/key_column_usage.md)                       | `key_column_usage` は、一意、主キー、または外部キーの制約によって制限されるすべての列を識別するために使用されます。 |
| [load_tracking_logs](../information_schema/load_tracking_logs.md)                   | インポートジョブに関連するエラー情報を提供します。                                 |
| [loads](../information_schema/loads.md)                                             | インポートジョブの結果情報を提供します。現在は [Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) と [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) のインポートジョブの結果情報のみをサポートしています。 |
| [materialized_views](../information_schema/materialized_views.md)                   | `materialized_views` は、すべての非同期マテリアライズドビューに関する情報を提供します。        |
| [partitions](../information_schema/partitions.md)                                   | `partitions` はテーブルのパーティションに関する情報を提供します。                          |
| [pipe_files](../information_schema/pipe_files.md)                                   | `pipe_files` は指定された Pipe の下でのデータファイルのインポート状態を提供します。            |
| [pipes](../information_schema/pipes.md)                                             | `pipes` は現在のデータベースまたは指定されたデータベースのすべての Pipe の詳細情報を提供します。   |
| [referential_constraints](../information_schema/referential_constraints.md)         | `referential_constraints` はすべての参照（外部キー）制約を含んでいます。         |
| [routines](../information_schema/routines.md)                                       | `routines` は、すべてのストアドプロシージャ（ルーチン）、プロセスおよび関数を含んでいます。   |
| [schema_privileges](../information_schema/schema_privileges.md)                     | `schema_privileges` はデータベース権限に関する情報を提供します。               |
| [schemata](../information_schema/schemata.md)                                       | `schemata` はデータベースに関する情報を提供します。                            |
| [session_variables](../information_schema/session_variables.md)                     | `session_variables` はセッション変数に関する情報を提供します。            |
| [statistics](../information_schema/statistics.md)                                   | `statistics` はテーブルのインデックスに関する情報を提供します。                          |
| [table_constraints](../information_schema/table_constraints.md)                     | `table_constraints` は制約を持つテーブルを記述します。                       |
| [table_privileges](../information_schema/table_privileges.md)                       | `table_privileges` はテーブル権限に関する情報を提供します。                    |
| [tables](../information_schema/tables.md)                                           | `tables` はテーブルに関する情報を提供します。                                  |
| [tables_config](../information_schema/tables_config.md)                             | `tables_config` はテーブル設定に関する情報を提供します。                       |
| [task_runs](../information_schema/task_runs.md)                                     | `task_runs` は非同期タスクの実行に関する情報を提供します。                     |
| [tasks](../information_schema/tasks.md)                                             | `tasks` は非同期タスクに関する情報を提供します。                             |
| [triggers](../information_schema/triggers.md)                                       | `triggers` はトリガーに関する情報を提供します。                            |
| [user_privileges](../information_schema/user_privileges.md)                         | `user_privileges` はユーザー権限に関する情報を提供します。                   |
| [views](../information_schema/views.md)                                             | `views` はすべてのユーザー定義ビューに関する情報を提供します。                     |
