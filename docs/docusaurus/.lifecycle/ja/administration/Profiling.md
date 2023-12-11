---
displayed_sidebar: "Japanese"
---

# パフォーマンスの最適化

## テーブルタイプの選択

StarRocksは4つのテーブルタイプをサポートしています: Duplicate Keyテーブル、Aggregateテーブル、Unique Keyテーブル、Primary KeyテーブルのいずれもがKEYでソートされています。

- `AGGREGATE KEY`: StarRocksに同じAGGREGATE KEYのレコードがロードされると、古いレコードと新しいレコードが集計されます。現在、集計テーブルは以下の集計関数をサポートしています: SUM, MIN, MAX, REPLACE。集計テーブルはデータの事前集計をサポートし、ビジネスステートメントや多次元分析を容易にします。
- `DUPLICATE KEY`: DUPLICATE KEYテーブルでは、ソートキーの指定のみが必要です。DUPLICATE KEYを持つレコードは、同時に存在します。事前のデータ集計を必要としない解析に適しています。
- `UNIQUE KEY`: StarRocksに同じUNIQUE KEYのレコードがロードされると、新しいレコードが古いものに上書きされます。UNIQUE KEYテーブルは、REPLACE関数を持つ集計テーブルと類似しています。どちらも定期的なアップデートを行う解析に適しています。
- `PRIMARY KEY`: Primary Keyテーブルはレコードの一意性を保証し、リアルタイムの更新を可能にします。

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

## テーブルの共有

クエリの高速化のために、同じ配布を持つテーブルは共通のバケット化カラムを使用することができます。その場合、データはクラスタを跨いで転送することなくローカルで結合できます。

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

共有結合およびレプリカ管理についての詳細は、[共有結合](../using_starrocks/Colocate_join.md)を参照してください。

## フラットテーブルとスタースキーマ

StarRocksはスタースキーマをサポートし、フラットテーブルよりもモデリングが柔軟です。モデリング中にフラットテーブルを置き換えるためにビューを作成し、それから複数のテーブルからデータをクエリすることでクエリを高速化できます。

フラットテーブルには以下のような欠点があります:

- 通常、フラットテーブルには大量の次元が含まれているため、次元の更新がコストがかかります。次元が更新されるたびに、全体のテーブルを更新する必要があります。更新頻度が増加するにつれて、これらの状況は悪化します。
- フラットテーブルは追加の開発ワークロード、ストレージスペース、およびデータバックフィルの操作を必要とするため、高いメンテナンスコストがかかります。
- フラットテーブルには多くのフィールドが含まれており、集計テーブルにはさらに多くのキーフィールドが含まれているため、データローディング時にはより多くのフィールドがソートされる必要があります。

クエリの同時性や低レイテンシを求める場合は、フラットテーブルを使用することができます。

## パーティションとバケット

StarRocksは2つのレベルのパーティションをサポートしています: 第1レベルは範囲パーティション、第二レベルはハッシュバケツです。

- 範囲パーティション: 範囲パーティションはデータを異なる間隔に分割するために使用されます（元のテーブルが複数のサブテーブルに分割されたと考えることができます）。ほとんどのユーザーは時系列でパーティションを設定することを選択しています。これには以下の利点があります:

  - ホットデータとコールドデータを簡単に区別できる
  - StarRocksのティアードストレージ（SSD + SATA）を利用できる
  - パーティションごとにデータを削除する操作が迅速になります

- ハッシュバケツ: データをハッシュ値に従って異なるバケツに分割します。

  - バケツ化には区別が高いカラムを使用することをお勧めします。データスキューを避けるために
  - データの回復を容易にするために、各バケツで圧縮データのサイズを100 MBから1 GBの間に保つことをお勧めします。テーブルを作成するか、パーティションを追加するときには適切な数のバケツを構成することをお勧めします
  - ランダムなバケティングはお勧めしません。テーブルを作成するときには、はっきりとハッシュバケツのカラムを指定する必要があります。

## Sparse indexとBloomfilter index

StarRocksはデータを順序付けて保存し、1024行の単位でSparse indexを構築します。

