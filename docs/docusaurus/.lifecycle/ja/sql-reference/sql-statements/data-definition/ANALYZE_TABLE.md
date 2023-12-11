---
displayed_sidebar: "Japanese"
---

# ANALYZE TABLE（テーブルの集計）

## 説明

CBO（コストベース最適化）統計の収集のための手動コレクションタスクを作成します。デフォルトでは、手動収集は同期動作です。非同期動作にも設定できます。非同期モードでは、ANALYZE TABLEを実行した後、システムはすぐにこのステートメントの成功を返します。ただし、コレクションタスクはバックグラウンドで実行され、結果を待つ必要はありません。SHOW ANALYZE STATUSを実行してタスクの状態を確認できます。非同期コレクションは大容量のデータを持つテーブルに適しており、同期コレクションは小容量のデータを持つテーブルに適しています。

手動コレクションタスクは作成後に1度だけ実行されます。手動コレクションタスクを削除する必要はありません。

このステートメントはv2.4からサポートされています。

### 基本統計の手動収集

基本統計の詳細については、[CBOの統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md#basic-statistics)を参照してください。

#### 構文

```SQL
ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name]) [WITH SYNC | ASYNC MODE] PROPERTIES (property [,property])
```

#### パラメータの説明

- 収集タイプ
  - FULL: 完全な収集を示します。
  - SAMPLE: サンプリングされた収集を示します。
  - 収集タイプが指定されていない場合、デフォルトで完全収集が使用されます。

- `col_name`：統計情報を収集する列。複数の列をカンマ（`,`）で区切ります。このパラメータが指定されていない場合、テーブル全体が収集されます。

- [WITH SYNC | ASYNC MODE]：手動の収集タスクを同期モードまたは非同期モードで実行するかどうか。このパラメータを指定しない場合、デフォルトで同期収集が使用されます。

- `PROPERTIES`：カスタムパラメータ。`PROPERTIES`が指定されていない場合、`fe.conf`ファイルのデフォルト設定が使用されます。実際に使用されるプロパティは、SHOW ANALYZE STATUSの出力の`Properties`列で確認できます。

| **PROPERTIES**                | **タイプ** | **デフォルト値** | **説明**                                              |
| ----------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_sample_collect_rows | INT      | 200000            | サンプル収集のために収集する最小行数。パラメータの値が実際のテーブルの行数を超える場合、完全収集が実行されます。 |

#### 例

例1：手動完全収集

```SQL
-- デフォルト設定を使用して、テーブルの完全な統計情報を手動で収集します。
ANALYZE TABLE tbl_name;

-- デフォルト設定を使用して、テーブルの完全な統計情報を手動で収集します。
ANALYZE FULL TABLE tbl_name;

-- デフォルト設定を使用して、テーブル内の指定された列の統計情報を手動で収集します。
ANALYZE TABLE tbl_name(c1, c2, c3);
```

例2：手動サンプリング収集

```SQL
-- デフォルト設定を使用して、テーブルの一部の統計情報を手動で収集します。
ANALYZE SAMPLE TABLE tbl_name;

-- 指定された列の統計情報を手動で収集し、収集する行数を指定します。
ANALYZE SAMPLE TABLE tbl_name (v1, v2, v3) PROPERTIES(
    "statistic_sample_collect_rows" = "1000000"
);
```

### ヒストグラムの手動収集

ヒストグラムについて詳しくは、[CBOの統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md#histogram)を参照してください。

#### 構文

```SQL
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON col_name [, col_name] [WITH SYNC | ASYNC MODE] [WITH N BUCKETS] PROPERTIES (property [,property])
```

#### パラメータの説明

- `col_name`：統計情報を収集する列。複数の列をカンマ（`,`）で区切ります。このパラメータが指定されていない場合、テーブル全体が収集されます。ヒストグラムの場合は必須です。

- [WITH SYNC | ASYNC MODE]：手動の収集タスクを同期モードまたは非同期モードで実行するかどうか。このパラメータを指定しない場合、デフォルトで同期収集が使用されます。

- `WITH N BUCKETS`：`N`はヒストグラム収集のバケット数です。指定されていない場合、`fe.conf`のデフォルト値が使用されます。

- PROPERTIES：カスタムパラメータ。`PROPERTIES`が指定されていない場合、`fe.conf`のデフォルト設定が使用されます。実際に使用されるプロパティは、SHOW ANALYZE STATUSの出力の`Properties`列で確認できます。

| **PROPERTIES**                 | **Type** | **デフォルト値** | **説明**                                              |
| ------------------------------ | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_sample_collect_rows  | INT      | 200000            | 収集する行数の最小値。パラメータの値が実際のテーブルの行数を超える場合、完全収集が実行されます。 |
| histogram_buckets_size         | LONG     | 64                | ヒストグラムのデフォルトバケット数。                   |
| histogram_mcv_size             | INT      | 100               | ヒストグラムの最頻値（MCV）の数。      |
| histogram_sample_ratio         | FLOAT    | 0.1               | ヒストグラムのサンプリング比率。                          |
| histogram_max_sample_row_count | LONG     | 10000000          | ヒストグラムのために収集される行の最大数。       |

ヒストグラムのための収集行数は複数のパラメータで制御されます。`statistic_sample_collect_rows`とテーブル行数 * `histogram_sample_ratio`の大きい方です。この数値は`histogram_max_sample_row_count`で指定された値を超えることはできません。値を超えた場合は、`histogram_max_sample_row_count`が優先されます。

#### 例

```SQL
-- デフォルト設定を使用して、v1に対してヒストグラムを手動で収集します。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1;

-- v1とv2に対して、32のバケット、32のMCV、50％のサンプリング比率でヒストグラムを手動で収集します。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1,v2 WITH 32 BUCKETS 
PROPERTIES(
   "histogram_mcv_size" = "32",
   "histogram_sample_ratio" = "0.5"
);
```

## 参照

[SHOW ANALYZE STATUS](../data-definition/SHOW_ANALYZE_STATUS.md)：手動収集タスクのステータスを表示します。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md)：実行中の手動収集タスクをキャンセルします。

CBOの統計情報の収集についての詳細は、[CBOのための統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。