---
displayed_sidebar: "Japanese"
---

# DataXライター

## はじめに

StarRocksWriterプラグインを使用すると、StarRocksの宛先テーブルにデータを書き込むことができます。具体的には、StarRocksWriterは[Stream Load](./StreamLoad.md)を介してCSVまたはJSON形式でデータをStarRocksにインポートし、`reader`によって読み取られたデータをStarRocksに内部的にキャッシュし、バルクインポートして書き込みパフォーマンスを向上させます。データの全体的なフローは `source -> Reader -> DataX channel -> Writer -> StarRocks` です。

[プラグインをダウンロードする](https://github.com/StarRocks/DataX/releases)

DataXの完全なパッケージをダウンロードするには、`https://github.com/alibaba/DataX` にアクセスし、starrockswriterプラグインを `datax/plugin/writer/` ディレクトリに配置してください。

次のコマンドを使用してテストしてください：
`python datax.py --jvm="-Xms6G -Xmx6G" --loglevel=debug job.json`

## 関数の説明

### サンプル構成

以下は、MySQLからデータを読み取り、StarRocksにロードするための構成ファイルです。

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
                                     "jdbc:mysql://127.0.0.1:3306/datax_test1"
                                ]
                            },
                            {
                                "table": [ "table3", "table4" ],
                                "jdbcUrl": [
                                     "jdbc:mysql://127.0.0.1:3306/datax_test2"
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
                        "jdbcUrl": "jdbc:mysql://172.28.17.100:9030/",
                        "loadUrl": ["172.28.17.100:8030", "172.28.17.100:8030"],
                        "loadProps": {}
                    }
                }
            }
        ]
    }
}

```

## Starrockswriterパラメータの説明

* **username**

  * 説明: StarRocksデータベースのユーザー名

  * 必須: はい

  * デフォルト値: なし

* **password**

  * 説明: StarRocksデータベースのパスワード

  * 必須: はい

  * デフォルト: なし

* **database**

  * 説明: StarRocksテーブルのデータベース名

  * 必須: はい

  * デフォルト: なし

* **table**

  * 説明: StarRocksテーブルのテーブル名

  * 必須: はい

  * デフォルト: なし

* **loadUrl**

  * 説明: Stream Load用のStarRocks FEのアドレス。複数のFEアドレスを指定することができ、`fe_ip:fe_http_port`の形式で表します。

  * 必須: はい

  * デフォルト値: なし

* **column**

  * 説明: **データに書き込む必要がある**宛先テーブルのフィールド。列はコンマで区切られます。例: "column": ["id", "name", "age"]。
    > **columnの構成項目は指定する必要があり、空白にしてはいけません。**
    > 注意: 変更する際に列の数、タイプなどの変更があるとジョブが正しく実行されないか失敗する可能性があるため、空にしないことを強くお勧めします。構成項目はリーダーのquerySQLまたはcolumnと同じ順序でなければなりません。

  * 必須: はい
  * デフォルト値: なし

* **preSql**

  * 説明: データを宛先テーブルに書き込む前に実行される標準ステートメント

  * 必須: いいえ

  * デフォルト: なし

* **jdbcUrl**

  * 説明: `preSql`および`postSql`を実行するための宛先データベースのJDBC接続情報

  * 必須: いいえ

  * デフォルト: なし

* **loadProps**

  * 説明: StreamLoad用のリクエストパラメータ。詳細についてはStreamLoad紹介ページを参照してください。

  * 必須: いいえ

  * デフォルト値: なし

## 型変換

デフォルトでは、受信データは文字列に変換され、`t`が列区切り記号であり、`n`が行区切り記号で、StreamLoadのインポートのための `csv` ファイルが形成されます。
列区切り記号を変更する場合は、`loadProps`を適切に構成してください。

```json
"loadProps": {
    "column_separator": "\\x01",
    "row_delimiter": "\\x02" 
}
```

インポート形式を `json` に変更する場合は、`loadProps`を適切に構成してください。

```json
"loadProps": {
    "format": "json",
    "strip_outer_array": true
}
```

> `json` フォーマットは、ライターがJSON形式でデータをStarRocksにインポートするためのものです。

## タイムゾーンについて

ソースTPライブラリが別のタイムゾーンにある場合、`datax.py`を実行する際に、コマンドラインの後に以下のパラメータを追加してください。

```json
"-Duser.timezone=xx"
```

例: もしDataXがPostgresデータをインポートし、ソースライブラリがUTC時間である場合は、スタートアップ時にパラメータ`"-Duser.timezone=GMT+0"`を追加してください。