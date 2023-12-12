---
displayed_sidebar: "Japanese"
---

# その他のFAQ

このトピックは、一般的な質問に対する回答を提供します。

## VARCHAR（32）とSTRINGは同じストレージスペースを占有するか？

どちらも可変長データ型です。同じ長さのデータを格納する場合、VARCHAR（32）とSTRINGは同じストレージスペースを占有します。

## VARCHAR（32）とSTRINGはデータクエリで同じ動作をしますか？

はい。

## OracleからインポートされたTXTファイルは、文字セットをUTF-8に設定した後でも文字化けする理由は？

この問題を解決するには、次の手順を実行します。

1. 例として、テキストが文字化けしている **original** というファイルがあるとします。このファイルの文字セットはISO-8859-1です。次のコードを実行してファイルの文字セットを取得します。

    ```plaintext
    file --mime-encoding origin.txt
    origin.txt: iso-8859-1
    ```

2. `iconv` コマンドを使用して、このファイルの文字セットをUTF-8に変換します。

    ```plaintext
    iconv -f iso-8859-1 -t utf-8 origin.txt > origin_utf-8.txt
    ```

3. 変換後、このファイルのテキストがまだ文字化けしている場合は、ファイルの文字セットをGBKに戻し、その後再度UTF-8に変換します。

    ```plaintext
    iconv -f gbk -t utf-8 origin.txt > origin_utf-8.txt
    ```

## MySQLでのSTRINGの長さの定義は、StarRocksでの定義と同じですか？

VARCHAR(n)の場合、StarRocksは "n" をバイト単位で定義し、MySQLは "n" を文字単位で定義します。UTF-8に従うと、1つの中国語の文字は3バイトに相当します。 StarRocksとMySQLが同じ数値で "n" を定義すると、MySQLはStarRocksの3倍の文字を保存します。

## テーブルのパーティションフィールドのデータ型をFLOAT、DOUBLE、またはDECIMALにすることはできますか？

いいえ、DATE、DATETIME、およびINTのみがサポートされています。

## テーブル内のデータが占有するストレージスペースをチェックする方法は？

SHOW DATA ステートメントを実行して対応するストレージスペースを確認します。データ量、コピー数、および行数も表示できます。

**注**: データ統計には時間の遅延があります 。

## StarRocksデータベースのクォータ増加をリクエストする方法は？

クォータを増やすには、次のコードを実行します。

```plaintext
ALTER DATABASE example_db SET DATA QUOTA 10T;
```

## StarRocksは、UPSERTステートメントを使用してテーブル内の特定のフィールドを更新できますか？

StarRocks 2.2以降では、プライマリキー・テーブルを使用してテーブル内の特定のフィールドを更新することができます。StarRocks 1.9以降では、プライマリキー・テーブルを使用してテーブル内のすべてのフィールドを更新することができます。詳細については、StarRocks 2.2の[プライマリキー・テーブル](../table_design/table_types/primary_key_table.md) を参照してください。

## 2つのテーブルまたは2つのパーティションのデータを交換する方法は？

SWAP WITH ステートメントを実行して、2つのテーブルまたは2つのパーティション間でデータを交換します。SWAP WITH ステートメントは、INSERT OVERWRITE ステートメントよりも安全です。データを交換する前に、まずデータを確認し、交換後のデータが交換前のデータと一致するかどうかを確認します。

- 2つのテーブルの交換：例として、table1という名前のテーブルがあるとします。table1を別のテーブルに置き換えたい場合は、次の手順を実行します。

    1. table1と同じ構造のtable2を作成します。

        ```SQL
        create table2 like table1;
        ```

    2. Stream Load、Broker Load、またはInsert Intoを使用して、table1からtable2にデータをロードします。

    3. table1をtable2で置き換えます。

        ```SQL
        ALTER TABLE table1 SWAP WITH table2;
        ```

        これにより、データが正確にtable1にロードされます。

- 2つのパーティションの交換：例として、table1という名前のテーブルがあるとします。テーブル1内のパーティションデータを置き換えたい場合は、次の手順を実行します。

    1. 一時的なパーティションを作成します。

        ```SQL
        ALTER TABLE table1

        ADD TEMPORARY PARTITION tp1

        VALUES LESS THAN("2020-02-01");
        ```

    2. table1から一時的なパーティションにパーティションデータをロードします。

    3. table1のパーティションを一時的なパーティションと置き換えます。

        ```SQL
        ALTER TABLE table1

        REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
        ```

## フロントエンド（FE）を再起動すると、"error to open replicated environment, will exit" のエラーが発生します

このエラーはBDBJEのバグに起因します。この問題を解決するには、BDBJEのバージョンを1.17以上に更新します。

## 新しいApache Hiveテーブルからデータをクエリすると "Broker list path exception" エラーが発生します

### 問題の詳細

```plaintext
msg:Broker list path exception

path=hdfs://172.31.3.136:9000/user/hive/warehouse/zltest.db/student_info/*, broker=TNetworkAddress(hostname:172.31.4.233, port:8000)
```

### 解決策

StarRocksの技術サポートに連絡し、namenodeのアドレスとポートが正しいかどうか、およびnamenodeのアドレスとポートにアクセスする権限があるかどうかを確認します。

## 新しいApache Hiveテーブルからデータをクエリすると "get hive partition metadata failed" エラーが発生します

### 問題の詳細

```plaintext
msg:get hive partition meta data failed: java.net.UnknownHostException: emr-header-1.cluster-242
```

### 解決策

ネットワークが接続されていることを確認し、**host**ファイルをStarRocksクラスタ内のすべてのバックエンド（BE）にアップロードします。

