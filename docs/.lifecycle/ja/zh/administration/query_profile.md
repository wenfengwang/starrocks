---
displayed_sidebar: Chinese
---

# Query Profile の確認と分析

この文書では、Query Profile の確認と分析方法について説明します。Query Profile は、クエリに関わるすべてのワークノードの実行情報を記録します。Query Profile を通じて、StarRocks クラスターのクエリパフォーマンスに影響を与えるボトルネックを迅速に特定できます。

## Query Profile の有効化

StarRocks v2.5 以前のバージョンでは、変数 `is_report_success` を `true` に設定して Query Profile を有効にします：

```SQL
SET is_report_success = true;
```

StarRocks v2.5 以降のバージョンでは、変数 `enable_profile` を `true` に設定して Query Profile を有効にします：

```SQL
SET enable_profile = true;
```

### Runtime Profile

v3.1 から、StarRocks は Runtime Profile 機能をサポートしています。クエリが完了する前にその Query Profile を確認できます。

この機能を使用するには、`enable_profile` を `true` に設定した後、変数 `runtime_profile_report_interval` を設定する必要があります。`runtime_profile_report_interval` は Query Profile の報告間隔を制御し、単位は秒で、デフォルト値は `10` です。デフォルト設定は、クエリ時間が 10 秒を超えると、システムが自動的に Runtime Profile 機能を有効にすることを意味します。

```SQL
SET runtime_profile_report_interval = 10;
```

Runtime Profile は通常の Query Profile と同じ情報を表示します。通常の Query Profile を分析するように Runtime Profile を分析して、クラスター内で実行中のクエリのパフォーマンス指標を理解できます。

ただし、実行計画の一部の Operator が他の Operator に依存している可能性があるため、Runtime Profile が不完全な場合があります。実行中の Operator と完了した Operator を区別するために、実行中の Operator は `Status: Running` とマークされます。

## Query Profile の取得

> **説明**
>
> StarRocks のエンタープライズ版を使用している場合は、StarRocks Manager を使用して Query Profile を取得し、可視化することができます。

StarRocks のコミュニティ版を使用している場合は、以下の手順で Query Profile を取得してください：

1. ブラウザで `http://<fe_ip>:<fe_http_port>` にアクセスします。
2. 表示されたページで、上部ナビゲーションの **queries** をクリックします。
3. **Finished Queries** リストで、分析したいクエリを選択し、**Profile** 列のリンクをクリックします。

![img](../assets/profile-1.png)

ページが対応する Query Profile にリダイレクトされます。

![img](../assets/profile-2.png)

## Query Profile の分析

### Query Profile の構造

以下は Query Profile の例です：

```SQL
Query:
  Summary:
  Planner:
  Execution Profile 7de16a85-761c-11ed-917d-00163e14d435:
    Fragment 0:
      Pipeline (id=2):
        EXCHANGE_SINK (plan_node_id=18):
        LOCAL_MERGE_SOURCE (plan_node_id=17):
      Pipeline (id=1):
        LOCAL_SORT_SINK (plan_node_id=17):
        AGGREGATE_BLOCKING_SOURCE (plan_node_id=16):
      Pipeline (id=0):
        AGGREGATE_BLOCKING_SINK (plan_node_id=16):
        EXCHANGE_SOURCE (plan_node_id=15):
    Fragment 1:
       ...
    Fragment 2:
       ...
```

Query Profile は以下の三つの部分で構成されています：

- Fragment：実行ツリー。クエリは一つまたは複数の Fragment で構成されます。
- Pipeline：実行チェーン。実行チェーンには分岐がありません。一つの Fragment は複数の Pipeline に分けられます。
- Operator：オペレータ。一つの Pipeline は複数の Operator で構成されます。

![img](../assets/profile-3.png)

### 重要な指標

Query Profile には、クエリ実行の詳細情報を含む多くの指標が含まれています。ほとんどの場合、オペレータの実行時間と処理したデータ量のサイズに注目するだけで十分です。ボトルネックを見つけたら、それらをターゲットに解決策を講じることができます。

