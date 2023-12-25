---
displayed_sidebar: Chinese
---

# その他

この記事では、StarRocksの使用時によくあるその他の問題についてまとめています。

## VARCHAR(32)とSTRINGは同じストレージスペースを占有しますか？

VARCHAR(32)とSTRINGはどちらも可変長のデータ型です。同じ長さのデータを格納する場合、VARCHAR(32)とSTRINGは同じストレージスペースを占有します。

## クエリ時、VARCHAR(32)とSTRINGのパフォーマンスは同じですか？

同じです。

## OracleからエクスポートしたTXTファイルをUTF-8に変換しても文字化けが発生する場合、どのように処理すればよいですか？

ファイルの文字セットをGBKとして文字セット変換を行います。手順は以下の通りです：

1. 例えば、**origin**という名前のファイルが文字化けしている場合、以下のコマンドでその文字セットがISO-8859-1であることがわかります。

    ```Plain_Text
    file --mime-encoding origin.txt
    origin.txt：iso-8859-1
    ```

2. `iconv`コマンドを使用してファイルの文字セットをUTF-8に変換します。

    ```Plain_Text
    iconv -f iso-8859-1 -t utf-8 origin.txt > origin_utf-8.txt
    ```

3. 変換後のファイルがまだ文字化けしている場合、ファイルの文字セットをGBKとしてUTF-8に変換します。

    ```Shell
    iconv -f gbk -t utf-8 origin.txt > origin_utf-8.txt
    ```

## MySQLで定義された文字列の長さはStarRocksで定義されたものと同じですか？

StarRocksでは、VARCHAR(n)のnはバイト数を表し、MySQLではVARCHAR(n)のnは文字数を表します。UTF-8に基づいて、1つの漢字は3バイトに相当します。StarRocksとMySQLが同じ数字nを定義する場合、MySQLはStarRocksの3倍の文字数を保存します。

## テーブルのパーティションフィールドにはFLOAT、DOUBLE、またはDECIMALデータ型を使用できますか？

できません。DATE、DATETIME、およびINTのデータ型のみサポートされています。

## テーブルのデータがどれだけのストレージを占有しているかを確認するにはどうすればよいですか？

SHOW DATAステートメントを実行して、データのストレージスペース、データ量、レプリカ数、および行数を確認します。

> 注意：データのインポートはリアルタイムではなく、インポート後約1分で最新のデータを確認できます。

## StarRocksデータベースのクォータ（quota）を調整するにはどうすればよいですか？

次のコードを実行してデータベースのクォータを調整します：

```SQL
ALTER DATABASE example_db SET DATA QUOTA 10T;
```

## StarRocksはUPSERT構文を使用して一部のフィールドを更新できますか？

StarRocksのバージョン2.2以降では、主キー（Primary Key）モデルを使用して一部のフィールドを更新することができます。StarRocksのバージョン1.9以降では、主キー（Primary Key）モデルを使用してすべてのフィールドを更新することができます。詳細については、StarRocks 2.2バージョンの[主キーモデル](../table_design/table_types/primary_key_table.md)を参照してください。

## 原子的なテーブルおよびパーティションの置換機能を使用する方法は？

SWAP WITHステートメントを実行して、原子的なテーブルおよびパーティションの置換機能を実現します。SWAP WITHステートメントはINSERT OVERWRITEステートメントよりも安全です。置換前にデータをチェックして、置換後のデータと置換前のデータが同じかどうかを確認することができます。

- テーブルの原子的な置換：例えば、`table1`という名前のテーブルを別のテーブルで原子的に置換する場合、以下の手順を実行します：

    1. `table1`と同じ構造の新しいテーブル`table2`を作成します。

        ```SQL
        create table2 like table1;
        ```

    2. Stream Load、Broker Load、またはInsert Intoなどの方法を使用して、`table1`のデータを新しいテーブル`table2`にインポートします。
    3. `table1`と`table2`を原子的に置換します。

        ```SQL
        ALTER TABLE table1 SWAP WITH table2;
        ```

    これにより、データが`table1`に正確にインポートされます。

