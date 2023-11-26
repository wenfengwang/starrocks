---
displayed_sidebar: "Japanese"
---

# 一時パーティション

このトピックでは、一時パーティション機能の使用方法について説明します。

定義済みのパーティショニングルールを持つパーティションテーブルに一時パーティションを作成し、これらの一時パーティションに対して新しいデータ分散戦略を定義することができます。一時パーティションは、パーティション内のデータを原子的に上書きする場合や、パーティショニングおよびバケット化戦略を調整する場合に一時的なデータキャリアとして機能することができます。一時パーティションでは、パーティション範囲、バケット数、レプリカ数、およびストレージメディアなどのデータ分散戦略をリセットして、特定の要件を満たすことができます。

以下のシナリオで一時パーティション機能を使用することができます。

- 原子的な上書き操作

  パーティション内のデータを書き換える必要があり、書き換えプロセス中にデータをクエリできるようにする場合、元の正式なパーティションを基に一時パーティションを作成し、新しいデータを一時パーティションにロードすることができます。その後、置換操作を使用して、元の正式なパーティションを一時パーティションで原子的に置き換えることができます。非パーティションテーブルの原子的な上書き操作については、[ALTER TABLE - SWAP](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md#swap)を参照してください。

- パーティションデータのクエリ同時性の調整

  パーティションのバケット数を変更する必要がある場合、元の正式なパーティションと同じパーティション範囲を持つ一時パーティションを作成し、新しいバケット数を指定することができます。その後、`INSERT INTO`コマンドを使用して、元の正式なパーティションのデータを一時パーティションにロードします。最後に、置換操作を使用して、元の正式なパーティションを一時パーティションで原子的に置き換えることができます。

- パーティショニングルールの変更

  パーティショニング戦略を変更したい場合、パーティションのマージや大きなパーティションを複数の小さなパーティションに分割するなどの変更を行うことができます。まず、期待されるマージまたは分割範囲を持つ一時パーティションを作成します。その後、`INSERT INTO`コマンドを使用して、元の正式なパーティションのデータを一時パーティションにロードします。最後に、置換操作を使用して、元の正式なパーティションを一時パーティションで原子的に置き換えることができます。

## 一時パーティションの作成

[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)コマンドを使用して、一度に1つ以上のパーティションを作成することができます。

### 構文

#### 単一の一時パーティションの作成

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

#### 一度に複数のパーティションを作成

```SQL
ALTER TABLE <table_name>
ADD TEMPORARY PARTITIONS START ("value1") END ("value2") EVERY {(INTERVAL <num> <time_unit>)|<num>}
[(partition_desc)]
[DISTRIBUTED BY HASH(<bucket_key>)];
```

### パラメータ

`partition_desc`: 一時パーティションのバケット数やレプリカ数、ストレージメディアなどのプロパティを指定します。

### 例

テーブル`site_access`に一時パーティション`tp1`を作成し、その範囲を`[2020-01-01, 2020-02-01)`として`VALUES [(...), (...)]`構文を使用して指定します。

```SQL
ALTER TABLE site_access
ADD TEMPORARY PARTITION tp1 VALUES [("2020-01-01"), ("2020-02-01"));
```

テーブル`site_access`に一時パーティション`tp2`を作成し、その上限を`2020-03-01`として`VALUES LESS THAN (...)`構文を使用します。StarRocksは、前の一時パーティションの上限をこの一時パーティションの下限として使用し、左閉区間と右開区間の範囲`[2020-02-01, 2020-03-01)`を生成します。

```SQL
ALTER TABLE site_access
ADD TEMPORARY PARTITION tp2 VALUES LESS THAN ("2020-03-01");
```

テーブル`site_access`に一時パーティション`tp3`を作成し、その上限を`2020-04-01`として`VALUES LESS THAN (...)`構文を使用し、レプリカ数を`1`として指定します。

```SQL
ALTER TABLE site_access
ADD TEMPORARY PARTITION tp3 VALUES LESS THAN ("2020-04-01")
 ("replication_num" = "1")
DISTRIBUTED BY HASH (site_id);
```

テーブル`site_access`に`START (...) END (...) EVERY (...)`構文を使用して一度に複数のパーティションを作成し、これらのパーティションの範囲を`[2020-04-01, 2021-01-01)`として、月次のパーティション粒度を指定します。

```SQL
ALTER TABLE site_access 
ADD TEMPORARY PARTITIONS START ("2020-04-01") END ("2021-01-01") EVERY (INTERVAL 1 MONTH);
```

### 使用上の注意

- 一時パーティションのパーティション列は、一時パーティションを作成するための元の正式なパーティションのパーティション列と同じでなければなりません。
- 一時パーティションの名前は、正式なパーティションや他の一時パーティションの名前と同じにすることはできません。
- テーブルのすべての一時パーティションの範囲は重複してはなりませんが、一時パーティションと正式なパーティションの範囲は重複することがあります。

## 一時パーティションの表示

[SHOW TEMPORARY PARTITIONS](../sql-reference/sql-statements/data-manipulation/SHOW_PARTITIONS.md)コマンドを使用して、一時パーティションを表示することができます。

```SQL
SHOW TEMPORARY PARTITIONS FROM [db_name.]table_name [WHERE] [ORDER BY] [LIMIT]
```

## 一時パーティションにデータをロード

`INSERT INTO`コマンド、STREAM LOAD、またはBROKER LOADを使用して、1つ以上の一時パーティションにデータをロードすることができます。

### `INSERT INTO`コマンドを使用してデータをロード

例：

```SQL
INSERT INTO site_access TEMPORARY PARTITION (tp1) VALUES ("2020-01-01",1,"ca","lily",4);
INSERT INTO site_access TEMPORARY PARTITION (tp2) SELECT * FROM site_access_copy PARTITION p2;
INSERT INTO site_access TEMPORARY PARTITION (tp3, tp4,...) SELECT * FROM site_access_copy PARTITION (p3, p4,...);
```

詳細な構文とパラメータの説明については、[INSERT INTO](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。

### STREAM LOADを使用してデータをロード

例：

```bash
curl --location-trusted -u root: -H "label:123" -H "Expect:100-continue" -H "temporary_partitions: tp1, tp2, ..." -T testData \
    http://host:port/api/example_db/site_access/_stream_load    
```

詳細な構文とパラメータの説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

### BROKER LOADを使用してデータをロード

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

ここで、`StorageCredentialParams`は、選択した認証方法によって異なる認証パラメータのグループを表します。詳細な構文とパラメータの説明については、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### ROUTINE LOADを使用してデータをロード

例：

```SQL
CREATE ROUTINE LOAD example_db.site_access ON example_tbl
COLUMNS(col, col2,...),
TEMPORARY PARTITIONS(tp1, tp2, ...)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest"
);
```

詳細な構文とパラメータの説明については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

## 一時パーティション内のデータのクエリ

[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)ステートメントを使用して、指定した一時パーティション内のデータをクエリすることができます。

```SQL
SELECT * FROM
site_access TEMPORARY PARTITION (tp1);

SELECT * FROM
site_access TEMPORARY PARTITION (tp1, tp2, ...);

SELECT event_day,site_id,pv FROM
site_access TEMPORARY PARTITION (tp1, tp2, ...);
```

JOIN句を使用して、2つのテーブルから一時パーティション内のデータをクエリすることもできます。

```SQL
SELECT * FROM
site_access TEMPORARY PARTITION (tp1, tp2, ...)
JOIN
site_access_copy TEMPORARY PARTITION (tp1, tp2, ...)
ON site_access.site_id=site_access1.site_id and site_access.event_day=site_access1.event_day;
```

## 元の正式なパーティションを一時パーティションで置き換える

[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)ステートメントを使用して、元の正式なパーティションを一時パーティションで置き換えて新しい正式なパーティションを作成することができます。

> **注意**
>
> ALTER TABLEステートメントで操作した元の正式なパーティションと一時パーティションは削除され、復元することはできません。

### 構文

```SQL
ALTER TABLE table_name REPLACE PARTITION (partition_name) WITH TEMPORARY PARTITION (temporary_partition_name1, ...)
PROPERTIES ("key" = "value");
```

### パラメータ

- **strict_range**
  
  デフォルト値: `true`。

  このパラメータの値が`true`に設定されている場合、すべての元の正式なパーティションの範囲の和集合は、置換に使用される一時パーティションの範囲の和集合と完全に一致する必要があります。このパラメータの値が`false`に設定されている場合、新しい正式なパーティションの範囲が、置換後に他の正式なパーティションと重複しないことを確認するだけで十分です。

  - 例1：
  
    次の例では、元の正式なパーティション`p1`、`p2`、および`p3`の範囲の和集合が、置換に使用される一時パーティション`tp1`と`tp2`の範囲の和集合と同じであり、`tp1`と`tp2`を`p1`、`p2`、および`p3`の置換に使用することができます。

      ```plaintext
      # 元の正式なパーティションp1、p2、およびp3の範囲 => これらの範囲の和集合
      [10, 20), [20, 30), [40, 50) => [10, 30), [40, 50)
      
      # 一時パーティションtp1とtp2の範囲 => これらの範囲の和集合
      [10, 30), [40, 45), [45, 50) => [10, 30), [40, 50)
      ```

  - 例2：

    次の例では、元の正式なパーティションの範囲の和集合が一時パーティションの範囲の和集合と異なるため、パラメータ`strict_range`の値が`true`に設定されている場合、一時パーティション`tp1`は元の正式なパーティション`p1`を置き換えることはできません。値が`false`に設定されている場合、一時パーティションの範囲[10, 30)と[40, 50)が他の正式なパーティションと重複しない場合、一時パーティションは元の正式なパーティションを置き換えることができます。

      ```plaintext
      # 元の正式なパーティションp1の範囲 => 範囲の和集合
      [10, 50) => [10, 50)
      
      # 一時パーティションtp1とtp2の範囲 => これらの範囲の和集合
      [10, 30), [40, 50) => [10, 30), [40, 50)
      ```

- **use_temp_partition_name**
  
  デフォルト値: `false`。

  置換に使用する元の正式なパーティションの数と置換に使用する一時パーティションの数が同じ場合、このパラメータを`false`に設定すると、置換後の新しい正式なパーティションの名前は変更されず、一時パーティションの名前が新しい正式なパーティションの名前として使用されます。

  次の例では、パラメータを`false`に設定した場合、置換後の新しい正式なパーティションのパーティション名は`p1`のままですが、関連するデータとプロパティは一時パーティション`tp1`のデータとプロパティで置き換えられます。パラメータを`true`に設定した場合、置換後の新しい正式なパーティションのパーティション名は`tp1`に変更されます。元の正式なパーティション`p1`は存在しなくなります。

    ```sql
    ALTER TABLE tbl1 REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
    ```

  置換する正式なパーティションの数が置換に使用する一時パーティションの数と異なる場合、およびこのパラメータがデフォルト値`false`のままの場合、このパラメータの値`false`は無効です。

  次の例では、置換後の新しい正式なパーティションの名前が`tp1`に変更され、元の正式なパーティション`p1`および`p2`は存在しなくなります。

    ```SQL
    ALTER TABLE site_access REPLACE PARTITION (p1, p2) WITH TEMPORARY PARTITION (tp1);
    ```

### 例

元の正式なパーティション`p1`を一時パーティション`tp1`で置き換えます。

```SQL
ALTER TABLE site_access REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
```

元の正式なパーティション`p2`および`p3`を一時パーティション`tp2`および`tp3`で置き換えます。

```SQL
ALTER TABLE site_access REPLACE PARTITION (p2, p3) WITH TEMPORARY PARTITION (tp2, tp3);
```

元の正式なパーティション`p4`および`p5`を一時パーティション`tp4`および`tp5`で置き換え、パラメータ`strict_range`を`false`、`use_temp_partition_name`を`true`として指定します。

```SQL
ALTER TABLE site_access REPLACE PARTITION (p4, p5) WITH TEMPORARY PARTITION (tp4, tp5)
PROPERTIES (
    "strict_range" = "false",
    "use_temp_partition_name" = "true"
);
```

### 使用上の注意

- テーブルに一時パーティションがある場合、`ALTER`コマンドを使用してテーブルに対してスキーマ変更操作を行うことはできません。
- テーブルに対してスキーマ変更操作を行う場合、一時パーティションをテーブルに追加することはできません。

## 一時パーティションの削除

次のコマンドを使用して一時パーティション`tp1`を削除します。

```SQL
ALTER TABLE site_access DROP TEMPORARY PARTITION tp1;
```

次の制限に注意してください。

- `DROP`コマンドを使用してデータベースまたはテーブルを直接削除した場合、`RECOVER`コマンドを使用して一定期間内にデータベースまたはテーブルを復元することができます。ただし、一時パーティションは復元できません。
- `ALTER`コマンドを使用して正式なパーティションを削除した場合、`RECOVER`コマンドを使用して一定期間内に復元することができます。一時パーティションは正式なパーティションと結び付けられていないため、一時パーティションの操作は正式なパーティションに影響を与えません。
- `ALTER`コマンドを使用して一時パーティションを削除した場合、`RECOVER`コマンドを使用して復元することはできません。
- `TRUNCATE`コマンドを使用してテーブルのデータを削除する場合、テーブルの一時パーティションが削除され、復元することはできません。
- `TRUNCATE`コマンドを使用して正式なパーティションのデータを削除する場合、一時パーティションは影響を受けません。
- `TRUNCATE`コマンドは、一時パーティションのデータを削除するために使用することはできません。
