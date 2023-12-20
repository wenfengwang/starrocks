---
displayed_sidebar: English
---

### 背景

&emsp;分布式追踪（通常称为追踪）记录请求（由应用程序或最终用户发出）在多服务架构（如微服务和无服务器应用程序）中传播时所采用的路径。如果没有追踪，就很难查明分布式系统中性能问题的原因。它提高了我们的应用程序或系统健康状况的可见性，并让我们能够调试难以在本地重现的行为。追踪对于分布式系统至关重要，这些系统通常存在不确定性问题，或者过于复杂而无法在本地重现。
&emsp;追踪通过分解请求在分布式系统中流动时发生的情况，使得调试和理解分布式系统变得不那么令人生畏。一条追踪由一个或多个 Span 组成。第一个 Span 代表根 Span。每个根 Span 代表一个从开始到结束的请求。子 Span 则提供了请求期间发生的情况（或构成请求的步骤）的更深入的上下文。许多可观测性后端将追踪可视化为瀑布图，可能看起来像这张图片。

![trace_pic1](../../assets/trace_pic1.png)

&emsp;瀑布图显示了根 Span 及其子 Span 之间的父子关系。当一个 Span 封装了另一个 Span 时，这也表示了嵌套关系。
&emsp;最近 SR 添加了一个追踪框架。它利用 OpenTelemetry 和 Jaeger 来追踪系统中的分布式事件。

*   OpenTelemetry 是一个仪器化/追踪 SDK。开发人员可以使用它来检测代码并将追踪数据发送到可观测性后端。它支持多种语言。我们在 SR 中使用 Java 和 CPP SDK。
*   目前，Jaeger 被用作可观测性后端。

### 基本用法

在 SR 中启用追踪的步骤：

1.  安装 [Jaeger](https://www.jaegertracing.io/docs/1.31/getting-started)
上面的指南使用了 Docker。为了简单起见，您也可以只下载 [binary package](https://github.com/jaegertracing/jaeger/releases) 并在本地运行。

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

2.  配置 FE\&BE 以启用追踪。目前 OpenTelemetry Java 和 CPP SDK 使用不同的协议，Java 使用 gRPC proto，而 CPP 使用 Thrift\&UDP，因此端点端口不同。

```
    fe.conf

    # Enable Jaeger tracing by setting jaeger_grpc_endpoint
    # jaeger_grpc_endpoint = http://localhost:14250


    be.conf

    # Enable Jaeger tracing by setting jaeger_endpoint
    # jaeger_endpoint = localhost:6831
```

3.  打开 Jaeger Web UI，通常在 `http://localhost:16686/search`
4.  进行一些数据摄取（StreamLoad/INSERT INTO）并在 Web UI 上搜索 TXN 追踪

![trace_pic2.png](../../assets/trace_pic2.png)![trace_pic3.png](../../assets/trace_pic3.png)

### 添加追踪

*   要添加追踪，首先要熟悉基本概念，如 Tracer、Span、追踪传播等，阅读[可观测性入门](https://opentelemetry.io/docs/concepts/observability-primer)。
*   阅读实用程序类及其在 SR 中的用法：`TraceManager.java`(Java)、`common/tracer.h/cpp` (CPP)，了解其当前用法（如写入 TXN（Load/Insert/Update/Delete）追踪，以及它的传播到 BE）。
*   添加您自己的追踪