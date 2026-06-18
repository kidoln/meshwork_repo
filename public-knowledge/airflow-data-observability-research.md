---
id: 50622B64-AE1A-419F-9C2E-A43DDACB4929
title: airflow-data-observability-research.md
status: draft
created-at: 2026-06-18T07:17:37Z
updated-at: 2026-06-18T07:19:48Z
---

# Apache Airflow 数据可观测性调研报告

## 概述

Apache Airflow 作为开源工作流编排平台，在数据可观测性方面提供了丰富的原生能力和扩展选项。本报告从多个维度分析 Airflow 在数据可观测性方面的能力和特色。

## 一、原生监控和日志能力

### 1.1 内置 Web UI 监控

Airflow 提供了功能完善的 Web UI，包含多种监控视图：





* **Grid View**：显示 DAG 和任务的执行状态
* **Graph View**：可视化 DAG 的依赖关系
* **Tree View**：展示任务实例的历史执行情况
* **Code View**：查看 DAG 代码
* **Task Instance Details**：查看任务实例的详细信息，包括执行日志、XCom 数据、渲染模板等

### 1.2 内置报告系统

Airflow 提供了丰富的报告功能，可通过 Browse 菜单访问：

* **DAG Runs**：DAG 运行历史记录
* **Jobs**：后台任务执行情况
* **Audit Logs**：审计日志
* **Task Instances**：任务实例详细信息
* **Task Reschedules**：任务重新调度记录
* **Triggers**：触发器记录
* **SLA Misses**：SLA 违规记录
* **DAG Dependencies**：DAG 依赖关系

### 1.3 日志系统

Airflow 提供了完整的日志记录功能：

* **任务执行日志**：记录每个任务的执行过程和输出
* **调度器日志**：记录调度器的运行状态
* **Web 服务器日志**：记录 Web 服务器的访问和错误信息
* **Worker 日志**：记录任务执行器的工作状态

## 二、数据血缘和可观测性

### 2.1 OpenLineage 集成

Airflow 通过 OpenLineage 提供了强大的数据血缘追踪能力：

**核心组件：**

* **OpenLineage Airflow Provider**：作为适配器集成 OpenLineage 与 Airflow
* **OpenLineage Client**：负责构建和发送 OpenLineage 事件到后端

**核心概念：**

* **Job（作业）**：对应 Airflow 中的任务或 DAG
* **Dataset（数据集）**：代表数据集合，如数据库表或 S3 存储桶
* **Run（运行）**：作业执行实例，每次 DAG 和任务运行都会生成
* **Facet（方面）**：关于作业、数据集或运行的元数据片段

**能力特色：**

* 快速定位任务失败的根本原因
* 可视化作业和数据集之间的关系，包括列级血缘
* 识别敏感数据在组织中的使用位置
* 支持列级血缘追踪（针对特定操作符）
* 从 Airflow 2.10 版本开始，自动从支持的 hooks 收集血缘

> 📖 详细信息请参考：[[raw-sources/integrate-openlineage-and-airflow-astronomer-documentation.md]]

### 2.2 数据血缘的价值

* **故障恢复**：通过识别上游数据集问题，加快复杂故障的恢复速度
* **团队协作**：可视化数据资产的使用范围，减少调查时间
* **合规性**：全面理解数据在组织中的使用情况，确保符合数据法规

## 三、数据质量监控

### 3.1 DAG 级别数据质量检查

Airflow 提供了多种 SQL 检查操作符用于数据质量验证：

**SQLColumnCheckOperator**：列级别验证

* null_check：NULL 值检查
* unique_check：重复值检查
* distinct_check：唯一值检查
* min/max：最小/最大值验证

**SQLTableCheckOperator**：表级别业务规则验证

* 支持跨列业务规则
* 聚合验证
* 自定义 SQL 表达式检查

**SQLValueCheckOperator**：值比较检查

* 与期望值比较
* 支持容差百分比设置

**SQLIntervalCheckOperator**：历史数据比较

* 与历史数据比较
* 识别异常值和趋势

**SQLThresholdCheckOperator**：阈值验证

* 最小/最大阈值检查
* 支持 SQL 表达式作为阈值

**SQLCheckOperator**：通用复杂查询

* 支持复杂多表查询
* 最灵活的检查方式

> 📖 详细信息请参考：[[raw-sources/data-quality-and-airflow-astronomer-documentation.md]]

### 3.2 第三方数据质量框架集成

**Great Expectations 集成：**

* GXValidateDataFrameOperator：内存 DataFrame 验证
* GXValidateBatchOperator：数据库/文件系统验证
* GXValidateCheckpointOperator：完整的 GX 功能和操作

**dbt 测试集成：**

