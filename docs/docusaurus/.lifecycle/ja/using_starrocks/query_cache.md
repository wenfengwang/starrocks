---
displayed_sidebar: "Japanese"
---

# クエリキャッシュ

クエリキャッシュは、StarRocksの強力な機能であり、集計クエリのパフォーマンスを大幅に向上させることができます。クエリキャッシュは、中間結果をメモリに格納することで、以前と同じまたは類似した新しいクエリの不要なディスクアクセスと計算を回避することができます。クエリキャッシュにより、StarRocksは集計クエリの高速かつ正確な結果を提供し、時間とリソースを節約し、スケーラビリティを向上させることができます。クエリキャッシュは、多くのユーザーが大規模で複雑なデータセットで類似のクエリを実行する高並行のシナリオに特に有用です。

この機能はバージョン2.5からサポートされています。

バージョン2.5では、クエリキャッシュは単一のフラットテーブルでの集計クエリのみをサポートしています。バージョン3.0以降、クエリキャッシュはスタースキーマで結合された複数のテーブルでの集計クエリもサポートしています。

## 適用シナリオ

次のシナリオでクエリキャッシュを使用することをおすすめします。

- 個々のフラットテーブルまたはスタースキーマで結合された複数のテーブルで頻繁に集計クエリを実行します。
- ほとんどの集計クエリがGROUP BYを使用しない集計クエリまたは低基数のGROUP BYを使用した集計クエリである場合。
- データが時間によるパーティションで追加モードでロードされ、アクセス頻度に基づいてホットデータとコールドデータに分類できる場合。

クエリキャッシュは以下の条件を満たすクエリをサポートしています。

- クエリエンジンがPipelineであること。Pipelineエンジンを有効にするには、セッション変数`enable_pipeline_engine`を`true`に設定します。

  > **注記**
  >
  > 他のクエリエンジンはクエリキャッシュをサポートしていません。

- クエリがネイティブOLAPテーブル（v2.5以降）またはクラウドネイティブテーブル（v3.0以降）上の集計クエリであること。クエリキャッシュは外部テーブルに対するクエリをサポートしていません。クエリキャッシュは同期物理ビューへのアクセスが必要なクエリのプランもサポートしていますが、非同期物理ビューへのアクセスが必要なクエリのプランはサポートしていません。

- クエリが個々のテーブルまたは結合された複数のテーブル上の集計クエリであること。

  **注記**
  >
  > - クエリキャッシュはBroadcast JoinとBucket Shuffle Joinをサポートしています。
  > - クエリキャッシュはJoin演算子を含む2つのツリー構造をサポートしています: 集約-結合および結合-集約。集約-結合ツリー構造ではShuffle Joinはサポートされておらず、結合-集約ツリー構造ではHash Joinはサポートされていません。

- クエリに`rand`、`random`、`uuid`、`sleep`などの非決定的な関数が含まれていないこと。

クエリキャッシュは、以下のパーティションポリシーを使用するテーブル上のクエリをサポートしています: パーティションされていないテーブル、複数列のパーティションを持つテーブル、および単一列のパーティションを持つテーブル。

## 機能境界

- クエリキャッシュは、パイプラインエンジンのテーブルごとの計算に基づいています。テーブルごとの計算は、パイプラインドライバが各テーブルを一度に処理することを意味します。つまり、個々のBEがクエリのために処理する必要があるテーブルの数が、このクエリを実行するために起動されるパイプラインドライバの数以上であれば、クエリキャッシュは機能します。起動されるパイプラインドライバの数は実際の並列性（DOP）を表します。テーブルの数がパイプラインドライバの数よりも少ない場合は、各パイプラインドライバが特定のテーブルの一部のみを処理することになります。この状況では、テーブルごとの計算結果を生成することができず、したがってクエリキャッシュは機能しません。
- StarRocksでは、集計クエリは少なくとも４つのステージで構成されます。最初のステージのAggregateNodeによって生成されるテーブルごとの計算結果は、OlapScanNodeとAggregateNodeが同じフラグメントからデータを処理する場合のみキャッシュされます。他のステージのAggregateNodeによって生成されるテーブルごとの計算結果はキャッシュすることができません。一部のDISTINCT集計クエリでは、セッション変数`cbo_cte_reuse`を`true`に設定すると、OlapScanNode（データを生成する）とステージ1のAggregateNode（生成されたデータを消費する）が異なるフラグメントからデータを計算し、ExchangeNodeで接続されている場合、クエリキャッシュは機能しません。次の２つの例はCTEの最適化が行われる場合であり、クエリキャッシュは機能しない場合です:
  - 出力の列は`avg(distinct)`という集計関数を使用して計算されます。
  - 出力の列は複数のDISTINCT集計関数によって計算されます。
