---
displayed_sidebar: Chinese
---

# 非同期マテリアライズドビュー

本文では、StarRocks における非同期マテリアライズドビューの理解、作成、使用、管理方法について説明します。StarRocks はバージョン2.4から非同期マテリアライズドビューをサポートしています。

同期マテリアライズドビューと比較して、非同期マテリアライズドビューは複数のテーブルの結合やより豊富な集約演算子をサポートします。非同期マテリアライズドビューは手動呼び出しや定期的なタスクによって更新が可能であり、部分的なパーティションの更新にも対応しており、更新コストを大幅に削減できます。さらに、非同期マテリアライズドビューは複数のクエリ改写シナリオをサポートし、自動的で透過的なクエリの高速化を実現します。

同期マテリアライズドビュー（Rollup）のシナリオと使用については、[同期マテリアライズドビュー（Rollup）](../using_starrocks/Materialized_view-single_table.md)を参照してください。

## 背景紹介

データウェアハウス環境におけるアプリケーションは、しばしば複数の大規模なテーブルを基に複雑なクエリを実行します。これには通常、数十億行に及ぶデータの結合と集約が関わります。このようなクエリの処理は、システムリソースと時間を大量に消費し、非常に高いクエリコストを引き起こします。

StarRocks の非同期マテリアライズドビューを使用することで、上記の問題を解決できます。非同期マテリアライズドビューは、特定のクエリステートメントに基づいた事前計算結果が格納された特殊な物理テーブルです。ベーステーブルに対して複雑なクエリを実行する際、StarRocks は事前計算結果を直接再利用でき、計算の重複を避け、クエリパフォーマンスを向上させることができます。クエリの頻度が高いほど、またはクエリステートメントが複雑であるほど、パフォーマンスの向上は顕著になります。

また、非同期マテリアライズドビューを使用してデータウェアハウスをモデリングし、上層のアプリケーションに統一されたデータインターフェースを提供し、下層の実装を隠蔽してベーステーブルの詳細データの安全を守ることができます。

### StarRocks マテリアライズドビューの理解

StarRocks のバージョン2.4以前では、同期更新される同期マテリアライズドビュー（Rollup）が提供されており、より良いデータの新鮮さと低い更新コストを提供していました。しかし、同期マテリアライズドビューには多くの制限があり、単一のベーステーブルに基づいてのみ作成可能で、限られた集約演算子のみをサポートしていました。バージョン2.4以降では、非同期マテリアライズドビューがサポートされ、複数のベーステーブルに基づいて作成可能で、より豊富な集約演算子をサポートしています。

以下の表は、StarRocks の非同期マテリアライズドビューと同期マテリアライズドビュー（Rollup）をサポートする特徴を比較しています：

|                              | **単一テーブル集約** | **複数テーブル結合** | **クエリ改写** | **更新戦略** | **ベーステーブル** |
| ---------------------------- | ----------- | ---------- | ----------- | ---------- | -------- |
| **非同期マテリアライズドビュー** | はい | はい | はい | <ul><li>非同期更新</li><li>手動更新</li></ul> | 複数テーブル構築をサポート。ベーステーブルは以下から選択可能：<ul><li>Default Catalog</li><li>External Catalog（v2.5以降）</li><li>既存の非同期マテリアライズドビュー（v2.5以降）</li><li>既存のビュー（v3.1以降）</li></ul> |
| **同期マテリアライズドビュー（Rollup）** | 一部の集約関数のみ | いいえ | はい | インポート時同期更新 | Default Catalog の単一テーブル構築のみサポート |

### 関連概念

- **ベーステーブル（Base Table）**

  マテリアライズドビューのドライブテーブル。

  StarRocks の非同期マテリアライズドビューにおいて、ベーステーブルは [Default catalog](../data_source/catalog/default_catalog.md) 内の内部テーブル、外部データカタログ内のテーブル（バージョン2.5以降でサポート）、既存の非同期マテリアライズドビュー（バージョン2.5以降でサポート）、またはビュー（バージョン3.1以降でサポート）である可能性があります。StarRocks は、すべての [StarRocks テーブルタイプ](../table_design/table_types/table_types.md) に非同期マテリアライズドビューを作成することをサポートしています。