- パーティションの原子的な置換：例えば、`table1`という名前のテーブルのパーティションデータを原子的に置換する場合、以下の手順を実行します：

    1. 一時的なパーティションを作成します。

        ```SQL
        ALTER TABLE table1

        ADD TEMPORARY PARTITION tp1

        VALUES LESS THAN("2020-02-01");
        ```

    2. `table1`のパーティションデータを一時的なパーティションにインポートします。
    3. パーティションを原子的に置換します。

        ```SQL
        ALTER TABLE table1

        REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
        ```

## FEの再起動時に「error to open replicated environment, will exit」というエラーが発生する

このエラーはBDBJEのバグによるものであり、BDBJEを1.17以上のバージョンにアップグレードすることで修正できます。

## 新しく作成したApache Hive™テーブルをクエリする際に「Broker list path exception」というエラーが発生する

### **問題の説明**

```Plain_Text
msg:Broker list path exception

path=hdfs://172.31.3.136:9000/user/hive/warehouse/zltest.db/student_info/*, broker=TNetworkAddress(hostname:172.31.4.233, port:8000)
```

### **解決策**

StarRocksのサポートチームと連絡し、namenodeのアドレスとポートが正しいか、およびnamenodeのアドレスとポートにアクセスする権限があるかを確認してください。

## 新しく作成したApache Hiveテーブルをクエリする際に「get hive™ partition meta data failed」というエラーが発生する

### **問題の説明**

```Plain_Text
msg:get hive partition meta data failed: java.net.UnknownHostException: emr-header-1.cluster-242
```

### **解決策**

ネットワーク接続があることを確認し、クラスタ内のすべてのBEマシンにクラスタの**host**ファイルをコピーしてください。

## Apache Hive™のORC外部テーブルにアクセスする際に「do_open failed.reason = Invalid ORC postscript length」というエラーが発生する

### **問題の説明**

Apache Hive™のメタデータはStarRocksのFEにキャッシュされますが、StarRocksはメタデータの更新に2時間の遅延があります。そのため、StarRocksが更新を完了する前にApache Hive™テーブルに新しいデータを挿入または更新すると、BEがスキャンするHDFSのデータとFEが取得するデータが一致しないため、このエラーが発生します。

```Plain_Text
MySQL [bdp_dim]> select * from dim_page_func_s limit 1;

ERROR 1064 (HY000): HdfsOrcScanner::do_open failed. reason = Invalid ORC postscript length
```

### **解決策**

解決策は以下の2つあります：

