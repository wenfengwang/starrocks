---
displayed_sidebar: "Japanese"
---

# ANALYZEの作成

## 説明

CBO（Cost Based Optimization）統計情報の取得をカスタマイズする自動収集タスクを作成します。

デフォルトでは、StarRocksはテーブルの完全な統計情報を自動的に収集します。データの更新は5分ごとにチェックされます。データの変更が検出された場合、データの収集が自動的にトリガーされます。自動的な完全な収集を使用したくない場合は、FE構成項目 `enable_collect_full_statistic` を `false` に設定し、収集タスクをカスタマイズできます。

カスタム自動収集タスクを作成する前に、自動的な完全な収集（`enable_collect_full_statistic = false`）を無効にする必要があります。そうしないと、カスタムタスクが有効になりません。

この文はv2.4からサポートされています。

## 構文

```SQL
-- すべてのデータベースの統計情報を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] ALL PROPERTIES (property [,property])

-- データベース内のすべてのテーブルの統計情報を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] DATABASE db_name
PROPERTIES (property [,property])

-- 指定したテーブルの指定した列の統計情報を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
PROPERTIES (property [,property])
```

## パラメータの説明

- 収集タイプ
  - FULL: 完全な収集を示します。
  - SAMPLE: サンプリング収集を示します。
  - 収集タイプが指定されていない場合、デフォルトで完全な収集が使用されます。

- `col_name`: 統計情報を収集する列。複数の列をカンマ（`,`）で区切ります。このパラメータが指定されていない場合、テーブル全体が収集されます。

- `PROPERTIES`: カスタムパラメータ。`PROPERTIES` が指定されていない場合、`fe.conf` のデフォルト設定が使用されます。実際に使用されるプロパティは、SHOW ANALYZE JOB の出力の `Properties` 列で確認できます。

| **PROPERTIES**                        | **Type** | **Default value** | **Description**                                              |
| ------------------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_auto_collect_ratio          | FLOAT    | 0.8               | 自動収集の統計情報が健全かどうかを判断するための閾値。統計情報の健全性がこの閾値以下の場合、自動収集がトリガーされます。 |
| statistics_max_full_collect_data_size | INT      | 100               | 自動収集の最大パーティションのサイズ。単位: GB。パーティションがこの値を超えると、完全な収集は破棄され、サンプリング収集が代わりに実行されます。 |
| statistic_sample_collect_rows         | INT      | 200000            | 収集する行の最小数。パラメータの値が実際のテーブルの行数を超える場合、完全な収集が実行されます。 |

## 例

例1: 自動的な完全な収集

```SQL
-- すべてのデータベースの完全な統計情報を自動的に収集します。
CREATE ANALYZE ALL;

-- データベースの完全な統計情報を自動的に収集します。
CREATE ANALYZE DATABASE db_name;

-- データベース内のすべてのテーブルの完全な統計情報を自動的に収集します。
CREATE ANALYZE FULL DATABASE db_name;

-- 指定したテーブルの指定した列の完全な統計情報を自動的に収集します。
CREATE ANALYZE TABLE tbl_name(c1, c2, c3);
```

例2: 自動的なサンプリング収集

```SQL
-- デフォルト設定を使用して、データベース内のすべてのテーブルの統計情報を自動的に収集します。
CREATE ANALYZE SAMPLE DATABASE db_name;

-- 統計情報の健全性と収集する行の数が指定された、指定したテーブルの指定した列の統計情報を自動的に収集します。
CREATE ANALYZE SAMPLE TABLE tbl_name(c1, c2, c3) PROPERTIES(
   "statistic_auto_collect_ratio" = "0.5",
   "statistic_sample_collect_rows" = "1000000"
);
```

## 参照

[SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md): カスタム収集タスクの状態を表示します。

[DROP ANALYZE](../data-definition/DROP_ANALYZE.md): カスタム収集タスクを削除します。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md): 実行中のカスタム収集タスクをキャンセルします。

CBOの統計情報の収集に関する詳細情報については、[CBOのための統計情報を収集する](../../../using_starrocks/Cost_based_optimizer.md) を参照してください。