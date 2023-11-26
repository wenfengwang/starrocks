---
displayed_sidebar: "Japanese"
---

# Java UDFs

v2.2.0以降、Javaプログラミング言語を使用して、特定のビジネスニーズに合わせてユーザー定義関数（UDF）をコンパイルすることができます。

v3.0以降、StarRocksはグローバルUDFをサポートしており、関連するSQLステートメント（CREATE/SHOW/DROP）に`GLOBAL`キーワードを含めるだけで使用できます。

このトピックでは、さまざまなUDFの開発と使用方法について説明します。

現在、StarRocksはスカラーUDF、ユーザー定義集約関数（UDAF）、ユーザー定義ウィンドウ関数（UDWF）、およびユーザー定義テーブル関数（UDTF）をサポートしています。

## 前提条件

- [Apache Maven](https://maven.apache.org/download.cgi)をインストールしていること。これにより、Javaプロジェクトを作成およびコンパイルすることができます。

- サーバーにJDK 1.8がインストールされていること。

- Java UDF機能が有効になっていること。FEの設定ファイル**fe/conf/fe.conf**のFE設定項目`enable_udf`を`true`に設定してこの機能を有効にし、その後、FEノードを再起動して設定を有効にします。詳細については、[パラメータの設定](../../administration/Configuration.md)を参照してください。

## UDFの開発と使用

Javaプログラミング言語を使用して必要なUDFを作成し、コンパイルする必要があります。

### ステップ1：Mavenプロジェクトの作成

以下のディレクトリ構造を持つMavenプロジェクトを作成します。

```Plain
project
|--pom.xml
|--src
|  |--main
|  |  |--java
|  |  |--resources
|  |--test
|--target
```

### ステップ2：依存関係の追加

以下の依存関係を**pom.xml**ファイルに追加します。

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

### ステップ3：UDFのコンパイル

Javaプログラミング言語を使用してUDFをコンパイルします。

#### スカラーUDFのコンパイル

スカラーUDFは、1行のデータに対して操作を行い、1つの値を返します。クエリでスカラーUDFを使用する場合、各行は結果セット内の1つの値に対応します。典型的なスカラー関数には、`UPPER`、`LOWER`、`ROUND`、`ABS`などがあります。

JSONデータのフィールドの値がJSONオブジェクトではなくJSON文字列である場合、JSON文字列を抽出するために`GET_JSON_STRING`を2回実行する必要があります。たとえば、`GET_JSON_STRING(GET_JSON_STRING('{"key":"{\\"k0\\":\\"v0\\"}"}', "$.key"), "$.k0")`のようになります。

SQLステートメントを簡略化するために、JSON文字列を直接抽出できるスカラーUDFをコンパイルすることができます。たとえば、`MY_UDF_JSON_GET('{"key":"{\\"k0\\":\\"v0\\"}"}', "$.key.k0")`のようになります。

```Java
package com.starrocks.udf.sample;

import com.alibaba.fastjson.JSONPath;

public class UDFJsonGet {
    public final String evaluate(String obj, String key) {
        if (obj == null || key == null) return null;
        try {
            // JSONPathライブラリは、フィールドの値がJSON文字列であっても完全に展開することができます。
            return JSONPath.read(obj, key).toString();
        } catch (Exception e) {
            return null;
        }
    }
}
```

ユーザー定義クラスは、次の表に記載されているメソッドを実装する必要があります。

> **注記**
>
> メソッドの要求パラメータと戻りパラメータのデータ型は、[ステップ6](#step-6-create-the-udf-in-starrocks)で実行される`CREATE FUNCTION`ステートメントで宣言されたものと同じでなければなりません。また、このトピックの"[SQLデータ型とJavaデータ型のマッピング](#mapping-between-sql-data-types-and-java-data-types)"セクションで提供されるマッピングに準拠している必要があります。

| メソッド                     | 説明                                                  |
| -------------------------- | ------------------------------------------------------------ |
| TYPE1 evaluate(TYPE2, ...) | UDFを実行します。evaluate()メソッドは、publicメンバーアクセスレベルが必要です。 |

#### UDAFのコンパイル

UDAFは、複数の行のデータに対して操作を行い、1つの値を返します。典型的な集約関数には、`SUM`、`COUNT`、`MAX`、`MIN`などがあります。これらの関数は、GROUP BY句で指定された複数の行のデータを集約し、1つの値を返します。

組み込みの集約関数`SUM`はBIGINT型の値を返しますが、`MY_SUM_INT`関数はINTデータ型のリクエストパラメータと戻りパラメータのみをサポートします。

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
            state.counter+= val;
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

ユーザー定義クラスは、次の表に記載されているメソッドを実装する必要があります。

> **注記**
>
> メソッドの要求パラメータと戻りパラメータのデータ型は、[ステップ6](#step-6-create-the-udf-in-starrocks)で実行される`CREATE FUNCTION`ステートメントで宣言されたものと同じでなければなりません。また、このトピックの"[SQLデータ型とJavaデータ型のマッピング](#mapping-between-sql-data-types-and-java-data-types)"セクションで提供されるマッピングに準拠している必要があります。

| メソッド                            | 説明                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| State create()                    | ステートを作成します。                                             |
| void destroy(State)               | ステートを破棄します。                                            |
| void update(State, ...)           | ステートを更新します。UDF宣言で最初のパラメータ`State`のほかに、UDF宣言で1つ以上のリクエストパラメータを指定することもできます。 |
| void serialize(State, ByteBuffer) | ステートをバイトバッファにシリアライズします。                     |
| void merge(State, ByteBuffer)     | バイトバッファからステートをデシリアライズし、バイトバッファをステートにマージします。 |
| TYPE finalize(State)              | ステートからUDFの最終結果を取得します。            |

コンパイル時には、以下の表に記載されているバッファクラス`java.nio.ByteBuffer`とローカル変数`serializeLength`も使用する必要があります。

| クラスとローカル変数 | 説明                                                  |
| ------------------------ | ------------------------------------------------------------ |
| java.nio.ByteBuffer()    | バッファクラスで、中間結果を格納します。中間結果は、実行のためにノード間で転送される際にシリアライズまたはデシリアライズされる場合があります。そのため、中間結果のデシリアライズに許可される長さも`serializeLength`変数を使用して指定する必要があります。 |
| serializeLength()        | 中間結果のデシリアライズに許可される長さです。単位：バイト。このローカル変数をINT型の値に設定します。たとえば、`State { int counter = 0; public int serializeLength() { return 4; }}`は、中間結果がINTデータ型であり、デシリアライズの長さが4バイトであることを指定しています。これらの設定は、ビジネス要件に基づいて調整することができます。たとえば、中間結果のデータ型をLONGとし、デシリアライズの長さを8バイトとする場合は、`State { long counter = 0; public int serializeLength() { return 8; }}`を渡します。 |

`java.nio.ByteBuffer`クラスに格納されている中間結果のデシリアライズに関しては、次の点に注意してください。

- `ByteBuffer`クラスに依存する`remaining()`メソッドは、ステートのデシリアライズには呼び出せません。
- `ByteBuffer`クラスでは`clear()`メソッドを呼び出すことはできません。
- `serializeLength`の値は、書き込まれたデータの長さと同じでなければなりません。そうでない場合、シリアライズおよびデシリアライズ時に正しくない結果が生成されます。

#### UDWFのコンパイル

通常の集約関数とは異なり、UDWFは複数の行からなるセット（ウィンドウと呼ばれる）で操作を行い、各行に対して値を返します。典型的なウィンドウ関数には、行を複数のセットに分割する`OVER`句を含むものがあります。各セットの行に対して計算を実行し、各行に対して値を返します。

組み込みの集約関数`SUM`はBIGINT型の値を返しますが、`MY_WINDOW_SUM_INT`関数はINTデータ型のリクエストパラメータと戻りパラメータのみをサポートします。

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
            state.counter+=val;
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
        for (int i = (int)frame_start; i < (int)frame_end; ++i) {
            state.counter += inputs[i];
        }
    }
}
```

ユーザー定義クラスは、UDAF（UDWFは特別な集約関数であるため）で必要なメソッドと、以下の表に記載されているwindowUpdate()メソッドを実装する必要があります。

> **注記**
>
> メソッドの要求パラメータと戻りパラメータのデータ型は、[ステップ6](#step-6-create-the-udf-in-starrocks)で実行される`CREATE FUNCTION`ステートメントで宣言されたものと同じでなければなりません。また、このトピックの"[SQLデータ型とJavaデータ型のマッピング](#mapping-between-sql-data-types-and-java-data-types)"セクションで提供されるマッピングに準拠している必要があります。

| メソッド                                                   | 説明                                                  |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| void windowUpdate(State state, int, int, int , int, ...) | ウィンドウのデータを更新します。UDWFについての詳細は、[ウィンドウ関数](../sql-functions/Window_function.md)を参照してください。このメソッドは、入力として1行入力するたびにウィンドウ情報を取得し、中間結果を適切に更新します。<ul><li>`peer_group_start`：現在のパーティションの開始位置です。OVER句で`PARTITION BY`を使用してパーティション列を指定します。パーティション列の値が同じ行は、同じパーティションに属していると見なされます。</li><li>`peer_group_end`：現在のパーティションの終了位置です。</li><li>`frame_start`：現在のウィンドウフレームの開始位置です。ウィンドウフレーム句は、現在の行と現在の行から指定された距離内の行をカバーする計算範囲を指定します。たとえば、`ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING`は、現在の行、現在の行の前の行、および現在の行の後の行をカバーする計算範囲を指定します。</li><li>`frame_end`：現在のウィンドウフレームの終了位置です。</li><li>`inputs`：ウィンドウに入力されるデータ。データは特定のデータ型のみをサポートする配列パッケージです。この例では、INT値が入力され、配列パッケージはInteger[]です。</li></ul> |

#### UDTFのコンパイル

UDTFは1行のデータを読み取り、複数の値を返す関数であり、これらの値はテーブルと見なすことができます。テーブル値関数は通常、行を列に変換するために使用されます。

> **注記**
>
> StarRocksでは、UDTFは複数の行と1つの列で構成されるテーブルを返すことができます。

`MY_UDF_SPLIT`という名前のUDTFをコンパイルする場合、`MY_UDF_SPLIT`関数を使用してスペースを区切り文字として使用し、STRINGデータ型のリクエストパラメータと戻りパラメータをサポートすることができます。

```Java
package com.starrocks.udf.sample;

public class UDFSplit{
    public String[] process(String in) {
        if (in == null) return null;
        return in.split(" ");
    }
}
```

ユーザー定義クラスは、次の表に記載されているメソッドを実装する必要があります。

> **注記**
>
> メソッドの要求パラメータと戻りパラメータのデータ型は、[ステップ6](#step-6-create-the-udf-in-starrocks)で実行される`CREATE FUNCTION`ステートメントで宣言されたものと同じでなければなりません。また、このトピックの"[SQLデータ型とJavaデータ型のマッピング](#mapping-between-sql-data-types-and-java-data-types)"セクションで提供されるマッピングに準拠している必要があります。

| メソッド           | 説明                         |
| ---------------- | ----------------------------------- |
| TYPE[] process() | UDTFを実行し、配列を返します。 |

### ステップ4：Javaプロジェクトのパッケージ化

以下のコマンドを実行してJavaプロジェクトをパッケージ化します。

```Bash
mvn package
```

**target**フォルダに以下のJARファイルが生成されます：**udf-1.0-SNAPSHOT.jar**および**udf-1.0-SNAPSHOT-jar-with-dependencies.jar**。

### ステップ5：Javaプロジェクトのアップロード

**udf-1.0-SNAPSHOT-jar-with-dependencies.jar**ファイルを、StarRocksクラスタのすべてのFEおよびBEからアクセス可能なHTTPサーバーにアップロードします。その後、次のコマンドを実行してファイルをデプロイします。

```Bash
mvn deploy 
```

Pythonを使用して簡単なHTTPサーバーを設定し、そのHTTPサーバーにJARファイルをアップロードすることもできます。

> **注記**
>
> [ステップ6](#step-6-create-the-udf-in-starrocks)では、FEはUDFのコードを含むJARファイルをチェックし、チェックサムを計算し、BEはJARファイルをダウンロードして実行します。

### ステップ6：StarRocksでUDFを作成

StarRocksでは、データベース名前空間とグローバル名前空間の2つのタイプの名前空間でUDFを作成できます。

- UDFに対して表示または分離の要件がない場合、グローバルUDFとして作成することができます。その場合、関数名の接頭辞としてカタログ名とデータベース名を含める必要はありません。
- UDFに対して表示または分離の要件がある場合、または同じUDFを異なるデータベースに作成する必要がある場合、各個別のデータベースに作成することができます。その場合、セッションがターゲットデータベースに接続されている場合、関数名を使用してUDFを参照することができます。セッションがターゲットデータベース以外のカタログまたはデータベースに接続されている場合は、関数名の接頭辞としてカタログ名とデータベース名を含める必要があります。たとえば、`catalog.database.function`のようになります。

> **注意**
>
> グローバルUDFを作成して使用する前に、システム管理者に必要な権限を付与してもらう必要があります。詳細については、[GRANT](../sql-statements/account-management/GRANT.md)を参照してください。

JARパッケージをアップロードした後、StarRocksでUDFを作成することができます。グローバルUDFの場合、作成ステートメントに`GLOBAL`キーワードを含める必要があります。

#### 構文

```sql
CREATE [GLOBAL][AGGREGATE | TABLE] FUNCTION function_name
(arg_type [, ...])
RETURNS return_type
PROPERTIES ("key" = "value" [, ...])
```

#### パラメータ

| **パラメータ**      | **必須** | **説明**                                                     |
| ------------- | -------- | ------------------------------------------------------------ |
| GLOBAL        | いいえ       | グローバルUDFを作成するかどうか。v3.0からサポートされています。 |
| AGGREGATE     | いいえ       | UDAFまたはUDWFを作成するかどうか。       |
| TABLE         | いいえ       | UDTFを作成するかどうか。`AGGREGATE`および`TABLE`が指定されていない場合、スカラー関数が作成されます。               |
| function_name | はい       | 作成する関数の名前。このパラメータにデータベース名を含めることもできます。たとえば、`db1.my_func`のようになります。`function_name`にデータベース名が含まれている場合、UDFはそのデータベースに作成されます。それ以外の場合、UDFは現在のデータベースに作成されます。新しい関数の名前とそのパラメータは、宛先データベースの既存の名前と同じであってはなりません。そうでない場合、関数は作成できません。関数名が同じでもパラメータが異なる場合、作成は成功します。 |
| arg_type      | はい       | 関数の引数の型。追加する引数は`, ...`で表すことができます。サポートされているデータ型については、[SQLデータ型とJavaデータ型のマッピング](#mapping-between-sql-data-types-and-java-data-types)を参照してください。|
| return_type      | はい       | 関数の戻り値の型。サポートされているデータ型については、[Java UDF](#mapping-between-sql-data-types-and-java-data-types)を参照してください。 |
| PROPERTIES    | はい       | 関数のプロパティ。作成するUDFのタイプによって異なります。 |

#### スカラーUDFの作成

以下のコマンドを実行して、前述の例でコンパイルしたスカラーUDFを作成します。

```SQL
CREATE [GLOBAL] FUNCTION MY_UDF_JSON_GET(string, string) 
RETURNS string
PROPERTIES (
    "symbol" = "com.starrocks.udf.sample.UDFJsonGet", 
    "type" = "StarrocksJar",
    "file" = "http://http_host:http_port/udf-1.0-SNAPSHOT-jar-with-dependencies.jar"
);
```

| パラメータ | 説明                                                  |
| --------- | ------------------------------------------------------------ |
| symbol    | UDFが所属するMavenプロジェクトのクラス名。このパラメータの値は`<package_name>.<class_name>`形式です。 |
| type      | UDFのタイプ。UDFがJavaベースの関数であることを指定するために、値を`StarrocksJar`に設定します。 |
| file      | UDFのコードを含むJARファイルをダウンロードできるHTTP URL。このパラメータの値は`http://<http_server_ip>:<http_server_port>/<jar_package_name>`形式です。 |

#### UDAFの作成

以下のコマンドを実行して、前述の例でコンパイルしたUDAFを作成します。

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

PROPERTIES内のパラメータの説明は、[スカラーUDFの作成](#create-a-scalar-udf)と同じです。

#### UDWFの作成

以下のコマンドを実行して、前述の例でコンパイルしたUDWFを作成します。

```SQL
CREATE [GLOBAL] AGGREGATE FUNCTION MY_WINDOW_SUM_INT(Int)
RETURNS Int
properties 
(
    "analytic" = "true",
    "symbol" = "com.starrocks.udf.sample.WindowSumInt", 
    "type" = "StarrocksJar", 
    "file" = "http://http_host:http_port/udf-1.0-SNAPSHOT-jar-with-dependencies.jar"    
);
```

`analytic`：UDFがウィンドウ関数であるかどうか。値を`true`に設定します。その他のプロパティの説明は、[スカラーUDFの作成](#create-a-scalar-udf)と同じです。

#### UDTFの作成

以下のコマンドを実行して、前述の例でコンパイルしたUDTFを作成します。

```SQL
CREATE [GLOBAL] TABLE FUNCTION MY_UDF_SPLIT(string)
RETURNS string
properties 
(
    "symbol" = "com.starrocks.udf.sample.UDFSplit", 
    "type" = "StarrocksJar", 
    "file" = "http://http_host:http_port/udf-1.0-SNAPSHOT-jar-with-dependencies.jar"
);
```

PROPERTIES内のパラメータの説明は、[スカラーUDFの作成](#create-a-scalar-udf)と同じです。

### ステップ7：UDFの使用

UDFを作成した後、ビジネスニーズに基づいてテストおよび使用することができます。

#### スカラーUDFの使用

以下のコマンドを実行して、前述の例で作成したスカラーUDFを使用します。

```SQL
SELECT MY_UDF_JSON_GET('{"key":"{\\"in\\":2}"}', '$.key.in');
```

#### UDAFの使用

以下のコマンドを実行して、前述の例で作成したUDAFを使用します。

```SQL
SELECT MY_SUM_INT(col1);
```

#### UDWFの使用

以下のコマンドを実行して、前述の例で作成したUDWFを使用します。

```SQL
SELECT MY_WINDOW_SUM_INT(intcol) 
            OVER (PARTITION BY intcol2
                  ORDER BY intcol3
                  ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
FROM test_basic;
```

#### UDTFの使用

以下のコマンドを実行して、前述の例で作成したUDTFを使用します。

```Plain
-- t1という名前のテーブルがあり、その列a、b、およびc1の情報は次のようになっているとします：
SELECT t1.a,t1.b,t1.c1 FROM t1;
> 出力:
1,2.1,"hello world"
2,2.2,"hello UDTF."

-- MY_UDF_SPLIT()関数を実行します。
SELECT t1.a,t1.b, MY_UDF_SPLIT FROM t1, MY_UDF_SPLIT(t1.c1); 
> 出力:
1,2.1,"hello"
1,2.1,"world"
2,2.2,"hello"
2,2.2,"UDTF."
```

> **注記**
>
> - 前のコードスニペットの最初の`MY_UDF_SPLIT`は、2番目の`MY_UDF_SPLIT`によって返される列のエイリアスであり、関数です。
> - テーブルとその列のエイリアスを返すために`AS t2(f1)`を使用することはできません。

## UDFの表示

以下のコマンドを実行してUDFをクエリできます。

```SQL
SHOW [GLOBAL] FUNCTIONS;
```

詳細については、[SHOW FUNCTIONS](../sql-statements/data-definition/SHOW_FUNCTIONS.md)を参照してください。

## UDFの削除

以下のコマンドを実行してUDFを削除します。

```SQL
DROP [GLOBAL] FUNCTION <function_name>(arg_type [, ...]);
```

詳細については、[DROP FUNCTION](../sql-statements/data-definition/DROP_FUNCTION.md)を参照してください。

## SQLデータ型とJavaデータ型のマッピング

| SQL TYPE       | Java TYPE         |
| -------------- | ----------------- |
| BOOLEAN        | java.lang.Boolean |
| TINYINT        | java.lang.Byte    |
| SMALLINT       | java.lang.Short   |
| INT            | java.lang.Integer |
| BIGINT         | java.lang.Long    |
| FLOAT          | java.lang.Float   |
| DOUBLE         | java.lang.Double  |
| STRING/VARCHAR | java.lang.String  |

## パラメータの設定

メモリ使用量を制御するために、StarRocksクラスタの各Java仮想マシン（JVM）の**be/conf/hadoop_env.sh**ファイルで以下の環境変数を設定します。他のパラメータもファイルで設定できます。

```Bash
export LIBHDFS_OPTS="-Xloggc:$STARROCKS_HOME/log/be.gc.log -server"
```

## よくある質問

UDFを作成する際に静的変数を使用することはできますか？異なるUDFの静的変数は互いに影響を与えますか？

はい、UDFをコンパイルする際に静的変数を使用することができます。異なるUDFの静的変数は互いに影響を与えず、互いに分離されます。
