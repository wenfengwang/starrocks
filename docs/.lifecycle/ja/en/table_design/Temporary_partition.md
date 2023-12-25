---
displayed_sidebar: English
---

# 一時的なパーティション

このトピックでは、一時的なパーティション機能の使用方法について説明します。

既にパーティションルールが定義されているパーティションテーブルに一時的なパーティションを作成し、これらの一時的なパーティションに新しいデータ分散戦略を定義することができます。一時的なパーティションは、パーティション内のデータをアトミックに上書きする際や、パーティション分割やバケッティング戦略を調整する際に、一時的なデータキャリアとして機能します。一時的なパーティションでは、パーティション範囲、バケット数、レプリカ数、ストレージ媒体などのデータ分散戦略やプロパティをリセットして、特定の要件に合わせることができます。

一時的なパーティション機能は、以下のシナリオで使用できます：

- アトミック上書き操作
  
  パーティション内のデータを書き換える際に、書き換えプロセス中もデータがクエリ可能であることを保証する必要がある場合、まず元の正式なパーティションに基づいて一時的なパーティションを作成し、新しいデータを一時的なパーティションにロードします。その後、replace操作を使用して、元の正式なパーティションを一時的なパーティションとアトミックに置き換えます。パーティションされていないテーブルのアトミック上書き操作については、[ALTER TABLE - SWAP](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md#swap)を参照してください。

- パーティションデータクエリの並行性を調整する

  パーティションのバケット数を変更する必要がある場合、まず元の正式なパーティションと同じパーティション範囲を持つ一時的なパーティションを作成し、新しいバケット数を指定します。次に、`INSERT INTO`コマンドを使用して、元の正式なパーティションのデータを一時的なパーティションにロードします。最後に、replace操作を使用して、元の正式なパーティションを一時的なパーティションとアトミックに置き換えます。

- パーティションルールを変更する
  
  パーティション戦略を変更したい場合、例えばパーティションのマージや大きなパーティションを複数の小さなパーティションに分割するなど、まず期待されるマージまたは分割範囲の一時的なパーティションを作成します。次に、`INSERT INTO`コマンドを使用して、元の正式なパーティションのデータを一時的なパーティションにロードします。最後に、replace操作を使用して、元の正式なパーティションを一時的なパーティションとアトミックに置き換えます。

## 一時的なパーティションを作成する

[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)コマンドを使用して、一度に1つ以上のパーティションを作成できます。

### 構文

#### 単一の一時的なパーティションを作成する

```SQL
ALTER TABLE <table_name>
ADD TEMPORARY PARTITION <temporary_partition_name> VALUES [("value1"), {MAXVALUE|("value2")})]
[(partition_desc)]
[DISTRIBUTED BY HASH(<bucket_key>)];
ALTER TABLE <table_name> 
ADD TEMPORARY PARTITION <temporary_partition_name> VALUES LESS THAN {MAXVALUE|(<"value">)}
[(partition_desc)]
[DISTRIBUTED BY HASH(<bucket_key>)];
```

#### 一度に複数のパーティションを作成する

```SQL
ALTER TABLE <table_name>
ADD TEMPORARY PARTITIONS START ("value1") END ("value2") EVERY {(INTERVAL <num> <time_unit>)|<num>}
[(partition_desc)]
[DISTRIBUTED BY HASH(<bucket_key>)];
```

### パラメータ

`partition_desc`：一時的なパーティションのバケット数やプロパティ（レプリカ数やストレージ媒体など）を指定します。

### 例

`site_access`テーブルに`tp1`という一時的なパーティションを作成し、`VALUES [(...), (...)]`構文を使用してその範囲を`[2020-01-01, 2020-02-01)`と指定します。

```SQL
ALTER TABLE site_access
ADD TEMPORARY PARTITION tp1 VALUES [("2020-01-01"), ("2020-02-01"));
```

`site_access`テーブルに`tp2`という一時的なパーティションを作成し、`VALUES LESS THAN (...)`構文を使用してその上限を`2020-03-01`と指定します。StarRocksは、前の一時的なパーティションの上限をこの一時的なパーティションの下限として使用し、左閉じ右開の範囲`[2020-02-01, 2020-03-01)`の一時的なパーティションを生成します。

```SQL
ALTER TABLE site_access
ADD TEMPORARY PARTITION tp2 VALUES LESS THAN ("2020-03-01");
```

`site_access`テーブルに`tp3`という一時的なパーティションを作成し、`VALUES LESS THAN (...)`構文を使用してその上限を`2020-04-01`と指定し、レプリカ数を`1`として指定します。

```SQL
ALTER TABLE site_access
ADD TEMPORARY PARTITION tp3 VALUES LESS THAN ("2020-04-01")
 ("replication_num" = "1")
DISTRIBUTED BY HASH (site_id);
```

`START (...) END (...) EVERY (...)`構文を使用して`site_access`テーブルに一度に複数のパーティションを作成し、これらのパーティションの範囲を`[2020-04-01, 2021-01-01)`と指定し、月単位のパーティション粒度を設定します。

```SQL
ALTER TABLE site_access 
ADD TEMPORARY PARTITIONS START ("2020-04-01") END ("2021-01-01") EVERY (INTERVAL 1 MONTH);
```

### 使用上の注意点

- 一時的なパーティションのパーティション列は、そのパーティションを作成する元の正式なパーティションのパーティション列と同じでなければならず、変更することはできません。
- 一時的なパーティションの名前は、正式なパーティションや他の一時的なパーティションの名前と同じであってはなりません。
- テーブル内のすべての一時的なパーティションの範囲は重複してはいけませんが、一時的なパーティションと正式なパーティションの範囲は重複しても構いません。

## 一時的なパーティションを表示する

一時的なパーティションを表示するには、[SHOW TEMPORARY PARTITIONS](../sql-reference/sql-statements/data-manipulation/SHOW_PARTITIONS.md)コマンドを使用します。

```SQL
SHOW TEMPORARY PARTITIONS FROM [db_name.]table_name [WHERE] [ORDER BY] [LIMIT]
```

## 一時的なパーティションにデータをロードする

`INSERT INTO`コマンド、STREAM LOAD、またはBROKER LOADを使用して、一時的なパーティションにデータをロードできます。

### `INSERT INTO`コマンドを使用してデータをロードする

例：

```SQL
INSERT INTO site_access TEMPORARY PARTITION (tp1) VALUES ("2020-01-01",1,"ca","lily",4);
INSERT INTO site_access TEMPORARY PARTITION (tp2) SELECT * FROM site_access_copy PARTITION p2;
INSERT INTO site_access TEMPORARY PARTITION (tp3, tp4,...) SELECT * FROM site_access_copy PARTITION (p3, p4,...);
```

構文とパラメータの詳細については、[INSERT INTO](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。

### STREAM LOADを使用してデータをロードする

例：

```bash
curl --location-trusted -u root: -H "label:123" -H "Expect:100-continue" -H "temporary_partitions: tp1, tp2, ..." -T testData \
    http://host:port/api/example_db/site_access/_stream_load    
```

構文とパラメータの詳細な説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

### BROKER LOADを使用してデータをロードする

例：

```SQL
LOAD LABEL example_db.label1
(
    DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starrocks/data/input/file")
    INTO TABLE my_table
    TEMPORARY PARTITION (tp1, tp2, ...)
    ...
)
WITH BROKER
(
    StorageCredentialParams
);
```

`StorageCredentialParams`は、選択した認証方法に応じて異なる認証パラメータのグループを表します。構文とパラメータの詳細な説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### ROUTINE LOADを使用してデータをロードする

例：

```SQL

```
CREATE ROUTINE LOAD example_db.site_access ON example_tbl
COLUMNS(col, col2,...),
TEMPORARY PARTITIONS(tp1, tp2, ...)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest"
);
```

詳細な構文とパラメーターの説明については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

## 一時パーティション内のデータクエリ

[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) ステートメントを使用して、指定された一時パーティション内のデータをクエリできます。

```SQL
SELECT * FROM
site_access TEMPORARY PARTITION (tp1);

SELECT * FROM
site_access TEMPORARY PARTITION (tp1, tp2, ...);

SELECT event_day,site_id,pv FROM
site_access TEMPORARY PARTITION (tp1, tp2, ...);
```

JOIN 句を使用して、2つのテーブルから一時パーティション内のデータをクエリすることができます。

```SQL
SELECT * FROM
site_access TEMPORARY PARTITION (tp1, tp2, ...)
JOIN
site_access_copy TEMPORARY PARTITION (tp1, tp2, ...)
ON site_access.site_id=site_access1.site_id AND site_access.event_day=site_access1.event_day;
```

## 元の正式パーティションを一時パーティションで置き換える

[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) ステートメントを使用して、元の正式パーティションを一時パーティションで置き換え、新しい正式パーティションを作成できます。

> **注記**
>
> ALTER TABLE ステートメントで操作した元の正式パーティションと一時パーティションは削除され、復元することはできません。

### 構文

```SQL
ALTER TABLE table_name REPLACE PARTITION (partition_name) WITH TEMPORARY PARTITION (temporary_partition_name1, ...)
PROPERTIES ("key" = "value");
```

### パラメーター

- **strict_range**
  
  デフォルト値: `true`。

  このパラメーターが `true` に設定されている場合、元の正式パーティションの範囲の和集合は、置換に使用される一時パーティションの範囲の和集合と完全に一致している必要があります。このパラメーターが `false` に設定されている場合、置換後に新しい正式パーティションの範囲が他の正式パーティションと重複しないことを確認するだけで済みます。

  - 例 1:
  
    次の例では、元の正式パーティション `p1`、`p2`、`p3` の範囲の和集合は、一時パーティション `tp1` と `tp2` の範囲の和集合と同じです。したがって、`tp1` と `tp2` を使用して `p1`、`p2`、`p3` を置き換えることができます。

      ```plaintext
      # 元の正式パーティション p1、p2、p3 の範囲 => これらの範囲の和集合
      [10, 20), [20, 30), [40, 50) => [10, 30), [40, 50)
      
      # 一時パーティション tp1 と tp2 の範囲 => これらの範囲の和集合
      [10, 30), [40, 45), [45, 50) => [10, 30), [40, 50)
      ```

  - 例 2:

    次の例では、元の正式パーティションの範囲の和集合が、一時パーティションの範囲の和集合と異なります。`strict_range` の値が `true` に設定されている場合、一時パーティション `tp1` と `tp2` は元の正式パーティション `p1` を置き換えることはできません。値が `false` に設定されていて、一時パーティションの範囲 [10, 30) と [40, 50) が他の正式パーティションと重複しない場合、一時パーティションは元の正式パーティションを置き換えることができます。

      ```plaintext
      # 元の正式パーティション p1 の範囲 => 範囲の和集合
      [10, 50) => [10, 50)
      
      # 一時パーティション tp1 と tp2 の範囲 => これらの範囲の和集合
      [10, 30), [40, 50) => [10, 30), [40, 50)
      ```

- **use_temp_partition_name**
  
  デフォルト値: `false`。

  元の正式パーティションの数が置換に使用される一時パーティションの数と同じである場合、このパラメーターが `false` に設定されていると、置換後の新しい正式パーティションの名前は変更されません。このパラメーターが `true` に設定されている場合、一時パーティションの名前が置換後の新しい正式パーティションの名前として使用されます。

  次の例では、このパラメーターが `false` に設定されている場合、新しい正式パーティションのパーティション名は置換後も `p1` のままです。しかし、関連するデータとプロパティは一時パーティション `tp1` のものに置き換えられます。このパラメーターが `true` に設定されている場合、新しい正式パーティションのパーティション名は置換後 `tp1` に変更されます。元の正式パーティション `p1` はもう存在しません。

    ```sql
    ALTER TABLE tbl1 REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
    ```

  置き換える正式パーティションの数が置換に使用される一時パーティションの数と異なり、このパラメーターがデフォルト値 `false` のままの場合、このパラメーターの値 `false` は無効です。

  次の例では、置換後、新しい正式パーティションの名前は `tp1` に変更され、元の正式パーティション `p1` と `p2` はもう存在しません。

    ```SQL
    ALTER TABLE site_access REPLACE PARTITION (p1, p2) WITH TEMPORARY PARTITION (tp1);
    ```

### 例

元の正式パーティション `p1` を一時パーティション `tp1` で置き換えます。

```SQL
ALTER TABLE site_access REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
```

元の正式パーティション `p2` と `p3` を一時パーティション `tp2` と `tp3` で置き換えます。

```SQL
ALTER TABLE site_access REPLACE PARTITION (p2, p3) WITH TEMPORARY PARTITION (tp2, tp3);
```

元の正式パーティション `p4` と `p5` を一時パーティション `tp4` と `tp5` で置き換え、パラメーター `strict_range` を `false` に、`use_temp_partition_name` を `true` に指定します。

```SQL
ALTER TABLE site_access REPLACE PARTITION (p4, p5) WITH TEMPORARY PARTITION (tp4, tp5)
PROPERTIES (
    "strict_range" = "false",
    "use_temp_partition_name" = "true"
);
```

### 使用上の注意点

- テーブルに一時パーティションが存在する場合、`ALTER` コマンドを使用してテーブルにスキーマ変更操作を行うことはできません。
- テーブルにスキーマ変更操作を行う場合、テーブルに一時パーティションを追加することはできません。

## 一時パーティションの削除

以下のコマンドを使用して一時パーティション `tp1` を削除します。

```SQL
ALTER TABLE site_access DROP TEMPORARY PARTITION tp1;
```

以下の制限に注意してください：

- `DROP` コマンドを使用してデータベースやテーブルを直接削除した場合、`RECOVER` コマンドを使用して限られた期間内にデータベースやテーブルを復元できます。しかし、一時パーティションは復元できません。
- `ALTER` コマンドを使用して正式パーティションを削除した後、`RECOVER` コマンドを使用して限られた期間内にそれを復元することができます。一時パーティションは正式パーティションと関連付けられていないため、一時パーティションの操作は正式パーティションに影響しません。
- `ALTER` コマンドを使用して一時パーティションを削除した後、`RECOVER` コマンドを使用してそれを復元することはできません。
- `TRUNCATE` コマンドを使用してテーブル内のデータを削除すると、テーブルの一時パーティションも削除され、復元することはできません。
- `TRUNCATE` コマンドを使用して正式なパーティション内のデータを削除する場合、一時パーティションには影響しません。
- 一時パーティション内のデータを削除するために `TRUNCATE` コマンドは使用できません。
