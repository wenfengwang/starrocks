---
displayed_sidebar: Chinese
---

# CAST

## 機能

`input` を指定されたデータ型に変換します。例えば `cast (input as BIGINT)` は、現在の `input` をBIGINT型に変換します。

バージョン2.4からは、Array文字列とJSON arrayをARRAY型に変換することがサポートされています。

## 文法

```Haskell
cast(input as type)
```

## パラメータ説明

`input`: 変換されるデータ

`type`: 目的のデータ型

## 戻り値の説明

戻り値のデータ型は `type` で指定された型です。変換に失敗した場合はNULLを返します。

## 例

例1：一般的な変換。

```Plain Text
    select cast (1 as BIGINT);
    +-------------------+
    | CAST(1 AS BIGINT) |
    +-------------------+
    |                 1 |
    +-------------------+

    select cast('9.5' as DECIMAL(10,2));
    +--------------------------------+
    | CAST('9.5' AS DECIMAL(10,2)) |
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
```

例2：ARRAY型への変換。

```Plain Text
    -- 文字列をARRAY<ANY>に変換。

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
    +------------------------------------+
    | CAST('["a", "b", "c"]' AS ARRAY<STRING>) |
    +------------------------------------+
    | ["a","b","c"]                      |
    +------------------------------------+

    -- JSON配列をARRAY<ANY>に変換。

    select cast(parse_json('[{"a":1}, {"a": 2}]')  as array<json>);
    +------------------------------------------------+
    | CAST(parse_json('[{"a":1}, {"a": 2}]') AS ARRAY<JSON>) |
    +------------------------------------------------+
    | ['{"a": 1}','{"a": 2}']                        |
    +------------------------------------------------+

    select cast(parse_json('[1, 2, 3]')  as array<int>);
    +-----------------------------------------+
    | CAST(parse_json('[1, 2, 3]') AS ARRAY<INT>) |
    +-----------------------------------------+
    | [1,2,3]                                 |
    +-----------------------------------------+

    select cast(parse_json('["1","2","3"]') as array<string>);
    +--------------------------------------------------+
    | CAST(parse_json('["1","2","3"]') AS ARRAY<STRING>) |
    +--------------------------------------------------+
    | ["1","2","3"]                                    |
    +--------------------------------------------------+
```

例3：データのインポート時の変換。

```bash
curl --location-trusted -u <username>:<password> -T ~/user_data/bigint \

    -H "columns: tmp_k1, k1=cast(tmp_k1 as BIGINT)" \

    http://host:port/api/test/bigint/_stream_load
```

> **説明**
>
> インポートプロセス中に、元のデータが浮動小数点数のSTRING型で変換される場合、データはNULLに変換されます。例えば、浮動小数点数12.0はNULLになります。このようなデータをBIGINTに強制的に変換したい場合は、まずSTRING型の浮動小数点数をDOUBLEに変換し、その後BIGINTに変換する必要があります。以下の例を参照してください:

```bash
curl --location-trusted -u <username>:<password> -T ~/user_data/bigint \

    -H "columns: tmp_k1, k1=cast(cast(tmp_k1 as DOUBLE) as BIGINT)" \

    http://host:port/api/test/bigint/_stream_load
```

```plain text
select cast(cast ("11.2" as double) as bigint);
+----------------------------------------+
| CAST(CAST('11.2' AS DOUBLE) AS BIGINT) |
+----------------------------------------+
|                                     11 |
+----------------------------------------+
1 row in set (0.00 sec)
```
