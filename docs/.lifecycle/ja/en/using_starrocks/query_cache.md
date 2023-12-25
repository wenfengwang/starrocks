---
displayed_sidebar: English
---

# クエリキャッシュ

クエリキャッシュは、集約クエリのパフォーマンスを大幅に向上させるStarRocksの強力な機能です。クエリキャッシュは、ローカル集約の中間結果をメモリに格納することで、以前のクエリと同一または類似した新しいクエリの不要なディスクアクセスと計算を回避できます。クエリキャッシュを使用することで、StarRocksは集約クエリの迅速かつ正確な結果を提供し、時間とリソースを節約し、スケーラビリティを向上させることができます。クエリキャッシュは、多数のユーザーが大規模で複雑なデータセットに対して類似のクエリを実行する高コンカレンシーのシナリオに特に有効です。

この機能はv2.5以降でサポートされています。

v2.5では、クエリキャッシュは単一のフラットテーブルに対する集約クエリのみをサポートします。v3.0以降では、クエリキャッシュはスタースキーマで結合された複数のテーブルに対する集約クエリもサポートしています。

## 適用シナリオ

クエリキャッシュの使用を推奨するシナリオは以下の通りです：

- 個別のフラットテーブルやスタースキーマで接続された複数の結合テーブルに対して頻繁に集約クエリを実行します。
- ほとんどの集約クエリが非GROUP BY集約クエリや低カーディナリティのGROUP BY集約クエリです。
- データは時間パーティションによって追加モードでロードされ、アクセス頻度に基づいてホットデータとコールドデータに分類されます。

クエリキャッシュは、以下の条件を満たすクエリをサポートします：

- クエリエンジンはPipelineです。Pipelineエンジンを有効にするには、セッション変数 `enable_pipeline_engine` を `true` に設定します。

  > **注記**
  >
  > 他のクエリエンジンはクエリキャッシュをサポートしていません。

- クエリは、ネイティブOLAPテーブル（v2.5以降）またはクラウドネイティブテーブル（v3.0以降）に対して行われます。クエリキャッシュは外部テーブルに対するクエリをサポートしていません。クエリキャッシュは、同期マテリアライズドビューへのアクセスが必要なプランのクエリもサポートしますが、非同期マテリアライズドビューへのアクセスが必要なプランのクエリはサポートしません。

- クエリは、個々のテーブルまたは複数の結合テーブルに対する集約クエリです。

  **注記**
  >
  > - クエリキャッシュはBroadcast JoinとBucket Shuffle Joinをサポートしています。
  > - クエリキャッシュは、Join演算子を含む2つのツリー構造をサポートしています：Aggregation-JoinとJoin-Aggregation。Aggregation-Joinツリー構造ではShuffle Joinはサポートされておらず、Join-Aggregationツリー構造ではHash Joinはサポートされていません。

- クエリには、`rand`、`random`、`uuid`、`sleep`などの非決定論的関数は含まれていません。

クエリキャッシュは、以下のパーティションポリシーを使用するテーブルに対するクエリをサポートしています：未パーティション、マルチカラムパーティション、シングルカラムパーティション。

## 機能の限界

- クエリキャッシュは、Pipelineエンジンのタブレットごとの計算に基づいています。タブレットごとの計算とは、パイプラインドライバーがタブレットの一部分や多数のタブレットを交互に処理するのではなく、一つずつタブレット全体を処理できることを意味します。クエリに対して各BEが処理する必要があるタブレットの数が、このクエリを実行するために呼び出されるパイプラインドライバーの数以上であれば、クエリキャッシュは機能します。呼び出されるパイプラインドライバーの数は、実際の並列度（DOP）を表します。タブレットの数がパイプラインドライバーの数より少ない場合、各パイプラインドライバーは特定のタブレットの一部分のみを処理します。この場合、タブレットごとの計算結果を生成できず、したがってクエリキャッシュは機能しません。
- StarRocksでは、集約クエリは少なくとも4つのステージで構成されます。第1ステージでAggregateNodeによって生成されたタブレットごとの計算結果は、OlapScanNodeとAggregateNodeが同じフラグメントからデータを計算する場合にのみキャッシュ可能です。他のステージでAggregateNodeによって生成されたタブレットごとの計算結果はキャッシュできません。一部のDISTINCT集約クエリでは、セッション変数 `cbo_cte_reuse` が `true` に設定されている場合、データを生成するOlapScanNodeと、生成されたデータを消費する第1ステージのAggregateNodeが異なるフラグメントからデータを計算し、ExchangeNodeによってブリッジされると、クエリキャッシュは機能しません。次の2つの例は、CTE最適化が実行され、クエリキャッシュが機能しないシナリオを示しています：
  - 出力列は、集約関数 `avg(distinct)` を使用して計算されます。
  - 出力列は、複数のDISTINCT集約関数によって計算されます。
- 集約前にデータがシャッフルされる場合、クエリキャッシュはそのデータに対するクエリを加速できません。
- テーブルのグループ化列や重複排除列が高カーディナリティ列である場合、そのテーブルに対する集約クエリで大量の結果が生成されます。このような場合、クエリは実行時にクエリキャッシュをバイパスします。
- クエリキャッシュは、計算結果を保存するためにBEから提供される少量のメモリを使用します。クエリキャッシュのサイズはデフォルトで512MBです。したがって、大きなサイズのデータ項目を保存するにはクエリキャッシュは適していません。さらに、クエリキャッシュを有効にした後、キャッシュヒット率が低い場合はクエリのパフォーマンスが低下します。そのため、タブレットに対して生成された計算結果のサイズが `query_cache_entry_max_bytes` または `query_cache_entry_max_rows` パラメータで指定されたしきい値を超える場合、クエリキャッシュはそのクエリに対して機能しなくなり、クエリはパススルーモードに切り替わります。

## 仕組み

クエリキャッシュが有効になっている場合、各BEはクエリのローカル集約を以下の2つのステージに分割します：

1. タブレットごとの集約

   BEは各タブレットを個別に処理します。タブレットの処理を開始すると、BEはまずクエリキャッシュを調べて、そのタブレットの集約の中間結果がクエリキャッシュに存在するかどうかを確認します。存在する場合（キャッシュヒット）、BEはクエリキャッシュから中間結果を直接取得します。存在しない場合（キャッシュミス）、BEはディスク上のデータにアクセスし、ローカル集約を実行して中間結果を計算します。タブレットの処理が終了すると、BEはそのタブレットの集約の中間結果をクエリキャッシュに格納します。

2. タブレット間の集約

   BEは、クエリに関連するすべてのタブレットから中間結果を収集し、それらを最終結果に統合します。

   ![クエリキャッシュ - 仕組み - 1](../assets/query_cache_principle-1.png)