StarRocksはスキーマに固定長のプレフィックス（現在は36バイト）を選択し、Sparse indexとします。

テーブルを作成するときには、一般的なフィルタフィールドをスキーマ宣言の先頭に配置することをお勧めします。最も区別が高く、クエリ頻度が高いフィールドは、最初に配置する必要があります。

VARCHARフィールドは、Sparse indexの末尾に配置する必要があります。なぜなら、VARCHARフィールドからSparse indexが切り詰められるからです。もしVARCHARフィールドが最初に出現すると、インデックスは36バイト未満になる可能性があります。

上記の`site_visit`テーブルを例にとってみましょう。テーブルには`siteid, city, username, pv`の4つの列があります。ソートキーには`siteid，city，username`という3つの列が含まれており、それぞれ4、2、32バイトを占有しています。したがって、プレフィックス・インデックス（Sparse index）は`siteid + city + username`の最初の30バイトとなります。

Sparse indexに加えて、StarRocksはBloomfilterインデックスも提供しており、区別が高いフィルタフィールドに有効です。VARCHARフィールドを他のフィールドより前に配置する場合は、Bloomfilterインデックスを作成することができます。

## 逆インデックス

StarRocksでは、Bitmapインデックス技術を採用し、Duplicate Keyテーブル全体およびAggregateテーブル、Unique Keyテーブルのキーカラムに適用できる逆インデックスをサポートしています。Bitmapインデックスは、性別、都市、州などの値範囲が小さい列に適しています。値の範囲が広がるにつれて、Bitmapインデックスも拡張されます。

## マテリアライズド・ビュー（ロールアップ）

ロールアップは、本来のテーブル（ベーステーブル）のマテリアライズド・インデックスです。ロールアップを作成する際には、ベーステーブルの一部の列をスキーマとして選択し、スキーマ内のフィールドの順序をベーステーブルとは異なる順序にすることができます。以下はロールアップの使用ケースのいくつかです:

- ベーステーブルでのデータ集計が高くない場合、次元の差異が高い列があるため、一部の列を選択してロールアップを作成することを検討してください。上記の`site_visit`テーブルを例にとってみましょう:

  ~~~sql
  site_visit(siteid, city, username, pv)
  ~~~

  `siteid`によるデータの集計が劣る場合、`city`および`pv`のみを持つロールアップを作成することが考えられます。

  ~~~sql
  ALTER TABLE site_visit ADD ROLLUP rollup_city(city, pv);
  ~~~

- ベーステーブルでのプレフィックス・インデックスがヒットしない場合、ベーステーブルの構築方法ではすべてのクエリパターンをカバーできない場合、ロールアップを作成して列の順序を調整することを検討してください。上記の`session_data`テーブルを例にとってみましょう:

  ~~~sql
  session_data(visitorid, sessionid, visittime, city, province, ip, brower, url)
  ~~~

  `browser`および`province`の分析が`visitorid`に加えて必要な場合、別のロールアップを作成することが考えられます:

  ~~~sql
  ALTER TABLE session_data
  ADD ROLLUP rollup_brower(brower,province,ip,url)
  DUPLICATE KEY(brower,province);
  ~~~

## スキーマ変更

StarRocksでスキーマを変更するには3つの方法があります: ソートされたスキーマ変更、直接的なスキーマ変更、リンクされたスキーマ変更です。

- ソートされたスキーマ変更: 列のソートを変更し、データを再順序します。例えば、ソートされたスキーマの列を削除することにより、データの再順序が発生します。

  `ALTER TABLE site_visit DROP COLUMN city;`

- 直接的なスキーマ変更: データを再順序するのではなくデータを変換します。例えば、列の型を変更したり、スパース・インデックスに列を追加しました。

  `ALTER TABLE site_visit MODIFY COLUMN username varchar(64);`

- リンクされたスキーマ変更: データを変換することなく変更を完了します。例えば、列を追加することです。

  `ALTER TABLE site_visit ADD COLUMN click bigint SUM default '0';`

  スキーマの変更を早めるためには、テーブルを作成する際に適切なスキーマを選択することをお勧めします。