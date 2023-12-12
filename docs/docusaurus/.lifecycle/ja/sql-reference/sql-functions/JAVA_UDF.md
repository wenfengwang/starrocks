---
displayed_sidebar: "Japanese"
---

# Java UDF（ユーザー定義関数）

v2.2.0以降、Javaプログラミング言語を使用して、ユーザー定義関数（UDF）をコンパイルして特定のビジネスニーズに合わせることができます。

v3.0以降、StarRocksはグローバルUDFをサポートし、関連するSQLステートメント（CREATE/SHOW/DROP）に「GLOBAL」キーワードを含めるだけで済みます。

このトピックでは、さまざまなUDFの開発と使用方法について説明します。

現在、StarRocksは、スカラーUDF、ユーザー定義集約関数（UDAF）、ユーザー定義ウィンドウ関数（UDWF）、およびユーザー定義テーブル関数（UDTF）をサポートしています。

## 前提条件

- [Apache Maven](https://maven.apache.org/download.cgi)がインストールされていること。これにより、Javaプロジェクトを作成およびコンパイルできます。

- サーバーにJDK 1.8がインストールされていること。

- Java UDF機能が有効になっていること。「fe/conf/fe.conf」のFE構成ファイルでFE構成項目「enable_udf」を「true」に設定してこの機能を有効にし、その後FEノードを再起動して設定を有効にします。詳細については[パラメータ構成](../../administration/Configuration.md)を参照してください。

## UDFの開発と使用

Javaプログラミング言語を使用して、必要なUDFを作成しコンパイルする必要があります。

### ステップ1：Mavenプロジェクトの作成

以下の基本的なディレクトリ構造を持つMavenプロジェクトを作成します:

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

以下の依存関係を「pom.xml」ファイルに追加します:

```XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd\">
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

スカラーUDFは、1つのデータ行に対して操作を行い、1つの値を返します。クエリでスカラーUDFを使用すると、各行が結果セットの1つの値に対応します。典型的なスカラー関数には、`UPPER`、`LOWER`、`ROUND`、`ABS`などがあります。

JSONデータのフィールドの値がJSONオブジェクトではなくJSON文字列である場合を想定してみましょう。JSON文字列を抽出するSQLステートメントを使用する際には、「GET_JSON_STRING('{"key":"{\\"k0\\":\\"v0\\"}"}', "$.key")」のように2回実行する必要があります。

SQLステートメントを簡略化するために、たとえば、「MY_UDF_JSON_GET('{"key":"{\\"k0\\":\\"v0\\"}"}', "$.key.k0")」のように直接JSON文字列を抽出できるスカラーUDFをコンパイルできます。

```Java
package com.starrocks.udf.sample;

import com.alibaba.fastjson.JSONPath;

public class UDFJsonGet {
    public final String evaluate(String obj, String key) {
        if (obj == null || key == null) return null;
        try {
            // このJSONPathライブラリは、フィールドの値がJSON文字列である場合でも完全に展開できます。
            return JSONPath.read(obj, key).toString();
        } catch (Exception e) {
            return null;
        }
    }
}
```

ユーザー定義クラスは、次の表に記載されている方法を実装する必要があります。

> **注意**
>
> メソッド内のリクエストパラメータと戻りパラメータのデータ型は、[ステップ6](#step-6-create-the-udf-in-starrocks)で実行される`CREATE FUNCTION`ステートメントで宣言されているものと同じでなければならず、このトピックの「SQLデータ型とJavaデータ型のマッピング」セクションで提供されているマッピングに準拠しなければなりません。

| メソッド               | 説明                       |
| ---------------------- | -------------------------- |
| TYPE1 evaluate(TYPE2, ...) | UDFを実行します。`evaluate()`メソッドはpublicメンバーアクセスレベルを要求します。 |

#### UDAFのコンパイル

UDAFは、複数のデータ行に対して操作を行い、1つの値を返します。集約関数には、`SUM`、`COUNT`、`MAX`、`MIN`などがあり、それぞれのGROUP BY句で指定された複数のデータ行を集約し、1つの値を返します。

`MY_SUM_INT`という名前のUDAFをコンパイルしたいとします。`SUM`とは異なり、`MY_SUM_INT`関数はBIGINT型の値を返す代わりに、INTデータ型のリクエストパラメータと戻りパラメータのみをサポートします。

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

ユーザー定義クラスは、次の表に記載されている方法を実装する必要があります。

> **注意**
>
> メソッド内のリクエストパラメータと戻りパラメータのデータ型は、[ステップ6](#step-6-create-the-udf-in-starrocks)で実行される`CREATE FUNCTION`ステートメントで宣言されているものと同じでなければならず、このトピックの「SQLデータ型とJavaデータ型のマッピング」セクションで提供されているマッピングに準拠しなければなりません。

| メソッド                            | 説明                           |
| --------------------------------- | ------------------------------ |
| State create()                    | 状態を作成します。                 |
| void destroy(State)               | 状態を破棄します。                 |
| void update(State, ...)           | 状態を更新します。`State`の他にも、UDF宣言で1つ以上のリクエストパラメータを指定できます。 |
| void serialize(State, ByteBuffer) | 状態をバイトバッファにシリアライズします。      |
| void merge(State, ByteBuffer)     | 状態をバイトバッファからデシリアライズし、バッファを状態にマージします。 |
| TYPE finalize(State)              | 状態からUDFの最終結果を取得します。          |

コンパイル時には、次の表に記載されているバッファクラス`java.nio.ByteBuffer`およびローカル変数`serializeLength`も使用する必要があります。

| クラスおよびローカル変数 | 説明                                      |
| ------------------------ | --------------------------------------- |
| java.nio.ByteBuffer()    | バッファクラス。中間結果は、ノード間で実行のために転送される際にシリアライズまたはデシリアライズされる可能性があります。そのため、デシリアライズのために許可されるサイズを指定するために、`serializeLength`変数を使用する必要があります。 |
| serializeLength()        | 中間結果の逆シリアル化に許可されている長さ。単位: バイト。このローカル変数を INT 型の値に設定します。たとえば、`State { int counter = 0; public int serializeLength() { return 4; }}` は、中間結果が INT データ型であり、逆シリアル化の長さが 4 バイトであることを指定しています。これらの設定はビジネス要件に基づいて調整できます。たとえば、中間結果のデータ型を LONG として、逆シリアル化の長さを 8 バイトと指定したい場合は、`State { long counter = 0; public int serializeLength() { return 8; }}` を渡します。|

中間結果を `java.nio.ByteBuffer` クラスに逆シリアル化する際の以下の点に注意してください：

- `ByteBuffer` クラスに依存する remaining() メソッドは、状態を逆シリアル化するために呼び出すことはできません。
- `ByteBuffer` クラスに対して clear() メソッドを呼び出すことはできません。
- `serializeLength` の値は、書き込まれたデータの長さと同じでなければなりません。そうでない場合、シリアル化および逆シリアル化中に誤った結果が生成されます。

#### UDWF のコンパイル

通常の集約関数とは異なり、UDWF は複数の行のセット (つまり、ウィンドウと呼ばれる) 上で動作し、各行に対して値を返します。典型的なウィンドウ関数には、`OVER` 句が含まれ、行を複数のセットに分割します。各セットの行に対して計算を実行し、各行に値を返します。

例えば、`MY_WINDOW_SUM_INT` という UDWF をコンパイルしたいとします。`SUM` という組み込みの集約関数が BIGINT 型の値を返すのに対し、`MY_WINDOW_SUM_INT` 関数は INT データ型の要求パラメータと戻りパラメータをサポートします。

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

UDWF は特別な集約関数であるため、ユーザー定義クラスでは UDAF で必要なメソッド (ウィンドウ関数の場合は windowUpdate()) を実装する必要があります。

> **注記**
>
> メソッドで使用される要求パラメータと戻りパラメータのデータ型は、[ステップ 6](#step-6-create-the-udf-in-starrocks) で実行される `CREATE FUNCTION` ステートメントで宣言されたものと同じでなければなりません。また、このトピックの "[SQL データ型と Java データ型のマッピング](#mapping-between-sql-data-types-and-java-data-types)" セクションで提供されるマッピングに準拠していなければなりません。

#### UDTF のコンパイル

UDTF は1行のデータを読み込み、複数の値を返すことができるテーブルとみなせる関数です。テーブル値関数は通常、行を列に変換するために使用されます。

例えば、`MY_UDF_SPLIT` という UDTF をコンパイルしたいとします。`MY_UDF_SPLIT` 関数では、区切り文字としてスペースを使用し、STRING データ型の要求パラメータと戻りパラメータをサポートします。

```Java
package com.starrocks.udf.sample;

public class UDFSplit{
    public String[] process(String in) {
        if (in == null) return null;
        return in.split(" ");
    }
}
```

ユーザー定義クラスで定義されたメソッドは、以下の要件を満たす必要があります。

> **注記**
>
> メソッドで使用される要求パラメータと戻りパラメータのデータ型は、[ステップ 6](#step-6-create-the-udf-in-starrocks) で実行される `CREATE FUNCTION` ステートメントで宣言されたものと同じでなければなりません。また、このトピックの "[SQL データ型と Java データ型のマッピング](#mapping-between-sql-data-types-and-java-data-types)" セクションで提供されるマッピングに準拠していなければなりません。

| メソッド           | 説明                         |
| ---------------- | ----------------------------------- |
| TYPE[] process() | UDTF を実行し、配列を返します。 |

### ステップ 4: Java プロジェクトをパッケージ化

以下のコマンドを実行して Java プロジェクトをパッケージ化します:

```Bash
mvn package
```

次の JAR ファイルが **target** フォルダに生成されます: **udf-1.0-SNAPSHOT.jar** と **udf-1.0-SNAPSHOT-jar-with-dependencies.jar**。

### ステップ 5: Java プロジェクトをアップロード

**udf-1.0-SNAPSHOT-jar-with-dependencies.jar** ファイルを、StarRocks クラスタ内のすべての FE および BE からアクセス可能で、継続的に動作している HTTP サーバにアップロードします。その後、次のコマンドを実行してファイルをデプロイします:

```Bash
mvn deploy 
```

Python を使用して簡単な HTTP サーバを設定し、その HTTP サーバに JAR ファイルをアップロードできます。

> **注記**
>
> [ステップ 6](#step-6-create-the-udf-in-starrocks) で、FE は UDF のコードを含む JAR ファイルをチェックし、チェックサムを計算し、BE は JAR ファイルをダウンロードして実行します。

### ステップ 6: StarRocks で UDF を作成

StarRocks では、データベース名前空間およびグローバル名前空間の2種類の名前空間で UDF を作成できます。

- UDF に対して可視性または隔離性の要件がない場合、グローバル UDF として作成できます。その後、関数名の接頭辞にカタログ名とデータベース名を含めずに、関数名のみでグローバル UDF を参照できます。
- UDF に対して可視性または隔離性の要件があり、または異なるデータベースで同じ UDF を作成する必要がある場合は、各個々のデータベース内で作成できます。そのような場合、セッションが対象のデータベースに接続されている場合は、関数名を使用して UDF を参照できます。セッションが対象のデータベースとは異なるカタログまたはデータベースに接続されている場合は、関数名の接頭辞にカタログとデータベース名を含める必要があります。たとえば、`catalog.database.function` とします。

> **注意**
>
> グローバル UDF を作成して使用する前に、必要なアクセス許可を与えるためにシステム管理者に連絡する必要があります。詳細については、[GRANT](../sql-statements/account-management/GRANT.md) を参照してください。
| 集約関数     | いいえ       | UDAFまたはUDWFを作成するかどうか。       |
| TABLE         | いいえ       | UDTFを作成するかどうか。 `AGGREGATE`と`TABLE`の両方が指定されていない場合、スカラー関数が作成されます。               |
| 関数名 | はい       | 作成する関数の名前。このパラメータにデータベースの名前を含めることができます。「db1.my_func」のように、関数名にデータベース名を含めると、そのデータベースにUDFが作成されます。それ以外の場合は、現在のデータベースにUDFが作成されます。新しい関数の名前とそのパラメータは、宛先データベースの既存の名前と同じであってはなりません。そうでない場合は、関数を作成できません。関数名が同じでも、パラメータが異なる場合は作成に成功します。|
| 引数のタイプ      | はい       | 関数の引数タイプ。追加された引数は、「, ...」で表すことができます。サポートされているデータ型については、[SQLデータ型とJavaデータ型のマッピング](#mapping-between-sql-data-types-and-java-data-types)を参照してください。|
| 戻り値のタイプ      | はい       | 関数の戻り値のタイプ。サポートされているデータ型については、[Java UDF](#mapping-between-sql-data-types-and-java-data-types)を参照してください。 |
| PROPERTIES    | はい       | UDFの種類に応じて異なる関数のプロパティ。 |

#### スカラーUDFの作成

前の例でコンパイルしたスカラーUDFを作成するには、次のコマンドを実行します。

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
| symbol    | UDFが属するMavenプロジェクトのクラスの名前。このパラメータの値は `<package_name>.<class_name>`形式です。 |
| type      | UDFのタイプ。UDFがJavaベースの関数であることを指定するために値を`StarrocksJar`に設定します。 |
| file      | UDFのコードが含まれるJARファイルをダウンロードできるHTTPのURL。このパラメータの値は `http://<http_server_ip>:<http_server_port>/<jar_package_name>`形式です。 |

#### UDAFの作成

前の例でコンパイルしたUDAFを作成するには、次のコマンドを実行します。

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

前の例でコンパイルしたUDWFを作成するには、次のコマンドを実行します。

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

`analytic`: UDFがウィンドウ関数であるかどうか。値を`true`に設定します。その他のプロパティの説明は[スカラーUDFの作成](#create-a-scalar-udf)と同じです。

#### UDTFの作成

前の例でコンパイルしたUDTFを作成するには、次のコマンドを実行します。

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

### ステップ7: UDFの使用

UDFを作成した後は、ビジネスのニーズに基づいてテストして使用することができます。

#### スカラーUDFの使用

前の例で作成したスカラーUDFを使用するには、次のコマンドを実行します。

```SQL
SELECT MY_UDF_JSON_GET('{"key":"{\\"in\\":2}"}', '$.key.in');
```

#### UDAFの使用

前の例で作成したUDAFを使用するには、次のコマンドを実行します。

```SQL
SELECT MY_SUM_INT(col1);
```

#### UDWFの使用

前の例で作成したUDWFを使用するには、次のコマンドを実行します。

```SQL
SELECT MY_WINDOW_SUM_INT(intcol) 
            OVER (PARTITION BY intcol2
                  ORDER BY intcol3
                  ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
FROM test_basic;
```

#### UDTFの使用

前の例で作成したUDTFを使用するには、次のコマンドを実行します。

```Plain
-- t1という名前のテーブルがあり、そのカラムa、b、c1の情報が次のようになっているとします：
SELECT t1.a,t1.b,t1.c1 FROM t1;
> 出力:
1,2.1,"hello world"
2,2.2,"hello UDTF."

-- MY_UDF_SPLIT() 関数を実行します。
SELECT t1.a,t1.b, MY_UDF_SPLIT FROM t1, MY_UDF_SPLIT(t1.c1); 
> 出力:
1,2.1,"hello"
1,2.1,"world"
2,2.2,"hello"
2,2.2,"UDTF."
```

> **注**
>
> - 前のコードスニペットの最初の`MY_UDF_SPLIT`は、2番目の`MY_UDF_SPLIT`によって返される列のエイリアスであり、関数です。
> - 返されるテーブルとその列のエイリアスを指定するために `AS t2(f1)` を使用することはできません。

## UDFの表示

UDFをクエリするには、次のコマンドを実行します。

```SQL
SHOW [GLOBAL] FUNCTIONS;
```

詳細については、[SHOW FUNCTIONS](../sql-statements/data-definition/SHOW_FUNCTIONS.md)を参照してください。

## UDFの削除

UDFを削除するには、次のコマンドを実行します。

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

メモリ使用量を制御するために、StarRocksクラスタの各Java仮想マシン（JVM）の**be/conf/hadoop_env.sh**ファイルで次の環境変数を設定します。ファイルでその他のパラメータを設定することもできます。

```Bash
export LIBHDFS_OPTS="-Xloggc:$STARROCKS_HOME/log/be.gc.log -server"
```

## よくある質問

UDFを作成する際に静的変数を使用することはできますか？異なるUDFの静的変数は互いに影響を及ぼしますか？

はい、UDFをコンパイルする際に静的変数を使用することができます。異なるUDFの静的変数は互いに影響を及ぼさず、名前が同じクラスを持っていても影響を及ぼしません。