将来、同様のクエリが発行されると、以前のクエリのキャッシュされた結果を再利用できます。例えば、次の図に示すクエリには3つのタブレット（タブレット0から2）が含まれており、最初のタブレット（タブレット0）の中間結果は既にクエリキャッシュにあります。この例では、BEはディスク上のデータにアクセスする代わりに、クエリキャッシュからタブレット0の結果を直接取得できます。クエリキャッシュが完全にウォームアップされている場合、3つのタブレットすべての中間結果を含むことができ、そのためBEはディスク上のデータにアクセスする必要がありません。

![クエリキャッシュ - 仕組み - 2](../assets/query_cache_principle-2.png)

余分なメモリを解放するために、クエリキャッシュは最近最も使用されていない(LRU: Least Recently Used)ベースの削除ポリシーを採用して、キャッシュエントリを管理します。この削除ポリシーに従って、クエリキャッシュが占有するメモリの量が事前定義されたサイズ(`query_cache_capacity`)を超えると、最も長く使用されていないキャッシュエントリがクエリキャッシュから削除されます。

> **注記**
>
> 将来的には、StarRocksはTTL(Time to Live)ベースの削除ポリシーもサポートする予定です。これにより、クエリキャッシュからキャッシュエントリが削除されるようになります。

FEは、クエリキャッシュを使用して各クエリを高速化する必要があるかどうかを判断し、クエリのセマンティクスに影響を与えない些細なリテラルの詳細を排除してクエリを正規化します。

クエリキャッシュの不良ケースによって発生するパフォーマンスの低下を防ぐために、BEは実行時にクエリキャッシュをバイパスする適応ポリシーを採用します。

## クエリキャッシュを有効にする

このセクションでは、クエリキャッシュを有効にして設定するために使用されるパラメータとセッション変数について説明します。

### FEセッション変数

