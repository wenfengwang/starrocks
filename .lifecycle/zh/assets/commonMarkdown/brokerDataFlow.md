
### 经纪人加载的优势

- Broker Load 支持在加载过程中进行数据转换、UPSERT 和 DELETE 操作。
- Broker Load 在后台执行，客户端无需保持连接作业也能继续进行。
- Broker Load 适用于长时间运行的作业，其默认超时时间为 4 小时。
- Broker Load 支持 Parquet、ORC 和 CSV 文件格式。

### 数据流程

![Workflow of Broker Load](../broker_load_how-to-work_en.png)

1. 用户创建加载任务
2. 前端（FE）制定查询计划并将该计划分发至后端节点（BE）
3. 后端节点（BE）从数据源拉取数据，并将其加载进 StarRocks。
