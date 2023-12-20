---
displayed_sidebar: English
---

### 背景

分布式追踪，通常简称为追踪，记录了请求（由应用程序或终端用户发起）在多服务架构中的传播路径，例如在微服务和无服务器应用程序中。没有追踪功能，我们很难确定分布式系统中性能问题的根源。追踪提高了我们对应用程序或系统健康状况的可见性，并帮助我们调试那些在本地难以重现的行为。对于通常存在不确定性问题或过于复杂以至于无法在本地重现的分布式系统来说，追踪至关重要。追踪通过分解请求在分布式系统中流动时的各个环节，使得调试和理解分布式系统变得不再令人望而生畏。一个追踪由一个或多个跨度（Span）组成。第一个跨度代表根跨度（Root Span），每个根跨度代表一个请求从开始到结束的全过程。父级跨度下的子跨度提供了关于请求期间发生的详细上下文（或构成请求的各个步骤）。许多可观测性后端将追踪以瀑布图的形式进行可视化，可能看起来像这张图片。

![trace_pic1](../../assets/trace_pic1.png)

&emsp;瀑布图展示了根跨度与其子跨度之间的父子关系。当一个跨度包含另一个跨度时，这也表示了它们之间的嵌套关系。最近SR引入了一个追踪框架，它利用OpenTelemetry和Jaeger来追踪系统中的分布式事件。

*   OpenTelemetry是一个用于检测和追踪的SDK。开发者可以使用它来为代码添加检测点，并将追踪数据发送到可观测性后端。它支持多种编程语言。在SR中，我们使用Java和C++的SDK。
*   目前，Jaeger被用作可观测性后端。

### 基本使用方法

在SR中启用追踪的步骤：

1.  安装[Jaeger](https://www.jaegertracing.io/docs/1.31/getting-started)
上述指南使用了Docker。为了简便，您也可以只需下载[binary package](https://github.com/jaegertracing/jaeger/releases)并在本地运行。

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

2.  配置前端和后端以启用追踪。目前OpenTelemetry的Java和C++ SDK使用不同的协议，Java使用gRPC协议，而C++使用Thrift和UDP，因此它们的端点端口不同。

```
    fe.conf

    # Enable jaeger tracing by setting jaeger_grpc_endpoint
    # jaeger_grpc_endpoint = http://localhost:14250


    be.conf

    # Enable jaeger tracing by setting jaeger_endpoint
    # jaeger_endpoint = localhost:6831
```

3.  打开Jaeger Web UI，通常位于 http://localhost:16686/search
4.  执行一些数据导入操作（如流式加载/插入数据），然后在Web UI上搜索事务追踪信息

![trace_pic2.png](../../assets/trace_pic2.png)![trace_pic3.png](../../assets/trace_pic3.png)

### 添加追踪

*   要添加追踪，请首先熟悉基础概念，例如追踪器、跨度、追踪传播等，阅读有关[可观测性的基础知识](https://opentelemetry.io/docs/concepts/observability-primer)。
*   阅读并理解SR中的工具类及其用法：TraceManager.java（Java）和common/tracer.h/cpp（C++），了解其当前的用法（例如，编写事务（加载/插入/更新/删除）追踪及其在后端的传播）。
*   添加您自己的追踪记录。