- **更新（Refresh）**

  非同期マテリアライズドビューを作成した後、そのデータは作成時点のベーステーブルの状態を反映しています。ベーステーブルのデータに変更があった場合、非同期マテリアライズドビューを更新してデータ変更を反映させる必要があります。

  現在、StarRocks は以下の2種類の非同期更新戦略をサポートしています：

  - ASYNC：非同期更新。ベーステーブルのデータに変更があるたびに、指定された更新間隔に基づいて自動的に更新タスクがトリガーされます。
  - MANUAL：手動トリガー更新。マテリアライズドビューは自動的には更新されず、ユーザーが手動で更新タスクを管理する必要があります。

- **クエリ改写（Query Rewrite）**

  既にマテリアライズドビューが構築されているベーステーブルに対するクエリを実行する際、システムは自動的にマテリアライズドビューの事前計算結果を再利用してクエリを処理できるかどうかを判断します。再利用可能であれば、システムは関連するマテリアライズドビューから事前計算結果を直接読み取り、システムリソースと時間の消費を避けます。

  バージョン2.5以降、StarRocks は SPJG タイプの非同期マテリアライズドビューに基づく自動的で透過的なクエリ改写をサポートしています。SPJG タイプのマテリアライズドビューとは、マテリアライズドビューのプランに Scan、Filter、Project、Aggregate のタイプの演算子のみが含まれているものを指します。

  > **注記**
  >
  > [JDBC Catalog](../data_source/catalog/jdbc_catalog.md) のテーブルを基に構築された非同期マテリアライズドビューは、現在クエリ改写をサポートしていません。

## 使用シナリオ

データウェアハウス環境に以下のようなニーズがある場合、非同期マテリアライズドビューの作成をお勧めします：

- **繰り返しの集約クエリの高速化**

  データウェアハウス環境に同じ集約関数のサブクエリを含むクエリが大量に存在し、多くの計算リソースを消費している場合、そのサブクエリに基づいて非同期マテリアライズドビューを作成し、すべての結果を計算して保存することができます。作成後、システムはクエリステートメントを自動的に改写し、非同期マテリアライズドビューの中間結果を直接クエリすることで、負荷を軽減し、クエリを高速化します。

- **定期的な複数テーブル結合クエリ**

  定期的にデータウェアハウス内の複数のテーブルを結合して新しいワイドテーブルを生成する必要がある場合、これらのテーブルに非同期マテリアライズドビューを作成し、定期的な更新ルールを設定することで、手動で結合タスクをスケジュールする必要を避けることができます。非同期マテリアライズドビューが作成されると、クエリは非同期マテリアライズドビューに基づいて結果を直接返すため、結合操作による遅延を回避できます。

- **データウェアハウスの階層化**

  ベーステーブルに大量の原始データが含まれており、クエリに複雑なETL操作が必要な場合、データに対して複数層の非同期マテリアライズドビューを作成することでデータウェアハウスの階層化を実現できます。これにより、複雑なクエリを複数の単純なクエリに分解することができ、重複する計算を減らすとともに、メンテナンス担当者が問題を迅速に特定するのに役立ちます。さらに、データウェアハウスの階層化により、原始データと統計データを分離し、機密性の高い原始データを保護することができます。

- **データレイクのクエリ高速化**

  データレイクへのクエリは、ネットワーク遅延やオブジェクトストレージのスループット制限により遅くなる可能性があります。データレイク上に非同期マテリアライズドビューを構築することで、クエリパフォーマンスを向上させることができます。また、StarRocks は既存のマテリアライズドビューを使用するようにクエリをインテリジェントに改写するため、手動でクエリを変更する手間を省くことができます。

非同期マテリアライズドビューの具体的な使用例については、以下の内容を参照してください：

- [データモデリング](./data_modeling_with_materialized_views.md)
- [クエリ改写](./query_rewrite_with_materialized_views.md)
- [データレイクのクエリ高速化](./data_lake_query_acceleration_with_materialized_views.md)

