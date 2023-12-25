---
displayed_sidebar: English
---

# Stream Load

## 1. Stream Loadは、CSV形式のファイルの最初の数行に含まれる列名を識別する機能、またはデータ読み込み時に最初の数行をスキップする機能をサポートしていますか？

Stream Loadは、CSV形式のファイルの最初の数行に含まれる列名を識別する機能をサポートしていません。Stream Loadは最初の数行を他の行と同様に通常のデータとして扱います。

v2.5以前のバージョンでは、Stream Loadはデータ読み込み時にCSVファイルの最初の数行をスキップする機能をサポートしていません。ロードしたいCSVファイルの最初の数行に列名が含まれている場合、以下のいずれかの対応を行ってください：

- データをエクスポートするツールの設定を変更し、最初の数行に列名が含まれないCSVファイルとしてデータを再エクスポートしてください。
- `sed -i '1d' filename` のようなコマンドを使用して、CSVファイルの最初の数行を削除してください。
- ロードコマンドまたはステートメントで `-H "where: <column_name> != '<column_name>'"` を使用し、CSVファイルの最初の数行をフィルタリングして除外してください。ここで `<column_name>` は最初の数行に含まれる任意の列名です。StarRocksはまずデータを変換してからフィルタリングするため、最初の数行の列名が変換先のデータ型に適切に変換されない場合、それらの値は `NULL` として返されます。これは、変換先のStarRocksテーブルに `NOT NULL` と設定された列を含めることができないことを意味します。
- ロードコマンドまたはステートメントで `-H "max_filter_ratio:0.01"` を追加し、最大エラー許容率を1%以下に設定して、数行のエラーを許容するようにしてください。これにより、StarRocksは最初の数行のデータ変換失敗を無視することができます。この場合、`ErrorURL` がエラー行を示すために返されても、Stream Loadジョブは成功する可能性があります。`max_filter_ratio` を大きな値に設定しないでください。`max_filter_ratio` を大きな値に設定すると、重要なデータ品質の問題を見逃す可能性があります。

v3.0以降、Stream Loadは `skip_header` パラメータをサポートしており、これによりCSVファイルの最初の数行をスキップするかどうかを指定できます。詳細は [CSV parameters](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#csv-parameters) を参照してください。

## 2. パーティション列にロードするデータが標準のDATE型やINT型ではない場合、例えば202106.00のような形式のデータの場合、Stream Loadを使用してデータをロードする際にどのようにデータを変換しますか？

StarRocksはロード時のデータ変換をサポートしています。詳細は [Transform data at loading](../../loading/Etl_in_loading.md) を参照してください。

例えば、`TEST` という名前のCSV形式のファイルをロードし、そのファイルが `NO`、`DATE`、`VERSION`、`PRICE` の4つの列から構成されているとします。そのうち `DATE` 列のデータは202106.00のような非標準形式です。StarRocksで `DATE` 列をパーティション列として使用する場合、まず `NO`、`VERSION`、`PRICE`、`DATE` の4つの列から構成されるStarRocksテーブルを作成する必要があります。次に、StarRocksテーブルの `DATE` 列のデータ型をDATE、DATETIME、またはINTとして指定します。最後に、Stream Loadジョブを作成する際に、ロードコマンドまたはステートメントで以下の設定を指定して、ソース `DATE` 列のデータ型から変換先列のデータ型へデータを変換します：

```Plain
-H "columns: NO,DATE_1, VERSION, PRICE, DATE=LEFT(DATE_1,6)"
```

上記の例では、`DATE_1` は変換先 `DATE` 列にマッピングされる一時的な名前付き列と考えることができ、変換先 `DATE` 列にロードされる最終結果は `LEFT()` 関数によって計算されます。ソース列の一時的な名前を最初にリストし、その後で関数を使用してデータを変換する必要があることに注意してください。サポートされている関数には、非集約関数やウィンドウ関数を含むスカラー関数があります。

## 3. Stream Loadジョブで「body exceed max size: 10737418240, limit: 10737418240」というエラーが報告された場合、どうすればよいですか？

ソースデータファイルのサイズがStream Loadでサポートされている最大ファイルサイズである10GBを超えています。以下のいずれかの対応を行ってください：

- `seq -w 0 n` を使用してソースデータファイルをより小さなファイルに分割してください。
- `curl -XPOST http:///be_host:http_port/api/update_config?streaming_load_max_mb=<file_size>` を使用して、[BE configuration item](../../administration/BE_configuration.md#configure-be-dynamic-parameters) `streaming_load_max_mb` の値を調整し、最大ファイルサイズを増やしてください。
