---
displayed_sidebar: English
---

# クエリプロファイルの分析

このトピックでは、クエリプロファイルを確認する方法について説明します。クエリプロファイルは、クエリに関連するすべてのワーカーノードの実行情報を記録します。クエリプロファイルは、StarRocksクラスタのクエリパフォーマンスに影響を与えるボトルネックを迅速に特定するのに役立ちます。

## クエリプロファイルを有効にする

v2.5より前のStarRocksバージョンでは、変数`is_report_success`を`true`に設定することでクエリプロファイルを有効にできます：

```SQL
SET is_report_success = true;
```

StarRocks v2.5以降のバージョンでは、変数`enable_profile`を`true`に設定することでクエリプロファイルを有効にできます：

```SQL
SET enable_profile = true;
```

### ランタイムプロファイル

v3.1以降、StarRocksはランタイムプロファイル機能をサポートし、クエリが完了する前にクエリプロファイルにアクセスできるようになりました。

この機能を使用するには、`enable_profile`を`true`に設定するだけでなく、セッション変数`runtime_profile_report_interval`も設定する必要があります。`runtime_profile_report_interval`（単位：秒、デフォルト：`10`）は、プロファイルレポートの間隔を制御し、デフォルトで10秒に設定されています。つまり、クエリの実行時間が10秒を超えると、ランタイムプロファイル機能が自動的に有効になります。

```SQL
SET runtime_profile_report_interval = 10;
```

ランタイムプロファイルは、通常のクエリプロファイルと同じ情報を表示します。通常のクエリプロファイルと同様に分析して、実行中のクエリのパフォーマンスに関する貴重な洞察を得ることができます。

ただし、ランタイムプロファイルは不完全な場合があります。なぜなら、実行計画の一部のオペレーターは他のオペレーターに依存している可能性があるためです。実行中のオペレーターと完了したオペレーターを容易に区別するために、実行中のオペレーターは`Status: Running`とマークされます。

## クエリプロファイルへのアクセス

> **注記**
>
> StarRocksのEnterprise Editionを使用している場合は、StarRocks Managerを使用してクエリプロファイルにアクセスし、視覚化することができます。

StarRocksのCommunity Editionを使用している場合は、以下の手順でクエリプロファイルにアクセスします：

1. ブラウザに`http://<fe_ip>:<fe_http_port>`を入力します。
2. 表示されたページで、上部ナビゲーションパネルの**queries**をクリックします。
3. **Finished Queries**リストで、確認したいクエリを選択し、**Profile**列のリンクをクリックします。

![img](../assets/profile-1.png)

ブラウザは、対応するクエリプロファイルを含む新しいページにリダイレクトします。

![img](../assets/profile-2.png)

## クエリプロファイルの解釈

### クエリプロファイルの構造

以下はクエリプロファイルの例です：

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

クエリプロファイルは、次の3つのセクションで構成されます：

- Fragment：実行ツリー。1つのクエリは1つ以上のフラグメントに分割されます。
- Pipeline：実行チェーン。実行チェーンには分岐がありません。フラグメントは複数のパイプラインに分割されます。
- Operator：パイプラインは複数のオペレーターで構成されます。

![img](../assets/profile-3.png)

*フラグメントは複数のパイプラインで構成されています。*

### 主要指標

クエリプロファイルには、クエリ実行の詳細を示す多数のメトリックが含まれています。ほとんどの場合、オペレーターの実行時間と処理したデータのサイズだけを観察する必要があります。ボトルネックを特定したら、それに応じて対処できます。

#### 概要

| メトリック       | 説明                                                  |
| ------------ | ------------------------------------------------------------ |
| Total        | クエリによって消費された総時間、計画、実行、プロファイリングに費やされた時間を含む。 |
| QueryCpuCost | クエリのCPU時間の総コスト。CPU時間コストは、並行プロセスに集約されます。その結果、このメトリックの値はクエリの実際の実行時間よりも大きくなることがあります。 |
| QueryMemCost | クエリのメモリコストの総額。                              |

#### オペレーターのための普遍的なメトリック

| メトリック            | 説明                                           |
| ----------------- | ----------------------------------------------------- |
| OperatorTotalTime | オペレーターのCPU時間コストの総額。                  |
| PushRowNum        | オペレーターがプッシュしたデータの総行数。 |
| PullRowNum        | オペレーターがプルしたデータの総行数。 |

