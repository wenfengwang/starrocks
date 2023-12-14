---
displayed_sidebar: "Chinese"
---

### 背景

&emsp;分布式跟踪，更常被称为跟踪，记录了请求（由应用程序或最终用户发起）在多服务架构中传播的路径，比如微服务和无服务器应用程序。没有跟踪，要确定分布式系统中性能问题的原因是具有挑战性的。它改善了我们应用程序或系统的健康状态的可见性，并让我们调试在本地难以复现的行为。对于通常存在非确定性问题或在本地复现过于复杂的分布式系统而言，跟踪是必不可少的。
&emsp;跟踪通过分解请求在分布式系统中的发生情况，使得调试和理解分布式系统变得不那么可怕。一个跟踪由一个或多个跨度（Span）组成。第一个跨度代表根跨度。每个根跨度代表着一个从开始到结束的请求。父跨度下面的跨度提供了请求期间发生的更深层次的上下文（或者组成请求的步骤）。许多可观察性后端将跟踪可视化为瀑布图，可能看起来像这张图片。

![trace_pic1](../../assets/trace_pic1.png)

&emsp;瀑布图展示了根跨度和其子跨度之间的父子关系。当一个跨度封装另一个跨度时，这也代表着一个嵌套关系。
&emsp;最近，SR添加了一个跟踪框架。它利用了opentelemetry和jaeger来跟踪系统中的分布式事件。

*   Opentelemetry是一个工具/跟踪SDK。开发人员可以使用它来为代码进行仪器化，并将跟踪数据发送到可观察性后端。它支持多种语言。在SR中，我们使用java和CPP SDK。
*   目前，Jaeger被用作可观察性后端。

### 基本用法

在SR中启用跟踪的步骤：

1.  安装[Jaeger](https://www.jaegertracing.io/docs/1.31/getting-started)
    上面的指南使用了docker。为了简单起见，你也可以只下载[二进制包](https://github.com/jaegertracing/jaeger/releases)并在本地运行。

```
    decster@decster-MS-7C94:~/soft/jaeger-1.31.0-linux-amd64$ ll
    总共 215836
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

2.  配置前端和后端以启用跟踪。
    目前，opentelemetry java和CPP SDK使用不同的协议，java使用grpc proto，而CPP使用thrift和UDP，因此端口是不同的。

```
    fe.conf

    # 通过设置jaeger_grpc_endpoint启用jaeger跟踪
    # jaeger_grpc_endpoint = http://localhost:14250


    be.conf

    # 通过设置jaeger_endpoint启用jaeger跟踪
    # jaeger_endpoint = localhost:6831
```

3.  在jaeger web UI中打开，通常为`http://localhost:16686/search`
4.  进行一些数据摄入（流加载/插入），并在web UI上搜索TXN跟踪

![trace_pic2.png](../../assets/trace_pic2.png)(trace_pic2.png) 
![trace_pic3.png](../../assets/trace_pic3.png)(trace_pic3.png) 

### 添加跟踪

*   要添加跟踪，首先要熟悉基本概念，如跟踪器（tracer）、跨度（span）、跟踪传播，请阅读[可观察性入门指南](https://opentelemetry.io/docs/concepts/observability-primer)。
*   阅读实用类及其在SR中的使用情况：TraceManager.java（java）`common/tracer.h/cpp (cpp)`，它目前的用法（如写入txn（加载/插入/更新/删除）跟踪及其传播到BE）。
*   添加你自己的跟踪