| **変数**                | **デフォルト値** | **動的に設定可能** | **説明**                                              |
| --------------------------- | ----------------- | --------------------------------- | ------------------------------------------------------------ |
| enable_query_cache          | false             | はい                               | クエリキャッシュを有効にするかどうかを指定します。有効な値は `true` と `false` です。`true` はこの機能を有効にし、`false` は無効にします。クエリキャッシュが有効の場合、このトピックの「[アプリケーションシナリオ](../using_starrocks/query_cache.md#application-scenarios)」セクションに指定された条件を満たすクエリに対してのみ機能します。 |
| query_cache_entry_max_bytes | 4194304           | はい                               | パススルーモードをトリガーするしきい値を指定します。有効な値は `0` から `9223372036854775807` です。クエリがアクセスする特定のタブレットからの計算結果のバイト数または行数が `query_cache_entry_max_bytes` または `query_cache_entry_max_rows` パラメータで指定されたしきい値を超えると、クエリはパススルーモードに切り替わります。<br />`query_cache_entry_max_bytes` または `query_cache_entry_max_rows` パラメータが `0` に設定されている場合、関連するタブレットから計算結果が生成されない場合でもパススルーモードが使用されます。 |
| query_cache_entry_max_rows  | 409600            | はい                               | 上記と同じです。                                                |

### BEパラメーター

次のパラメータはBEの設定ファイル **be.conf** で設定する必要があります。BEでこのパラメータを再設定した後、新しいパラメータ設定を有効にするためにBEを再起動する必要があります。

| **パラメータ**        | **必須** | **説明**                                              |
| -------------------- | ------------ | ------------------------------------------------------------ |
| query_cache_capacity | いいえ           | クエリキャッシュのサイズを指定します。単位はバイトです。デフォルトサイズは512MBです。<br />各BEはメモリ内に自身のローカルクエリキャッシュを持ち、自身のクエリキャッシュのみを操作します。<br />クエリキャッシュのサイズは4MB未満に設定することはできません。BEのメモリ容量が期待されるクエリキャッシュサイズを確保するには不十分な場合、BEのメモリ容量を増やすことができます。 |

## すべてのシナリオで最大のキャッシュヒット率を実現するために設計されています

クエリが文字通り同一でない場合でも、クエリキャッシュが有効である3つのシナリオを考えてみましょう。これらのシナリオは以下の通りです。

- 意味的に等価なクエリ
- スキャンされたパーティションが重複するクエリ
- 追記のみのデータ変更（UPDATEやDELETE操作なし）があるデータに対するクエリ

### 意味的に等価なクエリ

2つのクエリが類似している場合、それらが文字通り同一である必要はなく、実行プランに意味的に等価なスニペットが含まれている場合、それらは意味的に等価であると見なされ、互いの計算結果を再利用することができます。広義には、2つのクエリが同じソースからデータをクエリし、同じ計算方法を使用し、同じ実行プランを持つ場合、それらは意味的に等価です。StarRocksは、2つのクエリが意味的に等価かどうかを評価するために以下のルールを適用します。

- 2つのクエリに複数の集約が含まれている場合、最初の集約が意味的に等価である限り、それらは意味的に等価であると評価されます。例えば、以下の2つのクエリQ1とQ2には複数の集約が含まれていますが、最初の集約が意味的に等価です。したがって、Q1とQ2は意味的に等価であると評価されます。

  - Q1

    ```SQL
    SELECT
        (
            ifnull(sum(murmur_hash3_32(hour)), 0) + ifnull(sum(murmur_hash3_32(k0)), 0) + ifnull(sum(murmur_hash3_32(__c_0)), 0)
        ) AS fingerprint
    FROM
        (
            SELECT
                date_trunc('hour', ts) AS hour,
                k0,
                sum(v1) AS __c_0
            FROM
                t0
            WHERE
                ts BETWEEN '2022-01-03 00:00:00'
                AND '2022-01-03 23:59:59'
            GROUP BY
                date_trunc('hour', ts),
                k0
        ) AS t;
    ```

  - Q2

    ```SQL
    SELECT
        date_trunc('hour', ts) AS hour,
        k0,
        sum(v1) AS __c_0
    FROM
        t0
    WHERE
        ts BETWEEN '2022-01-03 00:00:00'
        AND '2022-01-03 23:59:59'
    GROUP BY
        date_trunc('hour', ts),
        k0
    ```

- 2つのクエリが以下のいずれかのクエリタイプに属している場合、それらは意味的に等価であると評価できます。HAVING句を含むクエリは、HAVING句を含まないクエリと意味的に等価であるとは評価されません。ただし、ORDER BY句やLIMIT句の有無は、2つのクエリが意味的に等価であるかどうかの評価に影響しません。

  - GROUP BY集約

    ```SQL
    SELECT <GroupByItems>, <AggFunctionItems> 
    FROM <Table> 
    WHERE <Predicates> [AND <PartitionColumnRangePredicate>]
    GROUP BY <GroupByItems>
    [HAVING <HavingPredicate>] 
    ```

    > **注記**
    >
    > 上記の例では、HAVING句はオプションです。

  - GROUP BY DISTINCT集約

    ```SQL
    SELECT DISTINCT <GroupByItems>, <Items> 
    FROM <Table> 
    WHERE <Predicates> [AND <PartitionColumnRangePredicate>]
    GROUP BY <GroupByItems>
    [HAVING <HavingPredicate>]
    ```

    > **注記**
    >
    > 上記の例では、HAVING句はオプションです。

  - GROUP BYなしの集約

    ```SQL
    SELECT <AggFunctionItems> FROM <Table> 
    WHERE <Predicates> [AND <PartitionColumnRangePredicate>]
    ```

  - GROUP BY DISTINCTなしの集約

    ```SQL
    SELECT DISTINCT <Items> FROM <Table> 
    WHERE <Predicates> [AND <PartitionColumnRangePredicate>]
    ```

- いずれかのクエリに`PartitionColumnRangePredicate`が含まれている場合、2つのクエリが意味的に等価であるかを評価する前に`PartitionColumnRangePredicate`は除去されます。`PartitionColumnRangePredicate`は、パーティション列を参照する以下のタイプの述語を指定します：

  - `col BETWEEN v1 AND v2`：パーティション列の値が[v1, v2]の範囲内にある場合、ここで`v1`と`v2`は定数式です。
  - `v1 < col AND col < v2`：パーティション列の値が(v1, v2)の範囲内にある場合、ここで`v1`と`v2`は定数式です。
  - `v1 < col AND col <= v2`：パーティション列の値が(v1, v2]の範囲内にある場合、ここで`v1`と`v2`は定数式です。
  - `v1 <= col AND col < v2`：パーティション列の値が[v1, v2)の範囲内にある場合、ここで`v1`と`v2`は定数式です。
  - `v1 <= col AND col <= v2`：パーティション列の値が[v1, v2]の範囲内にある場合、ここで`v1`と`v2`は定数式です。

- 2つのクエリのSELECT句の出力列が再配置後に同じである場合、2つのクエリは意味的に等価であると評価されます。

- 2つのクエリのGROUP BY句の出力列が再配置後に同じである場合、2つのクエリは意味的に等価であると評価されます。

- `PartitionColumnRangePredicate`を除去した後、2つのクエリのWHERE句の残りの述語が意味的に等価である場合、2つのクエリは意味的に等価であると評価されます。

- 2つのクエリのHAVING句の述語が意味的に等しい場合、2つのクエリは意味的に等価であると評価されます。

例として、次の表 `lineorder_flat` を使用します：

```SQL
CREATE TABLE `lineorder_flat`
(
    `lo_orderdate` date NOT NULL COMMENT "",
    `lo_orderkey` int(11) NOT NULL COMMENT "",
    `lo_linenumber` tinyint(4) NOT NULL COMMENT "",
    `lo_custkey` int(11) NOT NULL COMMENT "",
    `lo_partkey` int(11) NOT NULL COMMENT "",
    `lo_suppkey` int(11) NOT NULL COMMENT "",
    `lo_orderpriority` varchar(100) NOT NULL COMMENT "",
    `lo_shippriority` tinyint(4) NOT NULL COMMENT "",
    `lo_quantity` tinyint(4) NOT NULL COMMENT "",
    `lo_extendedprice` int(11) NOT NULL COMMENT "",
    `lo_ordtotalprice` int(11) NOT NULL COMMENT "",
    `lo_discount` tinyint(4) NOT NULL COMMENT "",
    `lo_revenue` int(11) NOT NULL COMMENT "",
    `lo_supplycost` int(11) NOT NULL COMMENT "",
    `lo_tax` tinyint(4) NOT NULL COMMENT "",
    `lo_commitdate` date NOT NULL COMMENT "",
    `lo_shipmode` varchar(100) NOT NULL COMMENT "",
    `c_name` varchar(100) NOT NULL COMMENT "",
    `c_address` varchar(100) NOT NULL COMMENT "",
    `c_city` varchar(100) NOT NULL COMMENT "",
    `c_nation` varchar(100) NOT NULL COMMENT "",
    `c_region` varchar(100) NOT NULL COMMENT "",
    `c_phone` varchar(100) NOT NULL COMMENT "",
    `c_mktsegment` varchar(100) NOT NULL COMMENT "",
    `s_name` varchar(100) NOT NULL COMMENT "",
    `s_address` varchar(100) NOT NULL COMMENT "",
    `s_city` varchar(100) NOT NULL COMMENT "",
    `s_nation` varchar(100) NOT NULL COMMENT "",
    `s_region` varchar(100) NOT NULL COMMENT "",
    `s_phone` varchar(100) NOT NULL COMMENT "",
    `p_name` varchar(100) NOT NULL COMMENT "",
    `p_mfgr` varchar(100) NOT NULL COMMENT "",
    `p_category` varchar(100) NOT NULL COMMENT "",
    `p_brand` varchar(100) NOT NULL COMMENT "",
    `p_color` varchar(100) NOT NULL COMMENT "",
    `p_type` varchar(100) NOT NULL COMMENT "",
    `p_size` tinyint(4) NOT NULL COMMENT "",
    `p_container` varchar(100) NOT NULL COMMENT ""
)
ENGINE=OLAP 
DUPLICATE KEY(`lo_orderdate`, `lo_orderkey`)
COMMENT "olap"
PARTITION BY RANGE(`lo_orderdate`)
(PARTITION p1 VALUES [('0000-01-01'), ('1993-01-01')),
PARTITION p2 VALUES [('1993-01-01'), ('1994-01-01')),
PARTITION p3 VALUES [('1994-01-01'), ('1995-01-01')),
PARTITION p4 VALUES [('1995-01-01'), ('1996-01-01')),
PARTITION p5 VALUES [('1996-01-01'), ('1997-01-01')),
PARTITION p6 VALUES [('1997-01-01'), ('1998-01-01')),
PARTITION p7 VALUES [('1998-01-01'), ('1999-01-01']))
DISTRIBUTED BY HASH(`lo_orderkey`)
PROPERTIES 
(
    "replication_num" = "1",
    "colocate_with" = "groupxx1",
    "storage_format" = "DEFAULT",
    "enable_persistent_index" = "false",
    "compression" = "LZ4"
);
```

テーブル `lineorder_flat` に対する次の2つのクエリ、Q1とQ2は、以下のように処理された後、意味的に等価です：

1. SELECT文の出力列を並べ替えます。
2. GROUP BY句の出力列を並べ替えます。
3. ORDER BY句の出力列を削除します。
4. WHERE句の述語を並べ替えます。
5. `PartitionColumnRangePredicate`を追加します。

- Q1

  ```SQL
  SELECT sum(lo_revenue), year(lo_orderdate) AS year, p_brand
  FROM lineorder_flat
  WHERE p_category = 'MFGR#12' AND s_region = 'AMERICA'
  GROUP BY year, p_brand
  ORDER BY year, p_brand;
  ```

- Q2

  ```SQL
  SELECT year(lo_orderdate) AS year, p_brand, sum(lo_revenue)
  FROM lineorder_flat
  WHERE s_region = 'AMERICA' AND p_category = 'MFGR#12' AND 
     lo_orderdate >= '1993-01-01' AND lo_orderdate <= '1993-12-31'
  GROUP BY p_brand, year(lo_orderdate)
  ```

セマンティック等価性はクエリの物理プランに基づいて評価されるため、クエリのリテラルの違いはセマンティック等価性の評価に影響しません。さらに、クエリからは定数式が削除され、クエリ最適化中に`cast`式も削除されます。したがって、これらの式はセマンティック等価性の評価に影響しません。また、列やリレーションのエイリアスもセマンティック等価性の評価には影響しません。

### スキャンされたパーティションが重複するクエリ

クエリキャッシュは述語ベースのクエリ分割をサポートしています。

述語のセマンティクスに基づいてクエリを分割することで、部分計算結果の再利用を実現します。クエリにテーブルのパーティショニングカラムを参照する述語が含まれ、その述語が値の範囲を指定している場合、StarRocksはテーブルのパーティショニングに基づいて範囲を複数の間隔に分割できます。各個別の間隔からの計算結果は、他のクエリで個別に再利用することができます。

例として、次の表 `t0` を使用します：

```SQL
CREATE TABLE if not exists t0
(
    ts DATETIME NOT NULL,
    k0 VARCHAR(10) NOT NULL,
    k1 BIGINT NOT NULL,
    v1 DECIMAL64(7, 2) NOT NULL 
)
ENGINE=OLAP
DUPLICATE KEY(`ts`, `k0`, `k1`)
COMMENT "OLAP"
PARTITION BY RANGE(ts)
(
  START ("2022-01-01 00:00:00") END ("2022-02-01 00:00:00") EVERY (INTERVAL 1 day) 
)
DISTRIBUTED BY HASH(`ts`, `k0`, `k1`)
PROPERTIES
(
    "replication_num" = "1", 
    "storage_format" = "default"
);
```

テーブル `t0` は日ごとにパーティション分割され、カラム `ts` はテーブルのパーティション分割カラムです。以下の4つのクエリのうち、Q2、Q3、およびQ4は、Q1のキャッシュされた計算結果の一部を再利用できます：

- Q1

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v1)
  FROM t0
  WHERE ts BETWEEN '2022-01-02 12:30:00' AND '2022-01-14 23:59:59'
  GROUP BY day;
  ```

  Q1の述語 `ts BETWEEN '2022-01-02 12:30:00' AND '2022-01-14 23:59:59'` によって指定された値の範囲は、以下の間隔に分割できます：

  ```SQL
  1. [2022-01-02 12:30:00, 2022-01-03 00:00:00),
  2. [2022-01-03 00:00:00, 2022-01-04 00:00:00),
  3. [2022-01-04 00:00:00, 2022-01-05 00:00:00),
  ...
  12. [2022-01-13 00:00:00, 2022-01-14 00:00:00),
  13. [2022-01-14 00:00:00, 2022-01-15 00:00:00),
  ```

- Q2

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v1)
  FROM t0
  WHERE ts >= '2022-01-02 12:30:00' AND ts < '2022-01-05 00:00:00'
  GROUP BY day;
  ```

  Q2は、Q1の以下の間隔内の計算結果を再利用できます：

  ```SQL
  1. [2022-01-02 12:30:00, 2022-01-03 00:00:00),
  2. [2022-01-03 00:00:00, 2022-01-04 00:00:00),
  3. [2022-01-04 00:00:00, 2022-01-05 00:00:00),
  ```

