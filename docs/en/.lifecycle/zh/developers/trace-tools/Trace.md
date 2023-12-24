---
displayed_sidebar: English
---

### 背景

&emsp;分布式跟踪，通常称为跟踪，记录请求（由应用程序或最终用户发出）在多服务体系结构（如微服务和无服务器应用程序）中传播时所采用的路径。没有跟踪，要准确定位分布式系统中性能问题的原因是具有挑战性的。它提高了我们应用程序或系统健康状况的可见性，并使我们能够调试在本地难以复现的行为。对于通常存在非确定性问题或过于复杂而无法在本地复现的分布式系统来说，跟踪是至关重要的。
&emsp;跟踪通过分解请求在分布式系统中传输时发生的情况，使得调试和理解分布式系统变得不那么艰巨。一个跟踪由一个或多个 Span 组成。第一个 Span 表示根 Span。每个根 Span 代表了一个请求的整个过程。父 Span 下的 Spans 提供了请求期间发生的更详细的情况（或者构成请求的步骤）。许多可观测性后端将跟踪可视化为瀑布图，可能看起来类似于下图。

![trace_pic1](../../assets/trace_pic1.png)

&emsp;瀑布图显示了根 Span 与其子 Span 之间的父子关系。当一个 Span 封装另一个 Span 时，这也表示了一种嵌套关系。
&emsp;最近，SR 添加了一个跟踪框架。它利用 opentelemetry 和 jaeger 来跟踪系统中的分布式事件。

*   Opentelemetry 是一个检测/跟踪 SDK。开发人员可以使用它来为代码添加检测，并将跟踪数据发送到可观测性后端。它支持多种语言。在 SR 中，我们使用 java 和 CPP SDK。
*   目前，Jaeger 被用作可观测性后端。

### 基本用法

在 SR 中启用跟踪的步骤：

1.  安装 [Jaeger](https://www.jaegertracing.io/docs/1.31/getting-started)
    上面的指南使用了 docker。为简单起见，您也可以只下载 [二进制包](https://github.com/jaegertracing/jaeger/releases) 并在本地运行。

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

2.  配置 FE\&FE 以启用跟踪。
    目前，opentelemetry java 和 cpp sdk 使用不同的协议，java 使用 grpc proto，而 cpp 使用 thrift\&UDP，因此端点端口不同。

```
    fe.conf

    # 通过设置 jaeger_grpc_endpoint 启用 jaeger 跟踪
    # jaeger_grpc_endpoint = http://localhost:14250


    be.conf

    # 通过设置 jaeger_endpoint 启用 jaeger 跟踪
    # jaeger_endpoint = localhost:6831
```

3.  打开 jaeger Web UI，通常在 `http://localhost:16686/search`
4.  在 Web UI 上执行一些数据引入（流加载/插入）并搜索 TXN 跟踪

![trace_pic2.png](../../assets/trace_pic2.png)（trace_pic2.png）
![trace_pic3.png](../../assets/trace_pic3.png)（trace_pic3.png） 

### 添加跟踪

*   要添加跟踪，首先要熟悉跟踪器、跨度、跟踪传播等基本概念，请阅读[可观测性入门。](https://opentelemetry.io/docs/concepts/observability-primer)
*   阅读实用程序类及其在 SR 中的用法：TraceManager.java（java）， `common/tracer.h/cpp (cpp)`它的当前用法（如写入 txn（load\/insert\/update\/delete） 跟踪，以及它在 BE 中的传播）。
*   添加您自己的跟踪
