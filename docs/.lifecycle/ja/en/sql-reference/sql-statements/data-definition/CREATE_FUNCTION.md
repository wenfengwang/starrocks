---
displayed_sidebar: English
---

# CREATE FUNCTION

## 説明

ユーザー定義関数（UDF）を作成します。現在、スカラー関数、ユーザー定義集計関数（UDAF）、ユーザー定義ウィンドウ関数（UDWF）、およびユーザー定義テーブル関数（UDTF）を含むJava UDFのみを作成できます。

**Java UDFのコンパイル、作成、および使用方法の詳細については、[Java UDF](../../sql-functions/JAVA_UDF.md)を参照してください。**

> **注記**
>
> グローバルUDFを作成するには、SYSTEMレベルのCREATE GLOBAL FUNCTION権限が必要です。データベース全体のUDFを作成するには、データベースレベルのCREATE FUNCTION権限が必要です。

## 構文

```sql
CREATE [GLOBAL][AGGREGATE | TABLE] FUNCTION function_name
(arg_type [, ...])
RETURNS return_type
PROPERTIES ("key" = "value" [, ...])
```

## パラメータ

| **パラメータ**      | **必須** | **説明**                                                     |
| ------------- | -------- | ------------------------------------------------------------ |
| GLOBAL        | いいえ       | v3.0からサポートされるグローバルUDFを作成するかどうか。  |
| AGGREGATE     | いいえ       | UDAFまたはUDWFを作成するかどうか。       |
| TABLE         | いいえ       | UDTFを作成するかどうか。`AGGREGATE`と`TABLE`の両方が指定されていない場合、スカラー関数が作成されます。               |
| function_name | はい       | 作成する関数の名前。このパラメータには、例えば`db1.my_func`のようにデータベースの名前を含めることができます。`function_name`にデータベース名が含まれている場合、UDFはそのデータベースに作成されます。それ以外の場合、UDFは現在のデータベースに作成されます。新しい関数とそのパラメータの名前が目的のデータベースに既に存在する名前と同じである場合、関数は作成できません。関数名が同じでもパラメータが異なる場合は、作成が成功します。|
| arg_type      | はい       | 関数の引数の型。追加された引数は`[, ...]`で表されます。サポートされているデータ型については、[Java UDF](../../sql-functions/JAVA_UDF.md#mapping-between-sql-data-types-and-java-data-types)を参照してください。|
| return_type      | はい       | 関数の戻り値の型。サポートされているデータ型については、[Java UDF](../../sql-functions/JAVA_UDF.md#mapping-between-sql-data-types-and-java-data-types)を参照してください。 |
| PROPERTIES    | はい       | 関数のプロパティで、作成するUDFのタイプによって異なります。詳細については、[Java UDF](../../sql-functions/JAVA_UDF.md#step-6-create-the-udf-in-starrocks)を参照してください。|
