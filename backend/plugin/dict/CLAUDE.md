# Dict 插件 - 数据字典

[根目录](../../../CLAUDE.md) > [backend](../../) > [plugin](../) > **dict**

> 最后更新：2025-12-30 11:02:40 +0800

## 插件职责

`dict` 插件提供**数据字典**功能，用于管理系统中的各类枚举数据。

**核心功能：**
- 字典类型管理（如：性别、状态）
- 字典数据管理（如：男、女）
- 字典缓存（Redis）

---

## API 路由

`/api/v1/sys/dict/*`

### 字典类型

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/types` | 分页查询字典类型 |
| GET | `/types/{pk}` | 获取字典类型详情 |
| POST | `/types` | 创建字典类型 |
| PUT | `/types/{pk}` | 更新字典类型 |
| DELETE | `/types` | 批量删除字典类型 |

### 字典数据

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/data` | 分页查询字典数据 |
| GET | `/data/{pk}` | 获取字典数据详情 |
| POST | `/data` | 创建字典数据 |
| PUT | `/data/{pk}` | 更新字典数据 |
| DELETE | `/data` | 批量删除字典数据 |

---

## 数据模型

### 字典类型表 (`sys_dict_type`)

```python
class DictType(Base):
    id: int
    name: str                              # 字典名称
    code: str                              # 字典编码（唯一）
    status: int                            # 状态
    remark: str | None                     # 备注
```

### 字典数据表 (`sys_dict_data`)

```python
class DictData(Base):
    id: int
    type_id: int                           # 字典类型 ID
    label: str                             # 字典标签
    value: str                             # 字典值
    sort: int                              # 排序
    status: int                            # 状态
    remark: str | None                     # 备注
```

---

## 相关文件

- `api/v1/sys/dict_type.py` - 字典类型接口
- `api/v1/sys/dict_data.py` - 字典数据接口
- `model/dict_type.py` - 字典类型模型
- `model/dict_data.py` - 字典数据模型
- `schema/dict_type.py` - 字典类型 Schema
- `schema/dict_data.py` - 字典数据 Schema
- `crud/crud_dict_type.py` - 字典类型 CRUD
- `crud/crud_dict_data.py` - 字典数据 CRUD
- `service/dict_type_service.py` - 字典类型服务
- `service/dict_data_service.py` - 字典数据服务
- `plugin.toml` - 插件配置
