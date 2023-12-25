---
displayed_sidebar: English
---

# Colocate Join

シャッフル結合とブロードキャスト結合では、結合条件が満たされた場合、2つの結合テーブルのデータ行が単一のノードにマージされて結合が完了します。これら2つの結合方法は、ノード間のデータネットワーク転送による遅延やオーバーヘッドを避けることはできません。

核心となるアイデアは、同じColocation Group内のテーブルでバケッティングキー、レプリカ数、レプリカ配置を一貫させることです。結合カラムがバケッティングキーであれば、計算ノードは他のノードからデータを取得することなくローカル結合のみを必要とします。Colocate Joinは等価結合をサポートします。

このドキュメントでは、Colocate Joinの原理、実装、使用方法、および考慮事項について紹介します。

## 用語

* **Colocation Group（CG）**: CGは1つ以上のテーブルを含みます。CG内のテーブルは同じバケッティングとレプリカ配置を持ち、Colocation Group Schemaを用いて記述されます。
* **Colocation Group Schema（CGS）**: CGSは、CGのバケッティングキー、バケット数、レプリカ数を含みます。

## 原理

Colocate Joinは、同じCGSを持つテーブルのセットでCGを形成し、これらのテーブルの対応するバケットコピーが同じBEノードセットに配置されるようにすることです。CG内のテーブルがバケット化されたカラムで結合操作を行う際、ローカルデータを直接結合でき、ノード間でデータを転送する時間が節約されます。

バケットSeqは `hash(key) mod buckets` によって得られます。例えば、テーブルに8つのバケットがある場合、\[0, 1, 2, 3, 4, 5, 6, 7\]の8つのバケットがあり、各バケットには1つ以上のサブテーブルがあり、サブテーブルの数はパーティション数に依存します。マルチパーティションテーブルの場合、複数のタブレットが存在します。

同じデータ分布を持つために、同じCG内のテーブルは以下の条件に従う必要があります。

1. 同じCG内のテーブルは、複数のテーブルのデータスライスを一つずつ分散して制御できるように、同一のバケッティングキー（型、数、順序）と同じ数のバケットを持つ必要があります。バケッティングキーは、テーブル作成ステートメントの `DISTRIBUTED BY HASH(col1, col2, ...)` で指定されたカラムです。バケッティングキーは異なるバケットSeqにデータのどのカラムをハッシュするかを決定します。バケッティングキーの名前は、同じCG内のテーブルで異なる場合があります。バケッティングカラムは作成ステートメントで異なる場合がありますが、`DISTRIBUTED BY HASH(col1, col2, ...)` での対応するデータ型の順序は完全に同じである必要があります。
2. 同じCG内のテーブルは、同じ数のパーティションコピーを持つ必要があります。そうでない場合、タブレットコピーが同じBEのパーティションに対応するコピーを持たない可能性があります。
3. 同じCG内のテーブルは、パーティション数とパーティションキーが異なる場合があります。

テーブルを作成する際には、テーブルPROPERTIESの属性 `"colocate_with" = "group_name"` を指定してCGを指定します。CGが存在しない場合、それはテーブルがCGの最初のテーブルであり、親テーブルと呼ばれることを意味します。親テーブルのデータ分布（分割バケッティングキーの型、数、順序、コピー数、分割バケット数）がCGSを決定します。CGが存在する場合、テーブルのデータ分布がCGSと一致しているかどうかをチェックします。

同じCG内のテーブルのコピー配置は以下を満たします：

1. すべてのテーブルのBucket SeqとBEノード間のマッピングは、親テーブルのそれと同じです。
2. 親テーブル内のすべてのパーティションのBucket SeqとBEノード間のマッピングは、最初のパーティションのそれと同じです。
3. 親テーブルの最初のパーティションのBucket SeqとBEノード間のマッピングは、ネイティブのRound Robinアルゴリズムを使用して決定されます。

一貫したデータ分布とマッピングにより、バケッティングキーで取得された同じ値を持つデータ行が同じBEに落ちることが保証されるため、バケッティングキーを使用して結合カラムを結合する場合、ローカル結合のみが必要です。

## 使用方法

### テーブル作成

