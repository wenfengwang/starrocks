---
displayed_sidebar: "Japanese"
---

# クエリ解析

クエリのパフォーマンスを最適化する方法はよく問われる質問です。遅いクエリはユーザーエクスペリエンスやクラスタのパフォーマンスに影響を与えます。クエリのパフォーマンスを分析して最適化することが重要です。

`fe/log/fe.audit.log` でクエリ情報を表示することができます。各クエリには`QueryID`が対応しており、これを使用してクエリの`QueryPlan`および`Profile`を検索することができます。`QueryPlan` は FE によって生成された実行プランであり、SQL ステートメントを解析しています。`Profile` は BE の実行結果であり、各ステップで消費される時間や各ステップで処理されるデータ量などの情報が含まれています。

## プラン解析

StarRocks では、SQL ステートメントのライフサイクルをクエリ解析、クエリプランニング、クエリ実行の３つのフェーズに分けることができます。クエリ解析は一般的にボトルネックにはなりません。なぜなら、分析ワークロードの必要な QPS が高くないためです。

StarRocks におけるクエリのパフォーマンスはクエリプランニングとクエリ実行によって決定されます。クエリプランニングは演算子（Join/Order/Aggregate）を調整し、クエリ実行は特定の操作を実行する責任があります。

クエリプランは DBA にとってマクロな視点でクエリ情報にアクセスできるようにします。クエリプランはクエリパフォーマンスの鍵であり、DBA が参照できる優れたリソースです。以下のコードスニペットは`TPCDS query96`を使用して、クエリプランを表示する方法を示しています。

~~~SQL
-- query96.sql
select  count(*)
from store_sales
    ,household_demographics
    ,time_dim
    , store
where ss_sold_time_sk = time_dim.t_time_sk
    and ss_hdemo_sk = household_demographics.hd_demo_sk
    and ss_store_sk = s_store_sk
    and time_dim.t_hour = 8
    and time_dim.t_minute >= 30
    and household_demographics.hd_dep_count = 5
    and store.s_store_name = 'ese'
order by count(*) limit 100;
~~~

クエリプランには、論理クエリプランと物理クエリプランの２種類があります。ここで説明するクエリプランは論理クエリプランを指します。`TPCDS query96.sql` に対応するクエリプランは以下の通りです。

~~~sql
+------------------------------------------------------------------------------+
| 説明文字列                                                                    |
+------------------------------------------------------------------------------+
| PLAN FRAGMENT 0                                                              |
|  OUTPUT EXPRS: <slot 11>                                                     |
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
|   |  colocate: false, reason: left hash join node can not do colocate        |
|   |  equal join conjunct: `ss_store_sk` = `s_store_sk`                       |
|   |  tuple ids: 0 2 1 3                                                      |
|   |                                                                          |
|   |----11:EXCHANGE                                                           |
|   |       tuple ids: 3                                                       |
|   |                                                                          |
|   4:HASH JOIN                                                                |
|   |  join op: INNER JOIN (BROADCAST)                                         |
|   |  hash predicates:                                                        |
|   |  colocate: false, reason: left hash join node can not do colocate        |
|   |  equal join conjunct: `ss_hdemo_sk`=`household_demographics`.`hd_demo_sk`|
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
|      PREAGGREGATION: OFF. Reason: `ss_sold_time_sk` is value column          |
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
128 行中 (0.02 秒)
~~~

クエリ96には、複数のStarRocksの概念を含むクエリプランが示されています。

|名前|説明|
|--|--|
|avgRowSize|スキャンされたデータ行の平均サイズ|
|cardinality|スキャンされたテーブルのデータ行の合計数|
|colocate|テーブルの共置モードの有無|
|numNodes|スキャンするノードの数|
|rollup|マテリアライズド・ビュー|
|preaggregation|プリアグリゲーション|
|predicates|クエリフィルタ|

クエリ96のクエリプランは0から4までの５つのフラグメントに分かれており、ボトムアップで順に読むことができます。
フラグメント4は`time_dim`テーブルをスキャンし、関連するクエリ条件（つまり、`time_dim.t_hour = 8かつtime_dim.t_minute>= 30`）を事前に実行します。このステップは述部押し込みとしても知られています。StarRocksは、集計テーブルの`PREAGGREGATION`を有効にするかどうかを決定します。前の図では、`time_dim`の事前集約は無効になっています。この場合、`time_dim`のすべての次元列が読み取られ、テーブルに多くの次元列がある場合にパフォーマンスに悪影響を及ぼす可能性があります。`time_dim`テーブルがデータ分割のために`range partition`を選択する場合、クエリプランでいくつかのパーティションが当たり、無関係なパーティションが自動的にフィルタリングされます。マテリアライズド・ビューがある場合、StarRocksはクエリに基づいてマテリアライズド・ビューを自動的に選択します。マテリアライズド・ビューがない場合、クエリは自動的に基本テーブルをアクセスします（たとえば、前の図で`ロールアップ：time_dim`）。

