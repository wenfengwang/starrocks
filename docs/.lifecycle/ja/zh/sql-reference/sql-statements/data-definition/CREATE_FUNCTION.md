---
displayed_sidebar: Chinese
---

# CREATE FUNCTION

## 機能

UDF（ユーザー定義関数）を作成します。現在はJava UDFのみの作成がサポートされており、これはJava言語で書かれたカスタム関数を指します。具体的には以下が含まれます：

- Scalar UDF：カスタムスカラー関数。
- UDAF：カスタム集約関数。
- UDWF：カスタムウィンドウ関数。
- UDTF：カスタムテーブル値関数。

**UDFの作成と使用に関する詳細は、[Java UDF](../../sql-functions/JAVA_UDF.md)を参照してください。**

> **注意**
>
> - グローバルUDFを作成するには、SYSTEMレベルのCREATE GLOBAL FUNCTION権限が必要です。
> - データベースレベルのUDFを作成するには、DATABASEレベルのCREATE FUNCTION権限が必要です。

## 文法

```SQL
CREATE [GLOBAL][AGGREGATE | TABLE] FUNCTION function_name(arg_type [, ...])
RETURNS return_type
[PROPERTIES ("key" = "value" [, ...]) ]
```

## パラメータ説明

| **パラメータ** | **必須** | **説明**                                                     |
| --------------- | -------- | ------------------------------------------------------------ |
| GLOBAL          | いいえ       | グローバルUDFを作成する場合は、このキーワードを指定する必要があります。バージョン3.0からサポートされています。 |
| AGGREGATE       | いいえ       | UDAFやUDWFを作成する場合は、このキーワードを指定する必要があります。 |
| TABLE           | いいえ       | UDTFを作成する場合は、このキーワードを指定する必要があります。 |
| function_name   | はい       | 関数名で、データベース名を含むことができます。例：`db1.my_func`。`function_name`にデータベース名が含まれている場合、そのUDFは対応するデータベースに作成されます。そうでない場合は、現在のデータベースに作成されます。新しい関数名とパラメータがターゲットデータベース内の既存の関数と同じ場合、作成に失敗します。ただし、関数名のみが同じでパラメータが異なる場合は、作成に成功します。 |
| arg_type        | はい       | 関数のパラメータタイプです。サポートされているデータタイプの詳細は、[Java UDF](../../sql-functions/JAVA_UDF.md#型マッピング)を参照してください。 |
| return_type     | はい       | 関数の戻り値のタイプです。サポートされているデータタイプの詳細は、[Java UDF](../../sql-functions/JAVA_UDF.md#型マッピング)を参照してください。 |
| PROPERTIES      | はい       | 関数に関連する属性です。異なるタイプのUDFを作成するには、異なる属性を設定する必要があります。詳細と例については、[Java UDF](../../sql-functions/JAVA_UDF.md#ステップ6-StarRocksでUDFを作成する)を参照してください。 |