テーブルを作成する際には、PROPERTIES内で `"colocate_with" = "group_name"` 属性を指定することで、そのテーブルがColocate Joinテーブルであり、指定されたColocation Groupに属することを示すことができます。
> **注記**
>
> バージョン2.5.4から、異なるデータベースのテーブル間でColocate Joinを実行できます。テーブルを作成する際に同じ `colocate_with` プロパティを指定するだけです。

例：

~~~SQL
CREATE TABLE tbl (k1 int, v1 int sum)
DISTRIBUTED BY HASH(k1)
BUCKETS 8
PROPERTIES(
    "colocate_with" = "group1"
);
~~~

指定されたグループが存在しない場合、StarRocksは自動的に現在のテーブルのみを含むグループを作成します。グループが存在する場合、StarRocksは現在のテーブルがColocation Group Schemaに適合しているかどうかを確認します。適合する場合は、テーブルを作成し、グループに追加します。同時に、テーブルは既存のグループのデータ分散ルールに基づいてパーティションとタブレットを作成します。

Colocation Groupはデータベースに属しています。Colocation Groupの名前はデータベース内でユニークです。内部ストレージでは、Colocation Groupの完全な名前は `dbId_groupName` ですが、ユーザーは `groupName` のみを認識します。
> **注記**
>
> 異なるデータベースのテーブルを同じColocation Groupに関連付けるために指定する場合、Colocation Groupはそれぞれのデータベースに存在します。`show proc "/colocation_group"` を実行して、異なるデータベースのColocation Groupを確認できます。

### 削除

 完全削除とは、リサイクルビンからの削除を指します。通常、`DROP TABLE` コマンドでテーブルを削除した後、デフォルトでは1日間リサイクルビンに残り、その後削除されます。グループ内の最後のテーブルが完全に削除されると、グループも自動的に削除されます。

### グループ情報の表示

次のコマンドを使用すると、クラスタ内に既に存在するグループ情報を表示できます。

~~~Plain Text
SHOW PROC '/colocation_group';

+-------------+--------------+--------------+------------+----------------+----------+----------+
| GroupId     | GroupName    | TableIds     | BucketsNum | ReplicationNum | DistCols | IsStable |
+-------------+--------------+--------------+------------+----------------+----------+----------+
| 10005.10008 | 10005_group1 | 10007, 10040 | 10         | 3              | int(11)  | true     |
+-------------+--------------+--------------+------------+----------------+----------+----------+
~~~

* **GroupId**: グループのクラスタ全体における一意の識別子で、前半はdb id、後半はgroup idです。
* **GroupName**: グループの正式名称。
* **TableIds**: グループ内のテーブルのIDリスト。
* **BucketsNum**: バケットの数。
* **ReplicationNum**: レプリカの数。
* **DistCols**: 分散列、つまりバケッティング列の型。
* **IsStable**: グループが安定しているかどうか（安定性の定義については、Colocation Replica Balancing and Repairのセクションを参照）。

次のコマンドを使用して、グループのデータ分布をさらに確認できます。

~~~Plain Text
SHOW PROC '/colocation_group/10005.10008';

+-------------+---------------------+
| BucketIndex | BackendIds          |
+-------------+---------------------+
| 0           | 10004, 10002, 10001 |
| 1           | 10003, 10002, 10004 |
| 2           | 10002, 10004, 10001 |
| 3           | 10003, 10002, 10004 |
| 4           | 10002, 10004, 10003 |
| 5           | 10003, 10002, 10001 |
| 6           | 10003, 10004, 10001 |
| 7           | 10003, 10004, 10002 |
+-------------+---------------------+
~~~

* **BucketIndex**: バケットシーケンスのインデックス。
* **BackendIds**: バケットデータのスライスが配置されているBEノードのID。

> 注意: 上記のコマンドにはADMIN権限が必要です。一般ユーザーはアクセスできません。

### テーブルグループプロパティの変更

テーブルのコロケーショングループプロパティを変更することができます。例えば：

~~~SQL
ALTER TABLE tbl SET ("colocate_with" = "group2");
~~~

