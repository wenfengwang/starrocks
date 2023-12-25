---
displayed_sidebar: English
---

# パフォーマンス最適化

## テーブルタイプの選択

StarRocksは、Duplicate Keyテーブル、Aggregateテーブル、Unique Keyテーブル、Primary Keyテーブルの4つのテーブルタイプをサポートしています。これらはすべてKEYでソートされます。

- `AGGREGATE KEY`: 同じAGGREGATE KEYを持つレコードがStarRocksにロードされると、古いレコードと新しいレコードが集約されます。現在、Aggregateテーブルは以下の集約関数をサポートしています: SUM、MIN、MAX、REPLACE。Aggregateテーブルはデータを事前に集約することをサポートし、ビジネスレポートと多次元分析を容易にします。
- `DUPLICATE KEY`: DUPLICATE KEYテーブルにはソートキーのみを指定する必要があります。同じDUPLICATE KEYを持つレコードが同時に存在します。これは事前にデータを集約しない分析に適しています。
- `UNIQUE KEY`: 同じUNIQUE KEYを持つレコードがStarRocksにロードされると、新しいレコードが古いものを上書きします。UNIQUE KEYテーブルは、REPLACE関数を持つAggregateテーブルに似ており、両方とも頻繁な更新を伴う分析に適しています。
- `PRIMARY KEY`: Primary Keyテーブルはレコードの一意性を保証し、リアルタイムでの更新を可能にします。

~~~sql
CREATE TABLE site_visit
(
    siteid      INT,
    city        SMALLINT,
    username    VARCHAR(32),
    pv          BIGINT   SUM DEFAULT '0'
)
AGGREGATE KEY(siteid, city, username)
DISTRIBUTED BY HASH(siteid);


CREATE TABLE session_data
(
    visitorid   SMALLINT,
    sessionid   BIGINT,
    visittime   DATETIME,
    city        CHAR(20),
    province    CHAR(20),
    ip          VARCHAR(32),
    browser     CHAR(20),
    url         VARCHAR(1024)
)
DUPLICATE KEY(visitorid, sessionid)
DISTRIBUTED BY HASH(sessionid, visitorid);

CREATE TABLE sales_order
(
    orderid     BIGINT,
    status      TINYINT,
    username    VARCHAR(32),
    amount      BIGINT DEFAULT '0'
)
UNIQUE KEY(orderid)
DISTRIBUTED BY HASH(orderid);

CREATE TABLE sales_order
(
    orderid     BIGINT,
    status      TINYINT,
    username    VARCHAR(32),
    amount      BIGINT DEFAULT '0'
)
PRIMARY KEY(orderid)
DISTRIBUTED BY HASH(orderid);
~~~

## コロケートテーブル

クエリを高速化するために、同じ分布を持つテーブルは共通のバケットカラムを使用できます。その場合、`join`操作中にクラスター間でデータを転送することなく、ローカルでデータを結合できます。

~~~sql
CREATE TABLE colocate_table
(
    visitorid   SMALLINT,
    sessionid   BIGINT,
    visittime   DATETIME,
    city        CHAR(20),
    province    CHAR(20),
    ip          VARCHAR(32),
    browser     CHAR(20),
    url         VARCHAR(1024)
)
DUPLICATE KEY(visitorid, sessionid)
DISTRIBUTED BY HASH(sessionid, visitorid)
PROPERTIES(
    "colocate_with" = "group1"
);
~~~

コロケートジョインとレプリカ管理についての詳細は、[コロケートジョイン](../using_starrocks/Colocate_join.md)を参照してください。

## フラットテーブルとスタースキーマ

StarRocksはスタースキーマをサポートしており、フラットテーブルよりも柔軟なモデリングが可能です。モデリング中にフラットテーブルを置き換えるビューを作成し、その後複数のテーブルからデータをクエリしてクエリの高速化を図ることができます。

フラットテーブルには以下の欠点があります:

- フラットテーブルには通常多数のディメンションが含まれており、ディメンションの更新にはコストがかかります。ディメンションが更新されるたびに、テーブル全体を更新する必要があります。更新頻度が高まると問題が悪化します。
- 追加の開発作業、ストレージスペース、データバックフィリング作業が必要なため、フラットテーブルのメンテナンスコストは高くなります。
- フラットテーブルには多くのフィールドがあり、集約テーブルにはさらに多くのキーフィールドが含まれる可能性があるため、データの取り込みコストが高くなります。データロード中には、より多くのフィールドをソートする必要があり、データロード時間が長くなります。

クエリの同時実行性や低遅延に高い要求がある場合は、フラットテーブルを使用することもできます。

## パーティションとバケット

StarRocksは、第1レベルがRANGEパーティション、第2レベルがHASHバケットの2レベルのパーティショニングをサポートしています。

- RANGEパーティション: RANGEパーティションは、データを異なる区間に分割するために使用されます（元のテーブルを複数のサブテーブルに分割すると考えることができます）。多くのユーザーは時間によってパーティションを設定することを選択しており、以下の利点があります:

  - ホットデータとコールドデータを簡単に区別できる
  - StarRocksの階層型ストレージ（SSD + SATA）を活用できる
  - パーティションによるデータの迅速な削除が可能

