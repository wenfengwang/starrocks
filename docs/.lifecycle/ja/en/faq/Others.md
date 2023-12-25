---
displayed_sidebar: English
---

# その他のFAQ

このトピックでは、いくつかの一般的な質問に対する回答を示します。

## VARCHAR (32) と STRING は同じストレージスペースを占有しますか?

どちらも可変長データ型です。同じ長さのデータを保管する場合、VARCHAR (32) と STRING は同じストレージスペースを占有します。

## VARCHAR (32) と STRING はデータクエリにおいて同じパフォーマンスを発揮しますか?

はい。

## Oracle からインポートした TXT ファイルが、文字セットを UTF-8 に設定した後も文字化けして表示されるのはなぜですか?

この問題を解決するには、次の手順を実行します。

1. たとえば、**original** という名前のファイルがあり、そのテキストが文字化けしています。このファイルの文字セットは ISO-8859-1 です。次のコードを実行して、ファイルの文字セットを取得します。

    ```plaintext
    file --mime-encoding origin.txt
    origin.txt: iso-8859-1
    ```

2. `iconv` コマンドを実行して、このファイルの文字セットを UTF-8 に変換します。

    ```plaintext
    iconv -f iso-8859-1 -t utf-8 origin.txt > origin_utf-8.txt
    ```

3. 変換後も、このファイルのテキストは文字化けして表示されます。その後、このファイルの文字セットをGBKとして再評価し、文字セットをUTF-8に再度変換します。

    ```plaintext
    iconv -f gbk -t utf-8 origin.txt > origin_utf-8.txt
    ```

## MySQLで定義されるSTRINGの長さは、StarRocksで定義されるものと同じですか?

VARCHAR(n) の場合、StarRocks は "n" をバイト単位で、MySQL は "n" を文字単位で定義します。UTF-8によると、1つの漢字は3バイトに相当します。StarRocks と MySQL が "n" を同じ数値で定義した場合、MySQLはStarRocksよりも3倍多くの文字を保存できます。

## テーブルのパーティションフィールドのデータタイプに FLOAT、DOUBLE、または DECIMAL を使用できますか?

いいえ、DATE、DATETIME、および INT のみがサポートされています。

## テーブル内のデータが占有するストレージスペースを確認するにはどうすればよいですか?

SHOW DATA ステートメントを実行して、対応するストレージスペースを確認します。また、データ量、レプリカ数、行数も確認できます。

**注**: データ統計にはタイムラグがあります。

## StarRocksデータベースのクォータを増やすリクエストはどのように行いますか?

クォータを増やすリクエストをするには、次のコードを実行します。

```plaintext
ALTER DATABASE example_db SET DATA_QUOTA 10T;
```

## StarRocksは、UPSERTステートメントを実行してテーブル内の特定フィールドを更新することをサポートしていますか?

StarRocks 2.2 以降は、プライマリキーテーブルを使用してテーブル内の特定フィールドを更新することをサポートしています。StarRocks 1.9 以降は、プライマリキーテーブルを使用してテーブル内の全フィールドを更新することをサポートしています。詳細については、StarRocks 2.2 の[プライマリキーテーブル](../table_design/table_types/primary_key_table.md)を参照してください。

## 2つのテーブルまたは2つのパーティション間でデータを交換するにはどうすればよいですか?

SWAP WITH ステートメントを実行して、2つのテーブルまたは2つのパーティション間でデータを交換します。SWAP WITH ステートメントは、INSERT OVERWRITE ステートメントよりも安全です。データを交換する前に、まずデータを確認し、交換後のデータが交換前のデータと一致するかどうかを確認してください。

- 2つのテーブルを交換する: たとえば、table1 という名前のテーブルがあるとします。table1 を別のテーブルと交換したい場合は、次の手順を実行します。

    1. table2 という名前の新しいテーブルを作成します。

        ```SQL
        CREATE TABLE table2 LIKE table1;
        ```

    2. Stream Load、Broker Load、または INSERT INTO を使用して、table1 から table2 へデータをロードします。

    3. table1 を table2 と交換します。

        ```SQL
        ALTER TABLE table1 SWAP WITH table2;
        ```

        これにより、データは正確に table1 にロードされます。

