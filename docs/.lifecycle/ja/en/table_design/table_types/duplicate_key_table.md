---
displayed_sidebar: English
---

# Duplicate Key テーブル

Duplicate Key テーブルは、StarRocksのデフォルトモデルです。テーブル作成時にモデルを指定しない場合、デフォルトでDuplicate Key テーブルが作成されます。

Duplicate Key テーブルを作成する際、そのテーブルのソートキーを定義することができます。フィルタ条件にソートキーの列が含まれている場合、StarRocksはテーブルからデータを迅速にフィルタリングし、クエリの速度を向上させることができます。Duplicate Key テーブルでは、テーブルに新しいデータを追加することは可能ですが、既存のデータを変更することはできません。

## シナリオ

Duplicate Key テーブルは、以下のシナリオに適しています：

- 生ログや操作記録などの生データの分析。
- 事前集計方法に制限されず、さまざまな方法でデータをクエリする。
- ログデータや時系列データのロード。新しいデータは追記専用モードで書き込まれ、既存のデータは更新されません。

## テーブルの作成

特定の時間範囲にわたるイベントデータを分析したいとします。この例では、`detail` という名前のテーブルを作成し、`event_time` と `event_type` をソートキーの列として定義します。

テーブル作成のステートメント：

```SQL
CREATE TABLE IF NOT EXISTS detail (
    event_time DATETIME NOT NULL COMMENT "eventのdatetime",
    event_type INT NOT NULL COMMENT "eventのtype",
    user_id INT COMMENT "userのid",
    device_code INT COMMENT "deviceのcode",
    channel INT COMMENT ""
)
DUPLICATE KEY(event_time, event_type)
DISTRIBUTED BY HASH(user_id);
```

> **注意**
>
> - テーブルを作成する際には、`DISTRIBUTED BY HASH` 句を使用してバケット列を指定する必要があります。詳細は [bucketing](../Data_distribution.md#design-partitioning-and-bucketing-rules) を参照してください。
> - v2.5.7 以降、StarRocksはテーブル作成時やパーティション追加時にバケット数（BUCKETS）を自動的に設定することができます。もはやバケット数を手動で設定する必要はありません。詳細は [バケット数の決定](../Data_distribution.md#determine-the-number-of-buckets) を参照してください。

## 使用上の注意点

- テーブルのソートキーに関して、以下の点に注意してください：
  - `DUPLICATE KEY` キーワードを使用して、ソートキーに使用される列を明示的に定義することができます。

    > 注意: デフォルトでは、ソートキーの列を指定しない場合、StarRocksは**最初の3つ**の列をソートキーの列として使用します。

  - Duplicate Key テーブルでは、ソートキーはディメンション列の一部または全部から構成することができます。

- テーブル作成時に、BITMAP インデックスやBloomfilter インデックスなどのインデックスを作成することができます。

- 同一のレコードが2つロードされた場合、Duplicate Key テーブルはそれらを1つのレコードではなく、2つのレコードとして保持します。

## 次に行うこと

テーブルが作成された後、さまざまなデータ取り込み方法を使用してStarRocksにデータをロードすることができます。StarRocksでサポートされているデータ取り込み方法については、[データロードの概要](../../loading/Loading_intro.md)を参照してください。
> 注意: Duplicate Key テーブルを使用するテーブルにデータをロードする場合、テーブルにデータを追加することのみが可能です。テーブル内の既存のデータを変更することはできません。
