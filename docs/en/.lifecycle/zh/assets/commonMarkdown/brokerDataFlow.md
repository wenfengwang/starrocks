
### Broker Load 的优势

- Broker Load 支持在加载过程中进行数据转换、UPSERT 和 DELETE 操作。
- Broker Load 在后台运行，客户端无需保持连接即可继续作业。
- 对于长时间运行的作业，Broker Load 是首选，其默认超时时间为 4 小时。
- Broker Load 支持 Parquet、ORC 和 CSV 文件格式。

### 数据流程

![Broker Load 的工作流程](../broker_load_how-to-work_en.png)

1. 用户创建一个加载作业
2. 前端（FE）创建查询计划并将该计划分发到后端节点（BE）
3. 后端节点（BE）从数据源拉取数据，并将数据加载进 StarRocks