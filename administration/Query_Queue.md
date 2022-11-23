# 查询排队【公测中】

本文介绍如何使用查询排队功能。

自 2.5 版本，StarRocks 支持查询排队功能，集群可以在 BE 节点的资源超过限制时，对查询进行排队，避免 BE 资源过载时继续被投递更多的查询任务，导致资源过载的程度加剧。

通过查询排队功能，可以设置集群 CPU、内存、并发资源的阈值。当新的查询到来时，如果任意一个 BE 的以下任意资源达到阈值，就对查询进行排队。

- BE 的 CPU 使用率；

- BE 的内存使用率；

- BE 上正在运行的查询数量。

## 开启查询排队功能

通过以下三个 Global Session 变量来分别控制是否对SELECT 查询、统计信息查询、INSERT INTO 开启排队功能，默认都是关闭 `false`。

```sql
SET GLOBAL query_queue_select_enable = true;
SET GLOBAL query_queue_statistic_enable = true;
SET GLOBAL query_queue_insert_enable = true;
```

## 设置资源阈值

可以通过如下三个 Global Session 变量来控制对应的资源阈值，以及是否使用该资源阈值进行排队。

注意，BE 默认每 1s 才会向 FE 汇报一次资源使用量，FE 根据汇报的信息判断是否需要对查询进行排队。可以通过修改 BE 的配置 `report_resource_usage_interval_ms` 来控制汇报的周期。

| **配置**                            | **含义**                                                     | **默认值** |
| ----------------------------------- | ------------------------------------------------------------ | ---------- |
| query_queue_concurrency_limit       | 一个 BE 上正在运行的查询数量超过该阈值，则新的查询需要排队。>0 才生效。 | 0          |
| query_queue_mem_used_pct_limit      | 一个 BE 的内存使用率超过该阈值，则新的查询需要排队。取值范围：[0, 1]。>0 才生效。 | 0          |
| query_queue_cpu_used_permille_limit | 一个 BE 的 CPU 使用率*1000 超过该阈值，则新的查询需要排队。 取值范围：[0, 1000]。>0 才生效。 | 0          |

## 配置队列

可以通过 Gobal Session 变量来控制队列的容量，以及查询的最大排队时间。

| **配置**                           | **含义**                                                     | **默认值** |
| ---------------------------------- | ------------------------------------------------------------ | ---------- |
| query_queue_pending_timeout_second | 一个查询最大的排队时间，超过该阈值则拒绝执行该查询。         | 300s       |
| query_queue_max_queued_queries     | 允许排队查询的最大数量，超过该阈值后，直接拒绝新到来的需要排队的查询。>0 才生效。 | 0          |

## 获取排队信息

通过执行 `show backends` 可以获取每个 BE 正在运行的查询数量、内存使用率、CPU 使用率。

```plain text
MySQL> show backends \G
***************************[ 1. row ]***************************
...
NumRunningQueries     | 0
MemUsedPct            | 12.21 %
CpuUsedPct            | 6.3 %
***************************[ 2. row ]***************************
...
NumRunningQueries     | 0
MemUsedPct            | 12.11 %
CpuUsedPct            | 6.2 %

```

通过执行 `show prcesslist`，可以通过字段 `IsPending` 判断每个连接的查询是否处于排队状态。

```plain text
MySQL> show prcesslist
+------+--------+---------------------+---------+---------+---------------------+------+-------+------------------+-----------+
| Id   |  User  | Host                | Db      | Command | ConnectionStartTime | Time | State | Info             | IsPending |
+------+--------+---------------------+---------+---------+---------------------+------+-------+------------------+-----------+
| <id> | <user> | <host>              | <db>    | Query   | 2022-11-23 17:12:19 | 0    | OK    | show processlist | false     |
+------+--------+---------------------+---------+---------+---------------------+------+-------+------------------+-----------+
```

审计日志 audit.log 的 `PendingTimeMs` 会输出每个查询排队等待的时间，单位为毫秒。

通过 FE 的 metrics 的如下字段可以获取查询队列的信息：

- starrocks_fe_query_queue_pending 正在排队的查询数量。
- starrocks_fe_query_queue_total 进行过排队的查询总数量。
- starrocks_fe_query_queue_timeout 排队超时的查询总数量。
