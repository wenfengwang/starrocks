---
displayed_sidebar: "Japanese"
---

# ローカルファイルシステムからデータをロードする

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksでは、ローカルファイルシステムからデータをロードするための2つの方法が提供されています：

- [ストリームロード](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を使用した同期ロード
- [ブローカーロード](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を使用した非同期ロード

これらのオプションにはそれぞれ独自の利点があります：

- ストリームロードはCSVおよびJSONファイル形式をサポートしています。この方法は、個々のサイズが10GBを超えない少数のファイルからデータをロードしたい場合に推奨されます。
- ブローカーロードはParquet、ORC、CSVファイル形式をサポートしています。この方法は、個々のサイズが10GBを超える大量のファイルからデータをロードしたい場合や、ファイルがネットワークアタッチドストレージ（NAS）デバイスに保存されている場合に推奨されます。**この機能はv2.5以降でサポートされています。この方法を選択する場合は、データファイルが配置されているマシンに[ブローカーをデプロイ](../deployment/deploy_broker.md)する必要があります。**

CSVデータの場合、次の点に注意してください：

- 50バイトを超えないUTF-8文字列（例：カンマ（,）、タブ、またはパイプ（|））をテキスト区切り記号として使用できます。
- Null値は`\N`を使用して示されます。たとえば、データファイルが3つの列から構成されており、そのデータファイルのレコードが1番目と3番目の列にデータを保持しているが2番目の列にはデータが存在しない場合、この状況では2番目の列に対して`\N`を使用してnull値を示す必要があります。これは、`a,\N,b`としてレコードを編集する必要があります。`a,,b`は、レコードの2番目の列に空の文字列があることを示しています。

ストリームロードとブローカーロードの両方がデータロード時のデータ変換をサポートし、データロード中のUPSERTおよびDELETE操作によるデータ変更をサポートしています。詳細は、[ロード時のデータ変換](../loading/Etl_in_loading.md)および[ロードを介したデータの変更](../loading/Load_to_Primary_Key_tables.md)を参照してください。

## 開始する前に

### 権限を確認する

<InsertPrivNote />

## ストリームロードを使用してローカルファイルシステムからロードする

ストリームロードはHTTP PUTベースの同期ロード方法です。ロードジョブを提出すると、StarRocksは同期的にジョブを実行し、ジョブの結果をジョブが完了した後に返します。ジョブの結果に基づいてジョブが成功したかどうかを判断できます。

> **注意**
>
> Stream Loadを使用してStarRocksテーブルにデータをロードした後、そのテーブルで作成されたマテリアライズドビューのデータも更新されます。

### 動作方法

クライアントでHTTPに従ってロードリクエストをFEに送信し、FEはその後、特定のBEにロードリクエストを転送するためのHTTPリダイレクトを使用します。また、クライアントで直接BEにロードリクエストを送信することもできます。
```
    -H "Expect:100-continue" \
    -H "format: json" -H "jsonpaths: [\"$.name\", \"$.code\"]" \
    -H "columns: city,tmp_id, id = tmp_id * 100" \
    -T example2.json -XPUT \
    http://<fe_host>:<fe_http_port>/api/mydatabase/table2/_stream_load
```

:::note

- パスワードを設定していないアカウントを使用する場合は、`<username>:`のみを入力する必要があります。
- [SHOW FRONTENDS](../sql-reference/sql-statements/Administration/SHOW_FRONTENDS.md)を使用してFEノードのIPアドレスとHTTPポートを表示できます。

:::

`example2.json` は、`name` と `code` の2つのキーで構成されており、これらは次の図に示すように、`table2`の`id`と`city`のカラムにマップされます。

![JSON - Column Mapping](../assets/4.2-2.png)

前の図に示されているマッピングは次のように説明されます。

- StarRocksは、`example2.json`の`name`と`code`キーを抽出し、`jsonpaths`パラメータで宣言された`name`と`code`フィールドにマッピングします。

- StarRocksは、`columns`パラメータで宣言された`city`と`tmp_id`フィールドを抽出し、これらを**連続して**`columns`パラメータで宣言された`city`と `tmp_id`のフィールドにマッピングします。

- StarRocksは、`columns`パラメータで宣言された`city`と`tmp_id`のフィールドを抽出し、これを名前によって`table2`の`city`と`id`のカラムにマッピングします。

:::note

前述の例では、`example2.json`の`code`の値は`table2`の`id`カラムにロードされる前に100倍になります。

:::

`jsonpaths`、`columns`、StarRocksテーブルのカラムとの間の詳細なマッピングについては、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)の「Column mappings」セクションを参照してください。

ロードが完了したら、ロードが成功したことを確認するために`table2`をクエリできます。

```SQL
SELECT * FROM table2;
+------+--------+
| id   | city   |
+------+--------+
| 200  | Beijing|
+------+--------+
4 rows in set (0.01 sec)
```

#### ストリームロードの進捗状況を確認

ロードジョブが完了すると、StarRocksはジョブの結果をJSON形式で返します。詳細については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)の「Return value」セクションを参照してください。

