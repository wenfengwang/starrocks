
**在启动CN之前**，请在CN配置文件**cn.conf**中添加以下配置项：

```Properties
starlet_port = <starlet_port>
storage_root_path = <storage_root_path>
```

#### starlet_port

用于StarRocks共享数据集群的CN心跳服务端口，默认值为9070。

#### storage_root_path

依赖本地缓存数据的存储卷目录和存储介质类型。多个存储卷用分号(;)分隔。如果存储介质为SSD，在目录后添加,medium:ssd；如果存储介质为HDD，在目录后添加,medium:hdd。例如：/data1,medium:hdd;/data2,medium:ssd。

storage_root_path的默认值是${STARROCKS_HOME}/storage。

在查询频繁且数据较新的情况下，本地缓存效果显著，但在某些情况下，您可能需要完全关闭本地缓存：

- 在Kubernetes环境中，根据需求动态伸缩的CN Pod可能没有挂载存储卷。
- 当查询的数据存储在远程的数据湖中，且大多数数据为归档（旧）数据时。如果查询不频繁，数据缓存的命中率可能很低，保留缓存的好处可能并不明显。

要关闭数据缓存，请设置：

```Properties
storage_root_path =
```

> **注意**
> 缓存数据存放在**`<storage_root_path>/starlet_cache`**目录下。
