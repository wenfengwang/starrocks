---
displayed_sidebar: English
---

# CREATE MATERIALIZED VIEW

## 説明

マテリアライズドビューを作成します。マテリアライズドビューの使用方法については、[同期マテリアライズドビュー](../../../using_starrocks/Materialized_view-single_table.md)および[非同期マテリアライズドビュー](../../../using_starrocks/Materialized_view.md)を参照してください。

> **注意**
>
> ベーステーブルが存在するデータベースでCREATE MATERIALIZED VIEW権限を持つユーザーのみがマテリアライズドビューを作成できます。

マテリアライズドビューの作成は非同期操作です。このコマンドが正常に実行された場合、マテリアライズドビューの作成タスクが正常に提出されたことを示します。データベース内の同期マテリアライズドビューのビルドステータスは[SHOW ALTER MATERIALIZED VIEW](../data-manipulation/SHOW_ALTER_MATERIALIZED_VIEW.md)コマンドで、非同期マテリアライズドビューのビルドステータスは[Information Schema](../../../reference/overview-pages/information_schema.md)内のメタデータビュー[`tasks`](../../../reference/information_schema/tasks.md)および[`task_runs`](../../../reference/information_schema/task_runs.md)をクエリすることで確認できます。

StarRocksはv2.4から非同期マテリアライズドビューをサポートしています。以前のバージョンの同期マテリアライズドビューと非同期マテリアライズドビューの主な違いは以下の通りです。

|                       | **単一テーブル集計** | **複数テーブル結合** | **クエリ改写** | **リフレッシュ戦略** | **ベーステーブル** |
| --------------------- | ---------------------------- | -------------------- | ----------------- | -------------------- | -------------- |
| **ASYNC MV** | はい | はい | はい | <ul><li>非同期リフレッシュ</li><li>手動リフレッシュ</li></ul> | 複数テーブル:<ul><li>デフォルトカタログ</li><li>外部カタログ (v2.5)</li><li>既存のマテリアライズドビュー (v2.5)</li><li>既存のビュー (v3.1)</li></ul> |
| **SYNC MV (ロールアップ)**  | 集約関数の選択が限定される | いいえ | はい | データロード時の同期リフレッシュ | デフォルトカタログ内の単一テーブル |

## 同期マテリアライズドビュー

### 構文

```SQL
CREATE MATERIALIZED VIEW [IF NOT EXISTS] [database.]<mv_name>
[COMMENT ""]
[PROPERTIES ("key"="value", ...)]
AS 
<query_statement>
```

角括弧[]内のパラメータはオプションです。

### パラメータ

**mv_name**（必須）

マテリアライズドビューの名前です。命名要件は以下の通りです。

- 名前は、文字（a-zまたはA-Z）、数字（0-9）、またはアンダースコア（_）で構成され、文字で始まる必要があります。
- 名前の長さは64文字を超えてはいけません。
- 名前は大文字と小文字を区別します。

**COMMENT**（オプション）

マテリアライズドビューにコメントを追加します。`COMMENT`は`mv_name`の後に配置する必要があります。そうでないと、マテリアライズドビューを作成できません。

**query_statement**（必須）

マテリアライズドビューを作成するためのクエリステートメントです。その結果はマテリアライズドビューのデータになります。構文は以下の通りです。

```SQL
SELECT select_expr[, select_expr ...]
[WHERE where_expr]
[GROUP BY column_name[, column_name ...]]
[ORDER BY column_name[, column_name ...]]
```

