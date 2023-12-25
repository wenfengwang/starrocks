---
displayed_sidebar: English
---

# ローカルファイルシステムからのデータロード

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksは、ローカルファイルシステムからデータをロードする2つの方法を提供します：

- [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を使用した同期ロード
- [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を使用した非同期ロード

これらのオプションには、それぞれ独自の利点があります：

- Stream LoadはCSVとJSONファイル形式をサポートしています。この方法は、個々のサイズが10GBを超えない少数のファイルからデータをロードしたい場合に推奨されます。
- Broker LoadはParquet、ORC、およびCSVファイル形式をサポートしています。この方法は、個々のサイズが10GBを超える大量のファイルからデータをロードする場合、またはファイルがネットワークアタッチドストレージ（NAS）デバイスに保存されている場合に推奨されます。**この機能はv2.5以降でサポートされています。この方法を選択する場合は、データファイルが配置されているマシンに[ブローカーをデプロイ](../deployment/deploy_broker.md)する必要があることに注意してください。**

CSVデータの場合、以下の点に注意してください：

- コンマ（,）、タブ、パイプ（|）など、長さが50バイトを超えないUTF-8文字列をテキスト区切り文字として使用できます。
- Null値は`\N`を使用して表されます。例えば、データファイルが3つの列で構成されており、そのデータファイルのレコードが1列目と3列目にデータを保持しているが2列目にはデータがない場合、2列目で`\N`を使用してnull値を示す必要があります。つまり、レコードは`a,\N,b`としてコンパイルされる必要があり、`a,,b`ではない。`a,,b`はレコードの2列目が空文字列を保持していることを示します。

Stream LoadとBroker Loadはどちらもデータロード時のデータ変換をサポートし、データロード中にUPSERTおよびDELETE操作によるデータ変更をサポートします。詳細については、[ロード時のデータ変換](../loading/Etl_in_loading.md)および[ロードを通じたデータ変更](../loading/Load_to_Primary_Key_tables.md)を参照してください。

## 始める前に

### 権限の確認

<InsertPrivNote />

## Stream Loadを通じたローカルファイルシステムからのロード

Stream LoadはHTTP PUTベースの同期ロードメソッドです。ロードジョブを送信した後、StarRocksはジョブを同期的に実行し、ジョブが完了した後にジョブの結果を返します。ジョブの結果に基づいて、ジョブが成功したかどうかを判断できます。

> **注意**
>
> Stream Loadを使用してStarRocksテーブルにデータをロードした後、そのテーブルに作成されたマテリアライズドビューのデータも更新されます。

### 仕組み

HTTPに従ってクライアントからFEにロードリクエストを送信すると、FEはHTTPリダイレクトを使用してロードリクエストを特定のBEに転送します。または、クライアントから直接選択したBEにロードリクエストを送信することもできます。

:::note

FEにロードリクエストを送信すると、FEはポーリングメカニズムを使用して、ロードリクエストを受信して処理するコーディネーターBEを決定します。ポーリングメカニズムは、StarRocksクラスタ内の負荷分散を実現するのに役立ちます。したがって、FEにロードリクエストを送信することを推奨します。

:::

ロードリクエストを受信したBEはコーディネーターBEとして動作し、使用されるスキーマに基づいてデータを分割し、各部分を関与する他のBEに割り当てます。ロードが完了すると、コーディネーターBEはロードジョブの結果をクライアントに返します。ロード中にコーディネーターBEを停止すると、ロードジョブは失敗します。

以下の図は、Stream Loadジョブのワークフローを示しています。

![Stream Loadのワークフロー](../assets/4.2-1.png)

### 制限

Stream Loadは、JSON形式の列を含むCSVファイルのデータをロードすることはサポートしていません。

### 典型的な例

このセクションでは、curlを例にして、CSVまたはJSONファイルのデータをローカルファイルシステムからStarRocksにロードする方法を説明します。詳細な構文とパラメーターの説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

StarRocksでは、一部のリテラルがSQL言語によって予約キーワードとして使用されます。これらのキーワードをSQLステートメントで直接使用しないでください。SQLステートメントでこのようなキーワードを使用したい場合は、バッククォート(`)で囲んでください。[キーワード](../sql-reference/sql-statements/keywords.md)を参照してください。

#### CSVデータのロード

##### データセットの準備

ローカルファイルシステムに`example1.csv`という名前のCSVファイルを作成します。このファイルは、ユーザーID、ユーザー名、ユーザースコアを順に表す3つの列で構成されています。

```Plain
1,Lily,23
2,Rose,23
3,Alice,24
4,Julia,25
```

##### データベースとテーブルの作成

データベースを作成し、それに切り替えます：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

`table1`という名前のプライマリキーテーブルを作成します。このテーブルは、`id`、`name`、`score`の3つの列で構成され、`id`がプライマリキーです。

```SQL
CREATE TABLE `table1`
(
    `id` int(11) NOT NULL COMMENT "user ID",
    `name` varchar(65533) NULL COMMENT "user name",
    `score` int(11) NOT NULL COMMENT "user score"
)
ENGINE=OLAP
PRIMARY KEY(`id`)
DISTRIBUTED BY HASH(`id`) BUCKETS 10;
```

:::note

v2.5.7以降、StarRocksはテーブルを作成する際やパーティションを追加する際に、バケット数（BUCKETS）を自動的に設定することができます。バケット数を手動で設定する必要はもうありません。詳細については、[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

:::

##### Stream Loadの開始

次のコマンドを実行して、`example1.csv`のデータを`table1`にロードします：

```Bash
curl --location-trusted -u <username>:<password> -H "label:123" \
    -H "Expect:100-continue" \
    -H "column_separator:," \
    -H "columns: id, name, score" \
    -T example1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/mydatabase/table1/_stream_load
```

:::note

- パスワードが設定されていないアカウントを使用する場合は、`<username>:`のみを入力します。
- [SHOW FRONTENDS](../sql-reference/sql-statements/Administration/SHOW_FRONTENDS.md)を使用してFEノードのIPアドレスとHTTPポートを表示できます。

:::

`example1.csv`は、カンマ（,）で区切られた3つの列で構成され、`id`、`name`、`score`の列に順にマッピングできます。したがって、`column_separator`パラメータを使用して、列区切り文字としてカンマ（,）を指定する必要があります。また、`columns`パラメータを使用して、`example1.csv`の3つの列に一時的に`id`、`name`、`score`という名前を付け、`table1`の3つの列に順にマッピングする必要があります。

ロードが完了したら、`table1`をクエリしてロードが成功したことを確認できます：

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

ローカルファイルシステムに`example2.json`という名前のJSONファイルを作成します。このファイルは、都市IDと都市名を順に表す2つの列で構成されています。

```JSON
{"name": "Beijing", "code": 2}
```

##### データベースとテーブルの作成

データベースを作成し、それに切り替えます：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

`table2`という名前のプライマリキーを持つテーブルを作成します。このテーブルは2つのカラム、`id`と`city`で構成され、`id`がプライマリキーです。

```SQL
CREATE TABLE `table2`
(
    `id` int(11) NOT NULL COMMENT "city ID",
    `city` varchar(65533) NULL COMMENT "city name"
)
ENGINE=OLAP
PRIMARY KEY(`id`)
DISTRIBUTED BY HASH(`id`);
```

:::note

v2.5.7以降、StarRocksはテーブル作成時やパーティション追加時に自動的にバケット数(BUCKETS)を設定できるようになりました。手動でバケット数を設定する必要はありません。詳細情報については、[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

:::

##### ストリームロードを開始する

次のコマンドを実行して、`example2.json`のデータを`table2`にロードします：

```Bash
curl -v --location-trusted -u <username>:<password> -H "strict_mode: true" \
    -H "Expect:100-continue" \
    -H "format: json" -H "jsonpaths: [\"$.name\", \"$.code\"]" \
    -H "columns: city,tmp_id, id = tmp_id * 100" \
    -T example2.json -XPUT \
    http://<fe_host>:<fe_http_port>/api/mydatabase/table2/_stream_load
```

:::note

- パスワードが設定されていないアカウントを使用する場合は、`<username>:`のみを入力します。
- [SHOW FRONTENDS](../sql-reference/sql-statements/Administration/SHOW_FRONTENDS.md)を使用して、FEノードのIPアドレスとHTTPポートを確認できます。

:::

`example2.json`は`name`と`code`の2つのキーで構成されており、それぞれ`table2`の`id`と`city`カラムにマッピングされます。以下の図に示されています。

![JSON - Column Mapping](../assets/4.2-2.png)

上記の図に示されたマッピングは以下の通りです：

- StarRocksは`example2.json`の`name`と`code`キーを抽出し、`jsonpaths`パラメータで宣言された`name`と`code`フィールドにマッピングします。

- StarRocksは`jsonpaths`パラメータで宣言された`name`と`code`フィールドを抽出し、`columns`パラメータで宣言された`city`と`tmp_id`フィールドに**順番に**マッピングします。

- StarRocksは`columns`パラメータで宣言された`city`と`tmp_id`フィールドを抽出し、`table2`の`city`と`id`カラムに**名前で**マッピングします。

:::note

上記の例では、`example2.json`の`code`の値を100倍してから`table2`の`id`カラムにロードします。

:::

`jsonpaths`、`columns`、StarRocksテーブルのカラム間の詳細なマッピングについては、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)の「カラムマッピング」セクションを参照してください。

ロードが完了したら、`table2`をクエリしてロードが成功したかを確認できます：

```SQL
SELECT * FROM table2;
+------+--------+
| id   | city   |
+------+--------+
| 200  | Beijing|
+------+--------+
4 rows in set (0.01 sec)
```

#### ストリームロードの進捗を確認する

ロードジョブが完了すると、StarRocksはジョブの結果をJSON形式で返します。詳細については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)の「戻り値」セクションを参照してください。

ストリームロードでは、SHOW LOADステートメントを使用してロードジョブの結果を照会することはできません。

#### ストリームロードジョブをキャンセルする

ストリームロードでは、ロードジョブをキャンセルすることはできません。ロードジョブがタイムアウトするかエラーに遭遇した場合、StarRocksは自動的にジョブをキャンセルします。

### パラメータ設定

このセクションでは、ロード方法としてストリームロードを選択した場合に設定する必要があるいくつかのシステムパラメータについて説明します。これらのパラメータ設定は、すべてのストリームロードジョブに影響します。

- `streaming_load_max_mb`: ロードしたい各データファイルの最大サイズです。デフォルトの最大サイズは10GBです。詳細については、[BE動的パラメータの設定](../administration/BE_configuration.md#configure-be-dynamic-parameters)を参照してください。
  
  一度に10GB以上のデータをロードしないことを推奨します。データファイルのサイズが10GBを超える場合は、データファイルを10GB未満の小さなファイルに分割し、それらを順番にロードすることをお勧めします。10GBを超えるデータファイルを分割できない場合は、ファイルサイズに基づいてこのパラメータの値を増やすことができます。

  このパラメータの値を増やした後、新しい値はStarRocksクラスタのBEを再起動した後にのみ有効になります。さらに、システムパフォーマンスが低下する可能性があり、ロード失敗時のリトライコストも増加します。

  :::note
  
  JSONファイルのデータをロードする際には、以下の点に注意してください：
  
  - ファイル内の各JSONオブジェクトのサイズは4GBを超えることはできません。ファイル内のJSONオブジェクトが4GBを超える場合、StarRocksは「このパーサーは、それほど大きなドキュメントをサポートできません」というエラーを投げます。
  
  - デフォルトでは、HTTPリクエストのJSON本文は100MBを超えることはできません。JSON本文が100MBを超える場合、StarRocksは「このバッチのサイズがjson型データの最大サイズ[104857600]を超えています。チェックを無視するにはignore_json_sizeを設定してください。ただし、これにより大量のメモリが消費される可能性があります」というエラーを投げます。このエラーを防ぐためには、HTTPリクエストヘッダーに`"ignore_json_size:true"`を追加して、JSON本文サイズのチェックを無視できます。

  :::

- `stream_load_default_timeout_second`: 各ロードジョブのタイムアウト期間です。デフォルトのタイムアウト期間は600秒です。詳細については、[FE動的パラメータの設定](../administration/FE_configuration.md#configure-fe-dynamic-parameters)を参照してください。
  
  作成するロードジョブの多くがタイムアウトする場合は、次の式から得られる計算結果に基づいて、このパラメータの値を増やすことができます：

  **各ロードジョブのタイムアウト期間 > ロードするデータ量 / 平均ロード速度**

  例えば、ロードしたいデータファイルのサイズが10GBで、StarRocksクラスタの平均ロード速度が100MB/秒の場合、タイムアウト期間を100秒以上に設定します。

  :::note
  
  上記の式での**平均ロード速度**は、StarRocksクラスタの平均ロード速度です。これは、ディスクI/OとStarRocksクラスタ内のBEの数によって異なります。

  :::

  Stream Loadには`timeout`パラメータもあり、個別のロードジョブのタイムアウト期間を指定できます。詳細については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

### 使用上のノート

ロードしたいデータファイルのレコードにフィールドが欠けており、そのフィールドがマッピングされるStarRocksテーブルのカラムが`NOT NULL`として定義されている場合、StarRocksはレコードのロード中にマッピングカラムに自動的に`NULL`値を挿入します。`ifnull()`関数を使用して、挿入したいデフォルト値を指定することもできます。

例えば、`example2.json`ファイルの市区町村IDを表すフィールドが欠けており、`table2`のマッピングカラムに`x`値を挿入したい場合、`"columns: city, tmp_id, id = ifnull(tmp_id, 'x')"`を指定できます。

## ブローカーロードを使用したローカルファイルシステムからのロード

ストリームロードに加えて、ブローカーロードを使用してローカルファイルシステムからデータをロードすることもできます。この機能はv2.5以降でサポートされています。

ブローカーロードは非同期ロード方式です。ロードジョブを送信した後、StarRocksはジョブを非同期で実行し、すぐには結果を返しません。ジョブの結果を手動で照会する必要があります。[ブローカーロードの進捗を確認する](#check-broker-load-progress)を参照してください。

### 制限

- 現在、Broker Loadは、バージョンv2.5以降の単一のブローカーを介してのみ、ローカルファイルシステムからのロードをサポートしています。
- 単一のブローカーに対する高い同時実行クエリは、タイムアウトやOOMなどの問題を引き起こす可能性があります。影響を軽減するために、`pipeline_dop`変数（[システム変数](../reference/System_variable.md#pipeline_dop)を参照）を使用してBroker Loadのクエリの並列度を設定できます。単一ブローカーに対するクエリでは、`pipeline_dop`を`16`未満の値に設定することを推奨します。

### 始める前に

ローカルファイルシステムからデータをBroker Loadでロードする前に、以下の準備を完了させてください：

1. ローカルファイルがあるマシンを、「[デプロイ前提条件](../deployment/deployment_prerequisites.md)」、「[環境設定の確認](../deployment/environment_configurations.md)」、および「[デプロイファイルの準備](../deployment/prepare_deployment_files.md)」に従って設定します。その後、そのマシンにブローカーをデプロイします。操作はBEノードにブローカーをデプロイするのと同じです。詳細な操作については、[ブローカーノードのデプロイと管理](../deployment/deploy_broker.md)を参照してください。

   > **注意**
   >
   > 単一のブローカーのみをデプロイし、そのブローカーのバージョンがv2.5以降であることを確認してください。

2. [ALTER SYSTEM](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md#broker)を実行して、前の手順でデプロイしたブローカーをStarRocksクラスターに追加し、ブローカーに新しい名前を定義します。以下の例では、ブローカー`172.26.199.40:8000`をStarRocksクラスターに追加し、ブローカー名を`sole_broker`として定義しています：

   ```SQL
   ALTER SYSTEM ADD BROKER sole_broker "172.26.199.40:8000";
   ```

### 典型的な例

Broker Loadは、単一のデータファイルから単一のテーブルへのロード、複数のデータファイルから単一のテーブルへのロード、および複数のデータファイルから複数のテーブルへのロードをサポートしています。このセクションでは、複数のデータファイルから単一のテーブルへのロードを例に説明します。

StarRocksでは、一部のリテラルがSQL言語によって予約されたキーワードとして使用されていることに注意してください。これらのキーワードをSQLステートメントで直接使用しないでください。このようなキーワードをSQLステートメントで使用したい場合は、バッククォート（`）で囲んでください。[キーワード](../sql-reference/sql-statements/keywords.md)を参照してください。

#### データセットの準備

CSVファイル形式を例にします。ローカルファイルシステムにログインし、特定の保存場所（例：`/user/starrocks/`）に`file1.csv`と`file2.csv`の2つのCSVファイルを作成します。両ファイルとも、ユーザーID、ユーザー名、ユーザースコアを順に表す3列から構成されています。

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

データベースを作成し、それに切り替えます：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

`mytable`という名前のプライマリキーテーブルを作成します。このテーブルは、`id`、`name`、`score`の3列で構成され、`id`がプライマリキーです。

```SQL
CREATE TABLE `mytable`
(
    `id` int(11) NOT NULL COMMENT "User ID",
    `name` varchar(65533) NULL DEFAULT "" COMMENT "User name",
    `score` int(11) NOT NULL DEFAULT "0" COMMENT "User score"
)
ENGINE=OLAP
PRIMARY KEY(`id`)
DISTRIBUTED BY HASH(`id`)
PROPERTIES("replication_num"="1");
```

#### Broker Loadの開始

ローカルファイルシステムの`/user/starrocks/`パスに保存されたすべてのデータファイル（`file1.csv`と`file2.csv`）からStarRocksテーブル`mytable`へのデータをロードするBroker Loadジョブを開始するには、以下のコマンドを実行します：

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

このジョブには4つの主要なセクションがあります：

- `LABEL`：ロードジョブの状態を照会する際に使用される文字列。
- `LOAD`宣言：ソースURI、ソースデータ形式、および宛先テーブル名。
- `BROKER`：ブローカーの名前。
- `PROPERTIES`：タイムアウト値とロードジョブに適用するその他のプロパティ。

構文とパラメータの詳細な説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

#### Broker Loadの進捗状況の確認

v3.0以前では、[SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)ステートメントまたはcurlコマンドを使用してBroker Loadジョブの進捗状況を表示します。

v3.1以降では、[`information_schema.loads`](../reference/information_schema/loads.md)ビューからBroker Loadジョブの進捗状況を確認できます：

```SQL
SELECT * FROM information_schema.loads;
```

複数のロードジョブを提出した場合、関連する`LABEL`でフィルタリングできます。例えば：

```SQL
SELECT * FROM information_schema.loads WHERE LABEL = 'label_local';
```

ロードジョブが完了したことを確認した後、テーブルをクエリしてデータが正常にロードされたかどうかを確認できます。例：

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

#### Broker Loadジョブのキャンセル

ロードジョブが**CANCELLED**または**FINISHED**ステージにない場合、[CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md)ステートメントを使用してジョブをキャンセルできます。

例えば、以下のステートメントを実行して、データベース`mydatabase`で`label_local`というラベルのロードジョブをキャンセルできます：

```SQL
CANCEL LOAD
FROM mydatabase
WHERE LABEL = "label_local";
```

## Broker Loadを使用したNASからのロード

Broker Loadを使用してNASからデータをロードするには2つの方法があります：

- NASをローカルファイルシステムとして扱い、ブローカーを使用してロードジョブを実行します。前述の「[Broker Loadを介したローカルファイルシステムからのロード](#loading-from-a-local-file-system-via-broker-load)」を参照してください。
- （推奨）NASをクラウドストレージシステムとして扱い、ブローカーなしでロードジョブを実行します。

このセクションでは後者の方法を紹介します。詳細な操作は以下の通りです：

1. NASデバイスをStarRocksクラスタのすべてのBEノードとFEノードに同じパスでマウントします。これにより、すべてのBEは自分のローカルに保存されたファイルにアクセスするようにNASデバイスにアクセスできます。

2. Broker Loadを使用して、NASデバイスからStarRocksの宛先テーブルにデータをロードします。例：

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

   このジョブには4つの主要なセクションがあります：

   - `LABEL`：ロードジョブの状態を照会する際に使用される文字列。
   - `LOAD`宣言：ソースURI、ソースデータ形式、および宛先テーブル名。`DATA INFILE`宣言では、NASデバイスのマウントポイントフォルダパスを指定するために使用されます。上記の例では、`file:///`がプレフィックスで、`/home/disk1/sr`がマウントポイントフォルダパスです。
   - `BROKER`：ブローカー名を指定する必要はありません。
   - `PROPERTIES`: タイムアウト値およびロードジョブに適用するその他のプロパティ。

   詳細な構文とパラメーターの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

ジョブを送信した後、必要に応じてロードの進行状況を確認したり、ジョブをキャンセルしたりすることができます。詳細な操作方法については、このトピック内の「[ブローカーロードの進行状況を確認する](#check-broker-load-progress)」および「[ブローカーロードジョブをキャンセルする](#cancel-a-broker-load-job)」を参照してください。
