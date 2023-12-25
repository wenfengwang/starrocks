---
displayed_sidebar: English
---

# ソートキーとプレフィックスインデックス

テーブルを作成する際に、1つ以上の列をソートキーとして選択することができます。ソートキーは、データがディスクに保存される前にテーブルのデータがどの順序でソートされるかを決定します。ソートキーの列はクエリのフィルタ条件として使用できるため、StarRocksは必要なデータを迅速に特定でき、テーブル全体をスキャンして処理すべきデータを見つける手間を省くことができます。これにより検索の複雑さが減り、クエリが加速されます。

さらに、メモリ消費を削減するため、StarRocksはテーブルにプレフィックスインデックスを作成することをサポートしています。プレフィックスインデックスはスパースインデックスの一種です。StarRocksはテーブルの1024行ごとにブロックを格納し、それぞれのブロックに対してインデックスエントリを生成してプレフィックスインデックステーブルに保存します。ブロックのプレフィックスインデックスエントリは36バイトを超えることはできず、その内容はそのブロックの最初の行にあるテーブルのソートキー列から構成されるプレフィックスです。これにより、StarRocksはプレフィックスインデックステーブルで検索を実行した際に、その行のデータを格納しているブロックの開始列番号を迅速に特定できます。テーブルのプレフィックスインデックスはテーブル自体のサイズの1024分の1です。そのため、プレフィックスインデックス全体をメモリにキャッシュしてクエリを加速することができます。

## 原則

Duplicate Keyテーブルでは、ソートキーの列は`DUPLICATE KEY`キーワードを使用して定義されます。

Aggregate Keyテーブルでは、ソートキーの列は`AGGREGATE KEY`キーワードを使用して定義されます。

Unique Keyテーブルでは、ソートキーの列は`UNIQUE KEY`キーワードを使用して定義されます。

v3.0以降、Primary Keyテーブルではプライマリキーとソートキーが分離されています。ソートキーの列は`ORDER BY`キーワードを使用して定義され、プライマリキーの列は`PRIMARY KEY`キーワードを使用して定義されます。

Duplicate Keyテーブル、Aggregate Keyテーブル、またはUnique Keyテーブルのソートキー列を定義する際には、以下の点に注意してください：

- ソートキー列は連続して定義された列でなければならず、最初に定義された列が開始ソートキー列でなければなりません。

- ソートキー列として選択する予定の列は、他の一般的な列よりも先に定義される必要があります。

- ソートキー列をリストする順序は、テーブルの列を定義する順序に準拠している必要があります。

以下の例は、`site_id`、`city_code`、`user_id`、`pv`の4列から構成されるテーブルの許可されるソートキー列と許可されないソートキー列を示しています：

- 許可されるソートキー列の例
  - `site_id` と `city_code`
  - `site_id`、`city_code`、および `user_id`

- 許可されないソートキー列の例
  - `city_code` と `site_id`
  - `city_code` と `user_id`
  - `site_id`、`city_code`、および `pv`

次のセクションでは、異なるタイプのテーブルを作成する際にソートキー列を定義する方法の例を提供します。これらの例は、少なくとも3つのBEを持つStarRocksクラスターに適しています。

### Duplicate Key

`site_access_duplicate`という名前のテーブルを作成します。このテーブルは`site_id`、`city_code`、`user_id`、`pv`の4列から構成され、`site_id`と`city_code`がソートキー列として選択されています。

テーブルを作成するためのステートメントは次のとおりです：

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

