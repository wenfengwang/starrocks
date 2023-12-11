---
displayed_sidebar: "Japanese"
---

# ソートキーとプレフィックスインデックス

テーブルを作成する際、その一つまたは複数の列を選択してソートキーを構成することができます。ソートキーによって、テーブルのデータがディスクに保存される前に整列される順序が決まります。また、クエリのフィルタ条件として、ソートキーの列を使用することができます。これによりStarRocksは、関心のあるデータを迅速に特定し、処理に必要なデータを検索するためにテーブル全体をスキャンする必要がなくなります。これにより、検索の複雑さが低減され、したがってクエリが加速されます。

また、メモリ消費量を削減するために、StarRocksはテーブルにプレフィックスインデックスを作成することをサポートしています。プレフィックスインデックスは、スペアインデックスの一種です。StarRocksは、テーブルの1024行ごとにブロックに保存し、そのブロックの先頭行における、テーブルのソートキー列からなるプレフィックスから、インデックスエントリが生成され、プレフィックスインデックステーブルに保存されます。プレフィックスインデックステーブルのブロックに対するプレフィックスインデックスエントリは36バイトを超えることはできず、その内容はその行のデータを格納するブロックの開始列番号を際立たせます。テーブルのプレフィックスインデックスは、サイズがテーブルそのものの1024倍少ないため、テーブル全体のプレフィックスインデックスをクエリの高速化に寄与するためにメモリにキャッシュできます。

## 原則

重複キーのテーブルでは、ソートキーの列は`DUPLICATE KEY`キーワードを使用して定義されます。

集約テーブルでは、ソートキーの列は`AGGREGATE KEY`キーワードを使用して定義されます。

ユニークキーのテーブルでは、ソートキーの列は`UNIQUE KEY`キーワードを使用して定義されます。

v3.0以降、プライマリキーとソートキーはプライマリキーテーブルにおいて切り離されます。そのため、ソートキーの列は`ORDER BY`キーワードを使用して定義されます。プライマリキーの列は`PRIMARY KEY`キーワードを使用して定義されます。

重複キーのテーブル、集約テーブル、またはユニークキーのテーブルにソートキー列を定義する際には、以下の点に注意してください。

- ソートキーの列は連続して定義された列でなければならず、最初に定義される列が開始ソートキー列でなければなりません。

- ソートキーの列として選択する予定の列は、他の一般的な列よりも前に定義されなければなりません。

- ソートキーの列をリストアップする順序は、テーブルの列を定義する順序に準拠しなければなりません。

以下の例は、列`site_id`、`city_code`、`user_id`、および`pv`から構成される、4つの列からなるテーブルの許可されたソートキー列および許可されていないソートキー列を示しています。

- 許可されたソートキー列の例
  - `site_id`と`city_code`
  - `site_id`、`city_code`、および`user_id`

- 許可されていないソートキー列の例
  - `city_code`と`site_id`
  - `city_code`と`user_id`
  - `site_id`、`city_code`、`pv`

以下のセクションでは、異なるタイプのテーブルを作成する際に、ソートキー列をどのように定義するかについての例を提供しています。これらの例は、少なくとも3つのBEを持つStarRocksクラスタに適しています。

### 重複キー

`site_access_duplicate`という名前のテーブルを作成します。このテーブルは、`site_id`、`city_code`、`user_id`、および`pv`の4つの列から構成されており、そのうち`site_id`と`city_code`がソートキー列として選択されています。

テーブルを作成するための文は以下のとおりです。

```SQL
CREATE TABLE site_access_duplicate
(
    site_id INT DEFAULT '10',
    city_code SMALLINT,
    user_id INT,
    pv BIGINT DEFAULT '0'
)
DUPLICATE KEY(site_id, city_code)
DISTRIBUTED BY HASH(site_id);
```

