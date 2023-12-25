---
displayed_sidebar: Chinese
---

# ANALYZE TABLE

## 機能

手動で統計情報収集タスクを作成し、CBO統計情報を収集します。**手動収集はデフォルトで同期操作です。手動タスクを非同期に設定することもでき、コマンドを実行すると、直ちにコマンドの状態が返されますが、統計情報収集タスクはバックグラウンドで実行され、その状態はSHOW ANALYZE STATUSで確認できます。手動タスクは作成後一度だけ実行され、手動で削除する必要はありません。**

### 手動で基本統計情報を収集

基本統計情報については、[CBO統計情報](../../../using_starrocks/Cost_based_optimizer.md#統計情報データタイプ)を参照してください。

#### 構文

```SQL
ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
[WITH SYNC | ASYNC MODE]
PROPERTIES (property [,property])
```

#### パラメータ説明

- 収集タイプ
  - FULL：全量収集。
  - SAMPLE：サンプリング収集。
  - 収集タイプを指定しない場合、デフォルトは全量収集です。

- `WITH SYNC | ASYNC MODE`: 指定しない場合、デフォルトは同期操作です。

- `col_name`: 統計情報を収集する列、複数の列はカンマで区切ります。指定しない場合は、テーブル全体の情報を収集します。

- PROPERTIES: 収集タスクのカスタムパラメータ。設定しない場合は、`fe.conf`のデフォルト設定を使用します。タスク実行中に使用されるPROPERTIESは、SHOW ANALYZE STATUSの結果にある`Properties`列で確認できます。

| **PROPERTIES**                | **タイプ** | **デフォルト値** | **説明**                                                     |
| ----------------------------- | -------- | ---------- | ------------------------------------------------------------ |
| statistic_sample_collect_rows | INT      | 200000     | 最小サンプリング行数。パラメータの値が実際のテーブル行数を超える場合、デフォルトで全量収集されます。 |

#### 例

例1：手動で全量収集

```SQL
-- 手動で指定したテーブルの統計情報を全量収集、デフォルト設定を使用。
ANALYZE TABLE tbl_name;

-- 手動で指定したテーブルの統計情報を全量収集、デフォルト設定を使用。
ANALYZE FULL TABLE tbl_name;

-- 手動で指定したテーブルの特定の列の統計情報を全量収集、デフォルト設定を使用。
ANALYZE TABLE tbl_name(c1, c2, c3);
```

例2：手動でサンプリング収集

```SQL
-- 手動で指定したテーブルの統計情報をサンプリング収集、デフォルト設定を使用。
ANALYZE SAMPLE TABLE tbl_name;

-- 手動で指定したテーブルの特定の列の統計情報をサンプリング収集、サンプリング行数を設定。
ANALYZE SAMPLE TABLE tbl_name (v1, v2, v3) PROPERTIES(
    "statistic_sample_collect_rows" = "1000000"
);
```

### 手動でヒストグラム統計情報を収集

ヒストグラムについては、[CBO統計情報](../../../using_starrocks/Cost_based_optimizer.md#統計情報データタイプ)を参照してください。

#### 構文

```SQL
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON col_name [, col_name]
[WITH SYNC | ASYNC MODE]
[WITH N BUCKETS]
PROPERTIES (property [,property]);
```

#### パラメータ説明

- `col_name`: 統計情報を収集する列、複数の列はカンマで区切ります。このパラメータは必須です。

- `WITH SYNC | ASYNC MODE`: 指定しない場合、デフォルトは同期収集です。

- `WITH N BUCKETS`: `N`はヒストグラムのバケット数です。指定しない場合は、`fe.conf`のデフォルト値を使用します。

- PROPERTIES: 収集タスクのカスタムパラメータ。指定しない場合は、`fe.conf`のデフォルト設定を使用します。

| **PROPERTIES**                 | **タイプ** | **デフォルト値** | **説明**                                                     |
| ------------------------------ | -------- | ---------- | ------------------------------------------------------------ |
| statistic_sample_collect_rows  | INT      | 200000     | 最小サンプリング行数。パラメータの値が実際のテーブル行数を超える場合、デフォルトで全量収集されます。 |
| histogram_mcv_size             | INT      | 100        | ヒストグラムのMost Common Value (MCV) の数。                      |
| histogram_sample_ratio         | FLOAT    | 0.1        | ヒストグラムのサンプリング比率。                                             |
| histogram_max_sample_row_count | LONG     | 10000000   | ヒストグラムの最大サンプリング行数。                                         |

ヒストグラムのサンプリング行数は複数のパラメータによって制御され、`statistic_sample_collect_rows`とテーブルの総行数`histogram_sample_ratio`のうち大きい方の値を取ります。`histogram_max_sample_row_count`で指定された行数を超えることはありません。超える場合は、このパラメータで定義された上限行数で収集されます。

ヒストグラムタスク実行中に使用される**PROPERTIES**は、SHOW ANALYZE STATUSの結果にある**PROPERTIES**列で確認できます。

#### 例

```SQL
-- 手動でv1列のヒストグラム情報を収集、デフォルト設定を使用。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1;

-- 手動でv1列のヒストグラム情報を収集、32個のバケットを指定し、MCVを32個に設定し、サンプリング比率を50%に設定。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1,v2 WITH 32 BUCKETS 
PROPERTIES(
   "histogram_mcv_size" = "32",
   "histogram_sample_ratio" = "0.5"
);
```

## 関連文書

[SHOW ANALYZE STATUS](../data-definition/SHOW_ANALYZE_STATUS.md)：現在のすべての**手動収集タスク**の状態を確認します。

[KILL ANALYZE](../data-definition/KILL_ANALYZE.md)：**実行中（Running）**の統計情報収集タスクをキャンセルします。

CBO統計情報収集の詳細については、[CBO統計情報](../../../using_starrocks/Cost_based_optimizer.md)を参照してください。