#### Summary 指標

| 指標         | 説明                                                         |
| ------------ | ------------------------------------------------------------ |
| Total        | クエリが消費した総時間。Planning、Executing、および Profiling の各段階の時間を含みます。 |
| QueryCpuCost | クエリが累積した CPU 使用時間。すべての並行プロセスが加算されるため、この指標は実際の実行時間よりも大きくなります。 |
| QueryMemCost | クエリの総メモリ消費量。                                     |

#### Operator 共通指標

| 指標              | 説明                    |
| ----------------- | ----------------------- |
| OperatorTotalTime | Operator が消費した総時間。 |
| PushRowNum        | Operator が累積した入力行数。 |
| PullRowNum        | Operator が累積した出力行数。 |

#### Unique 指標

| 指標              | 説明                        |
| ----------------- | -------------------------- |
| IOTaskExecTime    | すべての I/O タスクの累計実行時間。 |
| IOTaskWaitTime    | すべての I/O タスクの累計待機時間。 |
| MorselsCount      | I/O タスクの総数。            |

#### Scan Operator

| 指標                            | 説明                                              |
| ------------------------------- | ------------------------------------------------- |
| Table                           | テーブル名。                                      |
| ScanTime                        | Scan の累計時間。Scan 操作は非同期 I/O スレッドプールで完了します。 |
| TabletCount                     | Tablet の数。                                     |
| PushdownPredicates              | 下方向への述語の数。                              |
| BytesRead                       | 読み取ったデータのサイズ。                        |
| CompressedBytesRead             | 読み取った圧縮データのサイズ。                    |
| IOTime                          | 累計 I/O 時間。                                   |
| BitmapIndexFilterRows           | Bitmap インデックスによってフィルタリングされた行数。 |
| BloomFilterFilterRows           | Bloomfilter によってフィルタリングされた行数。    |
| SegmentRuntimeZoneMapFilterRows | Runtime Zone Map によってフィルタリングされた行数。 |
| SegmentZoneMapFilterRows        | Zone Map によってフィルタリングされた行数。       |
| ShortKeyFilterRows              | Short Key によってフィルタリングされた行数。      |
| ZoneMapIndexFilterRows          | Zone Map インデックスによってフィルタリングされた行数。 |

#### Exchange Operator

| 指標              | 説明                                                         |
| ----------------- | ------------------------------------------------------------ |
| PartType          | データ分布モード。`UNPARTITIONED`、`RANDOM`、`HASH_PARTITIONED`、`BUCKET_SHUFFLE_HASH_PARTITIONED` があります。 |
| BytesSent         | 送信されたデータのサイズ。                                   |
| OverallThroughput | 吞吐量。                                                     |
| NetworkTime       | データパケットの転送時間（受信後の処理時間は含まない）。下記の FAQ を参照して、指標の計算方法と潜在的な例外についての詳細を確認してください。 |
| WaitTime          | 送信側のキューが満杯で発生した待機時間。                     |

#### Aggregate Operator

| 指標               | 説明               |
| ------------------ | ------------------ |
| GroupingKeys       | GROUP BY の列。    |
| AggregateFunctions | 集約関数。         |
| AggComputeTime     | 集約関数の計算時間。 |
| ExprComputeTime    | 式の計算時間。     |
| HashTableSize      | Hash Table のサイズ。 |

#### Join Operator

| 指標                      | 説明                        |
| ------------------------- | --------------------------- |
| JoinPredicates            | Join の述語。               |
| JoinType                  | Join のタイプ。             |
| BuildBuckets              | Hash Table のバケット数。   |
| BuildHashTableTime        | Hash Table の構築時間。     |
| ProbeConjunctEvaluateTime | Probe Conjunct の評価時間。 |
| SearchHashTableTimer      | Hash Table の検索時間。     |

#### Window Function Operator

| 指標               | 説明               |
| ------------------ | ------------------ |
| ComputeTime        | ウィンドウ関数の計算時間。 |
| PartitionKeys      | パーティションのキー。   |
| AggregateFunctions | 集約関数。         |