#### ユニークなメトリック

| メトリック            | 説明                             |
| ----------------- | --------------------------------------- |
| IOTaskExecTime    | すべてのI/Oタスクの実行時間の合計。 |
| IOTaskWaitTime    | すべてのI/Oタスクの待機時間の合計。      |
| MorselsCount      | I/Oタスクの総数。              |

#### スキャンオペレーター

| メトリック                          | 説明                                                  |
| ------------------------------- | ------------------------------------------------------------ |
| Table                           | テーブル名。                                                  |
| ScanTime                        | スキャンの総時間。スキャンは非同期I/Oスレッドプールで実行されます。 |
| TabletCount                     | タブレットの数。                                           |
| PushdownPredicates              | プッシュダウンされた述語の数。              |
| BytesRead                       | StarRocksによって読み取られたデータのサイズ。                          |
| CompressedBytesRead             | StarRocksによって読み取られた圧縮データのサイズ。               |
| IOTime                          | I/Oの総時間。                                              |
| BitmapIndexFilterRows           | ビットマップインデックスによってフィルタリングされたデータの行数。 |
| BloomFilterFilterRows           | ブルームフィルターによってフィルタリングされたデータの行数。 |
| SegmentRuntimeZoneMapFilterRows | ランタイムゾーンマップによってフィルタリングされたデータの行数。 |
| SegmentZoneMapFilterRows        | ゾーンマップによってフィルタリングされたデータの行数。    |
| ShortKeyFilterRows              | ショートキーフィルターによってフィルタリングされたデータの行数。   |
| ZoneMapIndexFilterRows          | ゾーンマップインデックスによってフィルタリングされたデータの行数。 |

#### エクスチェンジオペレーター

| メトリック            | 説明                                                  |
| ----------------- | ------------------------------------------------------------ |
| PartType          | データ配布タイプ。有効な値: `UNPARTITIONED`, `RANDOM`, `HASH_PARTITIONED`, `BUCKET_SHUFFLE_HASH_PARTITIONED`。 |
| BytesSent         | 送信されたデータのサイズ。                             |
| OverallThroughput | 全体的なスループット。                                          |
| NetworkTime       | データパッケージの送信時間（受信後の処理時間を除く）。このメトリックの計算方法と例外が発生する場合の詳細については、以下のFAQを参照してください。 |
| WaitTime          | 送信側のキューがいっぱいであるために発生する待機時間。   |

#### アグリゲートオペレーター

| メトリック             | 説明                                   |
| ------------------ | --------------------------------------------- |
| GroupingKeys       | グルーピングキー（GROUP BY列）の名前。 |
| AggregateFunctions | 集計関数。                          |
| AggComputeTime     | 集計関数の計算に消費される時間。 |

| ExprComputeTime    | 式の計算に消費される時間。      |
| HashTableSize      | ハッシュテーブルのサイズ。                       |

#### Join operator

| Metric                    | Description                         |
| ------------------------- | ----------------------------------- |
| JoinPredicates            | JOIN操作の述語。   |
| JoinType                  | JOINのタイプ。                      |
| BuildBuckets              | ハッシュテーブルのバケット数。    |
| BuildHashTableTime        | ハッシュテーブルを構築するのにかかった時間。  |
| ProbeConjunctEvaluateTime | Probe結合の評価にかかった時間。    |
| SearchHashTableTimer      | ハッシュテーブルを検索するのにかかった時間。 |

#### Window Function operator

| Metric             | Description                                           |
| ------------------ | ----------------------------------------------------- |
| ComputeTime        | ウィンドウ関数の計算にかかった時間。         |
| PartitionKeys      | パーティションキー（PARTITION BYカラム）の名前。 |
| AggregateFunctions | 集約関数。                                  |

#### Sort operator

| Metric   | Description                                                  |
| -------- | ------------------------------------------------------------ |
| SortKeys | ソートキー（ORDER BYカラム）の名前。                    |
| SortType | 結果のソートタイプ：全ての結果をリストするか、トップn結果のみをリストするか。 |

#### TableFunction operator

| Metric                 | Description                                     |
| ---------------------- | ----------------------------------------------- |
| TableFunctionExecTime  | テーブル関数の実行にかかった計算時間。    |
| TableFunctionExecCount | テーブル関数が実行された回数。 |

#### Project operator

