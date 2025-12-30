# Task 模块 - Celery 异步任务

[根目录](../../CLAUDE.md) > [backend](../) > **app/task**

> 最后更新：2025-12-30 11:02:40 +0800

## 变更记录 (Changelog)

### 2025-12-30
- **初始化模块文档**：首次生成 task 模块架构文档

---

## 模块职责

`task` 模块是基于 **Celery** 的分布式异步任务调度系统，支持定时任务、延迟任务、周期任务等功能。

**核心功能：**
- **任务调度**：创建、启动、暂停、恢复、删除定时任务
- **任务监控**：实时查看任务执行状态、结果、日志
- **任务控制**：手动触发任务、终止任务、清理任务结果
- **Cron 表达式**：支持标准 Cron 和时区感知的 Cron
- **消息队列**：支持 RabbitMQ（生产）和 Redis（开发）作为 Broker

---

## 入口与启动

### Celery 应用入口

`backend/app/task/celery.py`：Celery 应用配置和初始化

```python
# Celery 应用实例
celery_app = Celery('fba_tasks')

# 配置项
celery_app.conf.update(
    broker_url=broker_url,              # 消息代理 URL
    result_backend=result_backend,      # 结果存储后端
    timezone=settings.DATETIME_TIMEZONE,
    enable_utc=False,
    task_serializer='msgpack',
    result_serializer='msgpack',
    accept_content=['msgpack', 'json'],
    task_track_started=True,
    task_time_limit=30 * 60,            # 任务超时：30分钟
)
```

### 启动 Celery 服务

```bash
# 启动 Worker（处理任务）
fba celery worker
# 或
celery -A backend.app.task.celery worker -l info -P gevent

# 启动 Beat（定时调度）
fba celery beat
# 或
celery -A backend.app.task.celery beat -l info

# 启动 Flower（监控界面）
fba celery flower
# 访问：http://localhost:8555
```

### API 路由入口

`backend/app/task/api/router.py`

```python
v1 = APIRouter(prefix=settings.FASTAPI_API_V1_PATH)

v1.include_router(scheduler_router)  # /api/v1/scheduler/*
v1.include_router(result_router)     # /api/v1/result/*
v1.include_router(control_router)    # /api/v1/control/*
```

---

## 对外接口

### 任务调度接口 (`/api/v1/scheduler`)

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/schedulers` | 分页查询任务调度列表 |
| GET | `/schedulers/all` | 获取所有任务调度（不分页） |
| GET | `/schedulers/{pk}` | 获取任务调度详情 |
| POST | `/schedulers` | 创建任务调度 |
| PUT | `/schedulers/{pk}` | 更新任务调度 |
| DELETE | `/schedulers` | 批量删除任务调度 |
| POST | `/schedulers/{pk}/start` | 启动任务调度 |
| POST | `/schedulers/{pk}/pause` | 暂停任务调度 |
| POST | `/schedulers/{pk}/resume` | 恢复任务调度 |

### 任务结果接口 (`/api/v1/result`)

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/results` | 分页查询任务结果列表 |
| GET | `/results/{pk}` | 获取任务结果详情 |
| DELETE | `/results` | 批量删除任务结果 |

### 任务控制接口 (`/api/v1/control`)

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/inspect/active` | 获取正在执行的任务 |
| GET | `/inspect/scheduled` | 获取计划执行的任务 |
| GET | `/inspect/reserved` | 获取保留的任务 |
| POST | `/control/shutdown` | 关闭 Worker |
| POST | `/control/pool_restart` | 重启 Worker 进程池 |
| POST | `/control/pool_grow` | 增加 Worker 进程 |
| POST | `/control/pool_shrink` | 减少 Worker 进程 |
| POST | `/control/autoscale` | 自动伸缩 Worker 进程池 |
| POST | `/control/{task_id}/revoke` | 撤销任务 |
| POST | `/control/{task_id}/terminate` | 终止任务 |

---

## 关键依赖与配置

### 核心依赖

```python
# Celery
from celery import Celery
from celery.schedules import crontab
from celery_aio_pool import Pool  # 异步池（Celery < 6.0）

