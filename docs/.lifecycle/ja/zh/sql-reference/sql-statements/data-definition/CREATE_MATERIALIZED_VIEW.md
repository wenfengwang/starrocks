---
displayed_sidebar: Chinese
---

# CREATE MATERIALIZED VIEW

## 機能

物化ビューを作成します。物化ビューが適用されるシナリオについては、[同期物化ビュー](../../../using_starrocks/Materialized_view-single_table.md)と[非同期物化ビュー](../../../using_starrocks/Materialized_view.md)を参照してください。

物化ビューの作成は非同期の操作です。このコマンドが成功すると、物化ビューの作成タスクが正常に提出されたことを意味します。現在のデータベースで同期物化ビューの構築状態を確認するには、[SHOW ALTER MATERIALIZED VIEW](../data-manipulation/SHOW_ALTER_MATERIALIZED_VIEW.md)コマンドを使用するか、[Information Schema](../../../reference/overview-pages/information_schema.md)の[`tasks`](../../../reference/information_schema/tasks.md)と[`task_runs`](../../../reference/information_schema/task_runs.md)を照会して、非同期物化ビューの構築状態を確認できます。

> **注意**
>
> 基表があるデータベースのCREATE MATERIALIZED VIEW権限を持つユーザーのみが物化ビューを作成できます。

StarRocksはv2.4から非同期物化ビューをサポートしています。非同期物化ビューは、以前のバージョンの同期物化ビューとは以下の点で異なります：

|                              | **単表集約** | **複数表結合** | **クエリ改写** | **リフレッシュ戦略** | **基表** |
| ---------------------------- | ----------- | ---------- | ----------- | ---------- | -------- |
| **非同期物化ビュー** | はい | はい | はい | <ul><li>非同期リフレッシュ</li><li>手動リフレッシュ</li></ul> | 複数表からの構築をサポート。基表は以下から来ることができます：<ul><li>Default Catalog</li><li>External Catalog（v2.5以降）</li><li>既存の非同期物化ビュー（v2.5以降）</li><li>既存のビュー（v3.1以降）</li></ul> |
| **同期物化ビュー（Rollup）** | 一部の集約関数のみ | いいえ | はい | インポート同期リフレッシュ | Default Catalogに基づく単表構築のみをサポート |

## 同期物化ビュー

### 構文

```SQL
CREATE MATERIALIZED VIEW [IF NOT EXISTS] [database.]<mv_name>
[COMMENT ""]
[PROPERTIES ("key"="value", ...)]
AS 
<query_statement>
```

### パラメータ

**mv_name**（必須）

物化ビューの名前。命名規則は以下の通りです：

- 英字（a-zまたはA-Z）、数字（0-9）、アンダースコア（_）のみを含む必要があり、英字で始まる必要があります。
- 全体の長さは64文字を超えてはいけません。
- ビュー名は大文字と小文字を区別します。

**COMMENT**（オプション）

物化ビューのコメントです。物化ビューを作成する際には、`COMMENT`は`mv_name`の後に指定する必要があります。そうでないと作成に失敗します。

**query_statement**（必須）

物化ビューを作成するためのクエリステートメントで、その結果が物化ビューのデータになります。構文は以下の通りです：

```SQL
SELECT select_expr[, select_expr ...]
[WHERE where_expr]
[GROUP BY column_name[, column_name ...]]
[ORDER BY column_name[, column_name ...]]
```

- select_expr（必須）

  同期物化ビューを構築するためのクエリステートメントです。

  - 単一列または集約列：`SELECT a, b, c FROM table_a`のように、`a`、`b`、`c`は基表の列名です。物化ビューに列名を指定しない場合、StarRocksが自動的にこれらの列に名前を付けます。
  - 式：`SELECT a+1 AS x, b+2 AS y, c*c AS z FROM table_a`のように、`a+1`、`b+2`、`c*c`は基表の列を含む式で、`x`、`y`、`z`は物化ビューの列名です。

  > **説明**
  >
  > - このパラメータには少なくとも1つの単一列が含まれている必要があります。
  > - 集約関数を使用して同期物化ビューを作成する場合、GROUP BY句を指定し、`select_expr`に少なくとも1つのGROUP BY列を指定する必要があります。
  > - 同期物化ビューはJOIN、WHERE、GROUP BYのHAVING句をサポートしていません。
  > - v3.1から、各同期物化ビューは基表の各列に複数の集約関数を使用することをサポートし、`select b, sum(a), min(a) from table group by b`のようなクエリステートメントをサポートしています。
  > - v3.1から、同期物化ビューはSELECTと集約関数の複雑な式をサポートし、`select b, sum(a + 1) as sum_a1, min(cast (a as bigint)) as min_a from table group by b`や`select abs(b) as col1, a + 1 as col2, cast(a as bigint) as col3 from table`のようなクエリステートメントが可能です。同期物化ビューの複雑な式には以下の制限があります：
  >   - 各複雑な式には列名が1つ必要であり、基表のすべての同期物化ビューで異なる複雑な式のエイリアスは異なっている必要があります。例えば、`select b, sum(a + 1) as sum_a from table group by b`と`select b, sum(a) as sum_a from table group by b`のクエリステートメントは、同じ基表に対して同時に同期物化ビューを作成することはできませんが、同じ複雑な式に対して異なるエイリアスを複数設定することはできます。
  >   - `EXPLAIN <sql_statement>`を実行して、クエリが複雑な式を使用して作成された同期物化ビューによって改写されているかどうかを確認できます。詳細は[クエリ分析](../../../administration/Query_planning.md)を参照してください。

- WHERE（オプション）

  v3.2から、同期物化ビューはWHERE句を使用してデータをフィルタリングすることをサポートしています。

- GROUP BY（オプション）

  物化ビューのクエリステートメントのグループ化列を構築します。このパラメータを指定しない場合、デフォルトではデータはグループ化されません。

- ORDER BY（オプション）

  物化ビューのクエリステートメントのソート列を構築します。

  - ソート列の宣言順序は、`select_expr`の列の宣言順序と一致している必要があります。
  - クエリステートメントにグループ化列が含まれている場合、ソート列はグループ化列と一致している必要があります。
  - ソート列を指定しない場合、システムは以下のルールに基づいて自動的にソート列を補完します：
  
    - 物化ビューが集約タイプの場合、すべてのグループ化列は自動的にソート列として補完されます。
    - 物化ビューが非集約タイプの場合、システムはプレフィックス列に基づいて自動的にソート列を選択します。

