---
displayed_sidebar: "Japanese"
---

# 関数を作成

## 説明

ユーザー定義関数（UDF）を作成します。現在、Java UDF、スカラー関数、ユーザー定義集約関数（UDAF）、ユーザー定義ウィンドウ関数（UDWF）、ユーザー定義テーブル関数（UDTF）のみ作成できます。

**Java UDFのコンパイル、作成、使用方法の詳細については、[Java UDF](../../sql-functions/JAVA_UDF.md)を参照してください。**

> **注意**
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

| **パラメータ**      | **必須** | **説明**                                                    |
| ------------- | -------- | ------------------------------------------------------- |
| GLOBAL        | いいえ       | グローバルUDFを作成するかどうか。v3.0からサポートされています。  |
| AGGREGATE     | いいえ       | UDAFまたはUDWFを作成するかどうか。  |
| TABLE         | いいえ       | UDTFを作成するかどうか。`AGGREGATE`と`TABLE`のどちらも指定されていない場合は、スカラー関数が作成されます。           |
| 関数名 | はい       | 作成する関数の名前。このパラメータにデータベースの名前を含めることができます。例：`db1.my_func`。`関数名`にデータベース名を含めた場合は、そのデータベースにUDFが作成されます。それ以外の場合、現在のデータベースにUDFが作成されます。新しい関数の名前とそのパラメータは、宛先データベース内で既存の名前と同じにすることはできません。それ以外の場合、関数は作成できません。関数名が同じでもパラメータが異なる場合は作成に成功します。 |
| arg_type      | はい       | 関数の引数のデータ型。追加された引数は`, ...`で表されることができます。サポートされているデータ型については、[Java UDF](../../sql-functions/JAVA_UDF.md#mapping-between-sql-data-types-and-java-data-types)を参照してください。|
| return_type      | はい       | 関数の戻り値のデータ型。サポートされているデータ型については、[Java UDF](../../sql-functions/JAVA_UDF.md#mapping-between-sql-data-types-and-java-data-types)を参照してください。 |
| PROPERTIES    | はい       | UDFの種類に応じて異なる機能を持つ関数のプロパティ。詳細については、[Java UDF](../../sql-functions/JAVA_UDF.md#step-6-create-the-udf-in-starrocks)を参照してください。 |