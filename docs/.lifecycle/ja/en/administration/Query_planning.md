---
displayed_sidebar: English
---

# クエリ分析

クエリのパフォーマンスを最適化する方法は、よく寄せられる質問です。クエリが遅いと、ユーザーエクスペリエンスとクラスターのパフォーマンスが損なわれます。クエリのパフォーマンスを分析して最適化することが重要です。

クエリ情報は、`fe/log/fe.audit.log`で表示できます。各クエリは、`QueryID`に対応しており、これを使用してクエリの`QueryPlan`と`Profile`を検索できます。`QueryPlan`は、FEがSQLステートメントを解析して生成する実行計画です。`Profile`はBEの実行結果であり、各ステップで消費された時間や各ステップで処理されたデータ量などの情報が含まれています。

## プラン分析

StarRocksでは、SQLステートメントのライフサイクルは、クエリ解析、クエリ計画、クエリ実行の3つのフェーズに分けられます。クエリ解析は、分析ワークロードに必要なQPSが高くないため、一般にボトルネックにはなりません。

StarRocksのクエリパフォーマンスは、クエリ計画とクエリ実行によって決まります。クエリ計画は演算子（Join/Order/Aggregate）の調整を担当し、クエリ実行は特定の操作の実行を担当します。

クエリプランは、クエリ情報にアクセスするためのマクロな視点をDBAに提供します。クエリプランはクエリパフォーマンスの鍵であり、DBAが参照するための優れたリソースです。以下のコードスニペットは、`TPCDS query96`を例に、クエリプランの表示方法を示しています。

~~~SQL
-- query96.sql
select  count(*)
from store_sales
    ,household_demographics
    ,time_dim
    ,store
where ss_sold_time_sk = time_dim.t_time_sk
    and ss_hdemo_sk = household_demographics.hd_demo_sk
    and ss_store_sk = store.s_store_sk
    and time_dim.t_hour = 8
    and time_dim.t_minute >= 30
    and household_demographics.hd_dep_count = 5
    and store.s_store_name = 'ese'
order by count(*) limit 100;
~~~

クエリプランには、論理クエリプランと物理クエリプランの2種類があります。ここで説明するクエリプランは、論理クエリプランを指します。`TPCDS query96.sql`に対応するクエリプランは以下に示します。