- データが集計前にシャッフルされている場合、クエリキャッシュではそのデータのクエリを高速化することはできません。
- テーブルのGROUP BY列または重複排除列が高基数の列である場合、そのテーブル上の集計クエリに対して大量の結果が生成されます。これらの場合、クエリは実行時にクエリキャッシュをバイパスします。
- クエリキャッシュはBEが提供する少量のメモリを占有します。クエリキャッシュのサイズはデフォルトで512 MBです。そのため、大きなサイズのデータ項目をクエリキャッシュに保存するのは適していません。また、クエリキャッシュを有効にした後、キャッシュヒット率が低い場合はクエリのパフォーマンスが低下します。そのため、特定のテーブルの計算結果のサイズが`query_cache_entry_max_bytes`または`query_cache_entry_max_rows`の指定された閾値を超える場合、クエリキャッシュはクエリに対して機能せず、クエリはPassthroughモードに切り替わります。

## 動作仕組み

クエリキャッシュが有効になっている場合、各BEはクエリのローカル集約を以下の２つのステージに分割します：

1. テーブルごとの集計

   BEは各テーブルを個別に処理します。BEがテーブルの処理を開始すると、まずクエリキャッシュをプローブしてそのテーブルの集約の中間結果がクエリキャッシュに存在するかどうかを確認します。存在する場合（キャッシュヒット）は、BEはクエリキャッシュから中間結果を直接取得します。存在しない場合（キャッシュミス）は、BEはディスク上のデータにアクセスし、ローカル集計を実行して中間結果を計算します。BEがテーブルの処理を完了したら、そのテーブルの集計の中間結果でクエリキャッシュを更新します。

2. テーブル間の集計

   BEはクエリに関与するすべてのテーブルの中間結果を収集し、最終結果にマージします。

   ![クエリキャッシュ - 動作仕組み - 1](../assets/query_cache_principle-1.png)

将来、同様のクエリが発行された場合、以前のクエリのキャッシュされた結果を再利用することができます。例えば、次の図に示すクエリでは3つのテーブル（テーブル0から2）が関与しており、最初のテーブル（テーブル0）の中間結果が既にクエリキャッシュに含まれています。この場合、BEはディスク上のデータにアクセスする代わりに、テーブル0の結果をクエリキャッシュから直接取得することができます。クエリキャッシュが完全にウォームアップされている場合、すべての３つのテーブルの中間結果がクエリキャッシュに含まれていますので、BEはディスク上のデータにアクセスする必要はありません。

![クエリキャッシュ - 動作仕組み - 2](../assets/query_cache_principle-2.png)

余分なメモリを解放するために、クエリキャッシュは最も最近使用された（LRU）ベースのエビクションポリシーを採用してキャッシュエントリを管理します。このエビクションポリシーにより、クエリキャッシュが事前定義されたサイズ（`query_cache_capacity`）を超えるメモリを占有する場合は、最も最近使用されていないキャッシュエントリがクエリキャッシュからエビクトされます。

> **注記**
>
> 将来、StarRocksはクエリキャッシュからキャッシュエントリをエビクトするためのTime to Live（TTL）ベースのエビクションポリシーもサポートする予定です。

FEは、各クエリがクエリキャッシュを使用して加速する必要があるかどうかを決定し、クエリのセマンティクスに影響を与えないトリビアルなリテラルの詳細を正規化します。

クエリキャッシュの悪影響を防ぐために、BEは実行時にクエリキャッシュをバイパスするための適応ポリシーを採用しています。

## クエリキャッシュの有効化

このセクションでは、クエリキャッシュを有効にし、構成するために使用されるパラメータおよびセッション変数について説明します。

### FEセッション変数

