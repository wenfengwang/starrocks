---
displayed_sidebar: English
---

# DataX Writer

## はじめに

StarRocksWriterプラグインは、StarRocksの宛先テーブルへのデータ書き込みを可能にします。具体的には、StarRocksWriterは[Stream Load](./StreamLoad.md)を介してCSVまたはJSON形式でデータをStarRocksにインポートし、`reader`によって読み込まれたデータを内部キャッシュし、一括インポートすることで書き込みパフォーマンスを向上させます。全体的なデータフローは`source -> Reader -> DataX channel -> Writer -> StarRocks`です。

[プラグインをダウンロード](https://github.com/StarRocks/DataX/releases)

`https://github.com/alibaba/DataX`からDataXのフルパッケージをダウンロードし、starrockswriterプラグインを`datax/plugin/writer/`ディレクトリに配置してください。

以下のコマンドでテストを行います:
`python datax.py --jvm="-Xms6G -Xmx6G" --loglevel=debug job.json`

## 機能説明

### サンプル設定

MySQLからデータを読み込み、StarRocksにロードする設定ファイルの例です。

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

## Starrockswriterパラメータ説明

* **username**

  * 説明: StarRocksデータベースのユーザー名

  * 必須: はい

  * デフォルト値: なし

* **password**

  * 説明: StarRocksデータベースのパスワード

  * 必須: はい

  * デフォルト値: なし

* **database**

  * 説明: StarRocksテーブルのデータベース名

  * 必須: はい

  * デフォルト値: なし

* **table**

  * 説明: StarRocksテーブルのテーブル名

  * 必須: はい

  * デフォルト値: なし

* **loadUrl**

  * 説明: ストリームロード用のStarRocks FEのアドレス。複数のFEアドレスを`fe_ip:fe_http_port`の形式で指定できます。

  * 必須: はい

  * デフォルト値: なし

* **column**

  * 説明: データを書き込む必要がある宛先テーブルのフィールド。列はコンマで区切ります。例: "column": ["id", "name", "age"]。
    >**column設定項目は指定が必須で、空白にすることはできません。**
    >注意: 宛先テーブルの列数、型などが変更された場合、ジョブが正しく実行されないか、失敗する可能性があるため、空白にすることはお勧めしません。設定項目はreaderのquerySQLまたはcolumnと同じ順序でなければなりません。

  * 必須: はい

  * デフォルト値: なし

* **preSql**

  * 説明: 宛先テーブルにデータを書き込む前に実行されるSQLステートメント。

  * 必須: いいえ

  * デフォルト値: なし

* **jdbcUrl**

  * 説明: `preSql`と`postSql`を実行する宛先データベースのJDBC接続情報。

  * 必須: いいえ

  * デフォルト値: なし

* **loadProps**

  * 説明: StreamLoadのリクエストパラメータ。詳細はStreamLoadの紹介ページを参照してください。

  * 必須: いいえ

  * デフォルト値: なし

## 型変換

デフォルトでは、受信データは文字列に変換され、`t`を列区切り文字、`n`を行区切り文字として使用し、StreamLoadインポート用のcsvファイルを形成します。
列区切り文字を変更するには、`loadProps`を適切に設定してください。

```json
"loadProps": {
    "column_separator": "\\x01",
    "row_delimiter": "\\x02" 
}
```

インポート形式を`json`に変更するには、`loadProps`を適切に設定してください。

```json
"loadProps": {
    "format": "json",
    "strip_outer_array": true
}
```

> `json`形式は、WriterがJSON形式でStarRocksにデータをインポートするためのものです。

## タイムゾーンについて

ソースのtpライブラリが別のタイムゾーンにある場合、datax.pyを実行する際にコマンドラインの後に以下のパラメータを追加します。

```json
"-Duser.timezone=xx"
```

例えば、DataXがPostgresデータをインポートし、ソースライブラリがUTC時間である場合、起動時にパラメータ`"-Duser.timezone=GMT+0"`を追加します。
