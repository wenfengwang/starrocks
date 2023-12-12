---
displayed_sidebar: "Japanese"
---

# パフォーマンスの最適化

## テーブルタイプの選択

StarRocksは4つのテーブルタイプをサポートしています：Duplicate Key テーブル、Aggregate テーブル、Unique Key テーブル、および Primary Key テーブル。それらはすべてKEYでソートされています。

- `AGGREGATE KEY`: StarRocksに同じAGGREGATE KEYのレコードがロードされると、古いレコードと新しいレコードが集約されます。現在、集計テーブルは以下の集計関数をサポートしています：SUM, MIN, MAX, REPLACE。集計テーブルはデータを事前に集計しており、ビジネスの報告書や多次元分析を容易にしています。
- `DUPLICATE KEY`: DUPLICATE KEYテーブルでは、DUPLICATE KEYのソートキーのみを指定すればよいです。同じDUPLICATE KEYを持つレコードは同時に存在します。事前にデータの集計が関わらない分析に適しています。
- `UNIQUE KEY`: StarRocksに同じUNIQUE KEYのレコードがロードされると、新しいレコードが古いレコードを上書きします。UNIQUE KEYテーブルはREPLACE機能を持つ集計テーブルと似ています。両方とも定期的な更新が関わる分析に適しています。
- `PRIMARY KEY`: Primary Keyテーブルはレコードの一意性を保証し、リアルタイムの更新を行うことができます。

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

## テーブルのコロケーション

クエリの高速化のために、同じ分布を持つテーブルは共通のバケット化カラムを使用することができます。その場合、データは`join`操作中にクラスタ内でローカルに結合されるため、転送されることなく結合されます。

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

コロケーションジョインやレプリカ管理に関する詳細情報は、[コロケーションジョイン](../using_starrocks/Colocate_join.md) を参照してください。

## フラットテーブルとスタースキーマ

StarRocksはスタースキーマをサポートしており、フラットテーブルよりもモデリングの柔軟性が高いです。モデリング中にフラットテーブルの代わりにビューを作成し、複数のテーブルからデータをクエリすることでクエリを高速化することができます。

フラットテーブルには以下のような欠点があります：

- フラットテーブルには通常多数の次元が含まれているため、次元を更新するときに費用がかかります。次元が更新されるたびにテーブル全体を更新する必要があります。更新頻度が増えると状況は悪化します。
- フラットテーブルには追加の開発作業負荷、ストレージスペース、およびデータのバックフィリング操作が必要でメンテナンスコストが高くなります。
- フラットテーブルには多くのフィールドが存在し、集計テーブルにはさらに多くのキーフィールドが含まれるため、データローディング中には多くのフィールドをソートする必要があり、それによりデータローディングが長引きます。

クエリの同時実行数が高いか低いレイテンシが必要な場合、フラットテーブルを使用することができます。

## パーティションとバケット

StarRocksは2つのレベルのパーティショニングをサポートしています：1つ目は範囲パーティション、2つ目はハッシュバケットです。

- 範囲パーティション：範囲パーティションはデータを異なる間隔で分割するために使用されます（元のテーブルを複数のサブテーブルに分割することと理解できます）。ほとんどのユーザーは時間によってパーティションを設定することを選択します。以下の利点があります：

  - ホットデータとコールドデータを簡単に区別できます
  - StarRocksティアドストレージ（SSD + SATA）を活用することができます
  - パーティションごとにデータを削除するのが早くなります

- ハッシュバケット：ハッシュ値に基づいてデータを異なるバケットに分割します。

  - バケット化には差別性の高いカラムを使用することをお勧めします。データの偏りを避けるためです。
  - データの回復を容易にするために、各バケット内の圧縮データのサイズを100MBから1GBの間に保つことをお勧めします。テーブルを作成するかパーティションを追加する際には、適切な数のバケットを設定することをお勧めします。
  - ランダムなバケット化はお勧めしません。テーブルを作成する際には、明示的にHASHバケット化カラムを指定する必要があります。

## スパースインデックスとブルームフィルタインデックス

StarRocksはデータを順序付けて格納し、スパースインデックスを1024行単位で構築します。

