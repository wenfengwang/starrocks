---
displayed_sidebar: English
---

# Java UDFs

v2.2.0 以降、Java プログラミング言語を使用して特定のビジネスニーズに合わせたユーザー定義関数（UDF）をコンパイルできます。

v3.0 以降、StarRocks はグローバル UDF をサポートし、関連する SQL ステートメント（CREATE/SHOW/DROP）に `GLOBAL` キーワードを含めるだけで済みます。

このトピックでは、さまざまな UDF の開発と使用方法について説明します。

現在、StarRocks はスカラー UDF、ユーザー定義集約関数（UDAF）、ユーザー定義ウィンドウ関数（UDWF）、およびユーザー定義テーブル関数（UDTF）をサポートしています。

## 前提条件

- [Apache Maven](https://maven.apache.org/download.cgi) をインストールしており、Java プロジェクトを作成およびコンパイルできます。

- サーバーに JDK 1.8 をインストールしています。

- Java UDF 機能が有効です。FE 設定ファイル **fe/conf/fe.conf** で FE 設定項目 `enable_udf` を `true` に設定してこの機能を有効にし、FE ノードを再起動して設定を反映させます。詳細は[パラメータ設定](../../administration/FE_configuration.md)を参照してください。

## UDF の開発と使用

Maven プロジェクトを作成し、Java プログラミング言語を使用して必要な UDF をコンパイルする必要があります。

### ステップ 1: Maven プロジェクトを作成する

次のような基本的なディレクトリ構造を持つ Maven プロジェクトを作成します。

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

### ステップ 2: 依存関係を追加する

**pom.xml** ファイルに以下の依存関係を追加します。

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

### ステップ 3: UDF をコンパイルする

Java プログラミング言語を使用して UDF をコンパイルします。

#### スカラー UDF のコンパイル

スカラー UDF は、1 行のデータに対して操作を行い、1 つの値を返します。クエリでスカラー UDF を使用する場合、各行は結果セット内の単一の値に対応します。一般的なスカラー関数には `UPPER`、`LOWER`、`ROUND`、`ABS` などがあります。

例えば、JSON データのフィールドの値が JSON オブジェクトではなく JSON 文字列である場合、SQL ステートメントを使用して JSON 文字列を抽出する際には、`GET_JSON_STRING(GET_JSON_STRING('{"key":"{\\"k0\\":\\"v0\\"}"}', "$.key"), "$.k0")` のように `GET_JSON_STRING` を2回実行する必要があります。

SQL ステートメントを簡略化するために、JSON 文字列を直接抽出できるスカラー UDF をコンパイルすることができます。例：`MY_UDF_JSON_GET('{"key":"{\\"k0\\":\\"v0\\"}"}', "$.key.k0")`。

```Java
package com.starrocks.udf.sample;

import com.alibaba.fastjson.JSONPath;

public class UDFJsonGet {
    public final String evaluate(String obj, String key) {
        if (obj == null || key == null) return null;
        try {
            // JSONPath ライブラリは、フィールドの値が JSON 文字列であっても完全に展開できます。
            return JSONPath.read(obj, key).toString();
        } catch (Exception e) {
            return null;
        }
    }
}
```

ユーザー定義クラスは、以下の表に示すメソッドを実装する必要があります。

> **注記**
>
> メソッド内のリクエストパラメータおよび戻り値のデータ型は、[ステップ 6](#step-6-create-the-udf-in-starrocks)で実行される `CREATE FUNCTION` ステートメントで宣言されたものと同じであり、このトピックの「[SQL データ型と Java データ型のマッピング](#mapping-between-sql-data-types-and-java-data-types)」セクションに示されているマッピングに準拠している必要があります。

| メソッド                     | 説明                                                  |
| -------------------------- | ------------------------------------------------------------ |
| TYPE1 evaluate(TYPE2, ...) | UDF を実行します。evaluate() メソッドは public メンバーアクセスレベルが必要です。 |

#### UDAF のコンパイル

UDAF は、複数の行のデータに対して操作を行い、1 つの値を返します。一般的な集約関数には `SUM`、`COUNT`、`MAX`、`MIN` などがあり、それぞれ GROUP BY 句で指定された複数の行のデータを集約して単一の値を返します。

例えば、`MY_SUM_INT` という名前の UDAF をコンパイルしたいとします。組み込みの集約関数 `SUM` が BIGINT 型の値を返すのに対し、`MY_SUM_INT` 関数は INT データ型のリクエストパラメータと戻り値のみをサポートします。

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

ユーザー定義クラスは、以下の表に示すメソッドを実装する必要があります。

> **注記**
>
> メソッド内のリクエストパラメータおよび戻り値のデータ型は、[ステップ 6](#step-6-create-the-udf-in-starrocks)で実行される `CREATE FUNCTION` ステートメントで宣言されたものと同じであり、このトピックの「[SQL データ型と Java データ型のマッピング](#mapping-between-sql-data-types-and-java-data-types)」セクションに示されているマッピングに準拠している必要があります。

| メソッド                            | 説明                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| State create()                    | ステートを作成します。                                             |
| void destroy(State)               | ステートを破棄します。                                            |
| void update(State, ...)           | ステートを更新します。`State` 以外にも、UDF 宣言で指定されたリクエストパラメータを含めることができます。 |
| void serialize(State, ByteBuffer) | ステートを ByteBuffer にシリアライズします。                     |
| void merge(State, ByteBuffer)     | ByteBuffer からステートをデシリアライズし、ByteBuffer をステートにマージします。 |
| TYPE finalize(State)              | ステートから UDF の最終結果を取得します。            |

コンパイル時には、以下の表に示す `java.nio.ByteBuffer` クラスと `serializeLength` ローカル変数も使用する必要があります。

| クラスとローカル変数 | 説明                                                  |
| ------------------------ | ------------------------------------------------------------ |

| java.nio.ByteBuffer()    | 中間結果を格納するバッファクラスです。中間結果は、ノード間で実行のために送信される際にシリアライズまたはデシリアライズされることがあります。そのため、`serializeLength`変数を使用して、中間結果のデシリアライズに許容される長さを指定する必要があります。 |
| serializeLength()        | 中間結果のデシリアライズに許容される長さです。単位はバイトです。このローカル変数をINT型の値に設定します。例えば、`State { int counter = 0; public int serializeLength() { return 4; }}`は、中間結果がINTデータ型で、デシリアライズの長さが4バイトであることを指定しています。これらの設定はビジネス要件に基づいて調整可能です。例えば、中間結果のデータ型をLONGに指定し、デシリアライズの長さを8バイトに指定したい場合は、`State { long counter = 0; public int serializeLength() { return 8; }}`を使用します。 |

`java.nio.ByteBuffer`クラスに格納された中間結果のデシリアライズについては、以下の点に注意してください：

- `ByteBuffer`クラスに依存するremaining()メソッドは、状態をデシリアライズするために呼び出すことはできません。
- `ByteBuffer`クラスに対してclear()メソッドを呼び出すことはできません。
- `serializeLength`の値は、書き込まれたデータの長さと同じでなければなりません。そうでない場合、シリアライズとデシリアライズの間に誤った結果が生成されます。

#### UDWFのコンパイル

通常の集約関数とは異なり、UDWFは複数の行のセット（ウィンドウと総称される）に対して操作を行い、各行に対して値を返します。典型的なウィンドウ関数には、行を複数のセットに分割する`OVER`句が含まれています。それは各セットの行に対して計算を実行し、各行に対して値を返します。

`MY_WINDOW_SUM_INT`という名前のUDWFをコンパイルしたいとします。組み込みの集約関数`SUM`がBIGINT型の値を返すのに対し、`MY_WINDOW_SUM_INT`関数はリクエストパラメータと戻りパラメータのINTデータ型のみをサポートします。

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

ユーザー定義クラスは、UDAFに必要なメソッド（UDWFは特別な集約関数です）と、以下の表に記載されているwindowUpdate()メソッドを実装する必要があります。

> **注記**
>
> メソッド内のリクエストパラメータと戻りパラメータのデータ型は、[ステップ6](#step-6-create-the-udf-in-starrocks)で実行される`CREATE FUNCTION`ステートメントで宣言されたものと同じでなければならず、このトピックの「[SQLデータ型とJavaデータ型のマッピング](#mapping-between-sql-data-types-and-java-data-types)」セクションに記載されているマッピングに準拠している必要があります。

| メソッド                                                   | 説明                                                  |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| void windowUpdate(State state, int, int, int, int, ...) | ウィンドウのデータを更新します。UDWFの詳細については、「[ウィンドウ関数](../sql-functions/Window_function.md)」を参照してください。入力として行を受け取るたびに、このメソッドはウィンドウ情報を取得し、それに応じて中間結果を更新します。<ul><li>`peer_group_start`: 現在のパーティションの開始位置。`PARTITION BY`はOVER句でパーティション列を指定するために使用されます。パーティション列で同じ値を持つ行は同じパーティションにあると見なされます。</li><li>`peer_group_end`: 現在のパーティションの終了位置。</li><li>`frame_start`: 現在のウィンドウフレームの開始位置。ウィンドウフレーム句は、現在の行と、現在の行から指定された距離内の行を含む計算範囲を指定します。例えば、`ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING`は、現在の行、現在の行の前の行、および現在の行の後の行を含む計算範囲を指定します。</li><li>`frame_end`: 現在のウィンドウフレームの終了位置。</li><li>`inputs`: ウィンドウへの入力として渡されるデータ。データは特定のデータ型のみをサポートする配列パッケージです。この例では、INT値が入力として渡され、配列パッケージはInteger[]です。</li></ul> |

#### UDTFのコンパイル

UDTFは1行のデータを読み取り、テーブルと見なされる複数の値を返します。テーブル値関数は一般的に、行を列に変換するために使用されます。

> **注記**
>
> StarRocksでは、UDTFが複数の行と1列で構成されるテーブルを返すことができます。

`MY_UDF_SPLIT`という名前のUDTFをコンパイルしたいとします。`MY_UDF_SPLIT`関数では、スペースを区切り文字として使用し、STRINGデータ型のリクエストパラメータと戻りパラメータをサポートします。

```Java
package com.starrocks.udf.sample;

public class UDFSplit{
    public String[] process(String in) {
        if (in == null) return null;
        return in.split(" ");
    }
}
```

ユーザー定義クラスによって定義されたメソッドは、以下の要件を満たす必要があります。

> **注記**
>
> メソッド内のリクエストパラメータと戻りパラメータのデータ型は、[ステップ6](#step-6-create-the-udf-in-starrocks)で実行される`CREATE FUNCTION`ステートメントで宣言されたものと同じでなければならず、このトピックの「[SQLデータ型とJavaデータ型のマッピング](#mapping-between-sql-data-types-and-java-data-types)」セクションに記載されているマッピングに準拠している必要があります。

| メソッド           | 説明                         |
| ---------------- | ----------------------------------- |
| TYPE[] process() | UDTFを実行し、配列を返します。 |

### ステップ4: Javaプロジェクトのパッケージ化

以下のコマンドを実行してJavaプロジェクトをパッケージ化します：

```Bash
mvn package
```

**target**フォルダには、**udf-1.0-SNAPSHOT.jar**および**udf-1.0-SNAPSHOT-jar-with-dependencies.jar**というJARファイルが生成されます。

### ステップ5: Javaプロジェクトのアップロード

JARファイル**udf-1.0-SNAPSHOT-jar-with-dependencies.jar**を、StarRocksクラスタ内のすべてのFEおよびBEからアクセス可能で、常時稼働しているHTTPサーバーにアップロードします。その後、以下のコマンドを実行してファイルをデプロイします：

```Bash
mvn deploy 
```

Pythonを使用してシンプルなHTTPサーバーをセットアップし、そのHTTPサーバーにJARファイルをアップロードすることができます。

> **注記**
>
> [ステップ6](#step-6-create-the-udf-in-starrocks)では、FEはUDFのコードを含むJARファイルをチェックし、チェックサムを計算し、BEはJARファイルをダウンロードして実行します。

### ステップ6: StarRocksでUDFを作成する

StarRocksでは、データベース名前空間とグローバル名前空間の2種類の名前空間でUDFを作成することができます。

- UDFに可視性や分離の要件がない場合は、グローバルUDFとして作成することができます。その後、カタログ名やデータベース名を関数名の接頭辞として含めずに、関数名を使用してグローバルUDFを参照できます。
- UDFに可視性や分離の要件がある場合、または異なるデータベースで同じUDFを作成する必要がある場合は、それぞれのデータベースに個別に作成できます。そのため、セッションがターゲットデータベースに接続されている場合は、関数名を使用してUDFを参照できます。セッションがターゲットデータベース以外の別のカタログやデータベースに接続されている場合は、カタログ名とデータベース名を関数名の接頭辞として含めてUDFを参照する必要があります（例: `catalog.database.function`）。

> **注意**
>
> グローバルUDFを作成して使用する前に、システム管理者に連絡して必要な権限を付与してもらう必要があります。詳細については、[GRANT](../sql-statements/account-management/GRANT.md)を参照してください。

JARパッケージをアップロードした後、StarRocksでUDFを作成できます。グローバルUDFを作成する場合は、作成ステートメントに`GLOBAL`キーワードを含める必要があります。

#### 構文

```sql
CREATE [GLOBAL] [AGGREGATE | TABLE] FUNCTION function_name
(arg_type [, ...])
RETURNS return_type
PROPERTIES ("key" = "value" [, ...])
```

#### パラメータ

| **パラメータ**      | **必須** | **説明**                                                     |
| ------------- | -------- | ------------------------------------------------------------ |
| GLOBAL        | いいえ       | v3.0からサポートされているグローバルUDFを作成するかどうか。 |
| AGGREGATE     | いいえ       | UDAFまたはUDWFを作成するかどうか。       |
| TABLE         | いいえ       | UDTFを作成するかどうか。`AGGREGATE`と`TABLE`の両方が指定されていない場合は、スカラー関数が作成されます。               |
| function_name | はい       | 作成する関数の名前。このパラメータには、例えば`db1.my_func`のようにデータベースの名前を含めることができます。`function_name`にデータベース名が含まれている場合、UDFはそのデータベースに作成されます。それ以外の場合、UDFは現在のデータベースに作成されます。新しい関数の名前とそのパラメータは、目的のデータベースに既存の名前と同じであってはなりません。そうでないと、関数は作成できません。関数名が同じでもパラメータが異なれば、作成は成功します。|
| arg_type      | はい       | 関数の引数の型。追加された引数は`[, ...]`で表すことができます。サポートされるデータ型については、[SQLデータ型とJavaデータ型のマッピング](#mapping-between-sql-data-types-and-java-data-types)を参照してください。|
| return_type   | はい       | 関数の戻り値の型。サポートされるデータ型については、[Java UDF](#mapping-between-sql-data-types-and-java-data-types)を参照してください。 |
| PROPERTIES    | はい       | 作成するUDFのタイプに応じて異なる関数のプロパティ。 |

#### スカラーUDFの作成

次のコマンドを実行して、前の例でコンパイルしたスカラーUDFを作成します。

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
| type      | UDFのタイプ。値を`StarrocksJar`に設定すると、UDFがJavaベースの関数であることを指定します。 |
| file      | UDFのコードを含むJARファイルをダウンロードするHTTP URL。このパラメータの値は`http://<http_server_ip>:<http_server_port>/<jar_package_name>`形式です。 |

#### UDAFの作成

次のコマンドを実行して、前の例でコンパイルしたUDAFを作成します。

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

PROPERTIESのパラメータの説明は、[スカラーUDFの作成](#create-a-scalar-udf)と同じです。

#### UDWFの作成

次のコマンドを実行して、前の例でコンパイルしたUDWFを作成します。

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

`analytic`: UDFがウィンドウ関数であるかどうか。値を`true`に設定します。その他のプロパティの説明は、[スカラーUDFの作成](#create-a-scalar-udf)と同じです。

#### UDTFの作成

次のコマンドを実行して、前の例でコンパイルしたUDTFを作成します。

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

PROPERTIESのパラメータの説明は、[スカラーUDFの作成](#create-a-scalar-udf)と同じです。

### ステップ7: UDFの使用

UDFを作成したら、ビジネスニーズに基づいてテストし、使用することができます。

#### スカラーUDFの使用

次のコマンドを実行して、前の例で作成したスカラーUDFを使用します。

```SQL
SELECT MY_UDF_JSON_GET('{"key":"{\\"in\\":2}"}', '$.key.in');
```

#### UDAFの使用

次のコマンドを実行して、前の例で作成したUDAFを使用します。

```SQL
SELECT MY_SUM_INT(col1);
```

#### UDWFの使用

次のコマンドを実行して、前の例で作成したUDWFを使用します。

```SQL
SELECT MY_WINDOW_SUM_INT(intcol) 
            OVER (PARTITION BY intcol2
                  ORDER BY intcol3
                  ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
FROM test_basic;
```

#### UDTFの使用

次のコマンドを実行して、前の例で作成したUDTFを使用します。

```Plain
-- t1という名前のテーブルがあり、そのカラムa、b、c1についての情報は以下の通りです:
SELECT t1.a,t1.b,t1.c1 FROM t1;
> 出力:
1,2.1,"hello world"
2,2.2,"hello UDTF."

-- MY_UDF_SPLIT()関数を実行します。
SELECT t1.a,t1.b, MY_UDF_SPLIT(t1.c1) FROM t1, MY_UDF_SPLIT(t1.c1); 
> 出力:
1,2.1,"hello"
1,2.1,"world"
2,2.2,"hello"
2,2.2,"UDTF."
```

> **注記**
>
> - 上記のコードスニペットの最初の`MY_UDF_SPLIT`は、関数としての2番目の`MY_UDF_SPLIT`によって返される列のエイリアスです。
> - `AS t2(f1)`を使用して返されるテーブルとそのカラムのエイリアスを指定することはできません。

## UDFの表示

次のコマンドを実行してUDFをクエリします。

```SQL
SHOW [GLOBAL] FUNCTIONS;
```

詳細については、[SHOW FUNCTIONS](../sql-statements/data-definition/SHOW_FUNCTIONS.md)を参照してください。

## UDFの削除

次のコマンドを実行してUDFを削除します。

```SQL
DROP [GLOBAL] FUNCTION <function_name>(arg_type [, ...]);
```

詳細については、[DROP FUNCTION](../sql-statements/data-definition/DROP_FUNCTION.md)を参照してください。

## SQLデータ型とJavaデータ型のマッピング

| SQL型       | Java型         |
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

StarRocks クラスタ内の各 Java 仮想マシン (JVM) の **be/conf/hadoop_env.sh** ファイルで次の環境変数を設定して、メモリ使用量を制御します。また、ファイル内の他のパラメータを設定することもできます。

```Bash
export LIBHDFS_OPTS="-Xloggc:$STARROCKS_HOME/log/be.gc.log -server"
```

## FAQ

UDFを作成する際に静的変数を使用することは可能ですか？異なるUDFの静的変数には相互影響はありますか？

はい、UDFをコンパイルする際に静的変数を使用することができます。異なるUDFの静的変数は互いに独立しており、たとえUDFが同名のクラスを持っていたとしても、互いに影響を与えることはありません。
