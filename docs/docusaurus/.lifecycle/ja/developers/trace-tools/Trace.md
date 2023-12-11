---
displayed_sidebar: "Japanese"
---

### バックグラウンド

&emsp;分散トレース、一般的にはトレースとして知られているものは、マイクロサービスやサーバーレスアプリケーションなどのマルチサービスアーキテクチャを通るリクエストのパスを記録します（アプリケーションまたはエンドユーザーによって行われたもの）。トレースがないと、分散システムのパフォーマンスの問題の原因を特定することは難しいです。それは、アプリケーションまたはシステムの健康状態を可視化し、ローカルで再現が難しい振る舞いをデバッグすることを可能にします。トレースは、典型的には非決定論的な問題を抱えるか、ローカルで再現が難しいような複雑なものに必要です。
&emsp;トレースは、分散システムを理解し、デバッグする際に、リクエスト内で何が起こっているかを詳細に分析することで、より取り組みやすくします。トレースは1つ以上のスパンで構成されています。最初のスパンはルートスパンを表し、各ルートスパンは開始から終了までのリクエストを表します。親の下にあるスパンは、リクエスト中に何が起こっているか（またはリクエストを構成するステップ）のより詳細なコンテキストを提供します。多くのオブザーバビリティのバックエンドは、このようなウォーターフォール図としてトレースを視覚化します。

![trace_pic1](../../assets/trace_pic1.png)

&emsp;ウォーターフォール図は、ルートスパンとその子スパンの親子関係を示します。スパンが別のスパンをカプセル化すると、それはネストした関係を表します。
&emsp;最近、SRはトレースフレームワークを追加しました。それはopentelemetryとjaegerを利用して、システム内の分散イベントをトレースします。

*   Opentelemetryはインストゥルメンテーション/トレースのSDKです。開発者はそれを使用してコードにインストゥルメントを施し、トレースデータをオブザーバビリティバックエンドに送信できます。多くの言語をサポートしています。SRでは、javaとCPP SDKを使用しています。
*   現在、Jaegerがオブザーバビリティバックエンドとして使用されています。

### 基本的な使用法

SRでトレースを有効にする手順:

1.  [Jaeger](https://www.jaegertracing.io/docs/1.31/getting-started)をインストールします
    上記のガイドではdockerを使用しています。簡単にするために、[バイナリパッケージ](https://github.com/jaegertracing/jaeger/releases)をダウンロードしてローカルで実行することもできます。

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
    現在、opentelemetryのjavaおよびCPP SDKは異なるプロトコルを使用しています。javaはgrpcプロトを使用しており、一方cppはthrift\&UDPを使用しているため、エンドポイントのポートも異なります。

```
    fe.conf

    # jaeger_grpc_endpointを設定してjaegerトレースを有効化します
    # jaeger_grpc_endpoint = http://localhost:14250


    be.conf

    # jaeger_endpointを設定してjaegerトレースを有効化します
    # jaeger_endpoint = localhost:6831
```

3.  通常は`http://localhost:16686/search`でjaeger web UIを開きます
4.  いくつかのデータのインジェスチョン（ストリームロード/挿入）を行い、web UIでTXNトレースを検索します

![trace_pic2.png](../../assets/trace_pic2.png)(trace_pic2.png) 
![trace_pic3.png](../../assets/trace_pic3.png)(trace_pic3.png) 

### トレースの追加

*   トレースを追加するには、最初にトレーサー、スパン、トレースの伝播などの基本的な概念に慣れることが必要です。[オブザーバビリティプライマー](https://opentelemetry.io/docs/concepts/observability-primer)を読んでください。
*   SRでのユーティリティクラスとその使用法を確認してください： TraceManager.java（java）、`common/tracer.h/cpp (cpp)`、それの現在の使用法（トランザクションの書き込み（load/insert/update/delete）トレース、およびそのBEへの伝播）。
*   独自のトレースを追加してください