### 同期物化ビューのクエリ

同期物化ビューは本質的には基表のインデックスであり、物理表ではないため、Hint `[_SYNC_MV_]`を使用してのみ同期物化ビューをクエリできます：

```SQL
-- Hintの角括弧[]を省略しないでください。
SELECT * FROM <mv_name> [_SYNC_MV_];
```

> **注意**
>
> 現在、StarRocksは同期物化ビューの列に自動的に名前を生成します。同期物化ビューの列に指定したエイリアスは有効になりません。

### 同期物化ビューのクエリ自動改写

同期物化ビューを使用してクエリする場合、元のクエリステートメントは自動的に改写され、物化ビューに保存されている中間結果をクエリするために使用されます。

以下の表は、元のクエリの集約関数と同期物化ビューを構築するために使用される集約関数のマッチング関係を示しています。ビジネスシナリオに応じて、対応する集約関数を選択して同期物化ビューを構築できます。

| **元のクエリの集約関数**                                   | **物化ビューの構築集約関数** |
| ------------------------------------------------------ | ------------------------ |
| sum                                                    | sum                      |
| min                                                    | min                      |
| max                                                    | max                      |
| count                                                  | count                    |
| bitmap_union, bitmap_union_count, count(distinct)      | bitmap_union             |
| hll_raw_agg, hll_union_agg, ndv, approx_count_distinct | hll_union                |

## 非同期物化ビュー

### 構文

```SQL
CREATE MATERIALIZED VIEW [IF NOT EXISTS] [database.]<mv_name>
[COMMENT ""]
-- distribution_desc
[DISTRIBUTED BY HASH(<bucket_key>[,<bucket_key2> ...]) [BUCKETS <bucket_number>]]
-- refresh_desc
[REFRESH 
-- refresh_moment
    [IMMEDIATE | DEFERRED]
-- refresh_scheme
    [ASYNC [START (<start_time>)] [EVERY (INTERVAL <refresh_interval>)] | MANUAL]
]
-- partition_expression
[PARTITION BY 
    {<date_column> | date_trunc(fmt, <date_column>)}
]
-- order_by_expression
[ORDER BY (<sort_key>)]
[PROPERTIES ("key"="value", ...)]
AS 
<query_statement>
```

### パラメータ

**mv_name**（必須）

物化ビューの名前。命名規則は以下の通りです：

- 英字（a-zまたはA-Z）、数字（0-9）、アンダースコア（_）のみを含む必要があり、英字で始まる必要があります。
- 全体の長さは64文字を超えてはいけません。
- ビュー名は大文字と小文字を区別します。

> **注意**
>
> 同じ基表に対して複数の非同期物化ビューを作成できますが、同じデータベース内で非同期物化ビューの名前が重複してはいけません。

**COMMENT**（オプション）

物化ビューのコメントです。物化ビューを作成する際には、`COMMENT`は`mv_name`の後に指定する必要があります。そうでないと作成に失敗します。

**distribution_desc**（オプション）

非同期物化ビューのバケット分散方式には、ハッシュバケットとランダムバケット（v3.1以降）があります。このパラメータを指定しない場合、StarRocksはランダムバケット方式を使用し、自動的にバケット数を設定します。

> **説明**
>
> 非同期物化ビューを作成する際には、`distribution_desc`と`refresh_scheme`の少なくとも一方を指定する必要があります。