- Q3

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v1)
  FROM t0
  WHERE ts >= '2022-01-01 12:30:00' AND ts <= '2022-01-10 12:00:00'
  GROUP BY day;
  ```

  Q3は、Q1の以下の間隔内の計算結果を再利用できます：

  ```SQL
  2. [2022-01-03 00:00:00, 2022-01-04 00:00:00),
  3. [2022-01-04 00:00:00, 2022-01-05 00:00:00),
  ...
  8. [2022-01-09 00:00:00, 2022-01-10 00:00:00),
  ```

- Q4

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v1)
  FROM t0
  WHERE ts BETWEEN '2022-01-02 12:30:00' AND '2022-01-02 23:59:59'
  GROUP BY day;
  ```

  Q4は、Q1の以下の間隔内の計算結果を再利用できます：

  ```SQL
  1. [2022-01-02 12:30:00, 2022-01-03 00:00:00),
  ```

部分計算結果の再利用のサポートは、使用するパーティションポリシーによって異なります。以下の表に説明されています。

| **パーティションポリシー**   | **部分計算結果の再利用のサポート**         |
| ------------------------- | ------------------------------------------------------------ |
| パーティションなし             | サポートされていません                                                |
| 複数列パーティション  | サポートされていません<br />**注**<br />この機能は将来サポートされる可能性があります。 |
| 単一列パーティション | サポートされています                                                    |

### 追記のみのデータ変更を伴うデータに対するクエリ

クエリキャッシュはマルチバージョンキャッシュをサポートしています。

データロードが行われると、新しいバージョンのタブレットが生成されます。結果として、以前のバージョンのタブレットから生成されたキャッシュされた計算結果は古くなり、最新のタブレットバージョンに遅れをとります。この状況では、マルチバージョンキャッシュメカニズムは、クエリキャッシュに保存された古い結果と、ディスク上に保存されたタブレットのインクリメンタルバージョンをマージし、タブレットの最終結果を生成することで、新しいクエリが最新のタブレットバージョンを取り扱えるように試みます。マルチバージョンキャッシュは、テーブルタイプ、クエリタイプ、データ更新タイプによって制約されます。

マルチバージョンキャッシュのサポートは、テーブルタイプとクエリタイプによって異なり、以下の表で説明されています。