* Schema 测试：unique、not_null、accepted_values、relationships
* 自定义测试：基于 SQL 的业务特定验证

**Soda Core 集成：**

* 声明式 YAML 检查
* 通过 @task.bash 运行 Soda 扫描

### 3.3 平台级别数据质量监控

**Astro Observe 数据质量功能：**

* **表监控**：行数变化、列空值百分比、表结构变化
* **数据产品监控**：数据产品健康度和管道故障
* **自定义 SQL 监控**：使用 SQL 查询监控数据质量或业务逻辑

**监控能力：**

* 表流行度评分：帮助优先监控频繁访问的表
* 表级血缘：跨 DAG 显示上游和下游资产
* 影响分析：评估上游和下游资产的影响
* 事件驱动检查：数据落地时立即运行

> 📖 详细信息请参考：[[raw-sources/enable-data-observability-jobs-monitoring-for-apache-airflow.md]]

## 四、指标和监控集成

### 4.1 StatsD 指标导出

Airflow 原生支持 StatsD 协议导出指标：

**配置示例：**

```python
AIRFLOW__METRICS__STATSD_ON=True
AIRFLOW__METRICS__STATSD_HOST=statsd-exporter
AIRFLOW__METRICS__STATSD_PORT=9125
AIRFLOW__METRICS__STATSD_PREFIX=airflow
```

**导出的指标类型：**

* 执行器指标：open_slots、queued_tasks、running_tasks
* DAG 处理指标：dag_processing.processes、dag_loading_duration
* 调度器心跳：scheduler_heartbeat
* 任务执行指标：成功率、失败率、执行时间

### 4.2 Prometheus 和 Grafana 集成

**集成架构：**

1. Airflow 组件（Webserver、Scheduler、Worker）发送 StatsD 指标
2. statsd_exporter 将 StatsD 指标转换为 Prometheus 格式
3. Prometheus 服务器定期抓取指标
4. Grafana 仪表板可视化指标

**集成优势：**

* 时间序列数据存储
* 强大的可视化能力
* 告警和通知功能
* 可扩展的监控架构

> 📖 详细信息请参考：[[raw-sources/monitoring-apache-airflow-using-prometheus.md]]

### 4.3 REST API 监控

Airflow REST API 提供了编程方式获取指标：

**支持的端点：**

* `/dags`：获取所有 DAG 信息
* `/dagRuns`：获取 DAG 运行记录
* `/taskInstances`：获取任务实例信息
* `/jobs`：获取后台任务信息

**使用场景：**

* 导出指标到外部数据库
* 使用 BI 工具（Looker、Superset、Tableau）创建仪表板
* 自定义监控解决方案

## 五、告警和通知

### 5.1 内置通知机制

**邮件通知：**

* email_on_failure：任务失败时发送邮件
* email_on_retry：任务重试时发送邮件
* email_on_success：任务成功时发送邮件

**回调函数：**

* on_failure_callback：失败回调
* on_success_callback：成功回调
* on_retry_callback：重试回调
* sla_miss_callback：SLA 违规回调

### 5.2 通知器（Notifiers）

**SlackNotifier：**

```python
from airflow.providers.slack.notifications.slack_notifier import SlackNotifier

@task(
    on_failure_callback=SlackNotifier(
        slack_conn_id="slack_default",
        text="Data quality checks failed for table: `{{ task.table }}`!",
        channel="#data-alerts",
    )
)
def my_task():
    pass
```

**AppriseNotifier：**

* 支持 100+ 通知服务
* 统一接口集成 Slack、Email、PagerDuty、Teams 等

### 5.3 SLA 监控

**SLA 配置：**

* 任务级别 SLA：使用 `sla` 参数设置时间增量
* DAG 级别 SLA：为整个 DAG 设置 SLA
* SLA 违规回调：自定义 SLA 违规处理逻辑

**注意事项：**

* SLA 告警仅对计划 DAG 有效
* 手动触发的 DAG 不会触发 SLA 违规告警
* 任务的 SLA 基于 DAG 开始时间计算，可能产生误报

> 📖 详细信息请参考：[[raw-sources/airflow-monitoring-slas-dags-best-practices.md]]

## 六、高级可观测性功能

### 6.1 XCom 数据追踪

XCom（跨通信）允许任务之间交换数据：

**监控用途：**

* 共享状态消息或指标报告
* 错误代码、执行时间或状态指示器
* 使用 Grafana 聚合和显示 XCom 指标
* 分析数据交换的频率和量级

### 6.2 传感器（Sensors）监控

传感器用于等待外部条件满足：

**监控应用：**

* 数据到达监控
* 分区可用性检查
* 外部系统状态监控
* 文件系统变化检测

