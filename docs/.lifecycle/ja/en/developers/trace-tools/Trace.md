---
displayed_sidebar: English
---

### バックグラウンド

&emsp;分散トレース、一般にトレースとして知られているものは、マイクロサービスやサーバーレスアプリケーションなどのマルチサービスアーキテクチャを通じて伝播するリクエスト（アプリケーションやエンドユーザーによる）がたどったパスを記録します。トレースがなければ、分散システムにおけるパフォーマンス問題の原因を特定することは困難です。トレースはアプリケーションやシステムの健全性を可視化し、ローカルで再現が難しい挙動のデバッグを可能にします。非決定論的な問題を抱えることが多い分散システムや、複雑すぎてローカルでの再現が困難なシステムにとって、トレースは不可欠です。
&emsp;トレースは、分散システムを通じてリクエストが流れる際に、リクエスト内で何が起こるかを分解することで、分散システムのデバッグと理解を容易にします。トレースは1つ以上のSpanで構成され、最初のSpanがRoot Spanを表します。Root Spanはリクエストの開始から終了までを表し、その下のSpanはリクエスト中に起こること（またはリクエストを構成するステップ）のより詳細なコンテキストを提供します。多くのオブザーバビリティバックエンドは、トレースをこの図のようなウォーターフォール図で視覚化します。

![trace_pic1](../../assets/trace_pic1.png)

&emsp;ウォーターフォール図は、Root Spanとその子Spanの親子関係を示します。Spanが別のSpanをカプセル化する場合、これはネストされた関係を表します。
&emsp;最近、SRはトレーシングフレームワークを導入しました。これはOpenTelemetryとJaegerを活用して、システム内の分散イベントをトレースするものです。

*   OpenTelemetryはインストルメンテーション/トレーシングSDKです。開発者はこれを使用してコードに計測を施し、オブザーバビリティバックエンドにトレーシングデータを送信できます。多くの言語をサポートしており、SRではJavaとCPPのSDKを使用しています。
*   現在、Jaegerがオブザーバビリティバックエンドとして使用されています。

### 基本的な使用方法

SRでトレーシングを有効にする手順:

1.   [Jaeger](https://www.jaegertracing.io/docs/1.31/getting-started)のインストール
    上記のガイドではDockerを使用していますが、簡単にするために[バイナリパッケージ](https://github.com/jaegertracing/jaeger/releases)をダウンロードしてローカルで実行することもできます。

```
    decster@decster-MS-7C94:~/soft/jaeger-1.31.0-linux-amd64$ ll
    total 215836
    drwxr-xr-x  2 decster decster     4096 02-05 05:01:30 ./
    drwxrwxr-x 28 decster decster     4096 05-18 18:24:07 ../
    -rwxr-xr-x  1 decster decster 19323884 02-05 05:01:31 example-hotrod*
    -rwxr-xr-x  1 decster decster 23430444 02-05 05:01:29 jaeger-agent*
    -rwxr-xr-x  1 decster decster 51694774 02-05 05:01:29 jaeger-all-in-one*
    -rwxr-xr-x  1 decster decster 41273869 02-05 05:01:30 jaeger-collector*
    -rwxr-xr-x  1 decster decster 37576660 02-05 05:01:30 jaeger-ingester*
    -rwxr-xr-x  1 decster decster 47698843 02-05 05:01:30 jaeger-query*

    decster@decster-MS-7C94:~/soft/jaeger-1.31.0-linux-amd64$ ./jaeger-all-in-one 
```

2.  FEおよびBEでトレーシングを有効にするための設定。
    現在、OpenTelemetry Java SDKはgRPCプロトコルを使用し、CPP SDKはThriftとUDPを使用しているため、エンドポイントのポートが異なります。

```
    fe.conf

    # jaeger_grpc_endpointを設定してJaegerトレーシングを有効にする
    # jaeger_grpc_endpoint = http://localhost:14250


    be.conf

    # jaeger_endpointを設定してJaegerトレーシングを有効にする
    # jaeger_endpoint = localhost:6831
```

3.  Jaeger Web UIを開く（通常は `http://localhost:16686/search` にあります）
4.  データの取り込み（ストリームロード/インサートイン）を行い、Web UIでTXNトレースを検索します

![trace_pic2.png](../../assets/trace_pic2.png)(trace_pic2.png)
![trace_pic3.png](../../assets/trace_pic3.png)(trace_pic3.png) 

### トレースの追加

*   トレースを追加する前に、トレーサー、スパン、トレース伝播などの基本概念について理解を深めるために、[オブザーバビリティの入門書](https://opentelemetry.io/docs/concepts/observability-primer)を読んでください。
*   SR内でのユーティリティクラスとその使用法を読みます：TraceManager.java（Java）、`common/tracer.h/cpp`（CPP）、現在の使用法（トランザクションのトレース（ロード／インサート／アップデート／デリート）の書き込み、BEへの伝播など）。
*   独自のトレースを追加します
