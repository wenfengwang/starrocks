---
displayed_sidebar: "Japanese"
---

# DataX writer（データXライター）

## イントロダクション

StarRocksWriterプラグインは、データをStarRocksの宛先テーブルに書き込むことができます。具体的には、StarRocksWriterは[Stream Load](./StreamLoad.md)を介してCSVまたはJSON形式でデータをStarRocksにインポートし、`reader`が読み取ったデータを内部的にキャッシュし、バルクインポートして書き込みパフォーマンスを向上させます。全体的なデータフローは、`ソース -> リーダー -> DataXチャネル -> ライター -> StarRocks`です。

[プラグインをダウンロードする](https://github.com/StarRocks/DataX/releases)

DataXの完全なパッケージをダウンロードするには、`https://github.com/alibaba/DataX`にアクセスし、starrockswriterプラグインを`datax/plugin/writer/`ディレクトリに配置してください。

次のコマンドを使用してテストしてください：
`python datax.py --jvm="-Xms6G -Xmx6G" --loglevel=debug job.json`

## 機能の説明

### サンプルの設定

以下は、MySQLからデータを読み取り、StarRocksにロードするための設定ファイルです。

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

* **username（ユーザー名）**

  * 説明：StarRocksデータベースのユーザー名

  * 必須：はい

  * デフォルト値：なし

* **password（パスワード）**

  * 説明：StarRocksデータベースのパスワード

  * 必須：はい

  * デフォルト値：なし

* **database（データベース）**

  * 説明：StarRocksテーブルのデータベース名

  * 必須：はい

  * デフォルト値：なし

* **table（テーブル）**

  * 説明：StarRocksテーブルのテーブル名

  * 必須：はい

  * デフォルト値：なし

* **loadUrl（ロードURL）**

  * 説明：Stream Load用のStarRocks FEのアドレスです。複数のFEアドレスを指定できます。形式は`fe_ip:fe_http_port`です。

  * 必須：はい

  * デフォルト値：なし

* **column（カラム）**

  * 説明：データを書き込む必要がある宛先テーブルのフィールドです。カラムはカンマで区切られます。例： "column": ["id", "name", "age"]。
    >**columnの設定項目は指定する必要があり、空白にはできません。**
    >注意：空にすることは強くお勧めしません。宛先テーブルの列数や型などを変更すると、ジョブが正しく実行されないか失敗する可能性があります。設定項目は、querySQLまたはリーダーの列と同じ順序である必要があります。

* 必須：はい

* デフォルト値：なし

* **preSql（事前SQL）**

* 説明：データを宛先テーブルに書き込む前に実行される標準のステートメントです。

* 必須：いいえ

* デフォルト値：いいえ

* **jdbcUrl（JDBC URL）**

  * 説明：`preSql`および`postSql`を実行するための宛先データベースのJDBC接続情報です。
  
  * 必須：いいえ

* デフォルト値：いいえ

* **loadProps（ロードプロパティ）**

  * 説明：StreamLoadのリクエストパラメータです。詳細については、StreamLoadの紹介ページを参照してください。

  * 必須：いいえ

  * デフォルト値：いいえ

## 型変換

デフォルトでは、受信データは文字列に変換され、`t`が列の区切り記号、`n`が行の区切り記号となり、`csv`ファイルがStreamLoadのインポートに使用されます。
列の区切り記号を変更するには、`loadProps`を適切に設定してください。

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

> `json`形式は、ライターがデータをJSON形式でStarRocksにインポートするためのものです。

## タイムゾーンについて

ソースライブラリが別のタイムゾーンにある場合、datax.pyを実行する際に、コマンドラインの後に次のパラメータを追加してください。

```json
"-Duser.timezone=xx"
```

例えば、DataXがPostgreSQLデータをインポートし、ソースライブラリがUTC時間にある場合、パラメータ"-Duser.timezone=GMT+0"を起動時に追加してください。