- 2つのパーティションを交換する: たとえば、table1 という名前のテーブルがあるとします。table1 のパーティションデータを交換したい場合は、以下の手順を実行します。

    1. 一時パーティションを作成します。

        ```SQL
        ALTER TABLE table1
        ADD TEMPORARY PARTITION tp1
        VALUES LESS THAN("2020-02-01");
        ```

    2. table1 のパーティションデータを一時パーティションにロードします。

    3. table1 のパーティションを一時パーティションと交換します。

        ```SQL
        ALTER TABLE table1
        REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
        ```

## このエラー「レプリケーション環境を開く際のエラー、終了します」は、フロントエンド(FE)を再起動したときに発生します

このエラーは、BDBJEのバグによって発生します。この問題を解決するには、BDBJEのバージョンを1.17以降にアップデートしてください。

## このエラー「ブローカーリストパスの例外」は、新しいApache Hiveテーブルからデータをクエリする際に発生します

### 問題の説明

```plaintext
msg:Broker list path exception

path=hdfs://172.31.3.136:9000/user/hive/warehouse/zltest.db/student_info/*, broker=TNetworkAddress(hostname:172.31.4.233, port:8000)
```

### 解決策

StarRocksのテクニカルサポートに連絡し、ネームノードのアドレスとポートが正しいかどうか、およびネームノードのアドレスとポートへのアクセス権限があるかどうかを確認してください。

## このエラー「Hiveパーティションメタデータの取得に失敗しました」は、新しいApache Hiveテーブルからデータをクエリする際に発生します

### 問題の説明

```plaintext
msg:get hive partition metadata failed: java.net.UnknownHostException: emr-header-1.cluster-242
```

### 解決策


ネットワークが接続されていることを確認し、各バックエンド(BE)に**host**ファイルをアップロードしてください。StarRocksクラスタにおいて。

## このエラー「do_open failed. reason = Invalid ORC postscript length」がApache HiveのORC外部テーブルにアクセスする際に発生します

### 問題の説明

Apache HiveのメタデータはFEにキャッシュされています。しかし、StarRocksがメタデータを更新するまでには2時間のタイムラグがあります。StarRocksが更新を完了する前に、Apache Hiveテーブルに新しいデータを挿入したり、データを更新したりすると、BEがスキャンするHDFSのデータとFEが取得するデータが異なるため、このエラーが発生します。

```plaintext
MySQL [bdp_dim]> select * from dim_page_func_s limit 1;

ERROR 1064 (HY000): HdfsOrcScanner::do_open failed. reason = Invalid ORC postscript length
```

### 解決策

この問題を解決するには、以下のいずれかの操作を行ってください：

- 現在のバージョンをStarRocks 2.2以降にアップグレードする。
- Apache Hiveテーブルを手動でリフレッシュする。詳細は[メタデータキャッシング戦略](../data_source/External_table.md)を参照してください。

## このエラー「caching_sha2_password cannot be loaded」がMySQLの外部テーブルに接続する際に発生します

### 問題の説明

MySQL 8.0のデフォルト認証プラグインはcaching_sha2_passwordです。MySQL 5.7のデフォルト認証プラグインはmysql_native_passwordです。誤った認証プラグインを使用すると、このエラーが発生します。

### 解決策

この問題を解決するには、以下のいずれかの操作を行ってください：

- StarRocksに接続する。

```SQL
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
```

- `my.cnf`ファイルを編集する。

```plaintext
vim my.cnf

[mysqld]

default_authentication_plugin=mysql_native_password
```

## テーブルを削除した後、直ちにディスクスペースを解放するにはどうすればいいですか？