#### Sort Operator

| 指標     | 説明                                            |
| -------- | ----------------------------------------------- |
| SortKeys | ソートキー。                                    |
| SortType | ソートタイプ。全ソートまたはトップ N の結果のソート。 |

#### TableFunction Operator

| 指標                   | 説明                      |
| ---------------------- | ------------------------- |
| TableFunctionExecTime  | Table Function の計算時間。 |
| TableFunctionExecCount | Table Function の実行回数。 |

#### Project Operator

| 指標                     | 説明                   |
| ------------------------ | ---------------------- |
| ExprComputeTime          | 式の計算時間。         |
| CommonSubExprComputeTime | 共通サブ式の計算時間。 |

#### LocalExchange Operator

| 指標       | 説明                                                         |
| ---------- | ------------------------------------------------------------ |
| Type       | Local Exchange のタイプ。`Passthrough`、`Partition`、`Broadcast` があります。 |
| ShuffleNum | シャッフルの数。この指標は `Type` が `Partition` の場合にのみ有効です。 |

#### Hive Connector

| 指標                      | 説明                        |
| ------------------------- | --------------------------- |
| ScanRanges                | スキャンされたデータ範囲の総数。 |
| ReaderInit                | Reader の初期化時間。       |
| ColumnReadTime            | Reader によるデータの読み取りと解析時間。 |
| ExprFilterTime            | 式のフィルタリング時間。    |
| RowsRead                  | 読み取られた行数。          |

#### Input Stream

| 指標                      | 説明                        |
| ------------------------- | --------------------------- |
| AppIOBytesRead            | アプリケーション層で読み取られたデータ量。 |
| AppIOCounter              | アプリケーション層での I/O 回数。 |
| AppIOTime                 | アプリケーション層での累計読み取り時間。 |
| FSBytesRead               | ストレージシステムで読み取られたデータ量。 |
| FSIOCounter               | ストレージ層での I/O 回数。 |
| FSIOTime                  | ストレージ層での累計読み取り時間。 |

### Operator の実行時間

- OlapScan および ConnectorScan Operator については、実行時間は `OperatorTotalTime + ScanTime` に相当します。Scan Operator は非同期 I/O スレッドプールで I/O 操作を行うため、ScanTime は非同期 I/O 時間です。
- Exchange Operator の実行時間は `OperatorTotalTime + NetworkTime` に相当します。Exchange Operator は bRPC スレッドプールでデータパケットの送受信を行うため、NetworkTime はネットワーク転送にかかる時間です。
- その他の Operator については、実行時間は `OperatorTotalTime` です。

### Metric の統合および MIN/MAX 値


Pipeline Engineは並列計算エンジンです。各フラグメントは複数のマシンに分散して並列処理され、各マシン上のパイプラインは複数の並行インスタンスで同時に実行されます。そのため、プロファイルの統計時には、StarRocksは同じ指標を統合し、すべての並行インスタンスの各指標の最小値と最大値を記録します。

異なる種類の指標には異なる統合戦略が採用されています：

- 時間指標は平均値を求めます。例：
  - `OperatorTotalTime`はすべての並行インスタンスの平均処理時間です。
  - `__MAX_OF_OperatorTotalTime`はすべての並行インスタンスの最大処理時間です。
  - `__MIN_OF_OperatorTotalTime`はすべての並行インスタンスの最小処理時間です。

```SQL
             - OperatorTotalTime: 2.192us
               - __MAX_OF_OperatorTotalTime: 2.502us
               - __MIN_OF_OperatorTotalTime: 1.882us
```

- 時間指標以外の指標は合計値を求めます。例：
  - `PullChunkNum`はすべての並行インスタンスでの指標の合計です。
  - `__MAX_OF_PullChunkNum`はすべての並行インスタンスでの指標の最大値です。
  - `__MIN_OF_PullChunkNum`はすべての並行インスタンスでの指標の最小値です。