## 非同期マテリアライズドビューの作成

StarRocks は以下のデータソースに非同期マテリアライズドビューを作成することをサポートしています：

- StarRocks 内部テーブル（ベーステーブルはすべての StarRocks テーブルタイプをサポート）
- External Catalog 内のテーブル

  - Hive Catalog（バージョン2.5以降でサポート）
  - Hudi Catalog（バージョン2.5以降でサポート）
  - Iceberg Catalog（バージョン2.5以降でサポート）

  - JDBC Catalog（v3.0 以降）

- 非同期マテリアライズドビューが利用可能（v2.5 以降）
- ビューが利用可能（v3.1 以降）

### 準備作業

以下の例は、Default Catalog にある2つの基本テーブルを使用しています：

- テーブル `goods` には、商品ID `item_id1`、商品名 `item_name`、商品価格 `price` が含まれています。
- テーブル `order_list` には、注文ID `order_id`、顧客ID `client_id`、商品ID `item_id2` が含まれています。

ここで、`item_id1` と `item_id2` は同等です。

以下のデータでテーブルを作成し、インポートします：

```SQL
CREATE TABLE goods(
    item_id1          INT,
    item_name         STRING,
    price             FLOAT
) DISTRIBUTED BY HASH(item_id1);

INSERT INTO goods
VALUES
    (1001,"apple",6.5),
    (1002,"pear",8.0),
    (1003,"potato",2.2);

CREATE TABLE order_list(
    order_id          INT,
    client_id         INT,
    item_id2          INT,
    order_date        DATE
) DISTRIBUTED BY HASH(order_id);

INSERT INTO order_list
VALUES
    (10001,101,1001,"2022-03-13"),
    (10001,101,1002,"2022-03-13"),
    (10002,103,1002,"2022-03-13"),
    (10002,103,1003,"2022-03-14"),
    (10003,102,1003,"2022-03-14"),
    (10003,102,1001,"2022-03-14");
```

このビジネスシナリオでは、注文総額を頻繁に分析する必要があります。そのため、2つのテーブルを結合し、sum() 関数を呼び出して、注文IDと総額に基づいて新しいテーブルを生成する必要があります。さらに、このビジネスシナリオでは、毎日注文総額を更新する必要があります。

そのクエリ文は以下の通りです：

```SQL
SELECT
    order_id,
    sum(goods.price) as total
FROM order_list INNER JOIN goods ON goods.item_id1 = order_list.item_id2
GROUP BY order_id;
```

### クエリ文に基づいて非同期マテリアライズドビューを作成する

[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md) ステートメントを使用して、特定のクエリ文に基づいてマテリアライズドビューを作成できます。

以下の例では、上記のクエリ文に基づいて、テーブル `goods` と `order_list` から「注文IDでグループ化し、注文内の全商品の価格を合計する」非同期マテリアライズドビューを作成し、その更新方法を ASYNC に設定し、毎日自動更新するようにします。

```SQL
CREATE MATERIALIZED VIEW order_mv
DISTRIBUTED BY HASH(`order_id`)
REFRESH ASYNC START('2022-09-01 10:00:00') EVERY (interval 1 day)
AS SELECT
    order_list.order_id,
    sum(goods.price) as total
FROM order_list INNER JOIN goods ON goods.item_id1 = order_list.item_id2
GROUP BY order_id;
```