- **ハッシュバケット**：

  構文

  ```SQL
  DISTRIBUTED BY HASH (<bucket_key1>[,<bucket_key2> ...]) [BUCKETS <bucket_number>]
  ```

  詳細については、[バケット分散](../../../table_design/Data_distribution.md#分桶)を参照してください。

  > **説明**
  >
  > v2.5.7以降、StarRocksはテーブル作成やパーティション追加時に自動的にバケット数（BUCKETS）を設定することをサポートしています。手動でバケット数を設定する必要はありません。詳細は、[バケット数の決定](../../../table_design/Data_distribution.md#设置分桶数量)を参照してください。

- **ランダムバケット**：

  ランダムバケット方式を選択し、自動的にバケット数を設定する場合は、`distribution_desc`を指定する必要はありません。手動でバケット数を設定する必要がある場合は、以下の構文を使用します：

  ```SQL
  DISTRIBUTED BY RANDOM BUCKETS <bucket_number>
  ```

  > **注意**
  >
  > ランダムバケット方式を使用する非同期物化ビューは、Colocation Groupの設定をサポートしていません。

  詳細については、[ランダムバケット](../../../table_design/Data_distribution.md#随机分桶自-v31)を参照してください。

**refresh_moment**（オプション）

物化ビューのリフレッシュタイミングです。デフォルト値は`IMMEDIATE`です。有効な値は以下の通りです：

- `IMMEDIATE`：非同期物化ビューが作成された直後にリフレッシュされます。
- `DEFERRED`：非同期物化ビューが作成された後、リフレッシュは行われません。手動でリフレッシュをトリガーするか、定期的なタスクを作成してリフレッシュを行うことができます。

**refresh_scheme**（オプション）

> **説明**
>
> 非同期物化ビューを作成する際には、`distribution_desc`と`refresh_scheme`の少なくとも一方を指定する必要があります。

物化ビューのリフレッシュ方式です。このパラメータは以下の値をサポートしています：


- `ASYNC`: 非同期リフレッシュモード。基本テーブルのデータが変更されるたびに、マテリアライズドビューは事前に定義されたリフレッシュ間隔に基づいて自動的にリフレッシュされます。リフレッシュ開始時間 `START('yyyy-MM-dd hh:mm:ss')` とリフレッシュ間隔 `EVERY (interval n day/hour/minute/second)` をさらに指定することもできます。リフレッシュ間隔は `DAY`、`HOUR`、`MINUTE`、`SECOND` のみをサポートしています。例: `ASYNC START ('2023-09-12 16:30:25') EVERY (INTERVAL 5 MINUTE)`。リフレッシュ間隔が指定されていない場合、デフォルト値の `10 MINUTE` が使用されます。
- `MANUAL`: マニュアルリフレッシュモード。マテリアライズドビューは自動的にリフレッシュされず、リフレッシュタスクはユーザーによって手動でトリガーすることしかできません。

このパラメータが指定されていない場合、デフォルトで `MANUAL` モードが使用されます。

**partition_expression**（オプション）

非同期マテリアライズドビューのパーティション式。現在、非同期マテリアライズドビューの作成時には1つのパーティション式のみを使用できます。

> **注意**
>
> 非同期マテリアライズドビューでは、リストパーティション戦略はサポートされていません。

このパラメータは以下の値をサポートしています：

- `date_column`: パーティション化する列の名前。`PARTITION BY dt` のような形式で、`dt` 列を基準にパーティション化します。
- `date_trunc` 関数: `PARTITION BY date_trunc("MONTH", dt)` のような形式で、`dt` 列を月単位で切り捨ててパーティション化します。date_trunc 関数は `YEAR`、`MONTH`、`DAY`、`HOUR`、`MINUTE` の単位で切り捨てることができます。
- `str2date` 関数: 基本テーブルの STRING 型のパーティションキーをマテリアライズドビューのパーティションキーに変換するための関数です。`PARTITION BY str2date(dt, "%Y%m%d")` のように使用します。`dt` 列は STRING 型の日付であり、日付の形式は `"%Y%m%d"` です。str2date 関数は複数の日付形式をサポートしています。詳細は[str2date](../../sql-functions/date-time-functions/str2date.md)を参照してください。v3.1.4 以降でサポートされています。
- `time_slice` または `date_slice` 関数: v3.1 以降、指定した時間粒度周期に基づいて、指定した時間をその時間粒度周期の開始または終了時刻に変換するために、time_slice または date_slice 関数を使用することができます。例えば、`PARTITION BY date_trunc("MONTH", time_slice(dt, INTERVAL 7 DAY))` のように使用します。time_slice または date_slice の時間粒度は `date_trunc` の時間粒度よりも細かくする必要があります。これらの関数を使用して、パーティションキーよりも細かい時間粒度の GROUP BY 列を指定することができます。例えば、`GROUP BY time_slice(dt, INTERVAL 1 MINUTE) PARTITION BY date_trunc('DAY', ts)`。

このパラメータが指定されていない場合、マテリアライズドビューはパーティションを持たないものとなります。

**order_by_expression**（オプション）

非同期マテリアライズドビューのソートキー。このパラメータが指定されていない場合、StarRocks は SELECT 列から一部の接頭辞をソートキーとして選択します。例: `select a, b, c, d` の場合、ソート列は `a` と `b` のいずれかになる可能性があります。このパラメータはStarRocks 3.0 以降でサポートされています。

**PROPERTIES**（オプション）

非同期マテリアライズドビューのプロパティ。[ALTER MATERIALIZED VIEW](./ALTER_MATERIALIZED_VIEW.md) を使用して既存の非同期マテリアライズドビューのプロパティを変更することができます。

- `session.`: マテリアライズドビューに関連するセッション変数のプロパティを変更する場合は、プロパティの前に `session.` 接頭辞を付ける必要があります。例えば、`session.query_timeout` のようにします。セッション以外のプロパティ（例: `mv_rewrite_staleness_second`）では接頭辞を指定する必要はありません。
- `replication_num`: マテリアライズドビューのレプリケーション数を指定します。
- `storage_medium`: ストレージメディアのタイプを指定します。有効な値は `HDD` と `SSD` です。
- `storage_cooldown_time`: ストレージメディアを SSD に設定した場合、このプロパティで指定した時刻以降にこのパーティションが SSD から HDD に降冷されるようにします。設定する時刻は現在の時刻よりも大きくする必要があります。このプロパティが指定されていない場合、自動的な降冷は行われません。時刻の形式は "yyyy-MM-dd HH:mm:ss" です。
- `partition_ttl`: マテリアライズドビューのパーティションの有効期限（TTL）を指定します。指定された時間範囲内のデータが保持され、期限が切れたパーティションは自動的に削除されます。単位は `YEAR`、`MONTH`、`DAY`、`HOUR`、`MINUTE` です。例えば、このプロパティを `2 MONTH`（2ヶ月）に設定することができます。このプロパティを使用することをお勧めしますが、`partition_ttl_number` の使用は推奨されません。v3.1.5 以降でサポートされています。
- `partition_ttl_number`: 保持する最新のマテリアライズドビューのパーティション数を指定します。現在の時刻よりも小さい開始時刻を持つパーティションについては、この値を超える数のパーティションが存在する場合、余分なパーティションは削除されます。StarRocks は FE の設定項目 `dynamic_partition_check_interval_seconds` の間隔で定期的にマテリアライズドビューのパーティションをチェックし、期限が切れたパーティションを自動的に削除します。[ダイナミックパーティション](../../../table_design/dynamic_partitioning.md)のシナリオでは、事前に作成された未来のパーティションは TTL の対象外です。デフォルト値は `-1` です。値が `-1` の場合、マテリアライズドビューのすべてのパーティションが保持されます。
- `partition_refresh_number`: 1回のリフレッシュでリフレッシュする最大のパーティション数。リフレッシュするパーティションの数がこの値を超える場合、StarRocks はリフレッシュタスクを分割してバッチごとに完了させます。前のバッチのパーティションが正常にリフレッシュされた場合のみ、StarRocks は次のバッチのパーティションをリフレッシュし続けます。パーティションのリフレッシュに失敗した場合、後続のリフレッシュタスクは生成されません。デフォルト値は `-1` です。値が `-1` の場合、リフレッシュタスクは分割されません。
- `excluded_trigger_tables`: このプロパティにリストされている基本テーブルは、データが変更された場合に対応するマテリアライズドビューの自動リフレッシュがトリガされません。このパラメータはインポートトリガに対してのみ適用され、通常はプロパティ `auto_refresh_partitions_limit` と組み合わせて使用します。形式は `[db_name.]table_name` です。デフォルト値は空の文字列です。値が空の文字列の場合、基本テーブルのデータの変更はすべて対応するマテリアライズドビューのリフレッシュをトリガします。
- `auto_refresh_partitions_limit`: マテリアライズドビューのリフレッシュがトリガされる場合にリフレッシュする必要のある最新のマテリアライズドビューパーティションの数を指定します。このプロパティを使用してリフレッシュ範囲を制限することで、リフレッシュのコストを削減できますが、一部のパーティションのみがリフレッシュされるため、マテリアライズドビューのデータが基本テーブルと一致しない可能性があります。デフォルト値は `-1` です。値が `-1` の場合、StarRocks はすべてのパーティションをリフレッシュします。パラメータ値が正の整数 N の場合、StarRocks は既存のパーティションを時系列順に並べ替え、現在のパーティションと N-1 個の過去のパーティションをリフレッシュします。パーティション数が N 未満の場合、すべての既存のパーティションがリフレッシュされます。マテリアライズドビューに事前に作成された未来のパーティションが存在する場合、すべての事前に作成されたパーティションがリフレッシュされます。
- `mv_rewrite_staleness_second`: 現在のマテリアライズドビューの最後のリフレッシュがこのプロパティで指定された時間間隔内にある場合、マテリアライズドビューはクエリのリライトに使用できます。基本テーブルのデータが更新されているかどうかに関係なく、最後のリフレッシュ時間がこのプロパティで指定された時間間隔よりも前の場合、マテリアライズドビューの使用は基本テーブルのデータの変更の有無によって決定されます。単位は秒です。v3.0 以降でサポートされています。
- `colocate_with`: 非同期マテリアライズドビューのコロケーショングループ。詳細は [コロケーションジョイン](../../../using_starrocks/Colocate_join.md) を参照してください。v3.0 以降でサポートされています。
- `unique_constraints` および `foreign_key_constraints`: View Delta Join クエリのリライトに使用される非同期マテリアライズドビューのユニークキー制約および外部キー制約。詳細は [非同期マテリアライズドビュー - View Delta Join クエリのリライト](../../../using_starrocks/query_rewrite_with_materialized_views.md#view-delta-join-改写) を参照してください。v3.0 以降でサポートされています。
- `resource_group`: マテリアライズドビューのリフレッシュタスクにリソースグループを設定します。リソースグループに関する詳細は、[リソースグループ](../../../administration/resource_group.md) を参照してください。
- `query_rewrite_consistency`: 現在の非同期マテリアライズドビューのクエリリライトルールを指定します。v3.2 以降でサポートされています。有効な値は次のとおりです：
  - `disable`: この非同期マテリアライズドビューを使用した自動クエリリライトを無効にします。
  - `checked`（デフォルト値）: クエリリライトを自動的に有効にするためには、非同期マテリアライズドビューがタイムリーな要件を満たしている必要があります。具体的には、次の条件を満たす場合にのみ、マテリアライズドビューをクエリリライトに使用できます：
    - `mv_rewrite_staleness_second` が指定されていない場合、マテリアライズドビューのデータがすべての基本テーブルのデータと一致している場合。
    - `mv_rewrite_staleness_second` が指定されている場合、最後のリフレッシュが staleness の時間間隔内にある場合。
  - `loose`: 一貫性のチェックを行わずに、直接クエリリライトを有効にします。
- `force_external_table_query_rewrite`: External Catalog ベースのマテリアライズドビューのクエリリライトを有効にするかどうかを指定します。v3.2 以降でサポートされています。有効な値は次のとおりです：
  - `true`: External Catalog ベースのマテリアライズドビューのクエリリライトを有効にします。
  - `false`（デフォルト値）: External Catalog ベースのマテリアライズドビューのクエリリライトを無効にします。

  基本テーブルと External Catalog ベースのマテリアライズドビューのデータの強い整合性を保証することはできないため、デフォルトではこの機能は無効になっています。この機能を有効にすると、マテリアライズドビューは `query_rewrite_consistency` で指定されたルールに基づいてクエリをリライトします。

  > **注意**
  >
  > ユニークキー制約と外部キー制約はクエリリライトにのみ使用されます。データのインポート時に外部キー制約のチェックは行われません。インポートされるデータが制約条件を満たしていることを確認する必要があります。

**query_statement**（必須）

非同期マテリアライズドビューを作成するためのクエリ文。その結果が非同期マテリアライズドビューのデータとなります。v3.1.6 以降、StarRocks は Common Table Expression (CTE) を使用して非同期マテリアライズドビューを作成することができます。

> **注意**
>
> 非同期マテリアライズドビューは、リストパーティションを使用している基本テーブルをサポートしていません。

### 非同期マテリアライズドビューのクエリ

非同期マテリアライズドビューは実体テーブルです。直接データをインポートする以外の操作において、通常のテーブルと同様に使用することができます。

### サポートされるデータ型

- Default Catalog ベースで作成された非同期マテリアライズドビューは、次のデータ型をサポートしています：

  - **日付型**: DATE、DATETIME
  - **文字列型**: CHAR、VARCHAR
  - **数値型**: BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL、PERCENTILE
  - **半構造化型**: ARRAY、JSON、MAP（v3.1 以降）、STRUCT（v3.1 以降）
  - **その他の型**: BITMAP、HLL

> **注意**
>
> v2.4.5 以降、BITMAP、HLL、PERCENTILE がサポートされています。

- External Catalog ベースで作成された非同期マテリアライズドビューは、次のデータ型をサポートしています：

  - Hive Catalog

    - **数値型**: INT/INTEGER、BIGINT、DOUBLE、FLOAT、DECIMAL
    - **日付型**: TIMESTAMP
    - **文字列型**: STRING、VARCHAR、CHAR
    - **半構造化型**: ARRAY
  - Hudi カタログ

    - **数値型**: BOOLEAN、INT、LONG、FLOAT、DOUBLE、DECIMAL
    - **日付型**: DATE、TimeMillis/TimeMicros、TimestampMillis/TimestampMicros
    - **文字列型**: STRING
    - **半構造化型**: ARRAY

  - Iceberg カタログ

    - **数値型**: BOOLEAN、INT、LONG、FLOAT、DOUBLE、DECIMAL(P, S)
    - **日付型**: DATE、TIME、TIMESTAMP
    - **文字列型**: STRING、UUID、FIXED(L)、BINARY
    - **半構造化型**: LIST

## 注意事項

- 現在のバージョンでは、複数の物化ビューを同時に作成することはできません。現在の作成タスクが完了した後に、次の作成タスクを実行できます。

- 同期物化ビューについて：

  - 同期物化ビューは単一列の集約関数のみをサポートし、`sum(a+b)` のようなクエリ文はサポートしていません。
  - 同期物化ビューは、同一列に対して一種類の集約関数のみをサポートし、`select sum(a), min(a) from table` のようなクエリ文はサポートしていません。
  - 同期物化ビューで集約関数を使用する場合、GROUP BY 句と一緒に使用する必要があり、SELECT する列には少なくとも一つのグループ列が含まれている必要があります。
  - 同期物化ビューの作成文は JOIN や GROUP BY の HAVING 句をサポートしていません。
  - ALTER TABLE DROP COLUMN を使用してベーステーブルの特定の列を削除する場合、そのベーステーブルのすべての同期物化ビューに削除される列が含まれていないことを保証する必要があります。そうでないと削除操作を行うことができません。その列を削除する必要がある場合は、その列を含むすべての同期物化ビューを削除した後に、列の削除操作を行う必要があります。
  - 一つのテーブルに多くの同期物化ビューを作成すると、インポートの効率に影響を与えます。データをインポートする際、同期物化ビューとベーステーブルのデータは同時に更新され、ベーステーブルに n 個の物化ビューが含まれている場合、そのテーブルにデータをインポートする効率は n 個のテーブルにデータをインポートするのとほぼ同じになり、データのインポート速度が遅くなります。

- ネストされた非同期物化ビューについて：

  - 各非同期物化ビューのリフレッシュ方法は、現在の物化ビューにのみ影響します。
  - 現在の StarRocks はネストレベルに制限を設けていませんが、本番環境ではネストレベルを三層を超えないことを推奨します。

- 外部データディレクトリの非同期物化ビューについて：

  - 外部データディレクトリの物化ビューは、非同期の定期リフレッシュと手動リフレッシュのみをサポートしています。
  - 物化ビューのデータは外部データディレクトリのデータとの強い一貫性を保証しません。
  - 現在、リソース(Resource)に基づいて物化ビューを構築することはサポートしていません。
  - StarRocks は現在、外部データディレクトリのベーステーブルのデータが変更されたかどうかを感知することができません。そのため、リフレッシュする際にはすべてのパーティションがデフォルトでリフレッシュされます。手動リフレッシュを使用して、特定のパーティションのみをリフレッシュすることができます。

## 例

### 同期物化ビューの例

ベーステーブルの構造は以下の通りです：

```sql
mysql> desc duplicate_table;
+-------+--------+------+------+---------+-------+
| Field | Type   | Null | Key  | Default | Extra |
+-------+--------+------+------+---------+-------+
| k1    | INT    | Yes  | true | N/A     |       |
| k2    | INT    | Yes  | true | N/A     |       |
| k3    | BIGINT | Yes  | true | N/A     |       |
| k4    | BIGINT | Yes  | true | N/A     |       |
+-------+--------+------+------+---------+-------+
```

1. 元のテーブルの列 (k1, k2) のみを含む物化ビューを作成します。

    ```sql
    create materialized view k1_k2 as
    select k1, k2 from duplicate_table;
    ```

    物化ビューのスキーマは以下の通りで、k1、k2 の2列のみを含み、集約はありません。

    ```sql
    +-----------------+-------+--------+------+------+---------+-------+
    | IndexName       | Field | Type   | Null | Key  | Default | Extra |
    +-----------------+-------+--------+------+------+---------+-------+
    | k1_k2           | k1    | INT    | Yes  | true | N/A     |       |
    |                 | k2    | INT    | Yes  | true | N/A     |       |
    +-----------------+-------+--------+------+------+---------+-------+
    ```

2. k2 をソート列とする物化ビューを作成します。

    ```sql
    create materialized view k2_order as
    select k2, k1 from duplicate_table order by k2;
    ```

    物化ビューのスキーマは以下の通りで、k2、k1 の2列のみを含み、k2 がソート列で、集約はありません。

    ```sql
    +-----------------+-------+--------+------+-------+---------+-------+
    | IndexName       | Field | Type   | Null | Key   | Default | Extra |
    +-----------------+-------+--------+------+-------+---------+-------+
    | k2_order        | k2    | INT    | Yes  | true  | N/A     |       |
    |                 | k1    | INT    | Yes  | false | N/A     | NONE  |
    +-----------------+-------+--------+------+-------+---------+-------+
    ```

3. k1、k2 をグループ化し、k3 列を SUM 集約する物化ビューを作成します。

    ```sql
    create materialized view k1_k2_sumk3 as
    select k1, k2, sum(k3) from duplicate_table group by k1, k2;
    ```

    物化ビューのスキーマは以下の通りで、k1、k2、sum(k3) の3列を含み、k1、k2 がグループ化列で、sum(k3) は k1、k2 によるグループ化後の k3 列の合計値です。

    物化ビューにソート列が宣言されておらず、集約データを含むため、システムはデフォルトでグループ化列 k1、k2 をソート列として補完します。

    ```sql
    +-----------------+-------+--------+------+-------+---------+-------+
    | IndexName       | Field | Type   | Null | Key   | Default | Extra |
    +-----------------+-------+--------+------+-------+---------+-------+
    | k1_k2_sumk3     | k1    | INT    | Yes  | true  | N/A     |       |
    |                 | k2    | INT    | Yes  | true  | N/A     |       |
    |                 | k3    | BIGINT | Yes  | false | N/A     | SUM   |
    +-----------------+-------+--------+------+-------+---------+-------+
    ```

4. 重複行を除去する物化ビューを作成します。

    ```sql
    create materialized view deduplicate as
    select k1, k2, k3, k4 from duplicate_table group by k1, k2, k3, k4;
    ```

    物化ビューのスキーマは以下の通りで、k1、k2、k3、k4 の4列を含み、重複行はありません。

    ```sql
    +-----------------+-------+--------+------+-------+---------+-------+
    | IndexName       | Field | Type   | Null | Key   | Default | Extra |
    +-----------------+-------+--------+------+-------+---------+-------+
    | deduplicate     | k1    | INT    | Yes  | true  | N/A     |       |
    |                 | k2    | INT    | Yes  | true  | N/A     |       |
    |                 | k3    | BIGINT | Yes  | true  | N/A     |       |
    |                 | k4    | BIGINT | Yes  | true  | N/A     |       |
    +-----------------+-------+--------+------+-------+---------+-------+

    ```

5. ソート列を宣言せずに非集約型の物化ビューを作成します。

    all_type_table のスキーマは以下の通りです：

    ```sql
    +-------+--------------+------+-------+---------+-------+
    | Field | Type         | Null | Key   | Default | Extra |
    +-------+--------------+------+-------+---------+-------+
    | k1    | TINYINT      | Yes  | true  | N/A     |       |
    | k2    | SMALLINT     | Yes  | true  | N/A     |       |
    | k3    | INT          | Yes  | true  | N/A     |       |
    | k4    | BIGINT       | Yes  | true  | N/A     |       |
    | k5    | DECIMAL(9,0) | Yes  | true  | N/A     |       |
    | k6    | DOUBLE       | Yes  | false | N/A     | NONE  |
    | k7    | VARCHAR(20)  | Yes  | false | N/A     | NONE  |
    +-------+--------------+------+-------+---------+-------+
    ```

    物化ビューには k3、k4、k5、k6、k7 の列が含まれ、ソート列は宣言されていません。作成文は以下の通りです：

    ```sql
    create materialized view mv_1 as
    select k3, k4, k5, k6, k7 from all_type_table;
    ```

    システムによってデフォルトで補完されたソート列は k3、k4、k5 の3列です。これらの列の型のバイト数の合計は 4(INT) + 8(BIGINT) + 16(DECIMAL) = 28 < 36 です。したがって、これら3列がソート列として補完されます。
    物化ビューのスキーマは以下の通りで、k3、k4、k5 列の key フィールドが true であり、これがソート列です。k6、k7 列の key フィールドは false であり、これが非ソート列です。

    ```sql
    +----------------+-------+--------------+------+-------+---------+-------+
    | IndexName      | Field | Type         | Null | Key   | Default | Extra |
    +----------------+-------+--------------+------+-------+---------+-------+
    | mv_1           | k3    | INT          | Yes  | true  | N/A     |       |
    |                | k4    | BIGINT       | Yes  | true  | N/A     |       |
    |                | k5    | DECIMAL(9,0) | Yes  | true  | N/A     |       |
    |                | k6    | DOUBLE       | Yes  | false | N/A     | NONE  |
    |                | k7    | VARCHAR(20)  | Yes  | false | N/A     | NONE  |
    +----------------+-------+--------------+------+-------+---------+-------+
    ```

6. WHERE 句と複雑な式を使用して同期物化ビューを作成します。

  ```sql
  -- user_event ベーステーブルの作成
  CREATE TABLE user_event (
      ds date   NOT NULL,
      id  varchar(256)    NOT NULL,
      user_id int DEFAULT NULL,
      user_id1    varchar(256)    DEFAULT NULL,
      user_id2    varchar(256)    DEFAULT NULL,
      column_01   int DEFAULT NULL,
      column_02   int DEFAULT NULL,
      column_03   int DEFAULT NULL,
      column_04   int DEFAULT NULL,
      column_05   int DEFAULT NULL,
      column_06   DECIMAL(12,2)   DEFAULT NULL,
      column_07   DECIMAL(12,3)   DEFAULT NULL,
      column_08   JSON   DEFAULT NULL,
      column_09   DATETIME    DEFAULT NULL,
      column_10   DATETIME    DEFAULT NULL,
      column_11   DATE    DEFAULT NULL,
      column_12   varchar(256)    DEFAULT NULL,
      column_13   varchar(256)    DEFAULT NULL,
      column_14   varchar(256)    DEFAULT NULL,
      column_15   varchar(256)    DEFAULT NULL,
      column_16   varchar(256)    DEFAULT NULL,
      column_17   varchar(256)    DEFAULT NULL,
      column_18   varchar(256)    DEFAULT NULL,
      column_19   varchar(256)    DEFAULT NULL,
      column_20   varchar(256)    DEFAULT NULL,
      column_21   varchar(256)    DEFAULT NULL,
      column_22   varchar(256)    DEFAULT NULL,
      column_23   varchar(256)    DEFAULT NULL,
      column_24   varchar(256)    DEFAULT NULL,
      column_25   varchar(256)    DEFAULT NULL,
      column_26   varchar(256)    DEFAULT NULL,
      column_27   varchar(256)    DEFAULT NULL,

      column_28   varchar(256)    DEFAULT NULL,
      column_29   varchar(256)    DEFAULT NULL,
      column_30   varchar(256)    DEFAULT NULL,
      column_31   varchar(256)    DEFAULT NULL,
      column_32   varchar(256)    DEFAULT NULL,
      column_33   varchar(256)    DEFAULT NULL,
      column_34   varchar(256)    DEFAULT NULL,
      column_35   varchar(256)    DEFAULT NULL,
      column_36   varchar(256)    DEFAULT NULL,
      column_37   varchar(256)    DEFAULT NULL
  )
  PARTITION BY date_trunc("day", ds)
  DISTRIBUTED BY HASH(id);
  
  -- WHERE句と複雑な式を使用して同期マテリアライズドビューを作成する
  CREATE MATERIALIZED VIEW test_mv1
  AS 
  SELECT
  ds,
  column_19,
  column_36,
  sum(column_01) as column_01_sum,
  bitmap_union(to_bitmap(user_id)) as user_id_dist_cnt,
  bitmap_union(to_bitmap(CASE WHEN column_01 > 1 AND column_34 IN ('1','34') THEN user_id2 ELSE NULL END)) as filter_dist_cnt_1,
  bitmap_union(to_bitmap(CASE WHEN column_02 > 60 AND column_35 IN ('11','13') THEN user_id2 ELSE NULL END)) as filter_dist_cnt_2,
  bitmap_union(to_bitmap(CASE WHEN column_03 > 70 AND column_36 IN ('21','23') THEN user_id2 ELSE NULL END)) as filter_dist_cnt_3,
  bitmap_union(to_bitmap(CASE WHEN column_04 > 20 AND column_27 IN ('31','27') THEN user_id2 ELSE NULL END)) as filter_dist_cnt_4,
  bitmap_union(to_bitmap(CASE WHEN column_05 > 90 AND column_28 IN ('41','43') THEN user_id2 ELSE NULL END)) as filter_dist_cnt_5
  FROM user_event
  WHERE ds >= '2023-11-02'
  GROUP BY
  ds,
  column_19,
  column_36;
  ```

### 非同期マテリアライズドビューの例

以下の例は、次のベーステーブルに基づいています。

```SQL
CREATE TABLE `lineorder` (
  `lo_orderkey` int(11) NOT NULL COMMENT "",
  `lo_linenumber` int(11) NOT NULL COMMENT "",
  `lo_custkey` int(11) NOT NULL COMMENT "",
  `lo_partkey` int(11) NOT NULL COMMENT "",
  `lo_suppkey` int(11) NOT NULL COMMENT "",
  `lo_orderdate` int(11) NOT NULL COMMENT "",
  `lo_orderpriority` varchar(16) NOT NULL COMMENT "",
  `lo_shippriority` int(11) NOT NULL COMMENT "",
  `lo_quantity` int(11) NOT NULL COMMENT "",
  `lo_extendedprice` int(11) NOT NULL COMMENT "",
  `lo_ordtotalprice` int(11) NOT NULL COMMENT "",
  `lo_discount` int(11) NOT NULL COMMENT "",
  `lo_revenue` int(11) NOT NULL COMMENT "",
  `lo_supplycost` int(11) NOT NULL COMMENT "",
  `lo_tax` int(11) NOT NULL COMMENT "",
  `lo_commitdate` int(11) NOT NULL COMMENT "",
  `lo_shipmode` varchar(11) NOT NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY(`lo_orderkey`)
COMMENT "OLAP"
PARTITION BY RANGE(`lo_orderdate`)
(PARTITION p1 VALUES [("-2147483648"), ("19930101")),
PARTITION p2 VALUES [("19930101"), ("19940101")),
PARTITION p3 VALUES [("19940101"), ("19950101")),
PARTITION p4 VALUES [("19950101"), ("19960101")),
PARTITION p5 VALUES [("19960101"), ("19970101")),
PARTITION p6 VALUES [("19970101"), ("19980101")),
PARTITION p7 VALUES [("19980101"), ("19990101"]))
DISTRIBUTED BY HASH(`lo_orderkey`);

CREATE TABLE IF NOT EXISTS `customer` (
  `c_custkey` int(11) NOT NULL COMMENT "",
  `c_name` varchar(26) NOT NULL COMMENT "",
  `c_address` varchar(41) NOT NULL COMMENT "",
  `c_city` varchar(11) NOT NULL COMMENT "",
  `c_nation` varchar(16) NOT NULL COMMENT "",
  `c_region` varchar(13) NOT NULL COMMENT "",
  `c_phone` varchar(16) NOT NULL COMMENT "",
  `c_mktsegment` varchar(11) NOT NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY(`c_custkey`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`c_custkey`);

CREATE TABLE IF NOT EXISTS `dates` (
  `d_datekey` int(11) NOT NULL COMMENT "",
  `d_date` varchar(20) NOT NULL COMMENT "",
  `d_dayofweek` varchar(10) NOT NULL COMMENT "",
  `d_month` varchar(11) NOT NULL COMMENT "",
  `d_year` int(11) NOT NULL COMMENT "",
  `d_yearmonthnum` int(11) NOT NULL COMMENT "",
  `d_yearmonth` varchar(9) NOT NULL COMMENT "",
  `d_daynuminweek` int(11) NOT NULL COMMENT "",
  `d_daynuminmonth` int(11) NOT NULL COMMENT "",
  `d_daynuminyear` int(11) NOT NULL COMMENT "",
  `d_monthnuminyear` int(11) NOT NULL COMMENT "",
  `d_weeknuminyear` int(11) NOT NULL COMMENT "",
  `d_sellingseason` varchar(14) NOT NULL COMMENT "",
  `d_lastdayinweekfl` int(11) NOT NULL COMMENT "",
  `d_lastdayinmonthfl` int(11) NOT NULL COMMENT "",
  `d_holidayfl` int(11) NOT NULL COMMENT "",
  `d_weekdayfl` int(11) NOT NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY(`d_datekey`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`d_datekey`);

CREATE TABLE IF NOT EXISTS `supplier` (
  `s_suppkey` int(11) NOT NULL COMMENT "",
  `s_name` varchar(26) NOT NULL COMMENT "",
  `s_address` varchar(26) NOT NULL COMMENT "",
  `s_city` varchar(11) NOT NULL COMMENT "",
  `s_nation` varchar(16) NOT NULL COMMENT "",
  `s_region` varchar(13) NOT NULL COMMENT "",
  `s_phone` varchar(16) NOT NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY(`s_suppkey`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`s_suppkey`);

CREATE TABLE IF NOT EXISTS `part` (
  `p_partkey` int(11) NOT NULL COMMENT "",
  `p_name` varchar(23) NOT NULL COMMENT "",
  `p_mfgr` varchar(7) NOT NULL COMMENT "",
  `p_category` varchar(8) NOT NULL COMMENT "",
  `p_brand` varchar(10) NOT NULL COMMENT "",
  `p_color` varchar(12) NOT NULL COMMENT "",
  `p_type` varchar(26) NOT NULL COMMENT "",
  `p_size` int(11) NOT NULL COMMENT "",
  `p_container` varchar(11) NOT NULL COMMENT ""
) ENGINE=OLAP
DUPLICATE KEY(`p_partkey`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`p_partkey`);

ordersテーブルを作成する ( 
    dt date NOT NULL, 
    order_id bigint NOT NULL, 
    user_id int NOT NULL, 
    merchant_id int NOT NULL, 
    good_id int NOT NULL, 
    good_name string NOT NULL, 
    price int NOT NULL, 
    cnt int NOT NULL, 
    revenue int NOT NULL, 
    state tinyint NOT NULL 
) 
PRIMARY KEY (dt, order_id) 
PARTITION BY RANGE(`dt`) 
( PARTITION p20210820 VALUES [('2021-08-20'), ('2021-08-21')), 
PARTITION p20210821 VALUES [('2021-08-21'), ('2021-08-22')) ) 
DISTRIBUTED BY HASH(order_id)
PROPERTIES (
    "replication_num" = "3", 
    "enable_persistent_index" = "true"
);
```

例1：ソーステーブルから非分割マテリアライズドビューを作成する

```SQL
CREATE MATERIALIZED VIEW lo_mv1
DISTRIBUTED BY HASH(`lo_orderkey`)
REFRESH ASYNC
AS
SELECT
    lo_orderkey, 
    lo_custkey, 
    SUM(lo_quantity) AS total_quantity, 
    SUM(lo_revenue) AS total_revenue, 
    COUNT(lo_shipmode) AS shipmode_count
FROM lineorder 
GROUP BY lo_orderkey, lo_custkey 
ORDER BY lo_orderkey;
```

例2：ソーステーブルから分割マテリアライズドビューを作成する

```SQL
CREATE MATERIALIZED VIEW lo_mv2
PARTITION BY `lo_orderdate`
DISTRIBUTED BY HASH(`lo_orderkey`)
REFRESH ASYNC START('2023-07-01 10:00:00') EVERY (interval 1 day)
AS
SELECT
    lo_orderkey,
    lo_orderdate,
    lo_custkey, 
    SUM(lo_quantity) AS total_quantity, 
    SUM(lo_revenue) AS total_revenue, 
    COUNT(lo_shipmode) AS shipmode_count
FROM lineorder 
GROUP BY lo_orderkey, lo_orderdate, lo_custkey
ORDER BY lo_orderkey;

# `dt`列を月単位で分割するためにdate_trunc関数を使用する。
CREATE MATERIALIZED VIEW order_mv1
PARTITION BY date_trunc('month', `dt`)
DISTRIBUTED BY HASH(`order_id`)
REFRESH ASYNC START('2023-07-01 10:00:00') EVERY (interval 1 day)
AS
SELECT
    dt,
    order_id,
    user_id,
    SUM(cnt) AS total_cnt,
    SUM(revenue) AS total_revenue, 
    COUNT(state) AS state_count
FROM orders
GROUP BY dt, order_id, user_id;
```

例3：非同期マテリアライズドビューを作成する

```SQL
CREATE MATERIALIZED VIEW flat_lineorder
DISTRIBUTED BY HASH(`lo_orderkey`)
REFRESH MANUAL
AS
SELECT
    l.LO_ORDERKEY AS LO_ORDERKEY,
    l.LO_LINENUMBER AS LO_LINENUMBER,
    l.LO_CUSTKEY AS LO_CUSTKEY,
    l.LO_PARTKEY AS LO_PARTKEY,
    l.LO_SUPPKEY AS LO_SUPPKEY,
    l.LO_ORDERDATE AS LO_ORDERDATE,
    l.LO_ORDERPRIORITY AS LO_ORDERPRIORITY,
    l.LO_SHIPPRIORITY AS LO_SHIPPRIORITY,
    l.LO_QUANTITY AS LO_QUANTITY,
    l.LO_EXTENDEDPRICE AS LO_EXTENDEDPRICE,
    l.LO_ORDTOTALPRICE AS LO_ORDTOTALPRICE,
    l.LO_DISCOUNT AS LO_DISCOUNT,
    l.LO_REVENUE AS LO_REVENUE,
    l.LO_SUPPLYCOST AS LO_SUPPLYCOST,
    l.LO_TAX AS LO_TAX,
    l.LO_COMMITDATE AS LO_COMMITDATE,
    l.LO_SHIPMODE AS LO_SHIPMODE,
    c.C_NAME AS C_NAME,
    c.C_ADDRESS AS C_ADDRESS,
    c.C_CITY AS C_CITY,
    c.C_NATION AS C_NATION,
    c.C_REGION AS C_REGION,
    c.C_PHONE AS C_PHONE,
    c.C_MKTSEGMENT AS C_MKTSEGMENT,
    s.S_NAME AS S_NAME,
    s.S_ADDRESS AS S_ADDRESS,
    s.S_CITY AS S_CITY,
    s.S_NATION AS S_NATION,
    s.S_REGION AS S_REGION,
    s.S_PHONE AS S_PHONE,
    p.P_NAME AS P_NAME,
    p.P_MFGR AS P_MFGR,
    p.P_CATEGORY AS P_CATEGORY,
    p.P_BRAND AS P_BRAND,
    p.P_COLOR AS P_COLOR,
    p.P_TYPE AS P_TYPE,
    p.P_SIZE AS P_SIZE,
    p.P_CONTAINER AS P_CONTAINER 
FROM lineorder AS l 
INNER JOIN customer AS c ON c.C_CUSTKEY = l.LO_CUSTKEY
INNER JOIN supplier AS s ON s.S_SUPPKEY = l.LO_SUPPKEY
INNER JOIN part AS p ON p.P_PARTKEY = l.LO_PARTKEY;
```

例4：分割マテリアライズドビューを作成し、基表のSTRING型分割キーを日付型として非同期マテリアライズドビューの分割キーに変換する。

```sql
-- STRING型の分割キーを持つ基表を作成する。
CREATE TABLE `part_dates` (
  `d_date` varchar(20) DEFAULT NULL,
  `d_dayofweek` varchar(10) DEFAULT NULL,
  `d_month` varchar(11) DEFAULT NULL,
  `d_year` int(11) DEFAULT NULL,
  `d_yearmonthnum` int(11) DEFAULT NULL,
  `d_yearmonth` varchar(9) DEFAULT NULL,
  `d_daynuminweek` int(11) DEFAULT NULL,
  `d_daynuminmonth` int(11) DEFAULT NULL,
  `d_daynuminyear` int(11) DEFAULT NULL,
  `d_monthnuminyear` int(11) DEFAULT NULL,
  `d_weeknuminyear` int(11) DEFAULT NULL,
  `d_sellingseason` varchar(14) DEFAULT NULL,
  `d_lastdayinweekfl` int(11) DEFAULT NULL,
  `d_lastdayinmonthfl` int(11) DEFAULT NULL,
  `d_holidayfl` int(11) DEFAULT NULL,
  `d_weekdayfl` int(11) DEFAULT NULL,
  `d_datekey` varchar(11) DEFAULT NULL
) PARTITION BY (d_datekey);


-- `str2date`関数を使用して分割マテリアライズドビューを作成する。
CREATE MATERIALIZED VIEW IF NOT EXISTS `test_mv` 
PARTITION BY str2date(`d_datekey`, '%Y%m%d')
DISTRIBUTED BY HASH(`d_date`, `d_month`, `d_month`) 
REFRESH MANUAL 
AS
SELECT
  `d_date`,
  `d_dayofweek`,
  `d_month`,
  `d_yearmonthnum`,
  `d_yearmonth`,
  `d_daynuminweek`,
  `d_daynuminmonth`,
  `d_daynuminyear`,
  `d_monthnuminyear`,
  `d_weeknuminyear`,
  `d_sellingseason`,
  `d_lastdayinweekfl`,
  `d_lastdayinmonthfl`,
  `d_holidayfl`,
  `d_weekdayfl`,
  `d_datekey`
FROM
`hive_catalog`.`ssb_1g_orc`.`part_dates`;
```

## 更多操作

論理ビューを作成するには、[CREATE VIEW](./CREATE_VIEW.md)を参照してください。
