---
displayed_sidebar: Chinese
---
# データ分布

'@theme/Tabs'からタブをインポートします。
import TabItem from '@theme/TabItem';

テーブルを作成する際に、適切なパーティションとバケットを設定することで、データの均等な分布とクエリのパフォーマンス向上を実現することができます。データの均等な分布とは、データを一定のルールに従ってサブセットに分割し、異なるノードに均等に分散させることを指します。クエリの際にデータのスキャン量を効果的に削減し、クラスタの並列性能を最大限に活用してクエリのパフォーマンスを向上させることができます。

> **注意**
>
> - テーブル作成時にデータ分布を設定した後、ビジネスシナリオのクエリモードやデータ特性が変化した場合、3.2バージョン以降ではテーブル作成後に[データ分布に関連するプロパティを変更](#建表后优化数据分布自-32)することができます。
> - 3.1バージョン以降、テーブル作成および新しいパーティションの追加時に、バケットキー（DISTRIBUTED BY句）を設定する必要はありません。StarRocksはデフォルトでランダムなバケット分割を使用し、データをパーティション内のすべてのバケットにランダムに分散させます。詳細については、[ランダムなバケット分割](#随机分桶自-v31)を参照してください。
> - 2.5.7バージョン以降、テーブル作成および新しいパーティションの追加時に、バケット数（BUCKETS）を設定する必要はありません。StarRocksはデフォルトでバケット数を自動的に設定します。自動的に設定されたバケット数でパフォーマンスが期待通りにならない場合、およびバケットメカニズムに詳しい場合は、[バケット数の手動設定](#设置分桶数量)も行うことができます。

## データ分布の概要

### 一般的なデータ分布方法

現代の分散データベースでは、一般的なデータ分布方法として、Round-Robin、Range、List、およびHashがあります。以下の図に示すように：

![データ分布方法](../assets/3.3.2-1.png)

- Round-Robin：データを隣接するノードに順番に配置します。

- Range：範囲ごとにデータを分布させます。上の図では、範囲[1-3]、[4-6]がそれぞれ異なる範囲（Range）に対応しています。

- List：離散的な値に基づいてデータを分布させます。性別や都道府県などのデータはこの離散的な特性を満たしています。各離散値はノードにマッピングされ、異なる値が同じノードにマッピングされることもあります。

- Hash：ハッシュ関数を使用してデータを異なるノードにマッピングします。

これらのデータ分布方法のいずれかを単独で使用するだけでなく、具体的なビジネスシナリオの要件に応じてこれらのデータ分布方法を組み合わせて使用することもできます。一般的な組み合わせ方法には、Hash+Hash、Range+Hash、Hash+Listがあります。

### StarRocksのデータ分布方法

StarRocksは、単独または組み合わせてデータ分布方法をサポートしています。
> **注意**
>
> 一般的な分布方法に加えて、StarRocksはRandom分布もサポートしており、バケットの設定を簡素化することができます。

また、StarRocksはパーティション+バケットの設定によってデータの分布を実現しています。

- 第1層はパーティション：1つのテーブル内でパーティションを作成することができます。サポートされているパーティション方法には、式パーティション、Rangeパーティション、Listパーティション、またはパーティションなし（つまり、テーブル全体が1つのパーティションになる）があります。
- 第2層はバケット：1つのパーティション内では、バケットを作成する必要があります。サポートされているバケット方法には、ハッシュバケットとランダムバケットがあります。

| データ分布方法      | パーティションとバケットの方法                                               | 説明                                                         |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ランダム分布       | ランダムバケット                                                     | 1つのテーブルは1つのパーティションであり、テーブルのデータは異なるバケットにランダムに分散されます。これはデフォルトのデータ分布方法です。 |
| ハッシュ分布         | ハッシュバケット                                                     | 1つのテーブルは1つのパーティションであり、テーブルのデータのバケットキー値に対してハッシュ関数を使用して計算し、対応するバケットに分散させます。 |
| 範囲+ランダム分布 | <ol><li>式パーティションまたはRangeパーティション</li><li>ランダムバケット</li></ol> | <ol><li>テーブルのデータはパーティション列の値の範囲に基づいて対応するパーティションに分散されます。</li><li>同じパーティション内のデータは異なるバケットにランダムに分散されます。</li></ol> |
| 範囲+ハッシュ分布   | <ol><li>式パーティションまたはRangeパーティション</li><li>ハッシュバケット</li></ol> | <ol><li>テーブルのデータはパーティション列の値の範囲に基づいて対応するパーティションに分散されます。</li><li>同じパーティション内のデータのバケットキー値に対してハッシュ関数を使用して計算し、対応するバケットに分散させます。</li></ol> |
| リスト+ランダム分布  | <ol><li>式パーティションまたはListパーティション</li><li>ランダムバケット</li></ol>  | <ol><li>テーブルのデータはパーティション列の値の列挙値リストに基づいて対応するパーティションに分散されます。</li><li>同じパーティション内のデータは異なるバケットにランダムに分散されます。</li></ol> |
| リスト+ ハッシュ分布   | <ol><li>式パーティションまたはListパーティション</li><li>ハッシュバケット</li></ol>  | <ol><li>テーブルのデータはパーティション列の値の列挙値リストに基づいて対応するパーティションに分散されます。</li><li>同じパーティション内のデータのバケットキー値に対してハッシュ関数を使用して計算し、対応するバケットに分散させます。</li></ol> |

- ランダム分布

  パーティションとバケットの方法を設定せずにテーブルを作成すると、デフォルトでランダム分布が使用されます。

    ```SQL
    CREATE TABLE site_access1 (
        event_day DATE,
        site_id INT DEFAULT '10', 
        pv BIGINT DEFAULT '0' ,
        city_code VARCHAR(100),
        user_name VARCHAR(32) DEFAULT ''
    )
    DUPLICATE KEY (event_day,site_id,pv);
    -- パーティションとバケットの方法を設定せずに作成（現在は詳細モデルのテーブルのみサポート）
    ```

- ハッシュ分布

    ```SQL
    CREATE TABLE site_access2 (
        event_day DATE,
        site_id INT DEFAULT '10',
        city_code SMALLINT,
        user_name VARCHAR(32) DEFAULT '',
        pv BIGINT SUM DEFAULT '0'
    )
    AGGREGATE KEY (event_day, site_id, city_code, user_name)
    -- バケットの方法をハッシュバケットに設定し、バケットキーを指定する必要があります
    DISTRIBUTED BY HASH(event_day,site_id); 
    ```

- 範囲 + ランダム分布

    ```SQL
    CREATE TABLE site_access3 (
        event_day DATE,
        site_id INT DEFAULT '10', 
        pv BIGINT DEFAULT '0' ,
        city_code VARCHAR(100),
        user_name VARCHAR(32) DEFAULT ''
    )
    DUPLICATE KEY(event_day,site_id,pv)
    -- パーティションの方法を式パーティションに設定し、時間関数のパーティション式を使用します（もちろん、範囲パーティションに設定することもできます）
    PARTITION BY date_trunc('day', event_day);
    -- バケットの方法を設定せずに作成（現在は詳細モデルのテーブルのみサポート）
    ```

- 範囲 + ハッシュ分布

    ```SQL
    CREATE TABLE site_access4 (
        event_day DATE,
        site_id INT DEFAULT '10',
        city_code VARCHAR(100),
        user_name VARCHAR(32) DEFAULT '',
        pv BIGINT SUM DEFAULT '0'
    )
    AGGREGATE KEY(event_day, site_id, city_code, user_name)
    -- パーティションの方法を式パーティションに設定し、時間関数のパーティション式を使用します（もちろん、範囲パーティションに設定することもできます）
    PARTITION BY date_trunc('day', event_day)
    -- バケットの方法をハッシュバケットに設定し、バケットキーを指定する必要があります
    DISTRIBUTED BY HASH(event_day, site_id);
    ```

- リスト + ランダム分布

    ```SQL
    CREATE TABLE t_recharge_detail1 (
        id bigint,
        user_id bigint,
        recharge_money decimal(32,2), 
        city varchar(20) not null,
        dt date not null
    )
    DUPLICATE KEY(id)
    -- パーティションの方法を式パーティションに設定し、列パーティション式を使用します（もちろん、リストパーティションに設定することもできます）
    PARTITION BY (city);
    -- バケットの方法を設定せずに作成（現在は詳細モデルのテーブルのみサポート）
    ```

- リスト + ハッシュ分布

    ```SQL
    CREATE TABLE t_recharge_detail2 (
        id bigint,
        user_id bigint,
        recharge_money decimal(32,2), 
        city varchar(20) not null,
        dt date not null
    )
    DUPLICATE KEY(id)
    -- パーティションの方法を式パーティションに設定し、列パーティション式を使用します（もちろん、リストパーティションに設定することもできます）
    PARTITION BY (city)
    -- バケットの方法をハッシュバケットに設定し、バケットキーを指定する必要があります
    DISTRIBUTED BY HASH(city,id); 
    ```

#### パーティション

パーティションはデータを異なる範囲に分割するために使用されます。パーティションの主な目的は、1つのテーブルをパーティションキーに基づいて異なる管理単位に分割し、各管理単位に適切なストレージポリシー（バケット数、ホット/コールドポリシー、ストレージメディア、レプリカ数など）を選択することです。StarRocksは1つのクラスタ内で複数のストレージメディアを使用することができます。新しいデータをSSDディスクに配置して、優れたランダム読み書き性能を活用してクエリパフォーマンスを向上させ、古いデータをSATAディスクに保存してデータストレージコストを節約することができます。

| **パーティション方法**       | **適用シナリオ**                                                     | **パーティション作成方法**                                  |
| ------------------ | ------------------------------------------------------------ | --------------------------------------------- |
| 式パーティション（推奨） | 元々の自動作成パーティションで、ほとんどのシナリオに適しており、柔軟で使いやすいです。連続した日付範囲または列挙値に基づいてデータをクエリおよび管理するために使用されます。 | インポート時に自動作成                                |
| 範囲パーティション         | データが単純で順序があり、通常は連続した日付/数値範囲に基づいてクエリおよび管理されます。また、一部の特殊なシナリオでは、履歴データを月ごとにパーティション分割し、最新データを日ごとにパーティション分割する必要があります。 | 動的、バッチ、または手動作成 |
```
    建表後、ALTER TABLE文を使用してパーティションを一括作成することができます。関連する構文は、テーブル作成時のパーティション一括作成と同様で、ADD PARTITIONSキーワードを指定し、START、END、EVERY句を使用してパーティションを一括作成します。以下に例を示します：

    ```SQL
    ALTER TABLE site_access 
    ADD PARTITIONS START ("2021-01-04") END ("2021-01-06") EVERY (INTERVAL 1 DAY);
    ```

#### リストパーティション（v3.1以降）

[Listパーティション](./list_partitioning.md)は、列挙値に基づいてデータをクエリおよび管理するためのものです。特に、複数の値を含むパーティションに複数のパーティション列を含める必要がある場合に使用します。たとえば、国と都市でデータを頻繁にクエリおよび管理する場合、パーティション列として `city` を選択し、1つのパーティションに1つの国の複数の都市のデータを含めることができます。

StarRocksは、明示的に定義された列挙値リストとパーティションのマッピングに基づいてデータを対応するパーティションに割り当てます。

### パーティションの管理

#### パーティションの追加

RangeパーティションとListパーティションでは、新しいパーティションを手動で追加して新しいデータを保存することができます。一方、式パーティションでは、新しいデータをインポートする際に自動的にパーティションが作成されるため、手動でパーティションを追加する必要はありません。
新しいパーティションのデフォルトのバケット数は、元のパーティションと同じです。新しいパーティションのデータサイズに応じてバケット数を調整することもできます。

以下の例では、`site_access` テーブルに新しいパーティションを追加して新しい月のデータを保存しています：

```SQL
ALTER TABLE site_access
ADD PARTITION p4 VALUES LESS THAN ("2020-04-30")
DISTRIBUTED BY HASH(site_id);
```

#### パーティションの削除

次の文を実行すると、`site_access` テーブルからパーティション p1 とそのデータが削除されます：

> 注意：パーティション内のデータはすぐに削除されず、一定期間（デフォルトでは1日）Trashに保持されます。パーティションを誤って削除した場合は、[RECOVERコマンド](../sql-reference/sql-statements/data-definition/RECOVER.md)を使用してパーティションとデータを復元することができます。

```SQL
ALTER TABLE site_access
DROP PARTITION p1;
```

#### パーティションの復元

次の文を実行すると、`site_access` テーブルのパーティション `p1` とそのデータが復元されます：

```SQL
RECOVER PARTITION p1 FROM site_access;
```

#### パーティションの表示

次の文を実行すると、`site_access` テーブルのパーティションの状態を表示します：

```SQL
SHOW PARTITIONS FROM site_access;
```

## バケットの設定

### ランダムバケット（v3.1以降）

StarRocksは、各パーティションのデータをランダムにすべてのバケットに分散させるため、データ量が少なく、クエリのパフォーマンス要件が高くないシナリオに適しています。バケット方式を設定しない場合、StarRocksはランダムバケットを使用し、バケット数を自動的に設定します。

ただし、大量のデータをクエリし、頻繁に特定の列を条件として使用する場合、ランダムバケットは十分なクエリパフォーマンスを提供しない場合があります。このようなシナリオでは、[ハッシュバケット](#ハッシュバケット)を使用することをお勧めします。これにより、クエリ時には少数のバケットのみをスキャンおよび計算するため、クエリパフォーマンスが大幅に向上します。

**使用制限**

- 明細モデルテーブルのみをサポートします。
- [コロケーショングループ](../using_starrocks/Colocate_join.md)の指定はサポートされていません。
- [Spark Load](../loading/SparkLoad.md)はサポートされていません。

以下の例では、`DISTRIBUTED BY xxx` 文を使用せずに `site_access1` テーブルを作成しています。これにより、StarRocksはランダムバケットを使用し、バケット数を自動的に設定します。

```SQL
CREATE TABLE site_access1(
    event_day DATE,
    site_id INT DEFAULT '10', 
    pv BIGINT DEFAULT '0' ,
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT ''
)
DUPLICATE KEY(event_day,site_id,pv);
```

もちろん、StarRocksのバケットメカニズムに詳しい場合は、ランダムバケットを使用する場合でも、バケット数を手動で設定することもできます。

```SQL
CREATE TABLE site_access2(
    event_day DATE,
    site_id INT DEFAULT '10', 
    pv BIGINT DEFAULT '0' ,
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT ''
)
DUPLICATE KEY(event_day,site_id,pv)
DISTRIBUTED BY RANDOM BUCKETS 8; -- バケット数を8に手動設定
```

### ハッシュバケット

StarRocksは、各パーティションのデータをハッシュ関数を使用してハッシュ値を計算し、そのハッシュ値に基づいてデータを対応するバケットに割り当てます。

**利点**

- クエリパフォーマンスが向上します。同じハッシュキー値を持つ行は同じバケットに割り当てられるため、クエリ時にスキャンするデータ量が減ります。
- データが均等に分散されます。高基数（ユニークな値の数が多い）の列をハッシュキーとして選択することで、データを各バケットにより均等に分散させることができます。

**ハッシュキーの選択方法**

高基数かつ頻繁にクエリ条件として使用される列が存在する場合、それをハッシュキーとして選択し、ハッシュバケットを使用することをお勧めします。これらの条件を同時に満たす列が存在しない場合は、クエリに基づいて判断する必要があります。

- クエリが複雑な場合、データを各バケットに均等に分散させ、クラスタリソースの利用率を向上させるため、高基数の列をハッシュキーとして選択することをお勧めします。
- クエリが単純な場合、クエリの効率を向上させるため、頻繁にクエリ条件として使用される列をハッシュキーとして選択することをお勧めします。

また、データの偏りが深刻な場合は、データを均等に分散させるために複数の列をデータのハッシュキーとして使用することもできますが、3つ以上の列を使用することはお勧めしません。

**注意事項**

- **テーブル作成時にハッシュバケットを使用する場合、ハッシュキーを指定する必要があります**。
- ハッシュキーを構成する列は、整数型、DECIMAL型、DATE/DATETIME型、CHAR/VARCHAR/STRING型のみをサポートしています。
- 3.2以降、テーブル作成後にALTER TABLEを使用してハッシュキーを変更することができます。

以下の例では、`site_access` テーブルは `site_id` をハッシュキーとして使用しています。その理由は、`site_id` が高基数の列であるためです。また、`site_access` テーブルのクエリリクエストは、ほとんどがサイトをクエリのフィルタ条件として使用するため、`site_id` をハッシュキーとして選択することで、関係のないバケットを大幅に削減することができます。

```SQL
CREATE TABLE site_access(
    event_day DATE,
    site_id INT DEFAULT '10',
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(event_day, site_id, city_code, user_name)
PARTITION BY RANGE(event_day) (
    PARTITION p1 VALUES LESS THAN ("2020-01-31"),
    PARTITION p2 VALUES LESS THAN ("2020-02-29"),
    PARTITION p3 VALUES LESS THAN ("2020-03-31")
)
DISTRIBUTED BY HASH(site_id);
```

以下のクエリの場合、各パーティションに10個のバケットがあると仮定すると、9つのバケットが削減されるため、システムは `site_access` テーブルのデータの1/10のみをスキャンする必要があります。

```SQL
select sum(pv)
from site_access
where site_id = 54321;
```

ただし、`site_id` の分布が非常に偏っている場合、大量のアクセスデータが少数のウェブサイトに関連している場合（べき乗則の分布、80-20の法則）、上記のバケット方式を使用するとデータの分布が非常に偏ってしまい、システムの一部のパフォーマンスボトルネックが発生する可能性があります。この場合、パフォーマンスの問題を回避するために、バケットのフィールドを適切に調整する必要があります。たとえば、`site_id`、`city_code` の組み合わせをバケットのフィールドとして使用し、データをより均等に分散させることができます。関連するテーブル作成文は以下の通りです：

```SQL
CREATE TABLE site_access
(
    site_id INT DEFAULT '10',
    city_code SMALLINT,
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(site_id, city_code, user_name)
DISTRIBUTED BY HASH(site_id,city_code);
```

実際の使用では、ビジネスの特性に応じて上記の2つのバケット方式のいずれかを選択することができます。`site_id` のバケット方式は、短いクエリに非常に有利であり、ノード間のデータ交換を減らし、クラスタ全体のパフォーマンスを向上させることができます。`site_id`、`city_code` の組み合わせのバケット方式は、長いクエリに有利であり、分散クラスタの全体的な並列性能を活用することができます。

> 注意：
>
> - 短いクエリとは、データ量が少なく、単一のノードでスキャンが完了するクエリを指します。
>
> - 長いクエリとは、データ量が大きく、複数のノードで並列スキャンが大幅にパフォーマンスを向上させるクエリを指します。

### バケット数の設定

StarRocksでは、バケットは実際の物理ファイルの組織単位です。

#### テーブル作成時

- 自動設定（推奨）

  StarRocks 2.5.7以降、StarRocksはマシンリソースとデータ量に基づいて、パーティション内のバケット数を自動的に設定することができます。

  :::tip

  1つのパーティションの元のデータサイズが100 GBを超える場合、パーティション内のバケット数を手動で設定することをお勧めします。

  :::

  <Tabs groupId="automaticexamples1">
  <TabItem value="example1" label="ハッシュバケットテーブル" default>テーブルの作成例：
  
  ```sql
  CREATE TABLE site_access (
      site_id INT DEFAULT '10',
      city_code SMALLINT,
      user_name VARCHAR(32) DEFAULT '',
      event_day DATE,
      pv BIGINT SUM DEFAULT '0')
  AGGREGATE KEY(site_id, city_code, user_name,event_day)
  PARTITION BY date_trunc('day', event_day)
  DISTRIBUTED BY HASH(site_id,city_code); -- パーティション内のバケット数を手動で設定する必要はありません
  ```

  </TabItem>
  <TabItem value="example2" label="ランダムバケットテーブル">

  ランダムバケットテーブルについては、StarRocksはパーティション内のバケット数を自動的に設定するだけでなく、StarRocks 3.2以降では、**データをパーティションにインポートするプロセス中に**クラスタの能力とデータの量に基づいて**動的に増やす**こともサポートしています。これにより、テーブルの作成の使いやすさが向上するだけでなく、大規模なデータセットのインポートパフォーマンスも向上します。

  :::warning
```

以下は、元のドキュメントの翻訳結果です。修正が必要です。

    建表後、ALTER TABLE文を使用してパーティションを一括作成することができます。関連する構文は、テーブル作成時のパーティション一括作成と似ており、ADD PARTITIONSキーワードを指定し、START、END、EVERY句を使用してパーティションを一括作成します。以下に例を示します：

    ```SQL
    ALTER TABLE site_access 
    ADD PARTITIONS START ("2021-01-04") END ("2021-01-06") EVERY (INTERVAL 1 DAY);
    ```

#### リストパーティション（v3.1以降）

[Listパーティション](./list_partitioning.md)は、列挙値に基づいてデータをクエリおよび管理するためのものです。特に、1つのパーティションに複数のパーティション列の値を含める必要がある場合に適しています。たとえば、国と都市でデータを頻繁にクエリおよび管理する場合、パーティション列として`city`を選択し、1つのパーティションに1つの国の複数の都市のデータを含めることができます。

StarRocksは、明示的に定義された列挙値リストとパーティションのマッピングに基づいてデータを対応するパーティションに割り当てます。

### パーティションの管理

#### パーティションの追加

RangeパーティションとListパーティションでは、新しいパーティションを手動で追加して新しいデータを格納することができます。一方、式パーティションでは、新しいデータをインポートする際に自動的にパーティションが作成されるため、手動でパーティションを追加する必要はありません。
新しいパーティションのデフォルトのバケット数は、元のパーティションと同じです。新しいパーティションのデータサイズに応じてバケット数を調整することもできます。

以下の例では、`site_access`テーブルに新しいパーティションを追加して、新しい月のデータを格納しています：

```SQL
ALTER TABLE site_access
ADD PARTITION p4 VALUES LESS THAN ("2020-04-30")
DISTRIBUTED BY HASH(site_id);
```

#### パーティションの削除

次の文を実行すると、`site_access`テーブルからパーティションp1とそのデータが削除されます：

> 注意：パーティション内のデータはすぐに削除されず、一定期間（デフォルトで1日）Trashに保持されます。パーティションを誤って削除した場合は、[RECOVERコマンド](../sql-reference/sql-statements/data-definition/RECOVER.md)を使用してパーティションとデータを復元できます。

```SQL
ALTER TABLE site_access
DROP PARTITION p1;
```

#### パーティションの復元

次の文を実行すると、`site_access`テーブルのパーティションp1とそのデータが復元されます：

```SQL
RECOVER PARTITION p1 FROM site_access;
```

#### パーティションの表示

次の文を実行すると、`site_access`テーブルのパーティションの状態を表示します：

```SQL
SHOW PARTITIONS FROM site_access;
```

## バケットの設定

### ランダムバケット（v3.1以降）

StarRocksは、各パーティションのデータをランダムにすべてのバケットに分散させるランダムバケットを使用します。これは、データ量が少なく、クエリのパフォーマンス要件が高くないシナリオに適しています。バケット方式を設定しない場合、StarRocksはランダムバケットを使用し、バケット数を自動的に設定します。

ただし、大量のデータをクエリし、頻繁に特定の列を条件として使用する場合、ランダムバケットは十分なクエリパフォーマンスを提供しない場合があります。このようなシナリオでは、[ハッシュバケット](#ハッシュバケット)を使用することをお勧めします。これにより、クエリ時には少数のバケットのみをスキャンおよび計算するため、クエリパフォーマンスが大幅に向上します。

**使用制限**

- 明細モデルテーブルのみサポートされています。
- [Colocation Group](../using_starrocks/Colocate_join.md)を指定することはできません。
- [Spark Load](../loading/SparkLoad.md)はサポートされていません。

以下の例では、`DISTRIBUTED BY xxx`文を使用せずに`site_access1`テーブルを作成しています。これにより、StarRocksはランダムバケットを使用し、バケット数を自動的に設定します。

```SQL
CREATE TABLE site_access1(
    event_day DATE,
    site_id INT DEFAULT '10', 
    pv BIGINT DEFAULT '0' ,
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT ''
)
DUPLICATE KEY(event_day,site_id,pv);
```

もちろん、StarRocksのバケットメカニズムに精通している場合は、ランダムバケットを使用する場合でも、バケット数を手動で設定することができます。

```SQL
CREATE TABLE site_access2(
    event_day DATE,
    site_id INT DEFAULT '10', 
    pv BIGINT DEFAULT '0' ,
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT ''
)
DUPLICATE KEY(event_day,site_id,pv)
DISTRIBUTED BY RANDOM BUCKETS 8; -- バケット数を8に手動設定
```

### ハッシュバケット

StarRocksは、各パーティションのデータをハッシュバケットに基づいて分割します。ハッシュバケットでは、特定の列の値を入力として、ハッシュ関数を使用してハッシュ値を計算し、そのハッシュ値に基づいてデータを対応するバケットに割り当てます。

**利点**

- クエリのパフォーマンスが向上します。同じハッシュバケットキー値の行は、同じバケットに割り当てられるため、クエリ時のスキャンデータ量が減少します。
- データが均等に分布します。高基数（一意の値の数が多い）の列をハッシュバケットキーとして選択することで、データを各バケットにより均等に分散させることができます。

**ハッシュバケットキーの選択方法**

高基数かつ頻繁にクエリ条件として使用される列が存在する場合、それをハッシュバケットキーとして選択することをお勧めします。これらの条件を同時に満たす列が存在しない場合は、クエリに基づいて判断する必要があります。

- クエリが複雑な場合は、高基数の列をハッシュバケットキーとして選択し、データを各バケットに均等に分散させ、クラスタリソースの利用率を向上させることをお勧めします。
- クエリが単純な場合は、頻繁にクエリ条件として使用される列をハッシュバケットキーとして選択し、クエリの効率を向上させることをお勧めします。

また、データの偏りが深刻な場合は、データを均等に分散させるために複数の列をデータのハッシュバケットキーとして使用することもできますが、3つ以上の列を使用することはお勧めしません。

**注意事項**

- **テーブル作成時にハッシュバケットを使用する場合、ハッシュバケットキーを指定する必要があります**。
- ハッシュバケットキーを構成する列は、整数型、DECIMAL、DATE/DATETIME、CHAR/VARCHAR/STRINGデータ型のみサポートされています。
- 3.2以降、テーブル作成後にALTER TABLEを使用してハッシュバケットキーを変更することができます。

以下の例では、`site_access`テーブルは`site_id`をハッシュバケットキーとして使用しています。その理由は、`site_id`が高基数の列であるためです。また、`site_access`テーブルのクエリリクエストは、ほとんどがサイトをクエリのフィルタ条件として使用するため、`site_id`をハッシュバケットキーとして選択することで、関連のないバケットを大量にスキャンすることを回避できます。

```SQL
CREATE TABLE site_access(
    event_day DATE,
    site_id INT DEFAULT '10',
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(event_day, site_id, city_code, user_name)
PARTITION BY RANGE(event_day) (
    PARTITION p1 VALUES LESS THAN ("2020-01-31"),
    PARTITION p2 VALUES LESS THAN ("2020-02-29"),
    PARTITION p3 VALUES LESS THAN ("2020-03-31")
)
DISTRIBUTED BY HASH(site_id);
```

以下のクエリでは、各パーティションに10個のバケットがあると仮定した場合、9つのバケットが削減されるため、システムは`site_access`テーブルのデータの1/10のみをスキャンする必要があります。

```SQL
select sum(pv)
from site_access
where site_id = 54321;
```

ただし、`site_id`の分布が非常に均等でない場合、少数のウェブサイトに関する大量のアクセスデータ（べき乗則の分布、80-20の法則）がある場合、上記のバケット方式を使用するとデータの分布が非常に偏ってしまい、システムの一部の性能ボトルネックが発生する可能性があります。この場合、パフォーマンスの問題を回避するために、バケットのフィールドを適切に調整する必要があります。たとえば、`site_id`と`city_code`の組み合わせをバケットのフィールドとして使用し、データをより均等に分割することができます。関連するテーブル作成文は次のとおりです：

```SQL
CREATE TABLE site_access
(
    site_id INT DEFAULT '10',
    city_code SMALLINT,
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(site_id, city_code, user_name)
DISTRIBUTED BY HASH(site_id,city_code);
```

実際の使用では、ビジネスの特性に基づいて上記の2つのバケット方式のいずれかを選択することができます。`site_id`のバケット方式は、短いクエリに非常に有利であり、ノード間のデータ交換を減らし、クラスタ全体のパフォーマンスを向上させることができます。`site_id`と`city_code`の組み合わせのバケット方式は、長いクエリに有利であり、分散クラスタ全体の並列性能を活用することができます。

> 注意：
>
> - 短いクエリとは、データ量が少なく、単一のノードでスキャンが完了するクエリを指します。
>
> - 長いクエリとは、データ量が大きく、複数のノードで並列スキャンが性能を大幅に向上させるクエリを指します。

### バケット数の設定

StarRocksでは、バケットは実際の物理ファイルの組織単位です。

#### テーブル作成時

- 自動設定（推奨）

  StarRocksは、2.5.7以降、マシンリソースとデータ量に基づいてパーティション内のバケット数を自動的に設定する機能をサポートしています。

  :::tip

  単一のパーティションの元のデータサイズが100 GBを超える場合は、パーティション内のバケット数を手動で設定することをお勧めします。

  :::

  <Tabs groupId="automaticexamples1">
  <TabItem value="example1" label="ハッシュバケットテーブル" default>テーブル作成の例：
  
  ```sql
  CREATE TABLE site_access (
      site_id INT DEFAULT '10',
      city_code SMALLINT,
      user_name VARCHAR(32) DEFAULT '',
      event_day DATE,
      pv BIGINT SUM DEFAULT '0')
  AGGREGATE KEY(site_id, city_code, user_name,event_day)
  PARTITION BY date_trunc('day', event_day)
  DISTRIBUTED BY HASH(site_id,city_code); -- パーティション内のバケット数を手動で設定する必要はありません
  ```

  </TabItem>
  <TabItem value="example2" label="ランダムバケットテーブル">

  ランダムバケットテーブルに対して、StarRocksはパーティション内のバケット数を自動的に設定する機能をサポートしています。さらに、3.2以降のバージョンでは、データをパーティションにインポートするプロセス中に、クラスタの能力とデータ量に基づいてパーティション内のバケット数を**動的に増やす**機能も追加されました。これにより、テーブル作成の使いやすさが向上するだけでなく、大規模なデータセットのインポートパフォーマンスも向上します。

  :::warning
```

以下は、原文書の翻訳結果です。修正してください。

```markdown
    建表後,支持通过ALTER TABLE 语句批量创建分区。相关语法与建表时批量创建分区类似,通过指定 ADD PARTITIONS 关键字,以及 START、END 以及 EVERY 子句来批量创建分区。示例如下：

    ```SQL
    ALTER TABLE site_access 
    ADD PARTITIONS START ("2021-01-04") END ("2021-01-06") EVERY (INTERVAL 1 DAY);
    ```

#### List 分区(自 v3.1)

[List 分区](./list_partitioning.md)适用于按照枚举值来查询和管理数据。尤其适用于一个分区中需要包含各分区列的多个值。比如经常按照国家和城市来查询和管理数据，则可以使用该方式，选择分区列为 `city`，一个分区包含属于一个国家的多个城市的数据。

StarRocks 会按照您显式定义的枚举值列表与分区的映射关系将数据分配到相应的分区中。

### 管理分区

#### 增加分区

对于 Range 分区和 List 分区,您可以手动增加新的分区,用于存储新的数据,而表达式分区可以实现导入新数据时自动创建分区,您无需手动新增分区。
新增分区的默认分桶数量和原分区相同。您也可以根据新分区的数据规模调整分桶数量。

如下示例中,在 `site_access` 表添加新的分区,用于存储新月份的数据:

```SQL
ALTER TABLE site_access
ADD PARTITION p4 VALUES LESS THAN ("2020-04-30")
DISTRIBUTED BY HASH(site_id);
```

#### 删除分区

执行如下语句,删除 `site_access` 表中分区 p1 及数据:

> 说明:分区中的数据不会立即删除,会在 Trash 中保留一段时间(默认为一天)。如果误删分区,可以通过 [RECOVER 命令](../sql-reference/sql-statements/data-definition/RECOVER.md)恢复分区及数据。

```SQL
ALTER TABLE site_access
DROP PARTITION p1;
```

#### 恢复分区

执行如下语句,恢复 `site_access` 表中分区 `p1` 及数据:

```SQL
RECOVER PARTITION p1 FROM site_access;
```

#### 查看分区

执行如下语句,查看 `site_access` 表中分区情况:

```SQL
SHOW PARTITIONS FROM site_access;
```

## 设置分桶

### 随机分桶(自 v3.1)

对每个分区的数据,StarRocks 将数据随机地分布在所有分桶中,适用于数据量不大,对查询性能要求不高的场景。如果您不设置分桶方式,则默认由 StarRocks 使用随机分桶,并且自动设置分桶数量。

不过值得注意的是，如果查询海量数据且查询时经常使用一些列会作为条件列，随机分桶提供的查询性能可能不够理想。在该场景下建议您使用[哈希分桶](#哈希分桶)，当查询时经常使用这些列作为条件列时，只需要扫描和计算查询命中的少量分桶，则可以显著提高查询性能。

**使用限制**

- 仅支持明细模型表。
- 不支持指定 [Colocation Group](../using_starrocks/Colocate_join.md)。
- 不支持 [Spark Load](../loading/SparkLoad.md)。

如下建表示例中,没有使用 `DISTRIBUTED BY xxx` 语句,即表示默认由 StarRocks 使用随机分桶,并且由 StarRocks 自动设置分桶数量。

```SQL
CREATE TABLE site_access1(
    event_day DATE,
    site_id INT DEFAULT '10', 
    pv BIGINT DEFAULT '0' ,
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT ''
)
DUPLICATE KEY(event_day,site_id,pv);
```

当然,如果您比较熟悉 StarRocks 的分桶机制,使用随机分桶建表时,也可以手动设置分桶数量。

```SQL
CREATE TABLE site_access2(
    event_day DATE,
    site_id INT DEFAULT '10', 
    pv BIGINT DEFAULT '0' ,
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT ''
)
DUPLICATE KEY(event_day,site_id,pv)
DISTRIBUTED BY RANDOM BUCKETS 8; -- 手动设置分桶数量为 8
```

### 哈希分桶

对每个分区的数据,StarRocks 会根据分桶键和[分桶数量](#设置分桶数量)进行哈希分桶。在哈希分桶中，使用特定的列值作为输入，通过哈希函数计算出一个哈希值，然后将数据根据该哈希值分配到相应的桶中。

**优点**

- 提高查询性能。相同分桶键值的行会被分配到一个分桶中，在查询时能减少扫描数据量。
- 均匀分布数据。通过选取较高基数（唯一值的数量较多）的列作为分桶键，能更均匀的分布数据到每一个分桶中。

**如何选择分桶键**

假设存在列同时满足高基数和经常作为查询条件，则建议您选择其为分桶键，进行哈希分桶。如果不存在这些同时满足两个条件的列，则需要根据查询进行判断。

- 如果查询比较复杂，则建议选择高基数的列为分桶键，保证数据在各个分桶中尽量均衡，提高集群资源利用率。
- 如果查询比较简单，则建议选择经常作为查询条件的列为分桶键，提高查询效率。

并且，如果数据倾斜情况严重，您还可以使用多个列作为数据的分桶键，但是建议不超过 3 个列。

**注意事项**

- **建表时,如果使用哈希分桶,则必须指定分桶键**。
- 组成分桶键的列仅支持整型、DECIMAL、DATE/DATETIME、CHAR/VARCHAR/STRING 数据类型。
- 自 3.2 起,建表后支持通过 ALTER TABLE 修改分桶键。

如下示例中, `site_access` 表采用 `site_id` 作为分桶键,其原因在于 为 `site_id` 高基数列。此外，针对 `site_access` 表的查询请求，基本上都以站点作为查询过滤条件，采用 `site_id` 作为分桶键，还可以在查询时裁剪掉大量无关分桶。

```SQL
CREATE TABLE site_access(
    event_day DATE,
    site_id INT DEFAULT '10',
    city_code VARCHAR(100),
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(event_day, site_id, city_code, user_name)
PARTITION BY RANGE(event_day) (
    PARTITION p1 VALUES LESS THAN ("2020-01-31"),
    PARTITION p2 VALUES LESS THAN ("2020-02-29"),
    PARTITION p3 VALUES LESS THAN ("2020-03-31")
)
DISTRIBUTED BY HASH(site_id);
```

如下查询中,假设每个分区有 10 个分桶,则其中 9 个分桶被裁减,因而系统只需要扫描 `site_access` 表中 1/10 的数据:

```SQL
select sum(pv)
from site_access
where site_id = 54321;
```

但是如果 `site_id` 分布十分不均匀，大量的访问数据是关于少数网站的（幂律分布，二八规则），那么采用上述分桶方式会造成数据分布出现严重的倾斜，进而导致系统局部的性能瓶颈。此时，您需要适当调整分桶的字段，以将数据打散，避免性能问题。例如,可以采用 `site_id`、`city_code` 组合作为分桶键,将数据划分得更加均匀。相关建表语句如下:

```SQL
CREATE TABLE site_access
(
    site_id INT DEFAULT '10',
    city_code SMALLINT,
    user_name VARCHAR(32) DEFAULT '',
    pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(site_id, city_code, user_name)
DISTRIBUTED BY HASH(site_id,city_code);
```

在实际使用中，您可以依据自身的业务特点选择以上两种分桶方式。采用 `site_id` 的分桶方式对于短查询十分有利，能够减少节点之间的数据交换，提高集群整体性能；采用 `site_id`、`city_code` 的组合分桶方式对于长查询有利，能够利用分布式集群的整体并发性能。

> 说明：
>
> - 短查询是指扫描数据量不大、单机就能完成扫描的查询。
>
> - 长查询是指扫描数据量大、多机并行扫描能显著提升性能的查询。

### 设置分桶数量

在 StarRocks 中,分桶は実際の物理ファイルの組織単位です。

#### テーブル作成時

- 自動設定（推奨）

  StarRocksは、2.5.7以降、マシンリソースとデータ量に基づいてパーティション内のバケット数を自動的に設定する機能をサポートしています。

  :::tip

  単一のパーティションの元のデータサイズが100 GBを超える場合は、パーティション内のバケット数を手動で設定することをお勧めします。

  :::

  <Tabs groupId="automaticexamples1">
  <TabItem value="example1" label="ハッシュバケットテーブル" default>テーブル作成の例：
  
  ```sql
  CREATE TABLE site_access (
      site_id INT DEFAULT '10',
      city_code SMALLINT,
      user_name VARCHAR(32) DEFAULT '',
      event_day DATE,
      pv BIGINT SUM DEFAULT '0')
  AGGREGATE KEY(site_id, city_code, user_name,event_day)
  PARTITION BY date_trunc('day', event_day)
  DISTRIBUTED BY HASH(site_id,city_code); -- パーティション内のバケット数を手動で設定する必要はありません
  ```

  </TabItem>
  <TabItem value="example2" label="ランダムバケットテーブル">

  ランダムバケットテーブルに対して、StarRocksはパーティション内のバケット数を自動的に設定する機能をサポートしています。さらに、3.2以降のバージョンでは、データをパーティションにインポートするプロセス中に、クラスタの能力とデータ量に基づいてパーティション内のバケット数を**動的に増やす**機能も追加されました。これにより、テーブル作成の使いやすさが向上するだけでなく、大規模なデータセットのインポートパフォーマンスも向上します。

  :::warning
  - 分桶数量を動的に増やす必要がある場合は、テーブルプロパティ `PROPERTIES("bucket_size"="xxx")` を設定して、各バケットのサイズを指定する必要があります。パーティションのデータ量が少ない場合は、`bucket_size` を 1 GB に設定し、データ量が多い場合は、`bucket_size` を 4 GB に設定できます。
  - 有効にした後、バージョン 3.1 にロールバックする必要がある場合は、動的にバケット数を増やす機能を有効にしたテーブルを削除し、メタデータのチェックポイント [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) を手動で実行し、成功した後にロールバックできます。

  :::

  テーブル作成例：

  ```sql
  CREATE TABLE details1 (
      event_day DATE,
      site_id INT DEFAULT '10', 
      pv BIGINT DEFAULT '0',
      city_code VARCHAR(100),
      user_name VARCHAR(32) DEFAULT '')
  DUPLICATE KEY (event_day,site_id,pv)
  PARTITION BY date_trunc('day', event_day)
  -- このテーブルのパーティション内のバケット数は StarRocks によって自動的に設定され、各バケットのサイズが 1 GB に指定されているため、必要に応じてバケット数が動的に増加します。
  PROPERTIES("bucket_size"="1073741824")
  ;
  
  CREATE TABLE details2 (
      event_day DATE,
      site_id INT DEFAULT '10',
      pv BIGINT DEFAULT '0' ,
      city_code VARCHAR(100),
      user_name VARCHAR(32) DEFAULT '')
  DUPLICATE KEY (event_day,site_id,pv)
  PARTITION BY date_trunc('day', event_day)
  -- このテーブルのパーティション内のバケット数は StarRocks によって自動的に設定され、各バケットのサイズが指定されていないため、バケット数は動的に増加しません。
  ;
  ```

  </TabItem>
  </Tabs>

- 手動設定

  バージョン 2.4 以降、StarRocks はタブレットの並列スキャン機能を提供しており、クエリに関連する任意のタブレットが複数のスレッドによって並列にスキャンされる可能性があり、タブレットの数がクエリ性能に与える影響を減少させ、パーティション内のバケット数の設定を簡素化できます。簡素化後、パーティション内のバケット数を決定する方法は、まず各パーティションのデータ量を推定し、次に 10 GB の原データごとに 1 つのタブレットとして計算し、そこからバケット数を決定します。

  タブレットの並列スキャンを有効にするには、システム変数 `enable_tablet_internal_parallel` をグローバルに有効にする必要があります `SET GLOBAL enable_tablet_internal_parallel = true;`。

  <Tabs groupId="manualexamples1">
  <TabItem value="example1" label="ハッシュバケットテーブル" default>

  ```sql
  CREATE TABLE site_access (
      site_id INT DEFAULT '10',
      city_code SMALLINT,
      user_name VARCHAR(32) DEFAULT '',
      event_day DATE,
      pv BIGINT SUM DEFAULT '0')
  AGGREGATE KEY(site_id, city_code, user_name,event_day)
  PARTITION BY date_trunc('day', event_day)
  DISTRIBUTED BY HASH(site_id,city_code) BUCKETS 30; -- 例えば、1つのパーティションに 300 GB の原データがあると仮定すると、10 GB の原データごとに 1 つのタブレットとして、パーティション内のバケット数は 30 に設定できます。
  ```

  </TabItem>
  <TabItem value="example2" label="ランダムバケットテーブル">

    ```sql
  CREATE TABLE details (
      site_id INT DEFAULT '10', 
      city_code VARCHAR(100),
      user_name VARCHAR(32) DEFAULT '',
      event_day DATE,
      pv BIGINT DEFAULT '0'
  )
  DUPLICATE KEY (site_id,city_code)
  PARTITION BY date_trunc('day', event_day)
  DISTRIBUTED BY RANDOM BUCKETS 30
  ; 
    ```

    </TabItem>
    </Tabs>

#### テーブル作成後

- 自動設定（推奨）

  バージョン 2.5.7 以降、StarRocks はマシンリソースとデータ量に基づいてパーティション内のバケット数を自動的に設定する機能をサポートしています。

  :::tip

  テーブルの単一パーティションの原データ規模が 100 GB を超えると予想される場合は、パーティション内のバケット数を手動で設定することをお勧めします。

  :::

  <Tabs groupId="automaticexamples2">
  <TabItem value="example1" label="ハッシュバケットテーブル" default>

    ```sql
  -- すべてのパーティションのバケット数を自動設定
  ALTER TABLE site_access DISTRIBUTED BY HASH(site_id,city_code);
  
  -- 特定のパーティションのバケット数を自動設定
  ALTER TABLE site_access PARTITIONS (p20230101, p20230102)
  DISTRIBUTED BY HASH(site_id,city_code);
  
  -- 新規パーティションのバケット数を自動設定
  ALTER TABLE site_access ADD PARTITION p20230106 VALUES [('2023-01-06'), ('2023-01-07'))
  DISTRIBUTED BY HASH(site_id,city_code);
    ```

    </TabItem>
    <TabItem value="example2" label="ランダムバケットテーブル">

  ランダムバケットテーブルについては、StarRocks はバージョン 3.2 以降、パーティション内のバケット数を自動的に設定する機能をさらに最適化し、**データのインポート中**にクラスタの能力とインポートデータ量に基づいて**必要に応じて**バケット数を動的に増やすことをサポートしました。

  :::warning

  - 動的にバケット数を増やす機能を有効にするには、テーブルプロパティ `PROPERTIES("bucket_size"="xxx")` を設定して、各バケットのサイズを指定する必要があります。パーティションのデータ量が少ない場合は、`bucket_size` を 1 GB に設定し、データ量が多い場合は、`bucket_size` を 4 GB に設定できます。
  - 有効にした後、バージョン 3.1 にロールバックする必要がある場合は、動的にバケット数を増やす機能を有効にしたテーブルを削除し、メタデータのチェックポイント [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) を手動で実行し、成功した後にロールバックできます。

  :::

    ```sql
  -- すべてのパーティションのバケット数を自動設定し、動的に増やす機能を開始しない場合、バケット数は固定されます。
  ALTER TABLE details DISTRIBUTED BY RANDOM;
  -- すべてのパーティションのバケット数を自動設定し、動的に増やす機能を開始します。
  ALTER TABLE details SET("bucket_size"="1073741824");
  
  -- 特定のパーティションのバケット数を自動設定
  ALTER TABLE details PARTITIONS (p20230103, p20230104)
  DISTRIBUTED BY RANDOM;
  
  -- 新規パーティションのバケット数を自動設定
  ALTER TABLE details ADD PARTITION  p20230106 VALUES [('2023-01-06'), ('2023-01-07'))
  DISTRIBUTED BY RANDOM;
    ```

    </TabItem>
    </Tabs>

- 手動設定

  パーティション内のバケット数を手動で指定します。バケット数の計算方法は上記の[テーブル作成時の手動設定](#テーブル作成時)を参照してください。

  <Tabs groupId="manualexamples2">
  <TabItem value="example1" label="ハッシュバケットテーブル" default>

  ```sql
  -- すべてのパーティションのバケット数を手動で指定
  ALTER TABLE site_access
  DISTRIBUTED BY HASH(site_id,city_code) BUCKETS 30;
  -- 特定のパーティションのバケット数を手動で指定
  ALTER TABLE site_access
  PARTITIONS p20230104
  DISTRIBUTED BY HASH(site_id,city_code)  BUCKETS 30;
  -- 新規パーティションのバケット数を手動で指定
  ALTER TABLE site_access
  ADD PARTITION p20230106 VALUES [('2023-01-06'), ('2023-01-07'))
  DISTRIBUTED BY HASH(site_id,city_code) BUCKETS 30;
  ```

  </TabItem>
  <TabItem value="example2" label="ランダムバケットテーブル">

  ```sql
  -- すべてのパーティションのバケット数を手動で指定
  ALTER TABLE details
  DISTRIBUTED BY RANDOM BUCKETS 30;
  -- 特定のパーティションのバケット数を手動で指定
  ALTER TABLE details
  PARTITIONS p20230104
  DISTRIBUTED BY RANDOM BUCKETS 30;
  -- 新規パーティションのバケット数を手動で指定
  ALTER TABLE details
  ADD PARTITION p20230106 VALUES [('2023-01-06'), ('2023-01-07'))
  DISTRIBUTED BY RANDOM BUCKETS 30;
  ```

  動的パーティションのデフォルトバケット数を手動で設定します。

  ```sql
  ALTER TABLE details_dynamic
  SET ("dynamic_partition.buckets"="xxx");
  ```

  </TabItem>
  </Tabs>

#### バケット数の確認

パーティション内のバケット数を確認するには、[SHOW PARTITIONS](../sql-reference/sql-statements/data-manipulation/SHOW_PARTITIONS.md) を実行します。

:::info

- ランダムバケットテーブルで動的にバケット数を増やす機能を有効にしている場合、テーブル作成後のデータインポート中にパーティションのバケット数は**動的に増加**します。結果として表示されるのはパーティションの**現在**のバケット数です。

- ランダムバケットテーブルの場合、パーティション内部の実際の分割レベルは：パーティション > サブパーティション > バケットです。バケット数を増やすために、StarRocks は実際には新しいサブパーティションを追加します。サブパーティションには一定数のバケットが含まれているため、[SHOW PARTITIONS](../sql-reference/sql-statements/data-manipulation/SHOW_PARTITIONS.md) の結果には、同じパーティション内のサブパーティションの状況を示す同じパーティション名の複数のデータ行が表示されます。

:::

## テーブル作成後のデータ分布の最適化（バージョン 3.2 以降）

> **注意**
>
> StarRocks の[ストレージコンピューティング分離モード](../deployment/shared_data/s3.md)は、この機能をまだサポートしていません。

ビジネスシナリオにおけるクエリパターンとデータ量の変化に伴い、テーブル作成時に設定されたバケット方式とバケット数、およびソートキーが新しいビジネスシナリオに適応できなくなり、クエリ性能が低下する可能性があります。この場合、`ALTER TABLE` を使用してバケット方式とバケット数、およびソートキーを調整し、データ分布を最適化できます。例えば：

- **パーティション内のデータ量が増加し、バケット数を増やす**

  日ごとに分割されたパーティションのデータ量が元のサイズよりも大幅に増加した場合、元のバケット数では不適切になる可能性があります。この場合、バケット数を増やして、各タブレットのサイズを一般的に 1 GB から 10 GB の範囲に制御できます。

- **データの偏りを避けるためにバケットキーを調整する**

  元のバケットキー（例えば `k1` のみ）がデータの偏りを引き起こす可能性がある場合、より適切な列を設定するか、バケットキーにさらに多くの列を追加することができます。以下のように：

    ```SQL
    ALTER TABLE t DISTRIBUTED BY HASH(k1, k2) BUCKETS 20;
    -- StarRocks のバージョンが 3.1 以上で、詳細モデルのテーブルを使用している場合は、デフォルトのバケット設定、つまりランダムバケットと StarRocks によるバケット数の自動設定に直接変更することをお勧めします。
    ALTER TABLE t DISTRIBUTED BY RANDOM;
    ```

- テーブルがプライマリキーモデルであり、ビジネスのクエリパターンに大きな変化があり、テーブル内の他のいくつかの列を条件列として頻繁に使用する必要がある場合は、ソートキーを調整できます。以下のように：

    ```SQL
    ```SQL
    ALTER TABLE t ORDER BY k2, k1;
    ```

詳細は [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) を参照してください。

## ベストプラクティス

StarRocks において、パーティションとバケットの選択は非常に重要です。テーブル作成時に適切なパーティションキーとバケットキーを選択することで、クラスター全体のパフォーマンスを効果的に向上させることができます。そのため、パーティションキーとバケットキーを選択する際には、ビジネスの状況に応じて調整することをお勧めします。

- **データの偏り**
  
  ビジネスシナリオで偏りの大きい単一の列をバケットに使用すると、データアクセスの偏りが大きくなる可能性があるため、複数の列を組み合わせてデータをバケットに分けることをお勧めします。

- **高並行性**
  
  パーティションとバケットは、クエリ文が持つ条件をできるだけカバーするようにすべきです。これにより、スキャンするデータ量を効果的に減らし、並行性を向上させることができます。

- **高スループット**
  
  データをできるだけ分散させ、クラスターがより高い並行性でデータをスキャンし、計算を完了させるようにしましょう。

- **メタデータ管理**

  Tablet の数が多すぎると、FE/BE のメタデータ管理とスケジューリングのリソース消費が増加します。
