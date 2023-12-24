
**在启动CN之前**，请在CN配置文件**cn.conf**中添加以下配置项：

```Properties
starlet_port = <starlet_port>
storage_root_path = <storage_root_path>
```

#### starlet_port

StarRocks 共享数据集群的 CN 心跳服务端口。默认值为 `9070`。

#### storage_root_path

本地缓存数据依赖的存储卷目录和存储介质类型。多个卷之间用分号（;）分隔。如果存储介质是 SSD，在目录末尾添加`,medium:ssd`。如果存储介质是 HDD，在目录末尾添加`,medium:hdd`。例如：`/data1,medium:hdd;/data2,medium:ssd`。

`storage_root_path` 的默认值为`${STARROCKS_HOME}/storage`。

在查询频繁且查询的数据是最新的情况下，本地缓存是有效的。但在某些情况下，您可能希望完全关闭本地缓存。

- 在 Kubernetes 环境中，CN Pod 可能根据需求扩展或缩减数量，而未附加存储卷。
- 当被查询的数据位于远程存储的数据湖中，并且其中大部分是存档（旧）数据时。如果查询不频繁，则数据缓存的命中率较低，可能不值得使用缓存。

要关闭数据缓存，请执行以下操作：

```Properties
storage_root_path =
```

> **注意**
>
> 数据缓存在目录 **`<storage_root_path>/starlet_cache`** 下。