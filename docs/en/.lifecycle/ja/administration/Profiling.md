---
displayed_sidebar: "Japanese"
---

# パフォーマンスの最適化

## テーブルの種類の選択

StarRocksは、重複キー、集計テーブル、一意キー、および主キーテーブルの4つのテーブルタイプをサポートしています。すべてのテーブルはキーでソートされます。

- `AGGREGATE KEY`: StarRocksに同じAGGREGATE KEYを持つレコードがロードされると、古いレコードと新しいレコードが集計されます。現在、集計テーブルは次の集計関数をサポートしています: SUM、MIN、MAX、REPLACE。集計テーブルはデータの事前集計をサポートし、ビジネスレポートや多次元分析を容易にします。
- `DUPLICATE KEY`: DUPLICATE KEYテーブルでは、ソートキーのみを指定する必要があります。同じDUPLICATE KEYを持つレコードは同時に存在します。データの事前集計を行わない分析に適しています。
- `UNIQUE KEY`: StarRocksに同じUNIQUE KEYを持つレコードがロードされると、新しいレコードが古いレコードを上書きします。UNIQUE KEYテーブルはREPLACE関数を持つ集計テーブルに似ています。どちらも定期的な更新を伴う分析に適しています。
- `PRIMARY KEY`: 主キーテーブルはレコードの一意性を保証し、リアルタイムの更新を行うことができます。

