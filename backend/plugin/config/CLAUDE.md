# Config 插件 - 系统配置

[根目录](../../../CLAUDE.md) > [backend](../../) > [plugin](../) > **config**

> 最后更新：2025-12-30 11:02:40 +0800

## 插件职责

`config` 插件提供**系统配置**功能，用于动态管理系统参数。

**核心功能：**
- 配置项管理（键值对）
- 配置分组
- 配置缓存（Redis）

---

## API 路由

`/api/v1/sys/config/*`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/configs` | 分页查询配置列表 |
| GET | `/configs/{pk}` | 获取配置详情 |
| POST | `/configs` | 创建配置 |
| PUT | `/configs/{pk}` | 更新配置 |
| DELETE | `/configs` | 批量删除配置 |

---

## 数据模型

### 系统配置表 (`sys_config`)

```python
class Config(Base):
    id: int
    name: str                              # 配置名称
    code: str                              # 配置编码（唯一）
    value: str                             # 配置值
    group: str | None                      # 配置分组
    type: int                              # 类型（1文本 2数字 3布尔 4JSON）
    remark: str | None                     # 备注
```

---

## 相关文件

- `api/v1/sys/config.py` - 配置接口
- `model/config.py` - 配置模型
- `schema/config.py` - Schema
- `crud/crud_config.py` - CRUD
- `service/config_service.py` - 配置服务
- `enums.py` - 枚举定义
- `plugin.toml` - 插件配置
