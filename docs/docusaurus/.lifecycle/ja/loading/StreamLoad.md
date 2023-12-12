---
displayed_sidebar: "Japanese"
---

# ローカルファイルシステムからデータをロードする

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksはローカルファイルシステムからデータをロードするための2つの方法を提供しています。

- [ストリームロード](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を使用した同期ロード
- [ブローカーロード](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を使用した非同期ロード

それぞれのオプションにはそれぞれ利点があります。

- ストリームロードはCSVおよびJSONファイル形式をサポートしています。この方法は、個々のサイズが10GBを超えない少数のファイルからデータをロードしたい場合に推奨されます。
- ブローカーロードはParquet、ORC、およびCSVファイル形式をサポートしています。この方法は、個々のサイズが10GBを超える大量のファイルまたはファイルがネットワークアタッチストレージ（NAS）デバイスに保存されている場合に推奨されます。**この機能はv2.5以降でサポートされています。この方法を選択した場合は、データファイルが配置されているマシンに[ブローカーをデプロイ](../deployment/deploy_broker.md)する必要があります。**

CSVデータの場合、次の点に注意してください。

- 文字区切りとして、長さが50バイトを超えないUTF-8文字列（カンマ（,）、タブ、またはパイプ（|））を使用できます。
- Null値は`\N`を使用して示します。たとえば、データファイルが3つの列で構成され、そのデータファイルからのレコードが1つ目と3つ目の列にデータを保持しているが2つ目の列にはデータがない場合、この場合は2つ目の列に`\N`を使用してnull値を示す必要があります。これは、`a,\N,b`ではなく`a,,b`という風にレコードをコンパイルする必要があります。`a,,b`は、レコードの2つ目の列が空の文字列を保持していることを示します。

ストリームロードとブローカーロードは、データのロード時にデータ変換をサポートし、データロード中にUPSERTおよびDELETE操作によるデータ変更をサポートしています。詳細については、[ロード時のデータの変換](../loading/Etl_in_loading.md)および[ロードを通じたデータの変更](../loading/Load_to_Primary_Key_tables.md)を参照してください。

## 開始前の確認

### 権限の確認

<InsertPrivNote />

## ストリームロードを使用したローカルファイルシステムからのロード

ストリームロードは、HTTP PUTベースの同期ロード方法です。ロードジョブを送信した後、StarRocksはジョブを同期的に実行し、ジョブが完了した後にジョブの結果を返します。ジョブが成功したかどうかは、ジョブの結果に基づいて判断できます。

> **注意**
>
> Stream Loadを使用してStarRocksテーブルにデータをロードした後は、そのテーブルで作成されたマテリアライズドビューのデータも更新されます。

### 動作方法

クライアントからFEにHTTPによるロードリクエストを送信し、その後FEはHTTPリダイレクトを使用してロードリクエストを特定のBEに転送します。またはクライアントからBEに直接ロードリクエストを送信することもできます。

:::note

FEにロードリクエストを送信すると、FEはポーリングメカニズムを使用して、どのBEがコーディネータとしてロードリクエストを受け取り、処理するかを決定します。このポーリングメカニズムは、StarRocksクラスタ内での負荷分散を実現するのに役立ちます。そのため、ロードリクエストをFEに送信することをお勧めします。

:::

ロードリクエストを受け取ったBEは、使用されているスキーマに基づいてデータを分割し、データの各部分を他の関連するBEに割り当てます。ロードが完了すると、コーディネータBEはロードジョブの結果をクライアントに返します。なお、ロード中にコーディネータBEを停止させると、ロードジョブは失敗します。

以下の図は、ストリームロードジョブのワークフローを示しています。

![ストリームロードのワークフロー](../assets/4.2-1.png)

### 制限

ストリームロードは、JSON形式の列を含むCSVファイルのデータをロードすることをサポートしていません。

### 典型的な例

このセクションでは、curlを使用して、ローカルファイルシステムからCSVまたはJSONファイルのデータをStarRocksにロードする方法を説明します。詳細な構文とパラメータの説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

StarRocksでは、いくつかのリテラルはSQL言語によって予約語として使用されています。これらのキーワードをSQLステートメントで直接使用しないでください。SQLステートメントでそのようなキーワードを使用する場合は、バッククォート（`）で囲ってください。[Keywords](../sql-reference/sql-statements/keywords.md)を参照してください。

#### CSVデータのロード

##### データセットの準備

ローカルファイルシステムで、`example1.csv`という名前のCSVファイルを作成します。ファイルは、ユーザーID、ユーザー名、およびユーザースコアを順に表す3つの列で構成されています。

```Plain
1,Lily,23
2,Rose,23
3,Alice,24
4,Julia,25
```

##### データベースとテーブルの作成

データベースを作成し、そのデータベースに切り替えます。

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

Primary Keyテーブル`table1`を作成します。このテーブルは、`id`、`name`、および`score`の3つの列で構成されており、`id`がプライマリキーとなっています。

```SQL
CREATE TABLE `table1`
(
    `id` int(11) NOT NULL COMMENT "user ID",
    `name` varchar(65533) NULL COMMENT "user name",
    `score` int(11) NOT NULL COMMENT "user score"
)
ENGINE=OLAP
PRIMARY KEY(`id`)
DISTRIBUTED BY HASH(`id`);
```

:::note

v2.5.7以降、StarRocksはテーブルを作成したりパーティションを追加したりする際に、バケット数（BUCKETS）を自動的に設定できるようになりました。バケット数を手動で設定する必要はもうありません。詳細については、[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)をご覧ください。

:::

##### ストリームロードの開始

次のコマンドを実行して、`example1.csv`のデータを`table1`にロードします。

```Bash
curl --location-trusted -u <ユーザ名>:<パスワード> -H "label:123" \
    -H "Expect:100-continue" \
    -H "column_separator:," \
    -H "columns: id, name, score" \
    -T example1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/mydatabase/table1/_stream_load
```

:::note

- パスワードが設定されていないアカウントを使用する場合は、`<ユーザ名>:`のように入力する必要があります。
- FEノードのIPアドレスおよびHTTPポートを表示するには、[SHOW FRONTENDS](../sql-reference/sql-statements/Administration/SHOW_FRONTENDS.md)を使用できます。

:::

`example1.csv`は、カンマ（,）で区切られた3つの列で構成されており、これらは`table1`の`id`、`name`、および`score`の3つの列に順にマップできます。そのため、`column_separator`パラメータを使用して、カンマ（,）を列の区切り文字として指定する必要があります。また、一時的に`columns`パラメータを使用して、`example1.csv`の3つの列を`id`、`name`、および`score`として名前付けし、これらを`table1`の3つの列に順にマップする必要があります。

ロードが完了した後、`table1`をクエリして、ロードが成功したことを確認できます。

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

ローカルファイルシステムで、`example2.json`という名前のJSONファイルを作成します。ファイルは、都市IDと都市名を順に表す2つの列で構成されています。

```JSON
{"name": "Beijing", "code": 2}
```

##### データベースとテーブルの作成

データベースを作成し、そのデータベースに切り替えます。

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

Primary Keyテーブル`table2`を作成します。このテーブルは、`id`および`city`の2つの列で構成されており、`id`がプライマリキーとなっています。

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

v2.5.7以降、StarRocksはテーブルを作成したりパーティションを追加したりする際に、バケット数（BUCKETS）を自動的に設定できるようになりました。バケット数を手動で設定する必要はもうありません。詳細については、[バケット数の決定](../table_design/Data_distribution.md#determine-the-number-of-buckets)をご覧ください。

:::

##### ストリームロードの開始

次のコマンドを実行して、`example2.json`のデータを`table2`にロードします。

```Bash
curl -v --location-trusted -u <ユーザ名>:<パスワード> -H "strict_mode: true" \
```
    -H "Expect:100-continue" \
    -H "format: json" -H "jsonpaths: [\"$.name\", \"$.code\"]" \
    -H "columns: city,tmp_id, id = tmp_id * 100" \
    -T example2.json -XPUT \
    http://<fe_host>:<fe_http_port>/api/mydatabase/table2/_stream_load
```

:::note

- パスワードが設定されていないアカウントを使用する場合は、`<username>:` のみ入力する必要があります。
- [SHOW FRONTENDS](../sql-reference/sql-statements/Administration/SHOW_FRONTENDS.md)を使用して、FEノードのIPアドレスとHTTPポートを表示できます。

:::

`example2.json` には、`name` と `code` という2つのキーが含まれており、これらは次の図に示すように、`table2`の `id` と `city` のカラムにマッピングされます。

![JSON - Column Mapping](../assets/4.2-2.png)

前述の図に示されるマッピングは次のとおりです。

- StarRocksは、`example2.json`の `name` と `code` キーを抽出し、`jsonpaths`パラメータで宣言された `name` と `code` フィールドにマッピングします。

- StarRocksは、`columns` パラメータで宣言された `name` と `code` フィールドを**順次**抽出し、`columns` パラメータで宣言された `city` と `tmp_id` フィールドにマッピングします。

- StarRocksは、`columns` パラメータで宣言された `city` と `tmp_id` フィールドを**名前で**抽出し、`table2`の `city` と `id` のカラムにマッピングします。

:::note

前述の例では、`example2.json`の `code` の値が `id` のカラムにロードされる前に100倍されます。

:::

`jsonpaths`、`columns`、およびStarRocksテーブルのカラム間の詳細なマッピングについては、「STREAM LOAD」の「Column mappings」セクションを参照してください。

ロードが完了した後、`table2`をクエリしてロードが成功したことを確認できます。

```SQL
SELECT * FROM table2;
+------+--------+
| id   | city   |
+------+--------+
| 200  | Beijing|
+------+--------+
4 rows in set (0.01 sec)
```

#### Stream Loadの進捗状況をチェックする

ロードジョブが完了すると、StarRocksはJSON形式でジョブの結果を返します。詳細については、「STREAM LOAD」の「Return value」セクションを参照してください。

Stream Loadでは、SHOW LOADステートメントを使用してロードジョブの結果をクエリすることはできません。

#### Stream Loadジョブのキャンセル

Stream Loadでは、ロードジョブをキャンセルすることはできません。ロードジョブがタイムアウトした場合やエラーが発生した場合、StarRocksは自動的にジョブをキャンセルします。

### パラメータの設定

このセクションでは、ロード方法「Stream Load」を選択した場合に設定する必要があるいくつかのシステムパラメータについて説明します。これらのパラメータ設定は、すべてのStream Loadジョブに影響します。

- `streaming_load_max_mb`: ロードする各データファイルの最大サイズ。デフォルトの最大サイズは10 GBです。詳細については、「BE動的パラメータを設定する」を参照してください。

  一度に10 GBを超えるデータをロードしないでください。データファイルのサイズが10 GBを超える場合は、10 GB未満の小さいファイルにデータファイルを分割してからそれらを1つずつロードすることをお勧めします。10 GBを超えるデータファイルを分割できない場合は、このパラメータの値をファイルサイズに基づいて増やすことができます。

  このパラメータの値を増やした後は、StarRocksクラスタのBEを再起動するまで新しい値が適用されません。また、システムの性能が悪化し、ロードの失敗が発生した場合の再試行のコストも増加します。

  :::note
  
  JSONファイルのデータをロードする際には、次の点に注意してください。

  - ファイル内の各JSONオブジェクトのサイズは4 GBを超えることはできません。ファイル内のいずれかのJSONオブジェクトが4 GBを超える場合、StarRocksは「This parser can't support a document that big.」というエラーをスローします。

  - HTTPリクエストのJSONボディのデフォルトのサイズは100 MBを超えることはできません。JSONボディが100 MBを超える場合、StarRocksは「The size of this batch exceed the max size [104857600] of json type data data [8617627793]. Set ignore_json_size to skip check, although it may lead huge memory consuming.」というエラーをスローします。このエラーを防ぐためには、HTTPリクエストヘッダに`"ignore_json_size:true"`を追加して、JSONボディサイズのチェックを無視します。

  :::

- `stream_load_default_timeout_second`: 各ロードジョブのタイムアウト期間。デフォルトのタイムアウト期間は600秒です。

  たくさんのロードがタイムアウトする場合は、次の計算結果に基づいてこのパラメータの値を増やすことができます。

  **各ロードジョブのタイムアウト期間 > ロードするデータ量/平均ロード速度**

  たとえば、ロードするデータファイルのサイズが10 GBであり、StarRocksクラスタの平均ロード速度が100 MB/sである場合は、タイムアウト期間を100秒以上に設定します。

  :::note
  
  上記の式中の**平均ロード速度**は、StarRocksクラスタの平均ロード速度です。これはディスクI/OやStarRocksクラスタのBEの数によって異なります。

  :::

  Stream Loadは、個々のロードジョブのタイムアウト期間を指定する`timeout`パラメータも提供しています。詳細については、「STREAM LOAD」を参照してください。

### 使用上の注意

ロードしようとするデータファイルのレコードにフィールドが欠落しており、StarRocksテーブルにマッピングされるカラムが`NOT NULL`として定義されている場合、StarRocksはそのレコードのマッピングカラムに `NULL` の値を自動的に入力します。また、`ifnull()` 関数を使用して、入力するデフォルト値を指定することもできます。

例えば、前述の `example2.json` ファイルで都市IDを表すフィールドが欠落しており、`table2`のマッピングカラムに `x` の値を入力したい場合は、`"columns: city, tmp_id, id = ifnull(tmp_id, 'x')"` を指定します。

## Broker Loadを使用してローカルファイルシステムからロードする

Stream Loadに加えて、ローカルファイルシステムからデータをロードするためにBroker Loadを使用することもできます。この機能はv2.5以降でサポートされています。

Broker Loadは非同期のロード方法です。ロードジョブを送信した後、StarRocksはジョブを非同期で実行し、すぐにジョブの結果を返しません。ジョブ結果を手動でクエリする必要があります。"Check Broker Load progress"を参照してください。

### 制限事項

- 現在、Broker Loadはv2.5以降のバージョンを持つ単一のブローカーからのみローカルファイルシステムからのロードをサポートしています。
- 単一のブローカーに対する高度な並列クエリは、タイムアウトやOOMなどの問題を引き起こす可能性があります。影響を軽減するために、ブローカーへの問合せ並列度を設定するために`pipeline_dop`変数（「System variable」を参照してください）を使用できます。単一のブローカーに対するクエリでは、`pipeline_dop`を`16`よりも小さい値に設定することをお勧めします。

### 開始前に

ローカルファイルシステムからデータをロードするためにBroker Loadを使用するために、次の準備を完了してください。

1. ローカルファイルがあるマシンを"[Deployment prerequisites](../deployment/deployment_prerequisites.md)" で指示された通りに構成し、「[Check environment configurations](../deployment/environment_configurations.md)」と「[Prepare deployment files](../deployment/prepare_deployment_files.md)」で構成のチェックを完了し、そのマシンにブローカーをデプロイします。操作手順はBEノードにブローカーをデプロイする場合と同じです。詳細な操作手順については、「Deploy and manage Broker node」を参照してください。

   > **NOTICE**
   >
   > 単一のブローカーのみをデプロイし、そのブローカーのバージョンがv2.5以降であることを確認してください。

2. 以下の例では、`172.26.199.40:8000`をStarRocksクラスタに追加し、「sole_broker」の名前を定義する（`ALTER SYSTEM` の詳細については、「ALTER SYSTEM」を参照してください）。

   ```SQL
   ALTER SYSTEM ADD BROKER sole_broker "172.26.199.40:8000";
   ```

### 典型的な例

Broker Loadは、単一のデータファイルから単一のテーブルへのロード、複数のデータファイルから単一のテーブルへのロード、複数のデータファイルから複数のテーブルへのロードをサポートしています。このセクションでは、単一のテーブルへの複数のデータファイルからのロードの例を示します。
注意：StarRocksではSQL言語によって予約キーワードとして使用されるリテラルがあります。SQLステートメントでこれらのキーワードを直接使用しないでください。SQLステートメントでそのようなキーワードを使用する場合は、バッククォート（`）で囲んでください。[キーワード](../sql-reference/sql-statements/keywords.md)を参照してください。

#### データセットの準備

CSVファイル形式を例に使用します。ローカルファイルシステムにログインし、指定のストレージ位置（たとえば`/user/starrocks/`）に`file1.csv`と`file2.csv`の2つのCSVファイルを作成します。両方のファイルは、ユーザーID、ユーザー名、およびユーザースコアを順に表す3つの列で構成されます。

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

`mytable`という主キー表を作成します。このテーブルは、`id`、`name`、`score`の3つの列で構成されており、`id`がプライマリキーです。

```SQL
CREATE TABLE `mytable`
(
    `id` int(11) NOT NULL COMMENT "ユーザーID",
    `name` varchar(65533) NULL DEFAULT "" COMMENT "ユーザー名",
    `score` int(11) NOT NULL DEFAULT "0" COMMENT "ユーザースコア"
)
ENGINE=OLAP
PRIMARY KEY(`id`)
DISTRIBUTED BY HASH(`id`)
PROPERTIES("replication_num"="1");
```

#### ブローカーロードの開始

次のコマンドを実行して、ローカルファイルシステムの`/user/starrocks/`パスに保存されているすべてのデータファイル（`file1.csv`および`file2.csv`）からStarRocksテーブル`mytable`にデータをロードするブローカーロードジョブを開始します：

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

このジョブには4つの主要なセクションがあります。

- `LABEL`：ロードジョブの状態をクエリする際に使用される文字列。
- `LOAD`宣言：ソースURI、ソースデータ形式、および宛先テーブル名。
- `BROKER`：ブローカーの名前。
- `PROPERTIES`：ロードジョブに適用するタイムアウト値およびその他のプロパティ。

詳細な構文とパラメータの説明については、[ブローカーロード](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

#### ブローカーロードの進捗状況の確認

v3.0およびそれ以前では、[SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)ステートメントまたはcurlコマンドを使用して、ブローカーロードジョブの進捗状況を表示できます。

v3.1以降では、ブローカーロードジョブの進捗状況を[`information_schema.loads`](../reference/information_schema/loads.md)ビューから表示できます。

```SQL
SELECT * FROMinformation_schema.loads;
```

複数のロードジョブを送信した場合は、ジョブに関連付けられた`LABEL`でフィルタリングできます。例：

```SQL
SELECT * FROMinformation_schema.loadsWHERELABEL= 'label_local';
```

ロードジョブが完了したことを確認した後、データが正常にロードされたかどうかを確認するために、テーブルをクエリできます。例：

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

ロードジョブが**CANCELLED**または**FINISHED**ステージにない場合は、[CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md)ステートメントを使用してジョブをキャンセルできます。

たとえば、データベース`mydatabase`でラベルが`label_local`のロードジョブをキャンセルするには、次のようなステートメントを実行できます。

```SQL
CANCEL LOAD
FROM mydatabase
WHERE LABEL = "label_local";
```

## NAS経由のブローカーロード

ブローカーロードを使用してNASからデータをロードする方法は2つあります。

- NASをローカルファイルシステムとして考え、ブローカーを使用してロードジョブを実行します。前のセクションの"[Loading from a local system via Broker Load](#loading-from-a-local-file-system-via-broker-load)"を参照してください。
- (推奨) NASをクラウドストレージシステムとして考え、ブローカーを使用せずにロードジョブを実行します。

このセクションでは、2番目の方法を紹介します。詳細な操作は次のとおりです。

1. NASデバイスをStarRocksクラスターのすべてのBEノードおよびFEノードで同じパスにマウントします。これにより、すべてのBEがNASデバイスにローカルに保存されたファイルにアクセスできるようになります。

2. ブローカーロードを使用してNASデバイスから目的のStarRocksテーブルにデータをロードします。例：

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

   このジョブには4つの主要なセクションがあります。

   - `LABEL`：ロードジョブの状態をクエリする際に使用される文字列。
   - `LOAD`宣言：ソースURI、ソースデータ形式、および宛先テーブル名。この宣言の`DATA INFILE`は、NASデバイスのマウントポイントフォルダパスを指定するために使用されます。上記の例では`file:///`がプレフィックスであり、`/home/disk1/sr`がマウントポイントフォルダパスです。
   - `BROKER`：ブローカー名を指定する必要はありません。
   - `PROPERTIES`：ロードジョブに適用するタイムアウト値およびその他のプロパティ。

   詳細な構文とパラメータの説明については、[ブローカーロード](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

ジョブを送信すると、必要に応じてロードの進捗状況を表示したり、ジョブをキャンセルしたりすることができます。詳細な操作については、このトピックの"[Check Broker Load progress](#check-broker-load-progress)"および"[Cancel a Broker Load job](#cancel-a-broker-load-job)"を参照してください。