# 定时任务
from backend.app.task.utils.tzcrontab import TzCrontab  # 时区感知 Crontab

# 数据库会话（任务专用）
from backend.app.task.database import get_task_db_session
```

### 环境配置项

`.env` 文件中的关键配置：

```env
# Celery Broker（消息队列）
CELERY_BROKER='redis'                      # 开发环境：redis，生产环境：rabbitmq

# Redis Broker 配置
CELERY_BROKER_REDIS_DATABASE=1
REDIS_HOST='127.0.0.1'
REDIS_PORT=6379
REDIS_PASSWORD=''

# RabbitMQ Broker 配置（生产环境）
CELERY_RABBITMQ_HOST='127.0.0.1'
CELERY_RABBITMQ_PORT=5672
CELERY_RABBITMQ_USERNAME='guest'
CELERY_RABBITMQ_PASSWORD='guest'
CELERY_RABBITMQ_VHOST=''

# Celery 基础配置
CELERY_REDIS_PREFIX='fba:celery'
CELERY_TASK_MAX_RETRIES=5                  # 任务最大重试次数
```

### Broker 切换逻辑

在 `backend/core/conf.py` 中：

```python
@model_validator(mode='before')
def check_env(cls, values):
    if values.get('ENVIRONMENT') == 'prod':
        # 生产环境自动切换到 RabbitMQ
        values['CELERY_BROKER'] = 'rabbitmq'
    return values
```

---

## 数据模型

### 1. 任务调度表 (`task_scheduler`)

```python
class Scheduler(Base):
    id: int                                # 主键
    name: str                              # 任务名称
    task: str                              # Celery 任务路径（如 backend.app.task.tasks.tasks.test_task）
    schedule_type: str                     # 调度类型（crontab/interval/date）
    crontab: str | None                    # Crontab 表达式
    interval_seconds: int | None           # 间隔秒数
    start_time: datetime | None            # 开始时间
    end_time: datetime | None              # 结束时间
    timezone: str                          # 时区（默认：Asia/Shanghai）
    args: list | None                      # 位置参数
    kwargs: dict | None                    # 关键字参数
    description: str | None                # 任务描述
    enabled: bool                          # 是否启用
    one_off: bool                          # 是否一次性任务
    last_run_at: datetime | None           # 最后运行时间
    total_run_count: int                   # 总运行次数
    max_run_count: int                     # 最大运行次数（0=无限制）
```

### 2. 任务结果表 (`task_result`)

```python
class Result(Base):
    id: int                                # 主键
    task_id: str                           # Celery 任务 ID（UUID）
    task_name: str                         # 任务名称
    status: str                            # 状态（PENDING/STARTED/SUCCESS/FAILURE/RETRY/REVOKED）
    result: dict | str | None              # 任务结果
    traceback: str | None                  # 异常堆栈（失败时）
    args: list | None                      # 位置参数
    kwargs: dict | None                    # 关键字参数
    started_at: datetime | None            # 开始时间
    completed_at: datetime | None          # 完成时间
    runtime: float | None                  # 运行时长（秒）
    worker: str | None                     # Worker 名称
```

---

## 任务定义

### 任务存放位置

```
backend/app/task/tasks/
├── base.py                 # 基础任务类
├── tasks.py                # 通用任务
├── beat.py                 # Beat 任务（自动同步调度器到 Celery Beat）
└── db_log/
    └── tasks.py            # 数据库日志任务
```

### 示例：创建定时任务

```python
# backend/app/task/tasks/tasks.py

from backend.app.task.celery import celery_app
from backend.app.task.tasks.base import BaseTask

@celery_app.task(base=BaseTask, bind=True)
def my_scheduled_task(self, param1: str, param2: int):
    """自定义定时任务"""
    # 任务逻辑
    result = f"Processed {param1} with {param2}"

    # 更新任务状态（可选）
    self.update_state(state='PROGRESS', meta={'current': 50, 'total': 100})

    return result