スキャンが完了したら、フラグメント4が終了します。データは他のフラグメントに渡され、前の図でEXCHANGE ID: 09で示されているように、受信ノードラベル9に渡されます。

クエリ96のクエリプランでは、フラグメント2、3、および4は類似した機能を持っていますが、異なるテーブルをスキャンする責任があります。特に、クエリの`Order/Aggregation/Join`操作はフラグメント1で実行されます。

フラグメント1では、`BROADCAST`メソッドを使用して`Order/Aggregation/Join`操作を実行します。つまり、小さなテーブルを大きなテーブルにブロードキャストします。両方のテーブルが大きい場合は、`SHUFFLE`メソッドを使用することをお勧めします。現在、StarRocksは`HASH JOIN`のみをサポートしています。`colocate`フィールドは、2つの結合されたテーブルが同じ方法でパーティション分割およびバケット分割されていることを示すために使用され、したがって、データを移行せずに結合操作をローカルで実行できます。結合操作が完了すると、上位レベルの`aggregation`、`order by`、および`top-n`操作が実行されます。

特定の式を削除することにより（演算子のみを保持）、クエリプランをより全体的な視点で表示することができます。次の図に示すように。

![8-5](../assets/8-5.png)

## クエリヒント

クエリヒントは、クエリオプティマイザにクエリの実行方法を明示的に示唆する指示またはコメントです。現在、StarRocksは変数設定ヒントと結合ヒントの2種類のヒントをサポートしています。ヒントは単一のクエリ内でのみ有効です。

### 変数設定ヒント

`SET_VAR`ヒントを使用して、1つ以上の[システム変数](../reference/System_variable.md)を設定することができます。 `SET_VAR`ヒントの構文は、SELECTやSUBMIT TASKステートメントでの`/*+ SET_VAR(var_name = value) */`の形式で表され、またはCREATE MATERIALIZED VIEW AS SELECTやCREATE VIEW AS SELECTなどの他のステートメントに含まれるSELECT句で使用されます。

#### 構文

~~~SQL
[...] SELECT [/*+ SET_VAR(key=value [, key = value]*) */] ...
SUBMIT [/*+ SET_VAR(key=value [, key = value]*) */] TASK ...
~~~

#### 例

集計クエリの集計方法をヒント指定して、システム変数`streaming_preaggregation_mode`および`new_planner_agg_stage`を設定します。

~~~SQL
SELECT /*+ SET_VAR (streaming_preaggregation_mode = 'force_streaming',new_planner_agg_stage = '2') */ SUM(sales_amount) AS total_sales_amount FROM sales_orders;
~~~

クエリのタスク実行タイムアウト期間を`query_timeout`システム変数で設定するために、SUBMIT TASKステートメントで使用します。

~~~SQL
SUBMIT /*+ SET_VAR(query_timeout=3) */ TASK AS CREATE TABLE temp AS SELECT count(*) AS cnt FROM tbl1;
~~~

マテリアライズド・ビューを作成する際に、クエリの実行タイムアウト期間を`query_timeout`システム変数で設定する場合、使用します。

~~~SQL
CREATE MATERIALIZED VIEW mv 
PARTITION BY dt 
DISTRIBUTED BY HASH(`key`) 
BUCKETS 10 
REFRESH ASYNC 
AS SELECT /*+ SET_VAR(query_timeout=500) */ * from dual;
~~~

### 結合ヒント

複数のテーブルを結合するクエリの場合、オプティマイザは通常、最適な結合実行方法を選択します。特別な場合では、結合ヒントを使用して、結合実行方法をオプティマイザに明示的に示唆したり、結合の再順序を無効にしたりすることができます。現在の結合ヒントは、Shuffle Join、Broadcast Join、Bucket Shuffle Join、またはColocate Joinとして結合実行方法を示唆します。結合ヒントを使用すると、オプティマイザは結合の再順序を実行しません。そのため、小さいテーブルを右側のテーブルとして選択する必要があります。また、[Colocate Join](../using_starrocks/Colocate_join.md)またはBucket Shuffle Joinを結合実行方法として示唆する場合、結合されたテーブルのデータ分布がこれらの結合実行方法の要件を満たしていることを確認する必要があります。そうでない場合、示唆された結合実行方法は効果を発揮しません。

