---
displayed_sidebar: "Japanese"
---

# CREATE FUNCTION（関数の作成）

## 説明

ユーザー定義関数（UDF）を作成します。現在、Scalar関数、ユーザー定義集約関数（UDAF）、ユーザー定義ウィンドウ関数（UDWF）、およびユーザー定義テーブル関数（UDTF）を含むJava UDFの作成のみがサポートされています。

**Java UDFのコンパイル、作成、使用方法の詳細については、[Java UDF](../../sql-functions/JAVA_UDF.md)を参照してください。**

> **注意**
>
> グローバルUDFを作成するには、SYSTEMレベルのCREATE GLOBAL FUNCTION権限が必要です。データベース全体のUDFを作成するには、DATABASEレベルのCREATE FUNCTION権限が必要です。

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
| GLOBAL        | いいえ       | グローバルUDFを作成するかどうかを指定します。v3.0以降でサポートされています。  |
| AGGREGATE     | いいえ       | UDAFまたはUDWFを作成するかどうかを指定します。       |
| TABLE         | いいえ       | UDTFを作成するかどうかを指定します。`AGGREGATE`と`TABLE`の両方が指定されていない場合、Scalar関数が作成されます。               |
| function_name | はい       | 作成する関数の名前です。このパラメータにデータベースの名前を含めることができます。例えば、`db1.my_func`のように指定します。`function_name`にデータベース名が含まれている場合、UDFはそのデータベースに作成されます。それ以外の場合、UDFは現在のデータベースに作成されます。新しい関数の名前とそのパラメータは、宛先データベースの既存の名前と同じであってはなりません。そうでない場合、関数は作成できません。関数名は同じでもパラメータが異なる場合、作成は成功します。 |
| arg_type      | はい       | 関数の引数の型です。追加の引数は`, ...`で表現することができます。サポートされているデータ型については、[Java UDF](../../sql-functions/JAVA_UDF.md#mapping-between-sql-data-types-and-java-data-types)を参照してください。|
| return_type      | はい       | 関数の戻り値の型です。サポートされているデータ型については、[Java UDF](../../sql-functions/JAVA_UDF.md#mapping-between-sql-data-types-and-java-data-types)を参照してください。 |
| PROPERTIES    | はい       | 作成するUDFのタイプによって異なる関数のプロパティです。詳細については、[Java UDF](../../sql-functions/JAVA_UDF.md#step-6-create-the-udf-in-starrocks)を参照してください。 |
