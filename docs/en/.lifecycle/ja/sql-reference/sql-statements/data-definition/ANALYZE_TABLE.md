---
displayed_sidebar: "Japanese"
---

# ANALYZE TABLE

## 説明

CBO統計情報の収集のための手動コレクションタスクを作成します。デフォルトでは、手動コレクションは同期操作です。非同期操作にも設定することができます。非同期モードでは、ANALYZE TABLEを実行した後、システムはすぐにこのステートメントが成功したかどうかを返します。ただし、コレクションタスクはバックグラウンドで実行され、結果を待つ必要はありません。SHOW ANALYZE STATUSを実行してタスクのステータスを確認することができます。非同期コレクションはデータ量が多いテーブルに適しており、同期コレクションはデータ量が少ないテーブルに適しています。

**手動コレクションタスクは作成後に1回だけ実行されます。手動コレクションタスクを削除する必要はありません。**

このステートメントはv2.4からサポートされています。

### 基本統計情報の手動収集

基本統計情報についての詳細は、[CBOの統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md#basic-statistics)を参照してください。

#### 構文

```SQL
ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
[WITH SYNC | ASYNC MODE]
PROPERTIES (property [,property])
```

#### パラメータの説明

- コレクションタイプ
  - FULL: フルコレクションを示します。
  - SAMPLE: サンプルコレクションを示します。
  - コレクションタイプが指定されていない場合、デフォルトでフルコレクションが使用されます。

- `col_name`: 統計情報を収集する列。複数の列をカンマ（`,`）で区切って指定します。このパラメータが指定されていない場合、テーブル全体が収集されます。

- [WITH SYNC | ASYNC MODE]: 手動コレクションタスクを同期モードまたは非同期モードで実行するかどうかを指定します。このパラメータを指定しない場合、デフォルトで同期コレクションが使用されます。

- `PROPERTIES`: カスタムパラメータ。`PROPERTIES`が指定されていない場合、`fe.conf`ファイルのデフォルト設定が使用されます。実際に使用されるプロパティは、SHOW ANALYZE STATUSの出力の`Properties`列で確認できます。

| **PROPERTIES**                | **Type** | **Default value** | **Description**                                              |
| ----------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_sample_collect_rows | INT      | 200000            | サンプルコレクションのために収集する最小行数。パラメータの値がテーブルの実際の行数を超える場合、フルコレクションが実行されます。 |

#### 例

例1: フルコレクションの手動収集

```SQL
-- デフォルトの設定を使用してテーブルのフル統計情報を手動で収集します。
ANALYZE TABLE tbl_name;

-- デフォルトの設定を使用してテーブルのフル統計情報を手動で収集します。
ANALYZE FULL TABLE tbl_name;

-- 指定した列の統計情報を手動で収集します。
ANALYZE TABLE tbl_name(c1, c2, c3);
```

例2: サンプルコレクションの手動収集

```SQL
-- デフォルトの設定を使用してテーブルの一部の統計情報を手動で収集します。
ANALYZE SAMPLE TABLE tbl_name;

-- 指定した列の統計情報を収集し、収集する行数を指定します。
ANALYZE SAMPLE TABLE tbl_name (v1, v2, v3) PROPERTIES(
    "statistic_sample_collect_rows" = "1000000"
);
```

### ヒストグラムの手動収集

ヒストグラムについての詳細は、[CBOの統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md#histogram)を参照してください。

#### 構文

```SQL
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON col_name [, col_name]
[WITH SYNC | ASYNC MODE]
[WITH N BUCKETS]
PROPERTIES (property [,property]);
```

#### パラメータの説明

- `col_name`: 統計情報を収集する列。複数の列をカンマ（`,`）で区切って指定します。このパラメータが指定されていない場合、テーブル全体が収集されます。ヒストグラムの場合、このパラメータは必須です。

- [WITH SYNC | ASYNC MODE]: 手動コレクションタスクを同期モードまたは非同期モードで実行するかどうかを指定します。このパラメータを指定しない場合、デフォルトで同期コレクションが使用されます。

- `WITH N BUCKETS`: `N`はヒストグラムのバケット数です。指定されていない場合、`fe.conf`のデフォルト値が使用されます。

- PROPERTIES: カスタムパラメータ。`PROPERTIES`が指定されていない場合、`fe.conf`のデフォルト設定が使用されます。実際に使用されるプロパティは、SHOW ANALYZE STATUSの出力の`Properties`列で確認できます。

| **PROPERTIES**                 | **Type** | **Default value** | **Description**                                              |
| ------------------------------ | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_sample_collect_rows  | INT      | 200000            | 収集する行数の最小値。パラメータの値がテーブルの実際の行数 * `histogram_sample_ratio`を超える場合、フルコレクションが実行されます。 |
| histogram_buckets_size         | LONG     | 64                | ヒストグラムのデフォルトのバケット数。                   |
| histogram_mcv_size             | INT      | 100               | ヒストグラムの最も一般的な値（MCV）の数。      |
| histogram_sample_ratio         | FLOAT    | 0.1               | ヒストグラムのサンプリング比率。                          |
| histogram_max_sample_row_count | LONG     | 10000000          | ヒストグラムの収集する最大行数。       |

ヒストグラムの収集する行数は複数のパラメータで制御されます。`statistic_sample_collect_rows`とテーブルの行数 * `histogram_sample_ratio`の大きい方が使用されます。この値は`histogram_max_sample_row_count`で指定された値を超えることはできません。値が超える場合は、`histogram_max_sample_row_count`が優先されます。

#### 例

```SQL
-- デフォルトの設定を使用してv1のヒストグラムを手動で収集します。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1;

-- 32個のバケット、32個のMCV、50%のサンプリング比率でv1とv2のヒストグラムを手動で収集します。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1,v2 WITH 32 BUCKETS 
PROPERTIES(
   "histogram_mcv_size" = "32",
   "histogram_sample_ratio" = "0.5"
);
```

## 参考

[SHOW ANALYZE STATUS](../data-definition/SHOW_ANALYZE_STATUS.md): 手動コレクションタスクのステータスを表示します。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md): 実行中の手動コレクションタスクをキャンセルします。

CBOの統計情報の収集についての詳細は、[CBOの統計情報の収集](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。
