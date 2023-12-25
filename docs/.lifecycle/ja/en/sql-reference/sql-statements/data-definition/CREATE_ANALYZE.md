---
displayed_sidebar: English
---

# CREATE ANALYZE

## 説明

CBO統計の収集タスクをカスタマイズします。

デフォルトでは、StarRocksはテーブルの統計を自動的に全て収集します。5分ごとにデータ更新をチェックし、データ変更が検出された場合、データ収集が自動的にトリガーされます。自動全収集を使用しない場合は、FE設定項目`enable_collect_full_statistic`を`false`に設定し、収集タスクをカスタマイズできます。

カスタム自動収集タスクを作成する前に、自動全収集(`enable_collect_full_statistic = false`)を無効にする必要があります。そうしないと、カスタムタスクは有効になりません。

このステートメントはv2.4からサポートされています。

## 構文

```SQL
-- すべてのデータベースの統計を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] ALL PROPERTIES (property [,property])

-- データベース内のすべてのテーブルの統計を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] DATABASE db_name
PROPERTIES (property [,property])

-- テーブル内の指定された列の統計を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
PROPERTIES (property [,property])
```

## パラメータ説明

- 収集タイプ
  - FULL: 全収集を示します。
  - SAMPLE: サンプル収集を示します。
  - 収集タイプが指定されていない場合、デフォルトで全収集が使用されます。

- `col_name`: 統計を収集する列。複数の列はコンマ(`,`)で区切ります。このパラメータが指定されていない場合、テーブル全体が収集されます。

- `PROPERTIES`: カスタムパラメータ。`PROPERTIES`が指定されていない場合、`fe.conf`のデフォルト設定が使用されます。実際に使用されるプロパティは、SHOW ANALYZE JOBの出力の`Properties`列で確認できます。

| **PROPERTIES**                        | **タイプ** | **デフォルト値** | **説明**                                              |
| ------------------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_auto_collect_ratio          | FLOAT    | 0.8               | 自動収集の統計が健全かどうかを判断するしきい値。統計の健全性がこのしきい値以下であれば、自動収集がトリガーされます。 |
| statistics_max_full_collect_data_size | INT      | 100               | 自動収集でデータを収集する最大パーティションのサイズ。単位: GB。パーティションがこの値を超えると、全収集は行われず、サンプル収集が代わりに実行されます。 |
| statistic_sample_collect_rows         | INT      | 200000            | 収集する最小行数。パラメータ値がテーブルの実際の行数を超える場合、全収集が実行されます。 |

## 例

例1: 自動全収集

```SQL
-- すべてのデータベースの完全な統計を自動的に収集します。
CREATE ANALYZE ALL;

-- データベースの完全な統計を自動的に収集します。
CREATE ANALYZE DATABASE db_name;

-- データベース内のすべてのテーブルの完全な統計を自動的に収集します。
CREATE ANALYZE FULL DATABASE db_name;

-- テーブル内の指定された列の完全な統計を自動的に収集します。
CREATE ANALYZE TABLE tbl_name(c1, c2, c3); 
```

例2: 自動サンプル収集

```SQL
-- デフォルト設定を使用して、データベース内のすべてのテーブルの統計を自動的に収集します。
CREATE ANALYZE SAMPLE DATABASE db_name;

-- 統計の健全性と収集する行数を指定して、テーブル内の指定された列の統計を自動的に収集します。
CREATE ANALYZE SAMPLE TABLE tbl_name(c1, c2, c3) PROPERTIES(
   "statistic_auto_collect_ratio" = "0.5",
   "statistic_sample_collect_rows" = "1000000"
);
```

## 参照

[SHOW ANALYZE JOB](../data-definition/SHOW_ANALYZE_JOB.md): カスタム収集タスクのステータスを表示します。

[DROP ANALYZE](../data-definition/DROP_ANALYZE.md): カスタム収集タスクを削除します。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md): 実行中のカスタム収集タスクをキャンセルします。

CBOの統計収集についての詳細は、[CBOのための統計収集](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。
