---
displayed_sidebar: Chinese
---

# 一時的なパーティション

この記事では、一時的なパーティション機能の使用方法について説明します。

既にパーティションルールが定義されたパーティションテーブルに一時的なパーティションを作成し、それらの一時的なパーティションに独自のデータ分布戦略を設定することができます。アトミックなオーバーライト操作やパーティションバケット戦略の調整時に、一時的なパーティションを一時的なデータキャリアとして使用することができます。一時的なパーティションに設定できるデータ分布戦略には、パーティション範囲、バケット数、およびレプリカ数、ストレージメディアなどの一部の属性が含まれます。

以下の使用シナリオで、一時的なパーティション機能を使用できます：

- アトミックなオーバーライト操作

  ある正式なパーティションのデータを書き換える必要があり、書き換えプロセス中にデータを閲覧できるようにする場合、対応する一時的なパーティションを先に作成し、新しいデータを一時的なパーティションにインポートした後、置換操作を通じて、アトミックに元の正式なパーティションを置き換え、新しい正式なパーティションを生成できます。パーティションなしテーブルのアトミックなオーバーライト操作については、[ALTER TABLE - SWAP](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md#swap)を参照してください。
- パーティションデータのクエリ並行性の調整

  ある正式なパーティションのバケット数を変更する必要がある場合、対応するパーティション範囲の一時的なパーティションを先に作成し、新しいバケット数を指定した後、`INSERT INTO` コマンドを使用して元の正式なパーティションのデータを一時的なパーティションにインポートし、置換操作を通じて、アトミックに元の正式なパーティションを置き換え、新しい正式なパーティションを生成できます。
- パーティション戦略の変更

  正式なパーティションのパーティション範囲を変更したい場合、例えば複数の小さなパーティションを一つの大きなパーティションに統合する、または一つの大きなパーティションを複数の小さなパーティションに分割する場合、統合または分割後の範囲に対応する一時的なパーティションを先に作成し、`INSERT INTO` コマンドを使用して元の正式なパーティションのデータを一時的なパーティションにインポートし、置換操作を通じて、アトミックに元の正式なパーティションを置き換え、新しい正式なパーティションを生成できます。

## 一時的なパーティションの作成

ALTER TABLE コマンドを使用して一時的なパーティションを作成することも、一時的なパーティションを一括で作成することもできます。

### 構文

**一時的なパーティションを作成する**

```SQL
ALTER TABLE <table_name> 
ADD TEMPORARY PARTITION <temporary_partition_name> VALUES [("value1"), {MAXVALUE|("value2")})]
[(partition_desc)]
[DISTRIBUTED BY HASH(<bucket_key>)];
```

```SQL
ALTER TABLE <table_name> 
ADD TEMPORARY PARTITION <temporary_partition_name> VALUES LESS THAN {MAXVALUE|(<"value">)}
[(partition_desc)]
[DISTRIBUTED BY HASH(<bucket_key>)];
```

**一時的なパーティションを一括で作成する**

```SQL
ALTER TABLE <table_name>
ADD TEMPORARY PARTITIONS START ("value1") END ("value2") EVERY {(INTERVAL <num> <time_unit>)|<num>}
[(partition_desc)]
[DISTRIBUTED BY HASH(<bucket_key>)];
```

### **パラメータ説明**

`partition_desc`：一時的なパーティションにバケット数と一部の属性を指定します。これにはレプリカ数、ストレージメディアなどの情報が含まれます。

### 例

`site_access` テーブルに一時的なパーティション `tp1` を作成し、`VALUES [(...),(...))` 構文を使用してその一時的なパーティションの範囲を [2020-01-01, 2020-02-01) と指定します。

```SQL
ALTER TABLE site_access
ADD TEMPORARY PARTITION tp1 VALUES [("2020-01-01"), ("2020-02-01"));
```

`site_access` テーブルに一時的なパーティション `tp2` を作成し、`VALUES LESS THAN (...)` 構文を使用してその一時的なパーティションの上限を `2020-03-01` と指定します。StarRocks は前の一時的なパーティションの上限をこの一時的なパーティションの下限として使用し、範囲 [2020-02-01, 2020-03-01) の左閉じ右開きの一時的なパーティションを生成します。

```SQL
ALTER TABLE site_access
ADD TEMPORARY PARTITION tp2 VALUES LESS THAN ("2020-03-01");
```

`site_access` テーブルに一時的なパーティション `tp3` を作成し、`VALUES LESS THAN (...)` 構文を使用してその一時的なパーティションの上限を `2020-04-01` と指定し、一時的なパーティションのレプリカ数を `1`、バケット数を `5` と指定します。

```SQL
ALTER TABLE site_access
ADD TEMPORARY PARTITION tp3 VALUES LESS THAN ("2020-04-01")
 ("replication_num" = "1")
DISTRIBUTED BY HASH (site_id);
```

`site_access` テーブルに一時的なパーティションを一括で作成し、`START (...) END (...) EVERY (...)` 構文を使用して一括で作成する一時的なパーティションの範囲を [2020-04-01, 2021-01-01) と指定し、パーティションの粒度を1ヶ月とします。

```SQL
ALTER TABLE site_access 
ADD TEMPORARY PARTITIONS START ("2020-04-01") END ("2021-01-01") EVERY (INTERVAL 1 MONTH);
```

### 注意事項

- 一時的なパーティションのパーティション列は元の正式なパーティションと同じで、変更することはできません。
- 一時的なパーティションの名前は正式なパーティションや他の一時的なパーティションと重複してはいけません。
- 一つのテーブルのすべての一時的なパーティション間の範囲は重複してはいけませんが、一時的なパーティションの範囲は正式なパーティションの範囲と重複しても構いません。

## 一時的なパーティションの表示

以下の [SHOW TEMPORARY PARTITIONS](../sql-reference/sql-statements/data-manipulation/SHOW_PARTITIONS.md) コマンドを使用して、テーブルの一時的なパーティションを表示できます。

```SQL
SHOW TEMPORARY PARTITIONS FROM site_access;
```

## 一時的なパーティションへのデータのインポート

`INSERT INTO` コマンド、STREAM LOAD、BROKER LOAD、または ROUTINE LOAD を使用してデータを一時的なパーティションにインポートできます。

### `INSERT INTO` コマンドを使用したインポート

以下の [INSERT INTO](../sql-reference/sql-statements/data-manipulation/INSERT.md) コマンドを使用してデータを一時的なパーティションにインポートできます。

```SQL
INSERT INTO site_access TEMPORARY PARTITION (tp1) VALUES ("2020-01-01",1,"ca","lily",4);
INSERT INTO site_access TEMPORARY PARTITION (tp2) SELECT * FROM site_access_copy PARTITION p2;
INSERT INTO site_access TEMPORARY PARTITION (tp3, tp4,...) SELECT * FROM site_access_copy PARTITION (p3, p4,...);
```

### STREAM LOAD を使用したインポート

例：

```Bash
curl --location-trusted -u <username>:<password> -H "label:label1" -H "temporary_partitions: tp1, tp2, ..." -T testData \
    http://host:port/api/example_db/site_access/_stream_load    
```

構文やパラメータなどの詳細については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

### BROKER LOAD を使用したインポート

例：

```SQL
LOAD LABEL example_db.label2
(
    DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starrocks/data/input/file")
    INTO TABLE site_access
    TEMPORARY PARTITION (tp1, tp2, ...)
    ...
)

WITH BROKER
(
    StorageCredentialParams
);
```

詳細な構文やパラメータについては、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### ROUTINE LOADを使用したインポート

例：

```SQL
CREATE ROUTINE LOAD routine_load_job ON site_access
COLUMNS (event_day, site_id, city_code, user_name, pv),
TEMPORARY PARTITION (tp1, tp2, ...)
FROM KAFKA
(
    "kafka_broker_list" = "<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest"
);
```

構文やパラメータに関する詳細は、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

## 一時分区のデータをクエリする

指定した一時分区のデータをクエリすることができます。

```SQL
SELECT * FROM
site_access TEMPORARY PARTITION (tp1);

SELECT * FROM
site_access TEMPORARY PARTITION (tp1, tp2, ...);

SELECT event_day, site_id, pv FROM
site_access TEMPORARY PARTITION (tp1, tp2, ...);
```

二つのテーブルで指定した一時分区のデータを結合クエリすることができます。

```SQL
SELECT * FROM
site_access TEMPORARY PARTITION (tp1, tp2, ...)
JOIN
site_access_copy TEMPORARY PARTITION (tp1, tp2, ...)
ON site_access.site_id = site_access_copy.site_id AND site_access.event_day = site_access_copy.event_day
;
```

## 一時分区を使用した置換

以下のコマンドを使用して、一時分区で既存の正式分区を置換し、新しい正式分区を形成することができます。分区の置換に成功すると、元の正式分区と一時分区は削除され、復元することはできません。

### 構文

```SQL
ALTER TABLE <table_name>
REPLACE PARTITION (<partition_name1>[, ...]) WITH TEMPORARY PARTITION (<temporary_partition_name1>[, ...])
[PROPERTIES ("key" = "value")];
```

### パラメータ説明

- `strict_range`
  デフォルトは `true` です。

  このパラメータが `true` の場合、既存の正式分区と一時分区の範囲の和集合が完全に一致している必要があります。`false` に設定すると、新しい正式分区と他の正式分区の範囲が重複しないことだけを保証すればよいです。

  - 例 1：

    以下の例では範囲の和集合が同じで、`tp1` と `tp2` を `p1`、`p2`、`p3` と置換することができます。

    ```Plaintext
    # 既存の正式分区 p1, p2, p3 の範囲 (=> 和集合)
    [10, 20), [20, 30), [40, 50) => [10, 30), [40, 50)
    
    # 一時分区 tp1, tp2 の範囲 (=> 和集合)
    [10, 30), [40, 45), [45, 50) => [10, 30), [40, 50)
    ```

  - 例 2：

    上記の例では範囲の和集合が異なり、`strict_range` を `true` に設定すると、`tp1` と `tp2` を `p1` と置換することはできません。`false` に設定し、一時分区の範囲 [10, 30), [40, 50) が他の正式分区と重複しない場合は置換できます。

    ```Plaintext
    # 既存の正式分区 p1 の範囲 (=> 和集合)
    [10, 50) => [10, 50)
    
    # 一時分区 tp1, tp2 の範囲 (=> 和集合)
    [10, 30), [40, 50) => [10, 30), [40, 50)
    ```

- `use_temp_partition_name`
  デフォルトは `false` です。

  - 既存の正式分区と一時分区の数が同じ場合、このパラメータがデフォルトの `false` であれば、新しい正式分区の名前は変わりません。`true` に設定すると、新しい正式分区の名前は一時分区の名前に変更されます。
    以下の例では、このパラメータが `false` の場合、新しい正式分区の名前は `p1` のままですが、関連するデータと属性は一時分区 `tp1` のものに置換されます。このパラメータが `true` の場合、新しい正式分区の名前は `tp1` になり、正式分区 `p1` は存在しなくなります。

    ```SQL
    ALTER TABLE site_access REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
    ```

  - 既存の正式分区と一時分区の数が異なり、このパラメータがデフォルトの `false` である場合、既存の正式分区の数と一時分区の数が異なるため、このパラメータは `false` では機能しません。以下の例では、新しい正式分区の名前は `tp1` になり、正式分区 `p1` と `p2` は存在しなくなります。

    ```SQL
    ALTER TABLE site_access REPLACE PARTITION (p1, p2) WITH TEMPORARY PARTITION (tp1);
    ```

### 例

一時分区 `tp1` を使用して既存の正式分区 `p1` を置換します。

```SQL
ALTER TABLE site_access REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
```

一時分区 `tp2` と `tp3` を使用して既存の正式分区 `p2` と `p3` を置換します。

```SQL
ALTER TABLE site_access REPLACE PARTITION (p2, p3) WITH TEMPORARY PARTITION (tp2, tp3);
```

一時分区 `tp4` と `tp5` を使用して既存の正式分区 `p4` と `p5` を置換します。また、`strict_range` を `false` に、`use_temp_partition_name` を `true` に設定します。

```SQL
ALTER TABLE site_access REPLACE PARTITION (p4, p5) WITH TEMPORARY PARTITION (tp4, tp5)
PROPERTIES (
    "strict_range" = "false",
    "use_temp_partition_name" = "true"
);
```

**注意事項**

- テーブルに一時分区が存在する場合、`ALTER` コマンドを使用してテーブルのスキーマ変更を行うことはできません。
- テーブルのスキーマ変更を行う際、一時分区をテーブルに追加することはできません。

## 一時分区の削除

以下のコマンドで一時分区 `tp1` を削除します。

```SQL
ALTER TABLE site_access DROP TEMPORARY PARTITION tp1;
```

注意事項

- `DROP` コマンドを使用してデータベースやテーブルを直接削除した後、限定時間内に [RECOVER](../sql-reference/sql-statements/data-definition/RECOVER.md) コマンドを使用してそのデータベースやテーブルを復元することができますが、一時分区は復元できません。
- `ALTER` コマンドを使用して正式分区を削除した後、限定時間内に [RECOVER](../sql-reference/sql-statements/data-definition/RECOVER.md) コマンドを使用して復元することができます。正式分区と一時分区は関連していないため、正式分区の操作が一時分区に影響を与えることはありません。
- `ALTER` コマンドを使用して一時分区を削除した後、[RECOVER](../sql-reference/sql-statements/data-definition/RECOVER.md) コマンドを使用して復元することはできません。

- `TRUNCATE` コマンドを使用してテーブルをクリアすると、テーブルの一時的なパーティションが削除され、復元することはできません。
- `TRUNCATE` コマンドを使用して正式なパーティションをクリアする場合、一時的なパーティションは影響を受けません。
- 一時的なパーティションをクリアするために `TRUNCATE` コマンドを使用することはできません。