~~~sql
+------------------------------------------------------------------------------+
| Explain String                                                               |
+------------------------------------------------------------------------------+
| PLAN FRAGMENT 0                                                              |
|  OUTPUT EXPRS:<slot 11>                                                      |
|   PARTITION: UNPARTITIONED                                                   |
|   RESULT SINK                                                                |
|   12:MERGING-EXCHANGE                                                        |
|      limit: 100                                                              |
|      tuple ids: 5                                                            |
|                                                                              |
| PLAN FRAGMENT 1                                                              |
|  OUTPUT EXPRS:                                                               |
|   PARTITION: RANDOM                                                          |
|   STREAM DATA SINK                                                           |
|     EXCHANGE ID: 12                                                          |
|     UNPARTITIONED                                                            |
|                                                                              |
|   8:TOP-N                                                                    |
|   |  order by: <slot 11> ASC                                                 |
|   |  offset: 0                                                               |
|   |  limit: 100                                                              |
|   |  tuple ids: 5                                                            |
|   |                                                                          |
|   7:AGGREGATE (update finalize)                                              |
|   |  output: count(*)                                                        |
|   |  group by:                                                               |
|   |  tuple ids: 4                                                            |
|   |                                                                          |
|   6:HASH JOIN                                                                |
|   |  join op: INNER JOIN (BROADCAST)                                         |
|   |  hash predicates:                                                        |
|   |  colocate: false, reason: left hash join node cannot do colocate         |
|   |  equal join conjunct: `ss_store_sk` = `store`.`s_store_sk`               |
|   |  tuple ids: 0 2 1 3                                                      |
|   |                                                                          |
|   |----11:EXCHANGE                                                           |
|   |       tuple ids: 3                                                       |
|   |                                                                          |
|   4:HASH JOIN                                                                |
|   |  join op: INNER JOIN (BROADCAST)                                         |
|   |  hash predicates:                                                        |
|   |  colocate: false, reason: left hash join node cannot do colocate         |
|   |  equal join conjunct: `ss_hdemo_sk` = `household_demographics`.`hd_demo_sk`|
|   |  tuple ids: 0 2 1                                                        |
|   |                                                                          |
|   |----10:EXCHANGE                                                           |
|   |       tuple ids: 1                                                       |
|   |                                                                          |
|   2:HASH JOIN                                                                |
|   |  join op: INNER JOIN (BROADCAST)                                         |
|   |  hash predicates:                                                        |
|   |  colocate: false, reason: table not in same group                        |
|   |  equal join conjunct: `ss_sold_time_sk` = `time_dim`.`t_time_sk`         |
|   |  tuple ids: 0 2                                                          |
|   |                                                                          |
|   |----9:EXCHANGE                                                            |
|   |       tuple ids: 2                                                       |
|   |                                                                          |
|   0:OlapScanNode                                                             |
|      TABLE: store_sales                                                      |
|      PREAGGREGATION: OFF. Reason: `ss_sold_time_sk` is a value column        |
|      partitions=1/1                                                          |
|      rollup: store_sales                                                     |
|      tabletRatio=0/0                                                         |
|      tabletList=                                                             |
|      cardinality=-1                                                          |
|      avgRowSize=0.0                                                          |
|      numNodes=0                                                              |
|      tuple ids: 0                                                            |
|                                                                              |
| PLAN FRAGMENT 2                                                              |
|  OUTPUT EXPRS:                                                               |
|   PARTITION: RANDOM                                                          |
|                                                                              |
|   STREAM DATA SINK                                                           |
|     EXCHANGE ID: 11                                                          |
|     UNPARTITIONED                                                            |
|                                                                              |
|   5:OlapScanNode                                                             |
|      TABLE: store                                                            |
|      PREAGGREGATION: OFF. Reason: null                                       |
|      PREDICATES: `store`.`s_store_name` = 'ese'                              |
|      partitions=1/1                                                          |
|      rollup: store                                                           |
|      tabletRatio=0/0                                                         |
|      tabletList=                                                             |
|      cardinality=-1                                                          |
|      avgRowSize=0.0                                                          |
|      numNodes=0                                                              |
|      tuple ids: 3                                                            |
|                                                                              |
| PLAN FRAGMENT 3                                                              |
|  OUTPUT EXPRS:                                                               |
|   PARTITION: RANDOM                                                          |
|   STREAM DATA SINK                                                           |
|     EXCHANGE ID: 10                                                          |
|     UNPARTITIONED                                                            |
|                                                                              |
|   3:OlapScanNode                                                             |
|      TABLE: household_demographics                                           |
|      PREAGGREGATION: OFF. Reason: null                                       |
|      PREDICATES: `household_demographics`.`hd_dep_count` = 5                 |
|      partitions=1/1                                                          |
|      rollup: household_demographics                                          |
|      tabletRatio=0/0                                                         |
|      tabletList=                                                             |
|      cardinality=-1                                                          |
|      avgRowSize=0.0                                                          |
|      numNodes=0                                                              |
|      tuple ids: 1                                                            |
|                                                                              |
| PLAN FRAGMENT 4                                                              |
|  OUTPUT EXPRS:                                                               |
|   PARTITION: RANDOM                                                          |
|   STREAM DATA SINK                                                           |
|     EXCHANGE ID: 09                                                          |
|     UNPARTITIONED                                                            |
|                                                                              |
|   1:OlapScanNode                                                             |
|      TABLE: time_dim                                                         |
|      PREAGGREGATION: OFF. Reason: null                                       |
|      PREDICATES: `time_dim`.`t_hour` = 8, `time_dim`.`t_minute` >= 30        |
|      partitions=1/1                                                          |
|      rollup: time_dim                                                        |
|      tabletRatio=0/0                                                         |
|      tabletList=                                                             |
|      cardinality=-1                                                          |
|      avgRowSize=0.0                                                          |
|      numNodes=0                                                              |
|      tuple ids: 2                                                            |
+------------------------------------------------------------------------------+
128 rows in set (0.02 sec)
~~~

クエリ96は、いくつかのStarRocksの概念を含むクエリプランを示しています。

|名前|説明|
|--|--|
|avgRowSize|スキャンされたデータ行の平均サイズ|
|cardinality|スキャンされたテーブル内のデータ行の合計数|
|colocate|テーブルがコロケートモードかどうか|
|numNodes|スキャンするノードの数|
|rollup|マテリアライズドビュー|
|preaggregation|プリアグリゲーション|
|predicates|述語、クエリフィルタ|

クエリ96のクエリプランは、0から4までの番号が付けられた5つのフラグメントに分割されます。クエリプランは、ボトムアップ方式で1つずつ読み取ることができます。

フラグメント4は、`time_dim`テーブルをスキャンし、関連するクエリ条件（`time_dim.t_hour = 8 and time_dim.t_minute >= 30`）を事前に実行する役割を担います。このステップは、述語プッシュダウンとも呼ばれます。StarRocksは、集計テーブルに対してプリアグリゲーションを有効にするかどうかを決定します。前の図では、`time_dim`のプリアグリゲーションは無効になっています。この場合、`time_dim`のすべてのディメンション列が読み取られるため、テーブルに多数のディメンション列がある場合はパフォーマンスに悪影響を与える可能性があります。`time_dim`テーブルがデータ分割にレンジパーティションを選択すると、クエリプランで複数のパーティションがヒットし、無関係なパーティションが自動的に除外されます。マテリアライズドビューがある場合、StarRocksはクエリに基づいてマテリアライズドビューを自動的に選択します。マテリアライズドビューがない場合、クエリは自動的にベーステーブルにヒットします（たとえば、前の図の`rollup: time_dim`）。

