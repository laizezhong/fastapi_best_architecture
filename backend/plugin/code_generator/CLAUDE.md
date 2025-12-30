# Code Generator 插件 - 代码生成器

[根目录](../../../CLAUDE.md) > [backend](../../) > [plugin](../) > **code_generator**

> 最后更新：2025-12-30 11:02:40 +0800

## 变更记录 (Changelog)

### 2025-12-30
- **初始化插件文档**：首次生成代码生成插件文档

---

## 插件职责

`code_generator` 是一个强大的**代码生成插件**，能够根据数据库表结构自动生成完整的 CRUD 代码，极大提升开发效率。

**核心功能：**
- **数据库表导入**：自动扫描数据库表，导入表结构和列信息
- **业务模型管理**：配置代码生成的业务规则和参数
- **代码生成**：生成 Model、Schema、CRUD、Service、API 五层代码
- **模板引擎**：基于 Jinja2 模板，支持自定义模板
- **类型映射**：自动转换数据库类型到 Python/SQLAlchemy 类型
- **代码预览**：生成前可预览代码
- **代码下载**：支持 ZIP 打包下载生成的代码

---

## 入口与启动

### CLI 命令

```bash
# 导入数据库表（创建业务和模型列）
fba codegen import --app admin --tn sys_user
# --app: 应用名称（生成代码的目标应用）
# --tn: 表名

# 交互式生成代码
fba codegen
# 会列出所有已导入的业务，选择一个进行代码生成
```

### API 路由

`/api/v1/gen/*`

---

## 对外接口

### 业务管理接口

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/businesses` | 分页查询业务列表 |
| GET | `/businesses/all` | 获取所有业务（不分页） |
| GET | `/businesses/{pk}` | 获取业务详情 |
| POST | `/businesses` | 创建业务（已弃用，推荐使用 import） |
| PUT | `/businesses/{pk}` | 更新业务 |
| DELETE | `/businesses` | 批量删除业务 |

### 模型列管理接口

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/columns` | 分页查询模型列 |
| GET | `/columns/{pk}` | 获取模型列详情 |
| POST | `/columns` | 创建模型列（已弃用） |
| PUT | `/columns/{pk}` | 更新模型列 |
| DELETE | `/columns` | 批量删除模型列 |

### 代码生成接口

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/tables` | 获取数据库表列表 |
| POST | `/import` | 导入数据库表（自动创建业务和模型列） |
| POST | `/preview` | 预览生成的代码 |
| POST | `/generate` | 生成代码到磁盘 |
| POST | `/download` | 下载生成的代码（ZIP） |

---

## 关键依赖与配置

### 核心依赖

```python
# Jinja2 模板引擎
from jinja2 import Environment, FileSystemLoader

# 类型转换
from backend.plugin.code_generator.utils.type_conversion import (
    mysql_type_to_python,
    postgresql_type_to_python,
    python_type_to_pydantic,
)

# 代码模板
from backend.plugin.code_generator.utils.code_template import (
    MODEL_TEMPLATE,
    SCHEMA_TEMPLATE,
    CRUD_TEMPLATE,
    SERVICE_TEMPLATE,
    API_TEMPLATE,
)
```

### 环境配置项

```env
# 代码生成下载文件名
CODE_GENERATOR_DOWNLOAD_ZIP_FILENAME='fba_generator'
```

---

## 数据模型

### 1. 代码生成业务表 (`gen_business`)

```python
class Business(Base):
    id: int                                # 主键
    app_name: str                          # 应用名称（如 admin）
    table_name: str                        # 数据库表名
    table_comment: str | None              # 表注释
    class_name: str                        # 类名（如 User）
    func_name: str                         # 函数名（如 user）
    module_name: str                       # 模块名（如 user）
    gen_path: str | None                   # 生成路径（可选，默认为应用根路径）
    author: str | None                     # 作者
    remark: str | None                     # 备注
