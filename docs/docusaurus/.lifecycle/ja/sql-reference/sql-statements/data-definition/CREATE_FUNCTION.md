---
displayed_sidebar: "Japanese"
---

# 関数の作成

## 説明

ユーザー定義関数（UDF）を作成します。現在、Java UDF、つまりスカラー関数、ユーザー定義集約関数（UDAF）、ユーザー定義ウィンドウ関数（UDWF）、ユーザー定義テーブル関数（UDTF）のみを作成できます。

**Java UDFのコンパイル方法、作成方法、使用方法の詳細については、[Java UDF](../../sql-functions/JAVA_UDF.md)を参照してください。**

> **注**
>
> グローバルUDFを作成するには、SYSTEMレベルのCREATE GLOBAL FUNCTION権限が必要です。データベース全体のUDFを作成するには、DATABASEレベルのCREATE FUNCTION権限が必要です。

## 構文

```sql
CREATE [GLOBAL][AGGREGATE | TABLE] FUNCTION 関数名
(arg_type [, ...])
RETURNS return_type
PROPERTIES ("key" = "value" [, ...])
```

## パラメータ

| **Parameter**      | **Required** | **Description**                                                     |
| ------------- | -------- | ------------------------------------------------------------ |
| GLOBAL        | No       | v3.0からサポートされたグローバルUDFを作成するかどうか。  |
| AGGREGATE     | No       | UDAFまたはUDWFを作成するかどうか。        |
| TABLE         | No       | UDTFを作成するかどうか。`AGGREGATE`と`TABLE`の両方が指定されていない場合、スカラー関数が作成されます。               |
| function_name | Yes       | 作成する関数の名前。このパラメータにデータベースの名前を含めることができます。たとえば、`db1.my_func`のようにです。 `function_name`にデータベース名が含まれている場合、そのデータベースにUDFが作成されます。 それ以外の場合、現在のデータベースにUDFが作成されます。新しい関数とそのパラメータの名前は、宛先データベースの既存の名前と同じにすることはできません。それ以外の場合、関数を作成できません。関数名が同じでもパラメータが異なる場合は作成が成功します。 |
| arg_type      | Yes       | 関数の引数の型。追加された引数は`, ...`で表されます。サポートされているデータ型については、[Java UDF](../../sql-functions/JAVA_UDF.md#mapping-between-sql-data-types-and-java-data-types)を参照してください。|
| return_type      | Yes       | 関数の戻り値の型。サポートされているデータ型については、[Java UDF](../../sql-functions/JAVA_UDF.md#mapping-between-sql-data-types-and-java-data-types)を参照してください。 |
| PROPERTIES    | Yes       | UDFの種類に応じて異なる関数のプロパティ。詳細については、[Java UDF](../../sql-functions/JAVA_UDF.md#step-6-create-the-udf-in-starrocks)を参照してください。 |