- HASHバケット: ハッシュ値に基づいてデータを異なるバケットに分割します。

  - データの偏りを避けるために、識別度の高いカラムをバケット化に使用することを推奨します。

  - データ復旧を容易にするために、各バケット内の圧縮データのサイズは100 MBから1 GBの間に保つことを推奨します。テーブルを作成する際やパーティションを追加する際には、適切な数のバケットを設定することをお勧めします。
  - ランダムバケッティングは推奨されません。テーブルを作成する際には、HASHバケッティングのカラムを明示的に指定する必要があります。

## スパースインデックスとブルームフィルターインデックス

StarRocksはデータを順序付けて格納し、1024行ごとにスパースインデックスを構築します。

StarRocksはスキーマ内の固定長プレフィックス（現在は36バイト）をスパースインデックスとして選択します。

テーブルを作成する際には、スキーマ宣言の先頭に共通フィルターフィールドを配置することを推奨します。最も識別性が高くクエリ頻度の高いフィールドを最初に配置する必要があります。

VARCHARフィールドは、インデックスがVARCHARフィールドから切り捨てられるため、スパースインデックスの末尾に配置する必要があります。VARCHARフィールドが最初に来る場合、インデックスは36バイト未満になる可能性があります。

例として`site_visit`テーブルを挙げます。このテーブルには4つのカラムがあります：`siteid, city, username, pv`。ソートキーには`siteid`、`city`、`username`の3つのカラムが含まれ、それぞれ4バイト、2バイト、32バイトを占めます。したがって、プレフィックスインデックス（スパースインデックス）は`siteid + city + username`の最初の38バイトになります。

スパースインデックスに加えて、StarRocksは識別性の高いカラムのフィルタリングに有効なブルームフィルターインデックスも提供しています。他のフィールドの前にVARCHARフィールドを配置したい場合は、ブルームフィルターインデックスを作成できます。

## インバーテッドインデックス

StarRocksはビットマップインデックス技術を採用し、Duplicate Keyテーブルのすべてのカラムと、AggregateテーブルおよびUnique Keyテーブルのキーカラムに適用可能なインバーテッドインデックスをサポートしています。ビットマップインデックスは、性別、都市、県などの値の範囲が小さいカラムに適しています。範囲が広がるにつれて、ビットマップインデックスも並行して拡大します。

## マテリアライズドビュー（ロールアップ）

ロールアップは、本質的には元のテーブル（ベーステーブル）のマテリアライズドインデックスです。ロールアップを作成する際には、ベーステーブルの一部のカラムのみをスキーマとして選択でき、スキーマ内のフィールドの順序はベーステーブルと異なることがあります。以下はロールアップの使用例です：

- ベーステーブルのデータ集約度が低い場合、これはベーステーブルに識別性の高いフィールドがあるためです。この場合、いくつかのカラムを選択してロールアップを作成することを検討できます。`site_visit`テーブルを例に挙げます：

  ~~~sql
  site_visit(siteid, city, username, pv)
  ~~~

  `siteid`によりデータの集約度が低くなる可能性があります。市ごとのPVを頻繁に計算する必要がある場合、`city`と`pv`のみを含むロールアップを作成できます。

  ~~~sql
  ALTER TABLE site_visit ADD ROLLUP rollup_city(city, pv);
  ~~~

- ベーステーブルのプレフィックスインデックスがヒットしない場合、これはベーステーブルの構築方法がすべてのクエリパターンをカバーできないためです。この場合、カラムの順序を調整するためにロールアップを作成することを検討できます。`session_data`テーブルを例に挙げます：

  ~~~sql
  session_data(visitorid, sessionid, visittime, city, province, ip, browser, url)
  ~~~

  `visitorid`に加えて`browser`と`province`による訪問分析が必要な場合、別のロールアップを作成できます：

  ~~~sql
  ALTER TABLE session_data
  ADD ROLLUP rollup_browser(browser,province,ip,url)
  DUPLICATE KEY(browser,province);
  ~~~

## スキーマ変更

StarRocksでスキーマを変更するには、ソートスキーマ変更、ダイレクトスキーマ変更、リンクスキーマ変更の3つの方法があります。

- ソートスキーマ変更：カラムのソート順を変更し、データを再配置します。例えば、ソートされたスキーマのカラムを削除すると、データが再配置されます。

  `ALTER TABLE site_visit DROP COLUMN city;`

- ダイレクトスキーマ変更：データを再配置するのではなく、データを変換します。例えば、カラムタイプを変更するか、スパースインデックスにカラムを追加する場合などです。

  `ALTER TABLE site_visit MODIFY COLUMN username VARCHAR(64);`

- リンクドスキーマ変更: データ変換を伴わずに変更を完了させること、例えばカラムの追加など。

  `ALTER TABLE site_visit ADD COLUMN click bigint SUM default '0';`

  スキーマ変更を加速するために、テーブル作成時に適切なスキーマを選択することを推奨します。