```

### 2. 代码生成模型列表 (`gen_column`)

```python
class Column(Base):
    id: int                                # 主键
    business_id: int                       # 关联业务 ID
    column_name: str                       # 列名（数据库）
    column_type: str                       # 列类型（数据库）
    column_comment: str | None             # 列注释
    python_type: str                       # Python 类型
    pydantic_type: str                     # Pydantic 类型
    is_pk: bool                            # 是否主键
    is_required: bool                      # 是否必填
    is_query: bool                         # 是否查询字段
    is_insert: bool                        # 是否插入字段
    is_update: bool                        # 是否更新字段
    is_list: bool                          # 是否列表显示字段
    query_type: str | None                 # 查询类型（=、like、in、between）
    sort_order: int                        # 排序
```

---

## 使用流程

### 方式 1：CLI 命令（推荐）

```bash
# 1. 导入数据库表
fba codegen import --app admin --tn sys_user

# 2. 生成代码
fba codegen
# 选择业务编号，代码会生成到 backend/app/admin/ 目录

# 3. 查看生成的代码
# backend/app/admin/
# ├── model/user.py
# ├── schema/user.py
# ├── crud/crud_user.py
# ├── service/user_service.py
# └── api/v1/user.py
```

### 方式 2：API 调用

```bash
# 1. 获取数据库表列表
GET /api/v1/gen/tables

# 2. 导入表
POST /api/v1/gen/import
{
  "app": "admin",
  "table_schema": "fba",
  "table_name": "sys_user"
}

# 3. 预览代码
POST /api/v1/gen/preview
{
  "business_id": 1
}

# 4. 生成到磁盘
POST /api/v1/gen/generate
{
  "business_id": 1
}

# 5. 下载 ZIP
POST /api/v1/gen/download
{
  "business_id": 1
}
```

---

## 代码模板

### 生成的文件结构

```
backend/app/{app}/
├── model/{module}.py              # 数据模型
├── schema/{module}.py             # Pydantic Schema
├── crud/crud_{module}.py          # CRUD 操作
├── service/{module}_service.py    # 业务逻辑
└── api/v1/{module}.py             # API 路由
```

### Model 模板示例

```python
# backend/app/admin/model/user.py
from backend.common.model import Base, id_key
from sqlalchemy.orm import Mapped, mapped_column
import sqlalchemy as sa

class User(Base):
    """用户表"""
    __tablename__ = 'sys_user'

    id: Mapped[id_key] = mapped_column(init=False)
    username: Mapped[str] = mapped_column(sa.String(64), comment='用户名')
    nickname: Mapped[str] = mapped_column(sa.String(64), comment='昵称')
    # ... 其他字段
```

### Schema 模板示例

```python
# backend/app/admin/schema/user.py
from pydantic import BaseModel, Field

class UserBase(BaseModel):
    username: str = Field(..., max_length=64, description='用户名')
    nickname: str = Field(..., max_length=64, description='昵称')

class UserCreate(UserBase):
    pass

class UserUpdate(UserBase):
    username: str | None = None
    nickname: str | None = None

class UserSchema(UserBase):
    id: int
    created_time: datetime
    updated_time: datetime | None

    model_config = ConfigDict(from_attributes=True)
```

### CRUD 模板示例

```python
# backend/app/admin/crud/crud_user.py
from sqlalchemy_crud_plus import CRUDPlus
from backend.app.admin.model.user import User

class CRUDUser(CRUDPlus[User]):
    pass

crud_user = CRUDUser(User)
```

### Service 模板示例

```python
# backend/app/admin/service/user_service.py
from backend.database.db import CurrentSession
from backend.app.admin.crud.crud_user import crud_user
from backend.app.admin.schema.user import UserCreate, UserUpdate

class UserService:
    @staticmethod
    async def get(db: CurrentSession, pk: int) -> User:
        return await crud_user.select_model(db, pk)

    @staticmethod
    async def create(db: CurrentSession, obj: UserCreate) -> None:
        await crud_user.create_model(db, obj)

    @staticmethod
    async def update(db: CurrentSession, pk: int, obj: UserUpdate) -> int:
        return await crud_user.update_model(db, pk, obj)

    @staticmethod
    async def delete(db: CurrentSession, pk: int) -> int:
        return await crud_user.delete_model(db, pk)

user_service = UserService()
```

### API 模板示例

```python
# backend/app/admin/api/v1/user.py
from fastapi import APIRouter
from fastapi_pagination import Page, Params