| Metric                   | Description                                         |
| ------------------------ | --------------------------------------------------- |
| ExprComputeTime          | 式の計算にかかった時間。            |
| CommonSubExprComputeTime | 共通サブ式の計算にかかった時間。 |

#### LocalExchange operator

| metric     | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| Type       | LocalExchangeのタイプ。有効な値は`Passthrough`、`Partition`、`Broadcast`。 |
| ShuffleNum | シャッフルの回数。このメトリックは`Type`が`Partition`の場合のみ有効です。 |

#### Hive Connector

| Metric                      | Description                                            |
| --------------------------- | ------------------------------------------------------ |
| ScanRanges                  | スキャンされたタブレットの数。                    |
| ReaderInit                  | Readerの初期化時間。                         |
| ColumnReadTime              | Readerがデータを読み取り、解析するのにかかった時間。        |
| ExprFilterTime              | 式のフィルタリングに使われた時間。                       |
| RowsRead                    | 読み取られたデータ行の数。                     |

#### Input Stream

| Metric                    | Description                                                               |
| ------------------------- | ------------------------------------------------------------------------- |
| AppIOBytesRead            | アプリケーション層のI/Oタスクによって読み取られたデータのサイズ。            |
| AppIOCounter              | アプリケーション層のI/Oタスクの数。                           |
| AppIOTime                 | アプリケーション層のI/Oタスクがデータを読み取るのにかかった合計時間。 |
| FSBytesRead               | ストレージシステムによって読み取られたデータのサイズ。                              |
| FSIOCounter               | ストレージ層のI/Oタスクの数。                               |
| FSIOTime                  | ストレージ層がデータを読み取るのにかかった合計時間。                    |

### オペレーターによる時間消費

- OlapScanオペレーターとConnectorScanオペレーターについては、その時間消費は`OperatorTotalTime + ScanTime`に相当します。スキャンオペレーターは非同期I/OスレッドプールでI/O操作を行うため、ScanTimeは非同期I/O時間を表します。
- Exchangeオペレーターの時間消費は`OperatorTotalTime + NetworkTime`に相当します。ExchangeオペレーターはbRPCスレッドプールでデータパッケージを送受信するため、NetworkTimeはネットワーク伝送にかかる時間を表します。
- その他のすべてのオペレーターについては、その時間コストは`OperatorTotalTime`です。

### メトリックの統合とMIN/MAX

パイプラインエンジンは並列計算エンジンです。各フラグメントは複数のマシンに分散されて並列処理され、各マシン上のパイプラインは複数の同時インスタンスとして並列に実行されます。そのため、プロファイリング中にStarRocksは同じメトリックを統合し、すべての同時インスタンス間で各メトリックの最小値と最大値を記録します。

異なるタイプのメトリックには異なる統合戦略が採用されます：

- 時間メトリックは平均です。例えば：
  - `OperatorTotalTime`は、すべての同時インスタンスの平均時間コストを表します。
  - `__MAX_OF_OperatorTotalTime`は、すべての同時インスタンス間の最大時間コストです。
  - `__MIN_OF_OperatorTotalTime`は、すべての同時インスタンス間の最小時間コストです。

```SQL
             - OperatorTotalTime: 2.192us
               - __MAX_OF_OperatorTotalTime: 2.502us
               - __MIN_OF_OperatorTotalTime: 1.882us
```

- 時間以外のメトリックは合計されます。例えば：
  - `PullChunkNum`は、すべての同時インスタンスの合計数を表します。
  - `__MAX_OF_PullChunkNum`は、すべての同時インスタンスの最大値です。
  - `__MIN_OF_PullChunkNum`は、すべての同時インスタンスの最小値です。

  ```SQL
                 - PullChunkNum: 146.66K (146660)
                   - __MAX_OF_PullChunkNum: 24.45K (24450)
                   - __MIN_OF_PullChunkNum: 24.435K (24435)
  ```

- 最小値と最大値を持たない特別なメトリックは、すべての同時インスタンス間で同一の値を持ちます（例：`DegreeOfParallelism`）。

#### MINとMAXの顕著な差異

通常、MIN値とMAX値の顕著な差異はデータの偏りを示しています。これは集約やJOIN操作中に発生する可能性があります。

```SQL
             - OperatorTotalTime: 2m48s
               - __MAX_OF_OperatorTotalTime: 10m30s
               - __MIN_OF_OperatorTotalTime: 279.170us
```

## テキストベースのプロファイル分析を実行する