> **注意**
>
> v2.5.7以降、StarRocksはテーブルを作成する際やパーティションを追加する際にバケット数(BUCKETS)を自動的に設定することができます。バケット数を手動で設定する必要はもうありません。詳細については、[バケット数の決定](./Data_distribution.md#determine-the-number-of-buckets)を参照してください。

### Aggregate Key

`site_access_aggregate`という名前のテーブルを作成します。このテーブルは`site_id`、`city_code`、`user_id`、`pv`の4列から構成され、`site_id`と`city_code`がソートキー列として選択されています。

テーブルを作成するためのステートメントは次のとおりです：

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

>**注意**
>
> Aggregate Keyテーブルの場合、`agg_type`が指定されていない列はキー列であり、`agg_type`が指定されている列は値列です。詳細は[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。上記の例では、`site_id`と`city_code`のみがソートキー列として指定されているため、`user_id`と`pv`には`agg_type`を指定する必要があります。

### Unique Key

`site_access_unique`という名前のテーブルを作成します。このテーブルは`site_id`、`city_code`、`user_id`、`pv`の4列から構成され、`site_id`と`city_code`がソートキー列として選択されています。

テーブルを作成するためのステートメントは次のとおりです：

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

`site_access_primary` という名前のテーブルを作成します。このテーブルは4つのカラム `site_id`、`city_code`、`user_id`、`pv` から構成され、`site_id` がプライマリキーとして選択され、`site_id` と `city_code` がソートキーカラムとして選択されます。

テーブルを作成するためのステートメントは、次のとおりです：

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

## ソート効果

上記のテーブルを例に、ソート効果は次の3つの状況で異なります：

- クエリが `site_id` と `city_code` の両方でフィルタリングする場合、StarRocksがクエリ中にスキャンする必要がある行数が大幅に削減されます：

  ```Plain
  select sum(pv) from site_access_duplicate where site_id = 123 and city_code = 2;
  ```

- クエリが `site_id` のみでフィルタリングする場合、StarRocksはクエリ範囲を `site_id` の値を含む行に絞り込むことができます：

  ```Plain
  select sum(pv) from site_access_duplicate where site_id = 123;
  ```

- クエリが `city_code` のみでフィルタリングされる場合、StarRocksはテーブル全体をスキャンする必要があります：

  ```Plain
  select sum(pv) from site_access_duplicate where city_code = 2;
  ```

  > **注記**
  >
  > この状況では、ソートキーカラムは予期したソート効果をもたらしません。

前述したように、クエリが `site_id` と `city_code` の両方でフィルタリングすると、StarRocksはテーブルに対してバイナリ検索を実行し、クエリ範囲を特定の場所に絞り込みます。テーブルが多数の行で構成されている場合、StarRocksは `site_id` と `city_code` 列に対してバイナリ検索を実行します。これにより、StarRocksは2つの列のデータをメモリにロードする必要があるため、メモリ消費量が増加します。この場合、プレフィックスインデックスを使用してメモリにキャッシュされるデータの量を減らし、クエリを高速化できます。

また、ソートキーカラムの数が多いと、メモリ消費量も増加することに注意してください。メモリ消費を削減するために、StarRocksはプレフィックスインデックスの使用に次の制限を課しています：

- ブロックのプレフィックスインデックスエントリは、そのブロックの最初の行にあるテーブルのソートキーカラムのプレフィックスで構成される必要があります。

- プレフィックスインデックスは、最大3つのカラムに作成できます。

- プレフィックスインデックスエントリの長さは36バイトを超えることはできません。

- プレフィックスインデックスは、FLOATまたはDOUBLEデータ型のカラムには作成できません。

- プレフィックスインデックスが作成されるすべてのカラムのうち、VARCHARデータ型のカラムは1つだけ許可され、そのカラムはプレフィックスインデックスの終端カラムである必要があります。

- プレフィックスインデックスの終端カラムがCHARまたはVARCHARデータ型の場合、プレフィックスインデックスのエントリは36バイトを超えることはできません。

## ソートキーカラムの選択方法

このセクションでは、`site_access_duplicate` テーブルを例にソートキーカラムの選択方法について説明します。

- クエリが頻繁にフィルタリングするカラムを特定し、これらのカラムをソートキーカラムとして選択することを推奨します。

- 複数のソートキーカラムを選択する場合、頻繁にフィルタリングされ、識別レベルが高いカラムを他のカラムよりも前にリストすることを推奨します。
  
  カラムの識別レベルが高いとは、そのカラムの値の数が多く、継続的に増加することを意味します。例えば、`site_access_duplicate` テーブルの都市の数は固定されており、`city_code` カラムの値の数も固定されています。しかし、`site_id` カラムの値の数は `city_code` カラムの値の数よりもはるかに多く、継続的に増加しています。したがって、`site_id` カラムは `city_code` カラムよりも識別レベルが高いです。

- ソートキーカラムの数を多く選択しないことを推奨します。ソートキーカラムの数が多いと、クエリのパフォーマンスは向上しませんが、ソートとデータロードのオーバーヘッドが増加します。

要約すると、`site_access_duplicate` テーブルのソートキーカラムを選択する際には、以下の点に注意してください：

- クエリで `site_id` と `city_code` の両方が頻繁にフィルタリングされる場合、`site_id` を最初のソートキーカラムとして選択することを推奨します。

- クエリが `city_code` のみで頻繁にフィルタリングされ、`site_id` と `city_code` の両方でたまにフィルタリングされる場合、`city_code` を最初のソートキーカラムとして選択することを推奨します。
- クエリが`site_id`と`city_code`の両方でフィルタリングする回数が、`city_code`のみでフィルタリングする回数とほぼ同じである場合、最初の列が`city_code`であるマテリアライズドビューを作成することをお勧めします。そうすることで、StarRocksはマテリアライズドビューの`city_code`列にソートインデックスを作成します。