from backend.database.db import CurrentSession
from backend.app.admin.service.user_service import user_service
from backend.app.admin.schema.user import UserCreate, UserUpdate, UserSchema
from backend.common.response.response_schema import response_base

router = APIRouter()

@router.get('', summary='分页查询用户')
async def get_users(db: CurrentSession, params: Params) -> Page[UserSchema]:
    data = await user_service.get_list(db, params)
    return response_base.success(data=data)

@router.get('/{pk}', summary='获取用户详情')
async def get_user(pk: int, db: CurrentSession):
    data = await user_service.get(db, pk)
    return response_base.success(data=data)

@router.post('', summary='创建用户')
async def create_user(obj: UserCreate, db: CurrentSession):
    await user_service.create(db, obj)
    return response_base.success()

@router.put('/{pk}', summary='更新用户')
async def update_user(pk: int, obj: UserUpdate, db: CurrentSession):
    count = await user_service.update(db, pk, obj)
    return response_base.success(data=count)

@router.delete('/{pk}', summary='删除用户')
async def delete_user(pk: int, db: CurrentSession):
    count = await user_service.delete(db, pk)
    return response_base.success(data=count)
```

---

## 类型映射

### MySQL 类型映射

| MySQL 类型 | Python 类型 | Pydantic 类型 |
|-----------|------------|--------------|
| INT, BIGINT | int | int |
| VARCHAR, TEXT | str | str |
| DATETIME, TIMESTAMP | datetime | datetime |
| DECIMAL, FLOAT, DOUBLE | float | float |
| TINYINT(1) | bool | bool |
| JSON | dict | dict |

### PostgreSQL 类型映射

| PostgreSQL 类型 | Python 类型 | Pydantic 类型 |
|----------------|------------|--------------|
| INTEGER, BIGINT | int | int |
| VARCHAR, TEXT | str | str |
| TIMESTAMP | datetime | datetime |
| NUMERIC, REAL, DOUBLE PRECISION | float | float |
| BOOLEAN | bool | bool |
| JSONB | dict | dict |

---

## 常见问题 (FAQ)

**Q: 生成的代码保存在哪里？**
A: 默认保存在 `backend/app/{app_name}/` 目录下，可以在业务配置中通过 `gen_path` 自定义路径。

**Q: 如何自定义代码模板？**
A: 修改 `backend/plugin/code_generator/utils/code_template.py` 中的模板字符串。

**Q: 支持哪些数据库？**
A: 当前支持 PostgreSQL 和 MySQL。

**Q: 生成的代码会覆盖现有文件吗？**
A: **会覆盖**！生成前请确认，或者先下载 ZIP 查看代码。

**Q: 如何修改已导入的业务配置？**
A: 调用 `PUT /api/v1/gen/businesses/{pk}` 接口更新业务配置。

**Q: 可以只生成部分文件吗（如只生成 Model）？**
A: 当前不支持，会生成完整的五层代码。可以手动删除不需要的文件。

---

## 相关文件清单

### API 路由
- `api/router.py` - 路由聚合器
- `api/v1/business.py` - 业务管理接口
- `api/v1/column.py` - 模型列管理接口
- `api/v1/gen.py` - 代码生成核心接口

### 数据模型
- `model/business.py` - 业务模型
- `model/column.py` - 模型列

### Schema
- `schema/business.py` - 业务 Schema
- `schema/column.py` - 模型列 Schema
- `schema/code.py` - 代码生成 Schema

### CRUD
- `crud/crud_business.py` - 业务数据访问
- `crud/crud_column.py` - 模型列数据访问
- `crud/crud_code.py` - 代码生成数据访问

### Service
- `service/business_service.py` - 业务服务
- `service/column_service.py` - 模型列服务
- `service/code_service.py` - 代码生成服务

### 工具函数
- `utils/type_conversion.py` - 类型转换工具
- `utils/code_template.py` - 代码模板
- `path_conf.py` - 路径配置
- `enums.py` - 枚举定义

### 配置文件
- `plugin.toml` - 插件配置
- `README.md` - 插件说明文档