Stream Loadでは、SHOW LOADステートメントを使用してロードジョブの結果をクエリすることはできません。

#### ストリームロードジョブのキャンセル

Stream Loadを使用してロードジョブをキャンセルすることはできません。ロードジョブのタイムアウトやエラーが発生した場合、StarRocksは自動的にジョブをキャンセルします。

### パラメータ構成

このセクションでは、ロード方法であるStream Loadを選択した場合に設定する必要のある一部のシステムパラメータについて説明します。これらのパラメータ構成はすべてのStream Loadジョブに影響します。

- `streaming_load_max_mb`：ロードする各データファイルの最大サイズ。デフォルトの最大サイズは10 GBです。詳細については、[BE動的パラメータの構成](../administration/Configuration.md#configure-be-dynamic-parameters)を参照してください。
  
  1 回にロードするデータファイルのサイズは10 GBを超えないことをお勧めします。データファイルのサイズが10 GBを超える場合は、10 GB未満の小さなファイルにデータファイルを分割してから、これらのファイルを1つずつロードすることをお勧めします。10 GBを超えるデータファイルを分割できない場合は、このパラメータの値をファイルサイズに基づいて増やすことができます。

  このパラメータの値を増やした後は、StarRocksクラスタのBEを再起動するまで新しい値が有効になります。また、システムのパフォーマンスが低下し、ロードの失敗時のリトライのコストも増加する可能性があります。

  :::note
  
  JSONファイルのデータをロードする場合は、以下の点に注意してください：
  
  - ファイル内の各JSONオブジェクトのサイズは4 GBを超えてはいけません。ファイル内の任意のJSONオブジェクトが4 GBを超える場合、StarRocksはエラー「This parser can't support a document that big.」をスローします。
  
  - HTTPリクエストのJSONボディのデフォルトのサイズは100 MBを超えてはいけません。JSONボディが100 MBを超える場合、StarRocksはエラー「The size of this batch exceed the max size [104857600] of json type data data [8617627793]. Set ignore_json_size to skip check, although it may lead huge memory consuming.」をスローします。このエラーを回避するためには、HTTPリクエストヘッダに`"ignore_json_size:true"`を追加して、JSONボディサイズのチェックを無視できます。

  :::

- `stream_load_default_timeout_second`：各ロードジョブのタイムアウト期間。デフォルトのタイムアウト期間は600秒です。詳細については、[FE動的パラメータの構成](../administration/Configuration.md#configure-fe-dynamic-parameters)を参照してください。
  
  作成した多くのロードジョブがタイムアウトする場合、次の計算結果に基づいてこのパラメータの値を増やすことができます。

  **各ロードジョブのタイムアウト期間 > ロードするデータ量 / 平均ローディングスピード**

  たとえば、ロードするデータファイルのサイズが10 GBであり、StarRocksクラスタの平均ローディングスピードが100 MB/sである場合、タイムアウト期間を100秒よりも長く設定します。

  :::note
  
  前述の式の**平均ローディングスピード**は、StarRocksクラスタの平均ローディングスピードです。これはディスクI/OやStarRocksクラスタのBE数に応じて変化します。

  :::

  Stream Loadには個々のロードジョブのタイムアウト期間を指定する`timeout`パラメータも提供されています。詳細については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

### 使用上の注意

ロードするデータファイルのレコードでフィールドが欠落しており、これがStarRocksテーブルで`NOT NULL`として定義されている列にマッピングされている場合、StarRocksはレコードのロード中にマッピング列に`NULL`値を自動的に埋めます。また、`ifnull()`関数を使用して埋めるデフォルト値を指定することもできます。

例えば、前述の`example2.json`ファイルで都市IDを表すフィールドが欠落しており、`table2`のマッピング列に`x`値を埋めたい場合は、`"columns: city, tmp_id, id = ifnull(tmp_id, 'x')"`と指定できます。

## ブローカーロードを使用したローカルファイルシステムからのロード

Stream Loadに加えて、ローカルファイルシステムからデータをロードするためにBroker Loadを使用することもできます。この機能はv2.5以降でサポートされています。

Broker Loadは非同期ロード方法です。ロードジョブを送信した後、StarRocksは非同期でジョブを実行し、すぐにジョブの結果を返しません。ジョブの結果を手動でクエリする必要があります。[Broker Loadの進捗状況を確認](#check-broker-load-progress)を参照してください。

### 制限

- 現在、Broker Loadはv2.5以降のバージョンを持つ単一のブローカーからのみローカルファイルシステムからのロードをサポートしています。
- 単一ブローカーに対する高並列クエリは、タイムアウトやOOMなどの問題を引き起こす可能性があります。影響を緩和するために、[システム変数](../reference/System_variable.md#pipeline_dop)である`pipeline_dop`変数を使用してBroker Loadのクエリ並列処理度を設定することができます。単一ブローカーに対するクエリの場合は、`pipeline_dop`を`16`よりも小さい値に設定することを推奨します。

### 開始前の準備

ローカルファイルシステムからデータをロードするためにBroker Loadを使用する前に、以下の準備を完了してください。

1. ローカルファイルが配置されているマシンを「[デプロイ前提条件](../deployment/deployment_prerequisites.md)」、「[環境構成の確認](../deployment/environment_configurations.md)」、「[デプロイファイルの準備](../deployment/prepare_deployment_files.md)」の手順に従って構成してください。次に、そのマシンにブローカーを展開してください。これはBEノード上でブローカーを展開するときと同じ手順です。詳細な操作については、「[ブローカーノードのデプロイと管理](../deployment/deploy_broker.md)」を参照してください。

   > **注意**
   >
   > 単一のブローカーのみを展開し、ブローカーのバージョンがv2.5以降であることを確認してください。

2. 前の手順で展開したブローカーをStarRocksクラスタに追加し、ブローカーに新しい名前を定義するために[ALTER SYSTEM](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md#broker)を実行してください。次の例では、ブローカー`172.26.199.40:8000`をStarRocksクラスタに追加し、そのブローカーの名前を `sole_broker` に定義しています。

   ```SQL
   ALTER SYSTEM ADD BROKER sole_broker "172.26.199.40:8000";
   ```
### 典型的な例

Broker Loadは、単一のデータファイルから単一のテーブルに、複数のデータファイルから単一のテーブルに、および複数のデータファイルから複数のテーブルにロードすることをサポートしています。このセクションでは、複数のデータファイルから単一のテーブルにロードする例を使用しています。
```
```Plain
StarRocksでは、いくつかのリテラルがSQL言語で予約キーワードとして使用されています。SQLステートメントでこれらのキーワードを直接使用しないでください。SQLステートメントでこのようなキーワードを使用したい場合は、バッククォート（`）で囲んでください。[キーワード](../sql-reference/sql-statements/keywords.md)を参照してください。

#### データセットの準備

CSVファイル形式を使用する例として、ローカルファイルシステムにログインし、特定の保存場所（たとえば、`/user/starrocks/`）に2つのCSVファイル、`file1.csv`と`file2.csv`を作成してください。両方のファイルは、ユーザーID、ユーザー名、およびユーザースコアを順に表す3つの列で構成されています。

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

#### データベースおよびテーブルの作成

データベースを作成し、そのデータベースに切り替えます：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

`mytable`という名前の主キーテーブルを作成します。このテーブルは、`id`、`name`、`score`の3つの列で構成されており、`id`が主キーです。

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

#### Broker Loadの開始

次のコマンドを実行して、ローカルファイルシステムの`/user/starrocks/`パスに保存されているすべてのデータファイル（`file1.csv`および`file2.csv`）からStarRocksテーブル`mytable`にデータをロードするBroker Loadジョブを開始してください。

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

このジョブには、4つのメインセクションがあります：

- `LABEL`：ロードジョブの状態をクエリする際に使用する文字列です。
- `LOAD`宣言：ソースURI、ソースデータ形式、および宛先テーブル名です。
- `BROKER`：ブローカーの名前です。
- `PROPERTIES`：ロードジョブに適用するタイムアウト値およびその他のプロパティです。

詳細な構文とパラメータの説明については、「[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)」を参照してください。

#### Broker Loadの進捗状況の確認

v3.0およびそれ以前では、[SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)ステートメントまたはcurlコマンドを使用してBroker Loadジョブの進捗状況を表示できます。

v3.1およびそれ以降では、Broker Loadジョブの進捗状況を[`information_schema.loads`](../reference/information_schema/loads.md)ビューから表示できます：

```SQL
SELECT * FROMinformation_schema.loads;
```

複数のロードジョブを送信した場合は、ジョブに関連付けられた`LABEL`でフィルタリングできます。例：

```SQL
SELECT * FROMinformation_schema.loadsWHERELABEL= 'label_local';
```

ロードジョブが完了したことを確認したら、データが正常にロードされたかどうかを確認するためにテーブルをクエリできます。例：

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

ロードジョブが**CANCELLED**または**FINISHED**段階にない場合は、[CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md)ステートメントを使用してジョブをキャンセルできます。

たとえば、データベース`mydatabase`でラベルが`label_local`のロードジョブをキャンセルするには、次のステートメントを実行できます：

```SQL
CANCEL LOAD
FROM mydatabase
WHERE LABEL = "label_local";
```

## Broker Loadを使用してNASからのロード

Broker Loadを使用してNASからデータをロードするには、次の2つの方法があります：

- NASをローカルファイルシステムとして考え、ブローカーを使用してロードジョブを実行します。「[ローカルファイルシステムからのBroker Loadによるロード](#loading-from-a-local-file-system-via-broker-load)」の前のセクションを参照してください。
- （推奨）NASをクラウドストレージシステムとして考え、ブローカーなしでロードジョブを実行します。

このセクションでは、2番目の方法を紹介します。具体的な手順は次のとおりです：

1. NASデバイスをStarRocksクラスターのすべてのBEノードおよびFEノードの同じパスにマウントします。このようにすることで、すべてのBEがNASデバイスに自分のローカルに保存されているファイルと同様にアクセスできます。

2. Broker Loadを使用してNASデバイスからデータをStarRocksテーブルにロードします。例：

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

   このジョブには、4つのメインセクションがあります：

   - `LABEL`：ロードジョブの状態をクエリする際に使用する文字列です。
   - `LOAD`宣言：ソースURI、ソースデータ形式、および宛先テーブル名です。注意すべきは、`DATA INFILE`という宣言で、NASデバイスのマウントポイントフォルダパスを指定しています。上記の例では、`file:///`が接頭辞で、`/home/disk1/sr`がマウントポイントフォルダパスです。
   - `BROKER`：ブローカー名を指定する必要はありません。
   - `PROPERTIES`：ロードジョブに適用するタイムアウト値およびその他のプロパティです。

   詳細な構文とパラメータの説明については、「[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)」を参照してください。

ジョブを送信した後は、必要に応じてロードの進捗状況を表示したりジョブをキャンセルしたりすることができます。詳細な手順については、このトピックの「[Broker Loadの進捗状況の確認](#check-broker-load-progress)」および「[Broker Loadジョブのキャンセル](#cancel-a-broker-load-job)」を参照してください。
```