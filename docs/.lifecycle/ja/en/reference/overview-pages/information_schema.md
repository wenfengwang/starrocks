---
displayed_sidebar: English
---

# 情報スキーマ

StarRocks 情報スキーマは、各 StarRocks インスタンス内のデータベースです。情報スキーマには、StarRocks インスタンスが保持するすべてのオブジェクトの広範なメタデータ情報を格納する、読み取り専用のシステム定義ビューがいくつか含まれています。StarRocks 情報スキーマは、SQL-92 ANSI 情報スキーマに基づいていますが、StarRocks に固有のビューと関数が追加されています。

v3.2.0 以降、StarRocks 情報スキーマは外部カタログのメタデータ管理をサポートしています。

## 情報スキーマを通じてメタデータを表示

StarRocks インスタンス内のメタデータ情報を表示するには、情報スキーマのビューの内容をクエリします。

次の例では、StarRocks で `table1` という名前のテーブルに関するメタデータ情報を、ビュー `tables` をクエリしてチェックします。

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

## 情報スキーマのビュー

StarRocks 情報スキーマには、以下のメタデータビューが含まれています。

| **ビュー**                                                    | **説明**                                              |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| [be_bvars](../information_schema/be_bvars.md)                                       | `be_bvars` は bRPC に関する統計情報を提供します。  |
| [be_cloud_native_compactions](../information_schema/be_cloud_native_compactions.md) | `be_cloud_native_compactions` は、共有データクラスターの CN（v3.0 では BE）で実行されているコンパクショントランザクションに関する情報を提供します。 |
| [be_compactions](../information_schema/be_compactions.md)                           | `be_compactions` はコンパクションタスクに関する統計情報を提供します。 |
| [character_sets](../information_schema/character_sets.md)                           | `character_sets` は使用可能な文字セットを識別します。    |
| [collations](../information_schema/collations.md)                                   | `collations` は使用可能な照合順序を含みます。              |
| [column_privileges](../information_schema/column_privileges.md)                     | `column_privileges` は現在有効なロールによって、または現在有効なロールによって列に付与されたすべての権限を識別します。 |
| [columns](../information_schema/columns.md)                                         | `columns` はすべてのテーブル列（またはビュー列）に関する情報を含みます。 |
| [engines](../information_schema/engines.md)                                         | `engines` はストレージエンジンに関する情報を提供します。        |
| [events](../information_schema/events.md)                                           | `events` はイベントマネージャーのイベントに関する情報を提供します。    |
| [global_variables](../information_schema/global_variables.md)                       | `global_variables` はグローバル変数に関する情報を提供します。 |
| [key_column_usage](../information_schema/key_column_usage.md)                       | `key_column_usage` は一意性、主キー、または外部キー制約によって制限されるすべての列を識別します。 |
| [load_tracking_logs](../information_schema/load_tracking_logs.md)                   | `load_tracking_logs` はロードジョブのエラー情報（存在する場合）を提供します。 |
| [loads](../information_schema/loads.md)                                             | `loads` はロードジョブの結果を提供します。現在、このビューから表示できるのは [Broker Load](../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) と [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) ジョブの結果のみです。|
| [materialized_views](../information_schema/materialized_views.md)                   | `materialized_views` はすべての非同期マテリアライズドビューに関する情報を提供します。 |
| [partitions](../information_schema/partitions.md)                                   | `partitions` はテーブルパーティションに関する情報を提供します。    |
| [pipe_files](../information_schema/pipe_files.md)                                   | `pipe_files` は指定されたパイプを介してロードされるデータファイルのステータスを提供します。 |
| [pipes](../information_schema/pipes.md)                                             | `pipes` は現在のデータベースまたは指定されたデータベースに格納されているすべてのパイプに関する情報を提供します。 |
| [referential_constraints](../information_schema/referential_constraints.md)         | `referential_constraints` はすべての参照（外部キー）制約を含みます。 |
| [routines](../information_schema/routines.md)                                       | `routines` はすべてのストアドルーチン（ストアドプロシージャとストアドファンクション）を含みます。 |
| [schema_privileges](../information_schema/schema_privileges.md)                     | `schema_privileges` はデータベース権限に関する情報を提供します。 |
| [schemata](../information_schema/schemata.md)                                       | `schemata` はデータベースに関する情報を提供します。             |
| [session_variables](../information_schema/session_variables.md)                     | `session_variables` はセッション変数に関する情報を提供します。 |
| [statistics](../information_schema/statistics.md)                                   | `statistics` はテーブルインデックスに関する情報を提供します。       |
| [table_constraints](../information_schema/table_constraints.md)                     | `table_constraints` は制約を持つテーブルについて説明します。 |
| [table_privileges](../information_schema/table_privileges.md)                       | `table_privileges` はテーブル権限に関する情報を提供します。 |
| [tables](../information_schema/tables.md)                                           | `tables` はテーブルに関する情報を提供します。                  |
| [tables_config](../information_schema/tables_config.md)                             | `tables_config` はテーブルの設定に関する情報を提供します。 |
| [task_runs](../information_schema/task_runs.md)                                     | `task_runs` は非同期タスクの実行に関する情報を提供します。 |
| [tasks](../information_schema/tasks.md)                                             | `tasks` は非同期タスクに関する情報を提供します。       |
| [triggers](../information_schema/triggers.md)                                       | `triggers` はトリガーに関する情報を提供します。              |
| [user_privileges](../information_schema/user_privileges.md)                         | `user_privileges` はユーザー権限に関する情報を提供します。 |
| [views](../information_schema/views.md)                                             | `views` はすべてのユーザー定義ビューに関する情報を提供します。   |

