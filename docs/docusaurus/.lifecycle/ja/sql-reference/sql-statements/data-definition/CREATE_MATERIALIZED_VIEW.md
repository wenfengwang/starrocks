---
displayed_sidebar: "English"
---

# マテリアライズド・ビューの作成

## 概要

マテリアライズド・ビューを作成します。マテリアライズド・ビューの使用方法については、[同期マテリアライズド・ビュー](../../../using_starrocks/Materialized_view-single_table.md) および [非同期マテリアライズド・ビュー](../../../using_starrocks/Materialized_view.md) を参照してください。

> **注意**
>
> ベーステーブルが存在するデータベースで CREATE MATERIALIZED VIEW 権限を持つユーザーのみがマテリアライズド・ビューを作成できます。

マテリアライズド・ビューの作成は非同期操作です。このコマンドを正常に実行すると、マテリアライズド・ビューの作成タスクが正常に送信されたことを示します。同期マテリアライズド・ビューのビルドステータスは、[SHOW ALTER MATERIALIZED VIEW](../data-manipulation/SHOW_ALTER_MATERIALIZED_VIEW.md) コマンドを使用してデータベース内で表示し、非同期マテリアライズド・ビューのビルドステータスは[Information Schema](../../../reference/overview-pages/information_schema.md) のメタデータビュー [`tasks`](../../../reference/information_schema/tasks.md) および [`task_runs`](../../../reference/information_schema/task_runs.md) をクエリして表示できます。

StarRocks は v2.4 から非同期マテリアライズド・ビューをサポートしています。それ以前のバージョンの同期マテリアライズド・ビューと非同期マテリアライズド・ビューの主な違いは次のとおりです。

|                       | **単一テーブルの集計** | **複数テーブルの結合** | **クエリの書き換え** | **リフレッシュ戦略** | **ベーステーブル** |
| --------------------- | ---------------------------- | -------------------- | ----------------- | -------------------- | -------------- |
| **非同期 MV** | はい | はい | はい | <ul><li>非同期リフレッシュ</li><li>手動リフレッシュ</li></ul> | デフォルトカタログからの複数のテーブル:<ul><li>外部カタログ（v2.5）</li><li>既存のマテリアライズド・ビュー（v2.5）</li><li>既存のビュー（v3.1）</li></ul> |
| **同期 MV (Rollup)**  | 集約関数の選択肢が限られている | いいえ | はい | データのロード中に同期リフレッシュ | デフォルトカタログの単一テーブル |

## 同期マテリアライズド・ビュー

### 構文

```SQL
CREATE MATERIALIZED VIEW [IF NOT EXISTS] [database.]<mv_name>
[COMMENT ""]
[PROPERTIES ("key"="value", ...)]
AS 
<query_statement>
```

角括弧[]内のパラメーターは省略可能です。

### パラメーター

**mv_name** (必須)

マテリアライズド・ビューの名前。命名の要件は次のとおりです：

- 名前は、文字（a-z または A-Z）、数字（0-9）、またはアンダースコア(\_)で構成されている必要があり、先頭文字は文字である必要があります。
- 名前の長さは 64 文字を超えることはできません。
- 名前は大文字と小文字が区別されます。

**COMMENT** (任意)

マテリアライズド・ビューに関するコメント。`COMMENT` は `mv_name` の後に配置する必要があります。それ以外の場合、マテリアライズド・ビューは作成できません。

**query_statement** (必須)

マテリアライズド・ビューを作成するためのクエリ文。その結果はマテリアライズド・ビュー内のデータです。構文は次のとおりです：

```SQL
SELECT select_expr[, select_expr ...]
[WHERE where_expr]
[GROUP BY column_name[, column_name ...]]
[ORDER BY column_name[, column_name ...]]
```

