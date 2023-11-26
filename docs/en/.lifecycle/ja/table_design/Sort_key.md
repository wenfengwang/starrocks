---
displayed_sidebar: "Japanese"
---

# ソートキーとプレフィックスインデックス

テーブルを作成する際に、1つまたは複数の列を選択してソートキーを構成することができます。ソートキーは、データがディスクに格納される前にテーブルのデータがソートされる順序を決定します。ソートキーの列は、クエリのフィルタ条件として使用することができます。そのため、StarRocksは興味のあるデータを素早く見つけることができ、処理するためにテーブル全体をスキャンする必要がありません。これにより、検索の複雑さが低減され、クエリの実行が高速化されます。

さらに、メモリ消費を削減するために、StarRocksはテーブルにプレフィックスインデックスを作成することもサポートしています。プレフィックスインデックスは、スペアインデックスの一種です。StarRocksはテーブルの1024行ごとにブロックを格納し、そのブロックのプレフィックスインデックステーブルにインデックスエントリが生成されて格納されます。ブロックのプレフィックスインデックスエントリの長さは36バイトを超えることはできず、その内容はそのブロックの最初の行にあるテーブルのソートキー列のプレフィックスです。これにより、プレフィックスインデックステーブルの検索時に、その行のデータを格納するブロックの開始列番号をStarRocksが素早く特定することができます。テーブルのプレフィックスインデックスは、テーブル自体のサイズの1024分の1です。したがって、プレフィックスインデックス全体をメモリにキャッシュしてクエリの高速化に役立てることができます。

## 原則

重複キーテーブルでは、ソートキー列は `DUPLICATE KEY` キーワードを使用して定義されます。

集計キーテーブルでは、ソートキー列は `AGGREGATE KEY` キーワードを使用して定義されます。

一意キーテーブルでは、ソートキー列は `UNIQUE KEY` キーワードを使用して定義されます。

v3.0以降、プライマリキーテーブルではプライマリキーとソートキーが切り離されています。ソートキー列は `ORDER BY` キーワードを使用して定義されます。プライマリキー列は `PRIMARY KEY` キーワードを使用して定義されます。

重複キーテーブル、集計キーテーブル、または一意キーテーブルにソートキー列を定義する場合は、次の点に注意してください。

- ソートキー列は連続して定義された列でなければなりません。最初に定義された列は開始ソートキー列である必要があります。

- ソートキー列として選択する予定の列は、他の一般的な列よりも前に定義されている必要があります。

- ソートキー列のリストの順序は、テーブルの列を定義する順序に準拠する必要があります。

次の例は、`site_id`、`city_code`、`user_id`、`pv` の4つの列からなるテーブルの許可されるソートキー列と許可されないソートキー列を示しています。

- 許可されるソートキー列の例
  - `site_id` と `city_code`
  - `site_id`、`city_code`、`user_id`

- 許可されないソートキー列の例
  - `city_code` と `site_id`
  - `city_code` と `user_id`
  - `site_id`、`city_code`、`pv`

次のセクションでは、異なるタイプのテーブルを作成する際にソートキー列をどのように定義するかの例を示します。これらの例は、少なくとも3つのBEを持つStarRocksクラスタに適しています。

### 重複キー

`site_access_duplicate` という名前のテーブルを作成します。テーブルは `site_id`、`city_code`、`user_id`、`pv` の4つの列からなります。そのうち `site_id` と `city_code` をソートキー列として選択します。

