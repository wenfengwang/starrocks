---
displayed_sidebar: Chinese
---

# CREATE ANALYZE

## 機能

カスタム自動収集タスクを作成し、CBO統計情報の収集を行います。

デフォルトでは、StarRocksは定期的にテーブルの全量統計情報を自動収集します。デフォルトのチェック更新時間は5分ごとで、データ更新があると自動的に収集がトリガーされます。自動全量収集を使用しない場合は、FEの設定項目 `enable_collect_full_statistic` を `false` に設定することで、システムは自動全量収集を停止し、作成したカスタムタスクに基づいてカスタマイズされた収集を行います。

カスタム自動収集タスクを作成する前に、自動全量収集をオフにする必要があります `enable_collect_full_statistic=false`。そうしないとカスタム収集タスクは有効になりません。

## 文法

```SQL
-- 定期的にすべてのデータベースの統計情報を収集します。
CREATE ANALYZE [FULL|SAMPLE] ALL PROPERTIES (property [,property])

-- 定期的に特定のデータベースのすべてのテーブルの統計情報を収集します。
CREATE ANALYZE [FULL|SAMPLE] DATABASE db_name PROPERTIES (property [,property])

-- 定期的に特定のテーブルや列の統計情報を収集します。
CREATE ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name]) PROPERTIES (property [,property])
```

## パラメータ説明

- 収集タイプ
  - FULL：全量収集。
  - SAMPLE：サンプリング収集。
  - 収集タイプを指定しない場合、デフォルトは全量収集です。

- `col_name`: 統計情報を収集する列で、複数の列はコンマ (,) で区切ります。指定しない場合は、テーブル全体の情報を収集します。

- PROPERTIES: 収集タスクのカスタムパラメータ。設定しない場合は、`fe.conf` のデフォルト設定を使用します。タスク実行中に使用される PROPERTIES は、[SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md) の `Properties` 列で確認できます。

| **PROPERTIES**                        | **タイプ** | **デフォルト値** | **説明**                                                     |
| ------------------------------------- | ---------- | ---------------- | ------------------------------------------------------------ |
| statistic_auto_collect_ratio          | FLOAT      | 0.8              | 自動統計情報の健全性閾値。統計情報の健全性がこの閾値未満の場合、自動収集がトリガーされます。 |
| statistics_max_full_collect_data_size | INT        | 100              | 自動統計情報収集の最大パーティションサイズ。単位: GB。あるパーティションがこの値を超える場合、全量統計情報収集を中止し、そのテーブルのサンプリング統計情報収集に切り替えます。 |
| statistic_sample_collect_rows         | INT        | 200000           | サンプリング収集のサンプル行数。このパラメータの値が実際のテーブル行数を超える場合、全量収集を行います。 |

## 例

例 1：自動全量収集

```SQL
-- 定期的にすべてのデータベースの統計情報を全量収集します。
CREATE ANALYZE ALL;

-- 定期的に特定のデータベースのすべてのテーブルの統計情報を全量収集します。
CREATE ANALYZE DATABASE db_name;

-- 定期的に特定のデータベースのすべてのテーブルの統計情報を全量収集します。
CREATE ANALYZE FULL DATABASE db_name;

-- 定期的に特定のテーブルや列の統計情報を全量収集します。
CREATE ANALYZE TABLE tbl_name(c1, c2, c3); 
```

例 2：自動サンプリング収集

```SQL
-- 定期的に特定のデータベースのすべてのテーブルの統計情報をサンプリング収集し、デフォルト設定を使用します。
CREATE ANALYZE SAMPLE DATABASE db_name;

-- 定期的に特定のテーブルや列の統計情報をサンプリング収集し、サンプル行数と健全性閾値を設定します。
CREATE ANALYZE SAMPLE TABLE tbl_name(c1, c2, c3) PROPERTIES(
   "statistic_auto_collect_ratio" = "0.5",
   "statistic_sample_collect_rows" = "1000000"
);
```

## 関連文書

[SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md)：作成したカスタム自動収集タスクを確認します。

[DROP ANALYZE](../data-definition/DROP_ANALYZE.md)：自動収集タスクを削除します。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md)：実行中（Running）の統計情報収集タスクをキャンセルします。

CBO統計情報収集の詳細については、[CBO統計情報](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。