| **変数**                       | **デフォルト値** | **ダイナミックに構成可能** | **説明**                                                     |
| ------------------------------ | ----------------- | ------------------------ | ------------------------------------------------------------ |
| enable_query_cache              | false             | はい                     | クエリキャッシュを有効にするかどうかを指定します。有効な値はtrueおよびfalseです。`true`はこの機能を有効にし、`false`はこの機能を無効にします。クエリキャッシュは、このトピックの"[適用シナリオ](../using_starrocks/query_cache.md#適用シナリオ)"セクションで指定された条件を満たすクエリにのみ適用されます。 |
| query_cache_entry_max_bytes     | 4194304           | はい                     | Passthroughモードをトリガーするための閾値を指定します。有効な値は`0`から`9223372036854775807`までです。クエリが特定のテーブルの計算結果からアクセスするバイト数または行数が、`query_cache_entry_max_bytes`または`query_cache_entry_max_rows`パラメータで指定された閾値を超える場合、クエリはPassthroughモードに切り替わります。<br />`query_cache_entry_max_bytes`または`query_cache_entry_max_rows`パラメータが`0`に設定されている場合、関与するテーブルから計算結果が生成されていない場合でも、Passthroughモードが使用されます。 |
| query_cache_entry_max_rows      | 409600            | はい                     | 上記と同じ。                                                 |

### BEパラメータ

BEの設定ファイル**be.conf**で以下のパラメータを設定する必要があります。BEのパラメータを再構成した後は、BEを再起動して新しいパラメータ設定を有効にする必要があります。

| **パラメータ**        | **必須** | **説明**                                                     |
| --------------------- | -------- | ------------------------------------------------------------ |
| query_cache_capacity | なし | クエリキャッシュのサイズを指定します。単位：バイト。デフォルトサイズは512 MB です。<br />各BEには独自のローカルクエリキャッシュがあり、自分自身のクエリキャッシュのみを展開およびプローブします。<br />なお、クエリキャッシュのサイズは4 MB 未満にすることはできません。BE のメモリ容量が期待されるクエリキャッシュサイズを供給するのに不十分な場合は、BE のメモリ容量を増やすことができます。 |

## あらゆるシナリオで最大のキャッシュヒット率にエンジニアリングされています

リテラルに完全一致しない場合でも、クエリキャッシュが効果的である三つのシナリオを考えてみましょう。これらの三つのシナリオは次のとおりです。

- セマンティック的に等価なクエリ
- オーバーラップするスキャンされたパーティションを持つクエリ
- UPDATE や DELETE 操作のないデータに対するクエリ

### セマンティック的に等価なクエリ

2つのクエリが類似している場合、文字通り等価である必要はありませんが、実行計画で意味的に等価なスニペットを含んでいる場合、それらはセマンティック的に等価と見なされ、お互いの計算結果を再利用できます。広義には、2つのクエリが同じソースからデータをクエリし、同じ計算手法を使用し、同じ実行計画を持つ場合、それらはセマンティック的に等価です。StarRocks は、次のルールを適用して2つのクエリがセマンティック的に等価であるかどうかを評価します。

- 2つのクエリが複数の集計を含んでいる場合、その最初の集計が意味的に等価である限り、それらはセマンティック的に等価として評価されます。たとえば、次の2つのクエリ Q1 および Q2 は、複数の集計を含んでいますが、最初の集計が意味的に等価なため、Q1 と Q2 はセマンティック的に等価と評価されます。

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
                ts between '2022-01-03 00:00:00'
                and '2022-01-03 23:59:59'
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
        ts between '2022-01-03 00:00:00'
        and '2022-01-03 23:59:59'
    GROUP BY
        date_trunc('hour', ts),
        k0
    ```

- 2つのクエリが以下のクエリ種類のいずれかに属している場合、それらはセマンティック的に等価として評価できます。HAVING 句を含むクエリは HAVING 句を含まないクエリとセマンティック的に等価と評価されない点に注意してください。ただし、ORDER BY 句または LIMIT 句の含まれていることは、2つのクエリがセマンティック的に等価であるかどうかの評価に影響しません。

  - GROUP BY 集計

    ```SQL
    SELECT <GroupByItems>, <AggFunctionItems> 
    FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    GROUP BY <GroupByItems>
    [HAVING <HavingPredicate>] 
    ```

    > **注記**
    >
    > 上記の例では、HAVING 句はオプションです。

  - GROUP BY DISTINCT 集計

    ```SQL
    SELECT DISTINCT <GroupByItems>, <Items> 
    FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    GROUP BY <GroupByItems>
    HAVING <HavingPredicate>
    ```

    > **注記**
    >
    > 上記の例では、HAVING 句はオプションです。

  - GROUP BY を含まない集計

    ```SQL
    SELECT <AggFunctionItems> FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    ```

  - GROUP BY DISTINCT を含まない集計

    ```SQL
    SELECT DISTINCT <Items> FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    ```

- いずれかのクエリが `PartitionColumnRangePredicate` を含む場合、`PartitionColumnRangePredicate` は、2つのクエリがセマンティック的に等価かどうかを評価する前に削除されます。`PartitionColumnRangePredicate` は、パーティション列を参照する以下の種類の述語を指定します。

  - `col between v1 and v2`: パーティション列の値は [v1, v2] の範囲にあり、ただし `v1` および `v2` は定数式です。
  - `v1 < col and col < v2`: パーティション列の値は (v1, v2) の範囲にあり、ただし `v1` および `v2` は定数式です。
  - `v1 < col and col <= v2`: パーティション列の値は (v1, v2] の範囲にあり、ただし `v1` および `v2` は定数式です。
  - `v1 <= col and col < v2`: パーティション列の値は [v1, v2) の範囲にあり、ただし `v1` および `v2` は定数式です。
  - `v1 <= col and col <= v2`: パーティション列の値は [v1, v2] の範囲にあり、ただし `v1` および `v2` は定数式です。

- 2つのクエリの SELECT 句の出力列が並べ替えられた後も同じである場合、2つのクエリはセマンティック的に等価と評価されます。

- 2つのクエリの GROUP BY 句の出力列が並べ替えられた後も同じである場合、2つのクエリはセマンティック的に等価と評価されます。

- 2つのクエリの WHERE 句の残りの述語が `PartitionColumnRangePredicate` を削除した後も、意味的に等価である場合、2つのクエリはセマンティック的に等価と評価されます。

- 2つのクエリの HAVING 句の述語が意味的に等価である場合、2つのクエリはセマンティック的に等価と評価されます。

次の `lineorder_flat` テーブルを例として使用します。

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
```
PARTITION p5 VALUES [('1996-01-01'), ('1997-01-01')),
PARTITION p6 VALUES [('1997-01-01'), ('1998-01-01')),
PARTITION p7 VALUES [('1998-01-01'), ('1999-01-01')))
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