> **お知らせ**
>
> v2.5.7以降、StarRocksは、テーブルを作成したり、パーティションを追加したりする際にバケツ（BUCKETS）の数を自動的に設定できます。バケツの数を手動で設定する必要はもはやありません。詳細については、[バケツの数を決定](./Data_distribution.md#determine-the-number-of-buckets)を参照してください。

### 集約キー

`site_access_aggregate`という名前のテーブルを作成します。このテーブルは、`site_id`、`city_code`、`user_id`、および`pv`の4つの列から構成されており、そのうち`site_id`と`city_code`がソートキー列として選択されています。

テーブルを作成するための文は以下のとおりです。

```SQL
CREATE TABLE site_access_aggregate
(
    site_id INT DEFAULT '10',
    city_code SMALLINT,
    user_id BITMAP BITMAP_UNION,
    pv BIGINT SUM DEFAULT '0'
)
AGGREGATE KEY(site_id, city_code)
DISTRIBUTED BY HASH(site_id);
```

> **お知らせ**
>
> 集約テーブルの場合、`agg_type`が指定されていない列はキーカラムであり、`agg_type`が指定されている列は値カラムです。[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。前述の例では、`site_id`と`city_code`のみがソートキー列として指定されており、そのため、`user_id`と`pv`には`agg_type`が指定されなければなりません。

### ユニークキー

`site_access_unique`という名前のテーブルを作成します。このテーブルは、`site_id`、`city_code`、`user_id`、および`pv`の4つの列から構成されており、そのうち`site_id`と`city_code`がソートキー列として選択されています。

テーブルを作成するための文は以下のとおりです。

```SQL
CREATE TABLE site_access_unique
(
    site_id INT DEFAULT '10',
    city_code SMALLINT,
    user_id INT,
    pv BIGINT DEFAULT '0'
)
UNIQUE KEY(site_id, city_code)
DISTRIBUTED BY HASH(site_id);
```

### プライマリキー

`site_access_primary`という名前のテーブルを作成します。このテーブルは、`site_id`、`city_code`、`user_id`、および`pv`の4つの列から構成されており、そのうち`site_id`がプライマリキー列として選択され、`site_id`と`city_code`がソートキー列として選択されています。

テーブルを作成するための文は以下のとおりです。

```SQL
CREATE TABLE site_access_primary
(
    site_id INT DEFAULT '10',
    city_code SMALLINT,
    user_id INT,
    pv BIGINT DEFAULT '0'
)
PRIMARY KEY(site_id)
DISTRIBUTED BY HASH(site_id)
ORDER BY(site_id,city_code);
```

## ソーティング効果

前述のテーブルを例に、次の3つの状況におけるソーティング効果は異なります。

- もしクエリが`site_id`および`city_code`の両方をフィルタする場合、クエリ実行時にStarRocksがスキャンする行の数が著しく減少します。

  ```Plain
  select sum(pv) from site_access_duplicate where site_id = 123 and city_code = 2;
  ```

- もしクエリが`site_id`だけをフィルタする場合、StarRocksはクエリ範囲を`site_id`の値を含む行まで絞り込むことができます。

  ```Plain
  select sum(pv) from site_access_duplicate where site_id = 123;
  ```

- もしクエリが`city_code`だけをフィルタする場合、StarRocksはテーブル全体をスキャンしなければなりません。

  ```Plain
  select sum(pv) from site_access_duplicate where city_code = 2;
  ```

  > **注意**
  >
  > この状況では、ソートキー列は期待されるソーティング効果をもたらしません。

上記のように、クエリが`site_id`および`city_code`の両方をフィルタする場合、StarRocksはバイナリサーチを実行して、クエリ範囲を特定の場所に絞り込みます。テーブルが多数の行から成る場合、StarRocksは`site_id`および`city_code`のデータをメモリにロードし、メモリ消費量を増加させる必要があります。この場合、プレフィックスインデックスを使用して、メモリにキャッシュされるデータの量を減らし、クエリを加速できます。

さらに、多数のソートキー列もメモリ消費量を増加させます。メモリ消費量を削減するために、StarRocksはプレフィックスインデックスの使用に以下の制限を課しています。

- ブロックのプレフィックスインデックスエントリは、そのブロックの最初の行にあるテーブルのソートキー列のプレフィックスから構成されなければなりません。

- プレフィックスインデックスは、最大で3つの列に作成できます。

- プレフィックスインデックスエントリは、36バイトを超えることはできません。

- FLOATまたはDOUBLEのデータ型の列にプレフィックスインデックスを作成することはできません。

- プレフィックスインデックスが作成されたすべての列の中で1つのVARCHARデータ型の列のみが許可され、その列はプレフィックスインデックスの最後尾の列でなければなりません。

- プレフィックスインデックスの終端列がCHARまたはVARCHARデータ型の場合、プレフィックスインデックスのエントリは36バイトを超えてはいけません。

## ソートキー列の選択方法

このセクションでは、`site_access_duplicate`テーブルを例に、ソートキー列の選択方法について説明します。

- よくフィルタされるクエリによって選択される列をソートキー列として選択することをお勧めします。

- 複数のソートキー列を選択する場合は、高い識別レベルを持つ頻繁にフィルタされる列を他の列の前にリストアップすることをお勧めします。
  + 提案
    + 大きな値の数と継続的な増加がある場合、カラムの識別レベルが高いです。例えば、`site_access_duplicate` テーブルの都市の数は固定されているため、テーブルの `city_code` カラムの値の数も固定されています。しかし、`site_id` カラムの値の数は `city_code` カラムの値の数よりずっと多く、かつ継続的に増加しています。そのため、`site_id` カラムの方が `city_code` カラムよりも識別レベルが高いです。

- 多くのソートキー列を選択しないことをお勧めします。多くのソートキー列はクエリのパフォーマンスを向上させることができませんが、ソーティングやデータの読み込みの処理コストを増加させます。

以上を踏まえて、`site_access_duplicate` テーブルのソートキー列を選択する際には以下の点に注意してください:

- クエリが頻繁に `site_id` と `city_code` の両方でフィルタリングする場合には、最初のソートキー列として `site_id` を選択することをお勧めします。

- クエリが頻繁に `city_code` のみでフィルタリングし、場合によっては `site_id` と `city_code` の両方でフィルタリングする場合には、最初のソートキー列として `city_code` を選択することをお勧めします。

- クエリが `site_id` と `city_code` の両方でフィルタリングを行う回数が、`city_code` のみでフィルタリングを行う回数にほぼ等しい場合には、`city_code` を最初のカラムとするマテリアライズドビューを作成することをお勧めします。これにより、StarRocks はマテリアライズドビューの `city_code` カラムにソートインデックスを作成します。