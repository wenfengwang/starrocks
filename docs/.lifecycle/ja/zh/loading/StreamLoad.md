---
displayed_sidebar: Chinese
---
# 本地ファイルシステムからのインポート

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

StarRocksでは、ローカルファイルシステムからデータをインポートするための2つの方法が提供されています：

- [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を使用して同期的にインポートします。
- [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を使用して非同期にインポートします。

それぞれのインポート方法にはそれぞれの利点があります：

- Stream Loadは、CSVおよびJSONの2つのデータファイル形式をサポートしており、データファイルの数が少なく、個々のファイルのサイズが10 GB以下の場合に適しています。
- Broker Loadは、Parquet、ORC、およびCSVの3つのファイル形式をサポートしており、データファイルの数が多く、個々のファイルのサイズが10 GBを超える場合や、ファイルがNASに保存されている場合に適しています。**ただし、この機能はv2.5以降でサポートされており、このインポート方法を使用するには、データが保存されているマシンに[Brokerをデプロイ](../deployment/deploy_broker.md)する必要があります。**

CSV形式のデータの場合、次の2つの点に注意する必要があります：

- StarRocksは、最大50バイトのUTF-8エンコード文字列を列区切り記号として設定することができます。一般的なカンマ(,)、タブ、パイプ(|)などが含まれます。
- NULL値は`\N`で表されます。たとえば、データファイルには3つの列があり、ある行の1番目の列と3番目の列のデータがそれぞれ`a`と`b`である場合、2番目の列にデータがない場合は、2番目の列をNULL値で表すために`\N`を使用する必要があります。つまり、`a,\N,b`と書きます。`a,,b`ではなく、`a,,b`は2番目の列が空の文字列を表します。

Stream LoadおよびBroker Loadは、インポートプロセス中にデータ変換を行ったり、UPSERTおよびDELETE操作を使用してデータ変更を実現することができます。[インポートプロセスでのデータ変換の実現](../loading/Etl_in_loading.md)および[インポートによるデータ変更の実現](../loading/Load_to_Primary_Key_tables.md)を参照してください。

## 準備

### 権限の確認

<InsertPrivNote />

## Stream Loadを使用してローカルからインポート

Stream Loadは、HTTP PUTベースの同期的なインポート方法です。インポートジョブを送信すると、StarRocksはインポートジョブを同期的に実行し、インポートジョブの結果情報を返します。返された結果情報を使用して、インポートジョブの成功を判断することができます。

> **注意**
>
> Stream Load操作は、StarRocksの元のテーブルに関連するマテリアライズドビューのデータも同時に更新します。

### 基本原理

クライアント側でHTTPを使用してFEにインポートジョブリクエストを送信する必要があります。FEはHTTPリダイレクト（Redirect）命令を使用してリクエストを特定のBEに転送します。また、インポートジョブリクエストを直接特定のBEに送信することもできます。

:::note

インポートジョブリクエストをFEに送信すると、FEはラウンドロビンメカニズムを使用してどのBEがリクエストを受け取るかを選択し、StarRocksクラスタ内での負荷分散を実現します。したがって、インポートジョブリクエストをFEに送信することをお勧めします。

:::

インポートジョブリクエストを受け取るBEは、Coordinator BEとして機能し、データをテーブル構造に基づいて分割し、他の関連するBEにデータを配布します。インポートジョブの結果情報は、Coordinator BEによってクライアントに返されます。なお、インポートプロセス中にCoordinator BEを停止すると、インポートジョブが失敗します。

以下の図は、Stream Loadの主なフローを示しています：

![Stream Loadのフローチャート](../assets/4.2-1-zh.png)

### 使用制限

Stream Loadは、現在、JSONの列を持つCSVファイルのデータをインポートすることはサポートしていません。

### 操作例

この記事では、curlツールを使用して、ローカルファイルシステムからCSVまたはJSON形式のデータをStream Loadを使用してインポートする方法について説明します。インポートジョブの詳細な構文とパラメータの説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

StarRocksでは、一部のテキストがSQL言語の予約語であるため、直接SQLステートメントで使用することはできません。これらの予約語をSQLステートメントで使用する場合は、バッククォート（`）で囲む必要があります。[キーワード](../sql-reference/sql-statements/keywords.md)を参照してください。

#### CSV形式のデータのインポート

##### データの例

ローカルファイルシステムに、次のような3つの列を持つCSV形式のデータファイル`example1.csv`を作成します。ファイルには、ユーザーID、ユーザー名、およびユーザースコアが含まれています。

```Plain_Text
1,Lily,23
2,Rose,23
3,Alice,24
4,Julia,25
```

##### データベースとテーブルの作成

次のステートメントを使用して、データベースを作成し、そのデータベースに切り替えます。

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

次のステートメントを使用して、主キーモデルテーブル`table1`を手動で作成します。このテーブルには、`id`、`name`、および`score`の3つの列が含まれており、ユーザーIDを表す`id`列が主キーです。

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

バージョン2.5.7以降、StarRocksはテーブルとパーティションの作成時に自動的にバケット数（BUCKETS）を設定することができます。手動でバケット数を設定する必要はありません。詳細については、「[バケット数の決定](../table_design/Data_distribution.md#バケット数の決定)」を参照してください。

:::

##### インポートジョブの送信

次のコマンドを使用して、`example1.csv`ファイルのデータを`table1`テーブルにインポートします。

```Bash
curl --location-trusted -u <username>:<password> -H "label:123" \
    -H "Expect:100-continue" \
    -H "column_separator:," \
    -H "columns: id, name, score" \
    -T example1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/mydatabase/table1/_stream_load
```

:::note

- パスワードが設定されていない場合は、`<username>:`のように入力するだけです。
- [SHOW FRONTENDS](../sql-reference/sql-statements/Administration/SHOW_FRONTENDS.md)コマンドを使用して、FEノードのIPアドレスとHTTPポート番号を確認できます。

:::

`example1.csv`ファイルには3つの列が含まれており、`table1`テーブルの`id`、`name`、`score`の3つの列と対応しています。列区切り記号としてカンマ（,）を使用しているため、`column_separator`パラメータでカンマ（,）を指定し、`columns`パラメータで`example1.csv`ファイルの3つの列を`id`、`name`、`score`として一時的に指定する必要があります。`columns`パラメータで宣言された3つの列は、名前に基づいて`table1`テーブルの3つの列に対応します。

インポートが完了したら、`table1`テーブルをクエリして、データのインポートが成功したかどうかを確認できます。

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

#### JSON形式のデータのインポート

##### データの例

ローカルファイルシステムに、次のような2つのフィールドを持つJSON形式のデータファイル`example2.json`を作成します。ファイルには、都市名と都市IDが含まれています。

```JSON
{"name": "北京", "code": 2}
```

##### データベースとテーブルの作成

次のステートメントを使用して、データベースを作成し、そのデータベースに切り替えます。

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

次のステートメントを使用して、主キーモデルテーブル`table2`を手動で作成します。このテーブルには、`id`と`city`の2つの列が含まれており、都市IDを表す`id`列が主キーです。

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

バージョン2.5.7以降、StarRocksはテーブルとパーティションの作成時に自動的にバケット数（BUCKETS）を設定することができます。手動でバケット数を設定する必要はありません。詳細については、「[バケット数の決定](../table_design/Data_distribution.md#バケット数の決定)」を参照してください。

:::

##### インポートジョブの送信

次のステートメントを使用して、`example2.json`ファイルのデータを`table2`テーブルにインポートします。

```Bash
curl -v --location-trusted -u <username>:<password> -H "strict_mode: true" \
    -H "Expect:100-continue" \
    -H "format: json" -H "jsonpaths: [\"$.name\", \"$.code\"]" \
    -H "columns: city,tmp_id, id = tmp_id * 100" \
    -T example2.json -XPUT \
    http://<fe_host>:<fe_http_port>/api/mydatabase/table2/_stream_load
```

:::note

- パスワードが設定されていない場合は、`<username>:`のように入力するだけです。
- [SHOW FRONTENDS](../sql-reference/sql-statements/Administration/SHOW_FRONTENDS.md)コマンドを使用して、FEノードのIPアドレスとHTTPポート番号を確認できます。

:::

`example2.json`ファイルには、`name`と`code`の2つのキーが含まれており、`table2`テーブルの列との対応関係は以下の図に示されています。

![JSONマッピング図](../assets/4.2-2.png)

上記の図の対応関係は次のように説明されます：

- `example2.json`ファイルに含まれる`name`と`code`の2つのフィールドを、`jsonpaths`パラメータで宣言された`name`と`code`の2つのフィールドに順番にマッピングします。
- `jsonpaths`パラメータで宣言された`name`と`code`の2つのフィールドを、`columns`パラメータで宣言された`city`と`tmp_id`の2つの列に**順番にマッピング**します。
- `columns`パラメータで宣言された`city`と`id`の2つの列を、`table2`テーブルの`city`と`id`の2つの列に**名前でマッピング**します。

:::note

上記の例では、インポート中に`example2.json`ファイルの`code`フィールドに対応する値を100倍にし、それを`table2`テーブルの`id`に落とし込んでいます。

:::

インポートが完了したら、`table2`テーブルをクエリして、データのインポートが成功したかどうかを確認できます。

```SQL
SELECT * FROM table2;
+------+--------+
| id   | city   |
+------+--------+
| 200  | 北京    |
+------+--------+
4 rows in set (0.01 sec)
```

#### Stream Loadのインポート進捗状況の確認

インポートジョブが終了すると、StarRocksはJSON形式でインポートジョブの結果情報を返します。詳細については、STREAM LOADのドキュメントの「[返り値](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#返り値)」セクションを参照してください。

Stream Loadでは、SHOW LOADステートメントを使用してインポートジョブの実行状況を確認することはできません。

#### Stream Loadジョブのキャンセル

Stream Loadでは、インポートジョブを手動でキャンセルすることはできません。インポートジョブがタイムアウトした場合やインポートエラーが発生した場合、StarRocksは自動的にジョブをキャンセルします。

### パラメータの設定

ここでは、Stream Loadのインポート方法に注意する必要があるいくつかのシステムパラメータの設定について説明します。これらのパラメータは、すべてのStream Loadインポートジョブに適用されます。

```
- `streaming_load_max_mb`：単一のソースデータファイルのサイズ上限。デフォルトのファイルサイズ上限は10 GBです。詳細については、[BEの動的パラメータの設定](../administration/BE_configuration.md)を参照してください。

一度にインポートするデータ量は10 GBを超えないようにしてください。データファイルのサイズが10 GBを超える場合は、10 GB未満のファイルに分割して複数回にわたってインポートすることをお勧めします。データファイルを分割することができない場合は、このパラメータの値を適切に増やすことで、データファイルのサイズ上限を高めることができます。

ただし、このパラメータの値を増やす場合は、BEを再起動する必要があり、システムのパフォーマンスに影響を与える可能性があり、失敗時のリトライのコストも増えることに注意してください。

:::note

JSON形式のデータをインポートする場合、次の2つの点に注意してください：

- 単一のJSONオブジェクトのサイズは4 GBを超えることはできません。JSONファイル内の単一のJSONオブジェクトのサイズが4 GBを超える場合、「This parser can't support a document that big.」というエラーが表示されます。
- HTTPリクエストのJSONボディのデフォルトのサイズ上限は100 MBを超えることはできません。JSONボディのサイズが100 MBを超える場合、「The size of this batch exceeds max size of json [104857600] type data data [8617627793]. Set ignore_json_size to skip check, although it may lead huge memory consuming.」というエラーが表示されます。このエラーを回避するためには、HTTPリクエストヘッダに「"ignore_json_size:true"」を追加して、JSONボディのサイズのチェックを無視するように設定することができます。

:::

- `stream_load_default_timeout_second`：インポートジョブのタイムアウト時間。デフォルトのタイムアウト時間は600秒です。詳細については、[FEの動的パラメータの設定](../administration/FE_configuration.md)を参照してください。

作成したインポートジョブが頻繁にタイムアウトする場合は、タイムアウト時間を適切に増やすことができます。以下の式を使用して、インポートジョブのタイムアウト時間を計算することができます。

**インポートジョブのタイムアウト時間 > 待機中のデータ量/平均インポート速度**

例えば、ソースデータファイルのサイズが10 GBであり、現在のStarRocksクラスタの平均インポート速度が100 MB/sである場合、タイムアウト時間は100秒以上に設定する必要があります。

:::note

「平均インポート速度」とは、現在のStarRocksクラスタの平均インポート速度を指します。インポート速度は主にクラスタのディスクI/OとBEの数に制限されます。

:::

Stream Loadでは、現在のインポートジョブのタイムアウト時間を設定するための「timeout」パラメータも提供されています。詳細については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

### 使用方法

データファイルの中の特定のレコードの特定のフィールドが欠落しており、かつそのフィールドが対象のテーブルの列で「NOT NULL」に定義されている場合、StarRocksはそのフィールドを対象のテーブルの対応する列に「NULL」値で埋めることができます。また、インポートコマンドで「ifnull()」関数を使用して、デフォルト値を指定することもできます。

例えば、上記の例で、データファイル「example2.json」の中の都市IDを表すフィールドが欠落しており、そのフィールドに対応する目的のテーブルの列に「x」を埋めたい場合、インポートコマンドで「"columns: city, tmp_id, id = ifnull(tmp_id, 'x')"」と指定することができます。

## ローカルからのBroker Loadの使用

Stream Load以外にも、ローカルからBroker Loadを使用してデータをインポートすることもできます。この機能はv2.5以降でサポートされています。

Broker Loadは非同期のインポート方法です。インポートジョブを送信すると、StarRocksは非同期でジョブを実行し、ジョブの結果を直接返さず、ジョブの結果を手動でクエリする必要があります。[Broker Loadのインポートの進捗状況を確認する](#Broker-Load-のインポートの進捗状況を確認する)を参照してください。

### 使用制限

- 現時点では、単一のBrokerからのみデータをインポートできます。また、Brokerのバージョンは2.5以降である必要があります。
- 単一のBrokerへの同時アクセスはボトルネックになり、高い並行性はタイムアウトやOOMなどの問題を引き起こしやすくなります。Broker Loadの並行度を制限するために、`pipeline_dop`を設定することができます（[クエリの並行度を調整する](../administration/Query_management.md#クエリの並行度を調整する)を参照）。単一のBrokerへのアクセスの並行度は、`16`未満に設定することをお勧めします。

### 準備作業

ローカルファイルシステムからデータをインポートする前に、次の準備作業を完了する必要があります。

1. [デプロイの前提条件](../deployment/deployment_prerequisites.md)、[環境設定の確認](../deployment/environment_configurations.md)、および[デプロイファイルの準備](../deployment/prepare_deployment_files.md)の手順に従って、ローカルファイルが存在するマシンで必要な環境設定を完了します。次に、そのマシンにBrokerをデプロイします。操作の詳細については、BEノードと同様に[Brokerノードのデプロイ](../deployment/deploy_broker.md)を参照してください。

   > **注意**
   >
   > データを単一のBrokerからのみインポートできます。また、Brokerのバージョンは2.5以降である必要があります。

2. [ALTER SYSTEM](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md#broker)ステートメントを使用して、StarRocksに前の手順でデプロイしたBroker（例：`172.26.199.40:8000`）を追加し、Brokerに新しい名前（例：`sole_broker`）を指定します。

   ```SQL
   ALTER SYSTEM ADD BROKER sole_broker "172.26.199.40:8000";
   ```

### 操作の例

Broker Loadは、単一のデータファイルを単一のテーブルにインポートしたり、複数のデータファイルを単一のテーブルにインポートしたり、複数のデータファイルを個別のテーブルにそれぞれインポートしたりすることができます。ここでは、複数のデータファイルを単一のテーブルにインポートする例を示します。

StarRocksでは、一部の単語はSQL言語の予約語であり、直接SQLステートメントで使用することはできません。これらの予約語をSQLステートメントで使用する場合は、バッククォート（`）で囲む必要があります。[予約語](../sql-reference/sql-statements/keywords.md)を参照してください。

#### データの例

CSV形式のデータを例にします。ローカルファイルシステムにログインし、指定したパス（例：`/user/starrocks/`）に2つのCSV形式のデータファイル、`file1.csv`と`file2.csv`を作成します。2つのデータファイルはいずれも3つの列を含み、それぞれユーザーID、ユーザー名、ユーザースコアを表します。以下に例を示します：

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

次のステートメントを使用して、データベースを作成し、そのデータベースに切り替えます：

```SQL
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;
```

次のステートメントを使用して、主キーモデルテーブル`mytable`を手動で作成します。このテーブルには、`id`、`name`、`score`の3つの列があり、それぞれユーザーID、ユーザー名、ユーザースコアを表します。主キーは`id`列です。以下に例を示します：

```SQL
DROP TABLE IF EXISTS `mytable`

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

#### インポートジョブの送信

次のステートメントを使用して、ローカルファイルシステムの`/user/starrocks/`パスにあるすべてのデータファイル（`file1.csv`と`file2.csv`）のデータをターゲットテーブル`mytable`にインポートします：

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

このインポートステートメントには、次の4つの部分が含まれています：

- `LABEL`：インポートジョブのラベル。文字列型で、インポートジョブのステータスをクエリするために使用できます。
- `LOAD`ステートメント：ソースデータファイルのURI、ソースデータファイルの形式、およびターゲットテーブルの名前など、ジョブの説明情報を含みます。
- `BROKER`：Brokerの名前。
- `PROPERTIES`：タイムアウト時間などのオプションのジョブプロパティを指定するために使用されます。

詳細な構文とパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

#### Broker Loadのインポートの進捗状況を確認する

v3.0以前のバージョンでは、インポートジョブの進捗状況を確認するためには、[SHOW LOAD](../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)ステートメントやcurlコマンドを使用する必要がありました。

v3.1以降のバージョンでは、[information_schema.loads](../reference/information_schema/loads.md)ビューを使用して、Broker Loadジョブの進捗状況を確認することができます：

```SQL
SELECT * FROM information_schema.loads;
```

複数のインポートジョブを送信した場合は、`LABEL`を使用して表示したいジョブをフィルタリングすることができます。例：

```SQL
SELECT * FROM information_schema.loads WHERE LABEL = 'label_local';
```

インポートジョブが完了したことを確認したら、テーブルからデータをクエリして、データのインポートが成功したかどうかを確認することができます。例：

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

インポートジョブのステータスが「CANCELLED」または「FINISHED」でない場合は、[CANCEL LOAD](../sql-reference/sql-statements/data-manipulation/CANCEL_LOAD.md)ステートメントを使用して、そのインポートジョブをキャンセルすることができます。

例えば、`mydatabase`データベースでラベルが「label_local」のインポートジョブをキャンセルするには、次のステートメントを使用します：

```SQL
CANCEL LOAD
FROM mydatabase
WHERE LABEL = "label_local";
```

## NASからのBroker Loadの使用

NASからデータをインポートする場合、次の2つの方法があります：

- NASをローカルファイルシステムとして使用し、前述の「[ローカルからのBroker Loadの使用](#ローカルからのBroker-Loadの使用)」で説明されている方法に従ってインポートを実現します。
- 【推奨】NASをクラウドストレージデバイスとして使用し、Broker Loadを使用してインポートすることができます。

このセクションでは、後者の方法について説明します。具体的な手順は以下の通りです：

1. NASをすべてのBE、FEノードにマウントし、すべてのノードでマウントパスが完全に一致するようにします。これにより、すべてのBEがNASにアクセスできるようになります。

2. Broker Loadを使用してデータをインポートします。

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

   このインポートステートメントには、次の4つの部分が含まれています：

   - `LABEL`：インポートジョブのラベル、文字列型、インポートジョブのステータスをクエリするために使用できます。
   - `LOAD`ステートメント：ソースデータファイルのURI、ソースデータファイルの形式、およびターゲットテーブルの名前など、ジョブの説明情報を含みます。注意：`DATA INFILE`は、マウントパスを指定するために使用されます。上記の例では、`file:///`がプレフィックスで、`/home/disk1/sr`がマウントパスです。
```
   - `BROKER`：指定不要求。
   - `PROPERTIES`：オプションのジョブ属性としてタイムアウト時間などを指定するために使用します。

   詳細な構文とパラメーターの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

インポートジョブを送信した後、インポートの進行状況を確認したり、インポートジョブをキャンセルしたりすることができます。具体的な操作は、本文の「[Broker Loadのインポート進行状況を確認する](#查看-broker-load-导入进度)」および「[Broker Loadジョブをキャンセルする](#取消-broker-load-作业)」のセクションを参照してください。