テーブル`lineorder_flat`上の次の2つのクエリ、Q1とQ2は、以下の処理後に意味的に等価です：

1. SELECT文の出力列を再配置します。
2. GROUP BY句の出力列を再配置します。
3. ORDER BY句の出力列を削除します。
4. WHERE句の述語を再配置します。
5. `PartitionColumnRangePredicate`を追加します。

- Q1

  ```SQL
  SELECT sum(lo_revenue)), year(lo_orderdate) AS year,p_brand
  FROM lineorder_flat
  WHERE p_category = 'MFGR#12' AND s_region = 'AMERICA'
  GROUP BY year,p_brand
  ORDER BY year,p_brand;
  ```

- Q2

  ```SQL
  SELECT year(lo_orderdate) AS year, p_brand, sum(lo_revenue))
  FROM lineorder_flat
  WHERE s_region = 'AMERICA' AND p_category = 'MFGR#12' AND 
     lo_orderdate >= '1993-01-01' AND lo_orderdate <= '1993-12-31'
  GROUP BY p_brand, year(lo_orderdate)
  ```

意味的な等価性は、クエリの物理的な計画に基づいて評価されます。そのため、クエリのリテラルの違いは、意味的な等価性の評価には影響しません。さらに、クエリの中の定数式は削除され、クエリの最適化中に`cast`式も削除されます。そのため、これらの式は意味的な等価性の評価に影響しません。第三に、列とリレーションのエイリアスも意味的な等価性の評価に影響しません。

### オーバーラップするスキャンされたパーティションを持つクエリ

クエリキャッシュは、述語ベースのクエリの分割をサポートします。

述語の意味論に基づいてクエリを分割することで、部分的な計算結果の再利用を実装するのに役立ちます。クエリがテーブルのパーティション列を参照し、述語が値の範囲を指定する場合、StarRocksはテーブルの分割に基づいて範囲を複数の間隔に分割できます。個々の間隔からの計算結果は、他のクエリによって別々に再利用されることができます。

次の` t0`テーブルを例にします。

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

テーブル`t0`は日付でパーティション分割されており、列`ts`がテーブルのパーティション分割列です。次の4つのクエリのうち、Q2、Q3、Q4はQ1に対してキャッシュされた計算結果の一部を再利用できます。

- Q1

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v0)
  FROM t0
  WHERE ts BETWEEN '2022-01-02 12:30:00' AND '2022-01-14 23:59:59'
  GROUP BY day;
  ```

  Q1の述語`ts between '2022-01-02 12:30:00' and '2022-01-14 23:59:59'`で指定された値範囲は、以下の間隔に分割されます：

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
  SELECT date_trunc('day', ts) as day, sum(v0)
  FROM t0
  WHERE ts >= '2022-01-02 12:30:00' AND  ts < '2022-01-05 00:00:00'
  GROUP BY day;
  ```

  Q2は、Q1の次の間隔内で計算結果を再利用できます：

  ```SQL
  1. [2022-01-02 12:30:00, 2022-01-03 00:00:00),
  2. [2022-01-03 00:00:00, 2022-01-04 00:00:00),
  3. [2022-01-04 00:00:00, 2022-01-05 00:00:00),
  ```

