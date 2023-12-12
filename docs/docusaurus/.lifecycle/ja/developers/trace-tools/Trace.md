```yaml
---
displayed_sidebar: "Japanese"
---
### 背景

&emsp;分散トレース、一般的にはトレースとして知られているものは、マイクロサービスやサーバーレスアプリケーションなどのマルチサービスアーキテクチャを通るリクエストの経路を記録します（アプリケーションまたはエンドユーザーが行う）。トレースがないと、分散システムでのパフォーマンス問題の原因を特定することが難しくなります。これにより、アプリケーションまたはシステムの健全性の可視化が向上し、ローカルで再現が困難な動作のデバッグが可能になります。トレースは、非決定的な問題を抱えるか、ローカルで再現が難しいといった分散システムにとって不可欠です。
&emsp;トレースは、分散システムを流れるリクエスト内で何が起こっているかを細かく分解することで、分散システムのデバッグと理解を容易にします。トレースは1つまたは複数のスパンで構成されています。最初のスパンはルートスパンを表します。各ルートスパンは開始から終了までのリクエストを表しています。親の下にあるスパンは、リクエスト中に何が起こっているか（またはリクエストを構成するステップ）をより詳細に示します。多くの可観測性バックエンドは、このような滝のようなダイアグラムでトレースを視覚化します。

![trace_pic1](../../assets/trace_pic1.png)

&emsp;滝のようなダイアグラムは、ルートスパンとその子スパンとの親子関係を示しています。スパンが別のスパンをカプセル化すると、これもネスト関係を表します。
&emsp;最近SRはトレースフレームワークを追加しました。これは、分散イベントを追跡するためにopentelemetryとjaegerを活用しています。

*   Opentelemetryはインストゥルメンテーション/トレースのSDKです。開発者はこれを使用してコードをインストゥルメント化し、トレースデータを可観測性バックエンドに送信できます。多くの言語をサポートしています。私たちはSRでJavaとCPP SDKを使用しています。
*   現在、Jaegerが可観測性バックエンドとして使用されています。

### 基本的な使用法

SRでトレースを有効にする手順：

1.  [Jaeger](https://www.jaegertracing.io/docs/1.31/getting-started)をインストールしてください
    上記のガイドではDockerを使用しています。簡単にするために、[バイナリパッケージ](https://github.com/jaegertracing/jaeger/releases)をダウンロードしてローカルで実行することもできます。

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

2.  FE＆FEの設定を変更してトレースを有効にしてください
    現在、opentelemetry java＆cpp sdkでは異なるプロトコルを使用しています。Javaはgrpc protoを使用し、一方CPPはthrift＆UDPを使用しているため、エンドポイントポートは異なります。

```
    fe.conf

    # Enable jaeger tracing by setting jaeger_grpc_endpoint
    # jaeger_grpc_endpoint = http://localhost:14250


    be.conf

    # Enable jaeger tracing by setting jaeger_endpoint
    # jaeger_endpoint = localhost:6831
```

3.  通常は`http://localhost:16686/search`でJaegerウェブUIを開いてください
4.  いくつかのデータのインジェクション（streamload/insert into）を行い、ウェブUIでTXNトレースを検索してください

![trace_pic2.png](../../assets/trace_pic2.png)(trace_pic2.png) 
![trace_pic3.png](../../assets/trace_pic3.png)(trace_pic3.png) 

### トレースの追加

*   トレースを追加するには、まずトレーサーやスパン、トレース伝播などの基本的な概念に精通してください。[観測性入門](https://opentelemetry.io/docs/concepts/observability-primer)を読んでください。
*   SRでのユーティリティクラスとその使用法を把握してください: TraceManager.java（Java） `common/tracer.h/cpp (CPP)`、その現在の使用法（トランザクション（ロード/挿入/更新/削除）トレースの書き込みなど）およびBEへの伝播方法を把握してください。
*   独自のトレースを追加してください
```