#### 構文

~~~SQL
... JOIN { [BROADCAST] | [SHUFFLE] | [BUCKET] | [COLOCATE] | [UNREORDER]} ...
~~~

> **注意**
>
> 結合ヒントは大文字と小文字を区別しません。

#### 例

- Shuffle Join

  テーブルAとテーブルBから同じバケット化キー値を持つデータ行をJoin操作を実行する前に、データ行をシャッフルする必要がある場合、結合実行方法をShuffle Joinとして示唆することができます。

  ~~~SQL
  select k1 from t1 join [SHUFFLE] t2 on t1.k1 = t2.k2 group by t2.k2;
  ~~~

- Broadcast Join
  
  テーブルAが大きいテーブルで、テーブルBが小さいテーブルである場合、結合実行方法をBroadcast Joinとして示唆することができます。テーブルBのデータはテーブルAのデータが存在するマシンに完全にブロードキャストされ、その後Join操作が実行されます。Shuffle Joinに比べて、Broadcast JoinではテーブルAのデータをシャッフルするコストが節約されます。

  ~~~SQL
  select k1 from t1 join [BROADCAST] t2 on t1.k1 = t2.k2 group by t2.k2;
  ~~~

- Bucket Shuffle Join
  
  結合クエリの結合等値条件にテーブルAのバケット化キーが含まれる場合、特にテーブルAとテーブルBの両方が大きなテーブルの場合、結合実行方法をBucket Shuffle Joinとして示唆することができます。テーブルBのデータは、テーブルAのデータのデータ分布に従って、テーブルAのデータが存在するマシンにシャッフルされ、その後Join操作が実行されます。Broadcast Joinに比べて、Bucket Shuffle JoinはテーブルBのデータが一度だけグローバルにシャッフルされるため、データ転送のコストが著しく削減されます。

  ~~~SQL
  select k1 from t1 join [BUCKET] t2 on t1.k1 = t2.k2 group by t2.k2;
  ~~~

- Colocate Join
  
  テーブルAとテーブルBが作成時に指定された同じコロケーショングループに属する場合、結合クエリで結合等値条件がテーブルAとBのバケット化キーを含む場合、結合実行方法をColocate Joinとして示唆することができます。同じキー値を持つデータはローカルで直接結合され、ノード間のデータ転送にかかる時間が削減され、クエリのパフォーマンスが向上します。

  ~~~SQL
  select k1 from t1 join [COLOCATE] t2 on t1.k1 = t2.k2 group by t2.k2;
  ~~~

### ビューの結合実行方法

`EXPLAIN`コマンドを使用して実際の結合実行方法を表示します。返された結果が結合ヒントと一致する場合、結合ヒントが有効であることを意味します。

~~~SQL
EXPLAIN select k1 from t1 join [COLOCATE] t2 on t1.k1 = t2.k2 group by t2.k2;
~~~

![8-9](../assets/8-9.png)

## SQLフィンガープリント

SQLフィンガープリントは、遅いクエリを最適化し、システムリソースの利用を改善するために使用されます。StarRocksは、SQLフィンガープリント機能を使用して、遅いクエリログ（`fe.audit.log.slow_query`）のSQLステートメントを正規化し、異なるタイプのSQLステートメントをカテゴリ別に分類し、それぞれのSQLタイプのMD5ハッシュ値を計算して、遅いクエリを識別します。MD5ハッシュ値は、`Digest`フィールドで指定されます。

~~~SQL
2021-12-27 15:13:39,108 [slow_query] |Client=172.26.xx.xxx:54956|User=root|Db=default_cluster:test|State=EOF|Time=2469|ScanBytes=0|ScanRows=0|ReturnRows=6|StmtId=3|QueryId=824d8dc0-66e4-11ec-9fdc-00163e04d4c2|IsQuery=true|feIp=172.26.92.195|Stmt=select count(*) from test_basic group by id_bigint|Digest=51390da6b57461f571f0712d527320f4
~~~

SQLステートメントの正規化は、ステートメントテキストをより正規化された形式に変換し、重要なステートメント構造のみを保存します。

- データベースやテーブル名などのオブジェクト識別子を保存します。

- 定数を疑問符（?）に変換します。

- コメントを削除し、スペースを整形します。

たとえば、次の2つのSQLステートメントは、正規化後に同じタイプに属します。

- 正規化前のSQLステートメント
~~~SQL
SELECT * FROM orders WHERE customer_id=10 AND quantity>20



SELECT * FROM orders WHERE customer_id = 20 AND quantity > 100
~~~
```
SELECT * FROM orders WHERE customer_id=? AND quantity>?
```