- Q3

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v0)
  FROM t0
  WHERE ts >= '2022-01-01 12:30:00' AND  ts <= '2022-01-10 12:00:00'
  GROUP BY day;
  ```

  Q3は、Q1の次の間隔内で計算結果を再利用できます：

  ```SQL
  2. [2022-01-03 00:00:00, 2022-01-04 00:00:00),
  3. [2022-01-04 00:00:00, 2022-01-05 00:00:00),
  ...
  8. [2022-01-09 00:00:00, 2022-01-10 00:00:00),
  ```

- Q4

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v0)
  FROM t0
  WHERE ts BETWEEN '2022-01-02 12:30:00' and '2022-01-02 23:59:59'
  GROUP BY day;
  ```

  Q4は、Q1の次の間隔内で計算結果を再利用できます：

  ```SQL
  1. [2022-01-02 12:30:00, 2022-01-03 00:00:00),
  ```

部分的な計算結果の再利用のサポートは、使用される分割ポリシーによって異なります。詳細については次のテーブルを参照してください。

| **パーティショニングポリシー**   | **部分的な計算結果の再利用のサポート**         |
| ------------------------- | ------------------------------------------------------------ |
| パーティションされていない             | サポートされていない                                                |
| 複数列パーティショニング              | サポートされていない<br />**注意**<br />この機能は将来サポートされる可能性があります。 |
| 単一列パーティショニング | サポートされています                                                    |

### アペンド専用データの変更を含むデータに対するクエリ

クエリキャッシュは、マルチバージョンキャッシングをサポートします。

データのロードが行われると、新しいバージョンのタブレットが生成されます。その結果、クエリキャッシュに保存された前のタブレットバージョンから生成された古い計算結果は陳腐化し、最新のタブレットバージョンよりも遅れることになります。この状況下で、マルチバージョンキャッシングメカニズムは、クエリキャッシュに保存された陳腐化した結果とディスク上に保存されたタブレットの増分バージョンを最終結果にマージし、新しいクエリが最新のタブレットバージョンを持つようにします。マルチバージョンキャッシングは、テーブルのタイプ、クエリのタイプ、およびデータの更新のタイプによって制約されます。

マルチバージョンキャッシングのサポートは、テーブルのタイプとクエリのタイプによって異なります。詳細については次のテーブルを参照してください。

| **テーブルタイプ**       | **クエリタイプ**                                           | **マルチバージョンキャッシングのサポート**                        |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 重複キータブル | <ul><li>ベーステーブル上のクエリ</li><li>同期マテリアライズドビュー上のクエリ</li></ul> | <ul><li>ベーステーブル上のクエリ：増分タブレットバージョンにデータ削除レコードが含まれていない限り、すべての状況でサポートされます。</li><li>同期マテリアライズドビュー上のクエリ：クエリのGROUP BY、HAVING、またはWHERE句が集計列を参照している場合を除き、すべての状況でサポートされます。</li></ul> |
| 集計テーブル | ベーステーブル上のクエリまたは同期マテリアライズドビュー上のクエリ | 基本テーブルのスキーマに集計関数`replace`が含まれる増分タブレットバージョンにデータ削除レコードが含まれていない限り、すべての状況でサポートされます。|
| タイプテーブル | N/A | サポートされていません。ただし、クエリキャッシュはサポートされています。 |
| プライマリーキーテーブル | N/A | サポートされていません。ただし、クエリキャッシュはサポートされています。 |

マルチバージョンキャッシングへのデータ更新タイプの影響は次のとおりです。

- データの削除

  インクリメンタルなタブレットのバージョンに削除操作が含まれている場合、マルチバージョンキャッシングは動作しません。

- データの挿入

  - タブレットに空のバージョンが生成された場合、クエリキャッシュ内のタブレットの既存データは有効のままで、データを引き出すことができます。
  - タブレットに空ではないバージョンが生成された場合、クエリキャッシュ内のタブレットの既存データは有効のままですが、そのバージョンはタブレットの最新バージョンに遅れています。この場合、StarRocksは既存データから新規バージョンのタブレットの最新バージョンまでのインクリメンタルデータを読み込み、既存データとインクリメンタルデータをマージしてマージデータをクエリキャッシュに展開します。

