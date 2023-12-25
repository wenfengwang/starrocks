---
displayed_sidebar: Chinese
---

# 関数リスト

StarRocks は豊富な関数を提供しており、日常のデータクエリや分析に便利です。一般的な関数のカテゴリに加えて、StarRocks は ARRAY、JSON、MAP、STRUCT などの半構造化関数をサポートし、[Lambda 高階関数](Lambda_expression.md)もサポートしています。これらの関数が要件に合わない場合は、[Java UDF](JAVA_UDF.md) を自作してビジネスニーズに対応することもできます。

目的の関数を以下のカテゴリで探すことができます。

- [関数リスト](#関数リスト)
  - [日付関数](#日付関数)
  - [文字列関数](#文字列関数)
  - [集約関数](#集約関数)
  - [数学関数](#数学関数)
  - [Array 関数](#array-関数)
  - [Bitmap 関数](#bitmap-関数)
  - [JSON 関数](#json-関数)
  - [Map 関数](#map-関数)
  - [Struct 関数](#struct-関数)
  - [テーブル関数](#テーブル関数)
  - [Bit 関数](#bit-関数)
  - [Binary 関数](#binary-関数)
  - [暗号化関数](#暗号化関数)
  - [あいまい/正規表現マッチ関数](#あいまい正規表現マッチ関数)
  - [条件関数](#条件関数)
  - [パーセンタイル関数](#パーセンタイル関数)
  - [スカラー関数](#スカラー関数)
  - [ユーティリティ関数](#ユーティリティ関数)
  - [地理位置情報関数](#地理位置情報関数)
  - [Hash 関数](#hash-関数)

## 日付関数

| 関数                |                 機能      |
|  :-:                |                :-:       |
| [add_months](./date-time-functions/add_months.md)  |   指定された日付（DATE、DATETIME）に整数の月を加えます。     |
| [adddate、days_add](./date-time-functions/adddate.md)          |  日付に指定された時間間隔を加えます。        |
| [convert_tz](./date-time-functions/convert_tz.md)          |   指定された時間を別のタイムゾーンの時間に変換します。  |
| [current_date、curdate](./date-time-functions/curdate.md)          |   現在の日付を取得し、DATE 型で返します。  |
| [current_time、curtime](./date-time-functions/curtime.md)      |  現在の時間を取得し、TIME 型で返します。  |
| [current_timestamp](./date-time-functions/current_timestamp.md)      |  現在のタイムスタンプを取得し、DATETIME 型で返します。   |
| [date](./date-time-functions/date.md)      |  日付または日時表現から日付部分を抽出します。  |
| [date_add](./date-time-functions/date_add.md)      |  日付に指定された時間間隔を加えます。    |
|[date_diff](./date-time-functions/date_diff.md)| 指定された時間単位に基づいて、2つの日付の差を返します。 |
| [date_format](./date-time-functions/date_format.md)      |  format で指定された形式で日付/時間データを表示します。   |
| [date_slice](./date-time-functions/date_slice.md)      |  指定された時間粒度の周期に基づいて、指定された時間をその周期の開始または終了時刻に変換します。  |
| [date_sub、subdate](./date-time-functions/date_sub.md)    |    日付から指定された時間間隔を減らします。   |
| [date_trunc](./date-time-functions/date_trunc.md)     |    指定された精度レベルに基づいて日時を切り捨てます。  |
| [datediff](./date-time-functions/datediff.md)   |  2つの日付の差を計算し、結果は日単位で正確です。        |
| [day](./date-time-functions/day.md) | 指定された日付の日情報を返します。|
| [dayname](./date-time-functions/dayname.md)| 指定された日付に対応する曜日名を返します。|
| [dayofmonth](./date-time-functions/dayofmonth.md)| 日付の日情報を返し、返り値の範囲は 1~31 です。  |
| [dayofweek](./date-time-functions/dayofweek.md)| 指定された日付の曜日インデックス値を返します。  |
| [dayofweek_iso](./date-time-functions/day_of_week_iso.md)| ISO 標準に基づいて、特定の日付が週の何曜日にあたるかを計算します。  |
| [dayofyear](./date-time-functions/dayofyear.md)|  指定された日付がその年の何日目にあたるかを計算します。   |
| [days_add](./date-time-functions/adddate.md)| 日付に指定された時間間隔を加えます。  |
| [days_diff](./date-time-functions/days_diff.md)| 開始時間と終了時間が何日異なるかを計算します。 |
| [days_sub](./date-time-functions/days_sub.md)| 指定された日付または日時から指定された日数を減らし、新しい DATETIME 結果を得ます。  |
| [from_days](./date-time-functions/from_days.md)|  現在の時間が 0000-01-01 から何日経過したかを計算し、現在の日付を計算します。 |
| [from_unixtime](./date-time-functions/from_unixtime.md)|  UNIX タイムスタンプを対応する時間形式に変換します。 |
| [hour](./date-time-functions/hour.md)| 指定された日付の時間情報を取得します。  |
| [hours_add](./date-time-functions/hours_add.md)| 指定された日時に指定された時間数を加えます。  |
| [hours_diff](./date-time-functions/hours_diff.md)| 開始時間と終了時間が何時間異なるかを計算します。 |
| [hours_sub](./date-time-functions/hours_sub.md)| 指定された日時から指定された時間数を減らします。  |
| [jodatime_format](./date-time-functions/jodatime_format.md)| 特定の日付を指定された Joda DateTimeFormat 形式の文字列に変換します。  |
| [last_day](./date-time-functions/last_day.md)| 指定された時間単位に基づいて、入力された日付の最終日を返します。|
| [makedate](./date-time-functions/makedate.md)| 指定された年と日数に基づいて日付を構築します。 |
| [microseconds_add](./date-time-functions/microseconds_add.md)| 日時に指定された時間間隔を加えます。単位はマイクロ秒です。  |
| [microseconds_sub](./date-time-functions/microseconds_sub.md)| 日時から指定された時間間隔を減らします。単位はマイクロ秒です。  |
| [minute](./date-time-functions/minute.md)| 日付の分情報を取得します。返り値の範囲は 0~59 です。  |
| [minutes_add](./date-time-functions/minutes_add.md)| 指定された日時または日付に指定された分数を加えます。|
| [minutes_diff](./date-time-functions/minutes_diff.md)| 開始時間と終了時間が何分異なるかを計算します。  |
| [minutes_sub](./date-time-functions/minutes_sub.md)| 指定された日時または日付から指定された分数を減らします。  |
| [month](./date-time-functions/month.md)|  指定された日付の月を返します。 |
| [monthname](./date-time-functions/monthname.md)|  指定された日付に対応する月名を返します。 |
| [months_add](./date-time-functions/months_add.md)| 日付に指定された月数を加えます。  |
| [months_diff](./date-time-functions/months_diff.md)| 開始時間と終了時間が何ヶ月異なるかを計算します。  |
| [months_sub](./date-time-functions/months_sub.md)|  日付から指定された月数を減らします。 |
|[next_day](./date-time-functions/next_day.md)|入力された日付に基づいて、その後の特定の曜日にあたる日付を返します。 |
| [now](./date-time-functions/now.md)| 現在の時間を取得し、DATETIME 型で返します。  |
| [previous_day](./date-time-functions/previous_day.md)| 入力された日付に基づいて、その前の特定の曜日にあたる日付を返します。 |
| [quarter](./date-time-functions/quarter.md)| 指定された日付が属する四半期を返します。範囲は 1~4 です。  |
| [second](./date-time-functions/second.md)|  日付の秒情報を取得します。返り値の範囲は 0~59 です。 |
| [seconds_add](./date-time-functions/seconds_add.md)| 日時に指定された秒数を加えます。  |
| [seconds_diff](./date-time-functions/seconds_diff.md)| 開始時間と終了時間が何秒異なるかを計算します。  |
| [seconds_sub](./date-time-functions/seconds_sub.md)|  指定された日時または日付から指定された秒数を減らします。 |
| [str2date](./date-time-functions/str2date.md)| format で指定された形式に基づいて str を DATE 型の値に変換します。  |
| [str_to_date](./date-time-functions/str_to_date.md)| format で指定された形式に基づいて str を DATETIME 型の値に変換します。  |
| [str_to_jodatime](./date-time-functions/str_to_jodatime.md)| Joda 形式の文字列を指定された Joda DateTime 形式の DATETIME 値に変換します。  |
| [time_slice](./date-time-functions/time_slice.md)| 指定された時間粒度の周期に基づいて、指定された時間をその周期の開始または終了時刻に変換します。  |
| [time_to_sec](./date-time-functions/time_to_sec.md)| time 時間値を秒数に変換します。  |
| [timediff](./date-time-functions/timediff.md)| 2つの DATETIME 型の値の差を返し、TIME 型で返します。  |
| [timestamp](./date-time-functions/timestamp.md)|  時間表現を DATETIME 値に変換します。 |
| [timestampadd](./date-time-functions/timestampadd.md)| 整数表現の間隔を日付または日時表現に加えます。  |
| [timestampdiff](./date-time-functions/timestampdiff.md)|  2つの日付または日時表現の差を返します。 |
| [to_date](./date-time-functions/to_date.md)| DATETIME 型の値の日付部分を返します。  |
| [to_days](./date-time-functions/to_days.md)| 指定された日付が 0000-01-01 から何日経過したかを返します。  |
| [to_iso8601](./date-time-functions/to_iso8601.md)| 特定の日付を ISO 8601 標準形式の文字列に変換します。  |

| [to_tera_date](./date-time-functions/to_tera_date.md)| VARCHAR 型の値を指定されたフォーマットの日付に変換します。  |
| [to_tera_timestamp](./date-time-functions/to_tera_timestamp.md)| VARCHAR 型の値を指定されたフォーマットで DATETIME 型の値に変換します。  |
| [unix_timestamp](./date-time-functions/unix_timestamp.md)| DATE または DATETIME 型の値を UNIX タイムスタンプに変換します。  |
| [utc_timestamp](./date-time-functions/utc_timestamp.md)| 現在の UTC 日付と時刻を返します。  |
| [week_iso](./date-time-functions/week_iso.md)| ISO 標準に基づき、ある日付がその年の何週目にあたるかを計算します。  |
| [week](./date-time-functions/week.md)| 指定された週の計算ロジックに基づき、特定の日時がその年の何週目にあたるかを計算します。  |
| [weekofyear](./date-time-functions/weekofyear.md)|  特定の日時がその年の何週目にあたるかを計算します。 |
| [weeks_add](./date-time-functions/weeks_add.md)|  元の日時に指定された週数を加算します。 |
| [weeks_diff](./date-time-functions/weeks_diff.md)|  開始時刻と終了時刻の差が何週間かを計算します。 |
| [weeks_sub](./date-time-functions/weeks_sub.md)| 元の日付から指定された週数を減算します。  |
| [year](./date-time-functions/year.md)|  指定された日時の年を返します。 |
| [years_add](./date-time-functions/years_add.md)| 元の日時に指定された年数を加算します。  |
| [years_diff](./date-time-functions/years_diff.md)|  開始時刻と終了時刻の差が何年かを計算します。 |
| [years_sub](./date-time-functions/years_sub.md)  |  指定された日時から指定された年数を減算します。     |

## 文字列関数

| 関数                |                 機能      |
|  :-:                |                :-:       |
| [append_trailing_char_if_absent](./string-functions/append_trailing_char_if_absent.md)   | 文字列が空でなく、末尾に trailing_char 文字が含まれていない場合、trailing_char 文字を末尾に追加します。  |
| [ascii](./string-functions/ascii.md) | 文字列の最初の文字に対応する ASCII コードを返します。  |
| [char](./string-functions/char.md)| 入力された ASCII 値に対応する文字を返します。  |
| [char_length, character_length](./string-functions/char_length.md) | 文字列の長さを返します。  |
| [concat](./string-functions/concat.md)| 複数の文字列を連結します。 |
| [concat_ws](./string-functions/concat_ws.md) | 区切り文字を使用して、2つ以上の文字列を新しい文字列に結合します。  |
| [ends_with](./string-functions/ends_with.md) | 文字列が指定された接尾辞で終わる場合は true を、そうでない場合は false を返します。  |
| [find_in_set](./string-functions/find_in_set.md) | 指定された文字列が文字列リスト内で初めて出現する位置を返します。  |
| [group_concat](./string-functions/group_concat.md) | 結果セット内の複数行の結果を1つの文字列に連結します。  |
| [hex_decode_binary](./string-functions/hex_decode_binary.md) | 16進数でエンコードされた文字列を VARBINARY 型の値にデコードします。 |
| [hex_decode_string](./string-functions/hex_decode_string.md) | 入力文字列内の各16進数のペアを解析して数字にし、その数字を表すバイトに変換し、バイナリ文字列として返します。  |
| [hex](./string-functions/hex.md) | 入力された数字または文字に対して、16進数の文字列表現を返します。  |
| [instr](./string-functions/instr.md) | 指定された文字列内で部分文字列が初めて出現する位置を返します。 |
| [left](./string-functions/left.md) | 文字列の左側から指定された長さの文字を返します。  |
| [length](./string-functions/length.md) | 文字列のバイト長を返します。  |
| [locate](./string-functions/locate.md) | pos インデックスから始まる文字列内で、部分文字列が最初に出現する位置を探します。  |
| [lower](./string-functions/lower.md) | すべての文字列を小文字に変換します。  |
| [lpad](./string-functions/lpad.md) | 指定された長さになるように文字列の前（左側）に文字を追加します。  |
| [ltrim](./string-functions/ltrim.md) | 文字列の左側（開始部分）から連続するスペースまたは指定された文字を削除します。 |
| [money_format](./string-functions/money_format.md) | 数字を通貨形式で出力し、整数部分は3桁ごとにコンマで区切り、小数部分は2桁に保持します。  |
| [null_or_empty](./string-functions/null_or_empty.md) | 文字列が空文字列または NULL の場合は true を、そうでない場合は false を返します。 |
| [parse_url](./string-functions/parse_url.md) | 対象の URL から特定の情報を抽出します。 |
| [repeat](./string-functions/repeat.md) | 文字列を count 回繰り返して出力します。count が 1 未満の場合は空文字列を返します。  |
| [replace](./string-functions/replace.md) | 文字列内の指定されたパターンに一致するすべての文字を他の文字に置き換えます。  |
| [reverse](./string-functions/reverse.md) | 文字列または配列を反転させ、返される文字列または配列の順序が元の文字列または配列と逆になります。  |
| [right](./string-functions/right.md) | 文字列の右側から指定された長さの文字を返します。  |
| [rpad](./string-functions/rpad.md) | 指定された長さになるように文字列の後ろ（右側）に文字を追加します。 |
| [rtrim](./string-functions/rtrim.md) | 文字列の右側（終了部分）から連続するスペースまたは指定された文字を削除します。  |
| [space](./string-functions/space.md) | 指定された数のスペースで構成される文字列を返します。|
| [split](./string-functions/split.md) | 区切り文字に基づいて文字列を分割し、分割後のすべての文字列を ARRAY 形式で返します。  |
| [split_part](./string-functions/split_part.md) | 分割記号に基づいて文字列を分割し、指定された分割部分を返します。  |
| [starts_with](./string-functions/starts_with.md) | 文字列が指定された接頭辞で始まる場合は 1 を、そうでない場合は 0 を返します。  |
| [str_to_map](./string-functions/str_to_map.md) | 与えられた文字列をキーと値のペア (Key-Value pair) に分割し、これらのペアを含む Map を返します。  |
| [strleft](./string-functions/strleft.md) | 文字列の左側から指定された長さの文字を返します。  |
| [strright](./string-functions/strright.md) | 文字列の右側から指定された長さの文字を返します。  |
| [substr, substring](./string-functions/substring.md) | 文字列内の pos 位置から始まる指定された長さの部分文字列を返します。  |
| [substring_index](./string-functions/substring_index.md) | 与えられた文字列から、`count` 番目の区切り文字の前または後の文字列を切り出します。  |
| [translate](./string-functions/translate.md) | 与えられた文字列 `source` 内の `from_string` に出現する文字を `to_string` の対応する位置にある文字に置き換えます。 |
| [trim](./string-functions/trim.md) | 文字列の左側と右側から連続するスペースまたは指定された文字を削除します。  |
| [ucase](./string-functions/ucase.md) | この関数は upper と同じで、文字列を大文字に変換します。  |
| [unhex](./string-functions/unhex.md) | 入力された文字列内の2文字を1組として16進数の文字に変換し、それを連結して文字列として出力します。  |
| [upper](./string-functions/upper.md) | 文字列を大文字に変換します。  |
| [url_decode](./string-functions/url_decode.md) | 文字列を [application/x-www-form-urlencoded](https://www.w3.org/TR/html4/interact/forms.html#h-17.13.4.1) 形式からデコードします。 |
| [url_encode](./string-functions/url_encode.md)  | 文字列を [application/x-www-form-urlencoded](https://www.w3.org/TR/html4/interact/forms.html#h-17.13.4.1) 形式でエンコードします。  |
| [url_extract_parameter](./string-functions/url_extract_parameter.md)   | URL のクエリ部分から、指定されたパラメータ (`name`) の値を取得します。  |

## 集約関数

| 関数                |                 機能      |
|  :-:                |                :-:       |
|  [any_value](./aggregate-functions/any_value.md)| GROUP BY を含む集約クエリで、各集約グループから**ランダムに**1行を選択して返します。 |
|  [approx_count_distinct](./aggregate-functions/approx_count_distinct.md)| COUNT(DISTINCT col) の結果に似た近似値を返します。 |
|  [array_agg](./array-functions/array_agg.md) | 列内の値（NULL 値を含む）を配列に連結します（複数行を1行に）。  |
|  [avg](./aggregate-functions/avg.md)| 選択されたフィールドの平均値を返します。 |
|  [bitmap](./aggregate-functions/bitmap.md)| bitmap 関数を使用して集約を実行します。 |
|  [bitmap_agg](./bitmap-functions/bitmap_agg.md)| 列内の複数行の非 NULL 数値を1行の BITMAP 値にマージします（複数行を1行に）。 |
| [corr](./aggregate-functions/corr.md) | 2つのランダム変数のピアソン相関係数を返します。 |
| [covar_pop](./aggregate-functions/covar_pop.md)| 2つのランダム変数の母集団共分散を返します。 |
| [covar_samp](./aggregate-functions/covar_samp.md)| 2つのランダム変数の標本共分散を返します。 |
|  [count](./aggregate-functions/count.md)| 条件を満たす行数を返します。 |
|  [group_concat](./string-functions/group_concat.md)| 結果セット内の複数行の結果を1つの文字列に連結します。|

|  [grouping](./aggregate-functions/grouping.md)| 列が集約列かどうかを判断し、集約列であれば 0 を、そうでなければ 1 を返します。|
|  [grouping_id](./aggregate-functions/grouping_id.md)| 同じグルーピング基準の集約結果を区別するために使用されます。 |
|  [hll_empty](./aggregate-functions/hll_empty.md)| 空の HLL 列を生成し、INSERT またはデータのインポート時にデフォルト値として使用します。 |
|  [hll_hash](./aggregate-functions/hll_hash.md)| 数値を HLL 型に変換します。通常、データのインポート時に、元のデータの数値を StarRocks テーブルの HLL 列型にマッピングするために使用されます。 |
|  [hll_raw_agg](./aggregate-functions/hll_raw_agg.md)| HLL 型のフィールドを集約するために使用され、HLL 型を返します。 |
|  [hll_union](./aggregate-functions/hll_union.md)| HLL 値のセットの統合を返します。 |
|  [hll_union_agg](./aggregate-functions/hll_union_agg.md)| 複数の HLL 型データを一つの HLL にマージします。 |
|  [max](./aggregate-functions/max.md)| 式の中で最大の値を返します。 |
|  [max_by](./aggregate-functions/max_by.md)| y の最大値に関連付けられた x の値を返します。 |
|  [min](./aggregate-functions/min.md)| 式の中で最小の値を返します。 |
|  [min_by](./aggregate-functions/min_by.md)| y の最小値に関連付けられた x の値を返します。 |
|  [multi_distinct_count](./aggregate-functions/multi_distinct_count.md)| 式から重複を除いた後の行数を返し、COUNT(DISTINCT expr) と同等の機能です。 |
|  [multi_distinct_sum](./aggregate-functions/multi_distinct_sum.md)| 式から重複を除いた後の合計を返し、sum(distinct expr) と同等の機能です。 |
|  [percentile_approx](./aggregate-functions/percentile_approx.md)| p 番目のパーセンタイルの近似値を返します。 |
|  [percentile_cont](./aggregate-functions/percentile_cont.md)| 正確なパーセンタイルを計算します。 |
|  [percentile_disc](./aggregate-functions/percentile_disc.md)| パーセンタイルを計算します。 |
|  [retention](./aggregate-functions/retention.md)| 一定期間のユーザーリテンションを計算するために使用されます。  |
|  [sum](./aggregate-functions/sum.md)| 指定された列の全ての値の合計を返します。 |
|  [std](./aggregate-functions/std.md)| 指定された列の標準偏差を返します。 |
|  [stddev, stddev_pop](./aggregate-functions/stddev.md)| 式の母集団標準偏差を返します。 |
|  [stddev_samp](./aggregate-functions/stddev_samp.md)| 式の標本標準偏差を返します。 |
|  [variance, variance_pop, var_pop](./aggregate-functions/variance.md)| 式の分散を返します。 |
|  [var_samp](./aggregate-functions/var_samp.md)| 式の標本分散を返します。 |
|  [window_funnel](./aggregate-functions/window_funnel.md)| スライディングタイムウィンドウ内のイベントリストを検索し、条件に一致するイベントチェーンの最大連続イベント数を計算します。 |

## 数学関数

| 関数                |                 機能      |
|  :-:                |                :-:       |
|  [abs](./math-functions/abs.md)| 絶対値を計算します。 |
|  [acos](./math-functions/acos.md)| 逆余弦値（ラジアン単位）を計算します。 |
|  [asin](./math-functions/asin.md)| 逆正弦値（ラジアン単位）を計算します。 |
|  [atan](./math-functions/atan.md)| 逆正接値（ラジアン単位）を計算します。 |
|  [atan2](./math-functions/atan2.md)| 二つの引数の符号を使用して象限を決定し、x/y の逆正接の主値を計算し、返り値は [-π, π] の範囲です。 |
|  [bin](./math-functions/bin.md)| 入力された引数を二進数に変換します。 |
|  [ceil, dceil](./math-functions/ceil.md)| x 以上の最小の整数を返します。 |
|  [ceiling](./math-functions/ceiling.md)| x 以上の最小の整数を返します。 |
|  [conv](./math-functions/conv.md)| 入力された引数の基数変換を行います。 |
|  [cos](./math-functions/cos.md)| 余弦値を計算します。 |
|  [cosh](./math-functions/cosh.md)| 入力された数値の双曲余弦値を計算します。 |
|  [cosine_similarity](./math-functions/cos_similarity.md)| 二つのベクトルの余弦類似度を計算し、ベクトル間の類似度を評価します。 |
|  [cosine_similarity_norm](./math-functions/cos_similarity_norm.md)| 二つの正規化されたベクトルの余弦類似度を計算し、ベクトル間の類似度を評価します。|
|  [cot](./math-functions/cot.md)| 余接値（ラジアン単位）を計算します。 |
|  [degrees](./math-functions/degrees.md)| 引数 x を度数に変換します。x はラジアンです。 |
|  [divide](./math-functions/divide.md)| 除算関数で、x を y で割った結果を返します。 |
|  [e](./math-functions/e.md)| 自然対数の底を返します。 |
|  [exp, dexp](./math-functions/exp.md)| e の x 乗を返します。 |
|  [floor, dfloor](./math-functions/floor.md)| x 以下の最大の整数値を返します。 |
|  [fmod](./math-functions/fmod.md)| 剰余関数で、二つの数を割った後の浮動小数点の剰余を返します。 |
|  [greatest](./math-functions/greatest.md)| 複数の入力引数の中で最大の値を返します。 |
|  [least](./math-functions/least.md)| 複数の入力引数の中で最小の値を返します。 |
|  [ln, dlog1, log](./math-functions/ln.md)| 引数 x の自然対数を返します。底は e です。 |
|  [log](./math-functions/log.md)| base を底とする x の対数を返します。base が指定されていない場合、この関数は ln() と同等です。 |
|  [log2](./math-functions/log2.md)| 2 を底とする x の対数を返します。 |
|  [log10, dlog10](./math-functions/log10.md)| 10 を底とする x の対数を返します。 |
|  [mod](./math-functions/mod.md)| 剰余関数で、二つの数を割った後の剰余を返します。 |
|  [multiply](./math-functions/multiply.md)| 二つの引数の積を計算します。 |
|  [negative](./math-functions/negative.md)| 引数の負数を返します。 |
|  [pi](./math-functions/pi.md)| 円周率を返します。 |
|  [pmod](./math-functions/pmod.md)| 剰余関数で、二つの数を割った後の正の剰余を返します。 |
|  [positive](./math-functions/positive.md)| 式の結果を返します。 |
|  [pow, power, dpow, fpow](./math-functions/pow.md)| x の y 乗を返します。 |
|  [radians](./math-functions/radians.md)| 引数 x をラジアンに変換します。x は度数です。 |
|  [rand, random](./math-functions/rand.md)| 0（含む）から 1（含まない）の間のランダムな浮動小数点数を返します。 |
|  [round, dround](./math-functions/round.md)| 指定された小数点以下の桁数で数値を四捨五入します。 |
|  [sign](./math-functions/sign.md)| 引数 x の符号を返します。 |
|  [sin](./math-functions/sin.md)| 引数 x の正弦を計算します。x はラジアンです。 |
|  [sinh](./math-functions/sinh.md)| 入力された数値の双曲正弦値を計算します。 |
|  [sqrt, dsqrt](./math-functions/sqrt.md)| 引数の平方根を計算します。 |
|  [square](./math-functions/square.md)| 引数の平方を計算します。 |
|  [tan](./math-functions/tan.md)| 引数 x の正接を計算します。x はラジアンです。 |
|  [tanh](./math-functions/tanh.md)| 入力された数値の双曲正接値を計算します。 |
|  [truncate](./math-functions/truncate.md)| 数値 x を小数点以下 y 桁で切り捨てた値を返します。 |

## Array 関数

| 関数                |                 機能      |
|  :-:                |                :-:       |
|  [all_match](./array-functions/all_match.md)| 配列内のすべての要素が指定された条件に一致するかどうかを判断します。 |
|  [any_match](./array-functions/any_match.md)| 配列内に指定された条件に一致する要素が存在するかどうかを判断します。 |
|  [array_agg](./array-functions/array_agg.md)| 一列の値（null 値を含む）を配列に連結します（複数行を一行に変換）。 |
|  [array_append](./array-functions/array_append.md)| 配列の末尾に新しい要素を追加します。 |
|  [array_avg](./array-functions/array_avg.md)| ARRAY 内のすべてのデータの平均値を求めます。 |
|  [array_concat](./array-functions/array_concat.md)| 複数の配列を一つの配列に結合します。 |
|  [array_contains](./array-functions/array_contains.md)| 配列に特定の要素が含まれているかどうかをチェックし、含まれていれば 1 を、そうでなければ 0 を返します。 |
|  [array_contains_all](./array-functions/array_contains_all.md)| 配列 arr1 が配列 arr2 のすべての要素を含んでいるかどうかをチェックします。 |
|  [array_cum_sum](./array-functions/array_cum_sum.md)| 配列内の要素に対して累積和を計算します。 |
|  [array_difference](./array-functions/array_difference.md)| 数値配列に対して、隣接する二つの要素の差（後者から前者を引いたもの）を要素とする配列を返します。 |
|  [array_distinct](./array-functions/array_distinct.md)| 配列の要素を重複なしにします。 |
|  [array_filter](./array-functions/array_filter.md)| 設定されたフィルタ条件に一致する配列内の要素を返します。 |
|  [array_generate](./array-functions/array_generate.md)| start と end の間の数値要素を含む配列を生成し、step はステップ幅です。 |

|  [array_intersect](./array-functions/array_intersect.md)| 複数の同じ型の配列に対して、交差する要素を返します。 |
|  [array_join](./array-functions/array_join.md)| 配列内のすべての要素を連結して文字列を生成します。 |
|  [array_length](./array-functions/array_length.md)| 配列内の要素の数を計算します。 |
|  [array_map](./array-functions/array_map.md)| 入力された arr1、arr2 などの配列を lambda_function を使用して変換し、新しい配列を出力します。 |
|  [array_max](./array-functions/array_max.md)| 配列内のすべてのデータの中で最大値を求めます。 |
|  [array_min](./array-functions/array_min.md)| 配列内のすべてのデータの中で最小値を求めます。 |
|  [arrays_overlap](./array-functions/arrays_overlap.md)| 2つの同じ型の配列が同じ要素を含んでいるかどうかを判断します。 |
|  [array_position](./array-functions/array_position.md)| 配列内の特定の要素の位置を取得し、存在する場合はその位置を、存在しない場合は 0 を返します。 |
|  [array_remove](./array-functions/array_remove.md)| 配列から特定の要素を削除します。 |
|  [array_slice](./array-functions/array_slice.md)| 配列のサブセットを返します。 |
|  [array_sort](./array-functions/array_sort.md)| 配列内の要素を昇順に並べ替えます。 |
|  [array_sortby](./array-functions/array_sortby.md)| 配列内の要素を別のキー配列の要素または Lambda 関数によって生成されたキー配列の要素に基づいて昇順に並べ替えます。 |
|  [array_sum](./array-functions/array_sum.md)| 配列内のすべての要素の合計を計算します。 |
|  [array_to_bitmap](./array-functions/array_to_bitmap.md)| 配列型を bitmap 型に変換します。 |
|  [cardinality](./array-functions/cardinality.md)| 配列内の要素の数を計算します。 |
|  [element_at](./array-functions/element_at.md)| 配列内の指定された位置にある要素を取得します。 |
|  [reverse](./string-functions/reverse.md)| 文字列または配列を反転させ、返される文字列または配列の順序を元の文字列または配列の逆順にします。 |
|  [unnest](./array-functions/unnest.md)| テーブル関数で、配列を複数の行に展開します。 |

## Bitmap 関数

| 関数                |                 機能      |
|  :-:                |                :-:       |
|  [bitmap_agg](./bitmap-functions/bitmap_agg.md)| 複数行の非 NULL 値を1行の BITMAP 値に統合します。つまり、複数行を1行に変換します。 |
|  [bitmap_and](./bitmap-functions/bitmap_and.md)| 2つの bitmap の交差を計算し、新しい bitmap を返します。|
|  [bitmap_andnot](./bitmap-functions/bitmap_andnot.md)| 2つの入力された bitmap の差集合を計算します。|
|  [bitmap_contains](./bitmap-functions/bitmap_contains.md)| 入力値が Bitmap 列に含まれているかどうかを計算します。 |
|  [bitmap_count](./bitmap-functions/bitmap_count.md)| bitmap 内の重複しない値の数をカウントします。 |
|  [bitmap_empty](./bitmap-functions/bitmap_empty.md)| 空の bitmap を返し、主に insert や stream load でデフォルト値を埋めるのに使用します。|
|  [bitmap_from_binary](./bitmap-functions/bitmap_from_binary.md)| 特定の形式のバイナリ文字列を bitmap に変換します。|
|  [bitmap_from_string](./bitmap-functions/bitmap_from_string.md)| 文字列を bitmap に変換します。文字列はコンマで区切られた一連の UInt32 数字で構成されます。|
|  [bitmap_hash](./bitmap-functions/bitmap_hash.md)| 任意の型の入力に対して32ビットのハッシュ値を計算し、そのハッシュ値を含む bitmap を返します。|
|  [bitmap_has_any](./bitmap-functions/bitmap_has_any.md)| 2つの Bitmap 列に交差する要素が存在するかどうかを計算します。|
|  [bitmap_intersect](./bitmap-functions/bitmap_intersect.md)| 一連の bitmap 値の交差を求めます。|
|  [bitmap_max](./bitmap-functions/bitmap_max.md)| Bitmap 内の最大値を取得します。|
|  [bitmap_min](./bitmap-functions/bitmap_min.md)| Bitmap 内の最小値を取得します。|
|  [bitmap_or](./bitmap-functions/bitmap_or.md)| 2つの bitmap の結合を計算し、新しい bitmap を返します。|
|  [bitmap_remove](./bitmap-functions/bitmap_remove.md)| Bitmap から特定の数値を削除します。 |
|  [bitmap_subset_in_range](./bitmap-functions/bitmap_subset_in_range.md)| 指定された範囲内の値を持つ Bitmap から要素を返します。|
|  [bitmap_subset_limit](./bitmap-functions/bitmap_subset_limit.md)| 指定された開始値に基づいて、BITMAP から特定の数の要素を取得します。|
|  [bitmap_to_array](./bitmap-functions/bitmap_to_array.md)| BITMAP 内のすべての値を BIGINT 型の配列に組み合わせます。|
|  [bitmap_to_base64](./bitmap-functions/bitmap_to_base64.md)| bitmap を Base64 文字列に変換します。|
|  [bitmap_to_binary](./bitmap-functions/bitmap_to_binary.md)| bitmap を特定の形式のバイナリ文字列に変換します。|
|  [base64_to_bitmap](./bitmap-functions/base64_to_bitmap.md)| Base64 エンコードされた文字列を Bitmap に変換します。 |
|  [bitmap_to_string](./bitmap-functions/bitmap_to_string.md)| bitmap をコンマで区切られた文字列に変換します。|
|  [bitmap_union](./bitmap-functions/bitmap_union.md)| 一連の bitmap 値の結合を求めます。 |
|  [bitmap_union_count](./bitmap-functions/bitmap_union_count.md)| 一連の Bitmap 値の結合を計算し、結合の基数を返します。|
|  [bitmap_union_int](./bitmap-functions/bitmap_union_int.md)| TINYINT、SMALLINT、INT 型の列内の重複しない値の数を計算します。|
|  [bitmap_xor](./bitmap-functions/bitmap_xor.md)| 2つの Bitmap 内の重複しない要素で構成される集合を計算します。|
|  [intersect_count](./bitmap-functions/intersect_count.md)| bitmap の交差の大きさを求めます。|
|  [subdivide_bitmap](./bitmap-functions/subdivide_bitmap.md)| 大きな bitmap を複数の小さな bitmap に分割します。|
|  [sub_bitmap](./bitmap-functions/sub_bitmap.md)| 2つの bitmap 間の共通要素の数を計算します。|
|  [to_bitmap](./bitmap-functions/to_bitmap.md)| 入力値を bitmap に変換します。 |

## JSON 関数

| 関数                |                 機能      |
|  :-:                |                :-:       |
|  [json_array](./json-functions/json-constructor-functions/json_array.md)| SQL 配列を受け取り、JSON 型の配列を返します。|
|  [json_object](./json-functions/json-constructor-functions/json_object.md)| キーと値のコレクションを受け取り、それらのキーバリューペアを含む JSON 型のオブジェクトを返します。|
|  [parse_json](./json-functions/json-constructor-functions/parse_json.md)| 文字列型のデータを JSON 型のデータに構築します。|
|  [arrow_function](./json-functions/json-query-and-processing-functions/arrow-function.md)| arrow_function は、JSON オブジェクト内の指定されたパスの値をクエリします。|
|  [cast](./json-functions/json-query-and-processing-functions/cast.md)| JSON 型のデータと SQL 型のデータ間の相互変換を実現します。|
|  [get_json_double](./json-functions/json-query-and-processing-functions/get_json_double.md)| JSON 文字列内の指定されたパスの浮動小数点数の内容を解析して取得します。|
|  [get_json_int](./json-functions/json-query-and-processing-functions/get_json_int.md)| JSON 文字列内の指定されたパスの整数の内容を解析して取得します。|
|  [get_json_string](./json-functions/json-query-and-processing-functions/get_json_string.md)| JSON 文字列内の指定されたパスの文字列を解析して取得します。|
|  [json_each](./json-functions/json-query-and-processing-functions/json_each.md)| JSON オブジェクトの最外層をキーと値に展開し、1行または複数行のデータセットを返します。|
|  [json_exists](./json-functions/json-query-and-processing-functions/json_exists.md)| JSON オブジェクト内の指定されたパスに特定の条件を満たす値が存在するかどうかをクエリします。|
|  [json_keys](./json-functions/json-query-and-processing-functions/json_keys.md)| JSON オブジェクト内のすべての最上層のキーを含む配列を返します。|
|  [json_length](./json-functions/json-query-and-processing-functions/json_length.md)| JSON 文字列の長さを計算します。|
|  [json_query](./json-functions/json-query-and-processing-functions/json_query.md)| JSON オブジェクト内の指定されたパス（json_path）の値をクエリし、JSON 型の結果を出力します。 |
|  [json_string](./json-functions/json-query-and-processing-functions/json_string.md)| JSON 型を JSON 文字列に変換します。|
|  [to_json](./json-functions/json-query-and-processing-functions/to_json.md)| Map または Struct 型のデータを JSON データに変換します。 |

## Map 関数

| 関数                |                 機能      |
|  :-:                |                :-:       |
|  [cardinality](./map-functions/cardinality.md)| Map 内の要素の数を計算します。 |
|  [distinct_map_keys](./map-functions/distinct_map_keys.md)| Map 内の重複するキーを削除します。 |
|  [element_at](./map-functions/element_at.md)| Map 内の指定されたキーに対応する値を取得します。 |
|  [map_apply](./map-functions/map_apply.md)| Map 内のすべてのキーまたは値に対して Lambda 関数を適用した後の Map を返します。 |
|  [map_concat](./map-functions/map_concat.md)| 複数の Map を1つの Map に結合します。 |
|  [map_filter](./map-functions/map_filter.md)| 設定されたフィルター関数に基づいて Map 内の一致するキーバリューペアを返します。 |
|  [map_from_arrays](./map-functions/map_from_arrays.md)| 2つの配列をキーと値として組み合わせて1つの Map オブジェクトを作成します。 |
|  [map_keys](./map-functions/map_keys.md)| Map 内のすべてのキーを含む配列を返します。 |
|  [map_size](./map-functions/map_size.md)| Map 内の要素の数を計算します。 |
|  [map_values](./map-functions/map_values.md)| Map 内のすべての値を含む配列を返します。 |
|  [transform_keys](./map-functions/transform_keys.md)| Map 内のキーに対して Lambda 変換を適用します。 |
|  [transform_values](./map-functions/transform_values.md)| Map 内の値に対して Lambda 変換を適用します。 |

## Struct 関数

| 関数                |                 機能      |
|  :-:                |                :-:       |
|  [named_struct](./struct-functions/named_struct.md)| 与えられたフィールド名とフィールド値に基づいて STRUCT を構築します。 |
|  [row](./struct-functions/row.md)| 与えられた1つまたは複数の値に基づいて STRUCT を構築します。 |

## テーブル関数

| 関数                |                 機能      |
|  :-:                |                :-:       |
| [files](./table-functions/files.md) | クラウドストレージまたはHDFSからデータファイルを読み取ります。|
| [generate_series](./table-functions/generate_series.md) | startからendまでの数値のシリーズを生成し、ステップはstepです。 |
| [json_each](./json-functions/json-query-and-processing-functions/json_each.md) | JSONオブジェクトの最上層をキーと値に展開し、1行または複数行のデータの集合を返します。 |
| [subdivide_bitmap](./bitmap-functions/subdivide_bitmap.md)| 大きなbitmapを複数のサブbitmapに分割します。|
| [unnest](./array-functions/unnest.md) | 配列を複数の行に展開するために使用されます。|

## Bit関数

| 関数                |                 機能      |
|  :-:                |                :-:       |
|  [bitand](./bit-functions/bitand.md)| 2つの数値に対してビットAND演算を行った結果を返します。 |
|  [bitnot](./bit-functions/bitnot.md)| 引数`x`に対するビットNOT演算の結果を返します。 |
|  [bitor](./bit-functions/bitor.md)| 2つの数値に対してビットOR演算を行った結果を返します。  |
|  [bitxor](./bit-functions/bitxor.md)| 2つの数値に対してビットXOR演算を行った結果を返します。 |
|  [bit_shift_left](./bit-functions/bit_shift_left.md)| 数値または数値式の2進表現を指定されたビット数だけ左にシフトします。算術左シフトを実行します。 |
|  [bit_shift_right](./bit-functions/bit_shift_right.md)| 数値または数値式の2進表現を指定されたビット数だけ右にシフトします。算術右シフトを実行します。 |
|  [bit_shift_right_logical](./bit-functions/bit_shift_right_logical.md)| 数値または数値式の2進表現を指定されたビット数だけ右にシフトします。論理右シフトを実行します。 |

## Binary関数

| 関数                |                 機能      |
|  :-:                |                :-:       |
|  [from_binary](./binary-functions/from_binary.md)| 指定されたフォーマットに基づいてバイナリデータをVARCHAR型の文字列に変換します。 |
|  [to_binary](./binary-functions/to_binary.md)| 指定されたバイナリフォーマット(binary_type)に基づいてVARCHAR文字列をバイナリ型に変換します。|

## 暗号化関数

| 関数                |                 機能      |
|  :-:                |                :-:       |
|  [aes_decrypt](./cryptographic-functions/aes_decrypt.md)| AES_128_ECBアルゴリズムを使用して文字列を復号化し、バイナリ文字列を返します。 |
|  [aes_encrypt](./cryptographic-functions/aes_encrypt.md)| AES_128_ECBアルゴリズムを使用して文字列を暗号化し、バイナリ文字列を返します。 |
|  [base64_decode_binary](./cryptographic-functions/base64_decode_binary.md)| Base64エンコードされた文字列をデコードし、VARBINARY型の値を返します。 |
|  [base64_decode_string](./cryptographic-functions/base64_decode_string.md)| Base64エンコードされた文字列をデコードするために使用され、to_base64()関数の逆関数です。 |
|  [from_base64](./cryptographic-functions/from_base64.md)| Base64エンコードされた文字列strをデコードします。逆関数はto_base64です。 |
|  [md5](./cryptographic-functions/md5.md)| MD5暗号化アルゴリズムを使用して与えられた文字列を暗号化し、128ビットのチェックサム(checksum)を32文字の16進数文字列で出力します。 |
|  [md5sum](./cryptographic-functions/md5sum.md)| 複数の入力パラメータのMD5 128ビットチェックサム(checksum)を計算し、32文字の16進数文字列で表します。 |
|  [sha2](./cryptographic-functions/sha2.md)| SHA-2シリーズのハッシュ関数(SHA-224/SHA-256/SHA-384/SHA-512)を計算します。 |
|  [sm3](./cryptographic-functions/sm3.md)| SM3ダイジェストアルゴリズムを使用して文字列を256ビットの16進数文字列に暗号化し、32ビットごとにスペースで区切ります。 |
|  [to_base64](./cryptographic-functions/to_base64.md)| 文字列strをBase64でエンコードします。逆関数はfrom_base64です。 |

## 曖昧/正規表現マッチング関数

| 関数                |                 機能      |
|  :-:                |                :-:       |
|  [like](./like-predicate-functions/like.md) | 文字列が指定されたパターン`pattern`に**曖昧にマッチ**するかどうかを判断します。|
|  [regexp](./like-predicate-functions/regexp.md) | 文字列が指定された正規表現`pattern`にマッチするかどうかを判断します。 |
|  [regexp_extract](./like-predicate-functions/regexp_extract.md) | 文字列に対して正規表現マッチングを行い、patternに完全にマッチするstrの部分からpos番目のマッチ部分を抽出します。マッチしない場合は空文字列を返します。 |
|  [regexp_replace](./like-predicate-functions/regexp_replace.md) | 文字列に対して正規表現マッチングを行い、patternにヒットした部分をreplで置換します。 |

## 条件関数

| 関数                |                 機能      |
|  :-:                |                :-:       |
|  [case](./condition-functions/case_when.md)| 式を値と比較します。マッチするものが見つかれば、THENの結果を返します。見つからなければ、ELSEの結果を返します。 |
|  [coalesce](./condition-functions/coalesce.md)| 左から右へと最初の非NULLの式を返します。 |
|  [if](./condition-functions/if.md)| 式expr1が真であれば、結果expr2を返し、そうでなければ結果expr3を返します。 |
|  [ifnull](./condition-functions/ifnull.md)| expr1がNULLでなければexpr1を返し、NULLであればexpr2を返します。 |
|  [nullif](./condition-functions/nullif.md)| 式expr1とexpr2が等しい場合はNULLを返し、そうでなければexpr1の値を返します。 |

## パーセンタイル関数

| 関数                |                 機能      |
|  :-:                |                :-:       |
|  [percentile_approx_raw](./percentile-functions/percentile_approx_raw.md)| 与えられたパラメータxのパーセンタイルを計算します。 |
|  [percentile_empty](./percentile-functions/percentile_empty.md)| percentile型の値を構築し、主にINSERTやStream Loadのインポート時にデフォルト値を埋めるために使用されます。 |
|  [percentile_hash](./percentile-functions/percentile_hash.md)| double型の数値をpercentile型の値に構築します。 |
|  [percentile_union](./percentile-functions/percentile_union.md)| グループ化された結果を集約するために使用されます。 |

## スカラー関数

| 関数                |                 機能      |
|  :-:                |                :-:       |
| [hll_cardinality](./scalar-functions/hll_cardinality.md) |  HLL型の値の基数を計算するために使用されます。  |

## ユーティリティ関数

| 関数                |                 機能      |
|  :-:                |                :-:       |
| [catalog](./utility-functions/catalog.md)| 現在のセッションが属するカタログを照会します。 |
|  [current_role](./utility-functions/current_role.md)| 現在のユーザーがアクティブにしているロールを取得します。  |
|  [current_version](./utility-functions/current_version.md)| 現在のStarRocksのバージョンを取得します。 |
| [database](./utility-functions/database.md)| 現在のセッションが属するデータベースを照会します。 |
| [get_query_profile](./utility-functions/get_query_profile.md)| 指定されたクエリのプロファイルを取得します。|
|  [host_name](./utility-functions/host_name.md)| 計算が行われるノードのホスト名を取得します。|
|  [isnull](./utility-functions/isnull.md)| 入力値がNULLかどうかを判断します。|
| [is_role_in_session](./utility-functions/is_role_in_session.md) | 指定されたロール（ネストされたロールを含む）が現在のセッションでアクティブになっているかどうかをチェックします。 |
|  [last_query_id](./utility-functions/last_query_id.md)| 最近実行されたクエリのIDを返します。|
|  [sleep](./utility-functions/sleep.md)| 現在実行中のスレッドをx秒間スリープさせます。|
|  [uuid](./utility-functions/uuid.md)| VARCHAR形式でランダムなUUID値を返します。長さは36文字で、32の16進数文字が4つのハイフンで接続されており、形式は8-4-4-4-12です。|
|  [uuid_numeric](./utility-functions/uuid_numeric.md)|数値型のランダムなUUID値を返します。 |
|  [version](./utility-functions/version.md)|現在のMySQLデータベースのバージョンを返します。  |

## 地理位置関数

| 関数                |                 機能      |
|  :-:                |                :-:       |
|  [ST_AsText、ST_AsWKT](./spatial-functions/st_astext.md)| 幾何図形をWKT（Well Known Text）形式に変換します。 |
|  [st_circle](./spatial-functions/st_circle.md)| WKT（Well Known Text）を地球の球面上の円に変換します。 |
|  [st_contains](./spatial-functions/st_contains.md)| 幾何図形shape1が幾何図形shape2を完全に含むかどうかを判断します。 |
|  [st_distance_sphere](./spatial-functions/st_distance_sphere.md)| 地球上の2点間の球面距離を計算し、単位はメートルです。 |
|  [st_geometryfromtext](./spatial-functions/st_geometryfromtext.md)| WKT（Well Known Text）を対応するメモリ内の幾何形状に変換します。 |
|  [st_linefromtext、ST_LineStringFromText](./spatial-functions/st_linefromtext.md)| WKT（Well Known Text）をメモリ内のLine形状に変換します。 |
|  [st_point](./spatial-functions/st_point.md)| 与えられたX座標値、Y座標値から対応するPointを返します。 |
|  [st_polygon](./spatial-functions/st_polygon.md)| WKT（Well Known Text）を対応する多角形のメモリ形状に変換します。 |
|  [st_x](./spatial-functions/st_x.md)| pointが有効なPOINT型である場合、対応するX座標値を返します。 |
|  [st_y](./spatial-functions/st_y.md)| pointが有効なPOINT型である場合、対応するY座標値を返します。 |

## Hash関数

| 関数                |                 機能      |
|  :-:                |                :-:       |
| [murmur_hash3_32](./hash-functions/murmur_hash3_32.md) | 入力文字列の 32 ビット murmur3 hash 値を返します。 |
| [xx_hash3_64](./hash-functions/xx_hash3_64.md) | 入力文字列の 64 ビット xxhash3 値を返します。 |