- select_expr (必須)

  クエリ文内のすべての列、すなわちマテリアライズド・ビューのスキーマ内のすべての列。このパラメーターは次の値をサポートしています：

  - 「SELECT a, abs(b), min(c) FROM table_a」といった単純な列または集計列。ここで `a`、`b`、`c` はベーステーブル内の列の名前です。マテリアライズド・ビューの列名を指定しない場合、StarRocks は自動的に列に名前を割り当てます。
  - 「SELECT a+1 AS x, b+2 AS y, c*c AS z FROM table_a」といった式。ここで `a+1`、`b+2`、`c*c` はベーステーブル内の列を参照した式であり、`x`、`y`、`z` はマテリアライズド・ビューの列に割り当てられた別名です。

  > **注意**
  >
  > - `select_expr` には少なくとも 1 つの列を指定する必要があります。
  > - 集約関数を使用して同期マテリアライズド・ビューを作成する場合、`GROUP BY` 句を指定する必要があり、`select_expr` 内で少なくとも 1 つの `GROUP BY` 列を指定する必要があります。
  > - 同期マテリアライズド・ビューでは、JOIN および GROUP BY の HAVING 句などの句はサポートされていません。
  > - v3.1 以降、各同期マテリアライズド・ビューは、例えば `select b, sum(a), min(a) from table group by b` などのようなベーステーブルの各列に複数の集約関数をサポートできます。
  > - v3.1 以降、同期マテリアライズド・ビューは SELECT および集約関数に対して複雑な式をサポートし、例えば `select b, sum(a + 1) as sum_a1, min(cast (a as bigint)) as min_a from table group by b` や `select abs(b) as col1, a + 1 as col2, cast(a as bigint) as col3 from table` などのクエリ文をサポートします。同期マテリアライズド・ビューで使用される複雑な式には以下の制限があります：
  >   - 各複雑な式にはエイリアスが必要であり、同じベーステーブルのすべての同期マテリアライズド・ビューで異なるエイリアスを複雑な式に割り当てる必要があります。例えば、`select b, sum(a + 1) as sum_a from table group by b` と `select b, sum(a) as sum_a from table group by b` というクエリ文は同じベーステーブルの同期マテリアライズド・ビューを作成するために使用できません。複雑な式に対して異なるエイリアスを設定できます。
  >   - 複雑な式で作成された同期マテリアライズド・ビューがクエリによって書き換えられているかどうかは、`EXPLAIN <sql_statement>` を実行して確認できます。詳細については、[クエリ解析](../../../administration/Query_planning.md) を参照してください。

- WHERE (任意)

  v3.2 以降、同期マテリアライズド・ビューはマテリアライズド・ビューに使用される行をフィルタリングできる `WHERE` 句をサポートします。

- GROUP BY (任意)

  クエリの GROUP BY 列。このパラメーターが指定されていない場合、データはデフォルトでグループ化されません。

- ORDER BY (任意)

  クエリの ORDER BY 列です。

  - ORDER BY 句の列は、`select_expr` 内の列の順序と同じ順序で宣言されている必要があります。
  - クエリ文に GROUP BY 句が含まれている場合、ORDER BY 列は GROUP BY 列と同じである必要があります。
  - このパラメーターが指定されていない場合、システムは次の規則に従って自動的に ORDER BY 列を補完します：
    - マテリアライズド・ビューが AGGREGATE タイプである場合、すべての GROUP BY 列をソートキーとして自動選択します。
    - マテリアライズド・ビューが AGGREGATE タイプでない場合、StarRocks はプレフィックス列に基づいてソートキーを自動的に選択します。

### 同期マテリアライズド・ビューのクエリ

同期マテリアライズド・ビューは、物理テーブルではなくベーステーブルのインデックスですので、同期マテリアライズド・ビューをクエリする際にはヒント `[_SYNC_MV_]` を使用するだけです：

```SQL
-- ヒント内の角括弧 [] を省略しないでください。
SELECT * FROM <mv_name> [_SYNC_MV_];
```

- 名前は、文字（a-zまたはA-Z）、数字（0-9）、またはアンダースコア（\_）で構成する必要があり、文字で始める必要があります。
- 名前の長さは64文字を超えてはいけません。
- 名前は大文字と小文字を区別します。