スキャンが完了すると、フラグメント4は終了します。データは、前の図のEXCHANGE ID: 09で示されているように、ラベル9の付いた受信ノードに他のフラグメントに渡されます。

クエリ96のクエリプランでは、フラグメント2、3、および4は同様の機能を持ちますが、異なるテーブルをスキャンする役割を担います。具体的には、クエリ内の`Order`/`Aggregation`/`Join`操作はフラグメント1で実行されます。

フラグメント1は、`BROADCAST` メソッドを使用して `Order/Aggregation/Join` 操作を実行します。つまり、小さなテーブルを大きなテーブルにブロードキャストします。両方のテーブルが大きい場合は、`SHUFFLE` メソッドの使用をお勧めします。現在、StarRocksは `HASH JOIN` のみをサポートしています。`colocate` フィールドは、2つの結合テーブルが同じ方法でパーティション分割およびバケット化されていることを示すために使用され、データを移行せずに結合操作をローカルで実行できます。Join操作が完了すると、上位レベルの `aggregation`、`order by`、および `top-n` 操作が実行されます。

特定の式を削除する（演算子のみを保持する）ことで、次の図に示すように、クエリプランをよりマクロなビューで表示できます。

![8-5](../assets/8-5.png)

## クエリヒント

クエリヒントは、クエリの実行方法をクエリオプティマイザーに明示的に提案するディレクティブまたはコメントです。現在、StarRocksは変数設定ヒントと結合ヒントの2種類のヒントをサポートしています。ヒントは、1つのクエリ内でのみ有効です。

### 変数設定ヒント

1つ以上の[システム変数](../reference/System_variable.md)を設定するには、SELECTおよびSUBMIT TASKステートメント、またはCREATE MATERIALIZED VIEW AS SELECTやCREATE VIEW AS SELECTなどの他のステートメントに含まれるSELECT句で `/*+ SET_VAR(var_name = value) */` の形式の `SET_VAR` ヒントを使用します。

#### 構文

~~~SQL
[...] SELECT [/*+ SET_VAR(key=value [, key = value]*) */] ...
SUBMIT [/*+ SET_VAR(key=value [, key = value]*) */] TASK ...
~~~

#### 例

集約クエリの集約方法を指定するには、システム変数 `streaming_preaggregation_mode` と `new_planner_agg_stage` を設定します。

~~~SQL
SELECT /*+ SET_VAR(streaming_preaggregation_mode = 'force_streaming', new_planner_agg_stage = '2') */ SUM(sales_amount) AS total_sales_amount FROM sales_orders;
~~~

クエリのタスク実行タイムアウト期間を指定するには、SUBMIT TASKステートメントでシステム変数 `query_timeout` を設定します。

~~~SQL
SUBMIT /*+ SET_VAR(query_timeout=3) */ TASK AS CREATE TABLE temp AS SELECT count(*) AS cnt FROM tbl1;
~~~

マテリアライズドビューの作成時にSELECT句で `query_timeout` システム変数を設定することにより、クエリの実行タイムアウト期間を指定します。

~~~SQL
CREATE MATERIALIZED VIEW mv 
PARTITION BY dt 
DISTRIBUTED BY HASH(`key`) 
BUCKETS 10 
REFRESH ASYNC 
AS SELECT /*+ SET_VAR(query_timeout=500) */ * FROM dual;
~~~

### 結合ヒント

複数テーブル結合クエリの場合、オプティマイザは通常、最適な結合実行方法を選択します。特殊なケースでは、結合ヒントを使用してオプティマイザに結合実行方法を明示的に提案したり、Join Reorderを無効にしたりできます。現在、結合ヒントでは、Shuffle Join、Broadcast Join、Bucket Shuffle Join、またはColocate Joinを結合実行方法として提案することがサポートされています。結合ヒントを使用すると、オプティマイザはJoin Reorderを実行しません。したがって、小さいテーブルを右側のテーブルとして選択する必要があります。また、[Colocate Join](../using_starrocks/Colocate_join.md)またはBucket Shuffle Joinを結合実行方法として提案する場合は、結合されたテーブルのデータ分散がこれらの結合実行方法の要件を満たしていることを確認してください。そうでない場合、提案された結合実行方法は有効になりません。

#### 構文

~~~SQL
... JOIN { [BROADCAST] | [SHUFFLE] | [BUCKET] | [COLOCATE] | [UNREORDER] } ...
~~~

