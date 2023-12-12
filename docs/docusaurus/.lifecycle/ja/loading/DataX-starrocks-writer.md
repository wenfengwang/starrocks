---
displayed_sidebar: "Japanese"
---

# DataX ライター

## はじめに

StarRocksWriterプラグインを使用すると、StarRocksのテーブルにデータを書き込むことができます。具体的には、StarRocksWriterは[Stream Load](./StreamLoad.md)を介してCSVまたはJSON形式でデータをStarRocksにインポートし、`reader`で読み込まれたデータを内部的にキャッシュし、より良い書き込みパフォーマンスのためにStarRocksにバルクインポートします。全体のデータフローは `source -> Reader -> DataX channel -> Writer -> StarRocks`です。

[プラグインをダウンロード](https://github.com/StarRocks/DataX/releases)

DataXの完全なパッケージをダウンロードするには、`https://github.com/alibaba/DataX`にアクセスし、starrockswriterプラグインを`datax/plugin/writer/`ディレクトリに配置してください。

次のコマンドを使用してテストしてください:
`python datax.py --jvm="-Xms6G -Xmx6G" --loglevel=debug job.json`

## 関数の説明

### サンプル構成

MySQLからデータを読み込んでStarRocksにロードするための構成ファイルは次のとおりです。

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

  * 説明: StarRocksテーブルのデータベース名。

  * 必須: はい

  * デフォルト: なし

* **table**

  * 説明: StarRocksテーブルのテーブル名。

  * 必須: はい

  * デフォルト: なし

* **loadUrl**

  * 説明: Stream LoadのためのStarRocks FEのアドレスです。複数のFEアドレスを指定でき、`fe_ip:fe_http_port`の形式です。

  * 必須: はい

  * デフォルト値: なし

* **column**

  * 説明: **データに書き込む必要がある**宛先テーブルのフィールド。カンマで区切られた列。例: "column": ["id", "name", "age"]。
    > **columnの設定項目は指定する必要があり、空白にしてはいけません。**
    > 注意: 宛先テーブルのquerySQLやcolumnと同じ順序で設定項目を記述する必要があるため、空白にしないように強く推奨します。列の数やタイプなどを変更した場合、ジョブが誤って実行されるか失敗する場合があります。

  * 必須: はい

  * デフォルト値: なし

* **preSql**

  * 説明: データを宛先テーブルに書き込む前に実行される標準ステートメント。

  * 必須: いいえ

  * デフォルト: いいえ

* **jdbcUrl**

  * 説明: `preSql`および`postSql`を実行するための宛先データベースのJDBC接続情報。
  
  * 必須: いいえ

  * デフォルト: いいえ

* **loadProps**

  * 説明: StreamLoadのリクエストパラメータ。詳細についてはStreamLoadの紹介ページを参照してください。

  * 必須: いいえ

  * デフォルト値: いいえ

## 型変換

デフォルトでは、入力データは文字列に変換され、`t`を列の区切り文字、`n`を行の区切り文字として、`csv`ファイルを形成してStreamLoadにインポートされます。
列の区切り文字を変更するには、`loadProps`を適切に構成してください。

```json
"loadProps": {
    "column_separator": "\\x01",
    "row_delimiter": "\\x02" 
}
```

インポート形式を`json`に変更するには、`loadProps`を適切に構成してください。

```json
"loadProps": {
    "format": "json",
    "strip_outer_array": true
}
```

> `json`形式は、ライターがJSON形式でデータをStarRocksにインポートするためのものです。

## タイムゾーンについて

ソースのtpライブラリが別のタイムゾーンにある場合、datax.pyを実行する際に、次のパラメータをコマンドラインの後に追加してください。

```json
"-Duser.timezone=xx"
```

例: DataXがPostgreSQLデータをインポートし、ソースライブラリがUTC時間にある場合、スタートアップ時にパラメータ"-Duser.timezone=GMT+0"を追加してください。