- スキーマの変更およびタブレットの切り捨て

  テーブルのスキーマが変更されたり、テーブルの特定のタブレットが切り捨てられた場合、テーブルのタブレットの既存データはクエリキャッシュ内で無効になります。

## メトリクス

クエリキャッシュが動作するクエリのプロファイルには、`CacheOperator`統計情報が含まれます。

クエリのソースプランにおいて、パイプラインに`OlapScanOperator`が含まれる場合、`OlapScanOperator`および集約演算子の名前は、パイプラインが`MultilaneOperator`を使用してタブレットごとの演算を実行することを示すために、`ML_`が接頭語として付けられます。`CacheOperator`は`ML_CONJUGATE_AGGREGATE`の前に挿入され、Passthrough、Populate、およびProbeモードでクエリキャッシュの実行を制御するロジックを処理します。クエリのプロファイルには、クエリキャッシュの使用状況を理解するのに役立つ`CacheOperator`メトリクスが含まれています。

| **メトリクス**            | **説明**                                                     |
| ------------------------- | ------------------------------------------------------------ |
| CachePassthroughBytes     | Passthroughモードで生成されるバイト数。                   |
| CachePassthroughChunkNum  | Passthroughモードで生成されるチャンク数。                  |
| CachePassthroughRowNum    | Passthroughモードで生成される行数。                        |
| CachePassthroughTabletNum | Passthroughモードで生成されるタブレット数。                |
| CachePassthroughTime:     | Passthroughモードでかかる演算時間。                        |
| CachePopulateBytes        | Populateモードで生成されるバイト数。                       |
| CachePopulateChunkNum     | Populateモードで生成されるチャンク数。                     |
| CachePopulateRowNum       | Populateモードで生成される行数。                           |
| CachePopulateTabletNum    | Populateモードで生成されるタブレット数。                   |
| CachePopulateTime         | Populateモードでかかる演算時間。                           |
| CacheProbeBytes           | Probeモードでのキャッシュヒットによって生成されるバイト数。|
| CacheProbeChunkNum        | Probeモードでのキャッシュヒットによって生成されるチャンク数。|
| CacheProbeRowNum          | Probeモードでのキャッシュヒットによって生成される行数。  |
| CacheProbeTabletNum       | Probeモードでのキャッシュヒットによって生成されるタブレット数。 |
| CacheProbeTime            | Probeモードでかかる演算時間。                              |

`CachePopulate`*`XXX`*メトリクスは、クエリキャッシュが更新されるクエリキャッシュミスに関する統計情報を提供します。

`CachePassthrough`*`XXX`*メトリクスは、パイプラインごとの演算結果のサイズが大きいためにクエリキャッシュが更新されないクエリキャッシュミスに関する統計情報を提供します。

`CacheProbe`*`XXX`*メトリクスは、クエリキャッシュヒットに関する統計情報を提供します。

マルチバージョンキャッシングメカニズムでは、`CachePopulate`メトリクスおよび`CacheProbe`メトリクスに同じタブレットの統計情報が含まれることがあります。同様に、`CachePassthrough`メトリクスおよび`CacheProbe`メトリクスにも同じタブレットの統計情報が含まれることがあります。例えば、StarRocksが各タブレットのデータを計算する際、過去のバージョンのタブレット上で生成された計算結果を利用する場合があります。この場合、StarRocksは過去のバージョンから最新バージョンのタブレットまでのインクリメンタルデータを読み込み、データを計算し、インクリメンタルデータをキャッシュデータにマージします。このマージ後に生成された計算結果のサイズが`query_cache_entry_max_bytes`パラメータあるいは`query_cache_entry_max_rows`パラメータで指定されたしきい値を超えない場合、タブレットの統計情報は`CachePopulate`メトリクスに集約されます。それ以外の場合、タブレットの統計情報は`CachePassthrough`メトリクスに集約されます。

## RESTful API操作

- `metrics |grep query_cache`

  このAPI操作は、クエリキャッシュに関連するメトリクスをクエリするために使用されます。

  ```shell
  curl -s  http://<be_host>:<be_http_port>/metrics |grep query_cache
  
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

  このAPI操作は、クエリキャッシュの使用状況をクエリするために使用されます。

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
  curl  -XPUT http://<be_host>:<be_http_port>/api/query_cache/invalidate_all
  
  {
      "status": "OK"
  }
  ```

前述のAPI操作のパラメータは次のとおりです。

