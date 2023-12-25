---
displayed_sidebar: Chinese
---

# ソートキーとプレフィックスインデックス

テーブル作成時に、一つまたは複数の列をソートキー（Sort Key）として指定することができます。テーブル内の行はソートキーに基づいてソートされた後、ディスクに保存されます。データをクエリする際には、ソートキーの列をフィルタ条件として使用することで、StarRocksは全表をスキャンする必要がなく、必要なデータを迅速に見つけ出し、検索の複雑さを低減し、クエリを加速します。

また、メモリの使用量を減らすために、StarRocksはソートキーに基づいてプレフィックスインデックス（Prefix Index）を導入しました。プレフィックスインデックスはスパースインデックスの一種です。テーブル内の1024行ごとにデータブロック（Data Block）が形成されます。各データブロックはプレフィックスインデックステーブルにインデックス項目を一つ持ち、その長さは36バイトを超えません。内容はデータブロックの最初の行のソートキー列のプレフィックスで、プレフィックスインデックステーブルを検索する際に、その行のデータが属するデータブロックの開始行番号を特定するのに役立ちます。プレフィックスインデックスのサイズはデータ量の1024分の1以下であるため、メモリに全量キャッシュされ、実際の検索プロセスでクエリを効果的に加速することができます。

## ソートの原理

明細モデルでは、ソート列は `DUPLICATE KEY` キーワードで指定された列です。

集約モデルでは、ソート列は `AGGREGATE KEY` キーワードで指定された列です。

更新モデルでは、ソート列は `UNIQUE KEY` キーワードで指定された列です。

バージョン3.0から、主キーモデルでは主キー列とソート列が分離され、ソート列は `ORDER BY` キーワードで、主キー列は `PRIMARY KEY` キーワードで指定されます。

明細モデル、集約モデル、更新モデルでソート列を定義する際には、以下の点に注意する必要があります：

- ソート列は定義された最初の列から始まり、連続している必要があります。

- 各列を定義する際に、ソート列として計画されている列は他の通常の列よりも前に定義される必要があります。

- ソート列の順序はテーブル定義の列の順序と一致している必要があります。

例えば、テーブル作成文で `site_id`、`city_code`、`user_id`、`pv` の四つの列を作成すると宣言した場合、正しいソート列の組み合わせと誤ったソート列の組み合わせの例は以下の通りです：

- 正しいソート列
  - `site_id` と `city_code`
  - `site_id`、`city_code`、`user_id`

- 誤ったソート列
  - `city_code` と `site_id`
  - `city_code` と `user_id`
  - `site_id`、`city_code`、`pv`

以下の例を通じて、各データモデルを使用するテーブルの作成方法を説明します。以下のテーブル作成文は、少なくとも3つのBEノードがデプロイされたクラスタ環境で使用することができます。

### 明細モデル

`site_access_duplicate` という名前の明細モデルテーブルを作成し、`site_id`、`city_code`、`user_id`、`pv` の四つの列を含み、`site_id` と `city_code` がソート列です。