| **テーブルタイプ**       | **クエリタイプ**                                           | **マルチバージョンキャッシュのサポート**                        |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Duplicate Key テーブル | <ul><li>ベーステーブルに対するクエリ</li><li>同期マテリアライズドビューに対するクエリ</li></ul> | <ul><li>ベーステーブルに対するクエリ: インクリメンタルタブレットバージョンにデータ削除レコードが含まれていない場合、すべての状況でサポートされます。</li><li>同期マテリアライズドビューに対するクエリ: クエリのGROUP BY、HAVING、またはWHERE句が集約列を参照している場合を除き、すべての状況でサポートされます。</li></ul> |
| Aggregate テーブル | ベーステーブルに対するクエリまたは同期マテリアライズドビューに対するクエリ | 以下の状況を除き、すべての状況でサポートされます：ベーステーブルのスキーマに集約関数 `replace` が含まれている。クエリのGROUP BY、HAVING、またはWHERE句が集約列を参照している。インクリメンタルタブレットバージョンにデータ削除レコードが含まれている。 |
| Unique Key テーブル    | N/A                                                          | サポートされていませんが、クエリキャッシュはサポートされています。        |
| Primary Key テーブル   | N/A                                                          | サポートされていませんが、クエリキャッシュはサポートされています。        |

データ更新タイプがマルチバージョンキャッシュに与える影響は以下の通りです：

- データ削除

  タブレットのインクリメンタルバージョンに削除操作が含まれている場合、マルチバージョンキャッシュは機能しません。

- データ挿入

  - タブレットに空のバージョンが生成された場合、クエリキャッシュ内のタブレットの既存データは有効のままで、引き続き取得可能です。
  - タブレットに非空のバージョンが生成された場合、クエリキャッシュ内のタブレットの既存データは有効ですが、そのバージョンはタブレットの最新バージョンに遅れます。この場合、StarRocksは既存データのバージョンから最新バージョンのタブレットまでのインクリメンタルデータを読み込み、既存データとインクリメンタルデータをマージし、マージされたデータをクエリキャッシュに格納します。

- スキーマ変更とタブレット切り捨て

  テーブルのスキーマが変更されるか、テーブルの特定のタブレットが切り捨てられると、テーブルの新しいタブレットが生成されます。結果として、クエリキャッシュ内のテーブルのタブレットの既存データは無効になります。

## メトリクス

クエリキャッシュが機能するクエリのプロファイルには `CacheOperator` 統計が含まれています。

クエリのソースプランで、パイプラインに `OlapScanOperator` が含まれている場合、`OlapScanOperator` と集約演算子の名前には `ML_` プレフィックスが付けられ、パイプラインが `MultilaneOperator` を使用してタブレットごとの計算を実行することを示します。`CacheOperator` は `ML_CONJUGATE_AGGREGATE` の前に挿入され、Passthrough、Populate、Probeモードでクエリキャッシュがどのように動作するかを制御するロジックを処理します。クエリのプロファイルには、クエリキャッシュの使用状況を理解するのに役立つ以下の `CacheOperator` メトリクスが含まれています。

| **メトリック**                | **説明**                                              |
| ------------------------- | ------------------------------------------------------------ |
| CachePassthroughBytes     | Passthroughモードで生成されるバイト数。           |
| CachePassthroughChunkNum  | Passthroughモードで生成されるチャンク数。          |
| CachePassthroughRowNum    | Passthroughモードで生成される行数。            |
| CachePassthroughTabletNum | Passthroughモードで生成されるタブレット数。         |
| CachePassthroughTime      | Passthroughモードでの計算時間。    |
| CachePopulateBytes        | Populateモードで生成されるバイト数。              |
| CachePopulateChunkNum     | Populateモードで生成されるチャンク数。             |
| CachePopulateRowNum       | Populateモードで生成される行数。               |
| CachePopulateTabletNum    | Populateモードで生成されるタブレット数。            |
| CachePopulateTime         | Populateモードでの計算時間。       |
| CacheProbeBytes           | Probeモードでキャッシュヒット時に生成されるバイト数。  |
| CacheProbeChunkNum        | Probeモードでキャッシュヒット時に生成されるチャンク数。 |
| CacheProbeRowNum          | Probeモードでキャッシュヒット時に生成される行数。   |
| CacheProbeTabletNum       | Probeモードでキャッシュヒット時に生成されるタブレット数。 |
| CacheProbeTime            | Probeモードでの計算時間。          |

`CachePopulate`*`XXX`* メトリクスは、クエリキャッシュが更新されるクエリキャッシュミスに関する統計情報を提供します。

`CachePassthrough`*`XXX`* メトリクスは、生成されるタブレットごとの計算結果のサイズが大きいためにクエリキャッシュが更新されないクエリキャッシュミスに関する統計情報を提供します。

`CacheProbe`*`XXX`* メトリクスは、クエリキャッシュヒットに関する統計情報を提供します。

マルチバージョンキャッシュメカニズムでは、`CachePopulate` メトリクスと `CacheProbe` メトリクスが同じタブレット統計を含むことがあり、`CachePassthrough` メトリクスと `CacheProbe` メトリクスも同じタブレット統計を含むことがあります。例えば、StarRocksが各タブレットのデータを計算する際に、タブレットの歴史的バージョンで生成された計算結果にヒットした場合、StarRocksは歴史的バージョンから最新バージョンのタブレットまでのインクリメンタルデータを読み取り、データを計算し、インクリメンタルデータをキャッシュデータとマージします。マージ後に生成された計算結果のサイズが `query_cache_entry_max_bytes` または `query_cache_entry_max_rows` パラメータで指定された閾値を超えない場合、タブレットの統計は `CachePopulate` メトリクスに集計されます。それ以外の場合、タブレットの統計は `CachePassthrough` メトリクスに集計されます。

## RESTful API 操作

- `metrics | grep query_cache`

  このAPI操作は、クエリキャッシュに関連するメトリクスを取得するために使用されます。

  ```shell
  curl -s  http://<be_host>:<be_http_port>/metrics | grep query_cache
  
  # TYPE starrocks_be_query_cache_capacity gauge
  starrocks_be_query_cache_capacity 536870912
  # TYPE starrocks_be_query_cache_hit_count gauge
  starrocks_be_query_cache_hit_count 5084393
  # TYPE starrocks_be_query_cache_hit_ratio gauge
  starrocks_be_query_cache_hit_ratio 0.984098
  # TYPE starrocks_be_query_cache_lookup_count gauge
  starrocks_be_query_cache_lookup_count 5166553
  # TYPE starrocks_be_query_cache_usage gauge
  starrocks_be_query_cache_usage 0
  # TYPE starrocks_be_query_cache_usage_ratio gauge
  starrocks_be_query_cache_usage_ratio 0.000000
  ```

