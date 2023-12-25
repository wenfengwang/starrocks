---
displayed_sidebar: Chinese
---

# ALTER TABLE

## 機能

このステートメントは、既存のテーブルを変更するために使用されます。変更内容には以下が含まれます：

- [テーブル名、パーティション名、インデックス名の変更](#rename-名称の変更)
- [テーブルのコメントを変更する](#コメントを変更する-バージョン31から)
- [パーティションの変更（追加・削除とパーティション属性の変更）](#partition-操作の構文)
- [バケット方式とバケット数の変更](#バケット方式とバケット数の変更-バージョン32から)
- [列の変更（追加・削除と列の順序変更）](#列の変更-追加・削除と列の順序変更)
- [Rollup Indexの作成または削除](#rollup-index-操作の構文)
- [bitmap indexの変更](#bitmap-indexの変更)
- [テーブルの属性を変更する](#テーブルの属性を変更する)
- [テーブルをアトミックに置き換える](#swap-二つのテーブルをアトミックに置き換える)
- [手動でcompactionを実行してテーブルデータをマージする](#手動-compaction-バージョン31から)

> **注意**
>
> この操作には、対応するテーブルのALTER権限が必要です。

## 構文

ALTER TABLEの構文は以下の通りです:

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
alter_clause1[, alter_clause2, ...]
```

ここで **alter_clause** は rename、comment、partition、bucket、column、rollup index、bitmap index、table property、swap、compaction に関連する変更操作に分かれます:

- rename: テーブル名、rollup index名、partition名の変更、**列名の変更はサポートされていません**。
- comment: 既存テーブルのコメントを変更します。**バージョン3.1からサポートされています。**
- partition: パーティション属性の変更、パーティションの削除、パーティションの追加。
- bucket: バケット方式とバケット数の変更。
- column: 列の追加、列の削除、列の順序変更、列のタイプ変更。
- Rollup Index: Rollup Indexの作成または削除。
- bitmap index: bitmap indexの変更。
- swap: 二つのテーブルをアトミックに置き換える。
- compaction: 指定されたテーブルまたはパーティションで手動でCompaction（データバージョンのマージ）を実行します。**バージョン3.1からサポートされています。**

:::note

- partition、column、rollup index <!--compactionを含むかどうか、bucketとcolumn/rollup indexを一緒にできるか--> これらの操作は同じ `ALTER TABLE` ステートメントに同時に現れることはできません。
- bucket、column、rollup index <!--compactionとfast schema evolutionを含むかどうか--> は非同期操作で、コマンドが成功したらすぐに成功メッセージが返されます。操作の進行状況を確認するには [SHOW ALTER TABLE](../data-manipulation/SHOW_ALTER.md) ステートメントを使用できます。進行中の操作をキャンセルする場合は、[CANCEL ALTER TABLE](../data-manipulation/SHOW_ALTER.md) を使用できます。
- rename、comment、partition、bitmap index、swap は同期操作で、コマンドの戻り値は実行が完了したことを示します。
:::

### 名称の変更

#### テーブル名の変更

構文：

```sql
ALTER TABLE <tbl_name> RENAME <new_tbl_name>;
```

#### ROLLUP INDEX 名の変更 (RENAME ROLLUP)

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
RENAME ROLLUP old_rollup_name new_rollup_name;
```

#### partition 名の変更 (RENAME PARTITION)

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
RENAME PARTITION <old_partition_name> <new_partition_name>;
```

### コメントを変更する（バージョン3.1から）

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name> COMMENT = "<new table comment>";
```

### partition 操作の構文

#### パーティションの追加 (ADD PARTITION)

構文：

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD PARTITION [IF NOT EXISTS] <partition_name>
partition_desc ["key"="value"]
[DISTRIBUTED BY HASH (k1[,k2 ...]) [BUCKETS num]];
```

注意：

1. partition_desc は以下の二つの形式をサポートしています:

   - `VALUES LESS THAN [MAXVALUE|("value1", ...)]`
   - `VALUES [("value1", ...), ("value1", ...)]`

2. パーティションは左閉じ右開きの範囲で、ユーザーが右境界のみを指定した場合、システムは左境界を自動的に決定します。
3. バケット方式が指定されていない場合は、テーブル作成時に使用されたバケット方式が自動的に使用されます。
4. バケット方式が指定されている場合は、バケット数のみを変更でき、バケット方式やバケット列を変更することはできません。
5. `["key" = "value"]` の部分では、パーティションのいくつかの属性を設定できます。詳細は [CREATE TABLE](./CREATE_TABLE.md) を参照してください。

#### パーティションの削除 (DROP PARTITION)

構文：

```sql
-- バージョン2.0以前
ALTER TABLE [<db_name>.]<tbl_name>
DROP PARTITION [IF EXISTS | FORCE] <partition_name>;
-- バージョン2.0以降
ALTER TABLE [<db_name>.]<tbl_name>
DROP PARTITION [IF EXISTS] <partition_name> [FORCE];
```

注意：

1. パーティション方式を使用するテーブルは少なくとも一つのパーティションを保持する必要があります。
2. DROP PARTITIONを実行した後、一定期間内にRECOVERステートメントを使用して削除されたパーティションを復元できます。詳細は [RECOVER](../data-definition/RECOVER.md) ステートメントを参照してください。
3. DROP PARTITION FORCEを実行すると、システムはそのパーティションに未完了のトランザクションがあるかどうかをチェックせず、パーティションは直接削除され、復元することはできません。通常はこの操作を実行することはお勧めしません。

#### 一時パーティションの追加 (ADD TEMPORARY PARTITION)

詳細な使用情報については、[一時パーティション](../../../table_design/Temporary_partition.md)を参照してください。

構文：

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
ADD TEMPORARY PARTITION [IF NOT EXISTS] <partition_name>
partition_desc ["key"="value"]
[DISTRIBUTED BY HASH (k1[,k2 ...]) [BUCKETS num]];
```

#### 一時パーティションを使用して元のパーティションを置き換える

構文：

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
REPLACE PARTITION <partition_name> 
partition_desc ["key"="value"]
WITH TEMPORARY PARTITION
partition_desc ["key"="value"]
[PROPERTIES ("key"="value", ...)]
```

#### 一時パーティションの削除 (DROP TEMPORARY PARTITION)

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP TEMPORARY PARTITION <partition_name>;
```

#### パーティション属性の変更 (MODIFY PARTITION)

**構文：**

```sql
ALTER TABLE [<db_name>.]<tbl_name>
    MODIFY PARTITION { <partition_name> | partition_name_list | (*) }
        SET ("key" = "value", ...);
```

**使用説明：**

- 現在、以下のパーティション属性を変更することがサポートされています：
  - storage_medium
  - storage_cooldown_ttl または storage_cooldown_time
  - replication_num

- 単一パーティションテーブルの場合、パーティション名はテーブル名と同じです。複数パーティションテーブルの場合、すべてのパーティションの属性を変更する必要がある場合は `(*)` を使用すると便利です。
- `SHOW PARTITIONS FROM <tbl_name>` を実行して、変更後のパーティション属性を確認します。

### バケット方式とバケット数の変更（バージョン3.2から）

構文：

```SQL
ALTER TABLE [<db_name>.]<table_name>
[ partition_names ]
[ distribution_desc ]

partition_names ::= 
    (PARTITION | PARTITIONS) ( <partition_name> [, <partition_name> ...] )

distribution_desc ::=
    DISTRIBUTED BY RANDOM [ BUCKETS <num> ] |
    DISTRIBUTED BY HASH ( <column_name> [, <column_name> ...] ) [ BUCKETS <num> ]
```

例：

元のテーブルが詳細テーブルで、バケット方式はHashバケット、バケット数は自動設定です。

```SQL
CREATE TABLE IF NOT EXISTS details (
    event_time DATETIME NOT NULL COMMENT "イベントの日時",
    event_type INT NOT NULL COMMENT "イベントのタイプ",
    user_id INT COMMENT "ユーザーID",
    device_code INT COMMENT "デバイスコード",
    channel INT COMMENT ""
)
DUPLICATE KEY(event_time, event_type)
PARTITION BY date_trunc('day', event_time)
DISTRIBUTED BY HASH(user_id);

-- 複数日のデータを挿入
INSERT INTO details (event_time, event_type, user_id, device_code, channel) VALUES
-- 11月26日のデータ
('2023-11-26 08:00:00', 1, 101, 12345, 2),
('2023-11-26 09:15:00', 2, 102, 54321, 3),
('2023-11-26 10:30:00', 1, 103, 98765, 1),
-- 11月27日のデータ
('2023-11-27 08:30:00', 1, 104, 11111, 2),
('2023-11-27 09:45:00', 2, 105, 22222, 3),
('2023-11-27 11:00:00', 1, 106, 33333, 1),
-- 11月28日のデータ
('2023-11-28 08:00:00', 1, 107, 44444, 2),
('2023-11-28 09:15:00', 2, 108, 55555, 3),
('2023-11-28 10:30:00', 1, 109, 66666, 1);
```

#### バケット方式のみを変更する

> **注意**
>
> - バケット方式の変更は、テーブルのすべてのパーティションに対して有効であり、特定のパーティションにのみ有効にすることはできません。
> - バケット方式のみを変更しても、バケット数を変更していない場合でも、ステートメントにバケット数 `BUCKETS <num>` を明記する必要があります。指定しない場合は、StarRocksが自動的にバケット数を設定します。

- 元のHashバケット方式をRandomバケット方式に変更し、バケット数はStarRocksが自動的に設定します。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY RANDOM;
  ```

- Hashバケット時に使用されるバケットキーを元の `event_time, event_type` から `user_id, event_time` に変更します。バケット数はStarRocksが自動的に設定します。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY HASH(user_id, event_time);
  ```

#### バケット数のみを変更する

> **注意**
>
> バケット数のみを変更しても、バケット方式を変更していない場合でも、ステートメントにバケット方式を明記する必要があります。例えば、上記の例では `HASH(user_id)` です。

- すべてのパーティションのバケット数を元のStarRocksが自動的に設定したものから10に変更します。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY HASH(user_id) BUCKETS 10;
  ```

- 指定されたパーティションのバケット数を元のStarRocksが自動的に設定したものから15に変更します。

  ```SQL
  ALTER TABLE details PARTITIONS (p20231127, p20231128) DISTRIBUTED BY HASH(user_id) BUCKETS 15;
  ```

  > **説明**
  >
  > パーティション名は `SHOW PARTITIONS FROM <table_name>;` を実行して確認できます。

#### バケット方式とバケット数を同時に変更する

> **注意**
>
> バケット方式とバケット数を同時に変更する場合、テーブルのすべてのパーティションに対して有効であり、特定のパーティションにのみ有効にすることはできません。


- 分桶方式をHash分散からRandom分散に変更し、分桶数量をStarRocksが自動設定するのではなく10に変更しました。

   ```SQL
   ALTER TABLE details DISTRIBUTED BY RANDOM BUCKETS 10;
   ```

- Hash分散時に使用する分桶キーを`event_time, event_type`から`user_id, event_time`に変更し、分桶数量をStarRocksが自動設定するのではなく10に変更しました。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY HASH(user_id, event_time) BUCKETS 10;
  ```

### 列の変更（列の追加・削除および列順の変更）

以下のindexは物化インデックスです。テーブル作成成功後、テーブルはbaseテーブル（base index）となり、baseテーブルに基づいて[rollup indexを作成](#rollup-indexの作成-add-rollup)できます。

base indexとrollup indexはどちらも物化インデックスです。以下の文を書く際に`rollup_index_name`を指定しない場合、デフォルトで基本テーブルを操作します。

#### 指定されたindexの特定の位置に列を追加する (ADD COLUMN)

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD COLUMN column_name column_type [KEY | agg_type] [DEFAULT "default_value"]
[AFTER column_name|FIRST]
[TO rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

使用説明：

- 値列を追加する場合、agg_typeを指定する必要があります。
- 非集約モデル（DUPLICATE KEYなど）でキー列を追加する場合、KEYキーワードを指定する必要があります。
- rollup indexにbase indexに既に存在する列を追加することはできません。必要に応じて、新しいrollup indexを作成できます。

#### 指定されたindexに複数の列を追加する

構文：

- 複数の列を追加する：

  ```sql
  ALTER TABLE [<db_name>.]<tbl_name>
  ADD COLUMN (column_name1 column_type [KEY | agg_type] DEFAULT "default_value", ...)
  [TO rollup_index_name]
  [PROPERTIES ("key"="value", ...)]
  ```

- 複数の列を追加しつつ、`AFTER`を使って列の追加位置を指定する：

  ```sql
  ALTER TABLE [<db_name>.]<tbl_name>
  ADD COLUMN column_name1 column_type [KEY | agg_type] DEFAULT "default_value" AFTER column_name,
  ADD COLUMN column_name2 column_type [KEY | agg_type] DEFAULT "default_value" AFTER column_name
  [, ...]
  [TO rollup_index_name]
  [PROPERTIES ("key"="value", ...)]
  ```

注意：

1. 値列を追加する場合、agg_typeを指定する必要があります。
2. 非集約モデルでキー列を追加する場合、KEYキーワードを指定する必要があります。
3. rollup indexにbase indexに既に存在する列を追加することはできません。必要に応じて、新しいrollup indexを作成できます。

#### 生成列の追加

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD COLUMN col_name data_type [NULL] AS generation_expr [COMMENT 'string']
```

生成列を追加し、使用する式を指定します。[生成列](../generated_columns.md)は、複雑な式を含むクエリの速度を上げるために、式の結果を事前に計算して保存するために使用されます。v3.1から、StarRocksはこの機能をサポートしています。

#### 指定されたindexから列を削除する (DROP COLUMN)

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP COLUMN column_name
[FROM rollup_index_name];
```

注意：

1. パーティション列は削除できません。
2. base indexから列を削除する場合、rollup indexにその列が含まれている場合も削除されます。

#### 指定されたindexの列の型および列の位置を変更する (MODIFY COLUMN)

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
MODIFY COLUMN column_name column_type [KEY | agg_type] [NULL | NOT NULL] [DEFAULT "default_value"]
[AFTER column_name|FIRST]
[FROM rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

注意：

1. 値列を変更する場合、agg_typeを指定する必要があります。
2. 非集約型でキー列を変更する場合、KEYキーワードを指定する必要があります。
3. 列の型のみを変更でき、他の属性はそのまま維持されます（つまり、他の属性は文中で元の属性に従って明示的に記述する必要があります。[column](#column)セクションの8番目の例を参照してください）。
4. パーティション列は変更できません。
5. 現在、以下のタイプの変換がサポートされています（精度損失はユーザーが保証する必要があります）：

    - TINYINT/SMALLINT/INT/BIGINTをTINYINT/SMALLINT/INT/BIGINT/DOUBLEに変換。
    - TINYINT/SMALLINT/INT/BIGINT/LARGEINT/FLOAT/DOUBLE/DECIMALをVARCHARに変換。
    - VARCHARは最大長を変更できます。
    - VARCHARをTINYINT/SMALLINT/INT/BIGINT/LARGEINT/FLOAT/DOUBLEに変換。
    - VARCHARをDATEに変換（現在、"%Y-%m-%d"、"%y-%m-%d"、"%Y%m%d"、"%y%m%d"、"%Y/%m/%d"、"%y/%m/%d"の6つのフォーマットをサポート）
    DATETIMEをDATEに変換（年月日の情報のみを保持、例：`2019-12-09 21:47:05` <--> `2019-12-09`）
    DATEをDATETIMEに変換（時分秒は自動的にゼロで埋められます、例：`2019-12-09` <--> `2019-12-09 00:00:00`）
    - FLOATをDOUBLEに変換。
    - INTをDATEに変換（INT型のデータが無効な場合は変換に失敗し、元のデータは変わりません）。

6. NULLからNOT NULLへの変換はサポートされていません。

#### 指定されたindexの列を再配置する

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ORDER BY (column_name1, column_name2, ...)
[FROM rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

注意：

- indexに含まれるすべての列を記述する必要があります。
- value列はkey列の後に配置されます。

#### 主キーテーブルのソートキーを構成する列を変更する
<!--サポートされるバージョン-->

構文：

```SQL
ALTER TABLE [<db_name>.]<table_name>
[ order_desc ]

order_desc ::=
    ORDER BY <column_name> [, <column_name> ...]
```

例：

元のテーブルが主キーテーブルで、ソートキーが主キー`dt,order_id`と結合されているとします。

```SQL
create table orders (
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
) PRIMARY KEY (dt, order_id)
PARTITION BY date_trunc('day', dt)
DISTRIBUTED BY HASH(order_id);
```

ソートキーと主キーの結合を解除し、ソートキーを`dt,revenue,state`に変更します。

```SQL
ALTER TABLE orders ORDER BY (dt,revenue,state);
```

### rollup indexの操作構文

#### rollup indexの作成 (ADD ROLLUP)

**RollUpテーブルインデックス**: shortkey indexはデータ検索を加速できますが、shortkey indexは次元列の順序に依存しています。前置きのない次元列を使用して検索述語を構築する場合、ユーザーは複数のRollUpテーブルインデックスを作成できます。RollUpテーブルインデックスのデータの組織と保存はテーブルと同じですが、RollUpテーブルには独自のshortkey indexがあります。RollUpテーブルインデックスを作成する際、ユーザーは集約の粒度、列の数、次元列の順序を選択できます。これにより、頻繁に使用されるクエリ条件が対応するRollUpテーブルインデックスにヒットするようになります。

構文：

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD ROLLUP rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)];
```

PROPERTIES: タイムアウト時間を設定することができ、デフォルトのタイムアウト時間は1日です。

例：

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD ROLLUP r1(col1,col2) from r0;
```

#### 複数のrollup indexを一括で作成する

構文：

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
ADD ROLLUP [rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)],...];
```

例：

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
ADD ROLLUP r1(col1,col2) from r0, r2(col3,col4) from r0;
```

> 注意
>
1. from_index_nameが指定されていない場合、デフォルトでbase indexから作成されます。
2. rollupテーブルの列はfrom_indexに既に存在する列でなければなりません。
3. propertiesで、ストレージ形式を指定できます。詳細は[CREATE TABLE](../data-definition/CREATE_TABLE.md)のセクションを参照してください。

#### rollup indexの削除 (DROP ROLLUP)

構文：

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
DROP ROLLUP rollup_name [PROPERTIES ("key"="value", ...)];
```

例：

```sql
ALTER TABLE [<db_name>.]<tbl_name> DROP ROLLUP r1;
```

#### 複数のrollup indexを一括で削除する

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP ROLLUP [rollup_name [PROPERTIES ("key"="value", ...)],...];
```

例：

```sql
ALTER TABLE [<db_name>.]<tbl_name> DROP ROLLUP r1, r2;
```

注意：

base indexは削除できません。

### bitmap indexの変更

#### bitmap indexの作成 (ADD INDEX)

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD INDEX index_name (column [, ...],) [USING BITMAP] [COMMENT 'balabala'];
```

注意：

1. 現在、bitmap indexの変更のみがサポートされています。
2. bitmap indexは単一列にのみ作成できます。

#### bitmap indexの削除 (DROP INDEX)

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP INDEX index_name;
```

### テーブルの属性を変更する

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
SET ("key" = "value",...)
```

パラメータ説明：

- `key`はテーブル属性の名前を表し、`value`はそのテーブル属性の設定を表します。

- 以下のテーブル属性を変更できます：
  - `replication_num`
  - `default.replication_num`
  - `storage_cooldown_ttl`
  - `storage_cooldown_time`
  - [動的パーティション関連の属性 properties](../../../table_design/dynamic_partitioning.md)、例えば`dynamic_partition.enable`
  - `enable_persistent_index`
  - `bloom_filter_columns`
  - `colocate_with`
  - `bucket_size`（バージョン3.2からサポート）

注意：テーブルの属性の変更は、schema change操作に統合して変更することもできます。詳細は[例](#例)セクションを参照してください。

### 二つのテーブルをアトミックに置換する (SWAP)

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
SWAP WITH <tbl_name>;
```

### 手動 Compaction（3.1 版本起）

StarRocks は Compaction 機構を通じて、異なるデータバージョンを統合し、小さいファイルを大きいファイルにマージすることで、クエリ性能を効果的に向上させます。

3.1 版本以前は、Compaction を行うための二つの方法がサポートされていました：

- システムが自動的にバックグラウンドで Compaction を実行します。Compaction の粒度は BE レベルで、バックグラウンドで自動的に実行され、ユーザーは特定のデータベースやテーブルを制御できません。
- ユーザーが HTTP インターフェースを通じて Tablet を指定して Compaction を実行します。

3.1 版本以降は、SQL インターフェースが追加され、ユーザーは SQL コマンドを実行して手動で Compaction を行うことができ、テーブルや単一または複数のパーティションを指定して Compaction を行うことができます。

構文：

```sql
-- 全テーブルに対して compaction を実行。
ALTER TABLE <tbl_name> COMPACT

-- 特定のパーティションに対して compaction を実行。
ALTER TABLE <tbl_name> COMPACT <partition_name>

-- 複数のパーティションに対して compaction を実行。
ALTER TABLE <tbl_name> COMPACT (<partition1_name>[,<partition2_name>,...])

-- 複数のパーティションに対して cumulative compaction を実行。
ALTER TABLE <tbl_name> CUMULATIVE COMPACT (<partition1_name>[,<partition2_name>,...])

-- 複数のパーティションに対して base compaction を実行。
ALTER TABLE <tbl_name> BASE COMPACT (<partition1_name>[,<partition2_name>,...])
```

Compaction 実行後は、`information_schema` データベースの `be_compactions` テーブルを照会して、Compaction 後のデータバージョンの変更を確認できます（`SELECT * FROM information_schema.be_compactions;`）。

## 例

### テーブル

1. テーブルのデフォルトレプリカ数を変更し、新規パーティションのレプリカ数はこの値をデフォルトで使用します。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("default.replication_num" = "2");
    ```

2. 単一パーティションテーブルの実際のレプリカ数を変更します（単一パーティションテーブルのみ）。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("replication_num" = "3");
    ```

3. 複数レプリカ間のデータの書き込みと同期方法を変更します。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("replicated_storage" = "false");
    ```

    上記の例では、複数レプリカの書き込みと同期方法を leaderless replication に設定しています。つまり、データは複数のレプリカに同時に書き込まれ、プライマリとセカンダリの区別はありません。詳細は [CREATE TABLE](CREATE_TABLE.md) の `replicated_storage` パラメータの説明を参照してください。

### パーティション

1. パーティションを追加します。既存のパーティションは [MIN, 2013-01-01) で、新しいパーティション [2013-01-01, 2014-01-01) をデフォルトのバケット方式で追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES LESS THAN ("2014-01-01");
    ```

2. 新しいバケット数を使用してパーティションを追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES LESS THAN ("2015-01-01")
    DISTRIBUTED BY HASH(k1) BUCKETS 20;
    ```

3. 新しいレプリカ数を使用してパーティションを追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES LESS THAN ("2015-01-01")
    ("replication_num"="1");
    ```

4. パーティションのレプリカ数を変更します。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY PARTITION p1 SET("replication_num"="1");
    ```

5. 指定されたパーティションを一括で変更します。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY PARTITION (p1, p2, p4) SET("replication_num"="1");
    ```

6. すべてのパーティションを一括で変更します。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY PARTITION (*) SET("storage_medium"="HDD");
    ```

7. パーティションを削除します。

    ```sql
    ALTER TABLE example_db.my_table
    DROP PARTITION p1;
    ```

8. 上下限を指定したパーティションを追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES [("2014-01-01"), ("2014-02-01"));
    ```

### Rollup インデックス

1. `example_rollup_index` という Rollup インデックスを作成し、ベースインデックス（k1, k2, k3, v1, v2）に基づいて列式ストレージを使用します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD ROLLUP example_rollup_index(k1, k3, v1, v2)
    PROPERTIES("storage_type"="column");
    ```

2. `example_rollup_index`（k1, k3, v1, v2）に基づいて `example_rollup_index2` という Rollup インデックスを作成します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD ROLLUP example_rollup_index2 (k1, v1)
    FROM example_rollup_index;
    ```

3. ベースインデックス（k1, k2, k3, v1）に基づいて `example_rollup_index3` という Rollup インデックスを作成し、カスタムの Rollup タイムアウト時間を1時間に設定します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD ROLLUP example_rollup_index3(k1, k3, v1)
    PROPERTIES("storage_type"="column", "timeout" = "3600");
    ```

4. `example_rollup_index2` という Rollup インデックスを削除します。

    ```sql
    ALTER TABLE example_db.my_table
    DROP ROLLUP example_rollup_index2;
    ```

### カラム

1. `example_rollup_index` の `col1` の後にキーカラム `new_col` を追加します（非集約モデル）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT KEY DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

2. `example_rollup_index` の `col1` の後にバリューカラム `new_col` を追加します（非集約モデル）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

3. `example_rollup_index` の `col1` の後にキーカラム `new_col` を追加します（集約モデル）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

4. `example_rollup_index` の `col1` の後に SUM 集約タイプのバリューカラム `new_col` を追加します（集約モデル）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT SUM DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

5. `example_rollup_index` に複数のカラムを追加します（集約モデル）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN (col1 INT DEFAULT "1", col2 FLOAT SUM DEFAULT "2.3")
    TO example_rollup_index;
    ```

6. `example_rollup_index` に複数のカラムを追加し、`AFTER` を使用してカラムの追加位置を指定します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN col1 INT DEFAULT "1" AFTER `k1`,
    ADD COLUMN col2 FLOAT SUM AFTER `v2`,
    TO example_rollup_index;
    ```

7. `example_rollup_index` からカラムを削除します。

    ```sql
    ALTER TABLE example_db.my_table
    DROP COLUMN col2
    FROM example_rollup_index;
    ```

8. ベースインデックスの `col1` カラムのタイプを BIGINT に変更し、`col2` の後ろに移動します。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY COLUMN col1 BIGINT DEFAULT "1" AFTER col2;
    ```

9. ベースインデックスの `val1` カラムの最大長を変更します。元の `val1` は (`val1 VARCHAR(32) REPLACE DEFAULT "abc"`) です。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY COLUMN val1 VARCHAR(64) REPLACE DEFAULT "abc";
    ```

10. `example_rollup_index` のカラムの順序を再配置します（元の順序は k1, k2, k3, v1, v2）。

    ```sql
    ALTER TABLE example_db.my_table
    ORDER BY (k3,k1,k2,v2,v1)
    FROM example_rollup_index;
    ```

11. 二つの操作を同時に実行します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN v2 INT MAX DEFAULT "0" AFTER k2 TO example_rollup_index,
    ORDER BY (k3,k1,k2,v2,v1) FROM example_rollup_index;
    ```

12. テーブルのブルームフィルターカラムを変更します。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("bloom_filter_columns"="k1,k2,k3");
    ```

    上記のカラム変更操作に統合することもできます（複数の子句の構文にはわずかな違いがあります）

    ```sql
    ALTER TABLE example_db.my_table
    DROP COLUMN col2
    PROPERTIES ("bloom_filter_columns"="k1,k2,k3");
    ```

### テーブルプロパティ

1. テーブルの Colocate 属性を変更します。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("colocate_with" = "t1");
    ```

2. テーブルのダイナミックパーティション属性を変更します（ダイナミックパーティション属性が未追加のテーブルに属性を追加することをサポート）。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("dynamic_partition.enable" = "false");
    ```

    ダイナミックパーティション属性が未追加のテーブルに属性を追加する場合は、すべてのダイナミックパーティション属性を指定する必要があります。

    ```sql
    ALTER TABLE example_db.my_table
    SET (
        "dynamic_partition.enable" = "true",
        "dynamic_partition.time_unit" = "DAY",
        "dynamic_partition.end" = "3",
        "dynamic_partition.prefix" = "p",
        "dynamic_partition.buckets" = "32"
        );
    ```

### 名前の変更

1. テーブル `table1` の名前を `table2` に変更します。

    ```sql
    ALTER TABLE table1 RENAME TO table2;
    ```

2. テーブル `example_table` の Rollup インデックス `rollup1` の名前を `rollup2` に変更します。

    ```sql
    ALTER TABLE example_table RENAME ROLLUP rollup1 TO rollup2;
    ```

3. テーブル `example_table` のパーティション `p1` の名前を `p2` に変更します。

    ```sql
    ALTER TABLE example_table RENAME PARTITION p1 TO p2;
    ```

### ビットマップインデックス

1. `table1` の `siteid` にビットマップインデックスを作成します。

    ```sql
    ALTER TABLE table1
    ADD INDEX index_name (siteid) USING BITMAP COMMENT 'balabala';
    ```

2. `table1` の `siteid` カラムのビットマップインデックスを削除します。

    ```sql
    ALTER TABLE table1 DROP INDEX index_name;
    ```

### スワップ

`table1` と `table2` をアトミックに交換します。

```sql
ALTER TABLE table1 SWAP WITH table2;
```

### 手動 Compaction

```sql
CREATE TABLE compaction_test( 
    event_day DATE,
    pv BIGINT)
DUPLICATE KEY(event_day)
PARTITION BY RANGE(event_day)
(
    PARTITION p1 VALUES LESS THAN ("2023-03-01"),
    PARTITION p2 VALUES LESS THAN ("2033-04-01")
)
DISTRIBUTED BY HASH(event_day) BUCKETS 8
PROPERTIES("replication_num" = "3");

INSERT INTO compaction_test VALUES
('2023-02-14', 2),
('2033-03-01',2);
{'label':'insert_734648fa-c878-11ed-90d6-00163e0dcbfc', 'status':'VISIBLE', 'txnId':'5008'}

INSERT INTO compaction_test VALUES
('2023-02-14', 2),('2033-03-01',2);
{'label':'insert_85c95c1b-c878-11ed-90d6-00163e0dcbfc', 'status':'VISIBLE', 'txnId':'5009'}

ALTER TABLE compaction_test COMPACT;

ALTER TABLE compaction_test COMPACT p203303;

ALTER TABLE compaction_test COMPACT (p202302,p203303);

ALTER TABLE compaction_test CUMULATIVE COMPACT (p202302,p203303);

ALTER TABLE compaction_test BASE COMPACT (p202302,p203303);
```

## 関連参照

- [CREATE TABLE](./CREATE_TABLE.md)
- [SHOW CREATE TABLE](../data-manipulation/SHOW_CREATE_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [SHOW ALTER TABLE](../data-manipulation/SHOW_ALTER.md)
- [DROP TABLE](./DROP_TABLE.md)