- select_expr（必須）

  クエリステートメントのすべての列、つまりマテリアライズドビュースキーマのすべての列です。このパラメータは以下の値をサポートします。

  - 単純な列や集約列、例えば`SELECT a, abs(b), min(c) FROM table_a`、ここで`a`、`b`、`c`はベーステーブルの列の名前です。マテリアライズドビューの列名を指定しない場合、StarRocksは自動的に列に名前を割り当てます。
  - 式、例えば`SELECT a+1 AS x, b+2 AS y, c*c AS z FROM table_a`、ここで`a+1`、`b+2`、`c*c`はベーステーブルの列を参照する式で、`x`、`y`、`z`はマテリアライズドビューの列に割り当てられたエイリアスです。

  > **注記**
  >
  > - `select_expr`には少なくとも1つの列を指定する必要があります。
  > - 集約関数を使用して同期マテリアライズドビューを作成する場合、GROUP BY句を指定し、`select_expr`に少なくとも1つのGROUP BY列を指定する必要があります。
  > - 同期マテリアライズドビューは、JOINやGROUP BYのHAVING句などの句をサポートしていません。
  > - v3.1以降、各同期マテリアライズドビューは、ベーステーブルの各列に対して複数の集約関数をサポートできます。例えば、`select b, sum(a), min(a) from table group by b`のようなクエリステートメントです。
  > - v3.1以降、同期マテリアライズドビューはSELECTと集約関数の複雑な式をサポートしています。例えば、`select b, sum(a + 1) as sum_a1, min(cast(a as bigint)) as min_a from table group by b`や`select abs(b) as col1, a + 1 as col2, cast(a as bigint) as col3 from table`のようなクエリステートメントです。同期マテリアライズドビューで使用される複雑な式には以下の制限があります。
  >   - 各複雑な式にはエイリアスが必要で、ベーステーブルのすべての同期マテリアライズドビューで異なる複雑な式には異なるエイリアスを割り当てる必要があります。例えば、`select b, sum(a + 1) as sum_a from table group by b`と`select b, sum(a) as sum_a from table group by b`を使用して同じベーステーブルの同期マテリアライズドビューを作成することはできません。複雑な式には異なるエイリアスを設定できます。
  >   - あなたのクエリが複雑な式で作成された同期マテリアライズドビューによって改写されるかどうかは、`EXPLAIN <sql_statement>`を実行することで確認できます。詳細は[クエリ分析](../../../administration/Query_planning.md)を参照してください。

- WHERE（オプション）

  v3.2以降、同期マテリアライズドビューはマテリアライズドビューに使用される行をフィルタリングするWHERE句をサポートしています。

- GROUP BY（オプション）

  クエリのGROUP BY列です。このパラメータが指定されていない場合、デフォルトではデータはグループ化されません。

- ORDER BY（オプション）

  クエリのORDER BY列です。

  - ORDER BY句の列は、`select_expr`の列と同じ順序で宣言される必要があります。
  - クエリステートメントにGROUP BY句が含まれている場合、ORDER BY列はGROUP BY列と同一でなければなりません。
  - このパラメータが指定されていない場合、システムは以下のルールに従ってORDER BY列を自動的に補完します。
    - マテリアライズドビューがAGGREGATEタイプの場合、すべてのGROUP BY列が自動的にソートキーとして使用されます。
    - マテリアライズドビューがAGGREGATEタイプでない場合、StarRocksはプレフィックス列に基づいてソートキーを自動的に選択します。

### 同期マテリアライズドビューのクエリ

同期マテリアライズドビューは基本的に物理テーブルではなくベーステーブルのインデックスであるため、ヒント`[_SYNC_MV_]`を使用して同期マテリアライズドビューをクエリする必要があります。

```SQL
-- ヒントの角括弧[]を省略しないでください。
SELECT * FROM <mv_name> [_SYNC_MV_];
```

> **注意**
>
> 現在、StarRocksは同期マテリアライズドビューの列にエイリアスを指定しても、列の名前を自動的に生成します。

### 同期マテリアライズドビューによる自動クエリ改写

同期マテリアライズドビューのパターンに従ったクエリが実行されると、元のクエリ文が自動的に書き換えられ、マテリアライズドビューに格納された中間結果が使用されます。

次の表は、元のクエリの集計関数と、マテリアライズドビューの作成に使用された集計関数の対応関係を示しています。ビジネスシナリオに応じて対応する集計関数を選択し、マテリアライズドビューを構築できます。

| **元のクエリの集計関数**           | **マテリアライズドビューの集計関数** |
| ------------------------------------------------------ | ----------------------------------------------- |
| sum                                                    | sum                                             |
| min                                                    | min                                             |
| max                                                    | max                                             |
| count                                                  | count                                           |
| bitmap_union, bitmap_union_count, count(distinct)      | bitmap_union                                    |
| hll_raw_agg, hll_union_agg, ndv, approx_count_distinct | hll_union                                       |
| percentile_approx, percentile_union                    | percentile_union                                |

## 非同期マテリアライズドビュー

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

角括弧 [] 内のパラメーターはオプショナルです。

### パラメーター

**mv_name** (必須)

マテリアライズドビューの名前。命名要件は以下の通りです。

- 名前は、文字 (a-z または A-Z)、数字 (0-9)、またはアンダースコア (_) で構成され、文字で始まる必要があります。
- 名前の長さは64文字を超えてはなりません。
- 名前は大文字と小文字を区別します。

> **注意**
>
> 同じベーステーブルに複数のマテリアライズドビューを作成できますが、同じデータベース内のマテリアライズドビューの名前が重複することはできません。

**COMMENT** (オプショナル)

マテリアライズドビューにコメントを追加します。`COMMENT` は `mv_name` の後に配置する必要があります。そうでないと、マテリアライズドビューを作成できません。

**distribution_desc** (オプショナル)

