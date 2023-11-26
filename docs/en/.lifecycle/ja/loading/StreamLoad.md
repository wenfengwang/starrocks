---
displayed_sidebar: "Japanese"
---

# ローカルファイルシステムからデータをロードする

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksでは、ローカルファイルシステムからデータをロードするための2つの方法が提供されています。

- [ストリームロード](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を使用した同期ロード
- [ブローカーロード](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を使用した非同期ロード

<InsertPrivNote />

それぞれのオプションには、それぞれの利点があります。

- ストリームロードは、CSVおよびJSONファイル形式をサポートしています。この方法は、個々のファイルのサイズが10 GBを超えない小数のファイルからデータをロードする場合に推奨されます。
- ブローカーロードは、Parquet、ORC、およびCSVファイル形式をサポートしています。この方法は、個々のファイルのサイズが10 GBを超えるか、ファイルがネットワーク接続されたストレージ（NAS）デバイスに保存されている場合に推奨されます。**この機能はv2.5以降でサポートされています。この方法を選択する場合、データファイルが配置されているマシンに[ブローカーをデプロイ](../deployment/deploy_broker.md)する必要があることに注意してください。**

CSVデータの場合、次のポイントに注意してください。

- テキスト区切り文字として、コンマ（,）、タブ、またはパイプ（|）などのUTF-8文字列を使用できます。ただし、その長さは50バイトを超えてはなりません。
- ヌル値は`\N`を使用して示します。たとえば、データファイルは3つの列から構成されており、そのデータファイルのレコードは最初の列と3番目の列にデータを保持していますが、2番目の列にはデータがありません。この場合、ヌル値を示すために2番目の列に`\N`を使用する必要があります。これは、レコードを`a,\N,b`ではなく`a,,b`とコンパイルする必要があることを意味します。`a,,b`は、レコードの2番目の列に空の文字列が含まれていることを示します。

ストリームロードとブローカーロードは、データのロード時にデータ変換をサポートし、データロード中のUPSERTおよびDELETE操作によるデータの変更もサポートしています。詳細については、[ロード時のデータ変換](../loading/Etl_in_loading.md)および[ロードを介したデータの変更](../loading/Load_to_Primary_Key_tables.md)を参照してください。

## ストリームロードを使用したローカルファイルシステムからのロード

ストリームロードは、HTTP PUTに基づいて同期ロードモードで実行されます。ロードジョブを送信すると、StarRocksはジョブを同期的に実行し、ジョブが完了した後にジョブの結果を返します。ジョブの結果に基づいてジョブが成功したかどうかを判断することができます。

> **注意**
>
> Stream Loadを使用してStarRocksテーブルにデータをロードした後、そのテーブルに作成されたマテリアライズドビューのデータも更新されます。

### 動作原理

クライアント上でHTTPに基づいてロードリクエストをFEに送信し、FEはロードリクエストを特定のBEに転送するためにHTTPリダイレクトを使用します。また、クライアント上で直接ロードリクエストを選択したBEに送信することもできます。

:::note

FEにロードリクエストを送信する場合、FEはポーリングメカニズムを使用して、どのBEがコーディネータとしてロードリクエストを受け取り、処理するかを決定します。ポーリングメカニズムにより、StarRocksクラスタ内での負荷分散が実現されます。そのため、ロードリクエストをFEに送信することをお勧めします。

:::

ロードリクエストを受け取ったBEは、使用されるスキーマに基づいてデータを分割し、データの各部分を他の関連するBEに割り当てます。ロードが完了すると、コーディネータBEはロードジョブの結果をクライアントに返します。ただし、ロード中にコーディネータBEを停止すると、ロードジョブは失敗します。

次の図は、ストリームロードジョブのワークフローを示しています。

![ストリームロードのワークフロー](../assets/4.2-1.png)

### 制限事項

ストリームロードは、JSON形式の列を含むCSVファイルのデータをロードすることはサポートしていません。

### 典型的な例

このセクションでは、curlを使用して、ローカルファイルシステムからCSVまたはJSONファイルのデータをStarRocksにロードする方法を説明します。詳細な構文とパラメータの説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

StarRocksでは、SQL言語によって予約されたキーワードとしていくつかのリテラルが使用されています。SQLステートメントでこれらのキーワードを直接使用しないでください。SQLステートメントでこのようなキーワードを使用する場合は、バッククォート（`）で囲んでください。[キーワード](../sql-reference/sql-statements/keywords.md)を参照してください。

#### CSVデータのロード

##### データセットの準備

ローカルファイルシステムで、`example1.csv`という名前のCSVファイルを作成します。このファイルは、ユーザーID、ユーザー名、およびユーザースコアを順に表す3つの列で構成されています。

```Plain
1,Lily,23
2,Rose,23
3,Alice,24
4,Julia,25
```

##### データベースとテーブルの作成

データベースを作成し、それに切り替えます。

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

`table1`という名前のプライマリキーテーブルを作成します。このテーブルは、`id`、`name`、および`score`の3つの列で構成されており、`id`がプライマリキーです。

```SQL
CREATE TABLE `table1`
(
    `id` int(11) NOT NULL COMMENT "ユーザーID",
    `name` varchar(65533) NULL COMMENT "ユーザー名",
    `score` int(11) NOT NULL COMMENT "ユーザースコア"
)
ENGINE=OLAP
PRIMARY KEY(`id`)
DISTRIBUTED BY HASH(`id`);
```

:::note

v2.5.7以降、StarRocksはテーブルまたはパーティションを作成するときにバケットの数（BUCKETS）を自動的に設定できます。バケットの数を手動で設定する必要はありません。詳細については、「[バケットの数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets)」を参照してください。

:::

##### ストリームロードの開始

次のコマンドを実行して、`example1.csv`のデータを`table1`にロードします。

```Bash
curl --location-trusted -u <username>:<password> -H "label:123" \
    -H "Expect:100-continue" \
    -H "column_separator:," \
    -H "Expect:100-continue" \
    -H "columns: id, name, score" \
    -T example1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/mydatabase/table1/_stream_load
```

:::note

- パスワードが設定されていないアカウントを使用する場合は、`<username>:`のみを入力する必要があります。
- [SHOW FRONTENDS](../sql-reference/sql-statements/Administration/SHOW_FRONTENDS.md)を使用して、FEノードのIPアドレスとHTTPポートを表示できます。

:::

`example1.csv`は、コンマ（,）で区切られた3つの列で構成されており、これらの列は`table1`の`id`、`name`、および`score`の3つの列に順番にマッピングされます。したがって、`column_separator`パラメータを使用してコンマ（,）を列の区切り文字として指定する必要があります。また、`columns`パラメータを使用して、`example1.csv`の3つの列を一時的に`id`、`name`、および`score`という名前で指定する必要があります。これらの列は、`table1`の3つの列に順番にマッピングされます。

ロードが完了したら、`table1`をクエリしてロードが成功したかどうかを確認できます。

```SQL
SELECT * FROM table1;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Lily  |    23 |
|    2 | Rose  |    23 |
|    3 | Alice |    24 |
|    4 | Julia |    25 |
+------+-------+-------+
4 rows in set (0.00 sec)
```

#### JSONデータのロード

##### データセットの準備

ローカルファイルシステムで、`example2.json`という名前のJSONファイルを作成します。このファイルは、都市IDと都市名を順に表す2つの列で構成されています。

```JSON
{"name": "Beijing", "code": 2}
```

##### データベースとテーブルの作成

データベースを作成し、それに切り替えます。

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

`table2`という名前のプライマリキーテーブルを作成します。このテーブルは、`id`と`city`の2つの列で構成されており、`id`がプライマリキーです。

```SQL
CREATE TABLE `table2`
(
    `id` int(11) NOT NULL COMMENT "都市ID",
    `city` varchar(65533) NULL COMMENT "都市名"
)
ENGINE=OLAP
PRIMARY KEY(`id`)
DISTRIBUTED BY HASH(`id`);
```

:::note

v2.5.7以降、StarRocksはテーブルまたはパーティションを作成するときにバケットの数（BUCKETS）を自動的に設定できます。バケットの数を手動で設定する必要はありません。詳細については、「[バケットの数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets)」を参照してください。

:::

##### ストリームロードの開始

次のコマンドを実行して、`example2.json`のデータを`table2`にロードします。

```Bash
curl -v --location-trusted -u <username>:<password> -H "strict_mode: true" \
    -H "Expect:100-continue" \
    -H "format: json" -H "jsonpaths: [\"$.name\", \"$.code\"]" \
    -H "columns: city,tmp_id, id = tmp_id * 100" \
    -T example2.json -XPUT \
    http://<fe_host>:<fe_http_port>/api/mydatabase/table2/_stream_load
```

:::note

- パスワードが設定されていないアカウントを使用する場合は、`<username>:`のみを入力する必要があります。
- [SHOW FRONTENDS](../sql-reference/sql-statements/Administration/SHOW_FRONTENDS.md)を使用して、FEノードのIPアドレスとHTTPポートを表示できます。

:::

`example2.json`は、`name`と`code`の2つのキーで構成されており、これらのキーは`table2`の`id`と`city`の列にマッピングされます。マッピングの詳細は以下の図に示されています。

![JSON - 列のマッピング](../assets/4.2-2.png)

前の図のマッピングは次のように説明されます。

- StarRocksは、`example2.json`の`name`と`code`キーを抽出し、`jsonpaths`パラメータで宣言された`name`と`code`フィールドにマッピングします。

- StarRocksは、`columns`パラメータで宣言された`name`と`code`フィールドを抽出し、`city`と`tmp_id`フィールドにマッピングします。

- StarRocksは、`columns`パラメータで宣言された`city`と`tmp_id`フィールドを抽出し、`table2`の`city`と`id`の列に名前でマッピングします。

:::note

前の例では、`example2.json`の`code`の値をロードする前に100倍にすることを示しています。

:::

`jsonpaths`、`columns`、およびStarRocksテーブルの列との間の詳細なマッピングについては、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)の「列のマッピング」セクションを参照してください。

ロードが完了したら、`table2`をクエリしてロードが成功したかどうかを確認できます。

```SQL
SELECT * FROM table2;
+------+--------+
| id   | city   |
+------+--------+
| 200  | Beijing|
+------+--------+
4 rows in set (0.01 sec)
```

#### ストリームロードの進捗状況の確認

ロードジョブが完了すると、StarRocksはジョブの結果をJSON形式で返します。詳細については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)の「戻り値」セクションを参照してください。

ストリームロードでは、LOADステートメントを使用してロードジョブの結果をクエリすることはできません。

#### ストリームロードジョブのキャンセル

ストリームロードでは、ロードジョブをキャンセルすることはできません。ロードジョブがタイムアウトしたりエラーが発生した場合、StarRocksは自動的にジョブをキャンセルします。

### パラメータの設定

このセクションでは、ロード方法としてストリームロードを選択した場合に設定する必要があるいくつかのシステムパラメータについて説明します。これらのパラメータの設定は、すべてのストリームロードジョブに影響します。

- `streaming_load_max_mb`：ロードするデータファイルの最大サイズです。デフォルトの最大サイズは10 GBです。詳細については、「[BEの動的パラメータの設定](../administration/Configuration.md#configure-be-dynamic-parameters)」を参照してください。
  
  一度にロードするデータのサイズは10 GBを超えないようにすることをお勧めします。データファイルのサイズが10 GBを超える場合は、10 GB未満の小さなファイルにデータファイルを分割して、これらのファイルを1つずつロードすることをお勧めします。10 GBを超えるデータファイルを分割できない場合は、ファイルサイズに基づいてこのパラメータの値を増やすことができます。

  このパラメータの値を増やした後、新しい値はStarRocksクラスタのBEを再起動するまで有効になりません。また、システムのパフォーマンスが低下し、ロードの失敗時のリトライのコストも増加します。

  :::note
  
  JSONファイルのデータをロードする場合は、次の点に注意してください。
  
  - ファイル内の各JSONオブジェクトのサイズは4 GBを超えてはなりません。ファイル内の任意のJSONオブジェクトが4 GBを超える場合、StarRocksは「This parser can't support a document that big.」というエラーをスローします。
  
  - デフォルトでは、HTTPリクエストのJSONボディのサイズは100 MBを超えることはできません。JSONボディが100 MBを超える場合、StarRocksは「The size of this batch exceed the max size [104857600] of json type data data [8617627793]. Set ignore_json_size to skip check, although it may lead huge memory consuming.」というエラーをスローします。このエラーを防ぐためには、HTTPリクエストヘッダに`"ignore_json_size:true"`を追加して、JSONボディのサイズのチェックを無視することができます。

  :::

- `stream_load_default_timeout_second`：各ロードジョブのタイムアウト期間です。デフォルトのタイムアウト期間は600秒です。詳細については、「[FEの動的パラメータの設定](../administration/Configuration.md#configure-fe-dynamic-parameters)」を参照してください。
  
  多くのロードジョブがタイムアウトする場合は、次の式から計算結果に基づいてこのパラメータの値を増やすことができます。

  **各ロードジョブのタイムアウト期間 > ロードするデータ量/平均ロード速度**

  たとえば、ロードするデータファイルのサイズが10 GBで、StarRocksクラスタの平均ロード速度が100 MB/sの場合、タイムアウト期間を100秒以上に設定します。

  :::note
  
  前の式の**平均ロード速度**は、StarRocksクラスタの平均ロード速度です。これはディスクI/OとStarRocksクラスタのBEの数によって異なります。

  :::

  ストリームロードは、ロードジョブごとにタイムアウト期間を指定する`timeout`パラメータも提供しています。詳細については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

### 使用上の注意

データファイルをロードする際に、レコードのデータファイルに存在しないフィールドがあり、StarRocksテーブルのマッピング列が`NOT NULL`として定義されている場合、StarRocksはレコードのマッピング列に`NULL`値を自動的に埋めます。また、`ifnull()`関数を使用して埋めるデフォルト値を指定することもできます。

たとえば、前の`example2.json`ファイルの都市IDを表すフィールドが欠落しており、マッピング列に`table2`のマッピング列に`x`の値を埋めたい場合は、`"columns: city, tmp_id, id = ifnull(tmp_id, 'x')"`と指定できます。

## ブローカーロードを使用したローカルファイルシステムからのロード

ストリームロードに加えて、ブローカーロードを使用してローカルファイルシステムからデータをロードすることもできます。この機能はv2.5以降でサポートされています。

ブローカーロードは非同期のロード方法です。ロードジョブを送信すると、StarRocksはジョブを非同期に実行し、ジョブの結果をすぐに返しません。ジョブの結果を手動でクエリする必要があります。[ブローカーロードの進捗状況の確認](#ブローカーロードの進捗状況の確認)を参照してください。

### 制限事項

- 現在、ブローカーロードは、v2.5以降のバージョンを持つ単一のブローカーを介してのみローカルファイルシステムからのロードをサポートしています。
- 単一のブローカーに対する高並行のクエリは、タイムアウトやOOMなどの問題を引き起こす可能性があります。影響を軽減するためには、ブローカーロードのクエリ並列度を設定するために`pipeline_dop`変数（[システム変数](../reference/System_variable.md#pipeline_dop)を参照）を使用できます。単一のブローカーに対するクエリでは、`pipeline_dop`を`16`より小さい値に設定することをお勧めします。

### 開始前の準備

ローカルファイルシステムからデータをブローカーロードを使用してロードするには、次の準備を完了する必要があります。

1. [デプロイの前提条件](../deployment/deployment_prerequisites.md)、[環境設定の確認](../deployment/environment_configurations.md)、および[デプロイファイルの準備](../deployment/prepare_deployment_files.md)の指示に従って、ローカルファイルが配置されているマシンを構成し、そのマシンにブローカーをデプロイします。操作の詳細については、[ブローカーノードのデプロイと管理](../deployment/deploy_broker.md)を参照してください。

   > **注意**
   >
   > 単一のブローカーのみをデプロイし、ブローカーのバージョンがv2.5以降であることを確認してください。

2. [ALTER SYSTEM](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md#broker)を実行して、前の手順でデプロイしたブローカーをStarRocksクラスタに追加し、ブローカーに新しい名前を定義します。次の例では、ブローカー`172.26.199.40:8000`をStarRocksクラスタに追加し、ブローカーの名前を`sole_broker`と定義しています。

   ```SQL
   ALTER SYSTEM ADD BROKER sole_broker "172.26.199.40:8000";
   ```

### 典型的な例

ブローカーロードは、単一のデータファイルから単一のテーブルにデータをロードする場合、複数のデータファイルから単一のテーブルにデータをロードする場合、および複数のデータファイルから複数のテーブルにデータをロードする場合の3つの方法をサポートしています。このセクションでは、複数のデータファイルから単一のテーブルにデータをロードする方法を例として説明します。

StarRocksでは、SQL言語によって予約されたキーワードとしていくつかのリテラルが使用されています。SQLステートメントでこれらのキーワードを直接使用しないでください。SQLステートメントでこのようなキーワードを使用する場合は、バッククォート（`）で囲んでください。[キーワード](../sql-reference/sql-statements/keywords.md)を参照してください。

#### データセットの準備

CSVファイル形式を例に使用します。ローカルファイルシステムにログインし、特定のストレージ場所（たとえば、`/user/starrocks/`）に2つのCSVファイル、`file1.csv`および`file2.csv`を作成します。両方のファイルは、ユーザーID、ユーザー名、およびユーザースコアを順に表す3つの列で構成されています。

- `file1.csv`

  ```Plain
  1,Lily,21
  2,Rose,22
  3,Alice,23
  4,Julia,24
  ```

- `file2.csv`

  ```Plain
  5,Tony,25
  6,Adam,26
  7,Allen,27
  8,Jacky,28
  ```

#### データベースとテーブルの作成

データベースを作成し、それに切り替えます。

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

`mytable`という名前のプライマリキーテーブルを作成します。このテーブルは、`id`、`name`、および`score`の3つの列で構成されており、`id`がプライマリキーです。

```SQL
CREATE TABLE `mytable`
(
    `id` int(11) NOT NULL COMMENT "User ID",
    `name` varchar(65533) NULL DEFAULT "" COMMENT "User name",
    `score` int(11) NOT NULL DEFAULT "0" COMMENT "User score"
)
ENGINE=OLAP
PRIMARY KEY(`id`)
DISTRIBUTED BY HASH(`id`);
```

#### ブローカーロードの開始

次のコマンドを実行して、ローカルファイルシステムの`/user/starrocks/`パスに格納されているすべてのデータファイル（`file1.csv`および`file2.csv`）からStarRocksテーブル`mytable`にデータをロードするブローカーロードジョブを開始します。

```SQL
LOAD LABEL mydatabase.label_local
(
    DATA INFILE("file:///home/disk1/business/csv/*")
    INTO TABLE mytable
    COLUMNS TERMINATED BY ","
    (id, name, score)
)
WITH BROKER "sole_broker"
PROPERTIES
(
    "timeout" = "3600"
);
```

このジョブには、4つの主要なセクションがあります。

- `LABEL`：ロードジョブの状態をクエリする際に使用する文字列です。
- `LOAD`宣言：ソースURI、ソースデータ形式、および宛先テーブル名です。
- `BROKER`：ブローカーの名前です。
- `PROPERTIES`：タイムアウト値およびロードジョブに適用するその他のプロパティです。

詳細な構文とパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

#### ブローカーロードの進捗状況の確認

v3.0以前では、[SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)ステートメントまたはcurlコマンドを使用して、ブローカーロードジョブの進捗状況を表示できます。

v3.1以降では、[`information_schema.loads`](../reference/information_schema/loads.md)ビューからブローカーロードジョブの進捗状況を表示できます。

```SQL
SELECT * FROMinformation_schema.loads;
```

複数のロードジョブを送信した場合は、ジョブに関連付けられた`LABEL`でフィルタリングできます。例：

```SQL
SELECT * FROMinformation_schema.loadsWHERELABEL= 'label_local';
```

ロードジョブが完了したことを確認したら、テーブルをクエリしてデータが正常にロードされたかどうかを確認できます。例：

```SQL
SELECT * FROM mytable;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    3 | Alice |    23 |
|    5 | Tony  |    25 |
|    6 | Adam  |    26 |
|    1 | Lily  |    21 |
|    2 | Rose  |    22 |
|    4 | Julia |    24 |
|    7 | Allen |    27 |
|    8 | Jacky |    28 |
+------+-------+-------+
8 rows in set (0.07 sec)
```

#### ブローカーロードジョブのキャンセル

ロードジョブが**CANCELLED**または**FINISHED**のステージにない場合、[CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md)ステートメントを使用してジョブをキャンセルできます。

たとえば、次のステートメントを実行して、データベース`mydatabase`の中でラベルが`label_local`のロードジョブをキャンセルできます。

```SQL
CANCEL LOAD
FROM mydatabase
WHERE LABEL = "label_local";
```

## NAS経由でのブローカーロード

ブローカーロードを使用してNASからデータをロードする方法は2つあります。

- NASをローカルファイルシステムとして扱い、ブローカーを使用してロードジョブを実行します。前のセクション「[ローカルファイルシステムからのロード](#ローカルファイルシステムからのロード)」を参照してください。
- （推奨）NASをクラウドストレージシステムとして扱い、ブローカーを使用せずにロードジョブを実行します。

このセクションでは、2番目の方法について説明します。具体的な操作は次のとおりです。

1. NASデバイスをStarRocksクラスタのすべてのBEノードとFEノードに同じパスにマウントします。これにより、すべてのBEが自分自身のローカルに保存されているファイルと同じようにNASデバイスにアクセスできるようになります。

2. ブローカーロードを使用して、NASデバイスからデータをStarRocksテーブルにロードします。例：

   ```SQL
   LOAD LABEL test_db.label_nas
   (
       DATA INFILE("file:///home/disk1/sr/*")
       INTO TABLE mytable
       COLUMNS TERMINATED BY ","
   )
   WITH BROKER
   PROPERTIES
   (
       "timeout" = "3600"
   );
   ```

   このジョブには、4つの主要なセクションがあります。

   - `LABEL`：ロードジョブの状態をクエリする際に使用する文字列です。
   - `LOAD`宣言：ソースURI、ソースデータ形式、および宛先テーブル名です。注意点として、宣言の`DATA INFILE`は、NASデバイスのマウントポイントフォルダパスを指定するために使用されます。上記の例では、`file:///`が接頭辞であり、`/home/disk1/sr`がマウントポイントフォルダパスです。
   - `BROKER`：ブローカーの名前を指定する必要はありません。
   - `PROPERTIES`：タイムアウト値およびロードジョブに適用するその他のプロパティです。

   詳細な構文とパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

ジョブを送信した後、必要に応じてロードの進捗状況を表示したり、ジョブをキャンセルしたりすることができます。詳細な操作については、「[ブローカーロードの進捗状況の確認](#ブローカーロードの進捗状況の確認)」および「[ブローカーロードジョブのキャンセル](#ブローカーロードジョブのキャンセル)」を参照してください。
