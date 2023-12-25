---
displayed_sidebar: English
---

# ANALYZE TABLE

## 説明

CBO統計を収集するための手動収集タスクを作成します。デフォルトでは、手動収集は同期操作です。また、非同期操作に設定することもできます。非同期モードでは、ANALYZE TABLEを実行した後、このステートメントが成功したかどうかがすぐに返されます。しかし、収集タスクはバックグラウンドで実行され、結果を待つ必要はありません。タスクのステータスは、SHOW ANALYZE STATUSを実行することで確認できます。非同期コレクションはデータ量の多いテーブルに適しており、同期コレクションはデータ量の少ないテーブルに適しています。

**手動収集タスクは、作成後に一度だけ実行されます。手動収集タスクを削除する必要はありません。**

このステートメントはv2.4からサポートされています。

### 基本統計の手動収集

基本統計についての詳細は、[CBOの統計を収集する](../../../using_starrocks/Cost_based_optimizer.md#basic-statistics)を参照してください。

#### 構文

```SQL
ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [, col_name])
[WITH SYNC | ASYNC MODE]
PROPERTIES (property [, property])
```

#### パラメータ説明

- 収集タイプ
  - FULL: 全収集を示します。
  - SAMPLE: サンプル収集を示します。
  - 収集タイプが指定されていない場合、デフォルトで全収集が使用されます。

- `col_name`: 統計を収集する列。複数の列はコンマ(`,`)で区切ります。このパラメータが指定されていない場合、テーブル全体が収集されます。

- [WITH SYNC | ASYNC MODE]: 手動収集タスクを同期モードまたは非同期モードで実行するかどうか。このパラメータが指定されていない場合、デフォルトで同期収集が使用されます。

- `PROPERTIES`: カスタムパラメータ。`PROPERTIES`が指定されていない場合、`fe.conf`ファイルのデフォルト設定が使用されます。実際に使用されるプロパティは、SHOW ANALYZE STATUSの出力にある`Properties`列で確認できます。

| **PROPERTIES**                | **タイプ** | **デフォルト値** | **説明**                                              |
| ----------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_sample_collect_rows | INT      | 200000            | サンプル収集で収集する最小行数。パラメータ値がテーブルの実際の行数を超える場合、全収集が実行されます。 |

#### 例

例1: 手動による全収集

```SQL
-- デフォルト設定を使用してテーブルの全統計を手動で収集します。
ANALYZE TABLE tbl_name;

-- デフォルト設定を使用してテーブルの全統計を手動で収集します。
ANALYZE FULL TABLE tbl_name;

-- デフォルト設定を使用してテーブルの特定の列の統計を手動で収集します。
ANALYZE TABLE tbl_name(c1, c2, c3);
```

例2: 手動によるサンプル収集

```SQL
-- デフォルト設定を使用してテーブルの部分統計を手動で収集します。
ANALYZE SAMPLE TABLE tbl_name;

-- 収集する行数を指定して、テーブルの特定の列の統計を手動で収集します。
ANALYZE SAMPLE TABLE tbl_name (v1, v2, v3) PROPERTIES(
    "statistic_sample_collect_rows" = "1000000"
);
```

### ヒストグラムの手動収集

ヒストグラムについての詳細は、[CBOの統計を収集する](../../../using_starrocks/Cost_based_optimizer.md#histogram)を参照してください。

#### 構文

```SQL
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON col_name [, col_name]
[WITH SYNC | ASYNC MODE]
[WITH N BUCKETS]
PROPERTIES (property [, property]);
```

#### パラメータ説明

- `col_name`: 統計を収集する列。複数の列はコンマ(`,`)で区切ります。このパラメータが指定されていない場合、テーブル全体が収集されます。ヒストグラムにはこのパラメータが必須です。

- [WITH SYNC | ASYNC MODE]: 手動収集タスクを同期モードまたは非同期モードで実行するかどうか。このパラメータが指定されていない場合、デフォルトで同期収集が使用されます。

- `WITH N BUCKETS`: `N`はヒストグラム収集のためのバケット数です。指定されていない場合、`fe.conf`のデフォルト値が使用されます。

- PROPERTIES: カスタムパラメータ。`PROPERTIES`が指定されていない場合、`fe.conf`のデフォルト設定が使用されます。実際に使用されるプロパティは、SHOW ANALYZE STATUSの出力にある`Properties`列で確認できます。

| **PROPERTIES**                 | **タイプ** | **デフォルト値** | **説明**                                              |
| ------------------------------ | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_sample_collect_rows  | INT      | 200000            | 収集する最小行数。パラメータ値がテーブルの実際の行数を超える場合、全収集が実行されます。 |
| histogram_buckets_size         | LONG     | 64                | ヒストグラムのデフォルトのバケット数。                   |
| histogram_mcv_size             | INT      | 100               | ヒストグラムの最も一般的な値(MCV)の数。      |
| histogram_sample_ratio         | FLOAT    | 0.1               | ヒストグラムのサンプリング比率。                          |
| histogram_max_sample_row_count | LONG     | 10000000          | ヒストグラムで収集する最大行数。       |

ヒストグラムで収集する行数は、複数のパラメータによって制御されます。`statistic_sample_collect_rows`とテーブル行数 * `histogram_sample_ratio`の大きい方の値です。この数値は`histogram_max_sample_row_count`で指定された値を超えることはできません。値を超える場合、`histogram_max_sample_row_count`が優先されます。

#### 例

```SQL
-- デフォルト設定を使用してv1に対するヒストグラムを手動で収集します。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1;

-- 32バケット、32MCV、50%のサンプリング比率でv1とv2に対するヒストグラムを手動で収集します。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1,v2 WITH 32 BUCKETS 
PROPERTIES(
   "histogram_mcv_size" = "32",
   "histogram_sample_ratio" = "0.5"
);
```

## 参照

[SHOW ANALYZE STATUS](../data-definition/SHOW_ANALYZE_STATUS.md): 手動収集タスクのステータスを表示します。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md): 実行中の手動収集タスクをキャンセルします。

CBOの統計を収集するについての詳細は、[CBOの統計を収集する](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。