非同期マテリアライズドビューのバケット化戦略。StarRocksはハッシュバケットとランダムバケットをサポートしています（v3.1以降）。このパラメータを指定しない場合、StarRocksはランダムバケット戦略を使用し、バケット数を自動的に設定します。

> **ノート**
>
> 非同期マテリアライズドビューを作成する際は、`distribution_desc` または `refresh_scheme` のいずれか、または両方を指定する必要があります。

- **ハッシュバケット**:

  構文

  ```SQL
  DISTRIBUTED BY HASH(<bucket_key1>[,<bucket_key2> ...]) [BUCKETS <bucket_number>]
  ```

  詳細については、[データ分散](../../../table_design/Data_distribution.md#data-distribution)を参照してください。

  > **ノート**
  >
  > v2.5.7以降、StarRocksはテーブル作成時やパーティション追加時にバケット数（BUCKETS）を自動的に設定できます。バケット数を手動で設定する必要はありません。詳細については、[バケット数の決定](../../../table_design/Data_distribution.md#set-the-number-of-buckets)を参照してください。

- **ランダムバケット**:

  ランダムバケット戦略を選択し、StarRocksにバケット数を自動的に設定させる場合、`distribution_desc` を指定する必要はありません。しかし、バケット数を手動で設定したい場合は、以下の構文を参照してください：

  ```SQL
  DISTRIBUTED BY RANDOM BUCKETS <bucket_number>
  ```

  > **注意**
  >
  > ランダムバケット戦略を使用する非同期マテリアライズドビューは、コロケーショングループに割り当てることはできません。

  詳細については、[ランダムバケット](../../../table_design/Data_distribution.md#random-bucketing-since-v31)を参照してください。

**refresh_moment** (オプショナル)

マテリアライズドビューのリフレッシュタイミング。デフォルト値: `IMMEDIATE`。有効な値は以下の通りです：

- `IMMEDIATE`: 非同期マテリアライズドビューは、作成直後にリフレッシュされます。
- `DEFERRED`: 非同期マテリアライズドビューは、作成後にリフレッシュされません。マテリアライズドビューを手動でリフレッシュするか、定期的なリフレッシュタスクを設定することができます。

**refresh_scheme** (オプショナル)

> **ノート**
>
> 非同期マテリアライズドビューを作成する際は、`distribution_desc` または `refresh_scheme` のいずれか、または両方を指定する必要があります。

非同期マテリアライズドビューのリフレッシュ戦略。有効な値は以下の通りです：

- `ASYNC`: 非同期リフレッシュモード。ベーステーブルのデータが変更されるたびに、マテリアライズドビューは事前に定義されたリフレッシュ間隔に従って自動的にリフレッシュされます。更新開始時刻を `START('yyyy-MM-dd hh:mm:ss')` として指定し、リフレッシュ間隔を `EVERY (INTERVAL n day/hour/minute/second)` で指定できます。単位は `DAY`、`HOUR`、`MINUTE`、`SECOND` です。例：`ASYNC START ('2023-09-12 16:30:25') EVERY (INTERVAL 5 MINUTE)`。間隔を指定しない場合、デフォルト値 `10 MINUTE` が使用されます。
- `MANUAL`: 手動リフレッシュモード。マテリアライズドビューは自動的にリフレッシュされません。リフレッシュタスクはユーザーによって手動でのみトリガーされます。

このパラメータを指定しない場合、デフォルト値 `MANUAL` が使用されます。

**partition_expression** (オプショナル)

非同期マテリアライズドビューのパーティショニング戦略。StarRocksの現在のバージョンでは、非同期マテリアライズドビューの作成時に1つのパーティション式のみがサポートされています。

> **注意**
>
> 現在、非同期マテリアライズドビューはリストパーティショニング戦略をサポートしていません。

有効な値は以下の通りです：

- `column_name`: パーティショニングに使用される列の名前。`PARTITION BY dt` という式は、`dt` 列に基づいてマテリアライズドビューをパーティショニングすることを意味します。
- `date_trunc` 関数: 時間単位を切り捨てるために使用される関数。`PARTITION BY date_trunc("MONTH", dt)` は、`dt` 列が月単位で切り捨てられてパーティショニングされることを意味します。`date_trunc` 関数は `YEAR`、`MONTH`、`DAY`、`HOUR`、`MINUTE` などの時間単位を切り捨てることをサポートしています。
- `str2date` 関数: ベーステーブルの文字列型パーティションをマテリアライズドビューのパーティションに分割するために使用される関数。`PARTITION BY str2date(dt, "%Y%m%d")` は、`dt` 列が日付形式 `"%Y%m%d"` の文字列型日付であることを意味します。`str2date` 関数は多くの日付形式をサポートしており、詳細については [str2date](../../sql-functions/date-time-functions/str2date.md) を参照してください。v3.1.4 からサポートされています。
- `time_slice` または `date_slice` 関数: v3.1 以降、これらの関数を使用して、指定された時間を、指定された時間粒度に基づいた時間間隔の開始または終了に変換できます。例えば、`PARTITION BY date_trunc("MONTH", time_slice(dt, INTERVAL 7 DAY))` では、`time_slice` と `date_slice` は `date_trunc` よりも細かい粒度である必要があります。これらを使用して、パーティションキーよりも細かい粒度の GROUP BY カラムを指定することができます。例: `GROUP BY time_slice(dt, INTERVAL 1 MINUTE) PARTITION BY date_trunc('DAY', ts)`。

このパラメータが指定されない場合、デフォルトではパーティション戦略は採用されません。

**order_by_expression** (オプション)

非同期マテリアライズドビューのソートキー。ソートキーを指定しない場合、StarRocks は SELECT カラムのプレフィックスカラムの一部をソートキーとして選択します。例えば、`select a, b, c, d` では、ソートキーは `a` と `b` になります。このパラメータは StarRocks v3.0 以降でサポートされています。

**PROPERTIES** (オプション)

非同期マテリアライズドビューのプロパティ。既存のマテリアライズドビューのプロパティは、[ALTER MATERIALIZED VIEW](./ALTER_MATERIALIZED_VIEW.md) を使用して変更できます。

- `session.`: マテリアライズドビューのセッション変数関連プロパティを変更する場合、プロパティに `session.` 接頭辞を追加する必要があります。例: `session.query_timeout`。非セッションプロパティには接頭辞を指定する必要はありません。例: `mv_rewrite_staleness_second`。
- `replication_num`: 作成するマテリアライズドビューレプリカの数。
- `storage_medium`: ストレージ媒体のタイプ。有効な値: `HDD` と `SSD`。
- `storage_cooldown_time`: パーティションのストレージ冷却時間。HDD と SSD の両方のストレージ媒体を使用する場合、SSD ストレージ内のデータは、このプロパティで指定された時間が経過した後に HDD ストレージに移動されます。形式: "yyyy-MM-dd HH:mm:ss"。指定された時間は現在時刻より後でなければなりません。このプロパティが明示的に指定されていない場合、デフォルトではストレージ冷却は行われません。
- `partition_ttl`: パーティションの存続時間 (TTL)。指定された時間範囲内のデータを持つパーティションが保持されます。期限切れのパーティションは自動的に削除されます。単位: `YEAR`, `MONTH`, `DAY`, `HOUR`, `MINUTE`。例えば、このプロパティは `2 MONTH` として指定できます。このプロパティは `partition_ttl_number` より推奨されます。v3.1.5 以降でサポートされています。
- `partition_ttl_number`: 最新のマテリアライズドビューパーティションを保持する数。開始時刻が現在時刻より前のパーティションについて、これらのパーティションの数がこの値を超えた場合、古いパーティションが削除されます。StarRocks は FE 設定項目 `dynamic_partition_check_interval_seconds` で指定された時間間隔に従ってマテリアライズドビューパーティションを定期的にチェックし、期限切れのパーティションを自動的に削除します。[動的パーティション](../../../table_design/dynamic_partitioning.md) 戦略を有効にしている場合、事前に作成されたパーティションは含まれません。値が `-1` の場合、マテリアライズドビューのすべてのパーティションが保持されます。デフォルトは `-1`。
- `partition_refresh_number`: 単一のリフレッシュで更新されるパーティションの最大数。更新されるパーティションの数がこの値を超える場合、StarRocks はリフレッシュタスクを分割してバッチ処理で完了します。前のバッチのパーティションが成功裏にリフレッシュされた後にのみ、StarRocks は次のバッチのパーティションのリフレッシュを続け、すべてのパーティションがリフレッシュされるまで続けます。いずれかのパーティションのリフレッシュに失敗した場合、後続のリフレッシュタスクは生成されません。値が `-1` の場合、リフレッシュタスクは分割されません。デフォルトは `-1`。
- `excluded_trigger_tables`: マテリアライズドビューのベーステーブルがここにリストされている場合、ベーステーブルのデータが変更されたとしても自動リフレッシュタスクはトリガーされません。このパラメータは、ロードトリガー型のリフレッシュ戦略にのみ適用され、通常 `auto_refresh_partitions_limit` プロパティと共に使用されます。形式: `[db_name.]table_name`。値が空文字列の場合、すべてのベーステーブルのデータ変更がマテリアライズドビューのリフレッシュをトリガーします。デフォルト値は空文字列です。
- `auto_refresh_partitions_limit`: マテリアライズドビューのリフレッシュがトリガーされた際にリフレッシュする必要のある最新のマテリアライズドビューパーティションの数。このプロパティを使用してリフレッシュ範囲を制限し、リフレッシュコストを削減することができます。ただし、すべてのパーティションがリフレッシュされるわけではないため、マテリアライズドビューのデータがベーステーブルと一致しない可能性があります。デフォルトは `-1`。値が `-1` の場合、すべてのパーティションがリフレッシュされます。値が正の整数 N の場合、StarRocks は既存のパーティションを時系列順にソートし、現在のパーティションと N-1 の最新パーティションをリフレッシュします。パーティションの数が N 未満の場合、StarRocks は既存のすべてのパーティションをリフレッシュします。マテリアライズドビューに事前に作成された動的パーティションがある場合、StarRocks は事前に作成されたすべてのパーティションをリフレッシュします。
- `mv_rewrite_staleness_second`: マテリアライズドビューの最終リフレッシュがこのプロパティで指定された時間間隔内である場合、ベーステーブルのデータが変更されているかどうかに関わらず、このマテリアライズドビューはクエリ書き換えに直接使用できます。最終リフレッシュがこの時間間隔より前である場合、StarRocks はベーステーブルが更新されているかどうかをチェックし、マテリアライズドビューがクエリ書き換えに使用できるかどうかを判断します。単位は秒です。このプロパティは v3.0 からサポートされています。
- `colocate_with`: 非同期マテリアライズドビューのコロケーショングループ。詳細は [Colocate Join](../../../using_starrocks/Colocate_join.md) を参照してください。このプロパティは v3.0 からサポートされています。
- `unique_constraints` および `foreign_key_constraints`: ビューのデルタ結合シナリオでクエリ書き換えのために非同期マテリアライズドビューを作成する際のユニークキー制約と外部キー制約。詳細は [非同期マテリアライズドビュー - View Delta Join シナリオでのクエリ書き換え](../../../using_starrocks/query_rewrite_with_materialized_views.md) を参照してください。このプロパティは v3.0 からサポートされています。
- `resource_group`: マテリアライズドビューのリフレッシュタスクが属するリソースグループ。リソースグループの詳細については [リソースグループ](../../../administration/resource_group.md) を参照してください。
- `query_rewrite_consistency`: 非同期マテリアライズドビューのクエリ書き換えルール。このプロパティは v3.2 からサポートされています。有効な値は以下の通りです:
  - `disable`: 非同期マテリアライズドビューの自動クエリ書き換えを無効にします。
  - `checked` (デフォルト値): マテリアライズドビューがタイムリネス要件を満たしている場合のみ、自動クエリ書き換えを有効にします。

    - `mv_rewrite_staleness_second` が指定されていない場合、マテリアライズドビューは、そのデータがすべてのベーステーブルのデータと一致している場合にのみ、クエリ書き換えに使用できます。
    - `mv_rewrite_staleness_second` が指定されている場合、マテリアライズドビューは、最後のリフレッシュからの経過時間が許容される古さの時間間隔内であれば、クエリ書き換えに使用できます。
  - `loose`: 自動クエリ書き換えを直接有効にし、整合性チェックは不要です。
- `force_external_table_query_rewrite`: 外部カタログベースのマテリアライズドビューに対するクエリ書き換えを有効にするかどうか。このプロパティはv3.2からサポートされています。有効な値:
  - `true`: 外部カタログベースのマテリアライズドビューに対するクエリ書き換えを有効にします。
  - `false`（デフォルト値）: 外部カタログベースのマテリアライズドビューに対するクエリ書き換えを無効にします。

  ベーステーブルと外部カタログベースのマテリアライズドビュー間では強いデータ整合性が保証されていないため、この機能はデフォルトで`false`に設定されています。この機能が有効化された場合、マテリアライズドビューは`query_rewrite_consistency`で指定されたルールに従ってクエリ書き換えに使用されます。

> **注意**
>
> ユニークキー制約と外部キー制約はクエリ書き換えにのみ使用されます。データがテーブルにロードされる際の外部キー制約のチェックは保証されません。テーブルにロードされるデータが制約を満たしていることを確認する必要があります。

**query_statement**（必須）

非同期マテリアライズドビューを作成するためのクエリステートメントです。v3.1.6以降、StarRocksは共通テーブル式（CTE）を用いた非同期マテリアライズドビューの作成をサポートしています。

> **注意**
>
> 現在、StarRocksはリストパーティショニング戦略で作成されたベーステーブルを用いた非同期マテリアライズドビューの作成をサポートしていません。

### 非同期マテリアライズドビューのクエリ

非同期マテリアライズドビューは物理テーブルです。通常のテーブルと同様に操作できますが、**非同期マテリアライズドビューに直接データをロードすることはできません**。

### 非同期マテリアライズドビューを用いた自動クエリ書き換え

StarRocks v2.5は、SPJG型の非同期マテリアライズドビューを基にした自動かつ透過的なクエリ書き換えをサポートしています。SPJG型マテリアライズドビューとは、プランがスキャン、フィルタ、プロジェクト、アグリゲートの各種オペレーターのみを含むマテリアライズドビューを指します。SPJG型マテリアライズドビューのクエリ書き換えには、単一テーブルクエリの書き換え、ジョインクエリの書き換え、集約クエリの書き換え、ユニオンクエリの書き換え、およびネストされたマテリアライズドビューに基づくクエリの書き換えが含まれます。

詳細は[非同期マテリアライズドビュー - 非同期マテリアライズドビューによるクエリの書き換え](../../../using_starrocks/query_rewrite_with_materialized_views.md)をご覧ください。

### サポートされるデータ型

- StarRocksのデフォルトカタログに基づいて作成された非同期マテリアライズドビューは、以下のデータ型をサポートしています：

  - **日付**: DATE, DATETIME
  - **文字列**: CHAR, VARCHAR
  - **数値**: BOOLEAN, TINYINT, SMALLINT, INT, BIGINT, LARGEINT, FLOAT, DOUBLE, DECIMAL, PERCENTILE
  - **半構造化**: ARRAY, JSON, MAP（v3.1以降）、STRUCT（v3.1以降）
  - **その他**: BITMAP, HLL

> **注記**
>
> BITMAP、HLL、およびPERCENTILEはv2.4.5以降でサポートされています。

- StarRocksの外部カタログに基づいて作成された非同期マテリアライズドビューは、以下のデータ型をサポートしています：

  - Hiveカタログ

    - **数値**: INT/INTEGER, BIGINT, DOUBLE, FLOAT, DECIMAL
    - **日付**: TIMESTAMP
    - **文字列**: STRING, VARCHAR, CHAR
    - **半構造化**: ARRAY

  - Hudiカタログ

    - **数値**: BOOLEAN, INT, LONG, FLOAT, DOUBLE, DECIMAL
    - **日付**: DATE, TimeMillis/TimeMicros, TimestampMillis/TimestampMicros
    - **文字列**: STRING
    - **半構造化**: ARRAY

  - Icebergカタログ

    - **数値**: BOOLEAN, INT, LONG, FLOAT, DOUBLE, DECIMAL(P, S)
    - **日付**: DATE, TIME, TIMESTAMP
    - **文字列**: STRING, UUID, FIXED(L), BINARY
    - **半構造化**: LIST

## 使用上の注意点

- StarRocksの現行バージョンでは、複数のマテリアライズドビューを同時に作成することはできません。新しいマテリアライズドビューは、前のものが完了した後にのみ作成可能です。

- 同期マテリアライズドビューについて：

  - 同期マテリアライズドビューは、単一の列に対する集約関数のみをサポートしています。`sum(a+b)`の形式のクエリステートメントはサポートされていません。
  - 同期マテリアライズドビューは、ベーステーブルの各列に対して1つの集約関数のみをサポートしています。`select sum(a), min(a) from table`のようなクエリステートメントはサポートされていません。
  - 集約関数を用いた同期マテリアライズドビューを作成する際には、GROUP BY句を指定し、SELECT文に少なくとも1つのGROUP BYカラムを含める必要があります。
  - 同期マテリアライズドビューは、JOIN句やGROUP BYのHAVING句をサポートしていません。
  - ALTER TABLE DROP COLUMNを使用してベーステーブルの特定のカラムを削除する際には、そのカラムを含むすべての同期マテリアライズドビューが存在しないことを確認する必要があります。そうでなければ、削除操作は失敗します。カラムを削除する前に、そのカラムを含むすべての同期マテリアライズドビューを削除する必要があります。
  - テーブルに多くの同期マテリアライズドビューを作成すると、データのロード効率に影響を与える可能性があります。データがベーステーブルにロードされると、同期マテリアライズドビューとベーステーブルのデータは同時に更新されます。ベーステーブルに`n`個の同期マテリアライズドビューがある場合、ベーステーブルへのデータロードの効率は、`n`個のテーブルへのデータロードの効率とほぼ同じです。

- ネストされた非同期マテリアライズドビューについて：

  - 各マテリアライズドビューのリフレッシュ戦略は、それぞれのマテリアライズドビューにのみ適用されます。
  - 現在、StarRocksはネストレベルの数に制限を設けていませんが、運用環境ではネストレイヤーの数を3層を超えないことを推奨します。

- 外部カタログに基づく非同期マテリアライズドビューについて：

  - 外部カタログに基づくマテリアライズドビューは、非同期の固定間隔リフレッシュと手動リフレッシュのみをサポートしています。
  - マテリアライズドビューと外部カタログのベーステーブルとの間の厳密なデータ整合性は保証されていません。
  - 現在、外部リソースに基づいてマテリアライズドビューを構築することはサポートされていません。

  - 現在、StarRocksは外部カタログ内のベーステーブルデータが変更されたかどうかを検知できないため、ベーステーブルがリフレッシュされるたびにデフォルトで全てのパーティションがリフレッシュされます。[REFRESH MATERIALIZED VIEW](../data-manipulation/REFRESH_MATERIALIZED_VIEW.md)を使用して、特定のパーティションのみを手動でリフレッシュすることができます。

## 例

### 同期マテリアライズドビューの例

ベーステーブルのスキーマは以下の通りです：

```Plain Text
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

例1：元のテーブル（k1、k2）のカラムのみを含む同期マテリアライズドビューを作成します。

```sql
create materialized view k1_k2 as
select k1, k2 from duplicate_table;
```

このマテリアライズドビューには集約なしでカラムk1とk2のみが含まれます。

```plain text
+-----------------+-------+--------+------+------+---------+-------+
| IndexName       | Field | Type   | Null | Key  | Default | Extra |
+-----------------+-------+--------+------+------+---------+-------+
| k1_k2           | k1    | INT    | Yes  | true | N/A     |       |
|                 | k2    | INT    | Yes  | true | N/A     |       |
+-----------------+-------+--------+------+------+---------+-------+
```

例2：k2でソートされた同期マテリアライズドビューを作成します。

```sql
create materialized view k2_order as
select k2, k1 from duplicate_table order by k2;
```

マテリアライズドビューのスキーマは以下の通りです。このマテリアライズドビューにはカラムk2とk1のみが含まれ、カラムk2は集約なしのソートカラムです。

```plain text
+-----------------+-------+--------+------+-------+---------+-------+
| IndexName       | Field | Type   | Null | Key   | Default | Extra |
+-----------------+-------+--------+------+-------+---------+-------+
| k2_order        | k2    | INT    | Yes  | true  | N/A     |       |
|                 | k1    | INT    | Yes  | false | N/A     | NONE  |
+-----------------+-------+--------+------+-------+---------+-------+
```

例3：k1とk2でグループ化し、k3にSUM集計を行う同期マテリアライズドビューを作成します。

```sql
create materialized view k1_k2_sumk3 as
select k1, k2, sum(k3) from duplicate_table group by k1, k2;
```

マテリアライズドビューのスキーマは以下の通りです。このマテリアライズドビューにはカラムk1、k2、およびsum(k3)の3つのカラムが含まれ、k1とk2はグループ化されたカラムで、sum(k3)はk1とk2に基づいてグループ化されたk3カラムの合計です。

```plain text
+-----------------+-------+--------+------+-------+---------+-------+
| IndexName       | Field | Type   | Null | Key   | Default | Extra |
+-----------------+-------+--------+------+-------+---------+-------+
| k1_k2_sumk3     | k1    | INT    | Yes  | true  | N/A     |       |
|                 | k2    | INT    | Yes  | true  | N/A     |       |
|                 | k3    | BIGINT | Yes  | false | N/A     | SUM   |
+-----------------+-------+--------+------+-------+---------+-------+
```

マテリアライズドビューはソートカラムを宣言しておらず、集約関数を採用しているため、StarRocksはデフォルトでグループ化されたカラムk1とk2を補完します。

例4：重複する行を削除する同期マテリアライズドビューを作成します。

```sql
create materialized view deduplicate as
select k1, k2, k3, k4 from duplicate_table group by k1, k2, k3, k4;
```

マテリアライズドビューのスキーマは以下の通りです。このマテリアライズドビューにはカラムk1、k2、k3、およびk4が含まれ、重複する行はありません。

```plain text
+-----------------+-------+--------+------+-------+---------+-------+
| IndexName       | Field | Type   | Null | Key   | Default | Extra |
+-----------------+-------+--------+------+-------+---------+-------+
| deduplicate     | k1    | INT    | Yes  | true  | N/A     |       |
|                 | k2    | INT    | Yes  | true  | N/A     |       |
|                 | k3    | BIGINT | Yes  | true  | N/A     |       |
|                 | k4    | BIGINT | Yes  | true  | N/A     |       |
+-----------------+-------+--------+------+-------+---------+-------+
```

例5：ソートカラムを宣言しない非集約型の同期マテリアライズドビューを作成します。

ベーステーブルのスキーマは以下の通りです：

```plain text
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

マテリアライズドビューにはカラムk3、k4、k5、k6、およびk7が含まれ、ソートカラムは宣言されていません。以下のステートメントでマテリアライズドビューを作成します：

```sql
create materialized view mv_1 as
select k3, k4, k5, k6, k7 from all_type_table;
```

StarRocksはデフォルトでk3、k4、k5をソートカラムとして自動的に使用します。これら3つのカラムタイプが占めるバイト数の合計は4 (INT) + 8 (BIGINT) + 16 (DECIMAL) = 28 < 36です。したがって、これら3つのカラムはソートカラムとして追加されます。

マテリアライズドビューのスキーマは以下の通りです。

```plain text
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

k3、k4、およびk5カラムの`key`フィールドが`true`であることから、これらがソートキーであることがわかります。k6およびk7カラムの`key`フィールドは`false`であり、これらがソートキーではないことを示しています。

例6：WHERE句と複雑な式を含む同期マテリアライズドビューを作成します。

```SQL
-- ベーステーブルを作成する：user_event
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
  DISTRIBUTED BY hash(id);

  -- WHERE句と複雑な式を含むマテリアライズドビューを作成します。
  CREATE MATERIALIZED VIEW test_mv1
  AS 
  SELECT
  ds,
  column_19,
  column_36,
  sum(column_01) as column_01_sum,
  bitmap_union(to_bitmap(user_id)) as user_id_dist_cnt,
  bitmap_union(to_bitmap(case when column_01 > 1 and column_34 IN ('1','34') then user_id2 else null end)) as filter_dist_cnt_1,
  bitmap_union(to_bitmap(case when column_02 > 60 and column_35 IN ('11','13') then user_id2 else null end)) as filter_dist_cnt_2,
  bitmap_union(to_bitmap(case when column_03 > 70 and column_36 IN ('21','23') then user_id2 else null end)) as filter_dist_cnt_3,
  bitmap_union(to_bitmap(case when column_04 > 20 and column_27 IN ('31','27') then user_id2 else null end)) as filter_dist_cnt_4,
  bitmap_union(to_bitmap(case when column_05 > 90 and column_28 IN ('41','43') then user_id2 else null end)) as filter_dist_cnt_5
  FROM user_event
  WHERE ds >= '2023-11-02'
  GROUP BY
  ds,
  column_19,
  column_36;
 ```

### 非同期マテリアライズドビューの例

以下の例は、下記のベーステーブルに基づいています。

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

ordersテーブルを作成します (
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

例 1: パーティション化されていないマテリアライズドビューを作成します。

```SQL
CREATE MATERIALIZED VIEW lo_mv1
DISTRIBUTED BY HASH(`lo_orderkey`)
REFRESH ASYNC
AS
SELECT
    lo_orderkey, 
    lo_custkey, 
    sum(lo_quantity) as total_quantity, 
    sum(lo_revenue) as total_revenue, 
    count(lo_shipmode) as shipmode_count
FROM lineorder 
GROUP BY lo_orderkey, lo_custkey 
ORDER BY lo_orderkey;
```

例 2: パーティション化されたマテリアライズドビューを作成します。

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
    sum(lo_quantity) as total_quantity, 
    sum(lo_revenue) as total_revenue, 
    count(lo_shipmode) as shipmode_count
FROM lineorder 
GROUP BY lo_orderkey, lo_orderdate, lo_custkey
ORDER BY lo_orderkey;

-- date_trunc()関数を使用して、マテリアライズドビューを月単位でパーティションします。
CREATE MATERIALIZED VIEW order_mv1
PARTITION BY date_trunc('month', `dt`)
DISTRIBUTED BY HASH(`order_id`)
REFRESH ASYNC START('2023-07-01 10:00:00') EVERY (interval 1 day)
AS
SELECT
    dt,
    order_id,
    user_id,
    sum(cnt) as total_cnt,
    sum(revenue) as total_revenue, 
    count(state) as state_count
FROM orders
GROUP BY dt, order_id, user_id;
```

例 3: 非同期マテリアライズドビューを作成します。

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

例 4: パーティション化されたマテリアライズドビューを作成し、`str2date`を使用して、ベーステーブルのSTRING型のパーティションキーをマテリアライズドビューの日付型に変換します。

``` SQL

-- STRING型のパーティション列を持つHiveテーブル。
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


-- `str2date`を使用してマテリアライズドビューを作成します。
CREATE MATERIALIZED VIEW IF NOT EXISTS `test_mv` 
PARTITION BY str2date(`d_datekey`, '%Y%m%d')
DISTRIBUTED BY HASH(`d_date`, `d_month`, `d_year`) 
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