- `api/query_cache/stat`

  このAPI操作は、クエリキャッシュの使用状況を照会するために使用されます。

  ```shell
  curl  http://<be_host>:<be_http_port>/api/query_cache/stat
  {
      "capacity": 536870912,
      "usage": 0,
      "usage_ratio": 0.0,
      "lookup_count": 5025124,
      "hit_count": 4943720,
      "hit_ratio": 0.983800598751394
  }
  ```

- `api/query_cache/invalidate_all`

  このAPI操作は、クエリキャッシュをクリアするために使用されます。

  ```shell
  curl -X PUT http://<be_host>:<be_http_port>/api/query_cache/invalidate_all
  
  {
      "status": "OK"
  }
  ```

上記のAPI操作のパラメータは次のとおりです：

- `be_host`: BEが稼働しているノードのIPアドレス。
- `be_http_port`: BEが稼働しているノードのHTTPポート番号。

## 注意事項

- StarRocksは、初めて実行されるクエリの計算結果をクエリキャッシュに格納する必要があります。その結果、クエリのパフォーマンスが予想よりも若干低下し、クエリのレイテンシが増加する可能性があります。
- 大きなクエリキャッシュサイズを設定すると、BEでのクエリ評価に割り当てられるメモリ量が減少します。クエリキャッシュサイズは、クエリ評価に割り当てられたメモリ容量の1/6を超えないことを推奨します。
- 処理する必要があるタブレットの数が`pipeline_dop`の値よりも少ない場合、クエリキャッシュは機能しません。クエリキャッシュを有効にするためには、`pipeline_dop`を`1`のような小さな値に設定できます。v3.0以降、StarRocksはクエリの並列性に基づいてこのパラメータを適応的に調整します。

## 例

### データセット

1. StarRocksクラスタにログインし、目的のデータベースに移動して、次のコマンドを実行し、`t0`という名前のテーブルを作成します：

   ```SQL
   CREATE TABLE t0
   (
         `ts` datetime NOT NULL COMMENT "",
         `k0` varchar(10) NOT NULL COMMENT "",
         `k1` char(6) NOT NULL COMMENT "",
         `v0` bigint(20) NOT NULL COMMENT "",
         `v1` decimal64(7, 2) NOT NULL COMMENT ""
   )
   ENGINE=OLAP 
   DUPLICATE KEY(`ts`, `k0`, `k1`)
   COMMENT "OLAP"
   PARTITION BY RANGE(`ts`)
   (
       START ("2022-01-01 00:00:00") END ("2022-02-01 00:00:00") EVERY (INTERVAL 1 DAY)
   )
   DISTRIBUTED BY HASH(`ts`, `k0`, `k1`) 
   PROPERTIES
   (
       "replication_num" = "1",
       "storage_format" = "DEFAULT",
       "enable_persistent_index" = "false"
   );
   ```

