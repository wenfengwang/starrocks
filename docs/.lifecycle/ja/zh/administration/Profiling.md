---
displayed_sidebar: Chinese
---
# パフォーマンスの最適化

このドキュメントでは、StarRocksのパフォーマンスを最適化する方法について説明します。

## テーブルの作成によるパフォーマンスの最適化

StarRocksのパフォーマンスを最適化するために、テーブルの作成時に以下の方法を使用することができます。

### データモデルの選択

StarRocksは、主キーモデル（PRIMARY KEY）、集約キーモデル（AGGREGATE KEY）、ユニークキーモデル（UNIQUE KEY）、およびデュプリケートキーモデル（DUPLICATE KEY）の4つのデータモデルをサポートしています。これらのモデルでは、データはすべてキーに基づいてソートされます。

* **AGGREGATE KEYモデル**

    AGGREGATE KEYが同じ場合、新しいレコードと古いレコードが集約されます。現在サポートされている集約関数には、SUM、MIN、MAX、およびREPLACEがあります。AGGREGATE KEYモデルは、データを事前に集約するため、レポートや多次元分析のビジネスに適しています。

    ```sql
    CREATE TABLE site_visit
    (
        siteid      INT,
        city        SMALLINT,
        username    VARCHAR(32),
        pv BIGINT   SUM DEFAULT '0'
    )
    AGGREGATE KEY(siteid, city, username)
    DISTRIBUTED BY HASH(siteid);
    ```

* **UNIQUE KEYモデル**

    UNIQUE KEYが同じ場合、新しいレコードが古いレコードを上書きします。現在のUNIQUE KEYの実装は、AGGREGATE KEYのREPLACE集約関数と同じです。したがって、これらは本質的に同じと見なすことができます。UNIQUE KEYモデルは、更新がある分析ビジネスに適しています。

    ```sql
    CREATE TABLE sales_order
    (
        orderid     BIGINT,
        status      TINYINT,
        username    VARCHAR(32),
        amount      BIGINT DEFAULT '0'
    )
    UNIQUE KEY(orderid)
    DISTRIBUTED BY HASH(orderid);
    ```

* **DUPLICATE KEYモデル**

    DUPLICATE KEYはソートにのみ使用され、同じDUPLICATE KEYのレコードは同時に存在します。DUPLICATE KEYモデルは、データを事前に集約する必要のない分析ビジネスに適しています。

    ```sql
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
    ```

* **PRIMARY KEYモデル**

    PRIMARY KEYモデルは、同じ主キーのレコードが1つだけ存在することを保証します。更新モデルと比較して、主キーモデルはクエリ時に集約操作を実行する必要がなく、述語とインデックスのプッシュダウンをサポートしており、リアルタイムおよび頻繁な更新などのシナリオで効率的なクエリを提供することができます。

    ```sql
    CREATE TABLE orders (
        dt date NOT NULL,
        order_id bigint NOT NULL,
        user_id int NOT NULL,
        merchant_id int NOT NULL,
        good_id int NOT NULL,
        good_name string NOT NULL,
        price int NOT NULL,
        cnt int NOT NULL,
        revenue int NOT NULL,
        state tinyint NOT NULL
    )
    PRIMARY KEY (dt, order_id)
    DISTRIBUTED BY HASH(order_id);
    ```

### Colocateテーブルの使用

StarRocksは、同じ分散を持つ関連するテーブルを共通のバケット列として保存し、関連するテーブルのJOIN操作を直接ローカルで実行してクエリを高速化することができます。詳細については、[Colocate Join](../using_starrocks/Colocate_join.md)を参照してください。

```sql
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
```

### スターシェープモデルの使用

StarRocksは、伝統的なワイドテーブルのモデリング方法に代わるより柔軟なスターシェープモデル（スタースキーマ）を選択することができます。スターシェープモデルでは、ワイドテーブルの代わりにビューを使用してモデリングし、複数のテーブルの結合を直接使用してクエリを実行することができます。SSBの標準テストセットの比較では、StarRocksのマルチテーブル結合のパフォーマンスは、シングルテーブルクエリと比較して明らかな低下はありません。

ワイドテーブルに比べて、スターシェープモデルの欠点は次のとおりです。

* 寸法の更新コストが高い。ワイドテーブルでは、寸法情報の更新はテーブル全体に反映され、更新の頻度がクエリの効率に直接影響します。
* メンテナンスコストが高い。ワイドテーブルの構築には、追加の開発作業、ストレージスペース、およびデータバックフィルのコストが必要です。
* インポートコストが高い。ワイドテーブルのスキーマには、フィールド数が多く、集約モデルにはより多くのキーカラムが含まれる場合があり、インポートプロセスではソートする必要があり、それによりインポート時間が長くなります。

スターシェープモデルを優先することをお勧めします。柔軟性を確保しながら、効率的な指標分析効果を得ることができます。ただし、ビジネスが高並行性や低遅延を要求する場合は、ワイドテーブルモデルを選択することもできます。StarRocksは、ClickHouseと同等のワイドテーブルクエリパフォーマンスを提供します。

### パーティションとバケットの使用

StarRocksは、2段階のパーティションストレージをサポートしています。第1段階はRANGEパーティションであり、第2段階はHASHバケットです。

RANGEパーティションは、データを異なる範囲に分割するために使用され、論理的には元のテーブルを複数のサブテーブルに分割したものと同等です。プロダクション環境では、多くのユーザーが時間に基づいてパーティションを作成します。時間に基づいてパーティションを作成することには、次の利点があります。

* ホット/コールドデータを区別できます。
* StarRocksの階層ストレージ（SSD + SATA）機能を使用できます。
* パーティションごとにデータを削除する場合、より迅速に削除できます。