- `be_host`: BEが存在するノードのIPアドレス。
- `be_http_port`: BEが存在するノードのHTTPポート番号。

## 注意事項

- StarRocksは初回にイニシエートされるクエリの計算結果でクエリキャッシュを展開する必要があります。その結果、クエリのパフォーマンスが予想よりもわずかに低下し、クエリのレイテンシが増加する可能性があります。
- クエリキャッシュサイズを大きく設定すると、BEでのクエリ評価に供給できるメモリ量が減少します。クエリキャッシュのサイズは、クエリ評価に供給されるメモリ容量の1/6を超えないようにすることをお勧めします。
- 処理する必要があるタブレットの数が`pipeline_dop`の値よりも小さい場合、クエリキャッシュは動作しません。クエリキャッシュを有効にするには、`pipeline_dop`を`1`などの小さい値に設定することができます。v3.0以降、StarRocksはこのパラメータをクエリの並列性に基づいて適応的に調整します。

## 例

### データセット

1. StarRocksクラスタにログインし、対象のデータベースに移動し、次のコマンドを実行して`t0`という名前のテーブルを作成します。

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

2. 以下のデータレコードを`t0`に挿入します。

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
```
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

### クエリの例

このセクションのクエリキャッシュ関連メトリクスの統計情報は例であり、参考情報のみです。

#### ステージ1のローカル集計用のクエリキャッシュ

次の3つの状況が含まれます。

- クエリが単一のタブレットにのみアクセスする場合。
- クエリがテーブルの共有グループを構成し、データの集計にシャッフルが不要なテーブルの複数のパーティションから複数のタブレットにアクセスする場合。
- クエリがテーブルの同じパーティションから複数のタブレットにアクセスし、データの集計にシャッフルが不要な場合。

クエリの例:

```SQL
SELECT
    date_trunc('hour', ts) AS hour,
    k0,
    sum(v1) AS __c_0
FROM
    t0
WHERE
    ts between '2022-01-03 00:00:00'
    and '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    k0
```

次の図は、クエリプロファイルのステージ1のクエリキャッシュ関連メトリクスを示しています。

![クエリキャッシュ - ステージ1 - メトリクス](../assets/query_cache_stage1_agg_with_cache_en.png)

#### ステージ1のリモート集計用のクエリキャッシュ

複数のタブレットに対する集計がステージ1で強制的に実行される場合、データはまずシャッフルされ、その後に集計が行われます。

クエリの例:

```SQL
SET new_planner_agg_stage = 1;

SELECT
    date_trunc('hour', ts) AS hour,
    v0 % 2 AS is_odd,
    sum(v1) AS __c_0
FROM
    t0
WHERE
    ts between '2022-01-03 00:00:00'
    and '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    is_odd
```

#### ステージ2のローカル集計用のクエリキャッシュ

次の3つの状況が含まれます。

- クエリのステージ2での集計が同じタイプのデータを比較するようにコンパイルされる場合。 最初の集計はローカル集計です。最初の集計が完了すると、最初の集計から生成された結果が計算され、2番目の集計（グローバル集計）が実行されます。
- クエリがSELECT DISTINCTクエリである場合。
- クエリが次のDISTINCT集約関数のいずれかを含む場合: `sum(distinct)`, `count(distinct)`, `avg(distinct)`。 ほとんどの場合、このようなクエリの集計はステージ3または4で実行されます。 ただし、クエリの集計を強制的にステージ2で実行するには、`set new_planner_agg_stage = 1`を実行する必要があります。 クエリに`avg(distinct)`が含まれており、ステージで集計を実行したい場合は、`set cbo_cte_reuse = false`を実行してCTEの最適化を無効にする必要があります。

クエリの例:

```SQL
SELECT
    date_trunc('hour', ts) AS hour,
    v0 % 2 AS is_odd,
    sum(v1) AS __c_0
FROM
    t0
WHERE
    ts between '2022-01-03 00:00:00'
    and '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    is_odd
```

次の図は、クエリプロファイルのステージ2のクエリキャッシュ関連メトリクスを示しています。

![クエリキャッシュ - ステージ2 - メトリクス](../assets/query_cache_stage2_agg_with_cache_en.png)

#### ステージ3のローカル集計用のクエリキャッシュ

クエリは単一のDISTINCT集約関数を含むGROUP BY集約クエリです。

サポートされるDISTINCT集約関数は `sum(distinct)`, `count(distinct)`, `avg(distinct)` です。

> **注意**
>
> クエリに`avg(distinct)`が含まれている場合は、CTEの最適化を無効にするために`set cbo_cte_reuse = false`を実行する必要があります。

クエリの例:

```SQL
SELECT
    date_trunc('hour', ts) AS hour,
    v0 % 2 AS is_odd,
    sum(distinct v1) AS __c_0
