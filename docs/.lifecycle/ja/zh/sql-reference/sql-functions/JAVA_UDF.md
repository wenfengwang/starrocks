---
displayed_sidebar: Chinese
---

# Java UDF

バージョン 2.2.0 から、StarRocks は Java 言語でユーザー定義関数（User Defined Function、略して UDF）を書くことをサポートしています。

バージョン 3.0 から、StarRocks は Global UDF をサポートしており、関連する SQL ステートメント（CREATE/SHOW/DROP）に `GLOBAL` キーワードを追加するだけで、そのステートメントはグローバルに有効になり、各データベースに対して個別にこのステートメントを実行する必要はありません。ビジネスシーンに応じてカスタム関数を開発し、StarRocks の関数機能を拡張することができます。

この文書では、UDF の書き方と使い方について説明します。

現在 StarRocks がサポートしている UDF には、ユーザー定義スカラー関数（Scalar UDF）、ユーザー定義集約関数（User Defined Aggregation Function、UDAF）、ユーザー定義ウィンドウ関数（User Defined Window Function、UDWF）、ユーザー定義テーブル関数（User Defined Table Function、UDTF）が含まれます。

## 前提条件

StarRocks の Java UDF 機能を使用する前に、以下のことが必要です:

- [Apache Maven をインストール](https://maven.apache.org/download.cgi)して、関連する Java プロジェクトを作成し、コードを書きます。
- サーバーに JDK 1.8 をインストールします。
- UDF 機能を有効にします。FE の設定ファイル **fe/conf/fe.conf** で `enable_udf` を `true` に設定し、FE ノードを再起動して設定を有効にします。詳細な操作と設定項目のリストは[設定パラメータ](../../administration/FE_configuration.md)を参照してください。

## UDF の開発と使用

Maven プロジェクトを作成し、Java 言語で必要な機能をコーディングする必要があります。

### ステップ 1: Maven プロジェクトの作成

1. Maven プロジェクトを作成し、基本的なディレクトリ構造は以下の通りです:

    ```Plain Text
    project
    |--pom.xml
    |--src
    |  |--main
    |  |  |--java
    |  |  |--resources
    |  |--test
    |--target
    ```

### ステップ 2: 依存関係の追加

**pom.xml** に以下の依存関係を追加します:

```XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>udf</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.76</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <version>2.10</version>
                <executions>
                    <execution>
                        <id>copy-dependencies</id>
                        <phase>package</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>${project.build.directory}/lib</outputDirectory>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.3.0</version>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### ステップ 3: UDF の開発

Java 言語を使用して、必要な UDF を開発する必要があります。

#### Scalar UDF の開発

Scalar UDF は、ユーザー定義のスカラー関数で、単一行のデータを操作し、単一行の結果を出力することができます。Scalar UDF をクエリで使用すると、各行のデータが最終的に結果セットに行ごとに表示されます。典型的なスカラー関数には `UPPER`、`LOWER`、`ROUND`、`ABS` があります。

以下の例では、JSON データから値を抽出する機能を説明しています。例えば、ビジネスシーンでは、JSON データのあるフィールドの値が JSON オブジェクトではなく JSON 文字列である可能性があります。そのため、JSON 文字列を抽出する際には、SQL ステートメントで `GET_JSON_STRING` をネストして呼び出す必要があります。つまり、`GET_JSON_STRING(GET_JSON_STRING('{"key":"{\\"k0\\":\\"v0\\"}"}', "$.key"), "$.k0")` のようになります。

SQL ステートメントを簡素化するために、JSON 文字列を直接抽出する UDF を開発することができます。例えば：`MY_UDF_JSON_GET('{"key":"{\\"k0\\":\\"v0\\"}"}', "$.key.k0")`。

```Java
package com.starrocks.udf.sample;

import com.alibaba.fastjson.JSONPath;

public class UDFJsonGet {
    public final String evaluate(String obj, String key) {
        if (obj == null || key == null) return null;
        try {
            // JSONPath ライブラリは、フィールドの値が JSON 形式の文字列であっても、すべてを展開することができます
            return JSONPath.read(obj, key).toString();
        } catch (Exception e) {
            return null;
        }
    }
}
```

ユーザー定義クラスは、以下のメソッドを実装する必要があります：

> **説明**
>
> メソッドのリクエストパラメータと戻り値のデータ型は、ステップ 6 の `CREATE FUNCTION` ステートメントで宣言されたものと同じでなければならず、両者の型マッピング関係は[型マッピング関係](#型マッピング関係)に従う必要があります。

| メソッド                        | 説明                                                   |
| -------------------------- | ------------------------------------------------------ |
| TYPE1 evaluate(TYPE2, ...) | `evaluate` メソッドは UDF の呼び出し入口であり、public メンバーメソッドでなければなりません。    |

#### UDAF の開発

UDAF は、ユーザー定義の集約関数で、複数行のデータを操作し、単一行の結果を出力します。典型的な集約関数には `SUM`、`COUNT`、`MAX`、`MIN` があり、これらの関数は GROUP BY で分割された各グループの複数行のデータを集約して、単一行の結果を出力します。

以下の例では、`MY_SUM_INT` 関数について説明しています。内蔵関数 `SUM`（戻り値の型は BIGINT）との違いは、`MY_SUM_INT` 関数は入力パラメータと戻り値の型が INT であることをサポートしています。

```Java
package com.starrocks.udf.sample;

public class SumInt {
    public static class State {
        int counter = 0;
        public int serializeLength() { return 4; }
    }

    public State create() {
        return new State();
    }

    public void destroy(State state) {
    }

    public final void update(State state, Integer val) {
        if (val != null) {
            state.counter += val;
        }
    }

    public void serialize(State state, java.nio.ByteBuffer buff) {
        buff.putInt(state.counter);
    }

    public void merge(State state, java.nio.ByteBuffer buffer) {
        int val = buffer.getInt();
        state.counter += val;
    }

    public Integer finalize(State state) {
        return state.counter;
    }
}
```

ユーザー定義クラスは、以下のメソッドを実装する必要があります：

> **説明**
>
> メソッドの入力パラメータと戻り値のデータ型は、ステップ 6 の `CREATE FUNCTION` ステートメントで宣言されたものと同じでなければならず、両者の型マッピング関係は[型マッピング関係](#型マッピング関係)に従う必要があります。

|               **メソッド**             |                    **説明**                                 |
| --------------------------------- | ------------------------------------------------------------ |
| State create()                    | State を作成します。                                                 |
| void destroy(State)               | State を破棄します。                                                 |
| void update(State, ...)           | State を更新します。第一引数は State で、残りの引数は関数宣言の入力パラメータで、1 つ以上になることがあります。 |
| void serialize(State, ByteBuffer) | State をシリアライズします。                                               |
| void merge(State, ByteBuffer)     | State とシリアライズされた State をマージします。                                |
| TYPE finalize(State)              | State を通じて関数の最終結果を取得します。                              |

さらに、UDAF 関数を開発する際には、バッファクラス `java.nio.ByteBuffer` とローカル変数 `serializeLength` を使用して、中間結果を保存し、表現し、中間結果のシリアライズ長を指定する必要があります。

| **クラスとローカル変数**      | **説明**                                                     |
| --------------------- | ------------------------------------------------------------ |
| java.nio.ByteBuffer() | バッファクラスで、中間結果を保存します。また、中間結果が異なる実行ノード間で転送される際には、シリアライズとデシリアライズが行われるため、serializeLength を使用して中間結果のシリアライズ後の長さを指定する必要があります。 |
| serializeLength()     | 中間結果のシリアライズ後の長さで、単位は Byte です。serializeLength のデータ型は INT に固定されています。例えば、例の `State { int counter = 0; public int serializeLength() { return 4; }}` には、中間結果のシリアライズ後の説明が含まれており、中間結果のデータ型は INT で、シリアライズ長は 4 Byte です。ビジネス要件に応じて調整することもできます。例えば、中間結果のシリアライズ後のデータ型が LONG で、シリアライズ長が 8 Byte の場合は、`State { long counter = 0; public int serializeLength() { return 8; }}` を渡す必要があります。|

> **注意**
>
> `java.nio.ByteBuffer` のシリアライズに関する注意点：
>
> - State のデシリアライズに ByteBuffer の remaining() メソッドに依存することはサポートされていません。
> - ByteBuffer に対して clear() メソッドを呼び出すことはサポートされていません。
> - `serializeLength` は実際に書き込まれたデータの長さと一致している必要があります。そうでない場合、シリアライズとデシリアライズのプロセスで結果が誤ってしまう可能性があります。

#### UDWF の開発

UDWF は、ユーザー定義のウィンドウ関数です。通常の集約関数とは異なり、ウィンドウ関数は一連の行（ウィンドウ）に対して値を計算し、各行に対して結果を返します。通常、ウィンドウ関数には `OVER` 節が含まれ、データ行を複数のグループに分割し、ウィンドウ関数は各行のデータが属するグループ（ウィンドウ）に基づいて計算を行い、各行に対して結果を返します。

以下の例では、`MY_WINDOW_SUM_INT` 関数について説明しています。内蔵関数 `SUM`（戻り値の型は BIGINT）との違いは、`MY_WINDOW_SUM_INT` 関数は入力パラメータと戻り値の型が INT であることをサポートしています。

```Java
package com.starrocks.udf.sample;

public class WindowSumInt {    
    public static class State {
        int counter = 0;
        public int serializeLength() { return 4; }
        @Override
        public String toString() {
            return "State{" +
                    "counter=" + counter +
                    '}';
        }
    }

    public State create() {
        return new State();
    }

    public void destroy(State state) {

    }

    public void update(State state, Integer val) {
        if (val != null) {
            state.counter += val;
        }
    }

    public void serialize(State state, java.nio.ByteBuffer buff) {
        buff.putInt(state.counter);
    }

    public void merge(State state, java.nio.ByteBuffer buffer) {
        int val = buffer.getInt();
        state.counter += val;
    }

    public Integer finalize(State state) {
        return state.counter;
    }

    public void reset(State state) {
        state.counter = 0;
    }

    public void windowUpdate(State state,
                            int peer_group_start, int peer_group_end,
                            int frame_start, int frame_end,
                            Integer[] inputs) {
        for (int i = frame_start; i < frame_end; ++i) {
            state.counter += inputs[i];
        }
    }
}
```

ユーザー定義クラスは、UDAF に必要なメソッド（ウィンドウ関数は特別な集約関数です）と windowUpdate() メソッドを実装する必要があります。

> **説明**

> メソッドのリクエストパラメータと戻り値のデータ型は、ステップ6の `CREATE FUNCTION` ステートメントで宣言されたものと同じでなければならず、両者の型マッピング関係は[型マッピング関係](#型映射关系)に従う必要があります。

##### 追加で実装が必要なメソッド

`void windowUpdate(State state, int, int, int , int, ...)`

メソッドの意味

ウィンドウデータを更新します。ウィンドウ関数の詳細は、[ウィンドウ関数](../sql-reference/sql-functions/Window_function.md)を参照してください。各行のデータが入力されると、対応するウィンドウ情報が取得されて中間結果が更新されます。

- peer_group_start：現在のパーティションの開始位置。<br />パーティション：OVER句のPARTITION BYで指定されたパーティション列により、同じ値を持つ行は同一パーティション内とみなされます。
- peer_group_end：現在のパーティションの終了位置。
- frame_start：現在のウィンドウフレームの開始位置。<br />ウィンドウフレーム：window frame句は計算範囲を指定し、現在行を基準に前後の行をウィンドウ関数の計算対象とします。例えばROWS BETWEEN 1 PRECEDING AND 1 FOLLOWINGは、現在行とその前後1行ずつを計算範囲とすることを意味します。
- frame_end：現在のウィンドウフレームの終了位置。
- inputs：ウィンドウ内の入力データを表し、ラッパークラスの配列です。ラッパークラスは入力データの型に対応する必要があり、この例では入力データ型がINTなので、ラッパークラスの配列はInteger[]です。

#### UDTFの開発

UDTF（User-Defined Table-Valued Function）は、1行のデータを読み取り、複数の値を出力することで1つのテーブルと見なされます。表値関数は、行を列に変換するためによく使用されます。

> **説明**
> 現在、UDTFは複数行単一列のみを返すことができます。

以下の例では、`MY_UDF_SPLIT` 関数を例に説明します。`MY_UDF_SPLIT` 関数はスペースを区切り文字としてサポートし、入力パラメータと戻り値の型はSTRINGです。

```java
package com.starrocks.udf.sample;

public class UDFSplit{
    public String[] process(String in) {
        if (in == null) return null;
        return in.split(" ");
    }
}
```

ユーザー定義クラスは、以下のメソッドを実装する必要があります：

> **説明**
> メソッドのリクエストパラメータと戻り値のデータ型は、ステップ6の `CREATE FUNCTION` ステートメントで宣言されたものと同じでなければならず、両者の型マッピング関係は[型マッピング関係](#型映射关系)に従う必要があります。

| メソッド                | 意味                                      |
| ------------------ | ------------------------------------------- |
| TYPE[] process()   | `process()` メソッドはUDTFの呼び出しエントリであり、配列を返す必要があります。 |

### ステップ4：Javaプロジェクトのパッケージング

以下のコマンドでJavaプロジェクトをパッケージングします。

```shell
mvn package
```

**target** ディレクトリには、**udf-1.0-SNAPSHOT.jar** と **udf-1.0-SNAPSHOT-jar-with-dependencies.jar** の2つのファイルが生成されます。

### ステップ5：プロジェクトのアップロード

**udf-1.0-SNAPSHOT-jar-with-dependencies.jar** ファイルをFEとBEがアクセス可能なHTTPサーバーにアップロードし、HTTPサービスを常に稼働させます。

```shell
mvn deploy 
```

Pythonを使用して簡易HTTPサーバーを作成し、そのサーバーにファイルをアップロードすることができます。

> **説明**
> ステップ6では、FEはUDFが含まれるJarパッケージを検証し、チェックサムを計算します。BEはUDFが含まれるJarパッケージをダウンロードして実行します。

### ステップ6：StarRocksでUDFを作成する

StarRocksには、DatabaseレベルのNamespaceとGlobalレベルのNamespaceの2種類のUDFがあります。

- 特別なUDFの可視性の分離要件がない場合は、Global UDFを直接作成することができます。Global UDFを参照する際は、Function Nameを直接呼び出すだけで、CatalogやDatabaseをプレフィックスとして使用する必要はありません。これにより、アクセスがより便利になります。
- 特別なUDFの可視性の分離要件がある場合、または異なるDatabaseで同名のUDFを作成する必要がある場合は、Database内でUDFを作成することを選択できます。この場合、セッションが特定のDatabase内にある場合は、Function Nameを直接呼び出すことができます。セッションが他のCatalogやDatabaseにある場合は、CatalogとDatabaseのプレフィックスを付けて呼び出す必要があります。例：`catalog.database.function`。

> **注意**
>
> Global UDFを作成するには、SystemレベルのCREATE GLOBAL FUNCTION権限が必要です。DatabaseレベルのUDFを作成するには、DatabaseレベルのCREATE FUNCTION権限が必要です。UDFを使用するには、対応するUDFのUSAGE権限が必要です。権限の付与については、[GRANT](../sql-statements/account-management/GRANT.md)を参照してください。

JARパッケージのアップロードが完了したら、StarRocksで必要に応じて対応するUDFを作成する必要があります。Global UDFを作成する場合は、SQLステートメントに `GLOBAL` キーワードを含めるだけです。

#### 構文

```SQL
CREATE [GLOBAL][AGGREGATE | TABLE] FUNCTION function_name(arg_type [, ...])
RETURNS return_type
[PROPERTIES ("key" = "value" [, ...]) ]
```

#### パラメータ説明

| **パラメータ**      | **必須** | **説明**                                                     |
| ------------- | -------- | -----------------------------------------------------------|
| GLOBAL        | いいえ       | グローバルUDFを作成する場合は、このキーワードを指定する必要があります。バージョン3.0からサポートされています。            |
| AGGREGATE     | いいえ       | UDAFやUDWFを作成する場合は、このキーワードを指定する必要があります。                         |
| TABLE         | いいえ       | UDTFを作成する場合は、このキーワードを指定する必要があります。                              |
| function_name | はい       | 関数名で、データベース名を含むことができます。例えば、`db1.my_func`。`function_name`にデータベース名が含まれている場合、そのUDFは対応するデータベースに作成されます。そうでない場合は、現在のデータベースに作成されます。新しい関数名とパラメータがターゲットデータベースに既に存在する関数と同じであれば、作成に失敗します。関数名のみが同じでパラメータが異なる場合は、作成に成功します。 |
| arg_type      | はい       | 関数のパラメータ型です。サポートされているデータ型については、[型マッピング関係](#型映射关系)を参照してください。 |
| return_type   | はい       | 関数の戻り値の型です。サポートされているデータ型については、[型マッピング関係](#型映射关系)を参照してください。 |
| properties    | はい       | 関数に関連するプロパティです。異なるタイプのUDFを作成する際には、異なるプロパティを設定する必要があります。詳細と例については、以下の例を参照してください。 |

#### Scalar UDFの作成

以下のコマンドを実行して、StarRocksで先に示した例のScalar UDFを作成します。

```SQL
CREATE [GLOBAL] FUNCTION MY_UDF_JSON_GET(string, string) 
RETURNS string
PROPERTIES (
    "symbol" = "com.starrocks.udf.sample.UDFJsonGet", 
    "type" = "StarrocksJar",
    "file" = "http://http_host:http_port/udf-1.0-SNAPSHOT-jar-with-dependencies.jar"
);
```

|パラメータ|説明|
|---|----|
|symbol|UDFが含まれるプロジェクトのクラス名。形式は`<package_name>.<class_name>`です。|
|type|作成されるUDFのタイプをマークするために使用されます。`StarrocksJar`という値は、JavaベースのUDFを意味します。|
|file|UDFが含まれるJarパッケージのHTTPパス。形式は`http://<http_server_ip>:<http_server_port>/<jar_package_name>`です。|

#### UDAFの作成

以下のコマンドを実行して、StarRocksで先に示した例のUDAFを作成します。

```SQL
CREATE [GLOBAL] AGGREGATE FUNCTION MY_SUM_INT(INT) 
RETURNS INT
PROPERTIES 
( 
    "symbol" = "com.starrocks.udf.sample.SumInt", 
    "type" = "StarrocksJar",
    "file" = "http://http_host:http_port/udf-1.0-SNAPSHOT-jar-with-dependencies.jar"
);
```

PROPERTIESのパラメータ説明は[Scalar UDFの作成](#创建-scalar-udf)と同じです。

#### UDWFの作成

以下のコマンドを実行して、StarRocksで先に示した例のUDWFを作成します。

```SQL
CREATE [GLOBAL] AGGREGATE FUNCTION MY_WINDOW_SUM_INT(Int)
RETURNS Int
PROPERTIES 
(
    "analytic" = "true",
    "symbol" = "com.starrocks.udf.sample.WindowSumInt", 
    "type" = "StarrocksJar", 
    "file" = "http://http_host:http_port/udf-1.0-SNAPSHOT-jar-with-dependencies.jar"    
);
```

`analytic`：作成される関数がウィンドウ関数であるかどうかを示します。値は `true` で固定されています。他のパラメータの説明は[Scalar UDFの作成](#创建-scalar-udf)と同じです。

#### UDTFの作成

以下のコマンドを実行して、StarRocksで先に示した例のUDTFを作成します。

```SQL
CREATE [GLOBAL] TABLE FUNCTION MY_UDF_SPLIT(string)
RETURNS string
PROPERTIES 
(
    "symbol" = "com.starrocks.udf.sample.UDFSplit", 
    "type" = "StarrocksJar", 
    "file" = "http://http_host:http_port/udf-1.0-SNAPSHOT-jar-with-dependencies.jar"
);
```

PROPERTIESのパラメータ説明は[Scalar UDFの作成](#创建-scalar-udf)と同じです。

### ステップ7：UDFの使用

作成が完了したら、開発したUDFをテストして使用することができます。

#### Scalar UDFの使用

以下のコマンドを実行して、先に示した例のScalar UDF関数を使用します。

```SQL
SELECT MY_UDF_JSON_GET('{"key":"{\\"in\\":2}"}', '$.key.in');
```

#### UDAFの使用

以下のコマンドを実行して、先に示した例のUDAFを使用します。

```SQL
SELECT MY_SUM_INT(col1);
```

#### UDWFの使用

以下のコマンドを実行して、先に示した例のUDWFを使用します。

```SQL
SELECT MY_WINDOW_SUM_INT(intcol) 
            OVER (PARTITION BY intcol2
                  ORDER BY intcol3
                  ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
FROM test_basic;
```

#### UDTFの使用

以下のコマンドを実行して、先に示した例のUDTFを使用します。

```plain text
-- t1テーブルが存在し、その列a、b、c1の情報は以下の通りです。
SELECT t1.a,t1.b,t1.c1 FROM t1;
> output:
1,2.1,"hello world"
2,2.2,"hello UDTF."

-- MY_UDF_SPLIT()関数を使用します。
SELECT t1.a,t1.b, MY_UDF_SPLIT FROM t1, MY_UDF_SPLIT(t1.c1); 
> output:
1,2.1,"hello"
1,2.1,"world"
2,2.2,"hello"
2,2.2,"UDTF."
```

> **説明**
>
> - 最初の `MY_UDF_SPLIT` は `MY_UDF_SPLIT` を呼び出した後に生成される列のエイリアスです。
> - 現在、`AS t2(f1)` の形式でテーブル関数が返すテーブルのエイリアスと列のエイリアスを指定することはサポートされていません。

## UDF情報の確認

以下のコマンドを実行してUDF情報を確認します。

```sql
SHOW [GLOBAL] FUNCTIONS;
```

詳細は、[SHOW FUNCTIONS](../sql-statements/data-definition/SHOW_FUNCTIONS.md)を参照してください。

## UDFの削除

以下のコマンドを実行して指定されたUDFを削除します。

```sql
DROP [GLOBAL] FUNCTION <function_name>(arg_type [, ...]);
```

詳細は、[DROP FUNCTION](../sql-statements/data-definition/DROP_FUNCTION.md)を参照してください。

## 型マッピング関係

| SQLの型       | Javaの型         |
| -------------- | ----------------- |
| BOOLEAN        | java.lang.Boolean |
| TINYINT        | java.lang.Byte    |
| SMALLINT       | java.lang.Short   |
| INT            | java.lang.Integer |
| BIGINT         | java.lang.Long    |
| FLOAT          | java.lang.Float   |
| DOUBLE         | java.lang.Double  |
| STRING/VARCHAR | java.lang.String  |

## パラメータ設定

JVM パラメータ設定： **be/conf/hadoop_env.sh** に以下の環境変数を設定することで、メモリ使用量を制御できます。

```Bash
export LIBHDFS_OPTS="-Xloggc:$STARROCKS_HOME/log/be.gc.log -server"
```

## FAQ

**Q：UDF を開発する際に静的変数を使用できますか？異なる UDF 間で静的変数が影響し合うことはありますか？**

A：UDF 開発時に静的変数を使用することは可能で、異なる UDF 間（クラス名が同じであっても）、静的変数は互いに隔離されており、影響し合うことはありません。
