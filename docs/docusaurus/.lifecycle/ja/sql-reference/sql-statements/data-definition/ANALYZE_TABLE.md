---
displayed_sidebar: "Japanese"
---

# ANALYZE TABLE（テーブルの分析）

## 説明

CBO（Cost-Based Optimizer）の統計情報を収集するための手動の収集タスクを作成します。デフォルトでは、手動収集は同期動作です。非同期動作にも設定できます。非同期モードでは、ANALYZE TABLEを実行するとシステムからすぐにこのステートメントが成功したかどうかが返されます。しかし、収集タスクはバックグラウンドで実行され、結果を待つ必要はありません。SHOW ANALYZE STATUSを実行してタスクのステータスを確認できます。非同期収集は大容量のデータを持つテーブルに適しており、同期収集は小容量のデータを持つテーブルに適しています。

**手動収集タスクは作成後1回だけ実行されます。手動収集タスクを削除する必要はありません。**

このステートメントはv2.4からサポートされています。

### 基本統計情報を手動で収集する

基本統計情報についての詳細は、[Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md#basic-statistics)を参照してください。

#### 構文

```SQL
ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
[WITH SYNC | ASYNC MODE]
PROPERTIES (property [,property])
```

#### パラメータの説明

- 収集タイプ
  - FULL: 完全な収集を示します。
  - SAMPLE: サンプル収集を示します。
  - 収集タイプが指定されていない場合、デフォルトで完全な収集が使用されます。

- `col_name`: 統計情報を収集する列。複数の列をコンマ(,)で区切って指定します。このパラメータが指定されていない場合、テーブル全体が収集されます。

- [WITH SYNC | ASYNC MODE]: 手動収集タスクを同期モードまたは非同期モードで実行するかどうか。このパラメータを指定しない場合、デフォルトで同期収集が使用されます。

- `PROPERTIES`: カスタム パラメータ。`PROPERTIES`が指定されていない場合、`fe.conf`ファイルのデフォルト設定が使用されます。実際に使用されるプロパティは、SHOW ANALYZE STATUSの出力の`Properties`列から確認できます。

| **PROPERTIES**                | **Type** | **Default value** | **Description**                                              |
| ----------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_sample_collect_rows | INT      | 200000            | サンプル収集のために収集する最小行数。パラメータ値が実際のテーブルの行数を超えると、完全な収集が実行されます。 |

#### 例

例1: 完全な手動収集

```SQL
-- デフォルト設定を使用してテーブルの完全な統計情報を手動で収集します。
ANALYZE TABLE tbl_name;

-- デフォルト設定を使用してテーブルの完全な統計情報を手動で収集します。
ANALYZE FULL TABLE tbl_name;

-- 任意の設定でテーブルの指定された列の統計情報を手動で収集します。
ANALYZE TABLE tbl_name(c1, c2, c3);
```

例2: サンプル収集

```SQL
-- デフォルト設定を使用してテーブルの部分的な統計情報を手動で収集します。
ANALYZE SAMPLE TABLE tbl_name;

-- 指定された列の統計情報を手動で収集し、収集する行数を指定します。
ANALYZE SAMPLE TABLE tbl_name (v1, v2, v3) PROPERTIES(
    "statistic_sample_collect_rows" = "1000000"
);
```

### ヒストグラムを手動で収集する

ヒストグラムについての詳細は、[Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md#histogram)を参照してください。

#### 構文

```SQL
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON col_name [, col_name]
[WITH SYNC | ASYNC MODE]
[WITH N BUCKETS]
PROPERTIES (property [,property]);
```

#### パラメータの説明

- `col_name`: 統計情報を収集する列。複数の列をコンマ(,)で区切って指定します。このパラメータが指定されていない場合、テーブル全体が収集されます。ヒストグラムには必須のパラメータです。

- [WITH SYNC | ASYNC MODE]: 手動収集タスクを同期モードまたは非同期モードで実行するかどうか。このパラメータを指定しない場合、デフォルトで同期収集が使用されます。

- `WITH N BUCKETS`: `N`はヒストグラム収集のバケット数です。指定されていない場合、`fe.conf`のデフォルト値が使用されます。

- PROPERTIES: カスタム パラメータ。`PROPERTIES`が指定されていない場合、`fe.conf`のデフォルト設定が使用されます。実際に使用されるプロパティは、SHOW ANALYZE STATUSの出力の`Properties`列から確認できます。

| **PROPERTIES**                 | **Type** | **Default value** | **Description**                                              |
| ------------------------------ | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_sample_collect_rows  | INT      | 200000            | 収集する行数の最小値。パラメータ値がテーブルの実際の行数 * `histogram_sample_ratio`を超える場合、`histogram_max_sample_row_count`で指定した値を上回ることはありません。値が上回ると、`histogram_max_sample_row_count`が優先されます。 |
| histogram_buckets_size         | LONG     | 64                | ヒストグラムのデフォルトバケット数。                   |
| histogram_mcv_size             | INT      | 100               | ヒストグラムの最も一般的な値（MCV）の個数。           |
| histogram_sample_ratio         | FLOAT    | 0.1               | ヒストグラムのサンプリング率。                        |
| histogram_max_sample_row_count | LONG     | 10000000          | ヒストグラムに収集する行数の最大値。                   |

ヒストグラムに収集する行数は複数のパラメータで制御されます。`statistic_sample_collect_rows`とテーブルの行数 * `histogram_sample_ratio`の間の大きな値となります。この値は`histogram_max_sample_row_count`で指定された値を超えることはできません。

#### 例

```SQL
-- デフォルトの設定でv1のヒストグラムを手動で収集します。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1;

-- 32バケット、32 MCV、および50%のサンプリング率でv1とv2のヒストグラムを手動で収集します。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1,v2 WITH 32 BUCKETS 
PROPERTIES(
   "histogram_mcv_size" = "32",
   "histogram_sample_ratio" = "0.5"
);
```

## 参照

[SHOW ANALYZE STATUS](../data-definition/SHOW_ANALYZE_STATUS.md): 手動収集タスクの状態を表示します。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md): 実行中の手動収集タスクをキャンセルします。

CBOの統計情報の収集についての詳細については、[Gather statistics for CBO](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。