
**在启动CN之前**，请在CN配置文件**cn.conf**中添加以下配置项：

```Properties
starlet_port = <starlet_port>
storage_root_path = <storage_root_path>
```

#### starlet_port

StarRocks共享数据集群的CN心跳服务端口。默认值：`9070`。

#### storage_root_path

本地缓存数据所依赖的存储卷目录及存储介质类型。多个卷用分号（;）分隔。如果存储介质为SSD，在目录末尾添加`,medium:ssd`。如果存储介质为HDD，在目录末尾添加`,medium:hdd`。例如：`/data1,medium:hdd;/data2,medium:ssd`。

`storage_root_path`的默认值是`${STARROCKS_HOME}/storage`。

当查询频繁且所查询数据较新时，本地缓存是有效的，但在某些情况下，您可能希望完全关闭本地缓存：

- 在Kubernetes环境中，如果CN Pod需要根据需求动态扩缩容，这些Pod可能没有附加的存储卷。
- 当所查询的数据存储在远程存储的数据湖中，且大部分数据为存档（旧）数据时。如果查询不频繁，数据缓存的命中率可能很低，维护缓存可能得不偿失。

要关闭数据缓存，请设置：

```Properties
storage_root_path =
```

> **注意**
> 数据被缓存在目录**`<storage_root_path>/starlet_cache`**下。