> **説明**
>
> - 非同期マテリアライズドビューを作成する際は、少なくともバケットまたは更新戦略のいずれかを指定する必要があります。
> - 非同期マテリアライズドビューには、基本テーブルとは異なるパーティションやバケット戦略を設定できますが、マテリアライズドビューのパーティション列とバケット列はクエリ文に含まれている必要があります。
> - 非同期マテリアライズドビューはパーティションのロールアップをサポートしています。例えば、基本テーブルが日単位でパーティション分けされている場合、マテリアライズドビューを月単位でパーティション分けすることができます。
> - 非同期マテリアライズドビューは、List パーティション戦略を使用することはできず、List パーティションを使用する基本テーブルに基づいて作成することもできません。
> - マテリアライズドビューを作成するクエリ文では、rand()、random()、uuid()、sleep() などの非決定的関数はサポートされていません。
> - 非同期マテリアライズドビューは、多様なデータ型をサポートしています。詳細については、[CREATE MATERIALIZED VIEW - サポートされるデータ型](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md#サポートされるデータ型) を参照してください。
> - デフォルトでは、CREATE MATERIALIZED VIEW ステートメントを実行すると、StarRocks は直ちに更新タスクを開始しますが、これにはシステムリソースが一定量消費されます。更新を遅らせる場合は、REFRESH DEFERRED パラメータを追加してください。

- **非同期マテリアライズドビューの更新メカニズム**

  現在、StarRocks は ON DEMAND 更新戦略として、非同期更新（ASYNC）と手動更新（MANUAL）の2種類をサポートしています。

  これに加えて、非同期マテリアライズドビューは、更新のコストをコントロールし、更新の成功率を保証するために、複数の更新メカニズムをサポートしています：

  - 更新時の最大パーティション数を設定できます。多くのパーティションを持つ非同期マテリアライズドビューでは、一度の更新で多くのリソースが消費されます。この更新メカニズムを設定することで、一度の更新で処理するパーティションの最大数を指定し、更新タスクを分割して、データ量の多いマテリアライズドビューが段階的に安定して更新を完了できるようにします。
  - 非同期マテリアライズドビューのパーティションに Time to Live（TTL）を指定することで、使用するストレージスペースを減らすことができます。
  - 更新範囲を指定して、最新の数個のパーティションのみを更新し、更新コストを削減できます。
  - データの変更がマテリアライズドビューの自動更新をトリガーしない基本テーブルを設定できます。
  - 更新タスクにリソースグループを設定できます。
  
  詳細については、[CREATE MATERIALIZED VIEW - パラメータ](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md#パラメータ) の **PROPERTIES** セクションを参照してください。また、[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) を使用して、既存の非同期マテリアライズドビューの更新メカニズムを変更することもできます。

  > **注意**
  >
  > 全量更新タスクがシステムリソースを使い果たし、タスクが失敗するのを避けるために、パーティション分けされた基本テーブルに基づいてパーティションマテリアライズドビューを作成することをお勧めします。これにより、基本テーブルのパーティション内のデータが更新された場合には、マテリアライズドビューの対応するパーティションのみが更新され、マテリアライズドビュー全体が更新されることはありません。詳細については、[マテリアライズドビューを使用したデータモデリング - パーティションモデリング](./data_modeling_with_materialized_views.md#パーティションモデリング) を参照してください。

- **ネストされたマテリアライズドビュー**

  StarRocks v2.5 以降のバージョンでは、非同期マテリアライズドビューに基づいて新しい非同期マテリアライズドビューを構築する、ネストされたマテリアライズドビューがサポートされています。各非同期マテリアライズドビューの更新方法は、現在のビューにのみ影響します。現在、StarRocks はネストの深さに制限を設けていませんが、本番環境ではネストの深さを3層以下にすることを推奨します。

- **External Catalog マテリアライズドビュー**

  StarRocks は、Hive Catalog（v2.5 以降）、Hudi Catalog（v2.5 以降）、Iceberg Catalog（v2.5 以降）、および JDBC Catalog（v3.0 以降）を基にした非同期マテリアライズドビューの構築をサポートしています。外部データカタログのマテリアライズドビューの作成方法は通常の非同期マテリアライズドビューと同じですが、使用には制限があります。詳細については、[マテリアライズドビューを使用したデータレイククエリの高速化](./data_lake_query_acceleration_with_materialized_views.md) を参照してください。

## 非同期マテリアライズドビューの手動更新

[REFRESH MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/REFRESH_MATERIALIZED_VIEW.md) コマンドを使用して、指定された非同期マテリアライズドビューを手動で更新できます。StarRocks v2.5 では、非同期マテリアライズドビューの一部のパーティションを手動で更新することがサポートされています。v3.1 では、StarRocks は更新タスクを同期的に呼び出すことをサポートしています。

```SQL
-- 非同期で更新タスクを呼び出します。
REFRESH MATERIALIZED VIEW order_mv;
-- 更新タスクを同期的に呼び出します。
REFRESH MATERIALIZED VIEW order_mv WITH SYNC MODE;
```

[CANCEL REFRESH MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/CANCEL_REFRESH_MATERIALIZED_VIEW.md) を使用して、非同期で呼び出された更新タスクをキャンセルできます。

## 非同期マテリアライズドビューを直接クエリする

非同期マテリアライズドビューは本質的に物理テーブルであり、特定のクエリ文に基づいて事前に計算された完全な結果セットが格納されています。マテリアライズドビューが初めて更新された後、直接マテリアライズドビューをクエリできます。

```Plain
MySQL > SELECT * FROM order_mv;
+----------+--------------------+
| order_id | total              |
+----------+--------------------+
|    10001 |               14.5 |
|    10002 | 10.200000047683716 |
|    10003 |  8.700000047683716 |
+----------+--------------------+
3 rows in set (0.01 sec)
```

> **説明**
>
> 非同期マテリアライズドビューを直接クエリすることができますが、非同期リフレッシュメカニズムのため、基本テーブルからのクエリ結果と一致しない可能性があります。

## 非同期マテリアライズドビューを使用してクエリを高速化する

StarRocks v2.5 では、SPJGタイプの非同期マテリアライズドビュークエリの自動透過的な改写がサポートされています。クエリ改写には、シングルテーブル改写、Join改写、集約改写、Union改写、およびネストされたマテリアライズドビューの改写が含まれます。詳細は、[マテリアライズドビュークエリ改写](./query_rewrite_with_materialized_views.md)を参照してください。

現在、StarRocksはDefault catalog、Hive catalog、Hudi catalog、Iceberg catalogに基づく非同期マテリアライズドビューのクエリ改写をサポートしています。Default catalogのデータをクエリする場合、StarRocksはデータが基本テーブルと一致しないマテリアライズドビューを排除することで、改写後のクエリが元のクエリ結果と強い一致性を保つようにします。マテリアライズドビューのデータが古くなった場合、それは候補のマテリアライズドビューとしては使用されません。外部カタログのデータをクエリする場合、StarRocksは外部カタログのパーティション内のデータ変更を検知できないため、結果の強い一致性を保証しません。External Catalogに基づく非同期マテリアライズドビューについては、[データレイククエリの高速化のためのマテリアライズドビューの使用](./data_lake_query_acceleration_with_materialized_views.md)を参照してください。

> **注意**
>
> JDBC Catalogに基づいて構築された非同期マテリアライズドビューは、現在クエリ改写をサポートしていません。

## 非同期マテリアライズドビューの管理

### 非同期マテリアライズドビューの変更

[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) コマンドを使用して、非同期マテリアライズドビューの属性を変更できます。

- 無効になっている非同期マテリアライズドビューを有効にする（マテリアライズドビューの状態をActiveに設定）。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv ACTIVE;
  ```

- 非同期マテリアライズドビューの名前を `order_total` に変更する。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv RENAME TO order_total;
  ```

- 非同期マテリアライズドビューの最大リフレッシュ間隔を2日に変更する。

  ```SQL
  ALTER MATERIALIZED VIEW order_mv REFRESH ASYNC EVERY(INTERVAL 2 DAY);
  ```

### 非同期マテリアライズドビューの表示

[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md) コマンドまたはInformation Schemaのシステムメタデータビューをクエリして、データベース内の非同期マテリアライズドビューを表示できます。

- 現在のデータウェアハウス内のすべての非同期マテリアライズドビューを表示する。

  ```SQL
  SHOW MATERIALIZED VIEWS;
  ```

- 特定の非同期マテリアライズドビューを表示する。

  ```SQL
  SHOW MATERIALIZED VIEWS WHERE NAME = 'order_mv';
  ```

- 名前で非同期マテリアライズドビューを検索する。

  ```SQL
  SHOW MATERIALIZED VIEWS WHERE NAME LIKE 'order%';
  ```

- Information Schemaの `materialized_views` システムメタデータビューを通じて、すべての非同期マテリアライズドビューを表示する。詳細は、[information_schema.materialized_views](../reference/information_schema/materialized_views.md)を参照してください。

  ```SQL
  SELECT * FROM information_schema.materialized_views;
  ```

### 非同期マテリアライズドビューの作成文の表示

[SHOW CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_MATERIALIZED_VIEW.md) コマンドを使用して、非同期マテリアライズドビューの作成文を表示できます。

```SQL
SHOW CREATE MATERIALIZED VIEW order_mv;
```

### 非同期マテリアライズドビューの実行状態の表示

StarRocksの [Information Schema](../reference/overview-pages/information_schema.md) にある [`tasks`](../reference/information_schema/tasks.md) と [`task_runs`](../reference/information_schema/task_runs.md) のメタデータビューをクエリして、非同期マテリアライズドビューの実行（構築またはリフレッシュ）状態を表示できます。

以下は、最新に作成された非同期マテリアライズドビューの実行状態を表示する例です：

1. `tasks` テーブルで最新のタスクの `TASK_NAME` を表示する。

   ```Plain
   mysql> select * from information_schema.tasks order by CREATE_TIME desc limit 1\G;
   *************************** 1. row ***************************
     TASK_NAME: mv-59299
   CREATE_TIME: 2022-12-12 17:33:51
      SCHEDULE: MANUAL
      DATABASE: ssb_1
    DEFINITION: insert overwrite hive_mv_lineorder_flat_1 SELECT `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderkey`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_linenumber`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_custkey`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_partkey`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderpriority`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_ordtotalprice`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_revenue`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`p_mfgr`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`s_nation`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`c_city`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`c_nation`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderdate`
   FROM `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`
   WHERE `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderdate` = '1997-01-01'
   EXPIRE_TIME: NULL
   1 row in set (0.02 sec)
   ```

2. 取得した `TASK_NAME` を基に `task_runs` テーブルで実行状態を表示する。

   ```Plain
   mysql> select * from information_schema.task_runs where task_name='mv-59299' order by CREATE_TIME \G;
   *************************** 1. row ***************************
        QUERY_ID: d9cef11f-7a00-11ed-bd90-00163e14767f
       TASK_NAME: mv-59299
     CREATE_TIME: 2022-12-12 17:39:19
     FINISH_TIME: 2022-12-12 17:39:22
           STATE: SUCCESS
        DATABASE: ssb_1
      DEFINITION: insert overwrite hive_mv_lineorder_flat_1 SELECT `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderkey`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_linenumber`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_custkey`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_partkey`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderpriority`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_ordtotalprice`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_revenue`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`p_mfgr`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`s_nation`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`c_city`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`c_nation`, `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderdate`
   FROM `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`
   WHERE `hive_ci`.`dla_scan`.`lineorder_flat_1000_1000_orc`.`lo_orderdate` = '1997-01-01'
     EXPIRE_TIME: 2022-12-15 17:39:19
      ERROR_CODE: 0
   ERROR_MESSAGE: NULL
        PROGRESS: 100%
   2 rows in set (0.02 sec)
   ```

### 非同期マテリアライズドビューの削除

[DROP MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/DROP_MATERIALIZED_VIEW.md) コマンドを使用して、作成された非同期マテリアライズドビューを削除できます。

```SQL
DROP MATERIALIZED VIEW order_mv;
```

### 関連するセッション変数

以下の変数はマテリアライズドビューの挙動を制御します：

- `analyze_mv`：リフレッシュ後にマテリアライズドビューをどのように分析するか。有効な値は空文字列（つまり分析しない）、`sample`（サンプル収集）または `full`（全量収集）。デフォルトは `sample`です。
- `enable_materialized_view_rewrite`：マテリアライズドビューの自動改写を有効にするかどうか。有効な値は `true`（バージョン2.5からデフォルト）と `false`です。