v3.1以降、StarRocksはよりユーザーフレンドリーなテキストベースのプロファイル分析機能を提供します。この機能を使用すると、クエリのボトルネックと最適化の機会を効率的に特定できます。

### 既存のクエリを分析する

クエリが実行中であっても完了していても、`QueryID`を使用して既存のクエリのプロファイルを分析できます。

#### プロファイルの一覧

以下のSQLステートメントを実行して、既存のプロファイルを一覧表示します：

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

このSQL文を使用すると、各クエリに関連付けられた`QueryId`を簡単に取得できます。`QueryId`は、さらなるプロファイル分析と調査のための重要な識別子として機能します。

#### プロファイルの分析

`QueryId`を取得したら、`ANALYZE PROFILE`文を使用して、特定のクエリに対してより詳細な分析を実行できます。このSQLステートメントは、より深い洞察を提供し、クエリのパフォーマンス特性と最適化の包括的な調査を容易にします。

```sql
ANALYZE PROFILE FROM '<QueryId>' [, <plan_node_id>, ...]
```

デフォルトでは、分析出力には、各オペレーターの最も重要なメトリックのみが表示されます。ただし、1つ以上のプランノードのIDを指定して、対応するメトリックを表示できます。この機能により、クエリのパフォーマンスをより包括的に調べることができ、ターゲットを絞った最適化が容易になります。

例1: プランノードIDを指定せずにプロファイルを分析する：

![img](../assets/profile-16.png)

例2: プロファイルを分析し、プランノードのIDを指定する：

![img](../assets/profile-17.png)

`ANALYZE PROFILE`ステートメントは、ランタイムプロファイルを分析および理解するための拡張されたアプローチを提供します。これにより、ブロック、実行中、終了など、さまざまな状態を持つオペレーターが区別されます。さらに、このステートメントは、処理された行番号に基づいて個々のオペレーターの進行状況だけでなく、全体の進行状況を包括的に表示し、クエリの実行とパフォーマンスをより深く理解できるようにします。この機能により、StarRocksでのクエリのプロファイリングと最適化がさらに容易になります。

例3: 実行中のクエリのランタイムプロファイルを分析する：

![img](../assets/profile-20.png)

### シミュレートされたクエリを分析する

また、特定のクエリをシミュレートし、`EXPLAIN ANALYZE`ステートメントを使用してそのプロファイルを分析することもできます。

```sql
EXPLAIN ANALYZE <sql>
```

現在、`EXPLAIN ANALYZE`は、クエリ（SELECT）ステートメントとINSERT INTOステートメントの2種類のSQLステートメントをサポートしています。INSERT INTOステートメントをシミュレートし、StarRocksのネイティブOLAPテーブルでそのプロファイルを分析することしかできません。INSERT INTOステートメントのプロファイルを分析する場合、実際にはデータは挿入されないことに注意してください。デフォルトでは、トランザクションは中止され、プロファイル分析の過程でデータに意図しない変更が加えられないようにします。

例1: 特定のクエリのプロファイルを分析する：

![img](../assets/profile-18.png)

例1: INSERT INTO操作のプロファイルを分析する：

![img](../assets/profile-19.png)

## クエリプロファイルを視覚化する

StarRocks Enterprise Editionのユーザーであれば、StarRocks Managerを介してクエリプロファイルを視覚化できます。

**プロファイル概要**ページには、合計実行時間`ExecutionWallTime`、I/Oメトリック、ネットワーク転送サイズ、CPU時間とI/O時間の割合など、いくつかの概要メトリックが表示されます。

![img](../assets/profile-4.jpeg)

オペレーター（ノード）のカードをクリックすると、ページの右側のペインにその詳細情報が表示されます。ここには3つのタブがあります：

- **Node**: このオペレーターのコアメトリクス。
- **Node Detail**: このオペレーターのすべてのメトリクス。
- **Pipeline**: オペレーターが属するパイプラインのメトリクス。このタブはスケジューリングにのみ関連しているため、あまり注意を払う必要はありません。

![img](../assets/profile-5.jpeg)

### ボトルネックの特定

オペレーターが要する時間の割合が大きいほど、そのカードの色は暗くなります。これにより、クエリのボトルネックを簡単に特定できます。

![img](../assets/profile-6.jpeg)

### データが偏っていないか確認する

