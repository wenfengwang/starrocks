### Broker Load的优势

- Broker Load支持在加载过程中进行数据转换、UPSERT和DELETE操作。
- Broker Load在后台运行，客户端无需保持连接以使作业继续运行。
- Broker Load适用于长时间运行的作业，其默认超时时间为4小时。
- Broker Load支持Parquet、ORC和CSV文件格式。

### 数据流程

![Broker Load的工作流程](../broker_load_how-to-work_en.png)

1. 用户创建一个加载作业
2. 前端（FE）创建查询计划并将计划分发到后端节点（BE）
3. 后端（BE）节点从源头拉取数据并将数据加载到StarRocks