FROM
    t0
WHERE
    ts between '2022-01-03 00:00:00'
    and '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    is_odd;
```

次の図は、クエリプロファイルのステージ3のクエリキャッシュ関連メトリクスを示しています。

![クエリキャッシュ - ステージ3 - メトリクス](../assets/query_cache_stage3_agg_with_cache_en.png)

#### ステージ4のローカル集計用のクエリキャッシュ

クエリは単一のDISTINCT集約関数を含むGROUP BY集約クエリです。このようなクエリには、重複データを除去する古典的なクエリが含まれます。

クエリの例:

```SQL
SELECT
    count(distinct v1) AS __c_0
FROM
    t0
WHERE
    ts between '2022-01-03 00:00:00'
    and '2022-01-03 23:59:59'
```

次の図は、クエリプロファイルのステージ4のクエリキャッシュ関連メトリクスを示しています。

![クエリキャッシュ - ステージ4 - メトリクス](../assets/query_cache_stage4_agg_with_cache_en.png)

#### 意味的に等価な最初の集計がセマンティック的に等価な2つのクエリでキャッシュされた結果が再利用される

例として、以下の2つのクエリ、Q1とQ2を使用します。 Q1とQ2の両方に複数の集計が含まれていますが、それらの最初の集計は意味的に等価です。したがって、Q1とQ2は意味的に等価として評価され、クエリキャッシュに保存された他方の計算結果を再利用できます。

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
              ts between '2022-01-03 00:00:00'
              and '2022-01-03 23:59:59'
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
      ts between '2022-01-03 00:00:00'
      and '2022-01-03 23:59:59'
  GROUP BY
      date_trunc('hour', ts),
      k0
  ```

次の図は、Q1の`CachePopulate`メトリクスを示しています。

![クエリキャッシュ - Q1 - メトリクス](../assets/query_cache_reuse_Q1_en.png)

次の図は、Q2の`CacheProbe`メトリクスを示しています。

![クエリキャッシュ - Q2 - メトリクス](../assets/query_cache_reuse_Q2_en.png)

#### CTEの最適化が有効になっているDISTINCTクエリではクエリキャッシュが機能しない
```
```
After you run `set cbo_cte_reuse = true` to enable CTE optimizations, the computation results for specific queries that include DISTINCT aggregate functions cannot be cached. A few examples are as follows:

- The query contains a single DISTINCT aggregate function `avg(distinct)`:

  ```SQL
  SELECT
      avg(distinct v1) AS __c_0
  FROM
      t0
  WHERE
      ts between '2022-01-03 00:00:00'
      and '2022-01-03 23:59:59';
  ```

![Query Cache - CTE - 1](../assets/query_cache_distinct_with_cte_Q1_en.png)

- The query contains multiple DISTINCT aggregate functions that reference the same column:

  ```SQL
  SELECT
      avg(distinct v1) AS __c_0,
      sum(distinct v1) AS __c_1,
      count(distinct v1) AS __c_2
  FROM
      t0
  WHERE
      ts between '2022-01-03 00:00:00'
      and '2022-01-03 23:59:59';
  ```

![Query Cache - CTE - 2](../assets/query_cache_distinct_with_cte_Q2_en.png)

- The query contains multiple DISTINCT aggregate functions that each reference a different column:

  ```SQL
  SELECT
      sum(distinct v1) AS __c_1,
      count(distinct v0) AS __c_2
  FROM
      t0
  WHERE
      ts between '2022-01-03 00:00:00'
      and '2022-01-03 23:59:59';
  ```

![Query Cache - CTE - 3](../assets/query_cache_distinct_with_cte_Q3_en.png)

## Best practices

When you create your table, specify a reasonable partition description and a reasonable distribution method, including:

- Choose a single DATE-type column as the partition column. If the table contains more than one DATE-type column, choose the column whose values roll forward as data is incrementally ingested and that is used to define your interesting time ranges of queries.
- Choose a proper partition width. The data ingested most recently may modify the latest partitions of the table. Therefore, the cache entries involving the latest partitions are unstable and are apt to be invalidated.
- Specify a bucket number that is several dozen in the distribution description of the table creation statement. If the bucket number is exceedingly small, the query cache cannot take effect when the number of tablets that need to be processed by the BE is less than the value of `pipeline_dop`.
```