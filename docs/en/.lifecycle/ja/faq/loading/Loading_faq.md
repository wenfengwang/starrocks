---
displayed_sidebar: "Japanese"
---

# データの読み込み

## 1. 「close index channel failed」または「too many tablet versions」エラーが発生した場合の対処方法は何ですか？

データの読み込みを頻繁に実行し、データが適切なタイミングで圧縮されなかったため、読み込み中に生成されるデータバージョンの数が許容される最大数（デフォルトで1000）を超えています。次のいずれかの方法を使用して、この問題を解決してください：

- 各個別のジョブで読み込むデータ量を増やし、読み込み頻度を減らすことで解決します。

- 各BEのBE設定ファイル**be.conf**の構成項目を以下のように変更し、データの圧縮を高速化します：

    ```Plain
    cumulative_compaction_num_threads_per_disk = 4
    base_compaction_num_threads_per_disk = 2
    cumulative_compaction_check_interval_seconds = 2
    ```

  上記の構成項目の設定を変更した後は、メモリとI/Oが正常であることを確認する必要があります。

## 2. 「Label Already Exists」エラーが発生した場合の対処方法は何ですか？

このエラーは、同じStarRocksデータベース内で、他の読み込みジョブと同じラベルを持つ読み込みジョブが既に正常に実行されているか、実行中であるために発生します。

Stream LoadジョブはHTTPによって送信されます。一般的に、プログラム言語のHTTPクライアントにはリクエストの再試行ロジックが組み込まれています。StarRocksクラスタは、HTTPクライアントからの読み込みジョブリクエストを受け取ると、すぐにリクエストの処理を開始しますが、タイムリーにジョブの結果をHTTPクライアントに返しません。そのため、HTTPクライアントは同じ読み込みジョブリクエストを再度送信します。しかし、StarRocksクラスタは既に最初のリクエストを処理しているため、2番目のリクエストに対して「Label Already Exists」エラーを返します。

異なる読み込み方法で送信された読み込みジョブが同じラベルを持たず、繰り返し送信されていないことを確認するために、次の手順を実行してください：

- FEログを表示し、失敗した読み込みジョブのラベルが2回記録されているかどうかを確認します。ラベルが2回記録されている場合、クライアントは読み込みジョブリクエストを2回送信しています。

  > **注意**
  >
  > StarRocksクラスタは、読み込み方法に基づいて読み込みジョブのラベルを区別しません。そのため、異なる読み込み方法で送信された読み込みジョブは同じラベルを持つ場合があります。

- `SHOW LOAD WHERE LABEL = "xxx"`を実行し、同じラベルを持ち、**FINISHED**状態にある読み込みジョブを確認します。

  > **注意**
  >
  > `xxx`は確認したいラベルです。

読み込みジョブを送信する前に、データの読み込みに必要な時間のおおよその量を計算し、クライアント側のリクエストタイムアウト期間を調整することをおすすめします。これにより、クライアントが複数回読み込みジョブリクエストを送信することを防ぐことができます。

## 3. 「ETL_QUALITY_UNSATISFIED; msg:quality not good enough to cancel」エラーが発生した場合の対処方法は何ですか？

[SHOW LOAD](../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md)を実行し、返された実行結果のエラーURLを使用してエラーの詳細を表示します。

一般的なデータ品質のエラーは次のとおりです：

- 「convert csv string to INT failed.」

  ソースの列から数値型のデータ型に変換できなかった文字列があります。たとえば、`abc`を数値に変換できませんでした。

- 「the length of input is too long than schema.」

  ソースの列の値が、テーブル作成時に指定された最大長を超える長さであるため、対応する宛先の列でサポートされていません。たとえば、CHARデータ型のソース列の値がテーブルの最大長を超えている場合、またはINTデータ型のソース列の値が4バイトを超えている場合です。

- 「actual column number is less than schema column number.」

  指定された列区切り文字に基づいてソースの行が解析された後、取得される列の数が宛先テーブルの列数よりも少なくなります。読み込みコマンドまたはステートメントで指定された列区切り文字が、実際にその行で使用されている列区切り文字と異なる場合があります。

- 「actual column number is more than schema column number.」

  指定された列区切り文字に基づいてソースの行が解析された後、取得される列の数が宛先テーブルの列数よりも多くなります。読み込みコマンドまたはステートメントで指定された列区切り文字が、実際にその行で使用されている列区切り文字と異なる場合があります。

- 「the frac part length longer than schema scale.」

  DECIMAL型のソース列の小数部分の長さが指定された長さを超えています。

- 「the int part length longer than schema precision.」

  DECIMAL型のソース列の整数部分の長さが指定された長さを超えています。

- 「there is no corresponding partition for this key.」

  ソースの行のパーティション列の値がパーティション範囲内にありません。

## 4. RPCがタイムアウトした場合の対処方法は何ですか？

各BEのBE設定ファイル**be.conf**の`write_buffer_size`構成項目の設定を確認してください。この構成項目は、BE上のメモリブロックごとの最大サイズを制御するために使用されます。デフォルトの最大サイズは100 MBです。最大サイズが非常に大きい場合、リモートプロシージャコール（RPC）がタイムアウトする可能性があります。この問題を解決するには、BE設定ファイルの`write_buffer_size`および`tablet_writer_rpc_timeout_sec`構成項目の設定を調整します。詳細については、[BE configurations](../../loading/Loading_intro.md#be-configurations)を参照してください。

## 5. 「Value count does not match column count」エラーが発生した場合の対処方法は何ですか？

読み込みジョブが失敗した後、ジョブ結果で返されたエラーURLを使用してエラーの詳細を取得しました。その結果、「Value count does not match column count」というエラーが表示されました。これは、ソースデータファイルの列数と宛先のStarRocksテーブルの列数が一致しないことを示しています：

```Java
Error: Value count does not match column count. Expect 3, but got 1. Row: 2023-01-01T18:29:00Z,cpu0,80.99
Error: Value count does not match column count. Expect 3, but got 1. Row: 2023-01-01T18:29:10Z,cpu1,75.23
Error: Value count does not match column count. Expect 3, but got 1. Row: 2023-01-01T18:29:20Z,cpu2,59.44
```

この問題の原因は次のとおりです：

読み込みコマンドまたはステートメントで指定された列区切り文字が、ソースデータファイルで実際に使用されている列区切り文字と異なるためです。前述の例では、CSV形式のデータファイルは3つの列から構成されており、カンマ（`,`）で区切られています。しかし、読み込みコマンドまたはステートメントでは`\t`が列区切り文字として指定されています。その結果、ソースデータファイルの3つの列が誤って1つの列に解析されます。

読み込みコマンドまたはステートメントでカンマ（`,`）を列区切り文字として指定してください。その後、読み込みジョブを再度送信してください。
