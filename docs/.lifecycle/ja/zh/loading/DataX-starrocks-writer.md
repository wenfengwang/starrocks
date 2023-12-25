---
displayed_sidebar: Chinese
---

# DataXを使用したインポート

この記事では、DataXを利用して、MySQL、OracleなどのデータベースからStarRocksにデータをインポートする方法について説明します。このプラグインは、データをCSVまたはJSON形式に変換し、[Stream Load](./StreamLoad.md)を使用してStarRocksにバッチでインポートします。

DataXインポートは多くのデータソースをサポートしています。詳細については、DataXの[Support Data Channels](https://github.com/alibaba/DataX#support-data-channels)を参照してください。

## 前提条件

DataXを使用してデータをインポートする前に、以下の依存関係がインストールされていることを確認してください：

- [DataX](https://github.com/alibaba/DataX/releases)
- [Python](https://www.python.org/downloads/)

## StarRocks Writerのインストール

StarRocks Writerのインストールパッケージを[ダウンロード](https://github.com/StarRocks/DataX/releases)し、以下のコマンドを実行して **datax/plugin/writer** ディレクトリに解凍します。

```shell
tar -xzvf starrockswriter.tar.gz
```

また、StarRocks Writerの[ソースコード](https://github.com/StarRocks/DataX)をコンパイルすることもできます。

## 設定ファイルの作成

インポートジョブ用のJSON形式の設定ファイルを作成します。

以下の例は、MySQLからStarRocksへのインポートジョブの設定ファイルを模擬しています。実際の使用シナリオに応じて、対応するパラメータを変更することができます。

```json
{
    "job": {
        "setting": {
            "speed": {
                 "channel": 1
            },
            "errorLimit": {
                "record": 0,
                "percentage": 0
            }
        },
        "content": [
            {
                "reader": {
                    "name": "mysqlreader",
                    "parameter": {
                        "username": "xxxx",
                        "password": "xxxx",
                        "column": [ "k1", "k2", "v1", "v2" ],
                        "connection": [
                            {
                                "table": [ "table1", "table2" ],
                                "jdbcUrl": [
                                     "jdbc:mysql://x.x.x.x:3306/datax_test1"
                                ]
                            },
                            {
                                "table": [ "table3", "table4" ],
                                "jdbcUrl": [
                                     "jdbc:mysql://x.x.x.x:3306/datax_test2"
                                ]
                            }
                        ]
                    }
                },
               "writer": {
                    "name": "starrockswriter",
                    "parameter": {
                        "username": "xxxx",
                        "password": "xxxx",
                        "database": "xxxx",
                        "table": "xxxx",
                        "column": ["k1", "k2", "v1", "v2"],
                        "preSql": [],
                        "postSql": [], 
                        "jdbcUrl": "jdbc:mysql://y.y.y.y:9030/",
                        "loadUrl": ["y.y.y.y:8030", "y.y.y.y:8030"],
                        "loadProps": {}
                    }
                }
            }
        ]
    }
}
```

`writer`セクションでは、以下のパラメータを設定する必要があります：

| **パラメータ**      | **説明**                                                     | **必須** | **デフォルト値** |
| ------------- | ------------------------------------------------------------ | -------- | ---------- |
| username      | StarRocksクラスタのユーザー名。<br />インポート操作には、対象テーブルのINSERT権限が必要です。ユーザーアカウントにINSERT権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)を参照してユーザーに権限を付与してください。                                       | はい       | なし         |
| password      | StarRocksクラスタのユーザーパスワード。                                     | はい       | なし         |
| database      | StarRocksの対象データベース名。                                   | はい       | なし         |
| table         | StarRocksの対象テーブル名。                                       | はい       | なし         |
| loadUrl       | StarRocks FEノードのアドレス。形式は `"fe_ip:fe_http_port"`。複数のアドレスはカンマ（,）で区切ります。 | はい       | なし         |
| column        | 対象テーブルに書き込むデータの列。複数の列はカンマ（,）で区切ります。例：`"column": ["id","name","age"]`。このパラメータは列名とは関係なく、順序に対応しています。readerのcolumnと同じにすることをお勧めします。 | はい       | なし         |
| preSql        | 標準SQL文。データが対象テーブルに書き込まれる**前**に実行されます。                | いいえ       | なし         |
| postSql       | 標準SQL文。データが対象テーブルに書き込まれた**後**に実行されます。                | いいえ       | なし         |
| jdbcUrl       | `preSql`および`postSql`の実行に使用される対象データベースのJDBC接続情報。 | いいえ       | なし         |
| maxBatchRows  | 一回のStream Loadインポートでの最大行数。大量のデータをインポートする場合、StarRocks Writerは`maxBatchRows`または`maxBatchSize`に基づいてデータを複数のStream Loadジョブに分割してバッチインポートします。 | いいえ       | 500000     |
| maxBatchSize  | 一回のStream Loadインポートでの最大バイト数。単位はByte。大量のデータをインポートする場合、StarRocks Writerは`maxBatchRows`または`maxBatchSize`に基づいてデータを複数のStream Loadジョブに分割してバッチインポートします。 | いいえ       | 104857600  |
| flushInterval | 前回のStream Loadが終了してから次回が開始するまでの時間間隔。単位はms。   | いいえ       | 300000     |
| loadProps     | Stream Loadのインポートパラメータ。詳細は[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。 | いいえ       | なし         |

> 注意
>
> - `column`パラメータは**空にできません**。すべての列をインポートしたい場合は、`"*"`と設定できます。
> - `writer`セクションの`column`の**数**は`reader`セクションと一致している必要があります。

### 特別なパラメータの設定

- **区切り文字の設定**

デフォルト設定では、データは文字列に変換され、CSV形式でStream Loadを介してStarRocksにインポートされます。文字列は`\t`を列の区切り文字として、`\n`を行の区切り文字として使用します。

`loadProps`パラメータに以下の設定を追加することで、区切り文字を変更できます：

```json
"loadProps": {
    "column_separator": "\\x01",
    "row_delimiter": "\\x02"
}
```

ここで、`\x`は16進数を表し、追加の`\`は`x`をエスケープするために使用されます。

> 説明
>
> StarRocksは、最大50バイトのUTF-8エンコード文字列を列の区切り文字として設定することをサポートしています。

- **インポート形式の設定**

現在のインポート方法のデフォルト形式はCSVです。`loadProps`パラメータに以下の設定を追加することで、データ形式をJSONに変更できます：

```json
"loadProps": {
    "format": "json",
    "strip_outer_array": true
}
```

- **派生列のインポート**

StarRocksのテーブルに元のデータをインポートする際に派生列も同時にインポートしたい場合は、`loadProps`に`columns`パラメータを追加設定する必要があります。

以下のコードスニペットは、元のデータ列`a`、`b`および派生列`c`をインポートする例を示しています。

```json
"writer": {
    "column": ["a", "b"],
    "loadProps": {
        "columns": "a,b,c=murmur_hash3_32(a)"
    }
    ...
}
```

ここで、`column`には元のデータの列名を記入し、`columns`には派生列名とそのデータ処理方法を追加記入する必要があります。

## インポートタスクの開始

設定ファイルを完成させた後、コマンドラインを通じてインポートタスクを開始する必要があります。

以下の例は、前述のステップでの設定ファイル**job.json**を基にインポートタスクを開始し、JVMのチューニングパラメータ（`--jvm="-Xms6G -Xmx6G"`）およびログレベル（`--loglevel=debug`）を設定しています。実際の使用シナリオに応じて、対応するパラメータを変更できます。

```shell
python datax/bin/datax.py --jvm="-Xms6G -Xmx6G" --loglevel=debug datax/job/job.json
```

ソースデータベースとターゲットデータベースのタイムゾーンが異なる場合は、コマンドラインに`-Duser.timezone=GMTxxx`オプションを追加してソースデータベースのタイムゾーン情報を設定する必要があります。

例えば、ソースデータベースがUTCタイムゾーンを使用している場合は、タスクを開始する際に`-Duser.timezone=GMT+0`パラメータを追加します。

## インポートタスクの状態を確認

DataXインポートはStream Loadをラップして実装されているため、`datax/log/$date/`ディレクトリで対応するインポートジョブのログを検索できます。ログファイル名には、使用されたJSONファイル名とタスク開始時間（時、分、秒）が含まれています。例：**t_datax_job_job_json-20_52_19.196.log**。

- ログに"http://fe_ip:fe_http_port/api/db_name/tbl_name/_stream_load"の記録がある場合は、Stream Loadジョブが正常にトリガーされたことを意味します。ジョブが正常にトリガーされた後、[Stream Loadの戻り値](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#戻り値)を参照してタスクの状況を確認できます。
- 上記の情報がログにない場合は、エラーメッセージに基づいて問題を調査するか、[DataXコミュニティの問題](https://github.com/alibaba/DataX/issues)で解決策を探してください。

> 注意
>
> DataXインポートを使用すると、ターゲットテーブルの形式に合わないデータが存在する場合、StarRocksでそのフィールドがNULL値を許可する場合、不正なデータはNULLに変換されて正常にインポートされます。フィールドがNULL値を許可せず、タスクに容認率（max_filter_ratio）が設定されていない場合、タスクはエラーを報告し、インポートジョブは中断されます。

## インポートタスクをキャンセルまたは停止

DataXのPythonプロセスを終了することでインポートタスクを停止できます。

```shell
pkill datax
```