- StarRocksをバージョン2.2以上にアップグレードします。
- Apache Hive™テーブルを手動でリフレッシュします。詳細については、[キャッシュの更新](../data_source/External_table.md#手動更新元データキャッシュ)を参照してください。

## MySQL外部テーブルに接続する際に「caching_sha2_password cannot be loaded」というエラーが発生する

### **問題の説明**

MySQL 5.7のデフォルトの認証方式はmysql_native_passwordですが、MySQL 8.0のデフォルトの認証方式であるcaching_sha2_passwordを使用して認証しようとすると、接続エラーが発生します。

### **解決策**

以下の2つの解決策があります：

- rootユーザーを設定します。

    ```Plain_Text
    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
    ```

- **my.cnf**ファイルを編集します。

    ```Plain_Text
    vim my.cnf

    [mysqld]

    default_authentication_plugin=mysql_native_password
    ```

## テーブルを削除した後、ディスクスペースがすぐに解放されないのはなぜですか？

DROP TABLEステートメントを実行してテーブルを削除した後、ディスクスペースが解放されるまで待つ必要があります。ディスクスペースをすぐに解放したい場合は、DROP TABLE FORCEステートメントを使用できます。DROP TABLE FORCEステートメントを使用してテーブルを削除すると、テーブルに未完了のトランザクションが存在するかどうかをチェックせずに直接テーブルを削除します。DROP TABLE FORCEステートメントの使用は慎重に行ってください。このステートメントで削除されたテーブルは復元できません。

## StarRocksのバージョンを確認する方法は？

`select current_version();`コマンドまたはCLI `sh bin/show_fe_version.sh`コマンドを実行してバージョンを確認できます。

## FEのメモリサイズを設定する方法は？

メタデータ情報はすべてFEのメモリに保存されます。FEのメモリサイズは、以下の表を参考にして、Tabletの数に基づいて設定することができます。例えば、Tabletの数が100万以下の場合、FEには少なくとも16 GBのメモリを割り当てる必要があります。FEのメモリサイズを設定するには、**fe.conf**ファイルの`JAVA_OPTS`で`-Xms`と`-Xmx`パラメータを設定し、両方の値を同じにします。注意：クラスタ内のすべてのFEに統一した設定を行う必要があります。なぜなら、各FEがリーダーになる可能性があるためです。

| Tabletの数    | FEのメモリサイズ |
| -------------- | ----------- |
| 100万以下     | 16 GB        |
| 100万 ～ 200万 | 32 GB        |
| 200万 ～ 500万 | 64 GB        |
| 500万 ～ 1千万   | 128 GB       |

## StarRocksはクエリ時間をどのように計算しますか？

StarRocksはマルチスレッド計算です。クエリ時間は、最も遅いスレッドの実行時間です。

## StarRocksはデータをローカルにエクスポートする際にパスを設定できますか？

サポートされていません。

## StarRocksの同時接続数はどのくらいですか？

StarRocksの同時接続数は、ビジネスシナリオやビジネスシナリオのシミュレーションに基づいてテストすることをお勧めします。一部の顧客の同時接続数は最大で20,000 QPSまたは30,000 QPSに達することがあります。

## StarRocksのSSBテストの初回実行速度が遅く、2回目以降の実行速度が速いのはなぜですか？

初回のクエリはディスクアクセスに関連しており、初回のクエリ後にシステムのページキャッシュが有効になります。そのため、2回目以降のクエリはまずページキャッシュをスキャンするため、速度が向上します。

## クラスタには最低いくつのBEを構成することができますか？

StarRocksは単一ノードの展開をサポートしているため、BEの最小構成数は1です。BEはAVX2命令セットをサポートする必要があるため、BEをデプロイするマシンの構成は8コア16 GB以上を推奨します。通常のアプリケーション環境では、3つのBEを構成することをお勧めします。

## Apache Supersetフレームワークを使用してStarRocksのデータを表示する際に、データのアクセス権をどのように設定しますか？

新しいユーザーを作成し、そのユーザーにテーブルのクエリ権限（SELECT）を付与することで、データのアクセス権を制御することができます。

## `enable_profile`を`true`に設定すると、プロファイルが表示されなくなるのはなぜですか？

プロファイル情報は主FEにのみ報告されるため、プロファイル情報を表示できるのは主FEだけです。また、StarRocks Managerでプロファイルを表示する場合は、FEの設定項目`enable_collect_query_detail_info`が`true`に設定されていることを確認してください。

## StarRocksのテーブルのフィールドコメントを表示する方法は？

`show create table xxx`コマンドを使用してフィールドのコメントを表示することができます。

## テーブルの作成時にnow()関数のデフォルト値を指定できますか？

StarRocksのバージョン2.1以降では、関数にデフォルト値を指定することができます。StarRocksのバージョン2.1未満では、関数に定数を指定することしかサポートされていません。

## StarRocksの外部テーブルの同期エラーを解決する方法は？

**問題のヒント**：

SQLエラー[1064] [42000]: テーブルに空のパーティションにデータを挿入することはできません。このテーブルの現在のパーティションを表示するには、`SHOW PARTITIONS FROM external_t`を使用します。

パーティションを表示すると、別のエラーが表示されます：SHOW PARTITIONS FROM external_t
SQLエラー[1064] [42000]: テーブル[external_t]はOLAP/ELASTICSEARCH/HIVEテーブルではありません。

**解決策**：

問題は、外部テーブルの作成時に間違ったポートが指定されていたためです。正しいポートは"port"="9020"であり、9931ではありません。

## ディスクのストレージスペースが不足している場合、使用可能なスペースを解放する方法は？

`rm -rf`コマンドを使用して、`trash`ディレクトリ内のコンテンツを直接削除することができます。データのバックアップの復元が完了した後、`snapshot`ディレクトリ内のコンテンツを削除することで、ストレージスペースを解放することもできます。

## ディスクのストレージスペースが不足している場合、どのようにディスクスペースを拡張しますか？

BEノードのストレージスペースが不足している場合は、BEの設定項目`storage_root_path`に対応するディレクトリに直接ディスクを追加することができます。
