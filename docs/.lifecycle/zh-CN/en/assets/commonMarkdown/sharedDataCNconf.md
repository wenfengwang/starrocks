
**在开始CNs之前**，请在CN配置文件**cn.conf**中添加以下配置项：

```Properties
starlet_port = <starlet_port>
storage_root_path = <storage_root_path>
```

#### starlet_port

StarRocks共享数据集群的CN心跳服务端口。默认值为`9070`。

#### storage_root_path

本地缓存数据所依赖的存储卷目录和存储介质类型。多个卷由分号（;）分隔。如果存储介质为SSD，在目录末尾添加`,medium:ssd`。如果存储介质为HDD，在目录末尾添加`,medium:hdd`。例如：`/data1,medium:hdd;/data2,medium:ssd`。

`storage_root_path`的默认值为`${STARROCKS_HOME}/storage`。

在查询频繁且被查询数据是最近数据时，本地缓存会发挥作用，但也有可能需要完全关闭本地缓存，例如：

- 在Kubernetes环境中，根据需求扩展和缩减CN pods的数量，这些pods可能没有连接存储卷。
- 当被查询的数据存储在远程存储中的数据湖中且大部分是归档（旧）数据时。如果查询不频繁，数据缓存命中率将很低，可能得不偿失。

要关闭数据缓存，请设置：

```Properties
storage_root_path =
```

> **注意**
>
> 数据被缓存在目录**`<storage_root_path>/starlet_cache`**下。