2. 次のデータレコードを`t0`に挿入します：

   ```SQL
   INSERT INTO t0
   VALUES
       ('2022-01-11 20:42:26', 'n4AGcEqYp', 'hhbawx', '799393174109549', '8029.42'),
       ('2022-01-27 18:17:59', 'i66lt', 'mtrtzf', '100400167', '10000.88'),
       ('2022-01-28 20:10:44', 'z6', 'oqkeun', '-58681382337', '59881.87'),
       ('2022-01-29 14:54:31', 'qQ', 'dzytua', '-19682006834', '43807.02'),
       ('2022-01-31 08:08:11', 'qQ', 'dzytua', '7970665929984223925', '-8947.74'),
       ('2022-01-15 00:40:58', '65', 'hhbawx', '4054945', '156.56'),
       ('2022-01-24 16:17:51', 'onqR3JsK1', 'udtmfp', '-12962', '-72127.53'),
       ('2022-01-01 22:36:24', 'n4AGcEqYp', 'fabnct', '-50999821', '17349.85'),
       ('2022-01-21 08:41:50', 'Nlpz1j3h', 'dzytua', '-60162', '287.06'),
       ('2022-01-30 23:44:55', '', 'slfght', '62891747919627339', '8014.98'),
       ('2022-01-18 19:14:28', 'z6', 'dzytua', '-1113001726', '73258.24'),
       ('2022-01-30 14:54:59', 'z6', 'udtmfp', '111175577438857975', '-15280.41'),
       ('2022-01-08 22:08:26', 'z6', 'ympyls', '3', '2.07'),
       ('2022-01-03 08:17:29', 'Nlpz1j3h', 'udtmfp', '-234492', '217.58'),
       ('2022-01-27 07:28:47', 'Pc', 'cawanm', '-1015', '-20631.50'),
       ('2022-01-17 14:07:47', 'Nlpz1j3h', 'lbsvqu', '2295574006197343179', '93768.75'),
       ('2022-01-31 14:00:12', 'onqR3JsK1', 'umlkpo', '-227', '-66199.05'),
       ('2022-01-05 20:31:26', '65', 'lbsvqu', '684307', '36412.49'),
       ('2022-01-06 00:51:34', 'z6', 'dzytua', '11700309310', '-26064.10'),
       ('2022-01-26 02:59:00', 'n4AGcEqYp', 'slfght', '-15320250288446', '-58003.69'),
       ('2022-01-05 03:26:26', 'z6', 'cawanm', '19841055192960542', '-5634.36'),
       ('2022-01-17 08:51:23', 'Pc', 'ghftus', '35476438804110', '13625.99'),
       ('2022-01-30 18:56:03', 'n4AGcEqYp', 'lbsvqu', '3303892099598', '8.37'),
       ('2022-01-22 14:17:18', 'i66lt', 'umlkpo', '-27653110', '-82306.25'),
       ('2022-01-02 10:25:01', 'qQ', 'ghftus', '-188567166', '71442.87'),
       ('2022-01-30 04:58:14', 'Pc', 'ympyls', '-9983', '-82071.59'),
       ('2022-01-05 00:16:56', '7Bh', 'hhbawx', '43712', '84762.97'),
       ('2022-01-25 03:25:53', '65', 'mtrtzf', '4604107', '-2434.69'),
       ('2022-01-27 21:09:10', '65', 'udtmfp', '476134823953365199', '38736.04'),
       ('2022-01-11 13:35:44', '65', 'qmwhvr', '1', '0.28'),
       ('2022-01-03 19:13:07', '', 'lbsvqu', '11', '-53084.04'),
       ('2022-01-20 02:27:25', 'i66lt', 'umlkpo', '3218824416', '-71393.20'),
       ('2022-01-04 04:52:36', '7Bh', 'ghftus', '-112543071', '-78377.93'),
       ('2022-01-27 18:27:06', 'Pc', 'umlkpo', '477', '-98060.13'),
       ('2022-01-04 19:40:36', '', 'udtmfp', '433677211', '-99829.94'),
       ('2022-01-20 23:19:58', 'Nlpz1j3h', 'udtmfp', '361394977', '-19284.18'),
       ('2022-01-05 02:17:56', 'Pc', 'oqkeun', '-552390906075744662', '-25267.92'),
       ('2022-01-02 16:14:07', '65', 'dzytua', '132', '2393.77'),
       ('2022-01-28 23:17:14', 'z6', 'umlkpo', '61', '-52028.57'),
       ('2022-01-12 08:05:44', 'qQ', 'hhbawx', '-9579605666539132', '-87801.81'),
       ('2022-01-31 19:48:22', 'z6', 'lbsvqu', '9883530877822', '34006.42'),
       ('2022-01-11 20:38:41', '', 'piszhr', '56108215256366', '-74059.80'),
       ('2022-01-01 04:15:17', '65', 'cawanm', '-440061829443010909', '88960.51'),
       ('2022-01-05 07:26:09', 'qQ', 'umlkpo', '-24889917494681901', '-23372.12'),
       ('2022-01-29 18:13:55', 'Nlpz1j3h', 'cawanm', '-233', '-24294.42'),
       ('2022-01-10 00:49:45', 'Nlpz1j3h', 'ympyls', '-2396341', '77723.88'),
       ('2022-01-29 08:02:58', 'n4AGcEqYp', 'slfght', '45212', '93099.78'),
       ('2022-01-28 08:59:21', 'onqR3JsK1', 'oqkeun', '76', '-78641.65'),
       ('2022-01-26 14:29:39', '7Bh', 'umlkpo', '176003552517', '-99999.96'),
       ('2022-01-03 18:53:37', '7Bh', 'piszhr', '3906151622605106', '55723.01'),
       ('2022-01-04 07:08:19', 'i66lt', 'ympyls', '-240097380835621', '-81800.87'),
       ('2022-01-28 14:54:17', 'Nlpz1j3h', 'slfght', '-69018069110121', '90533.64'),
       ('2022-01-22 07:48:53', 'Pc', 'ympyls', '22396835447981344', '-12583.39'),
       ('2022-01-22 07:39:29', 'Pc', 'uqkghp', '10551305', '52163.82'),
       ('2022-01-08 22:39:47', 'Nlpz1j3h', 'cawanm', '67905472699', '87831.30'),
       ('2022-01-05 14:53:34', '7Bh', 'dzytua', '-779598598706906834', '-38780.41'),
       ('2022-01-30 17:34:41', 'onqR3JsK1', 'oqkeun', '346687625005524', '-62475.31'),
       ('2022-01-29 12:14:06', '', 'qmwhvr', '3315', '22076.88'),
       ('2022-01-05 06:47:04', 'Nlpz1j3h', 'udtmfp', '-469', '42747.17'),
       ('2022-01-19 15:20:20', '7Bh', 'lbsvqu', '347317095885', '-76393.49'),
       ('2022-01-08 16:18:22', 'z6', 'fghmcd', '2', '90315.60'),
       ('2022-01-02 00:23:06', 'Pc', 'piszhr', '-3651517384168400', '58220.34'),
       ('2022-01-12 08:23:31', 'onqR3JsK1', 'udtmfp', '5636394870355729225', '33224.25'),
       ('2022-01-28 10:46:44', 'onqR3JsK1', 'oqkeun', '-28102078612755', '6469.53'),
       ('2022-01-23 23:16:11', 'onqR3JsK1', 'ghftus', '-707475035515433949', '63422.66'),
       ('2022-01-03 05:32:31', 'z6', 'hhbawx', '-45', '-49680.52'),
       ('2022-01-27 03:24:33', 'qQ', 'qmwhvr', '375943906057539870', '-66092.96'),
       ('2022-01-25 20:07:22', '7Bh', 'slfght', '1', '72440.21'),
       ('2022-01-04 16:07:24', 'qQ', 'uqkghp', '751213107482249', '16417.31'),
       ('2022-01-23 19:22:00', 'Pc', 'hhbawx', '-740731249600493', '88439.40'),
       ('2022-01-05 09:04:20', '7Bh', 'cawanm', '23602', '302.44');
   ```

### クエリ例

このセクションのクエリキャッシュ関連メトリクスの統計は例であり、参考用のみです。

#### ステージ1のローカル集計でクエリキャッシュが機能する

これには3つの状況が含まれます：

- クエリが単一のタブレットにのみアクセスする。
- クエリが、自体がコロケートされたグループを構成するテーブルの複数のパーティションから複数のタブレットにアクセスし、集計のためにデータをシャッフルする必要がない。
- クエリがテーブルの同じパーティションから複数のタブレットにアクセスし、集計のためにデータをシャッフルする必要がない。

クエリ例：

```SQL
SELECT
    date_trunc('hour', ts) AS hour,
    k0,
    sum(v1) AS __c_0
FROM
    t0
WHERE
    ts BETWEEN '2022-01-03 00:00:00'
    AND '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    k0
```

次の図は、クエリプロファイルのクエリキャッシュ関連メトリクスを示しています。

![クエリキャッシュ - ステージ1 - メトリクス](../assets/query_cache_stage1_agg_with_cache_en.png)

#### ステージ1のリモート集計でクエリキャッシュが機能しない

複数のタブレットにわたる集計がステージ1で強制的に実行される場合、データは最初にシャッフルされ、その後集計されます。

クエリ例：

```SQL
SET new_planner_agg_stage = 1;

SELECT
    date_trunc('hour', ts) AS hour,
    v0 % 2 AS is_odd,
    sum(v1) AS __c_0
FROM
    t0
WHERE
    ts BETWEEN '2022-01-03 00:00:00'
    AND '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    is_odd
```

#### ステージ2のローカル集計でクエリキャッシュが機能する

これには3つの状況が含まれます：

- クエリのステージ2の集計は、同じタイプのデータを比較するためにコンパイルされます。最初の集計はローカル集計です。最初の集計が完了すると、その結果が計算され、2番目の集計（グローバル集計）が実行されます。
- クエリがSELECT DISTINCTクエリである。
- クエリに次のいずれかのDISTINCT集計関数が含まれている：`sum(distinct)`, `count(distinct)`, `avg(distinct)`。通常、このようなクエリの集計はステージ3または4で実行されます。しかし、`set new_planner_agg_stage = 1`を実行してクエリのステージ2で集計を強制的に実行することができます。クエリに`avg(distinct)`が含まれていて、ステージで集計を実行したい場合は、`set cbo_cte_reuse = false`を実行してCTE最適化を無効にする必要があります。

