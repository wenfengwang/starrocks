### Broker Load 的优势

- Broker Load 支持数据转换、UPSERT 和 DELETE 操作。
- Broker Load 在后台运行，客户端无需保持连接即可继续作业。
- Broker Load 适用于长时间运行的作业，默认超时时间为 4 小时。
- Broker Load 支持 Parquet、ORC 和 CSV 文件格式。

### 数据流

![Broker Load 的工作流程](../broker_load_how-to-work_en.png)

1. 用户创建一个加载作业
2. 前端 (FE) 创建查询计划，并将计划分发至后端节点 (BE)
3. 后端节点 (BE) 从源端拉取数据，并将数据加载到 StarRocks 中