### 6.3 自定义指标和日志

**自定义指标：**

* 通过 StatsD 发送自定义业务指标
* 使用 XCom 传递监控数据
* 集成第三方监控工具

**自定义日志：**

* 结构化日志记录
* 上下文信息添加
* 日志级别控制

## 七、可观测性最佳实践

### 7.1 分层监控策略

**第一层：DAG 级别检查**

* 在管道内运行
* 可以阻止管道执行
* 需要 DAG 代码更改
* 关键验证的理想选择

**第二层：平台级别检查**

* 独立于 DAG 执行
* 非阻塞：仅告警不停止管道
* 通过 UI 配置，无需部署
* 即使 Airflow 有问题也能监控数据

### 7.2 监控模式建议

**推荐顺序：**

1. 列检查（字段级别验证）
2. 表检查（业务逻辑）
3. 下游处理

**组织方式：**

* 使用任务组组织数据质量检查
* 利用 default_args 参数配置所有检查的通知
* 结合触发规则控制失败行为

### 7.3 告警策略

**告警级别：**

* **Critical**：立即阻断管道
* **Warning**：记录但不阻止
* **Info**：仅记录信息

**告警渠道：**

* 开发时间：邮件、Slack
* 运维时间：PagerDuty、Opsgenie
* 管理层：仪表板、定期报告

## 八、工具选择建议

### 8.1 不同场景的工具选择

| 场景     | 推荐工具                        | 理由             |
| ------ | --------------------------- | -------------- |
| 简单列验证  | SQLColumnCheckOperator      | 无需额外软件，SQL 实现  |
| 复杂业务规则 | SQLTableCheckOperator       | 支持聚合和多列验证      |
| 历史数据比较 | SQLIntervalCheckOperator    | 识别趋势和异常        |
| 内存数据验证 | GXValidateDataFrameOperator | 直接验证 DataFrame |
| 数据库验证  | GXValidateBatchOperator     | 完整的 GX 功能      |
| 无代码监控  | Astro Observe               | UI 配置，无需部署     |
| 自定义可视化 | REST API + BI 工具            | 灵活的仪表板创建       |
| 基础设施监控 | Prometheus + Grafana        | 时间序列监控         |

### 8.2 集成复杂度评估

**低复杂度：**

* Airflow 内置 UI 和通知
* SQL 检查操作符
* 基础邮件告警

**中等复杂度：**

* OpenLineage 集成
* Great Expectations 集成
* REST API 监控

**高复杂度：**

* Prometheus + Grafana 部署
* 自定义监控解决方案
* 多工具集成

## 九、总结和建议

### 9.1 Airflow 数据可观测性优势

1. **原生功能丰富**：内置 UI、日志、报告系统
2. **扩展性强**：支持多种第三方工具集成
3. **灵活性高**：可根据需求选择合适的监控级别
4. **社区支持好**：活跃的社区和丰富的文档

### 9.2 实施建议

**起步阶段：**

* 使用 Airflow 内置 UI 和通知
* 实施基础的 SQL 检查操作符
* 配置邮件告警

**发展阶段：**

* 集成 OpenLineage 进行数据血缘追踪
* 部署 Prometheus + Grafana 进行指标监控
* 实施数据质量框架（Great Expectations 或 Soda）

**成熟阶段：**

* 使用平台级别监控（如 Astro Observe）
* 建立完整的监控仪表板
* 实施自动化告警和故障处理

### 9.3 注意事项

1. **性能影响**：监控不应影响管道性能
2. **告警疲劳**：合理设置告警阈值，避免过多告警
3. **维护成本**：复杂监控方案需要专门的维护
4. **安全性**：确保监控数据的安全性，特别是包含敏感信息时

## 参考资源

本报告基于以下参考文档整理：

* [[raw-sources/airflow-apache-org.md]] - Apache Airflow 官方文档 - Logging & Monitoring
* [[raw-sources/data-quality-and-airflow-astronomer-documentation.md]] - Astronomer - Data Quality and Airflow
* [[raw-sources/integrate-openlineage-and-airflow-astronomer-documentation.md]] - Astronomer - Integrate OpenLineage and Airflow
* [[raw-sources/monitoring-apache-airflow-using-prometheus.md]] - Red Hat - Monitoring Apache Airflow using Prometheus
* [[raw-sources/enable-data-observability-jobs-monitoring-for-apache-airflow.md]] - Datadog - Data Observability for Apache Airflow
* [[raw-sources/airflow-monitoring-slas-dags-best-practices.md]] - Astronomer - Airflow Monitoring: SLAs, DAGs, & Best Practices

***

*本报告基于公开资料整理，涵盖了 Airflow 在数据可观测性方面的主要能力和特色。*