テーブルが以前にグループに割り当てられていない場合、コマンドはスキーマをチェックし、テーブルをグループに追加します（グループが存在しない場合は、最初に作成されます）。テーブルが以前に別のグループに割り当てられていた場合、コマンドは元のグループからテーブルを削除し、新しいグループに追加します（グループが存在しない場合は、最初に作成されます）。

次のコマンドを使用して、テーブルのコロケーションプロパティを削除することもできます。

~~~SQL
ALTER TABLE tbl SET ("colocate_with" = "");
~~~

### その他の関連操作

`ADD PARTITION` を使用してパーティションを追加する場合や、コロケーション属性を持つテーブルのレプリカ数を変更する場合、StarRocksは操作がコロケーショングループスキーマに違反していないかをチェックし、違反していれば拒否します。

## コロケーションレプリカのバランシングと修復

コロケーションテーブルのレプリカ分布は、グループスキーマで指定された分布ルールに従う必要があるため、通常のシャーディングとは異なり、レプリカの修復とバランシングが異なります。

グループ自体には`stable`プロパティがあります。`stable`が`true`の場合、グループ内のテーブルスライスに変更がなく、コロケーション機能が正常に機能していることを意味します。`stable`が`false`の場合、現在のグループ内の一部のテーブルスライスが修復または移行中であり、影響を受けるテーブルのColocate Joinは通常のJoinに劣化します。

### レプリカの修復

レプリカは指定されたBEノードにのみ保存できます。StarRocksは、使用できないBE（例：ダウン、退役）を置き換えるために最も負荷の少ないBEを探します。交換後、古いBE上のすべてのバケットデータスライスが修復されます。移行中、グループは**Unstable**とマークされます。

### レプリカのバランシング

StarRocksはコロケーションテーブルスライスをすべてのBEノードに均等に分散させようとします。通常のテーブルのバランシングはレプリカレベルで行われ、各レプリカは個別に負荷の低いBEノードを見つけます。コロケーションテーブルのバランシングはバケットレベルで行われ、バケット内のすべてのレプリカが一緒に移動されます。私たちは、レプリカの実際のサイズではなく、レプリカの数のみを考慮して、`BucketsSequence`をすべてのBEノードに均等に分散する単純なバランシングアルゴリズムを使用します。正確なアルゴリズムは`ColocateTableBalancer.java`のコードコメントに記載されています。

> 注1: 現在のコロケーションレプリカのバランシングおよび修復アルゴリズムは、異種デプロイメントのStarRocksクラスターではうまく機能しない可能性があります。異種デプロイメントとは、BEノードのディスク容量、ディスク数、ディスクタイプ（SSDとHDD）が一致していない状況を指します。異種デプロイメントの場合、小容量のBEノードが大容量のBEノードと同じ数のレプリカを格納することがあります。
>
> 注2: グループがUnstable状態の場合、そのテーブルのJoinは通常のJoinに劣化し、クラスタのクエリパフォーマンスが大幅に低下する可能性があります。システムを自動的にバランスさせたくない場合は、FE設定の`disable_colocate_balance`を設定して自動バランシングを無効にし、適切なタイミングで再び有効にすることができます。（詳細はAdvanced Operations（#Advanced Operations）セクションを参照）

## クエリ

コロケーションテーブルは通常のテーブルと同様にクエリされます。コロケーションテーブルが配置されているグループがUnstable状態の場合、自動的に通常のJoinに劣化します。以下の例を参照してください。

表1:

~~~SQL
CREATE TABLE `tbl1` (
    `k1` date NOT NULL COMMENT "日付",
    `k2` int(11) NOT NULL COMMENT "整数",
    `v1` int(11) SUM NOT NULL COMMENT "合計"
) ENGINE=OLAP
AGGREGATE KEY(`k1`, `k2`)
PARTITION BY RANGE(`k1`)
(
    PARTITION p1 VALUES LESS THAN ('2019-05-31'),
    PARTITION p2 VALUES LESS THAN ('2019-06-30')
)
DISTRIBUTED BY HASH(`k2`)
PROPERTIES (
    "colocate_with" = "group1"
);
~~~

テーブル2:

~~~SQL
CREATE TABLE `tbl2` (
    `k1` datetime NOT NULL COMMENT "日時",
    `k2` int(11) NOT NULL COMMENT "整数",
    `v1` double SUM NOT NULL COMMENT "合計"
) ENGINE=OLAP
AGGREGATE KEY(`k1`, `k2`)
DISTRIBUTED BY HASH(`k2`)
PROPERTIES (
    "colocate_with" = "group1"
);
~~~

クエリプランの表示:

~~~Plain Text
DESC SELECT * FROM tbl1 INNER JOIN tbl2 ON (tbl1.k2 = tbl2.k2);

+----------------------------------------------------+
| Explain String                                     |
+----------------------------------------------------+
| PLAN FRAGMENT 0                                    |
|  OUTPUT EXPRS:`tbl1`.`k1` |                        |
|   PARTITION: RANDOM                                |
|                                                    |
|   RESULT SINK                                      |
|                                                    |
|   2:HASH JOIN                                      |
|   |  join op: INNER JOIN                           |
|   |  hash predicates:                              |
|   |  colocate: true                                |
|   |    `tbl1`.`k2` = `tbl2`.`k2`                   |
|   |  tuple ids: 0 1                                |
|   |                                                |
|   |----1:OlapScanNode                              |
|   |       TABLE: tbl2                              |
|   |       PREAGGREGATION: OFF. 理由: null          |
|   |       partitions=0/1                           |
|   |       rollup: null                             |
|   |       buckets=0/0                              |
|   |       cardinality=-1                           |
|   |       avgRowSize=0.0                           |
|   |       numNodes=0                               |
|   |       tuple ids: 1                             |
|   |                                                |
|   0:OlapScanNode                                   |
|      TABLE: tbl1                                   |
|      PREAGGREGATION: OFF. 理由: No AggregateInfo   |
|      partitions=0/2                                |
|      rollup: null                                  |
|      buckets=0/0                                   |
|      cardinality=-1                                |
|      avgRowSize=0.0                                |
|      numNodes=0                                    |
|      tuple ids: 0                                  |
+----------------------------------------------------+
~~~

Colocate Joinが有効な場合、HASH JOINノードは `colocate: true` を表示します。

有効でない場合、クエリプランは以下の通りです:

~~~Plain Text
+----------------------------------------------------+
| Explain String                                     |
+----------------------------------------------------+
| PLAN FRAGMENT 0                                    |
|  OUTPUT EXPRS:`tbl1`.`k1` |                        |
|   PARTITION: RANDOM                                |
|                                                    |
|   RESULT SINK                                      |
|                                                    |
|   2:HASH JOIN                                      |
|   |  join op: INNER JOIN (BROADCAST)               |
|   |  hash predicates:                              |
|   |  colocate: false, 理由: group is not stable    |
|   |    `tbl1`.`k2` = `tbl2`.`k2`                   |
|   |  tuple ids: 0 1                                |
|   |                                                |
|   |----3:EXCHANGE                                  |
|   |       tuple ids: 1                             |
|   |                                                |
|   0:OlapScanNode                                   |
|      TABLE: tbl1                                   |
|      PREAGGREGATION: OFF. 理由: No AggregateInfo   |
|      partitions=0/2                                |
|      rollup: null                                  |
|      buckets=0/0                                   |
|      cardinality=-1                                |
|      avgRowSize=0.0                                |
|      numNodes=0                                    |
|      tuple ids: 0                                  |
|                                                    |
| PLAN FRAGMENT 1                                    |
|  OUTPUT EXPRS:                                     |
|   PARTITION: RANDOM                                |
|                                                    |
|   STREAM DATA SINK                                 |
|     EXCHANGE ID: 03                                |
|     UNPARTITIONED                                  |
|                                                    |
|   1:OlapScanNode                                   |
|      TABLE: tbl2                                   |
|      PREAGGREGATION: OFF. 理由: null               |
|      partitions=0/1                                |
|      rollup: null                                  |
|      buckets=0/0                                   |
|      cardinality=-1                                |
|      avgRowSize=0.0                                |
|      numNodes=0                                    |
|      tuple ids: 1                                  |
+----------------------------------------------------+
~~~