> **注記**
>
> 結合ヒントは大文字と小文字を区別しません。

#### 例

- Shuffle Join

  結合操作を実行する前に、テーブルAとBの同じバケットキー値を持つデータ行を同じマシンにシャッフルする必要がある場合は、結合実行方法としてShuffle Joinを指定できます。

  ~~~SQL
  SELECT k1 FROM t1 JOIN [SHUFFLE] t2 ON t1.k1 = t2.k2 GROUP BY t2.k2;
  ~~~

- Broadcast Join
  
  テーブルAが大きなテーブルで、テーブルBが小さなテーブルの場合、結合実行方法としてBroadcast Joinを指定できます。テーブルBのデータは、テーブルAのデータが存在するマシンに完全にブロードキャストされ、その後結合操作が実行されます。Shuffle Joinと比較して、Broadcast JoinはテーブルAのデータをシャッフルするコストを節約します。

  ~~~SQL
  SELECT k1 FROM t1 JOIN [BROADCAST] t2 ON t1.k1 = t2.k2 GROUP BY t2.k2;
  ~~~

- Bucket Shuffle Join
  
  結合クエリのJoin等価結合式にテーブルAのバケットキーが含まれている場合、特にテーブルAとBの両方が大きなテーブルの場合、結合実行方法としてBucket Shuffle Joinを指定できます。テーブルBのデータは、テーブルAのデータ分散に従って、テーブルAのデータが存在するマシンにシャッフルされ、その後結合操作が実行されます。Broadcast Joinと比較して、Bucket Shuffle Joinは、テーブルBのデータがグローバルに一度だけシャッフルされるため、データ転送を大幅に削減します。

  ~~~SQL
  SELECT k1 FROM t1 JOIN [BUCKET] t2 ON t1.k1 = t2.k2 GROUP BY t2.k2;
  ~~~

- Colocate Join
  
  テーブルAとBが、テーブル作成時に指定された同じColocation Groupに属している場合、テーブルAとBの同じバケットキー値を持つデータ行は、同じBEノードに分散されます。結合クエリのJoin等価結合式にテーブルAとBのバケットキーが含まれている場合、結合実行方法としてColocate Joinを指定できます。同じキー値を持つデータはローカルで直接結合されるため、ノード間のデータ転送に費やされる時間が短縮され、クエリのパフォーマンスが向上します。

  ~~~SQL
  SELECT k1 FROM t1 JOIN [COLOCATE] t2 ON t1.k1 = t2.k2 GROUP BY t2.k2;
  ~~~

### 結合実行方法の表示

`EXPLAIN` コマンドを使用して、実際の結合実行方法を表示します。返された結果が結合ヒントと一致している場合、それは結合ヒントが有効であることを意味します。

~~~SQL
EXPLAIN SELECT k1 FROM t1 JOIN [COLOCATE] t2 ON t1.k1 = t2.k2 GROUP BY t2.k2;
~~~

![8-9](../assets/8-9.png)

## SQLフィンガープリント

SQLフィンガープリントは、遅いクエリを最適化し、システムリソースの利用を改善するために使用されます。StarRocksはSQLフィンガープリント機能を使用して、スロークエリログ(`fe.audit.log.slow_query`)内のSQLステートメントを正規化し、SQLステートメントを異なるタイプに分類し、各SQLタイプのMD5ハッシュ値を計算して遅いクエリを特定します。MD5ハッシュ値は`Digest`フィールドによって指定されます。

~~~SQL
2021-12-27 15:13:39,108 [slow_query] |Client=172.26.xx.xxx:54956|User=root|Db=default_cluster:test|State=EOF|Time=2469|ScanBytes=0|ScanRows=0|ReturnRows=6|StmtId=3|QueryId=824d8dc0-66e4-11ec-9fdc-00163e04d4c2|IsQuery=true|feIp=172.26.92.195|Stmt=SELECT COUNT(*) FROM test_basic GROUP BY id_bigint|Digest=51390da6b57461f571f0712d527320f4
~~~

SQLステートメントの正規化は、ステートメントテキストをより正規化された形式に変換し、重要なステートメント構造のみを保持します。

- オブジェクト識別子、例えばデータベース名やテーブル名を保持します。

- 定数を疑問符(?)に変換します。

- コメントを削除し、スペースを整形します。

例えば、以下の2つのSQLステートメントは、正規化後に同じタイプに属します。

- 正規化前のSQLステートメント

~~~SQL
SELECT * FROM orders WHERE customer_id=10 AND quantity>20



SELECT * FROM orders WHERE customer_id = 20 AND quantity > 100
~~~

- 正規化後のSQLステートメント

~~~SQL
SELECT * FROM orders WHERE customer_id=? AND quantity>?
~~~