DROP TABLEステートメントでテーブルを削除すると、StarRocksは割り当てられたディスクスペースの解放に時間がかかります。割り当てられたディスクスペースを直ちに解放するためには、DROP TABLE FORCEステートメントを実行してテーブルを削除してください。DROP TABLE FORCEステートメントを実行すると、StarRocksはテーブル内の未完了イベントを確認せずに直接テーブルを削除します。テーブルを一度削除すると復元できないため、DROP TABLE FORCEステートメントの実行には注意してください。

## StarRocksの現在のバージョンを確認するにはどうすればよいですか？

`select current_version();`コマンドまたはCLIコマンド`./bin/show_fe_version.sh`を実行して、現在のバージョンを確認します。

## FEのメモリサイズを設定する方法は？

メタデータはFEが使用するメモリに格納されます。以下の表に示すように、タブレットの数に応じてFEのメモリサイズを設定できます。例えば、タブレットの数が100万未満の場合、FEには最低16GBのメモリを割り当てるべきです。`fe.conf`ファイルの**JAVA_OPTS**設定項目でパラメータ`-Xms`と`-Xmx`の値を設定し、これらのパラメータの値は一致している必要があります。どのFEもリーダーに選出される可能性があるため、すべてのFEで設定が同じであることが重要です。

| タブレットの数    | 各FEのメモリサイズ |
| -------------- | ----------- |
| 100万未満      | 16 GB        |
| 100万～200万 | 32 GB        |
| 200万～500万 | 64 GB        |
| 500万～1000万   | 128 GB       |

## StarRocksはクエリ時間をどのように計算しますか？

StarRocksは、複数のスレッドを使用してデータをクエリすることをサポートしています。クエリ時間は、複数のスレッドがデータをクエリするのに使用する時間です。

## StarRocksは、ローカルにデータをエクスポートする際にパスを設定することをサポートしていますか？

いいえ。

## StarRocksの同時実行の上限はいくつですか？

実際のビジネスシナリオまたはシミュレートされたビジネスシナリオに基づいて、同時実行の制限をテストすることができます。一部のユーザーからのフィードバックによると、最大で20,000 QPSまたは30,000 QPSを達成できるとされています。

## StarRocksで最初に実行したSSBテストのパフォーマンスが2回目よりも遅いのはなぜですか？

最初のクエリでのディスク読み取り速度は、ディスクのパフォーマンスに依存します。最初のクエリの後、後続のクエリ用にページキャッシュが生成されるため、クエリは以前よりも速くなります。

## クラスタに最低限設定する必要があるBEの数はいくつですか？

StarRocksはシングルノードデプロイメントをサポートしているため、最低1つのBEを設定する必要があります。BEはAVX2で実行する必要があるため、8コア16GB以上の構成のマシンにBEをデプロイすることを推奨します。

## Apache Supersetを使用してStarRocksのデータを視覚化する際にデータ権限を設定するにはどうすればよいですか？

新しいユーザーアカウントを作成し、そのユーザーにテーブルクエリの権限を付与することでデータ権限を設定できます。

## `enable_profile`を`true`に設定した後、プロファイルが表示されないのはなぜですか？

レポートはアクセスのためにリーダーFEにのみ提出されます。

## StarRocksのテーブルでフィールドの注釈を確認するにはどうすればよいですか？

`show create table xxx`コマンドを実行します。

## テーブルを作成する際、NOW()関数のデフォルト値を指定するにはどうすればよいですか？

StarRocks 2.1以降のバージョンでは、関数のデフォルト値を指定することがサポートされています。StarRocks 2.1より前のバージョンでは、関数には定数のみを指定できます。

## BEノードのストレージスペースを解放するにはどうすればいいですか?

`rm -rf` コマンドを使用して `trash` ディレクトリを削除できます。スナップショットからデータを既に復元している場合は、`snapshot` ディレクトリを削除できます。

## BEノードに追加ディスクを加えることはできますか?

はい。BEの設定項目 `storage_root_path` で指定されたディレクトリにディスクを追加することができます。