StarRocksはスキーマ内の固定長のプレフィックス（現在は36バイト）をスパースインデックスとして選択します。

テーブルを作成する際には、スキーマ宣言の最初の部分に一般的なフィルタフィールドを配置することをお勧めします。差別化が最も高いクエリ頻度の高いフィールドを最初に配置する必要があります。

VARCHARフィールドはスパースインデックスの末尾に配置する必要があります。なぜなら、VARCHARフィールドからインデックスが切り詰められるかもしれません。VARCHARフィールドが最初に配置されていると、インデックスが36バイト以下になる可能性があります。

上記の`site_visit`テーブルを例に取ると、テーブルには`siteid, city, username, pv`の4つの列があります。ソートキーには`siteid、city、username`の3つの列が含まれており、それぞれが4、2、32バイトを占有しています。そのため、プレフィックスインデックス（スパースインデックス）は`siteid + city + username`の最初の30バイトになります。

スパースインデックスに加えて、StarRocksはブルームフィルタインデックスも提供しており、差別性が高い列をフィルタするのに効果的です。VARCHARフィールドを他のフィールドの前に配置したい場合は、ブルームフィルタインデックスを作成することができます。

## 逆インデックス

StarRocksではBitmap Indexingテクノロジーを採用し、全てのDuplicate Keyテーブル、Aggregateテーブルのキーカラム、およびUnique Keyテーブルのキーカラムに逆インデックスを適用することができます。Bitmap Indexは、性別、都道府県などの値の範囲が小さい列に適しています。範囲が拡大するにつれて、ビットマップインデックスも平行して拡大します。

## マテリアライズドビュー（ロールアップ）

ロールアップは基底テーブル（元のテーブル）のマテリアライズドインデックスです。ロールアップを作成する際には、基底テーブルの一部の列をスキーマとして選択し、スキーマ内のフィールドの順序を基底テーブルと異なる順序にすることができます。以下はロールアップの使用事例の一部です：

- 基底テーブルでデータの集計が高くない場合、差別性の高いフィールドを持つため、ロールアップを作成することが考えられます。上記の`site_visit`テーブルを例に取ると：

  ~~~sql
  site_visit(siteid, city, username, pv)
  ~~~

  `siteid`はデータの集計を悪化させる可能性があります。都市ごとのPVを頻繁に集計する必要がある場合は、`city`と`pv`のみを持つロールアップを作成することができます。

  ~~~sql
  ALTER TABLE site_visit ADD ROLLUP rollup_city(city, pv);
  ~~~

- 基底テーブルでのプレフィックスインデックスがヒットしない場合、基底テーブルの構築方法がすべてのクエリパターンをカバーできない場合が考えられます。この場合は、カラムの順序を調整するためにロールアップを作成することを検討することができます。上記の`session_data`テーブルを例に取ると：

  ~~~sql
  session_data(visitorid, sessionid, visittime, city, province, ip, brower, url)
  ~~~

  `visitorid`に加えて`browser`と`province`で訪問を分析する必要がある場合、独立したロールアップを作成することができます。

  ~~~sql
  ALTER TABLE session_data
  ADD ROLLUP rollup_browser(brower,province,ip,url)
  DUPLICATE KEY(brower,province);
  ~~~

## スキーマ変更

StarRocksでスキーマを変更する方法は3つあります：ソートされたスキーマ変更、直接のスキーマ変更、リンクされたスキーマ変更です。

- ソートされたスキーマ変更：列のソーティングを変更してデータを並べ替えます。たとえば、ソートされたスキーマの列を削除することでデータが再配置されます。

  `ALTER TABLE site_visit DROP COLUMN city;`

- 直接のスキーマ変更：データを再配置する代わりに変換します。たとえば、列の型を変更したり、スパースインデックスに列を追加することができます。

  `ALTER TABLE site_visit MODIFY COLUMN username varchar(64);`

- リンクされたスキーマ変更：データを変換する代わりに変更を完了します。たとえば、列を追加します。

  `ALTER TABLE site_visit ADD COLUMN click bigint SUM default '0';`

  スキーマ変更を高速化するために適切なスキーマを選択することをお勧めします。