HASHバケットは、ハッシュ値に基づいてデータを異なるバケットに分割します。

* バケット分割には、分割度の高い列を使用することをお勧めします。データの偏りを防ぐためです。
* データの復元を容易にするために、個々のバケットのサイズを小さく保つことをお勧めします。圧縮後のサイズが100MBから1GB程度になるようにしてください。テーブルの作成またはパーティションの追加時に、バケットの数を適切に考慮することをお勧めします。異なるパーティションでは、異なるバケット数を指定できます。
* ランダムなバケット分割方法はお勧めしません。テーブルを作成する際に、明確なハッシュバケット列を指定してください。

### スパースインデックスとBloomフィルタの使用

StarRocksは、データを順序付けてスパースインデックスを作成し、インデックスの粒度をブロック（1024行）に設定することができます。

スパースインデックスは、スキーマの固定長のプレフィックスをインデックスの内容として選択します。現在、StarRocksは36バイトのプレフィックスをインデックスとして選択しています。

テーブルを作成する際には、クエリでよく使用されるフィルタリングフィールドをスキーマの前部に配置することをお勧めします。区別度が高く、頻度が高いクエリフィールドは、より前部に配置する必要があります。VARCHAR型のフィールドは、スパースインデックスの最後のフィールドにのみ使用できます。なぜなら、インデックスはVARCHARフィールドで切り捨てられるためです。VARCHARデータが先頭にある場合、インデックスの長さが36バイトに満たない可能性があります。

例えば、上記の`site_visit`テーブルでは、`siteid`、`city`、`username`の3つの列でソートされます。`siteid`は4バイト、`city`は2バイト、`username`は32バイトを占めるため、このテーブルのプレフィックスインデックスの内容は`city` + `username`の先頭30バイトです。

スパースインデックス以外にも、StarRocksはBloomフィルタインデックスも提供しています。Bloomフィルタインデックスは、区分度の高い列に対してフィルタリング効果があります。VARCHARフィールドを先頭に配置する必要がある場合は、Bloomフィルタインデックスを作成することができます。

### 逆インデックスの使用

StarRocksは逆インデックスをサポートしており、ビットマップ技術を使用してインデックス（ビットマップインデックス）を構築します。逆インデックスは、DUPLICATE KEYデータモデルのすべての列およびAGGREGATE KEYおよびUNIQUE KEYデータモデルのキーカラムに適用することができます。ビットマップインデックスは、性別、都市、州などの情報列など、値の範囲が比較的小さい列に適しています。値の範囲が増加するにつれて、ビットマップインデックスは膨張します。

### マテリアライズドビューの使用

マテリアライズドビュー（ロールアップ）は、基本テーブルの物理的なインデックスです。マテリアライズドビューを作成する際には、基本テーブルの一部の列をスキーマとして選択することができます。スキーマのフィールドの順序も基本テーブルと異なる場合があります。次の場合には、マテリアライズドビューを作成することを検討してください。

* 基本テーブルのデータの集約度が低い場合。これは通常、基本テーブルに区分度の高いフィールドが含まれているためです。たとえば、上記の`site_visit`テーブルでは、`siteid`がデータの集約度を低下させる可能性があります。`city`と`pv`に基づいて頻繁に`pv`を統計する必要がある場合は、`city`と`pv`のみを含むマテリアライズドビューを作成できます。

    ```sql
    ALTER TABLE site_visit ADD ROLLUP rollup_city(city, pv);
    ```

* 基本テーブルのプレフィックスインデックスがヒットしない場合。これは通常、基本テーブルの作成方法がすべてのクエリパターンをカバーできない場合です。この場合、列の順序を調整し、マテリアライズドビューを作成することを検討してください。上記の`session_data`テーブルの場合、`visitorid`に基づくアクセス分析だけでなく、`brower`と`province`に基づく分析もある場合は、個別にマテリアライズドビューを作成できます。

    ```sql
    ALTER TABLE session_data ADD ROLLUP rollup_brower(brower,province,ip,url) DUPLICATE KEY(brower,province);
    ```

## インポートパフォーマンスの最適化

StarRocksは現在、Broker LoadとStream Loadの2つのインポート方法を提供しており、インポートラベルを指定して一括インポートを保証します。StarRocksは、単一のバッチのインポートをアトミックに有効にするため、単一のインポートに複数のテーブルが含まれていても同じように保証します。

* Stream Load：HTTPストリーミングを使用してデータをインポートする方法で、マイクロバッチインポートに使用されます。このモードでは、1MBのデータのインポート遅延を秒単位で維持することができ、高頻度のインポートに適しています。
* Broker Load：プル方式を使用してデータを一括インポートする方法で、大量のデータのインポートに適しています。

## スキーマ変更のパフォーマンスの最適化

StarRocksは現在、ソートされたスキーマ変更、直接のスキーマ変更、リンクされたスキーマ変更の3つの方法をサポートしています。

* ソートされたスキーマ変更：列の並べ替え方法を変更し、データを再度並べ替える必要があります。たとえば、ソート列から列を削除したり、フィールドの並べ替えを行ったりします。

    ```sql
    ALTER TABLE site_visit DROP COLUMN city;
    ```

* 直接のスキーマ変更：再度並べ替える必要はありませんが、データを変換する必要があります。たとえば、列の型を変更したり、スパースインデックスに列を追加したりします。

    ```sql
    ALTER TABLE site_visit MODIFY COLUMN username varchar(64);
    ```

* リンクされたスキーマ変更：データの変換は必要ありませんが、直接完了します。たとえば、列の追加操作です。

    ```sql
    ALTER TABLE site_visit ADD COLUMN click bigint SUM default '0';
    ```

スキーマを作成する際には、スキーマ変更の速度を向上させるためにスキーマを考慮することをお勧めします。