テーブルを作成するためのステートメントは次のとおりです:

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
> v2.5.7以降、StarRocksはテーブルを作成するかパーティションを追加する際に、バケットの数（BUCKETS）を自動的に設定することができます。バケットの数を手動で設定する必要はもはやありません。詳細については、[バケットの数を決定する](./Data_distribution.md#determine-the-number-of-buckets)を参照してください。

### 集計キー

`site_access_aggregate` という名前のテーブルを作成します。テーブルは `site_id`、`city_code`、`user_id`、`pv` の4つの列からなります。そのうち `site_id` と `city_code` をソートキー列として選択します。

テーブルを作成するためのステートメントは次のとおりです:

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
> 集計キーテーブルでは、`agg_type` が指定されていない列はキーカラムであり、`agg_type` が指定されている列は値カラムです。詳細については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。前述の例では、`site_id` と `city_code` のみがソートキー列として指定されているため、`user_id` と `pv` には `agg_type` が指定される必要があります。

### 一意キー

`site_access_unique` という名前のテーブルを作成します。テーブルは `site_id`、`city_code`、`user_id`、`pv` の4つの列からなります。そのうち `site_id` と `city_code` をソートキー列として選択します。

テーブルを作成するためのステートメントは次のとおりです:

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

`site_access_primary` という名前のテーブルを作成します。テーブルは `site_id`、`city_code`、`user_id`、`pv` の4つの列からなります。そのうち `site_id` をプライマリキーカラム、`site_id` と `city_code` をソートキー列として選択します。

テーブルを作成するためのステートメントは次のとおりです:

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

前述のテーブルを使用して、次の3つの状況でソート効果が異なります:

- `site_id` と `city_code` の両方でクエリをフィルタリングする場合、クエリ中でStarRocksがスキャンする行数が大幅に減少します:

  ```Plain
  select sum(pv) from site_access_duplicate where site_id = 123 and city_code = 2;
  ```

- クエリが `site_id` のみをフィルタリングする場合、StarRocksは `site_id` の値を含む行にクエリ範囲を狭めることができます:

  ```Plain
  select sum(pv) from site_access_duplicate where site_id = 123;
  ```

- クエリが `city_code` のみをフィルタリングする場合、StarRocksはテーブル全体をスキャンする必要があります:

  ```Plain
  select sum(pv) from site_access_duplicate where city_code = 2;
  ```

  > **注意**
  >
  > この状況では、ソートキー列は期待されるソート効果を発揮しません。

上記のように、`site_id` と `city_code` の両方でクエリをフィルタリングする場合、StarRocksはテーブルをバイナリサーチしてクエリ範囲を特定の位置に狭めます。テーブルが大量の行で構成されている場合、StarRocksは `site_id` と `city_code` の列に対してバイナリサーチを実行します。これにより、StarRocksは2つの列のデータをメモリにロードする必要があり、メモリ消費量が増加します。この場合、プレフィックスインデックスを使用してメモリにキャッシュするデータ量を減らすことで、クエリの高速化が可能です。

また、大量のソートキー列もメモリ消費量を増加させます。メモリ消費量を削減するために、StarRocksはプレフィックスインデックスの使用に以下の制限を課しています:

- ブロックのプレフィックスインデックスエントリは、そのブロックの最初の行にあるテーブルのソートキー列のプレフィックスで構成されなければなりません。

- プレフィックスインデックスは最大3つの列に作成することができます。

- プレフィックスインデックスエントリの長さは36バイトを超えることはできません。

- FLOATまたはDOUBLEデータ型の列にはプレフィックスインデックスを作成できません。

- プレフィックスインデックスが作成されるすべての列のうち、VARCHARデータ型の列は1つだけ許可され、その列はプレフィックスインデックスの終了列でなければなりません。

- プレフィックスインデックスの終了列がCHARまたはVARCHARデータ型の場合、プレフィックスインデックスのエントリは36バイトを超えることはできません。

## ソートキー列の選択方法

このセクションでは、`site_access_duplicate` テーブルを例にして、ソートキー列の選択方法について説明します。

- クエリで頻繁にフィルタリングされる列を特定し、これらの列をソートキー列として選択することを推奨します。

- 複数のソートキー列を選択する場合、高い識別レベルを持つ頻繁にフィルタリングされる列を他の列よりも前にリストアップすることを推奨します。
  
  列の識別レベルが高いとは、列の値の数が多く、連続的に増加することを意味します。たとえば、`site_access_duplicate` テーブルの都市の数は固定されており、つまりテーブルの `city_code` 列の値の数は固定されています。しかし、`site_id` 列の値の数は `city_code` 列の値の数よりもはるかに多く、連続的に増加しています。したがって、`site_id` 列は `city_code` 列よりも高い識別レベルを持っています。

- 大量のソートキー列を選択しないことを推奨します。大量のソートキー列はクエリのパフォーマンスを向上させることはできませんが、ソートやデータのロードにかかるオーバーヘッドを増加させます。

まとめると、`site_access_duplicate` テーブルのソートキー列を選択する際には、次の点に注意してください:

- `site_id` と `city_code` の両方でクエリが頻繁にフィルタリングされる場合、開始ソートキー列として `site_id` を選択することをおすすめします。

- クエリが頻繁に `city_code` のみをフィルタリングし、時折 `site_id` と `city_code` の両方をフィルタリングする場合、開始ソートキー列として `city_code` を選択することをおすすめします。

- `site_id` と `city_code` の両方をフィルタリングするクエリの回数が `city_code` のみをフィルタリングするクエリの回数とほぼ同じである場合、マテリアライズドビューを作成し、最初の列を `city_code` にすることをおすすめします。これにより、StarRocksはマテリアライズドビューの `city_code` 列にソートインデックスを作成します。