> **注意**
>
> 同じ基本テーブルに複数のMATERIALIZED VIEWを作成できますが、同じデータベース内のMATERIALIZED VIEWの名前は重複してはいけません。

**COMMENT**（オプション）

MATERIALIZED VIEWに関するコメント。`COMMENT`は`mv_name`の後に配置する必要があります。そうでないと、MATERIALIZED VIEWを作成できません。

**distribution_desc**（オプション）

非同期MATERIALIZED VIEWのバケット化戦略。StarRocksはハッシュバケッティングとランダムバケッティングをサポートしています（v3.1以降）。このパラメータを指定しない場合、StarRocksはランダムバケッティング戦略を使用し、バケットの数を自動的に設定します。

> **注意**
>
> 非同期MATERIALIZED VIEWを作成する際には、`distribution_desc`または`refresh_scheme`、またはその両方を指定する必要があります。

- **ハッシュバケッティング**：

  構文

  ```SQL
  DISTRIBUTED BY HASH (<bucket_key1>[,<bucket_key2> ...]) [BUCKETS <bucket_number>]
  ```

  詳細については、[データ分散](../../../table_design/Data_distribution.md#data-distribution)を参照してください。

  > **注意**
  >
  > v2.5.7以降、StarRocksではテーブルを作成するかパーティションを追加する際に、バケットの数（BUCKETS）を自動的に設定することができます。バケットの数を手動で設定する必要はもうありません。詳細については、[バケットの数の決定](../../../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

- **ランダムバケッティング**：

  ランダムバケッティング戦略を選択し、StarRocksに自動的にバケットの数を設定させる場合、`distribution_desc`を指定する必要はありません。ただし、バケットの数を手動で設定したい場合は、次の構文を参照できます。

  ```SQL
  DISTRIBUTED BY RANDOM BUCKETS <bucket_number>
  ```

  > **注意**
  >
  > ランダムバケッティング戦略を持つ非同期MATERIALIZED VIEWは、コロケーショングループに割り当てることはできません。

  詳細については、[ランダムバケッティング](../../../table_design/Data_distribution.md#random-bucketing-since-v31)を参照してください。

**refresh_moment**（オプション）

MATERIALIZED VIEWのリフレッシュモーメント。デフォルト値は`IMMEDIATE`です。有効な値は次のとおりです。

- `IMMEDIATE`：非同期MATERIALIZED VIEWを作成した直後に即座にリフレッシュします。
- `DEFERRED`：非同期MATERIALIZED VIEWを作成した後はリフレッシュしません。MATERIALIZED VIEWを手動でリフレッシュするか定期的なリフレッシュタスクをスケジュールします。

**refresh_scheme**（オプション）

> **注意**
>
> 非同期MATERIALIZED VIEWを作成する際には、`distribution_desc`または`refresh_scheme`、またはその両方を指定する必要があります。

非同期MATERIALIZED VIEWのリフレッシュ戦略。有効な値は次のとおりです。

- `ASYNC`：非同期リフレッシュモード。基本テーブルのデータが変更されるたびに、事前に定義されたリフレッシュ間隔に従って自動的にMATERIALIZED VIEWがリフレッシュされます。リフレッシュの開始時間を`START('yyyy-MM-dd hh:mm:ss')`として指定し、リフレッシュ間隔を`EVERY (interval n day/hour/minute/second)`のように指定します。使用できる単位は`DAY`、`HOUR`、`MINUTE`、`SECOND`です。例：`ASYNC START ('2023-09-12 16:30:25') EVERY (INTERVAL 5 MINUTE)`。間隔を指定しない場合、デフォルト値として`10 MINUTE`が使用されます。
- `MANUAL`：手動リフレッシュモード。MATERIALIZED VIEWは自動的にリフレッシュされません。リフレッシュタスクはユーザーによってのみ手動でトリガーできます。

このパラメータが指定されていない場合、デフォルト値として`MANUAL`が使用されます。

**partition_expression**（オプション）

非同期MATERIALIZED VIEWのパーティション化戦略。現在のStarRocksのバージョンでは、非同期MATERIALIZED VIEWを作成する際には、1つのパーティション式のみがサポートされています。

> **注意**
>
> 現在、非同期MATERIALIZED VIEWでは、リストパーティション戦略はサポートされていません。

有効な値は次のとおりです。

- `column_name`：パーティション化に使用される列の名前。式`PARTITION BY dt`は、`dt`列に基づいてMATERIALIZED VIEWをパーティション化することを意味します。
- `date_trunc`関数：時間単位を切り捨てるために使用される関数。`PARTITION BY date_trunc("MONTH", dt)`は、`dt`列がパーティション化の単位として月に切り捨てられることを意味します。`date_trunc`関数は`YEAR`、`MONTH`、`DAY`、`HOUR`、`MINUTE`などの単位で時間を切り捨てることをサポートしています。
- `str2date`関数：ベーステーブルの文字列型のパーティションをMATERIALIZED VIEWのパーティションに分割するために使用される関数。`PARTITION BY str2date(dt, "%Y%m%d")`は、`dt`列が日付形式`"%Y%m%d"`の文字列日付型であることを意味します。この`str2date`関数は多くの日付フォーマットをサポートしており、詳細については[str2date](../../sql-functions/date-time-functions/str2date.md)を参照してください。v3.1.4以降でサポートされています。
- `time_slice`または`date_slice`関数：v3.1以降、これらの関数を使用して指定された時間を、指定した時間の粒度に基づいて時間間隔の始まりまたは終わりに変換できます。たとえば、`PARTITION BY date_trunc("MONTH", time_slice(dt, INTERVAL 7 DAY))`では、time_sliceとdate_sliceは、date_truncの粒度よりも細かくなければならず、これらを使用して、パーティションキーの粒度よりも細かいGROUP BY列を指定することができます。

このパラメータが指定されていない場合、デフォルトでパーティション化戦略は採用されません。

**order_by_expression**（オプション）

非同期MATERIALIZED VIEWのソートキー。ソートキーを指定しない場合、StarRocksはSELECT列からいくつかのプレフィックス列をソートキーとして選択します。たとえば、`select a, b, c, d`の場合、ソートキーは`a`と`b`になります。このパラメータは、StarRocks v3.0以降でサポートされています。

**PROPERTIES**（オプション）

非同期MATERIALIZED VIEWのプロパティ。[ALTER MATERIALIZED VIEW](./ALTER_MATERIALIZED_VIEW.md)を使用して既存のMATERIALIZED VIEWのプロパティを変更できます。

- `session.`：MATERIALIZED VIEWのセッション変数関連プロパティを変更する場合は、プロパティに`session.`接頭辞を追加する必要があります。たとえば、`session.query_timeout`のような場合はセッションをプレフィックスとして指定する必要があります。非セッションのプロパティの場合、たとえば`mv_rewrite_staleness_second`の場合は、プレフィックスを指定する必要はありません。
- `replication_num`：作成するMATERIALIZED VIEWのレプリカ数。
- `storage_medium`：ストレージメディアのタイプ。有効な値は`HDD`と`SSD`です。
- `storage_cooldown_time`：パーティションのためのストレージ冷却時間。HDDおよびSSDストレージメディアの両方が使用されている場合、このプロパティで指定された時間の後、SSDストレージのデータがHDDストレージに移動されます。形式："yyyy-MM-dd HH:mm:ss"。指定された時間は現在時刻よりも後でなければなりません。このプロパティが明示的に指定されていない場合、ストレージの冷却はデフォルトで行われません。
- `partition_ttl`：パーティションの有効期間（TTL）。指定された時間範囲内のデータを保持するパーティションがあります。期限切れのパーティションは自動的に削除されます。単位：`YEAR`、`MONTH`、`DAY`、`HOUR`、`MINUTE`。たとえば、このプロパティを`2 MONTH`として指定できます。このプロパティは`partition_ttl_number`よりも推奨されます。v3.1.5以降でサポートされています。
- `partition_ttl_number`：保持される最新のMATERIALIZED VIEWパーティションの数。現在時刻よりも前の開始時間を持つパーティションについては、これらのパーティションの数がこの値を超えると、より新しくないパーティションが削除されます。StarRocksは定期的にFE構成アイテム`dynamic_partition_check_interval_seconds`で指定された時間間隔に従ってMATERIALIZED VIEWパーティションを確認し、期限切れのパーティションを自動的に削除します。[動的パーティショニング](../../../table_design/dynamic_partitioning.md)戦略を有効にした場合、予め作成されたパーティションは数えられません。値が`-1`の場合、全てのMATERIALIZED VIEWのパーティションが保持されます。デフォルト値：`-1`。
- `excluded_trigger_tables`：このパラメータにマテリアライズド・ビューの基本テーブルがリストされている場合、基本テーブルのデータが変更されたときに自動リフレッシュがトリガーされません。このパラメータはロードトリガー・リフレッシュ戦略にのみ適用され、通常は`auto_refresh_partitions_limit`プロパティと一緒に使用されます。フォーマット：`[db_name.]table_name`。値が空の文字列の場合、すべての基本テーブルでのデータ変更が対応するマテリアライズド・ビューのリフレッシュをトリガーします。デフォルト値は空の文字列です。

- `auto_refresh_partitions_limit`：マテリアライズド・ビューのリフレッシュがトリガーされたときにリフレッシュする必要のある最も最近のマテリアライズド・ビューのパーティションの数です。このプロパティを使用して、リフレッシュ範囲を制限し、リフレッシュコストを削減できます。ただし、すべてのパーティションがリフレッシュされないため、マテリアライズド・ビューのデータは基本テーブルと一貫性がない場合があります。デフォルト：`-1`。値が`-1`の場合、すべてのパーティションがリフレッシュされます。値が正の整数Nの場合、StarRocksは既存のパーティションを時系列順にソートし、現在のパーティションと直近のN-1個のパーティションをリフレッシュします。パーティションの数がNより少ない場合、StarRocksはすべての既存のパーティションをリフレッシュします。マテリアライズド・ビューで事前に作成されたダイナミックなパーティションがある場合、StarRocksはすべての事前に作成されたパーティションをリフレッシュします。

- `mv_rewrite_staleness_second`：このプロパティで指定された時間間隔内にマテリアライズド・ビューの最終リフレッシュがある場合、このマテリアライズド・ビューは基本テーブルのデータが変更されているかどうかにかかわらず、クエリの書き換えに直接使用できます。前回のリフレッシュがこの時間間隔より前の場合、StarRocksはマテリアライズド・ビューがクエリの書き換えに使用できるかどうかを決定するために基本テーブルが更新されたかどうかをチェックします。単位：秒。このプロパティはv3.0からサポートされています。

- `colocate_with`：非同期マテリアライズド・ビューのコロケーション・グループ。詳細は[Colocate Join](../../../using_starrocks/Colocate_join.md)を参照してください。このプロパティはv3.0からサポートされています。

- `unique_constraints`および`foreign_key_constraints`：View Delta Joinシナリオでクエリの書き換え用に非同期マテリアライズド・ビューを作成する場合のUnique Key制約およびForeign Key制約。詳細は[非同期マテリアライズド・ビュー - View Delta Joinシナリオでのクエリ書き換え](../../../using_starrocks/query_rewrite_with_materialized_views.md)を参照してください。このプロパティはv3.0からサポートされています。

- `resource_group`：マテリアライズド・ビューのリフレッシュタスクが属するリソース・グループ。詳細は[リソース・グループ](../../../administration/resource_group.md)を参照してください。

- `query_rewrite_consistency`：非同期マテリアライズド・ビューのクエリ書き換えルール。このプロパティはv3.2からサポートされています。有効な値：
  - `disable`：非同期マテリアライズド・ビューの自動クエリ書き換えを無効にします。
  - `checked`（デフォルト値）：タイムリネス要件を満たすときのみ非同期マテリアライズド・ビューの自動クエリ書き換えを有効にします。つまり：
    - `mv_rewrite_staleness_second`が指定されていない場合、マテリアライズド・ビューはそのデータがすべての基本テーブルと一貫している場合のみクエリの書き換えに使用できます。
    - `mv_rewrite_staleness_second`が指定されている場合、マテリアライズド・ビューは最終リフレッシュがタイムリネス時間間隔内にある場合にクエリの書き換えに使用できます。
  - `loose`：自動クエリ書き換えを直接有効にし、一貫性チェックは必要ありません。

- `force_external_table_query_rewrite`：外部カタログベースのマテリアライズド・ビュー用のクエリ書き換えを有効にするかどうか。このプロパティはv3.2からサポートされています。有効な値：
  - `true`：外部カタログベースのマテリアライズド・ビュー用のクエリ書き換えを有効にします。
  - `false`（デフォルト値）：外部カタログベースのマテリアライズド・ビュー用のクエリ書き換えを無効にします。
| フィールド    | タイプ         | Null | Key  | デフォルト | その他 |
+--------------+--------------+------+------+---------+-------+
| k1           | INT          | Yes  | true | N/A     |       |
| k2           | INT          | Yes  | true | N/A     |       |
| k3           | BIGINT       | Yes  | true | N/A     |       |
| k4           | BIGINT       | Yes  | true | N/A     |       |
+--------------+--------------+------+------+---------+-------+
```

例1: 元のテーブル（k1、k2）の列のみを含む同期マテリアライズドビューを作成します。

```sql
create materialized view k1_k2 as
select k1, k2 from duplicate_table;
```

マテリアライズドビューには集約がないため、k1とk2の2つの列のみが含まれています。

```plain text
+-----------------+-------+--------+------+------+---------+-------+
| インデックス名  | フィールド | タイプ | Null | Key  | デフォルト | その他 |
+-----------------+-------+--------+------+------+---------+-------+
| k1_k2           | k1    | INT    | Yes  | true | N/A     |       |
|                 | k2    | INT    | Yes  | true | N/A     |       |
+-----------------+-------+--------+------+------+---------+-------+
```

例2: k2でソートされた同期マテリアライズドビューを作成します。

```sql
create materialized view k2_order as
select k2, k1 from duplicate_table order by k2;
```

マテリアライズドビューのスキーマは以下の通りです。マテリアライズドビューには、k2とk1の2つの列だけが含まれており、列k2は集約されず、ソート列です。

```plain text
+-----------------+-------+--------+------+-------+---------+-------+
| インデックス名  | フィールド | タイプ | Null | Key   | デフォルト | その他 |
+-----------------+-------+--------+------+-------+---------+-------+
| k2_order        | k2    | INT    | Yes  | true  | N/A     |       |
|                 | k1    | INT    | Yes  | false | N/A     | NONE  |
+-----------------+-------+--------+------+-------+---------+-------+
```

例3: k1とk2でグループ化され、k3にSUM集約を行った同期マテリアライズドビューを作成します。

```sql
create materialized view k1_k2_sumk3 as
select k1, k2, sum(k3) from duplicate_table group by k1, k2;
```

マテリアライズドビューのスキーマは以下の通りです。マテリアライズドビューには、k1、k2、sum(k3)の3つの列が含まれており、k1、k2はグループ化された列であり、sum(k3)はk1とk2に従ってグループ化されたk3列の合計です。

```plain text
+-----------------+-------+--------+------+-------+---------+-------+
| インデックス名  | フィールド | タイプ | Null | Key   | デフォルト | その他 |
+-----------------+-------+--------+------+-------+---------+-------+
| k1_k2_sumk3     | k1    | INT    | Yes  | true  | N/A     |       |
|                 | k2    | INT    | Yes  | true  | N/A     |       |
|                 | k3    | BIGINT | Yes  | false | N/A     | SUM   |
+-----------------+-------+--------+------+-------+---------+-------+
```

マテリアライズドビューにはソート列が宣言されておらず、集約関数が採用されているため、StarRocksはデフォルトでグループ化された列k1とk2を補完します。

例4: 重複行を削除する同期マテリアライズドビューを作成します。

```sql
create materialized view deduplicate as
select k1, k2, k3, k4 from duplicate_table group by k1, k2, k3, k4;
```

マテリアライズドビューのスキーマは以下の通りです。マテリアライズドビューには、k1、k2、k3、k4列が含まれており、重複する行はありません。

```plain text
+-----------------+-------+--------+------+-------+---------+-------+
| インデックス名  | フィールド | タイプ | Null | Key   | デフォルト | その他 |
+-----------------+-------+--------+------+-------+---------+-------+
| deduplicate     | k1    | INT    | Yes  | true  | N/A     |       |
|                 | k2    | INT    | Yes  | true  | N/A     |       |
|                 | k3    | BIGINT | Yes  | true  | N/A     |       |
|                 | k4    | BIGINT | Yes  | true  | N/A     |       |
+-----------------+-------+--------+------+-------+---------+-------+
```

例5: ソート列を宣言しない集約されていない同期マテリアライズドビューを作成します。

基底テーブルのスキーマは次の通りです：

```plain text
+-------+--------------+------+-------+---------+-------+
| フィールド | タイプ         | Null | Key   | デフォルト | その他 |
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

マテリアライズドビューには、k3、k4、k5、k6、k7の列が含まれており、ソート列が宣言されていません。次のステートメントでマテリアライズドビューを作成してください：

```sql
create materialized view mv_1 as
select k3, k4, k5, k6, k7 from all_type_table;
```

StarRocksはデフォルトでk3、k4、k5をソート列として使用します。これらの三つの列が占有するバイトの合計は、4 (INT) + 8 (BIGINT) + 16 (DECIMAL) = 28 で、これらの三つの列は36よりも少ないため、ソート列として追加されます。

マテリアライズドビューのスキーマは以下の通りです。

```plain text
+----------------+-------+--------------+------+-------+---------+-------+
| インデックス名 | フィールド | タイプ         | Null | Key   | デフォルト | その他 |
+----------------+-------+--------------+------+-------+---------+-------+
| mv_1           | k3    | INT          | Yes  | true  | N/A     |       |
|                | k4    | BIGINT       | Yes  | true  | N/A     |       |
|                | k5    | DECIMAL(9,0) | Yes  | true  | N/A     |       |
|                | k6    | DOUBLE       | Yes  | false | N/A     | NONE  |
|                | k7    | VARCHAR(20)  | Yes  | false | N/A     | NONE  |
+----------------+-------+--------------+------+-------+---------+-------+
```

k3、k4、k5の`key`フィールドが`true`であることが観察され、それはソートキーであることを示しています。k6 および k7の`key`フィールドが`false`であることが観察され、それはソートキーではないことを示しています。

例6: WHERE句および複雑な式を含む同期マテリアライズドビューを作成します。

```SQL
-- ベーステーブルを作成：user_event
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

  -- Create the materialized view with the WHERE clause and complex expresssions.
  CREATE MATERIALIZED VIEW test_mv1
  AS 
  SELECT
  ds,
  column_19,
  column_36,
  sum(column_01) as column_01_sum,
  bitmap_union(to_bitmap( user_id)) as user_id_dist_cnt,
  bitmap_union(to_bitmap(case when column_01 > 1 and column_34 IN ('1','34')   then user_id2 else null end)) as filter_dist_cnt_1,
  bitmap_union(to_bitmap( case when column_02 > 60 and column_35 IN ('11','13') then  user_id2 else null end)) as filter_dist_cnt_2,
  bitmap_union(to_bitmap(case when column_03 > 70 and column_36 IN ('21','23') then  user_id2 else null end)) as filter_dist_cnt_3,
  bitmap_union(to_bitmap(case when column_04 > 20 and column_27 IN ('31','27') then  user_id2 else null end)) as filter_dist_cnt_4,
  bitmap_union(to_bitmap( case when column_05 > 90 and column_28 IN ('41','43') then  user_id2 else null end)) as filter_dist_cnt_5
  FROM user_event
  WHERE ds >= '2023-11-02'
  GROUP BY
  ds,
  column_19,
  column_36;
```

### 非同期マテリアライズド・ビューの例

以下の例は、以下のベーステーブルに基づいています。

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
PARTITION p7 VALUES [("19980101"), ("19990101")))
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

create table orders ( 
    dt date NOT NULL, 
    order_id bigint NOT NOT NULL, 
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

例1：パーティションされていないマテリアライズド・ビューを作成します。

```SQL
CREATE MATERIALIZED VIEW lo_mv1
DISTRIBUTED BY HASH(`lo_orderkey`)
REFRESH ASYNC
AS
select
    lo_orderkey, 
    lo_custkey, 
    sum(lo_quantity) as total_quantity, 
    sum(lo_revenue) as total_revenue, 
    count(lo_shipmode) as shipmode_count
from lineorder 
group by lo_orderkey, lo_custkey 
order by lo_orderkey;
```

例2：パーティションされたマテリアライズド・ビューを作成します。

```SQL
CREATE MATERIALIZED VIEW lo_mv2
PARTITION BY `lo_orderdate`
DISTRIBUTED BY HASH(`lo_orderkey`)
```SQL
REFRESH ASYNC START('2023-07-01 10:00:00') EVERY (interval 1 day)
AS
select
    lo_orderkey,
    lo_orderdate,
    lo_custkey, 
    sum(lo_quantity) as total_quantity, 
    sum(lo_revenue) as total_revenue, 
    count(lo_shipmode) as shipmode_count
from lineorder 
group by lo_orderkey, lo_orderdate, lo_custkey
order by lo_orderkey;

-- date_trunc() 関数を使用して、マテリアライズド ビューを月でパーティション分割します。
CREATE MATERIALIZED VIEW order_mv1
PARTITION BY date_trunc('month', `dt`)
DISTRIBUTED BY HASH(`order_id`)
REFRESH ASYNC START('2023-07-01 10:00:00') EVERY (interval 1 day)
AS
select
    dt,
    order_id,
    user_id,
    sum(cnt) as total_cnt,
    sum(revenue) as total_revenue, 
    count(state) as state_count
from orders
group by dt, order_id, user_id;
```

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
    p.P_CONTAINER AS P_CONTAINER FROM lineorder AS l 
INNER JOIN customer AS c ON c.C_CUSTKEY = l.LO_CUSTKEY
INNER JOIN supplier AS s ON s.S_SUPPKEY = l.LO_SUPPKEY
INNER JOIN part AS p ON p.P_PARTKEY = l.LO_PARTKEY;
```

```SQL
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
) partition by (d_datekey);

CREATE MATERIALIZED VIEW IF NOT EXISTS `test_mv` 
PARTITION BY str2date(`d_datekey`,'%Y%m%d')
DISTRIBUTED BY HASH(`d_date`, `d_month`, `d_month`) 
REFRESH MANUAL 
AS
SELECT
`d_date` ,
  `d_dayofweek`,
  `d_month` ,
  `d_yearmonthnum` ,
  `d_yearmonth` ,
  `d_daynuminweek`,
  `d_daynuminmonth`,
  `d_daynuminyear` ,
  `d_monthnuminyear` ,
  `d_weeknuminyear` ,
  `d_sellingseason`,
  `d_lastdayinweekfl`,
  `d_lastdayinmonthfl`,
  `d_holidayfl` ,
  `d_weekdayfl`,
   `d_datekey`
FROM
 `hive_catalog`.`ssb_1g_orc`.`part_dates` ;
```