# Notice 插件 - 系统通知

[根目录](../../../CLAUDE.md) > [backend](../../) > [plugin](../) > **notice**

> 最后更新：2025-12-30 11:02:40 +0800

## 插件职责

`notice` 插件提供**系统通知**功能，用于发布公告和消息。

**核心功能：**
- 系统公告管理
- 通知类型（公告、消息）
- 发布状态管理

---

## API 路由

`/api/v1/sys/notice/*`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/notices` | 分页查询通知列表 |
| GET | `/notices/{pk}` | 获取通知详情 |
| POST | `/notices` | 创建通知 |
| PUT | `/notices/{pk}` | 更新通知 |
| DELETE | `/notices` | 批量删除通知 |

---

## 数据模型

### 系统通知表 (`sys_notice`)

```python
class Notice(Base):
    id: int
    title: str                             # 标题
    content: str                           # 内容
    type: int                              # 类型（1公告 2消息）
    status: int                            # 状态（0草稿 1已发布）
    publish_time: datetime | None          # 发布时间
```

---

## 相关文件

- `api/v1/sys/notice.py` - 通知接口
- `model/notice.py` - 通知模型
- `schema/notice.py` - Schema
- `crud/crud_notice.py` - CRUD
- `service/notice_service.py` - 通知服务
- `enums.py` - 枚举定义
- `plugin.toml` - 插件配置