```

### 示例：通过 API 创建调度

```bash
POST /api/v1/scheduler
{
  "name": "每日报表任务",
  "task": "backend.app.task.tasks.tasks.my_scheduled_task",
  "schedule_type": "crontab",
  "crontab": "0 9 * * *",              # 每天早上 9 点
  "timezone": "Asia/Shanghai",
  "args": ["daily_report"],
  "kwargs": {"param2": 100},
  "description": "生成每日运营报表",
  "enabled": true
}
```

---

## 测试与质量

### 测试状态

- ❌ 当前模块**暂无单元测试**
- ⚠️ 需要补充测试覆盖

### 建议测试场景

1. **任务调度 CRUD**
   - 创建、查询、更新、删除任务调度
   - 启动、暂停、恢复任务

2. **任务执行**
   - 手动触发任务
   - 验证任务结果写入
   - 异常处理和重试机制

3. **Cron 表达式解析**
   - 标准 Cron 表达式
   - 时区感知 Cron

4. **任务控制**
   - 撤销/终止正在执行的任务
   - Worker 进程池管理

---

## 常见问题 (FAQ)

**Q: 如何切换到 RabbitMQ Broker？**
A: 在 `.env` 中设置 `CELERY_BROKER='rabbitmq'`，并配置 RabbitMQ 连接信息。生产环境会自动切换。

**Q: 如何创建时区感知的定时任务？**
A: 使用 `TzCrontab` 类代替标准 `crontab`：
```python
from backend.app.task.utils.tzcrontab import TzCrontab

schedule = TzCrontab(
    minute='0',
    hour='9',
    day_of_week='*',
    timezone='Asia/Shanghai'
)
```

**Q: 任务执行失败后如何重试？**
A: Celery 会自动重试，最大次数由 `CELERY_TASK_MAX_RETRIES` 配置。可以在任务中手动触发重试：
```python
@celery_app.task(bind=True, max_retries=3)
def my_task(self):
    try:
        # 任务逻辑
        pass
    except Exception as exc:
        # 5 分钟后重试
        raise self.retry(exc=exc, countdown=300)
```

**Q: 如何监控 Celery 任务执行情况？**
A:
1. **Flower**：启动 `fba celery flower`，访问 http://localhost:8555
2. **Grafana**：项目已集成 Celery Exporter + Prometheus + Grafana 监控面板
3. **API**：调用 `/api/v1/control/inspect/*` 接口查看任务状态

**Q: 任务结果保留多久？**
A: 默认永久保留。可以通过定时任务或手动调用 API 清理历史结果。

**Q: 如何实现任务链（Task Chain）？**
A: 使用 Celery 的 `chain` 功能：
```python
from celery import chain

result = chain(
    task1.s(arg1),
    task2.s(),
    task3.s(arg3)
).apply_async()
```

---

## 相关文件清单

### API 路由
- `api/router.py` - 路由聚合器
- `api/v1/scheduler.py` - 任务调度接口
- `api/v1/result.py` - 任务结果接口
- `api/v1/control.py` - 任务控制接口

### 数据模型
- `model/scheduler.py` - 任务调度模型
- `model/result.py` - 任务结果模型

### Schema（数据传输对象）
- `schema/scheduler.py` - 任务调度 Schema
- `schema/result.py` - 任务结果 Schema
- `schema/control.py` - 任务控制 Schema

### CRUD（数据访问层）
- `crud/crud_scheduler.py` - 任务调度数据访问
- `crud/crud_result.py` - 任务结果数据访问

### Service（业务逻辑层）
- `service/scheduler_service.py` - 任务调度服务
- `service/result_service.py` - 任务结果服务

### 任务定义
- `tasks/base.py` - 基础任务类（自动记录结果）
- `tasks/tasks.py` - 通用任务
- `tasks/beat.py` - Beat 同步任务
- `tasks/db_log/tasks.py` - 数据库日志任务

### 工具函数
- `utils/tzcrontab.py` - 时区感知 Crontab
- `utils/schedulers.py` - 调度器工具
- `session.py` - 任务会话管理
- `database.py` - 任务数据库配置
- `actions.py` - WebSocket 动作
- `enums.py` - 枚举定义
- `celery.py` - Celery 应用配置

### 配置文件
- `README.md` - 模块说明文档
