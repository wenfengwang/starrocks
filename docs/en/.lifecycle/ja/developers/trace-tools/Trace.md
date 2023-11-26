---
displayed_sidebar: "Japanese"
---

### 背景

&emsp;分散トレース、より一般的にはトレースと呼ばれるものは、マイクロサービスやサーバーレスアプリケーションなどのマルチサービスアーキテクチャを通じてリクエスト（アプリケーションまたはエンドユーザーによって行われる）が伝播する経路を記録します。トレースがないと、分散システムのパフォーマンスの問題の原因を特定することは困難です。トレースは、アプリケーションまたはシステムの健全性を可視化し、ローカルで再現が困難な動作をデバッグするための手段を提供します。トレースは、非決定的な問題を持つか、ローカルで再現するのが困難な分散システムにとって不可欠です。
&emsp;トレースは、リクエストが分散システムを通過する際に何が起こるかを分解することにより、分散システムのデバッグと理解を容易にします。トレースは1つ以上のスパンで構成されています。最初のスパンはルートスパンを表します。各ルートスパンは、開始から終了までのリクエストを表します。親の下にあるスパンは、リクエスト中に何が起こるか（またはリクエストを構成するステップ）をより詳細に示します。多くのオブザーバビリティバックエンドは、トレースをウォーターフォールダイアグラムとして可視化します。以下のようなイメージです。

![trace_pic1](../../assets/trace_pic1.png)

&emsp;ウォーターフォールダイアグラムは、ルートスパンとその子スパンの親子関係を示しています。スパンが別のスパンをカプセル化する場合、これはネストされた関係を表します。
&emsp;最近、SRにトレースフレームワークが追加されました。これは、opentelemetryとjaegerを使用してシステム内の分散イベントをトレースするために使用されます。

*   Opentelemetryは、インストゥルメンテーション/トレースのSDKです。開発者はこれを使用してコードをインストゥルメント化し、トレースデータをオブザーバビリティバックエンドに送信できます。多くの言語をサポートしています。SRでは、JavaとCPPのSDKを使用しています。
*   現在、Jaegerがオブザーバビリティバックエンドとして使用されています。

### 基本的な使用方法

SRでトレースを有効にする手順：

1.  [Jaeger](https://www.jaegertracing.io/docs/1.31/getting-started)をインストールします。
    上記のガイドではDockerを使用しています。簡単のため、[バイナリパッケージ](https://github.com/jaegertracing/jaeger/releases)をダウンロードしてローカルで実行することもできます。

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

2.  FE\&FEを構成してトレースを有効にします。
    現在、opentelemetryのJavaとCPPのSDKは異なるプロトコルを使用しています。Javaはgrpcプロトを使用し、CPPはthrift\&UDPを使用しているため、エンドポイントのポートが異なります。

```
    fe.conf

    # jaeger_grpc_endpointを設定してJaegerトレースを有効にする
    # jaeger_grpc_endpoint = http://localhost:14250


    be.conf

    # jaeger_endpointを設定してJaegerトレースを有効にする
    # jaeger_endpoint = localhost:6831
```

3.  JaegerのWeb UIを開きます。通常は`http://localhost:16686/search`です。
4.  データのインジェクション（ストリームロード/挿入）を行い、Web UI上でTXNトレースを検索します。

![trace_pic2.png](../../assets/trace_pic2.png)(trace_pic2.png) 
![trace_pic3.png](../../assets/trace_pic3.png)(trace_pic3.png) 

### トレースの追加

*   トレースを追加するには、トレーサー、スパン、トレースの伝播などの基本的な概念について理解する必要があります。[オブザーバビリティプライマー](https://opentelemetry.io/docs/concepts/observability-primer)を読んでください。
*   SRでのユーティリティクラスとその使用方法を確認してください：TraceManager.java（Java）`common/tracer.h/cpp（CPP）`、現在の使用方法（トランザクション（ロード/挿入/更新/削除）トレースの書き込み、およびBEへの伝播）。
*   独自のトレースを追加してください。
