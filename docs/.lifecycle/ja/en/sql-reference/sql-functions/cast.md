---
displayed_sidebar: English
---

# CAST

## 説明

入力を指定された型に変換します。たとえば、`cast(input as BIGINT)`は入力をBIGINT値に変換します。

v2.4から、StarRocksはARRAY型への変換をサポートしています。

## 構文

```Haskell
cast(input as type)
```

## パラメーター

`input`: 変換したいデータ。
`type`: 目的のデータ型。

## 戻り値

`type`と同じデータ型の値を返します。

## 例

例1: 一般的なデータ変換

```Plain Text
    select cast('9.5' as DECIMAL(10,2));
    +--------------------------------+
    | CAST('9.5' AS DECIMAL(10,2))   |
    +--------------------------------+
    |                           9.50 |
    +--------------------------------+
    
    select cast(NULL AS INT);
    +-------------------+
    | CAST(NULL AS INT) |
    +-------------------+
    |              NULL |
    +-------------------+
    
    select cast(true AS BOOLEAN);
    +-----------------------+
    | CAST(TRUE AS BOOLEAN) |
    +-----------------------+
    |                     1 |
    +-----------------------+
    
    select cast(1 as BIGINT);
    +-------------------+
    | CAST(1 AS BIGINT) |
    +-------------------+
    |                 1 |
    +-------------------+
```

例2: 入力をARRAYに変換します。

```Plain Text
    -- 文字列をARRAY<ANY>に変換します。

    select cast('[1,2,3]' as array<int>);
    +-------------------------------+
    | CAST('[1,2,3]' AS ARRAY<INT>) |
    +-------------------------------+
    | [1,2,3]                       |
    +-------------------------------+

    select cast('[1,2,3]' as array<bigint>);
    +----------------------------------+
    | CAST('[1,2,3]' AS ARRAY<BIGINT>) |
    +----------------------------------+
    | [1,2,3]                          |
    +----------------------------------+

    select cast('[1,2,3]' as array<string>);
    +----------------------------------+
    | CAST('[1,2,3]' AS ARRAY<STRING>) |
    +----------------------------------+
    | ["1","2","3"]                    |
    +----------------------------------+

    select cast('["a", "b", "c"]' as array<string>);
    +----------------------------------+
    | CAST('["a", "b", "c"]' AS ARRAY<STRING>) |
    +----------------------------------+
    | ["a","b","c"]                    |
    +----------------------------------+

    -- JSON配列をARRAY<ANY>に変換します。

    select cast(parse_json('[{"a":1}, {"a": 2}]') as array<json>);
    +--------------------------------------------------+
    | CAST(parse_json('[{"a":1}, {"a": 2}]') AS ARRAY<JSON>) |
    +--------------------------------------------------+
    | ['{"a": 1}','{"a": 2}']                          |
    +--------------------------------------------------+
    
    select cast(parse_json('[1, 2, 3]') as array<int>);
    +-----------------------------------+
    | CAST(parse_json('[1, 2, 3]') AS ARRAY<INT>) |
    +-----------------------------------+
    | [1,2,3]                           |
    +-----------------------------------+
    
    select cast(parse_json('["1","2","3"]') as array<string>);
    +-----------------------------------+
    | CAST(parse_json('["1","2","3"]') AS ARRAY<STRING>) |
    +-----------------------------------+
    | ["1","2","3"]                     |
    +-----------------------------------+
```

例3: ロード中にデータを変換します。

```bash
    curl --location-trusted -u <username>:<password> -T ~/user_data/bigint \
        -H "columns: tmp_k1, k1=cast(tmp_k1 as BIGINT)" \
        http://host:port/api/test/bigint/_stream_load
```

> **注記**
>
> 元の値が浮動小数点値（例えば12.0）の場合、NULLに変換されます。この型をBIGINTに強制的に変換したい場合は、以下の例を参照してください：

```bash
    curl --location-trusted -u <username>:<password> -T ~/user_data/bigint \
        -H "columns: tmp_k1, k1=cast(cast(tmp_k1 as DOUBLE) as BIGINT)" \
        http://host:port/api/test/bigint/_stream_load
```

```plain text
    MySQL > select cast(cast("11.2" as double) as bigint);
    +----------------------------------------+
    | CAST(CAST('11.2' AS DOUBLE) AS BIGINT) |
    +----------------------------------------+
    |                                     11 |
    +----------------------------------------+
```