テーブル作成文は以下の通りです：

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
> バージョン2.5.7から、StarRocksはテーブル作成とパーティション追加時に自動的にバケット数（BUCKETS）を設定する機能をサポートしており、手動でバケット数を設定する必要はありません。詳細は [バケット数の決定](./Data_distribution.md#バケット数の決定) を参照してください。

### 集約モデル

`site_access_aggregate` という名前の集約モデルテーブルを作成し、`site_id`、`city_code`、`user_id`、`pv` の四つの列を含み、`site_id` と `city_code` がソート列です。

テーブル作成文は以下の通りです：

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
> 集約モデルテーブルでは、ある列が `agg_type` を指定していない場合、その列はKey列となります。`agg_type` が指定されている場合、その列はValue列となります。詳細は [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) を参照してください。上記の例では、`site_id` と `city_code` をソート列として指定しているため、`user_id` と `pv` 列にはそれぞれ `agg_type` を指定する必要があります。

### 更新モデル

`site_access_unique` という名前の更新モデルテーブルを作成し、`site_id`、`city_code`、`user_id`、`pv` の四つの列を含み、`site_id` と `city_code` がソート列です。

テーブル作成文は以下の通りです：

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

### 主キーモデル

`site_access_primary` という名前の主キーモデルテーブルを作成し、`site_id`、`city_code`、`user_id`、`pv` の四つの列を含み、`site_id` が主キー列、`site_id` と `city_code` がソート列です。

テーブル作成文は以下の通りです：

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

## ソートの効果

上記のテーブル作成文を例に、ソートの効果は以下の三つの状況に分けられます：

- クエリ条件が `site_id` と `city_code` の二つの列のみを含む場合、以下のように、クエリプロセスでスキャンする必要があるデータ行を大幅に減少させることができます：

  ```Plain
  select sum(pv) from site_access_duplicate where site_id = 123 and city_code = 2;
  ```

- クエリ条件が `site_id` の一つの列のみを含む場合、以下のように、`site_id` のみのデータ行を特定することができます：

  ```Plain
  select sum(pv) from site_access_duplicate where site_id = 123;
  ```

- クエリ条件が `city_code` の一つの列のみを含む場合、以下のように、全てのデータ行をスキャンする必要があり、ソートの効果は大きく低下します：

  ```Plain
  select sum(pv) from site_access_duplicate where city_code = 2;
  ```

  > **説明**
  >
  > この場合、ソート列は本来の効果を発揮できません。

最初の状況では、データ行の位置を特定するために、指定された範囲を見つけるために二分探索を行う必要があります。データ行が非常に多い場合、`site_id` と `city_code` の二つの列に直接二分探索を行うと、二つの列のデータを全てメモリにロードする必要があり、これにより大量のメモリスペースを消費します。この時、プレフィックスインデックスを使用してキャッシュするデータ量を減らし、クエリを効果的に加速することができます。

また、実際のビジネスシーンでは、指定されたソート列が非常に多い場合、メモリを大量に使用することになります。このような状況を避けるために、StarRocksはプレフィックスインデックスに以下の制限を設けています：

- プレフィックスインデックス項目の内容は、データブロックの最初の行のソート列のプレフィックスのみで構成される必要があります。

- プレフィックスインデックス列の数は3を超えてはいけません。

- プレフィックスインデックス項目の長さは36バイトを超えてはいけません。

- プレフィックスインデックスにはFLOATまたはDOUBLE型の列を含めることはできません。

- プレフィックスインデックスにVARCHAR型の列は一度だけ出現し、末尾に位置する必要があります。

- プレフィックスインデックスの末尾列がCHARまたはVARCHAR型の場合、プレフィックスインデックス項目の長さは36バイトを超えません。

## ソート列の選択

ここでは `site_access_duplicate` テーブルを例に、ソート列の選択方法を紹介します。

- よくクエリ条件として使用される列は、ソート列として選択することをお勧めします。

- ソートキーが複数の列に関係する場合、区別度が高く、よくクエリされる列を前に置くことをお勧めします。

  区別度が高い列とは、値の数が多く、持続的に増加する列を指します。例えば、上記の `site_access_duplicate` テーブルでは、都市の数は固定されているため、`city_code` 列の値の数は固定されていますが、`site_id` 列の値の数は `city_code` 列よりもはるかに多く、さらに増加し続けるため、`site_id` 列の区別度は `city_code` 列よりもはるかに高いです。

- ソートキーには多くの列を含めるべきではありません。多くのソート列を選択することはクエリ性能の向上には役立たず、ソートのオーバーヘッドを増加させ、データのインポートコストを増加させます。

以上を踏まえて、`site_access_duplicate` テーブルのソート列を選択する際には、以下の三点に注意する必要があります：

- `site_id` 列と `city_code` 列の組み合わせでよくクエリする場合、`site_id` 列をソートキーの最初の列として選択することをお勧めします。

- `city_code` 列でよくクエリし、時々 `site_id` 列と `city_code` 列の組み合わせでクエリする場合、`city_code` 列をソートキーの最初の列として選択することをお勧めします。

- 極端な状況で、`site_id` 列と `city_code` 列の組み合わせでのクエリの割合が `city_code` 列の単独クエリの割合と大差ない場合、`city_code` 列を最初の列とするRollupテーブルを作成することができます。Rollupテーブルでは `city_code` 列に別のソートインデックス（Sort Index）が作成されます。
