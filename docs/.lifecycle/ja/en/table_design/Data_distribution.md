---
displayed_sidebar: English
---

# データ分散

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

テーブル作成時に適切なパーティショニングとバケッティングを設定することで、データ分散を均等に行うことができます。データ分散とは、特定のルールに従ってデータをサブセットに分割し、異なるノードに均等に分配することを意味します。これによりスキャンするデータ量を減らし、クラスターの並列処理能力をフルに活用することで、クエリパフォーマンスを向上させることができます。

> **注記**
>
> - テーブル作成時にデータ分散を指定した後、ビジネスシナリオのクエリパターンやデータ特性が変化した場合、v3.2 StarRocksでは[テーブル作成後のデータ分散関連プロパティの変更](#optimize-data-distribution-after-table-creation-since-32)をサポートしており、最新のビジネスシナリオにおけるクエリパフォーマンスの要件に対応できます。
> - v3.1以降、テーブル作成時やパーティション追加時にDISTRIBUTED BY句でバケッティングキーを指定する必要はありません。StarRocksはランダムバケッティングをサポートしており、データを全バケットにランダムに分散します。詳細は[ランダムバケッティング](#random-bucketing-since-v31)を参照してください。
> - v2.5.7以降、テーブル作成時やパーティション追加時にバケット数を手動で設定する必要はありません。StarRocksはバケット数(BUCKETS)を自動的に設定することができます。しかし、StarRocksが自動的にバケット数を設定した後のパフォーマンスが期待に応えられない場合、バケッティングメカニズムに精通しているなら、[バケット数を手動で設定することもできます](#set-the-number-of-buckets)。

## 分散方法

### 一般的な分散方法

現代の分散データベースシステムでは、以下の基本的な分散方法が一般的に使用されます：ラウンドロビン、レンジ、リスト、ハッシュ。

![データ分散方法](../assets/3.3.2-1.png)

- **ラウンドロビン**: データをサイクリックに異なるノードに分散します。
- **レンジ**: パーティションカラムの値の範囲に基づいて、データを異なるノードに分散します。図に示されているように、範囲[1-3]と[4-6]は異なるノードに対応しています。
- **リスト**: パーティションカラムの離散値に基づいて、データを異なるノードに分散します。例えば性別や都道府県などです。各離散値はノードにマッピングされ、複数の異なる値が同じノードにマッピングされることもあります。
- **ハッシュ**: ハッシュ関数に基づいて、データを異なるノードに分散します。

より柔軟なデータパーティショニングを実現するために、上記の分散方法のいずれかを単独で使用するだけでなく、特定のビジネス要件に基づいてこれらの方法を組み合わせることもできます。一般的な組み合わせには、ハッシュ+ハッシュ、レンジ+ハッシュ、ハッシュ+リストなどがあります。

### StarRocksでの分散方法

StarRocksは、データ分散方法を個別にも組み合わせても使用することをサポートしています。

> **注記**
>
> 一般的な分散方法に加えて、StarRocksはバケッティング設定を簡素化するランダム分散もサポートしています。

また、StarRocksは2レベルのパーティショニングとバケッティングによってデータを分散します。

- 第一レベルはパーティショニングです：テーブル内のデータはパーティション分割されることができます。サポートされるパーティショニング方法には、式パーティショニング、レンジパーティショニング、リストパーティショニングがあります。または、パーティショニングを使用しないことを選択できます（テーブル全体が1つのパーティションと見なされます）。
- 第二レベルはバケッティングです：パーティション内のデータはさらに小さなバケットに分散される必要があります。サポートされるバケッティング方法には、ハッシュバケッティングとランダムバケッティングがあります。

| **分散方法**   | **パーティショニングとバケッティング方法**                        | **説明**                                              |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ランダム分散       | ランダムバケッティング                                             | テーブル全体がパーティションと見なされます。テーブル内のデータは異なるバケットにランダムに分散されます。これはデフォルトのデータ分散方法です。 |
| ハッシュ分散         | ハッシュバケッティング                                               | テーブル全体がパーティションと見なされます。テーブル内のデータは、データのバケッティングキーのハッシュ値に基づいて、ハッシュ関数を使用して対応するバケットに分散されます。 |
| レンジ+ランダム分散 | <ol><li>式パーティショニングまたはレンジパーティショニング </li><li>ランダムバケッティング </li></ol> | <ol><li>テーブル内のデータは、パーティションカラムの値が属する範囲に基づいて対応するパーティションに分散されます。 </li><li>パーティション内のデータは異なるバケットにランダムに分散されます。 </li></ol> |
| レンジ+ハッシュ分散   | <ol><li>式パーティショニングまたはレンジパーティショニング</li><li>ハッシュバケッティング </li></ol> | <ol><li>テーブル内のデータは、パーティションカラムの値が属する範囲に基づいて対応するパーティションに分散されます。</li><li>パーティション内のデータは、データのバケッティングキーのハッシュ値に基づいてハッシュ関数を使用して対応するバケットに分散されます。 </li></ol> |
| リスト+ランダム分散  | <ol><li>式パーティショニングまたはリストパーティショニング</li><li>ランダムバケッティング </li></ol> | <ol><li>テーブル内のデータは、パーティションカラムの値が属するリストに基づいて対応するパーティションに分散されます。</li><li>パーティション内のデータは異なるバケットにランダムに分散されます。</li></ol> |
| リスト+ハッシュ分散    | <ol><li>式パーティショニングまたはリストパーティショニング</li><li>ハッシュバケッティング </li></ol> | <ol><li>テーブル内のデータは、パーティションカラムの値が属するリストに基づいてパーティション分割されます。</li><li>パーティション内のデータは、データのバケッティングキーのハッシュ値に基づいてハッシュ関数を使用して対応するバケットに分散されます。</li></ol> |

- **ランダム分散**

  テーブル作成時にパーティショニングとバケッティング方法を設定しない場合、デフォルトでランダム分散が使用されます。この分散方法は現在、Duplicate Keyテーブルの作成にのみ使用できます。

  ```SQL
  CREATE TABLE site_access1 (
      event_day DATE,
      site_id INT DEFAULT '10', 
      pv BIGINT DEFAULT '0',
      city_code VARCHAR(100),
      user_name VARCHAR(32) DEFAULT ''
  )
  DUPLICATE KEY (event_day, site_id, pv);
  -- パーティショニングとバケッティング方法が設定されていないため、デフォルトでランダム分散が使用されます。
  ```

- **ハッシュ分散**

  ```SQL
  CREATE TABLE site_access2 (
      event_day DATE,
      site_id INT DEFAULT '10',
      city_code SMALLINT,

      user_name VARCHAR(32) DEFAULT '',
      pv BIGINT SUM DEFAULT '0'
  )
  AGGREGATE KEY (event_day, site_id, city_code, user_name)
  -- バケット方式としてハッシュバケットを使用し、バケットキーを指定する必要があります。
  DISTRIBUTED BY HASH(event_day,site_id); 
  ```

- **範囲+ランダム分散**（この分散方法は現在、Duplicate Keyテーブルを作成するためにのみ使用できます。）

  ```SQL
  CREATE TABLE site_access3 (
      event_day DATE,
      site_id INT DEFAULT '10', 
      pv BIGINT DEFAULT '0' ,
      city_code VARCHAR(100),
      user_name VARCHAR(32) DEFAULT ''
  )
  DUPLICATE KEY(event_day,site_id,pv)
  -- パーティション方式として式のパーティションを使用し、時間関数式を設定します。
  -- 範囲パーティションも使用できます。
  PARTITION BY date_trunc('day', event_day);
  -- バケット方式が設定されていないため、デフォルトでランダムバケットが使用されます。
  ```

- **範囲+ハッシュ分散**

  ```SQL
  CREATE TABLE site_access4 (
      event_day DATE,
      site_id INT DEFAULT '10',
      city_code VARCHAR(100),
      user_name VARCHAR(32) DEFAULT '',
      pv BIGINT SUM DEFAULT '0'
  )
  AGGREGATE KEY(event_day, site_id, city_code, user_name)
  -- パーティション方式として式のパーティションを使用し、時間関数式を設定します。
  -- 範囲パーティションも使用できます。
  PARTITION BY date_trunc('day', event_day)
  -- バケット方式としてハッシュバケットを使用し、バケットキーを指定する必要があります。
  DISTRIBUTED BY HASH(event_day, site_id);
  ```

- **リスト+ランダム分散**（この分散方法は現在、Duplicate Keyテーブルを作成するためにのみ使用できます。）

  ```SQL
  CREATE TABLE t_recharge_detail1 (
      id bigint,
      user_id bigint,
      recharge_money decimal(32,2), 
      city varchar(20) not null,
      dt date not null
  )
  DUPLICATE KEY(id)
  -- パーティション方式として式のパーティションを使用し、パーティション列を指定します。
  -- リストパーティションも使用できます。
  PARTITION BY (city);
  -- バケット方式が設定されていないため、デフォルトでランダムバケットが使用されます。
  ```

- **リスト+ハッシュ分散**

  ```SQL
  CREATE TABLE t_recharge_detail2 (
      id bigint,
      user_id bigint,
      recharge_money decimal(32,2), 
      city varchar(20) not null,
      dt date not null
  )
  DUPLICATE KEY(id)
  -- パーティション方式として式のパーティションを使用し、パーティション列を指定します。
  -- リストパーティションも使用できます。
  PARTITION BY (city)
  -- バケット方式としてハッシュバケットを使用し、バケットキーを指定する必要があります。
  DISTRIBUTED BY HASH(city,id); 
  ```

#### パーティション分割

パーティション分割方式は、テーブルを複数のパーティションに分割します。パーティション分割は、パーティションキーに基づいてテーブルを異なる管理単位（パーティション）に分割するために主に使用されます。各パーティションには、バケット数、ホットデータとコールドデータのストレージ戦略、ストレージメディアのタイプ、レプリカ数などのストレージ戦略を設定できます。StarRocksでは、クラスタ内で異なるタイプのストレージメディアを使用できます。例えば、最新データをSSDに保存してクエリパフォーマンスを向上させ、履歴データをSATAハードドライブに保存してストレージコストを削減することができます。

| **パーティション分割方式**                   | **シナリオ**                                                    | **パーティションの作成方法**               |
| ------------------------------------- | ------------------------------------------------------------ | ------------------------------------------ |
| 式のパーティション分割（推奨） | 以前は自動パーティション分割と呼ばれていました。このパーティション分割方法は、より柔軟で使いやすく、連続する日付範囲や列挙値に基づくデータのクエリや管理など、ほとんどのシナリオに適しています。 | データロード中に自動的に作成されます |
| 範囲パーティション分割                    | 一般的なシナリオは、連続した日付／数値範囲に基づいて頻繁にクエリされ、管理される単純で順序付けられたデータを格納することです。例えば、特定のケースでは、履歴データを月単位でパーティション分割し、最新データを日単位でパーティション分割する必要があります。 | 手動で、動的に、またはバッチで作成されます |
| リストパーティション分割                     | 一般的なシナリオは、列挙値に基づいてデータのクエリと管理を行い、パーティション分割列ごとに異なる値を持つデータを含む必要がある場合です。例えば、国や都市に基づいてデータのクエリと管理を頻繁に行う場合、この方法を使用し、パーティション分割列として`city`を選択できます。そうすることで、パーティションは同じ国に属する複数の都市のデータを格納できます。 | 手動で作成されます                           |

##### パーティション分割列と粒度の選択方法

- 適切なパーティション分割列を選択することで、クエリ中にスキャンされるデータ量を効果的に減らすことができます。多くのビジネスシステムでは、期限切れデータの削除によって生じる問題を解決し、ホットデータとコールドデータの階層的なストレージ管理を容易にするために、時間に基づくパーティション分割が一般的に採用されています。この場合、式のパーティション分割または範囲パーティション分割を使用し、時間列をパーティション分割列として指定できます。また、データが列挙値に基づいて頻繁にクエリされ、管理される場合は、式のパーティション分割またはリストパーティション分割を使用し、これらの値を含む列をパーティション分割列として指定できます。
- パーティション分割の粒度を選択する際には、データ量、クエリパターン、データ管理の粒度などの要因を考慮する必要があります。
  - 例1: テーブルの月間データ量が少ない場合、月単位でパーティション分割することで、日単位でパーティション分割する場合と比較してメタデータの量を減らすことができ、メタデータの管理とスケジューリングのリソース消費を削減できます。
  - 例2: テーブルの月間データ量が多く、クエリが主に特定の日のデータを要求する場合、日単位でパーティション分割することで、クエリ中にスキャンされるデータ量を効果的に減らすことができます。
  - 例3: データが毎日期限切れになる必要がある場合、日単位でパーティション分割することを推奨します。

#### バケット分割

バケット分割方式では、パーティションを複数のバケットに分割します。バケット内のデータはタブレットと呼ばれます。

サポートされているバケット分割方法は、[ランダムバケット分割](#random-bucketing-since-v31)（v3.1以降）と[ハッシュバケット分割](#hash-bucketing)です。

- ランダムバケット分割: テーブルを作成する際やパーティションを追加する際に、バケットキーを設定する必要はありません。パーティション内のデータはランダムに異なるバケットに分散されます。

- ハッシュバケット分割: テーブルを作成する際やパーティションを追加する際に、バケットキーを指定する必要があります。同じパーティション内のデータは、バケットキーの値に基づいてバケットに分割され、バケットキーで同じ値を持つ行は対応するユニークなバケットに分散されます。

バケット数: デフォルトでは、StarRocksは自動的にバケット数を設定します（v2.5.7以降）。また、バケット数を手動で設定することもできます。詳細については、[バケット数の設定](#set-the-number-of-buckets)を参照してください。

## パーティションの作成と管理

### パーティションの作成

#### 式のパーティション分割（推奨）

> **注意**
>

> v3.1以降、StarRocksの[共有データモード](../deployment/shared_data/s3.md)は時間関数式をサポートしていますが、カラム式はサポートしていません。

v3.0以降、StarRocksは[式によるパーティショニング](./expression_partitioning.md)（以前は自動パーティショニングと呼ばれていました）をサポートしており、これはより柔軟で使いやすいです。このパーティショニング方法は、連続する日付範囲や列挙値に基づいてデータをクエリや管理するなど、多くのシナリオに適しています。

テーブル作成時にパーティション式（時間関数式またはカラム式）を設定するだけで、StarRocksはデータロード時に自動的にパーティションを作成します。これにより、事前に多数のパーティションを手動で作成したり、動的パーティションのプロパティを設定したりする必要がなくなります。

#### 範囲パーティショニング

範囲パーティショニングは、時系列データ（日付やタイムスタンプ）や連続する数値データなど、シンプルで連続したデータを格納するのに適しています。また、連続する日付や数値の範囲に基づいてデータのクエリや管理を頻繁に行う場合にも適しています。さらに、履歴データを月単位で、最新データを日単位でパーティショニングする特殊なケースにも適用可能です。

StarRocksは、各パーティションの明確に定義された範囲に基づいてデータを対応するパーティションに格納します。

##### 動的パーティショニング

[動的パーティショニング](./dynamic_partitioning.md)に関連するプロパティはテーブル作成時に設定されます。StarRocksは自動的に新しいパーティションを事前に作成し、期限切れのパーティションを削除することでデータの新鮮さを保ち、パーティションのTTL（Time-To-Live）管理を実現します。

式によるパーティショニングが提供する自動的なパーティション作成機能とは異なり、動的パーティショニングはプロパティに基づいて定期的に新しいパーティションを作成するだけです。新しいデータがこれらのパーティションに属さない場合、ロードジョブにエラーが返されます。しかし、式によるパーティショニングが提供する自動的なパーティション作成機能は、ロードされたデータに基づいて常に対応する新しいパーティションを作成することができます。

##### 手動でパーティションを作成する

適切なパーティションキーを使用することで、クエリ時にスキャンするデータ量を効果的に削減できます。現在、日付型または整数型のカラムのみがパーティションキーを構成するパーティションカラムとして選択できます。ビジネスシナリオでは、パーティションキーは通常、データ管理の観点から選ばれます。一般的なパーティショニングカラムには、日付や地域を表すカラムが含まれます。

```SQL
CREATE TABLE site_access(
    event_day DATE,
    site_id INT DEFAULT '10',
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(event_day, site_id, city_code, user_name)
PARTITION BY RANGE(event_day)(
    PARTITION p1 VALUES LESS THAN ("2020-01-31"),
    PARTITION p2 VALUES LESS THAN ("2020-02-29"),
    PARTITION p3 VALUES LESS THAN ("2020-03-31")
)
DISTRIBUTED BY HASH(site_id);
```

##### バッチで複数のパーティションを作成する

テーブル作成時および作成後に、バッチで複数のパーティションを作成することができます。バッチで作成されるすべてのパーティションの開始時刻と終了時刻を`START()`と`END()`で指定し、パーティションの増分値を`EVERY()`で指定します。ただし、パーティションの範囲は右側半開区間であり、開始時刻は含まれますが終了時刻は含まれないことに注意してください。パーティションの命名規則は動的パーティショニングのそれと同様です。

- **テーブル作成時に日付型カラム（DATEおよびDATETIME）でテーブルをパーティション分割する**

  パーティショニングカラムが日付型の場合、テーブル作成時に`START()`と`END()`を使用してバッチで作成されるすべてのパーティションの開始日と終了日を指定し、`EVERY(INTERVAL xxx)`で2つのパーティション間の増分間隔を指定できます。現在、間隔の粒度は`HOUR`（v3.0以降）、`DAY`、`WEEK`、`MONTH`、および`YEAR`をサポートしています。

  次の例では、バッチで作成されるすべてのパーティションの日付範囲は2021-01-01から始まり2021-01-04で終わり、増分間隔は1日です。

    ```SQL
  CREATE TABLE site_access (
      datekey DATE,
      site_id INT,
      city_code SMALLINT,
      user_name VARCHAR(32),
      pv BIGINT DEFAULT '0'
  )
  ENGINE=olap
  DUPLICATE KEY(datekey, site_id, city_code, user_name)
  PARTITION BY RANGE (datekey) (
      START ("2021-01-01") END ("2021-01-04") EVERY (INTERVAL 1 DAY)
  )
  DISTRIBUTED BY HASH(site_id)
  PROPERTIES ("replication_num" = "3" );
    ```

  これは、CREATE TABLE文で次の`PARTITION BY`句を使用するのと同等です。

    ```SQL
  PARTITION BY RANGE (datekey) (
  PARTITION p20210101 VALUES [('2021-01-01'), ('2021-01-02')),
  PARTITION p20210102 VALUES [('2021-01-02'), ('2021-01-03')),
  PARTITION p20210103 VALUES [('2021-01-03'), ('2021-01-04'))
  )
    ```

- **テーブル作成時に異なる日付間隔で日付型カラム（DATEおよびDATETIME）でテーブルをパーティション分割する**

  `EVERY`で指定された異なる増分間隔を持つ日付パーティションのバッチを作成することができます（異なるバッチ間でパーティション範囲が重複しないようにしてください）。各バッチのパーティションは`START(xxx) END(xxx) EVERY(xxx)`句に従って作成されます。例えば：

    ```SQL
  CREATE TABLE site_access(
      datekey DATE,
      site_id INT,
      city_code SMALLINT,
      user_name VARCHAR(32),
      pv BIGINT DEFAULT '0'
  )
  ENGINE=olap
  DUPLICATE KEY(datekey, site_id, city_code, user_name)
  PARTITION BY RANGE (datekey) 
  (
      START ("2019-01-01") END ("2021-01-01") EVERY (INTERVAL 1 YEAR),
      START ("2021-01-01") END ("2021-05-01") EVERY (INTERVAL 1 MONTH),
      START ("2021-05-01") END ("2021-05-04") EVERY (INTERVAL 1 DAY)
  )
  DISTRIBUTED BY HASH(site_id)
  PROPERTIES(
      "replication_num" = "3"
  );
    ```

  これは、CREATE TABLE文で次の`PARTITION BY`句を使用するのと同等です。

    ```SQL
  PARTITION BY RANGE (datekey) (
  PARTITION p2019 VALUES [('2019-01-01'), ('2020-01-01')),
  PARTITION p2020 VALUES [('2020-01-01'), ('2021-01-01')),
  PARTITION p202101 VALUES [('2021-01-01'), ('2021-02-01')),
  PARTITION p202102 VALUES [('2021-02-01'), ('2021-03-01')),
  PARTITION p202103 VALUES [('2021-03-01'), ('2021-04-01')),
  PARTITION p202104 VALUES [('2021-04-01'), ('2021-05-01')),
  PARTITION p20210501 VALUES [('2021-05-01'), ('2021-05-02')),
  PARTITION p20210502 VALUES [('2021-05-02'), ('2021-05-03')),
  PARTITION p20210503 VALUES [('2021-05-03'), ('2021-05-04'))
  )
    ```

- **テーブル作成時に整数型カラムでテーブルをパーティション分割する**

  パーティショニングカラムがINT型の場合、`START`と`END`でパーティションの範囲を指定し、`EVERY`で増分値を定義します。例：

  > **注記**
  >

  > **START()** および **END()** のパーティション列の値は二重引用符で囲む必要がありますが、**EVERY()** の増分値は二重引用符で囲む必要はありません。

  次の例では、すべてのパーティションの範囲が `1` から始まり、`5` で終わり、パーティションの増分は `1` です。

    ```SQL
  CREATE TABLE site_access (
      datekey INT,
      site_id INT,
      city_code SMALLINT,
      user_name VARCHAR(32),
      pv BIGINT DEFAULT '0'
  )
  ENGINE=olap
  DUPLICATE KEY(datekey, site_id, city_code, user_name)
  PARTITION BY RANGE (datekey) (START("1") END("5") EVERY(1))
  DISTRIBUTED BY HASH(site_id)
  PROPERTIES ("replication_num" = "3");
    ```

  これは、CREATE TABLE ステートメントで次の `PARTITION BY` 句を使用するのと同じです。

    ```SQL
  PARTITION BY RANGE (datekey) (
  PARTITION p2019 VALUES [('2019-01-01'), ('2020-01-01')),
  PARTITION p2020 VALUES [('2020-01-01'), ('2021-01-01')),
  PARTITION p202101 VALUES [('2021-01-01'), ('2021-02-01')),
  PARTITION p202102 VALUES [('2021-02-01'), ('2021-03-01')),
  PARTITION p202103 VALUES [('2021-03-01'), ('2021-04-01')),
  PARTITION p202104 VALUES [('2021-04-01'), ('2021-05-01')),
  PARTITION p20210501 VALUES [('2021-05-01'), ('2021-05-02')),
  PARTITION p20210502 VALUES [('2021-05-02'), ('2021-05-03')),
  PARTITION p20210503 VALUES [('2021-05-03'), ('2021-05-04'))
  )
    ```

##### テーブル作成後にバッチで複数のパーティションを作成する

 テーブル作成後、ALTER TABLE ステートメントを使用してパーティションを追加することができます。構文は、テーブル作成時にバッチで複数のパーティションを作成するのと似ています。`ADD PARTITIONS` 句で `START`、`END`、および `EVERY` を設定する必要があります。

  ```SQL
  ALTER TABLE site_access 
  ADD PARTITIONS START("2021-01-04") END("2021-01-06") EVERY(INTERVAL 1 DAY);
  ```

#### リストパーティショニング (v3.1 以降)

[リストパーティショニング](./list_partitioning.md) は、列挙値に基づいてクエリを高速化し、データを効率的に管理するのに適しています。これは、パーティション分割列で異なる値を持つデータを含める必要があるシナリオで特に役立ちます。たとえば、国や市区町村に基づいてデータのクエリと管理を頻繁に行う場合、このパーティション分割方法を使用し、`city` 列をパーティション分割列として選択できます。この場合、1つのパーティションには、1つの国に属するさまざまな都市のデータを格納できます。

StarRocks は、各パーティションの事前定義された値リストの明示的なマッピングに基づいて、対応するパーティションにデータを格納します。

### パーティションの管理

#### パーティションの追加

範囲パーティショニングとリストパーティショニングでは、新しいパーティションを手動で追加して新しいデータを格納できます。しかし、式パーティショニングでは、データのロード中にパーティションが自動的に作成されるため、追加する必要はありません。

次のステートメントは、新しい月のデータを格納するためにテーブル `site_access` に新しいパーティションを追加します。

```SQL
ALTER TABLE site_access
ADD PARTITION p4 VALUES LESS THAN ("2020-04-30")
DISTRIBUTED BY HASH(site_id);
```

#### パーティションの削除

次のステートメントは、テーブル `site_access` からパーティション `p1` を削除します。

> **注記**
>
> この操作では、パーティション内のデータがすぐには削除されません。データは一定期間（デフォルトでは1日）ゴミ箱に保持されます。パーティションが誤って削除された場合、[RECOVER](../sql-reference/sql-statements/data-definition/RECOVER.md) コマンドを使用してパーティションとそのデータを復元できます。

```SQL
ALTER TABLE site_access
DROP PARTITION p1;
```

#### パーティションの復元

次のステートメントは、パーティション `p1` とそのデータをテーブル `site_access` に復元します。

```SQL
RECOVER PARTITION p1 FROM site_access;
```

#### パーティションの表示

次のステートメントは、テーブル `site_access` のすべてのパーティションの詳細を返します。

```SQL
SHOW PARTITIONS FROM site_access;
```

## バケッティングの設定

### ランダムバケッティング (v3.1 以降)

StarRocks は、パーティション内のデータをすべてのバケットにランダムに分散します。これは、データサイズが小さく、クエリ性能の要件が比較的低いシナリオに適しています。バケッティング方法を設定しない場合、StarRocks はデフォルトでランダムバケッティングを使用し、バケット数を自動的に設定します。

ただし、大量のデータをクエリし、特定の列をフィルタ条件として頻繁に使用する場合、ランダムバケッティングによって提供されるクエリ性能が最適でない可能性があることに注意してください。このようなシナリオでは、[ハッシュバケッティング](#hash-bucketing)の使用を推奨します。これらの列をクエリのフィルタ条件として使用すると、クエリがヒットする少数のバケット内のデータのみをスキャンして計算する必要があり、クエリ性能が大幅に向上する可能性があります。

#### 制限事項

- ランダムバケッティングは、Duplicate Key テーブルの作成にのみ使用できます。
- ランダムにバケッティングされたテーブルを[コロケーショングループ](../using_starrocks/Colocate_join.md)に指定することはできません。
- [Spark Load](../loading/SparkLoad.md) を使用して、ランダムにバケッティングされたテーブルにデータをロードすることはできません。

以下の CREATE TABLE の例では、`DISTRIBUTED BY xxx` ステートメントは使用されていないため、StarRocks はデフォルトでランダムバケッティングを使用し、バケット数を自動的に設定します。

```SQL
CREATE TABLE site_access1(
    event_day DATE,
    site_id INT DEFAULT '10', 
    pv BIGINT DEFAULT '0' ,
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT ''
)
DUPLICATE KEY(event_day, site_id, pv);
```

ただし、StarRocks のバケッティングメカニズムに精通している場合、ランダムバケッティングでテーブルを作成する際にバケット数を手動で設定することもできます。

```SQL
CREATE TABLE site_access2(
    event_day DATE,
    site_id INT DEFAULT '10', 
    pv BIGINT DEFAULT '0' ,
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT ''
)
DUPLICATE KEY(event_day, site_id, pv)
DISTRIBUTED BY RANDOM BUCKETS 8; -- バケット数を手動で8に設定
```

### ハッシュバケッティング

StarRocks は、ハッシュバケッティングを使用して、バケッティングキーと[バケット数](#set-the-number-of-buckets)に基づいてパーティション内のデータをバケットに細分化できます。ハッシュバケッティングでは、ハッシュ関数がデータのバケッティングキー値を入力として受け取り、ハッシュ値を計算します。データは、ハッシュ値とバケット間のマッピングに基づいて、対応するバケットに格納されます。

#### 利点

- クエリ性能の向上: 同じバケッティングキー値を持つ行は同じバケットに格納され、クエリ中にスキャンされるデータ量が削減されます。

- 均等なデータ分散: カーディナリティが高い（一意の値の数が多い）列をバケッティングキーとして選択することで、データをバケット間でより均等に分散できます。

#### バケッティング列の選択方法

バケッティング列として、次の2つの要件を満たす列を選択することを推奨します。


- IDなどの高カーディナリティ列
- クエリでフィルタとして頻繁に使用される列

しかし、両方の要件を満たす列がない場合、クエリの複雑さに応じてバケッティング列を決定する必要があります。

- クエリが複雑な場合、データが全バケットにできるだけ均等に分散されるように、高カーディナリティ列をバケッティング列として選択することを推奨します。これによりクラスタリソースの利用率が向上します。
- クエリが比較的シンプルな場合、クエリ効率を向上させるために、クエリでフィルタ条件として頻繁に使用される列をバケッティング列として選択することを推奨します。

パーティションデータを1つのバケッティング列を使用して全バケットに均等に分散できない場合、複数のバケッティング列を選択することができます。ただし、3列までを推奨します。

#### 注意事項

- **テーブル作成時には、バケッティング列を指定する必要があります**。
- バケッティング列のデータ型は、INTEGER、DECIMAL、DATE/DATETIME、またはCHAR/VARCHAR/STRINGでなければなりません。
- 3.2以降、テーブル作成後にALTER TABLEを使用してバケッティング列を変更することができます。

#### 例

以下の例では、`site_access`テーブルを作成し、バケッティング列として`site_id`を使用しています。さらに、`site_access`テーブルのデータをクエリする際、データはしばしばサイトによってフィルタリングされます。`site_id`をバケッティングキーとして使用することで、クエリ時に関連性のない多くのバケットを削減できます。

```SQL
CREATE TABLE site_access(
    event_day DATE,
    site_id INT DEFAULT '10',
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(event_day, site_id, city_code, user_name)
PARTITION BY RANGE(event_day)
(
    PARTITION p1 VALUES LESS THAN ("2020-01-31"),
    PARTITION p2 VALUES LESS THAN ("2020-02-29"),
    PARTITION p3 VALUES LESS THAN ("2020-03-31")
)
DISTRIBUTED BY HASH(site_id);
```

例えば、`site_access`テーブルの各パーティションに10個のバケットがあるとします。以下のクエリでは、10個のバケット中9個がプルーニングされるため、StarRocksは`site_access`テーブルのデータの1/10のみをスキャンする必要があります：

```SQL
select sum(pv)
from site_access
where site_id = 54321;
```

しかし、`site_id`が不均等に分散されており、多くのクエリが特定のサイトのデータのみを要求する場合、1つのバケッティング列のみを使用するとデータの偏りが生じ、システムパフォーマンスのボトルネックになる可能性があります。このような場合、バケッティング列の組み合わせを使用できます。例えば、以下のステートメントでは`site_id`と`city_code`をバケッティング列として使用しています。

```SQL
CREATE TABLE site_access
(
    site_id INT DEFAULT '10',
    city_code SMALLINT,
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(site_id, city_code, user_name)
DISTRIBUTED BY HASH(site_id, city_code);
```

実際には、ビジネスの特性に基づいて1つまたは2つのバケッティング列を使用できます。1つのバケッティング列`site_id`を使用することは、ノード間のデータ交換を減らし、クラスタの全体的なパフォーマンスを向上させるため、短いクエリに非常に有利です。一方で、2つのバケッティング列`site_id`と`city_code`を採用することは、分散クラスタの全体的な並行性を活用し、パフォーマンスを大幅に向上させるため、長いクエリに適しています。

> **注記**
>
> - 短いクエリは少量のデータをスキャンし、単一ノードで完了することができます。
> - 長いクエリは大量のデータをスキャンし、分散クラスタの複数ノードでの並列スキャンによりパフォーマンスが大幅に向上する可能性があります。

### バケット数の設定

バケットは、StarRocks内でデータファイルが実際にどのように整理されているかを反映しています。

#### テーブル作成時

- バケット数を自動的に設定する（推奨）

  v2.5.7以降、StarRocksはパーティションのマシンリソースとデータ量に基づいてバケット数を自動的に設定する機能をサポートしています。
  
  :::tip

  パーティションの生データサイズが100GBを超える場合は、方法2を使用してバケット数を手動で設定することを推奨します。

  :::

  <Tabs groupId="automaticexamples1">
  <TabItem value="example1" label="ハッシュバケッティングで構成されたテーブル" default>
  例：

  ```sql
  CREATE TABLE site_access (
      site_id INT DEFAULT '10',
      city_code SMALLINT,
      user_name VARCHAR(32) DEFAULT '',
      event_day DATE,
      pv BIGINT SUM DEFAULT '0')
  AGGREGATE KEY(site_id, city_code, user_name, event_day)
  PARTITION BY date_trunc('day', event_day)
  DISTRIBUTED BY HASH(site_id, city_code); -- バケット数を設定する必要はありません
  ```

  </TabItem>
  <TabItem value="example2" label="ランダムバケッティングで構成されたテーブル">
  ランダムバケッティングで構成されたテーブルについては、パーティション内のバケット数を自動的に設定することに加えて、v3.2.0以降のStarRocksはさらに最適化されたロジックを提供します。StarRocksは、クラスタの容量とロードされたデータの量に基づいて、データロード中にパーティション内のバケット数を動的に増やすことができます。

  :::warning

  - バケット数のオンデマンドかつ動的な増加を有効にするには、`PROPERTIES("bucket_size"="xxx")`を設定して単一バケットのサイズを指定する必要があります。パーティション内のデータ量が少ない場合は、`bucket_size`を1GBに設定できます。それ以外の場合は、`bucket_size`を4GBに設定できます。
  - バケット数のオンデマンドおよび動的な増加が有効になっていて、バージョン3.1にロールバックする必要がある場合は、まず動的な増加を可能にするテーブルを削除し、その後[ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md)を使用してメタデータのチェックポイントを手動で実行する必要があります。

  :::

  例：

  ```sql
  CREATE TABLE details1 (
      event_day DATE,
      site_id INT DEFAULT '10', 
      pv BIGINT DEFAULT '0',
      city_code VARCHAR(100),
      user_name VARCHAR(32) DEFAULT '')
  DUPLICATE KEY(event_day, site_id, pv)
  PARTITION BY date_trunc('day', event_day)
  -- パーティション内のバケット数はStarRocksによって自動的に決定され、バケットのサイズが1GBに設定されているため、需要に応じて動的に増加します。
  PROPERTIES("bucket_size"="1073741824")
  ;
  
  CREATE TABLE details2 (
      event_day DATE,
      site_id INT DEFAULT '10',
      pv BIGINT DEFAULT '0',
      city_code VARCHAR(100),
      user_name VARCHAR(32) DEFAULT '')
  DUPLICATE KEY(event_day, site_id, pv)
  PARTITION BY date_trunc('day', event_day)
  -- テーブルパーティション内のバケット数はStarRocksによって自動的に決定され、バケットのサイズが設定されていないため、固定されており需要に応じて動的には増加しません。
  ;
  ```

  </TabItem>
  </Tabs>

- バケット数を手動で設定する

  v2.4.0以降、StarRocksはクエリ時にタブレットを複数スレッドで並行スキャンすることをサポートしており、タブレット数に対するスキャンパフォーマンスの依存を軽減しています。各タブレットには約10GBの生データが含まれることを推奨します。バケット数を手動で設定する場合、テーブルの各パーティションのデータ量を見積もり、タブレット数を決定します。
  タブレットで並列スキャンを有効にするには、`enable_tablet_internal_parallel` パラメーターがシステム全体で `TRUE` に設定されていることを確認してください（`SET GLOBAL enable_tablet_internal_parallel = true;`）。

  <Tabs groupId="manualexamples1">
  <TabItem value="example1" label="ハッシュバケッティングで設定されたテーブル" default>

    ```SQL
    CREATE TABLE site_access (
        site_id INT DEFAULT '10',
        city_code SMALLINT,
        user_name VARCHAR(32) DEFAULT '',
        event_day DATE,
        pv BIGINT SUM DEFAULT '0')
    AGGREGATE KEY(site_id, city_code, user_name, event_day)
    PARTITION BY date_trunc('day', event_day)
    DISTRIBUTED BY HASH(site_id, city_code) BUCKETS 30;
    -- パーティションにロードしたい生データの量が300GBだと仮定します。
    -- 各タブレットに10GBの生データを含むことを推奨しているため、バケット数は30に設定できます。
    DISTRIBUTED BY HASH(site_id, city_code) BUCKETS 30;
    ```
  
  </TabItem>
  <TabItem value="example2" label="ランダムバケッティングで設定されたテーブル">

  ```sql
  CREATE TABLE details (
      site_id INT DEFAULT '10', 
      city_code VARCHAR(100),
      user_name VARCHAR(32) DEFAULT '',
      event_day DATE,
      pv BIGINT DEFAULT '0'
  )
  DUPLICATE KEY (site_id, city_code)
  PARTITION BY date_trunc('day', event_day)
  DISTRIBUTED BY RANDOM BUCKETS 30;
  ```

  </TabItem>
  </Tabs>

#### テーブル作成後

- バケット数を自動的に設定する（推奨）

  v2.5.7以降、StarRocksはパーティションのマシンリソースとデータ量に基づいてバケット数を自動的に設定する機能をサポートしています。

  :::tip
  
  パーティションの生データサイズが100GBを超える場合は、方法2を使用してバケット数を手動で設定することを推奨します。
  :::

  <Tabs groupId="automaticexamples2">
  <TabItem value="example1" label="ハッシュバケッティングで設定されたテーブル" default>

  ```sql
  -- すべてのパーティションのバケット数を自動的に設定します。
  ALTER TABLE site_access DISTRIBUTED BY HASH(site_id, city_code);
  
  -- 特定のパーティションのバケット数を自動的に設定します。
  ALTER TABLE site_access PARTITIONS (p20230101, p20230102)
  DISTRIBUTED BY HASH(site_id, city_code);
  
  -- 新しいパーティションのバケット数を自動的に設定します。
  ALTER TABLE site_access ADD PARTITION p20230106 VALUES [('2023-01-06'), ('2023-01-07'))
  DISTRIBUTED BY HASH(site_id, city_code);
  ```

  </TabItem>
  <TabItem value="example2" label="ランダムバケッティングで設定されたテーブル">

  ランダムバケッティングで設定されたテーブルについては、パーティション内のバケット数を自動的に設定することに加えて、StarRocksはv3.2.0以降、ロジックをさらに最適化しました。StarRocksは、クラスタの容量とロードされたデータの量に基づいて、**データのロード中に**パーティション内のバケット数を**動的に増やす**こともできます。これにより、パーティションの作成が容易になるだけでなく、一括読み込みのパフォーマンスも向上します。

  :::warning

  - バケット数のオンデマンドかつ動的な増加を有効にするには、`PROPERTIES("bucket_size"="xxx")` を設定して1つのバケットのサイズを指定する必要があります。パーティション内のデータ量が少ない場合は、`bucket_size` を1GBに設定できます。それ以外の場合は、`bucket_size` を4GBに設定できます。
  - バケット数のオンデマンドおよび動的な増加が有効になっていて、バージョン3.1にロールバックする必要がある場合は、バケット数の動的な増加を可能にするテーブルを最初に削除する必要があります。次に、ロールバックする前に、[ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) を使用してメタデータのチェックポイントを手動で実行する必要があります。

  :::

  ```sql
  -- すべてのパーティションのバケット数はStarRocksによって自動的に設定され、この数は固定されています。なぜなら、バケット数のオンデマンドかつ動的な増加は無効にされているからです。
  ALTER TABLE details DISTRIBUTED BY RANDOM;
  -- すべてのパーティションのバケット数はStarRocksによって自動的に設定され、バケット数のオンデマンドかつ動的な増加が有効になっています。
  ALTER TABLE details SET("bucket_size"="1073741824");
  
  -- 特定のパーティションのバケット数を自動的に設定します。
  ALTER TABLE details PARTITIONS (p20230103, p20230104)
  DISTRIBUTED BY RANDOM;
  
  -- 新しいパーティションのバケット数を自動的に設定します。
  ALTER TABLE details ADD PARTITION p20230106 VALUES [('2023-01-06'), ('2023-01-07'))
  DISTRIBUTED BY RANDOM;
  ```

  </TabItem>
  </Tabs>

- バケット数を手動で設定する

  バケット番号を手動で指定することもできます。パーティションのバケット数を計算するには、テーブル作成時にバケット数を手動で設定するときに使用する方法を参照できます（[上記の説明](#at-table-creation)）。

  <Tabs groupId="manualexamples2">
  <TabItem value="example1" label="ハッシュバケッティングで設定されたテーブル" default>

  ```sql
  -- すべてのパーティションのバケット数を手動で設定します。
  ALTER TABLE site_access
  DISTRIBUTED BY HASH(site_id, city_code) BUCKETS 30;
  -- 特定のパーティションのバケット数を手動で設定します。
  ALTER TABLE site_access
  PARTITIONS p20230104
  DISTRIBUTED BY HASH(site_id, city_code) BUCKETS 30;
  -- 新しいパーティションのバケット数を手動で設定します。
  ALTER TABLE site_access
  ADD PARTITION p20230106 VALUES [('2023-01-06'), ('2023-01-07'))
  DISTRIBUTED BY HASH(site_id, city_code) BUCKETS 30;
  ```

  </TabItem>
  <TabItem value="example2" label="ランダムバケッティングで設定されたテーブル">

  ```sql
  -- すべてのパーティションのバケット数を手動で設定します。
  ALTER TABLE details
  DISTRIBUTED BY RANDOM BUCKETS 30;
  -- 特定のパーティションのバケット数を手動で設定します。
  ALTER TABLE details
  PARTITIONS p20230104
  DISTRIBUTED BY RANDOM BUCKETS 30;
  -- 新しいパーティションのバケット数を手動で設定します。
  ALTER TABLE details
  ADD PARTITION p20230106 VALUES [('2023-01-06'), ('2023-01-07'))
  DISTRIBUTED BY RANDOM BUCKETS 30;
  ```

  動的パーティションのデフォルトのバケット数を手動で設定します。

  ```sql
  ALTER TABLE details_dynamic
  SET ("dynamic_partition.buckets"="xxx");
  ```

  </TabItem>
  </Tabs>

#### バケット数を表示する

テーブルを作成した後、[SHOW PARTITIONS](../sql-reference/sql-statements/data-manipulation/SHOW_PARTITIONS.md)を実行してStarRocksが各パーティションに設定したバケット数を確認できます。ハッシュバケッティングで構成されたテーブルの場合、各パーティションのバケット数は固定です。

:::info

- バケット数をオンデマンドで動的に増やすことができるランダムバケッティングで構成されたテーブルの場合、各パーティションのバケット数は動的に増加します。そのため、返される結果は各パーティションの現在のバケット数を表示します。
- このタイプのテーブルでは、パーティション内の実際の階層は次のとおりです：パーティション > サブパーティション > バケット。バケット数を増やすために、StarRocksは実際には特定の数のバケットを含む新しいサブパーティションを追加します。その結果、SHOW PARTITIONSステートメントは同じパーティション名を持つ複数のデータ行を返すことがあり、それらは同じパーティション内のサブパーティションの情報を示します。

:::

## テーブル作成後のデータ分散を最適化する（3.2以降）

> **注意**
>
> StarRocksの[共有データモード](../deployment/shared_data/s3.md)は現在、この機能をサポートしていません。

ビジネスシナリオでクエリパターンとデータ量が進化するにつれて、バケット化方法、バケット数、ソートキーなど、テーブル作成時に指定された設定が新しいビジネスシナリオに適さなくなり、クエリのパフォーマンスが低下する可能性があります。この場合、`ALTER TABLE`を使用してバケット化方法、バケット数、ソートキーを変更し、データ分散を最適化できます。例えば：

- **パーティション内のデータ量が大幅に増加した場合にバケット数を増やす**

  パーティション内のデータ量が以前よりも大幅に増加した場合、タブレットのサイズを一般的に1GBから10GBの範囲内に保つためにバケット数を変更する必要があります。
  
- **データの偏りを避けるためにバケットキーを変更する**
  現在のバケッティングキーがデータの偏りを引き起こす可能性がある場合（例えば、`k1`列のみがバケッティングキーとして設定されている場合）、より適切な列を指定するか、バケッティングキーに追加の列を加える必要があります。例えば：

  ```SQL
  ALTER TABLE t DISTRIBUTED BY HASH(k1, k2) BUCKETS 20;
  -- StarRocksのバージョンが3.1以降で、テーブルがDuplicate Keyテーブルの場合、システムのデフォルトバケッティング設定、つまりランダムバケッティングとStarRocksによって自動的に設定されるバケット数を直接使用することを検討できます。
  ALTER TABLE t DISTRIBUTED BY RANDOM;
  ```

- **クエリパターンの変更に伴うソートキーの適応**
  
  ビジネスクエリのパターンが大幅に変更され、追加の列が条件列として使用されるようになった場合、ソートキーを調整することが有益です。例えば：

  ```SQL
  ALTER TABLE t ORDER BY k2, k1;
  ```

詳細については、[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)を参照してください。