```SQL
             - PullChunkNum: 146.66K (146660)
               - __MAX_OF_PullChunkNum: 24.45K (24450)
               - __MIN_OF_PullChunkNum: 24.435K (24435)
```

- 最小値と最大値が存在しない一部の指標は、すべての並行インスタンスでの値が同じです。例：`DegreeOfParallelism`。

#### MINとMAXの値の差が大きい場合

通常、MINとMAXの値に明らかな差がある場合、データに偏りがある可能性があります。聚合や結合などのシナリオが考えられます。

```SQL
             - OperatorTotalTime: 2m48s
               - __MAX_OF_OperatorTotalTime: 10m30s
               - __MIN_OF_OperatorTotalTime: 279.170us
```

## テキストベースのクエリプロファイル分析

v3.1以降、StarRocksはテキストベースのクエリプロファイル分析機能をサポートしています。この機能を使用すると、クエリのボトルネックや最適化の機会を効果的に特定できます。

### 既存のクエリの分析

クエリの`QueryID`を使用して、既存のクエリのクエリプロファイルを分析できます。クエリは実行中のクエリまたは完了したクエリのいずれかです。

```sql
SHOW PROFILELIST;
```

例：

```sql
MySQL > show profilelist;
+--------------------------------------+---------------------+-------+----------+--------------------------------------------------------------------------------------------------------------------------------------+
| QueryId                              | StartTime           | Time  | State    | Statement                                                                                                                            |
+--------------------------------------+---------------------+-------+----------+--------------------------------------------------------------------------------------------------------------------------------------+
| b8289ffc-3049-11ee-838f-00163e0a894b | 2023-08-01 16:59:27 | 86ms  | Finished | SELECT o_orderpriority, COUNT(*) AS order_count\nFROM orders\nWHERE o_orderdate >= DATE '1993-07-01'\n    AND o_orderdate < DAT ...  |
| b5be2fa8-3049-11ee-838f-00163e0a894b | 2023-08-01 16:59:23 | 67ms  | Finished | SELECT COUNT(*)\nFROM (\n    SELECT l_orderkey, SUM(l_extendedprice * (1 - l_discount)) AS revenue\n        , o_orderdate, o_sh ...  |
| b36ac9c6-3049-11ee-838f-00163e0a894b | 2023-08-01 16:59:19 | 320ms | Finished | SELECT COUNT(*)\nFROM (\n    SELECT s_acctbal, s_name, n_name, p_partkey, p_mfgr\n        , s_address, s_phone, s_comment\n    F ... |
| b037b245-3049-11ee-838f-00163e0a894b | 2023-08-01 16:59:14 | 175ms | Finished | SELECT l_returnflag, l_linestatus, SUM(l_quantity) AS sum_qty\n    , SUM(l_extendedprice) AS sum_base_price\n    , SUM(l_exten ...   |
| a9543cf4-3049-11ee-838f-00163e0a894b | 2023-08-01 16:59:02 | 40ms  | Finished | select count(*) from lineitem                                                                                                        |
+--------------------------------------+---------------------+-------+----------+--------------------------------------------------------------------------------------------------------------------------------------+
5 rows in set
Time: 0.006s


MySQL > show profilelist limit 1;
+--------------------------------------+---------------------+------+----------+-------------------------------------------------------------------------------------------------------------------------------------+
| QueryId                              | StartTime           | Time | State    | Statement                                                                                                                           |
+--------------------------------------+---------------------+------+----------+-------------------------------------------------------------------------------------------------------------------------------------+
| b8289ffc-3049-11ee-838f-00163e0a894b | 2023-08-01 16:59:27 | 86ms | Finished | SELECT o_orderpriority, COUNT(*) AS order_count\nFROM orders\nWHERE o_orderdate >= DATE '1993-07-01'\n    AND o_orderdate < DAT ... |
+--------------------------------------+---------------------+------+----------+-------------------------------------------------------------------------------------------------------------------------------------+
1 row in set
Time: 0.005s
```

このSQL文を使用して、各クエリに関連付けられた`QueryID`を取得できます。`QueryID`は各クエリの重要な識別子です。