HASH JOINノードは対応する理由を表示します: `colocate: false, 理由: group is not stable`。同時にEXCHANGEノードが生成されます。

## 高度な操作

### FE設定項目

* **disable_colocate_relocate**

StarRocksのColocationレプリカの自動修復を無効にするかどうか。デフォルトはfalseで、これは有効であることを意味します。このパラメータはColocationテーブルのレプリカ修復のみに影響し、通常のテーブルには影響しません。

* **disable_colocate_balance**

StarRocksのColocationレプリカの自動バランシングを無効にするかどうか。デフォルトはfalseで、これは有効であることを意味します。このパラメータはColocationテーブルのレプリカバランシングのみに影響し、通常のテーブルには影響しません。

* **disable_colocate_join**

    この変数を変更することで、セッション粒度でColocate Joinを無効にすることができます。

* **disable_colocate_join**

    この変数を変更することでColocate Join機能を無効にすることができます。

### HTTP RESTful API

StarRocksは、Colocate Joinに関連するいくつかのHTTP RESTful APIを提供しており、Colocationグループの表示および変更に使用できます。

このAPIはFEに実装されており、`fe_host:fe_http_port`でADMIN権限を使用してアクセスできます。

1. クラスタのすべてのColocation情報を表示

    ~~~bash
    curl --location-trusted -u<username>:<password> 'http://<fe_host>:<fe_http_port>/api/colocate'  
    ~~~

    ~~~JSON
    // Json形式で内部Colocation情報を返します。
    {
        "colocate_meta": {
            "groupName2Id": {
                "g1": {
                    "dbId": 10005,
                    "grpId": 10008
                }
            },
            "group2Tables": {},
            "table2Group": {
                "10007": {
                    "dbId": 10005,
                    "grpId": 10008
                },
                "10040": {
                    "dbId": 10005,
                    "grpId": 10008
                }
            },
            "group2Schema": {
                "10005.10008": {
                    "groupId": {
                        "dbId": 10005,
                        "grpId": 10008
                    },
                    "distributionColTypes": [{
                        "type": "INT",
                        "len": -1,
                        "isAssignedStrLenInColDefinition": false,
                        "precision": 0,
                        "scale": 0
                    }],
                    "bucketsNum": 10,
                    "replicationNum": 2
                }
            },
            "group2BackendsPerBucketSeq": {
                "10005.10008": [
                    [10004, 10002],
                    [10003, 10002],
                    [10002, 10004],
                    [10003, 10002],
                    [10002, 10004],
                    [10003, 10002],
                    [10003, 10004],
                    [10003, 10004],
                    [10003, 10004],
                    [10002, 10004]
                ]
            },
            "unstableGroups": []
        },
        "status": "OK"
    }
    ~~~

2. グループを安定または不安定としてマーク

    ~~~bash
    # 安定としてマーク
    curl -XPOST --location-trusted -u<username>:<password> 'http://<fe_host>:<fe_http_port>/api/colocate/group_stable?db_id=<dbId>&group_id=<grpId>'
    # 不安定としてマーク
    curl -XPOST --location-trusted -u<username>:<password> 'http://<fe_host>:<fe_http_port>/api/colocate/group_unstable?db_id=<dbId>&group_id=<grpId>'
    ~~~

    結果が `200` を返した場合、グループは正常に安定または不安定としてマークされました。

3. グループのデータ分布を設定

    このインターフェースを使用して、グループのバケット配分を強制的に設定できます。

    `POST /api/colocate/bucketseq?db_id=10005&group_id=10008`

    `Body:`

    `[[10004,10002],[10003,10002],[10002,10004],[10003,10002],[10002,10004],[10003,10002],[10003,10004],[10003,10004],[10003,10004],[10002,10004]]`

    `returns: 200`

    ここで、`Body`はネストされた配列として表される `BucketsSequence` と、バケットスライスが配置されているBEのIDを意味します。

    > このコマンドを使用するためには、FEの設定 `disable_colocate_relocate` と `disable_colocate_balance` を true に設定し、システムによるコロケーションレプリカの自動修復とバランシングを無効にする必要があるかもしれません。そうでないと、変更がシステムによって自動的にリセットされてしまう可能性があります。