## Apache Hiveの外部テーブルにアクセスする際に "do_open failed. reason = Invalid ORC postscript length" エラーが発生します

### 問題の詳細

Apache HiveのメタデータはFEにキャッシュされます。しかし、StarRocksがメタデータを更新するには2時間の時間差があります。StarRocksが更新を完了する前に、Apache Hiveテーブルに新しいデータを挿入したり、データを更新したりすると、BEがスキャンしたHDFSのデータとFEが取得したデータが異なるため、このエラーが発生します。

```plaintext
MySQL [bdp_dim]> select * from dim_page_func_s limit 1;

ERROR 1064 (HY000): HdfsOrcScanner::do_open failed. reason = Invalid ORC postscript length
```

### 解決策

この問題を解決するには、次のいずれかの操作を実行します。

- 現在のバージョンをStarRocks 2.2またはそれ以降にアップグレードします。
- Apache Hiveテーブルを手動で更新します。詳細については、[メタデータのキャッシュ戦略](../data_source/External_table.md) を参照してください。

## MySQLの外部テーブルに接続すると "caching_sha2_password cannot be loaded" エラーが発生します

### 問題の詳細

MySQL 8.0のデフォルト認証プラグインはcaching_sha2_passwordです。MySQL 5.7のデフォルト認証プラグインはmysql_native_passwordです。このエラーは、認証プラグインを間違えて使用したためです。

### 解決策

この問題を解決するには、次のいずれかの操作を実行します。

- StarRocksに接続します。

```SQL
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
```

- `my.cnf` ファイルを修正します。

```plaintext
vim my.cnf

[mysqld]

default_authentication_plugin=mysql_native_password
```

## テーブルを削除した後、ディスクスペースをすぐに解放する方法は？

DROP TABLE ステートメントを実行してテーブルを削除すると、StarRocksは割り当てられたディスクスペースを解放するまでに時間がかかります。割り当てられたディスクスペースをすぐに解放するには、DROP TABLE FORCE ステートメントを実行してテーブルを削除します。DROP TABLE FORCE ステートメントを実行すると、StarRocksは未完了のイベントがあるかどうかを確認せずにテーブルを直接削除します。DROP TABLE FORCE ステートメントを注意して実行することをお勧めします。テーブルが削除されると、それを復元することはできません。

## StarRocksの現在のバージョンを表示する方法は？

`select current_version();` コマンドを実行するか、CLIコマンド `./bin/show_fe_version.sh` を実行して現在のバージョンを表示します。

## FEのメモリサイズを設定する方法は？

メタデータはFEが使用するメモリに保存されます。FEの数に応じてFEのメモリサイズを設定できます。例えば、テーブレットの数が100万未満の場合、FEに最低16GBのメモリを割り当てる必要があります。**fe.conf**ファイルの**JAVA_OPTS**構成項目の**-Xms**および**-Xmx**の値を設定し、**-Xms**および**-Xmx**の値を一致させる必要があります。また、任意のFEがリーダーになる可能性があるため、すべてのFEで同じ構成にする必要があります。

| テーブレットの数    | 各FEのメモリサイズ |
| -------------- | ----------- |
| 100万未満      | 16GB        |
| 100万 ～ 200万 | 32GB        |
| 200万 ～ 500万 | 64GB        |
| 500万 ～ 1000万   | 128GB       |
## StarRocksはクエリ時間をどのように計算しますか？

StarRocksは複数のスレッドを使用してデータをクエリすることをサポートしています。クエリ時間とは、複数のスレッドがデータをクエリするために使用される時間を指します。

## StarRocksはローカルにデータをエクスポートする際にパスの設定をサポートしていますか？

いいえ。

## StarRocksの並行性の上限はどのくらいですか？

実際のビジネスシナリオまたはシミュレートされたビジネスシナリオに基づいて、並行性の制限をテストできます。一部のユーザーのフィードバックによると、最大で20,000 QPSまたは30,000 QPSが達成できます。

## StarRocksの初回のSSBテストパフォーマンスが2回目よりも遅いのはなぜですか？

最初のクエリでディスクを読み取る速度はディスクのパフォーマンスに関連しています。初回のクエリの後、次のクエリのためにページキャッシュが生成されるため、クエリは以前よりも速くなります。

## クラスターには最低で何台のBEが構成される必要がありますか？

StarRocksは単一ノードの展開をサポートしているため、少なくとも1つのBEを構成する必要があります。BEはAVX2で実行する必要があるため、8コアと16GB以上の構成のマシンにBEを展開することをお勧めします。

## Apache Supersetを使用してStarRocksでデータを可視化する際のデータ権限の設定方法は？

新しいユーザーアカウントを作成し、その後、そのユーザーに対してテーブルクエリの権限を付与することでデータ権限を設定できます。

## `enable_profile`を`true`に設定した後にプロファイルが表示されないのはなぜですか？

レポートはリーダーFEにのみ提出されます。

## StarRocksのテーブルのフィールド注釈を確認する方法は？

`show create table xxx` コマンドを実行します。

## テーブルを作成する際、NOW()関数のデフォルト値を指定する方法は？

StarRocks 2.1以降のバージョンのみが関数のデフォルト値を指定することをサポートしています。StarRocks 2.1より前のバージョンでは、関数のために定数のみを指定できます。

## BEノードのストレージスペースを解放する方法は？

`rm -rf`コマンドを使用して`trash`ディレクトリを削除できます。スナップショットからデータを復元済みの場合は、`snapshot`ディレクトリを削除できます。

## BEノードに追加のディスクを追加できますか？

はい。BE構成項目`storage_root_path`で指定されたディレクトリにディスクを追加できます。