クエリ例：

```SQL
SELECT
    date_trunc('hour', ts) AS hour,
    v0 % 2 AS is_odd,
    sum(v1) AS __c_0
FROM
    t0
WHERE
    ts BETWEEN '2022-01-03 00:00:00'
    AND '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    is_odd
```

次の図は、クエリプロファイルのクエリキャッシュ関連メトリクスを示しています。

![クエリキャッシュ - ステージ2 - メトリクス](../assets/query_cache_stage2_agg_with_cache_en.png)

#### ステージ3のローカル集計でクエリキャッシュが機能する

クエリは、単一のDISTINCT集計関数を含むGROUP BY集計クエリです。

サポートされているDISTINCT集計関数は`sum(distinct)`, `count(distinct)`, `avg(distinct)`です。

> **注意**
>
> クエリに`avg(distinct)`が含まれる場合、`set cbo_cte_reuse = false`を実行してCTE最適化を無効にする必要があります。

クエリ例：

```SQL
SELECT
    date_trunc('hour', ts) AS hour,
    v0 % 2 AS is_odd,
    sum(distinct v1) AS __c_0
FROM
    t0
WHERE
    ts BETWEEN '2022-01-03 00:00:00'
    AND '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    is_odd;
```

次の図は、クエリプロファイルのクエリキャッシュ関連メトリクスを示しています。

![クエリキャッシュ - ステージ3 - メトリクス](../assets/query_cache_stage3_agg_with_cache_en.png)

#### ステージ4のローカル集計でクエリキャッシュが機能する

クエリは、単一のDISTINCT集計関数を含む非GROUP BY集計クエリです。このようなクエリには、デデュプリケートデータを除去するクラシックなクエリが含まれます。

クエリ例：

```SQL
SELECT
    count(distinct v1) AS __c_0
FROM
    t0
WHERE
    ts BETWEEN '2022-01-03 00:00:00'
    AND '2022-01-03 23:59:59'
```

次の図は、クエリプロファイルのクエリキャッシュ関連メトリクスを示しています。

![クエリキャッシュ - ステージ4 - メトリクス](../assets/query_cache_stage4_agg_with_cache_en.png)

#### 最初の集計が意味的に同等である2つのクエリでキャッシュされた結果が再利用される

例として、Q1とQ2の2つのクエリを使用します。Q1とQ2は両方とも複数の集計を含んでいますが、最初の集計は意味的に等価です。したがって、Q1とQ2は意味的に等価と評価され、クエリキャッシュに保存された互いの計算結果を再利用することができます。

- Q1

  ```SQL
  SELECT
      (
          ifnull(sum(murmur_hash3_32(hour)), 0) + ifnull(sum(murmur_hash3_32(k0)), 0) + ifnull(sum(murmur_hash3_32(__c_0)), 0)
        ) AS fingerprint
  FROM
      (
          SELECT
              date_trunc('hour', ts) AS hour,
              k0,
              sum(v1) AS __c_0
          FROM
              t0
          WHERE
              ts BETWEEN '2022-01-03 00:00:00'
              AND '2022-01-03 23:59:59'
          GROUP BY
              date_trunc('hour', ts),
              k0
      ) AS t;
  ```

- Q2

  ```SQL
  SELECT
      date_trunc('hour', ts) AS hour,
      k0,
      sum(v1) AS __c_0
  FROM
      t0
  WHERE
      ts BETWEEN '2022-01-03 00:00:00'
      AND '2022-01-03 23:59:59'
  GROUP BY
      date_trunc('hour', ts),
      k0
  ```

次の図は、Q1の`CachePopulate`メトリクスを示しています。

![クエリキャッシュ - Q1 - メトリクス](../assets/query_cache_reuse_Q1_en.png)

次の図は、Q2の`CacheProbe`メトリクスを示しています。

![クエリキャッシュ - Q2 - メトリクス](../assets/query_cache_reuse_Q2_en.png)

#### CTE最適化が有効なDISTINCTクエリではクエリキャッシュが機能しない

`set cbo_cte_reuse = true`を実行してCTE最適化を有効にした後、DISTINCT集計関数を含む特定のクエリの計算結果はキャッシュされません。以下にいくつかの例を挙げます。

- クエリに単一のDISTINCT集計関数`avg(distinct)`が含まれている場合：

  ```SQL
  SELECT
      avg(distinct v1) AS __c_0
  FROM
      t0
  WHERE
      ts BETWEEN '2022-01-03 00:00:00'
      AND '2022-01-03 23:59:59';
  ```

![クエリキャッシュ - CTE - 1](../assets/query_cache_distinct_with_cte_Q1_en.png)

- 同じ列を参照する複数のDISTINCT集計関数を含むクエリ：

  ```SQL
  SELECT
      avg(distinct v1) AS __c_0,
      sum(distinct v1) AS __c_1,
      count(distinct v1) AS __c_2
  FROM
      t0
  WHERE
      ts BETWEEN '2022-01-03 00:00:00'
      AND '2022-01-03 23:59:59';
  ```

![クエリキャッシュ - CTE - 2](../assets/query_cache_distinct_with_cte_Q2_en.png)

- それぞれ異なる列を参照する複数のDISTINCT集計関数を含むクエリ：

  ```SQL
  SELECT
      sum(distinct v1) AS __c_1,
      count(distinct v0) AS __c_2
  FROM
      t0
  WHERE
      ts BETWEEN '2022-01-03 00:00:00'
      AND '2022-01-03 23:59:59';
  ```

![クエリキャッシュ - CTE - 3](../assets/query_cache_distinct_with_cte_Q3_en.png)

## ベストプラクティス


テーブルを作成する際には、合理的なパーティションの説明と適切な分散方法を指定してください。

- パーティション列としてDATE型の単一の列を選択します。テーブルにDATE型の列が複数含まれている場合、データが増分的に取り込まれる際に値が進行し、クエリの興味深い時間範囲を定義するのに使用される列を選びます。
- 適切なパーティション幅を選択してください。最近取り込まれたデータはテーブルの最新のパーティションを変更する可能性があります。そのため、最新のパーティションに関連するキャッシュエントリは不安定で、無効化されやすいです。
- テーブル作成ステートメントの分散記述で、数十のバケット番号を指定します。バケット数が過度に少ない場合、BEが処理する必要があるタブレットの数が`pipeline_dop`の値よりも少ないと、クエリキャッシュは効果を発揮しません。
