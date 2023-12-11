---
displayed_sidebar: "Japanese"
---

# Java UDFs

v2.2.0以降、Javaプログラミング言語を使用して、特定のビジネスニーズに合わせてユーザー定義関数（UDF）をコンパイルできます。

v3.0以降、StarRocksはグローバルUDFをサポートしており、関連するSQLステートメント（CREATE/SHOW/DROP）に`GLOBAL`キーワードを含める必要があります。

このトピックでは、さまざまなUDFの開発および使用方法について説明します。

現在、StarRocksは、スカラーUDF、ユーザー定義集約関数（UDAF）、ユーザー定義ウィンドウ関数（UDWF）、ユーザー定義テーブル関数（UDTF）をサポートしています。

## 前提条件

- [Apache Maven](https://maven.apache.org/download.cgi)をインストールしていること。これにより、Javaプロジェクトを作成およびコンパイルできます。

- サーバーにJDK 1.8がインストールされていること。

- Java UDF機能が有効になっていること。FE構成ファイル**fe/conf/fe.conf**でFE構成項目`enable_udf`を`true`に設定してこの機能を有効にし、その後FEノードを再起動して設定を有効にします。詳細については、「[パラメーター構成](../../administration/Configuration.md)」を参照してください。

## UDFの開発および使用

Javaプログラミング言語を使用して、Mavenプロジェクトを作成し、必要なUDFをコンパイルする必要があります。

### ステップ1：Mavenプロジェクトの作成

以下は、基本的なディレクトリ構造を持つMavenプロジェクトを作成します。

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

スカラーUDFは、1行のデータに対して操作を行い、単一の値を返します。クエリでスカラーUDFを使用すると、各行が結果セットの単一の値に対応します。典型的なスカラー関数には、`UPPER`、`LOWER`、`ROUND`、`ABS`などがあります。

JSONデータのフィールドの値がJSONオブジェクトではなくJSON文字列であるとします。JSON文字列を抽出するSQLステートメントを使用する場合は、例えば`GET_JSON_STRING`を2回実行する必要があります。例えば、`GET_JSON_STRING('{"key":"{\\"k0\\":\\"v0\\"}"}', "$.key")`。

SQLステートメントを単純化するために、JSON文字列を直接抽出できるスカラーUDFをコンパイルできます。例えば、`MY_UDF_JSON_GET('{"key":"{\\"k0\\":\\"v0\\"}"}', "$.key.k0")`。

```Java
package com.starrocks.udf.sample;

import com.alibaba.fastjson.JSONPath;

public class UDFJsonGet {
    public final String evaluate(String obj, String key) {
        if (obj == null || key == null) return null;
        try {
            // The JSONPath library can be fully expanded even if the values of a field are JSON strings.
            return JSONPath.read(obj, key).toString();
        } catch (Exception e) {
            return null;
        }
    }
}
```

ユーザー定義クラスは、以下の表に記載されているメソッドを実装する必要があります。

> **注**
>
> メソッド内のリクエストパラメータと戻りパラメータのデータ型は、[ステップ6](#step-6-create-the-udf-in-starrocks)で実行される`CREATE FUNCTION`ステートメントで宣言されているものと同じでなければならず、本トピックの「SQLデータ型とJavaデータ型のマッピング」セクションで提供されているマッピングに準拠していなければなりません。

| メソッド名               | 説明                         |
| ------------------------ | -------------------------- |
| TYPE1 evaluate(TYPE2, ...) | UDFを実行します。evaluate()メソッドには、publicメンバーアクセスレベルが必要です。 |

#### UDAFのコンパイル

UDAFは複数のデータ行に操作を行い、単一の値を返します。一般的な集約関数には、`SUM`、`COUNT`、`MAX`、`MIN`などがあります。これらは、各GROUP BY句で指定された複数のデータ行を集約し、単一の値を返します。

`MY_SUM_INT`というUDAFをコンパイルしたいとします。`SUM`とは異なり、`MY_SUM_INT`関数はBIGINT型の値を返す代わりに、INTデータ型のリクエストパラメータと戻りパラメータをサポートしています。

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

ユーザー定義クラスは、以下の表に記載されているメソッドを実装する必要があります。

> **注**
>
> メソッド内のリクエストパラメータと戻りパラメータのデータ型は、[ステップ6](#step-6-create-the-udf-in-starrocks)で実行される`CREATE FUNCTION`ステートメントで宣言されているものと同じでなければならず、本トピックの「SQLデータ型とJavaデータ型のマッピング」セクションで提供されているマッピングに準拠していなければなりません。

| メソッド名                            | 説明                         |
| --------------------------------- | -------------------------- |
| State create()                    | ステータスを作成します。             |
| void destroy(State)               | ステータスを破棄します。             |
| void update(State, ...)           | ステータスを更新します。`State`以外にも、UDFの宣言で1つ以上のリクエストパラメータを指定できます。 |
| void serialize(State, ByteBuffer) | ステータスをバイトバッファにシリアル化します。  |
| void merge(State, ByteBuffer)     | バイトバッファからステータスをデシリアライズし、バッファをステータスにマージします。 |
| TYPE finalize(State)              | ステータスからUDFの最終結果を取得します。|

コンパイル時には、以下の表に記載されている`java.nio.ByteBuffer`クラスおよびローカル変数`serializeLength`を使用する必要があります。

| クラスおよびローカル変数 | 説明 |
| ------------------------ | -------------------------- |
| java.nio.ByteBuffer()    | 中間結果を格納するバッファクラス。中間結果は、実行のためにノード間で転送される際にシリアル化またはデシリアライズされる場合があります。そのため、中間結果のデシリアライズに許可されている長さを指定する`serializeLength`変数を使用する必要があります。 |
| serializeLength()        | 途中結果の逆シリアル化を許可する長さ。単位: バイト。 このローカル変数を INT型の値に設定します。たとえば、`State { int counter = 0; public int serializeLength() { return 4; }}`では、途中結果はINTデータ型で、逆シリアル化の長さは4バイトであることが指定されます。これらの設定はビジネス要件に基づいて調整できます。たとえば、途中結果のデータ型をLONGとし、逆シリアル化の長さを8バイトとする場合は、`State { long counter = 0; public int serializeLength() { return 8; }}`を渡します。 |

`java.nio.ByteBuffer` クラスに保存された途中結果の逆シリアル化を行う際に以下の点に注意してください：

- `ByteBuffer` クラスに依存する remaining() メソッドは、逆シリアル化のためのステートに対して呼び出すことはできません。
- `ByteBuffer` クラスに対して clear() メソッドを呼び出すことはできません。
- `serializeLength` の値は、書き込まれたデータの長さと同じでなければなりません。そうでない場合、シリアル化および逆シリアル化中に正しくない結果が生成されます。

#### UDWFをコンパイル

通常の集約関数とは異なり、UDWFは複数の行のセットで動作し、これらの行をまとめてウィンドウと呼ばれる値をそれぞれ返します。典型的なウィンドウ関数には`OVER`句が含まれており、これにより行が複数のセットに分割されます。ウィンドウ関数は各行のために値を返します。

`MY_WINDOW_SUM_INT`というUDWFをコンパイルするとします。`SUM`とは異なり、`MY_WINDOW_SUM_INT`関数はBIGINT型の値を返す代わりに、INTデータ型のリクエストパラメータと返り値をサポートします。

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

ユーザー定義クラスは、UDWFのメソッドが実装されている必要があります（UDWFは特殊な集約関数であるため）し、次の表に記載されている windowUpdate() メソッドを実装する必要があります。

> **注記**
>
> メソッド内のリクエストパラメータと返り値のデータ型は、[前提条件](#mapping-between-sql-data-types-and-java-data-types)で提供される SQLデータ型と Javaデータ型のマッピングに準拠している必要があります。

| メソッド名                                                   | 説明                                                  |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| void windowUpdate(State state, int, int, int , int, ...) | ウィンドウのデータを更新します。UDWFに関する詳細については、[ウィンドウ関数](../sql-functions/Window_function.md)を参照してください。各行を入力として入力するたびに、このメソッドはウィンドウ情報を取得し、途中結果を適宜更新します。<ul><li>`peer_group_start`: 現在のパーティションの開始位置。`OVER` 句でパーティション列を指定することで、パーティション列の値が同じ行は同じパーティションに含まれると見なされます。</li><li>`peer_group_end`: 現在のパーティションの終了位置。</li><li>`frame_start`: 現在のウィンドウフレームの開始位置。ウィンドウフレーム句は、現在の行と現在の行から指定した距離内にある行をカバーする計算範囲を指定します。たとえば、`ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING`では、現在の行、現在の行の前の1行、現在の行の後の1行をカバーする計算範囲を指定します。</li><li>`frame_end`: 現在のウィンドウフレームの終了位置。</li><li>`inputs`: ウィンドウへの入力として入力されるデータ。これは特定のデータ型のみをサポートする配列パッケージです。この例では、INT値が入力され、配列パッケージは Integer[] です。</li></ul> |

#### UDTFをコンパイル

UDTFは1行のデータを読み取り、複数の値を返すことができるテーブルと見なされることができるテーブルを返します。テーブル値関数は典型的には行を列に変換するために使用されます。

> **注記**
>
> StarRocksでは、UDTFは複数の行と1つの列で構成されるテーブルを返すことができます。

`MY_UDF_SPLIT`というUDTFをコンパイルするとします。`MY_UDF_SPLIT`関数では、デリミタとしてスペースを使用し、STRINGデータ型のリクエストパラメータおよび返り値をサポートします。

```Java
package com.starrocks.udf.sample;

public class UDFSplit{
    public String[] process(String in) {
        if (in == null) return null;
        return in.split(" ");
    }
}
```

ユーザー定義クラスで定義されたメソッドは、次の要件を満たしている必要があります：

> **注記**
>
> メソッド内のリクエストパラメータと返り値のデータ型は、[前提条件](#mapping-between-sql-data-types-and-java-data-types)で提供される SQLデータ型と Javaデータ型のマッピングに準拠している必要があります。

| メソッド名           | 説明                         |
| ---------------- | ----------------------------------- |
| TYPE[] process() | UDTFを実行して配列を返します。 |

### Step 4: Javaプロジェクトをパッケージ化

次のコマンドを実行してJavaプロジェクトをパッケージ化してください：

```Bash
mvn package
```

以下のJARファイルが **target** フォルダに生成されます: **udf-1.0-SNAPSHOT.jar** および **udf-1.0-SNAPSHOT-jar-with-dependencies.jar**。

### Step 5: Javaプロジェクトをアップロード

JARファイル **udf-1.0-SNAPSHOT-jar-with-dependencies.jar** を、StarRocksクラスタ内のすべてのFEおよびBEからアクセス可能で動作しているHTTPサーバーにアップロードしてください。そして、次のコマンドを実行してファイルをデプロイしてください：

```Bash
mvn deploy 
```

Pythonを使用して簡単なHTTPサーバーを設定し、JARファイルをそのHTTPサーバーにアップロードすることができます。

> **注記**
>
> [Step 6](#step-6-create-the-udf-in-starrocks)で、FEはUDFのコードを含むJARファイルをチェックし、チェックサムを計算し、BEはJARファイルをダウンロードして実行します。

### Step 6: StarRocksでUDFを作成

StarRocksでは、データベース名前空間とグローバル名前空間の2種類の名前空間でUDFを作成することができます：

- UDFに対する表示や分離の要件がない場合、グローバルUDFとして作成することができます。その後、関数名をカタログとデータベース名をプレフィックスとして関数名に含めることなく、グローバルUDFを参照することができます。
- UDFに対する表示や分離の要件があり、または異なるデータベースで同じUDFを作成する必要がある場合は、それぞれのデータベースに作成することができます。これにより、セッションが対象データベースに接続されている場合は、関数名を使用してUDFを参照することができます。セッションが対象データベース以外のカタログやデータベースに接続されている場合は、UDFを参照する際にカタログとデータベース名をプレフィックスとして関数名に含める必要があります。たとえば、`catalog.database.function` のようになります。

> **注意**
>
> グローバルUDFを作成して使用する前に、システム管理者に必要な許可を付与してもらう必要があります。詳細については、[GRANT](../sql-statements/account-management/GRANT.md)を参照してください。

JARパッケージをアップロードした後、StarRocksでUDFを作成することができます。グローバルUDFの場合、作成文に `GLOBAL` キーワードを含める必要があります。

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
| GLOBAL        | 不       | グローバルUDFを作成するかどうか。v3.0からサポートされています。 |
| AGGREGATE     | いいえ  | UDAFまたはUDWFを作成するかどうか。       |
| TABLE         | いいえ  | UDTFを作成するかどうか。 `AGGREGATE`および`TABLE`の両方が指定されていない場合、スカラー関数が作成されます。               |
| function_name | はい       | 作成する関数の名前。このパラメータにデータベースの名前を含めることができます。たとえば、`db1.my_func`のようになります。`function_name`がデータベース名を含む場合、そのデータベースにUDFが作成されます。それ以外の場合、現在のデータベースにUDFが作成されます。新しい関数とそのパラメータの名前は、宛先データベースの既存の名前と同じにすることはできません。そうでない場合、関数は作成できません。関数名は同じでも、パラメータが異なる場合、作成が成功します。 |
| arg_type      | はい       | 関数の引数のタイプ。追加される引数は`、 ...`によって表されることができます。サポートされるデータタイプについては、[SQLデータ型とJavaデータ型のマッピング](#mapping-between-sql-data-types-and-java-data-types)を参照してください。|
| return_type      | はい       | 関数の戻り値のタイプ。サポートされるデータタイプについては、[Java UDF](#mapping-between-sql-data-types-and-java-data-types)を参照してください。 |
| PROPERTIES    | はい       | 作成するUDFのタイプに応じて異なる関数のプロパティ。 |

#### スカラーUDFの作成

前述の例でコンパイルしたスカラーUDFを作成するには、次のコマンドを実行します。

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
| symbol    | UDFが属するMavenプロジェクトのクラスの名前。このパラメータの値は`<package_name>.<class_name>`形式です。 |
| type      | UDFのタイプ。UDFがJavaベースの関数であることを指定するには、値を`StarrocksJar`に設定します。 |
| file      | UDFのコードを含むJARファイルをダウンロードできるHTTP URL。このパラメータの値は`http://<http_server_ip>:<http_server_port>/<jar_package_name>`形式です。 |

#### UDAFの作成

前述の例でコンパイルしたUDAFを作成するには、次のコマンドを実行します。

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

PROPERTIESのパラメータの説明は、[スカラーUDFを作成する](#スカラーUDFの作成)と同じです。

#### UDWFの作成

前述の例でコンパイルしたUDWFを作成するには、次のコマンドを実行します。

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

`analytic`: UDFがウィンドウ関数であるかどうか。値を`true`に設定します。その他のプロパティの説明は、[スカラーUDFを作成する](#スカラーUDFの作成)と同じです。

#### UDTFの作成

前述の例でコンパイルしたUDTFを作成するには、次のコマンドを実行します。

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

PROPERTIESのパラメータの説明は、[スカラーUDFを作成する](#スカラーUDFの作成)と同じです。

### ステップ7：UDFの使用

UDFを作成したら、ビジネス上のニーズに基づいてテストおよび使用することができます。

#### スカラーUDFの使用

前述の例で作成したスカラーUDFを使用するには、次のコマンドを実行します。

```SQL
SELECT MY_UDF_JSON_GET('{"key":"{\\"in\\":2}"}', '$.key.in');
```

#### UDAFの使用

前述の例で作成したUDAFを使用するには、次のコマンドを実行します。

```SQL
SELECT MY_SUM_INT(col1);
```

#### UDWFの使用

前述の例で作成したUDWFを使用するには、次のコマンドを実行します。

```SQL
SELECT MY_WINDOW_SUM_INT(intcol) 
            OVER (PARTITION BY intcol2
                  ORDER BY intcol3
                  ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
FROM test_basic;
```

#### UDTFの使用

前述の例で作成したUDTFを使用するには、次のコマンドを実行します。

```Plain
-- t1という名前のテーブルがあり、そのカラムa、b、およびc1の情報は次のようになっていると仮定します。
テーブル名：t1
| a | b | c1          |
|---|---|-------------|
| 1 | 2.1 | "hello world" |
| 2 | 2.2 | "hello UDTF." |

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
> - 上記のコードスニペットの最初の`MY_UDF_SPLIT`は、2番目の`MY_UDF_SPLIT`によって返される列のエイリアスです。
> - 表とその戻り値の列のエイリアスを指定するために`AS t2(f1)`を使用することはできません。

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

## パラメータ設定

StarRocksクラスタの各Java仮想マシン（JVM）の**be/conf/hadoop_env.sh**ファイルで、次の環境変数を設定して、メモリ使用量を制御します。ファイル内の他のパラメータも構成できます。

```Bash
export LIBHDFS_OPTS="-Xloggc:$STARROCKS_HOME/log/be.gc.log -server"
```

## よくある質問

UDFを作成する際、静的変数を使用することはできますか？異なるUDFの静的変数は互いに影響を及ぼしますか？

はい、UDFをコンパイルする際に静的変数を使用することができます。異なるUDFの静的変数は互いに影響を及ぼしません。たとえUDFに同一の名前のクラスが含まれていても、それらは互いに影響しません。