~~~sql
CREATE TABLE site_visit
(
    siteid      INT,
    city        SMALLINT,
    username    VARCHAR(32),
    pv BIGINT   SUM DEFAULT '0'
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
    ip          varchar(32),
    brower      CHAR(20),
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

クエリの高速化のために、同じ分散を持つテーブルは共通のバケット列を使用することができます。その場合、データは`join`操作中にクラスタ間で転送されることなく、ローカルで結合されます。

~~~sql
CREATE TABLE colocate_table
(
    visitorid   SMALLINT,
    sessionid   BIGINT,
    visittime   DATETIME,
    city        CHAR(20),
    province    CHAR(20),
    ip          varchar(32),
    brower      CHAR(20),
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

StarRocksはスタースキーマをサポートしており、フラットテーブルよりもモデリングが柔軟です。モデリング中にフラットテーブルを置き換えるためのビューを作成し、複数のテーブルからデータをクエリすることでクエリを高速化することができます。

フラットテーブルには以下のような欠点があります:

- フラットテーブルには通常、膨大な数の次元が含まれているため、次元の更新にはコストがかかります。次元が更新されるたびに、テーブル全体を更新する必要があります。更新頻度が高くなるほど、状況は悪化します。
- フラットテーブルは追加の開発作業量、ストレージスペース、およびデータバックフィリング操作が必要となるため、メンテナンスコストが高くなります。
- フラットテーブルには多くのフィールドがあり、集計テーブルにはさらに多くのキーフィールドが含まれるため、データのロード時にはより多くのフィールドがソートされる必要があります。

クエリの同時実行数やレイテンシの低さに高い要件がある場合は、引き続きフラットテーブルを使用することができます。

## パーティションとバケット

StarRocksは2つのレベルのパーティショニングをサポートしています: 第1レベルはRANGEパーティション、第2レベルはHASHバケットです。

- RANGEパーティション: RANGEパーティションはデータを異なる間隔に分割するために使用されます（元のテーブルを複数のサブテーブルに分割すると考えることができます）。多くのユーザーは、時間によるパーティションを設定することを選択します。これには以下の利点があります:

  - ホットデータとコールドデータを簡単に区別できる
  - StarRocksのティアドストレージ（SSD + SATA）を活用できる
  - パーティションごとにデータを削除するのが速い

- HASHバケット: ハッシュ値に基づいてデータを異なるバケットに分割します。

  - データの偏りを避けるために、高い識別度を持つ列をバケット化することをおすすめします。
  - データの復旧を容易にするために、各バケットの圧縮データのサイズを100 MBから1 GBの間に保つことをおすすめします。テーブルを作成するか、パーティションを追加する際に適切な数のバケットを設定することをおすすめします。
  - ランダムなバケット分割は推奨されません。テーブルを作成する際に、明示的にHASHバケット化する列を指定する必要があります。

## スパースインデックスとブルームフィルタインデックス

StarRocksはデータを順序付けて格納し、スパースインデックスを1024行の粒度で構築します。

StarRocksは、スキーマの固定長のプレフィックス（現在は36バイト）をスパースインデックスとして選択します。

テーブルを作成する際には、一般的なフィルタフィールドをスキーマ宣言の先頭に配置することをおすすめします。最も識別度が高く、クエリ頻度が高いフィールドを最初に配置する必要があります。

VARCHARフィールドはスパースインデックスの末尾に配置する必要があります。なぜなら、VARCHARフィールドからインデックスが切り詰められるからです。VARCHARフィールドが最初に現れる場合、インデックスは36バイト未満になる可能性があります。

上記の`site_visit`テーブルを例にとると、テーブルには`siteid、city、username、pv`の4つの列があります。ソートキーには`siteid、city、username`の3つの列が含まれており、それぞれ4バイト、2バイト、32バイトを占有します。したがって、プレフィックスインデックス（スパースインデックス）は`siteid + city + username`の最初の30バイトとなります。

スパースインデックスに加えて、StarRocksはブルームフィルタインデックスも提供しており、識別度の高い列のフィルタリングに効果的です。他のフィールドよりも先にVARCHARフィールドを配置したい場合は、ブルームフィルタインデックスを作成することができます。

## 逆インデックス

StarRocksはBitmap Indexing技術を採用し、Duplicate Keyテーブルのすべての列およびAggregateテーブルとUnique Keyテーブルのキーカラムに逆インデックスを適用することができます。Bitmap Indexは、性別、都市、および州などの値範囲が小さい列に適しています。範囲が拡大するにつれて、ビットマップインデックスも並行して拡大します。

## マテリアライズドビュー（ロールアップ）

ロールアップは、元のテーブル（ベーステーブル）のマテリアライズドインデックスです。ロールアップを作成する際には、ベーステーブルの一部の列のみをスキーマとして選択し、スキーマのフィールドの順序をベーステーブルとは異なる順序にすることができます。以下はロールアップの使用例です:

- ベーステーブルのデータ集計が高くない場合、ベーステーブルには識別度の高いフィールドが含まれています。この場合、一部の列を選択してロールアップを作成することを検討することができます。上記の`site_visit`テーブルを例にとると:

  ~~~sql
  site_visit(siteid, city, username, pv)
  ~~~

  `siteid`はデータの集計が悪化する可能性があります。頻繁に都市ごとのPVを計算する必要がある場合は、`city`と`pv`のみを持つロールアップを作成することができます。

  ~~~sql
  ALTER TABLE site_visit ADD ROLLUP rollup_city(city, pv);
  ~~~

- ベーステーブルのプレフィックスインデックスがヒットしない場合、ベーステーブルの構築方法ではすべてのクエリパターンをカバーすることができません。この場合、ロールアップを作成して列の順序を調整することを検討することができます。上記の`session_data`テーブルを例にとると:

  ~~~sql
  session_data(visitorid, sessionid, visittime, city, province, ip, brower, url)
  ~~~

  `visitorid`に加えて、`browser`と`province`で訪問を分析する必要がある場合、別のロールアップを作成することができます:

  ~~~sql
  ALTER TABLE session_data
  ADD ROLLUP rollup_brower(brower,province,ip,url)
  DUPLICATE KEY(brower,province);
  ~~~

## スキーマの変更

StarRocksでは、ソートされたスキーマ変更、直接のスキーマ変更、リンクされたスキーマ変更の3つの方法でスキーマを変更することができます。

- ソートされたスキーマ変更: 列のソートを変更し、データの並べ替えを行います。例えば、ソートされたスキーマから列を削除すると、データの並べ替えが行われます。

  `ALTER TABLE site_visit DROP COLUMN city;`

- 直接のスキーマ変更: データを変換するだけで、並べ替えは行いません。例えば、列の型を変更したり、スパースインデックスに列を追加したりします。

  `ALTER TABLE site_visit MODIFY COLUMN username varchar(64);`

- リンクされたスキーマ変更: データを変換せずに変更を完了します。例えば、列を追加します。

  `ALTER TABLE site_visit ADD COLUMN click bigint SUM default '0';`

  スキーマの変更を加速するために、テーブルを作成する際に適切なスキーマを選択することをおすすめします。