#### プロファイルの分析

`QueryID`を取得したら、特定のクエリに対してより詳細な分析を実行するためにANALYZE PROFILEステートメントを使用できます。このSQL文はより詳細な情報を提供し、クエリのパフォーマンスを総合的に分析するのに役立ちます。

```sql
ANALYZE PROFILE FROM '<QueryId>' [, <plan_node_id>, ...]
```

デフォルトでは、分析結果には各Operatorの主要な指標のみが表示されます。特定の計画ノードのIDを指定して、それに対応する指標を表示することもできます。この機能は、クエリのパフォーマンスをより詳細に調査し、最適化を実施するために役立ちます。

例1：計画ノードIDを指定せずにプロファイルを分析する場合：

![IMGの](../assets/profile-21.png)

例2：計画ノードIDを指定してプロファイルを分析する場合：

![IMGの](../assets/profile-17.png)

ANALYZE PROFILEは、実行中のクエリのプロファイルを分析するために使用できます。その戻り値には、Operatorの異なる状態（ブロックされている、実行中、完了）が表示されます。さらに、このステートメントは処理された行数に基づいてクエリの進捗状況全体や各Operatorの進捗状況を包括的に分析することができます。このステートメントの戻り値を使用して、クエリの実行とパフォーマンスをより詳細に理解し、さらなる分析と最適化を行うことができます。

例3：実行中のクエリのランタイムプロファイルを分析する場合：

![IMGの](../assets/profile-20.png)

### シミュレートクエリの分析

EXPLAIN ANALYZEステートメントを使用して、指定されたクエリ文をシミュレートし、そのクエリプロファイルを分析することもできます。

```sql
EXPLAIN ANALYZE <sql>
```

現在、EXPLAIN ANALYZEは2種類のSQL文（SELECT文とINSERT INTO文）をサポートしています。INSERT INTO文のクエリプロファイルをシミュレートして分析する場合は、StarRocks内部テーブルでのみ使用できます。なお、INSERT INTO文のクエリプロファイルをシミュレートして分析する場合、実際にデータがインポートされるわけではないことに注意してください。デフォルトでは、インポートトランザクションは中止され、データが意図せず変更されることがないようになっています。

例1：指定されたクエリ文をシミュレートし、そのクエリプロファイルを分析する場合：

![IMGの](../assets/profile-18.png)

例2：指定されたINSERT INTO文をシミュレートし、そのクエリプロファイルを分析する場合：

![IMGの](../assets/profile-19.png)

## クエリプロファイルの可視化

StarRocks Enterprise Editionのユーザーであれば、StarRocks Managerを使用してクエリプロファイルを可視化することができます。

**実行の概要**ページには、総実行時間`ExecutionWallTime`、I/O指標、ネットワーク転送サイズ、およびCPUとI/Oの時間比のようないくつかのサマリ指標が表示されます。

![IMGの](../assets/profile-4.png)

Operator（ノード）のカードをクリックすると、右側のタブで詳細情報を表示できます。3つのタブがあります：

- **ノード**：そのOperatorの主要な指標。
- **ノードの詳細**：そのOperatorのすべての指標。
- **パイプライン**：そのOperatorが所属するパイプラインの指標。このタブの指標はスケジューリングに関連しており、詳細な注意が必要ではありません。

![IMGの](../assets/profile-5.png)

### クエリのボトルネックを確認する

Operatorの時間比率が大きいほど、対応するカードの色が濃くなります。これにより、クエリのボトルネックを簡単に確認できます。

![IMGの](../assets/profile-6.png)

### データの偏りを確認する

時間比率が大きいOperatorのカードをクリックし、`MaxTime`と`MinTime`の指標を確認します。通常、`MaxTime`と`MinTime`の間に明らかな差がある場合、データに偏りがあることを示しています。

![IMGの](../assets/profile-7.png)

次に、**ノードの詳細**タブをクリックし、異常な指標があるかどうかを確認します。以下の例では、集計演算子の`PushRowNum`指標がデータの偏りを示しています。