時間の大部分を占めるオペレーターのカードをクリックし、その`MaxTime`と`MinTime`を確認します。`MaxTime`と`MinTime`の顕著な違いは、通常、データが偏っていることを示しています。

![img](../assets/profile-7.jpeg)

次に、**Node Detail**タブをクリックし、例外を示すメトリックがあるかどうかを確認します。この例では、Aggregateオペレーターの`PushRowNum`メトリックはデータの偏りを示しています。

![img](../assets/profile-8.jpeg)

### パーティショニングまたはバケッティング戦略が効果的かどうかを確認する

パーティショニングまたはバケッティング戦略が効果的かどうかは、`EXPLAIN <sql_statement>`を使用して対応するクエリプランを表示することで確認できます。

![img](../assets/profile-9.png)

### 正しいマテリアライズドビューが使用されているかどうかを確認する

対応するScanオペレーターをクリックし、**Node Detail**タブの`Rollup`フィールドを確認します。

![img](../assets/profile-10.jpeg)

### JOINプランが左右のテーブルに適しているかどうかを確認する

通常、StarRocksは小さい方のテーブルをJoinの右テーブルとして選択します。クエリプロファイルにそれ以外の場合が示されている場合は、例外が発生していることになります。

![img](../assets/profile-11.jpeg)

### JOINの分散タイプが正しいかどうかを確認する

Exchangeオペレーターは、データ分散タイプに応じて3つのタイプに分類されます：

- `UNPARTITIONED`：ブロードキャスト。データは複数のコピーに作成され、複数のBEに配布されます。
- `RANDOM`：ラウンドロビン。
- `HASH_PARTITIONED`と`BUCKET_SHUFFLE_HASH_PARTITIONED`：シャッフル。`HASH_PARTITIONED`と`BUCKET_SHUFFLE_HASH_PARTITIONED`の違いは、ハッシュコードを計算するために使用されるハッシュ関数にあります。

Inner Joinの場合、右テーブルは`HASH_PARTITIONED`または`BUCKET_SHUFFLE_HASH_PARTITIONED`タイプ、または`UNPARTITIONED`タイプにすることができます。通常、`UNPARTITIONED`タイプは、右テーブルの行数が10万行未満の場合にのみ採用されます。

以下の例では、Exchangeオペレーターのタイプはブロードキャストですが、オペレーターが送信するデータのサイズがしきい値を大幅に超えています。

![img](../assets/profile-12.jpeg)

### JoinRuntimeFilterが効果的かどうかを確認する

Joinの右子がハッシュテーブルを作成するとき、ランタイムフィルターが作成されます。このランタイムフィルターは左子ツリーに送信され、可能であればScanオペレーターにプッシュダウンされます。`JoinRuntimeFilter`に関連するメトリクスは、Scanオペレーターの**Node Detail**タブで確認できます。

![img](../assets/profile-13.jpeg)

## FAQ

### Exchangeオペレーターの時間コストが異常なのはなぜですか？

![img](../assets/profile-14.jpeg)

Exchangeオペレーターの時間コストは、CPU時間とネットワーク時間の2つの部分で構成されます。ネットワーク時間はシステムクロックに依存します。ネットワーク時間は次のように計算されます：

1. 送信者は、bRPCインターフェースを呼び出してパッケージを送信する前に`send_timestamp`を記録します。
2. 受信者は、bRPCインターフェースからパッケージを受信した後に`receive_timestamp`を記録します（受信後の処理時間は除外）。
3. 処理が完了すると、受信者は応答を送信し、ネットワーク遅延を計算します。パッケージの転送遅延は`receive_timestamp` - `send_timestamp`に相当します。

マシン間のシステムクロックが一致していない場合、Exchangeオペレーターの時間コストは異常になります。

### すべてのオペレーターの合計時間コストがクエリ実行時間よりも大幅に少ないのはなぜですか？

考えられる原因: 高い並行性のシナリオでは、スケジュール可能であっても、キューに入っているために、一部のパイプラインドライバーがタイムリーに処理されないことがあります。オペレーターのメトリクスには待機時間は記録されませんが、`PendingTime`、`ScheduleTime`、そして`IOTaskWaitTime`には記録されます。

例:

プロファイルを見ると、`ExecutionWallTime`は約55ミリ秒であることがわかります。しかし、すべてのオペレーターの合計時間コストは10ミリ秒未満で、`ExecutionWallTime`よりも大幅に少ないです。

![img](../assets/profile-15.jpeg)
