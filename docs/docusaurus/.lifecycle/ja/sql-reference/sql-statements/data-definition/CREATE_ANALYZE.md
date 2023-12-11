---
displayed_sidebar: "Japanese"
---

# ANALYZE作成

## 説明

CBO統計を収集するための自動収集タスクをカスタマイズします。

StarRocksは、デフォルトでテーブルの全統計を自動的に収集します。データの更新を5分ごとに確認します。データの変更が検出された場合、データ収集が自動的にトリガーされます。自動的な全収集を使用したくない場合は、FE構成項目`enable_collect_full_statistic`を`false`に設定し、収集タスクをカスタマイズできます。

カスタム自動収集タスクを作成する前に、自動的な全収集を無効にする必要があります（`enable_collect_full_statistic = false`）。それ以外の場合、カスタムタスクは有効になりません。

このステートメントはv2.4からサポートされています。

## 構文

```SQL
-- 全データベースの統計を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] ALL PROPERTIES (property [,property])

-- データベース内のすべてのテーブルの統計を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] DATABASE db_name
PROPERTIES (property [,property])

-- 指定したテーブルの列の統計情報を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
PROPERTIES (property [,property])
```

## パラメータの説明

- 収集タイプ
  - FULL: 完全な収集を示します。
  - SAMPLE: サンプリングされた収集を示します。
  - 収集タイプが指定されていない場合、デフォルトで完全な収集が使用されます。

- `col_name`: 統計情報を収集する列。複数の列をカンマ（`,`）で区切ります。このパラメータが指定されていない場合、テーブル全体が収集されます。

- `PROPERTIES`: カスタムパラメータ。`PROPERTIES`が指定されていない場合、`fe.conf`のデフォルト設定が使用されます。実際に使用されるプロパティは、SHOW ANALYZE JOBの出力の`Properties`列で確認できます。

| **PROPERTIES**                        | **Type** | **Default value** | **説明**                                              |
| ------------------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_auto_collect_ratio          | FLOAT    | 0.8               | 自動収集の統計が健全かどうかを判定するための閾値。統計の健全性がこの閾値を下回る場合、自動収集がトリガーされます。 |
| statistics_max_full_collect_data_size | INT      | 100               | 自動収集でデータを収集する最大パーティションのサイズ。単位: GB。パーティションがこの値を超える場合、完全な収集は破棄され、サンプリングされた収集が代わりに実行されます。 |
| statistic_sample_collect_rows         | INT      | 200000            | 収集する最小の行数。パラメータ値が実際のテーブルの行数を超える場合、完全な収集が実行されます。 |

## 例

例1：自動的な完全な収集

```SQL
-- すべてのデータベースの完全な統計を自動的に収集します。
CREATE ANALYZE ALL;

-- データベースの完全な統計を自動的に収集します。
CREATE ANALYZE DATABASE db_name;

-- データベース内のすべてのテーブルの完全な統計を自動的に収集します。
CREATE ANALYZE FULL DATABASE db_name;

-- 指定したテーブルの列の完全な統計を自動的に収集します。
CREATE ANALYZE TABLE tbl_name(c1, c2, c3); 
```

例2：自動的なサンプリング収集

```SQL
-- データベース内のすべてのテーブルの統計をデフォルトの設定で自動的に収集します。
CREATE ANALYZE SAMPLE DATABASE db_name;

-- 指定したテーブルの列の統計を収集し、統計の健康状態と収集する行数を指定します。
CREATE ANALYZE SAMPLE TABLE tbl_name(c1, c2, c3) PROPERTIES(
   "statistic_auto_collect_ratio" = "0.5",
   "statistic_sample_collect_rows" = "1000000"
);
```

## 参照

[SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md): カスタム収集タスクの状態を表示します。

[DROP ANALYZE](../data-definition/DROP_ANALYZE.md): カスタム収集タスクを削除します。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md): 実行中のカスタム収集タスクをキャンセルします。

CBOの統計情報の収集に関する詳細については、[CBOのための統計情報を収集する](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。