![IMGの](../assets/profile-8.png)

### パーティションやバケットの剪定が効果的かどうかを確認する

`EXPLAIN <sql_statement>`文を使用して、クエリに対応するクエリプランを確認し、パーティションやバケットの剪定が効果的かどうかを確認できます。

![IMGの](../assets/profile-9.png)

### マテリアライズドビューの選択が正しいかどうかを確認する

対応するScan Operatorのカードをクリックし、**ノードの詳細**タブの`Rollup`フィールドを確認します。

![IMGの](../assets/profile-10.png)

### Joinの左右テーブルのプランが適切かどうかを確認する

通常、StarRocksはJoinの右テーブルとしてより小さいテーブルを選択します。クエリプロファイルが右テーブルのデータ量が明らかに左テーブルよりも大きい場合、Joinプランに問題がある可能性があります。

![IMGの](../assets/profile-11.png)

### Joinの分散方法が正しいかどうかを確認する

データの分散タイプに応じて、Exchange Operatorは次の3つのタイプに分類されます：

- `UNPARTITIONED`：ブロードキャスト。データは複数のBEにコピーされて送信されます。
- `RANDOM`：ラウンドロビン。
- `HASH_PARTITIONED`および`BUCKET_SHUFFLE_HASH_PARTITIONED`：シャッフル。`HASH_PARTITIONED`と`BUCKET_SHUFFLE_HASH_PARTITIONED`の違いは、ハッシュコードの計算に使用されるハッシュ関数が異なることです。

Inner Joinの場合、右テーブルは`HASH_PARTITIONED`または`BUCKET_SHUFFLE_HASH_PARTITIONED`タイプ、または`UNPARTITIONED`タイプである場合があります。通常、右テーブルの行数が100K未満の場合にのみ`UNPARTITIONED`タイプが使用されます。

以下の例では、Exchange Operatorのタイプがブロードキャストであり、そのOperatorが転送するデータ量が閾値を大幅に超えていることがわかります。

![IMGの](../assets/profile-12.png)

### JoinRuntimeFilterが効果的かどうかを確認する

Joinの右の子ノードがハッシュテーブルを構築する際、ランタイムフィルタが構築され、そのランタイムフィルタが左の子ツリーに配信され、できるだけScan Operatorにプッシュダウンされます。JoinRuntimeFilterに関連する指標は、Scan Operatorの**ノードの詳細**タブで確認できます。

![IMGの](../assets/profile-13.png)

## FAQ

### なぜExchange Operatorの時間が異常なのですか？

![IMGの](../assets/profile-14.png)

Exchange Operatorの時間は、CPU時間とネットワーク時間の2つの要素で構成されます。ネットワーク時間はシステムクロックに依存します。ネットワーク時間の計算方法は次のとおりです。

1. 送信側はbRPCインターフェースを呼び出す前に`send_timestamp`を記録します。
2. 受信側はbRPCインターフェースからパケットを受信した後、`receive_timestamp`を記録します（受信後の処理時間は含まれません）。
3. 処理が完了した後、受信側はレスポンスを送信し、ネットワーク遅延を計算します。データパケットの伝送遅延は`receive_timestamp` - `send_timestamp`です。

異なるマシンのシステムクロックが同期していない場合、Exchange Operatorの時間に異常が発生する可能性があります。

### 各Operatorの時間の合計が実際の実行時間よりもはるかに小さいのはなぜですか？

可能な原因：高並行性のシナリオでは、いくつかの Pipeline Driver はスケジュール可能ですが、キューイングによりタイムリーに実行されないことがあります。待機時間は Operator のメトリクスには記録されず、`PendingTime`、`ScheduleTime`、`IOTaskWaitTime` に記録されます。

例：

Pipeline Profile を見ると、`ExecutionWallTime` は約 463 ミリ秒であることがわかります。しかし、すべての Operator の合計実行時間は 20 ミリ秒に満たず、`ExecutionWallTime` よりも明らかに短いです。

![img](../assets/profile-16.png)

![img](../assets/assets/profile-15.png)
