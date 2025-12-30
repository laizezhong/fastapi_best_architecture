# Admin 模块 - 管理后台

[根目录](../../CLAUDE.md) > [backend](../) > **app/admin**

> 最后更新：2025-12-30 11:02:40 +0800

## 变更记录 (Changelog)

### 2025-12-30
- **初始化模块文档**：首次生成 admin 模块架构文档

---

## 模块职责

`admin` 模块是 FastAPI Best Architecture 的**核心管理后台模块**，提供企业级应用必需的用户管理、权限控制、系统监控等功能。

**核心功能：**
- **用户管理**：用户 CRUD、密码安全策略、账号锁定
- **角色权限**：RBAC 权限控制、角色-菜单关联
- **菜单管理**：动态菜单树、路由权限配置
- **部门管理**：组织架构树形管理
- **数据权限**：行级数据权限控制（数据范围 + 数据规则）
- **日志审计**：登录日志、操作日志
- **系统监控**：服务器状态、Redis 监控、在线用户
- **文件管理**：文件上传、头像管理

---

## 入口与启动

### API 路由入口

`backend/app/admin/api/router.py`：聚合所有 admin 子路由

```python
v1 = APIRouter(prefix=settings.FASTAPI_API_V1_PATH)

v1.include_router(auth_router)      # 认证登录
v1.include_router(sys_router)       # 系统管理
v1.include_router(log_router)       # 日志管理
v1.include_router(monitor_router)   # 系统监控
```

### 路由模块化

- **auth**：`/api/v1/auth/*` - 登录、登出、刷新 Token、验证码
- **sys**：`/api/v1/sys/*` - 用户、角色、菜单、部门、数据权限、文件上传
- **log**：`/api/v1/log/*` - 登录日志、操作日志
- **monitor**：`/api/v1/monitors/*` - 服务器监控、Redis 监控、在线用户

---

## 对外接口

### 认证接口 (`/api/v1/auth`)

| 方法 | 路径 | 说明 | 认证 |
|------|------|------|------|
| POST | `/login` | 用户登录（返回 access_token + refresh_token） | ❌ |
| POST | `/login/swagger` | Swagger 登录（OAuth2 密码流） | ❌ |
| POST | `/logout` | 用户登出（清除 Token） | ✅ |
| POST | `/refresh` | 刷新访问令牌 | ❌ |
| GET | `/captcha` | 获取图形验证码 | ❌ |

### 系统管理接口 (`/api/v1/sys`)

#### 用户管理

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/users` | 分页查询用户列表 |
| GET | `/users/{pk}` | 获取用户详情 |
| POST | `/users` | 创建用户 |
| PUT | `/users/{pk}` | 更新用户信息 |
| DELETE | `/users/{pk}` | 删除用户 |
| PUT | `/users/{pk}/password/reset` | 重置用户密码 |
| PUT | `/users/{pk}/roles` | 设置用户角色 |
| GET | `/users/current` | 获取当前登录用户信息 |
| PUT | `/users/current/password` | 修改当前用户密码 |
| PUT | `/users/current/avatar` | 更新当前用户头像 |

#### 角色管理

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/roles` | 分页查询角色列表 |
| GET | `/roles/{pk}` | 获取角色详情 |
| POST | `/roles` | 创建角色 |
| PUT | `/roles/{pk}` | 更新角色 |
| DELETE | `/roles/{pk}` | 删除角色 |
| PUT | `/roles/{pk}/menus` | 设置角色菜单权限 |

#### 菜单管理

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/menus` | 获取菜单树 |
| GET | `/menus/{pk}` | 获取菜单详情 |
| POST | `/menus` | 创建菜单 |
| PUT | `/menus/{pk}` | 更新菜单 |
| DELETE | `/menus/{pk}` | 删除菜单 |

#### 部门管理

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/depts` | 获取部门树 |
| GET | `/depts/{pk}` | 获取部门详情 |
| POST | `/depts` | 创建部门 |
| PUT | `/depts/{pk}` | 更新部门 |
| DELETE | `/depts/{pk}` | 删除部门 |

#### 数据权限

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/data_scopes` | 分页查询数据范围 |
| POST | `/data_scopes` | 创建数据范围 |
| PUT | `/data_scopes/{pk}` | 更新数据范围 |
| DELETE | `/data_scopes/{pk}` | 删除数据范围 |
| GET | `/data_rules` | 分页查询数据规则 |
| POST | `/data_rules` | 创建数据规则 |
| PUT | `/data_rules/{pk}` | 更新数据规则 |
| DELETE | `/data_rules/{pk}` | 删除数据规则 |

#### 文件管理

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/files/image` | 上传图片 |
| POST | `/files/video` | 上传视频 |

### 日志管理接口 (`/api/v1/log`)

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/login_logs` | 分页查询登录日志 |
| DELETE | `/login_logs` | 批量删除登录日志 |
| GET | `/opera_logs` | 分页查询操作日志 |
| DELETE | `/opera_logs` | 批量删除操作日志 |

### 监控接口 (`/api/v1/monitors`)

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/server` | 获取服务器状态（CPU、内存、磁盘） |
| GET | `/redis` | 获取 Redis 状态 |
| GET | `/online` | 获取在线用户列表 |
| DELETE | `/online/{pk}` | 强制下线用户 |

---

## 关键依赖与配置

### 核心依赖

```python
# 认证与安全
from backend.common.security.jwt import create_access_token, verify_token
from backend.common.security.rbac import DependsRBAC
from backend.common.security.permission import RequestPermission

# 数据库
from backend.database.db import CurrentSession, CurrentSessionTransaction

# 分页
from fastapi_pagination import Page, Params

# 限流
from fastapi_limiter.depends import RateLimiter

# 验证码
from fast_captcha import img_captcha
```

### 环境配置项

`.env` 文件中的关键配置：

```env
# Token 配置
TOKEN_SECRET_KEY='...'                    # JWT 密钥
TOKEN_ALGORITHM='HS256'                   # 加密算法
TOKEN_EXPIRE_SECONDS=86400                # 访问令牌过期时间（1天）
TOKEN_REFRESH_EXPIRE_SECONDS=604800       # 刷新令牌过期时间（7天）

# 用户安全
USER_LOCK_THRESHOLD=5                     # 密码错误锁定阈值
USER_LOCK_SECONDS=300                     # 锁定时长（5分钟）
USER_PASSWORD_EXPIRY_DAYS=365             # 密码有效期（天）
USER_PASSWORD_MIN_LENGTH=6                # 密码最小长度
USER_PASSWORD_REQUIRE_SPECIAL_CHAR=False  # 是否要求特殊字符

# 登录验证码
LOGIN_CAPTCHA_ENABLED=True                # 是否启用验证码
LOGIN_CAPTCHA_EXPIRE_SECONDS=300          # 验证码过期时间（5分钟）

# RBAC 权限
RBAC_ROLE_MENU_MODE=True                  # 启用角色-菜单权限模式
```

---

## 数据模型

### 核心表结构

#### 1. 用户表 (`sys_user`)

```python
class User(Base):
    id: int                                # 主键
    uuid: str                              # UUID（唯一标识）
    username: str                          # 用户名（唯一、索引）
    nickname: str                          # 昵称
    password: str | None                   # 密码（加密存储）
    salt: bytes | None                     # 加密盐
    email: str | None                      # 邮箱（唯一、索引）
    phone: str | None                      # 手机号
    avatar: str | None                     # 头像 URL
    status: int                            # 状态（0停用 1正常）
    is_superuser: bool                     # 超级管理员
    is_staff: bool                         # 后台管理权限
    is_multi_login: bool                   # 允许多设备登录
    dept_id: int | None                    # 部门 ID
    join_time: datetime                    # 注册时间
    last_login_time: datetime | None       # 最后登录时间
    last_password_changed_time: datetime | None  # 密码最后修改时间
```

#### 2. 角色表 (`sys_role`)

```python
class Role(Base):
    id: int
    name: str                              # 角色名称
    data_scope: int                        # 数据范围（1全部 2自定义 3本部门 4本部门及以下）
    status: int                            # 状态
    remark: str | None                     # 备注
```

#### 3. 菜单表 (`sys_menu`)

```python
class Menu(Base):
    id: int
    parent_id: int | None                  # 父菜单 ID
    title: str                             # 菜单标题
    name: str                              # 菜单名称（唯一）
    level: int                             # 菜单层级
    type: int                              # 类型（0目录 1菜单 2按钮）
    icon: str | None                       # 图标
    path: str | None                       # 路由路径
    component: str | None                  # 组件路径
    perms: str | None                      # 权限标识
    method: str | None                     # 请求方法
    status: int                            # 状态
    show: int                              # 是否显示
    cache: int                             # 是否缓存
    remark: str | None                     # 备注
```

#### 4. 部门表 (`sys_dept`)

```python
class Dept(Base):
    id: int
    name: str                              # 部门名称
    level: int                             # 部门层级
    sort: int                              # 排序
    leader: str | None                     # 负责人
    phone: str | None                      # 联系电话
    email: str | None                      # 邮箱
    status: int                            # 状态
    del_flag: bool                         # 删除标记
    parent_id: int | None                  # 父部门 ID
```

#### 5. 数据权限表

**数据范围表** (`sys_data_scope`)：定义数据权限范围（全部数据、自定义、本部门等）

**数据规则表** (`sys_data_rule`)：定义具体的数据过滤规则（如：部门 ID = 当前用户部门）

#### 6. 日志表

**登录日志表** (`sys_login_log`)：记录用户登录/登出行为

```python
class LoginLog(Base):
    id: int
    user_uuid: str                         # 用户 UUID
    username: str                          # 用户名
    status: int                            # 登录状态（0失败 1成功）
    ip: str                                # IP 地址
    country: str | None                    # 国家
    region: str | None                     # 地区
    city: str | None                       # 城市
    user_agent: str                        # User Agent
    browser: str | None                    # 浏览器
    os: str | None                         # 操作系统
    device: str | None                     # 设备类型
    msg: str                               # 登录消息
    login_time: datetime                   # 登录时间
```

**操作日志表** (`sys_opera_log`)：记录用户操作行为（异步写入）

```python
class OperaLog(Base):
    id: int
    user_uuid: str                         # 用户 UUID
    username: str                          # 用户名
    method: str                            # 请求方法
    title: str                             # 操作标题
    path: str                              # 请求路径
    ip: str                                # IP 地址
    country: str | None                    # 国家
    region: str | None                     # 地区
    city: str | None                       # 城市
    user_agent: str                        # User Agent
    browser: str | None                    # 浏览器
    os: str | None                         # 操作系统
    device: str | None                     # 设备类型
    args: dict | None                      # 请求参数（加密）
    status: int                            # 状态（0失败 1成功）
    code: str                              # 响应码
    msg: str                               # 操作消息
    cost_time: float                       # 耗时（秒）
    opera_time: datetime                   # 操作时间
```

### 多对多关系表

- **用户-角色关联表** (`sys_user_role`)
- **角色-菜单关联表** (`sys_role_menu`)

---

## 测试与质量

### 测试配置

位置：`backend/app/admin/tests/`

```
tests/
├── conftest.py              # Pytest 配置和 Fixtures
├── utils/
│   └── db.py                # 测试数据库覆盖
└── api_v1/
    └── test_auth.py         # 认证接口测试
```

### 测试覆盖

当前已覆盖：
- ✅ 认证接口（登录、登出、Token 刷新）

待补充测试：
- ❌ 用户管理 CRUD
- ❌ 角色权限管理
- ❌ 菜单管理
- ❌ 数据权限

### 运行测试

```bash
# 运行 admin 模块所有测试
pytest backend/app/admin/tests/

# 运行认证测试
pytest backend/app/admin/tests/api_v1/test_auth.py -v

# 生成覆盖率报告
pytest backend/app/admin/tests/ --cov=backend.app.admin --cov-report=html
```

---

## 常见问题 (FAQ)

**Q: 如何自定义密码强度规则？**
A: 修改 `.env` 中的以下配置：
```env
USER_PASSWORD_MIN_LENGTH=8
USER_PASSWORD_MAX_LENGTH=32
USER_PASSWORD_REQUIRE_SPECIAL_CHAR=True
```

**Q: 如何禁用登录验证码？**
A: 在 `.env` 中设置 `LOGIN_CAPTCHA_ENABLED=False`。

**Q: 如何实现行级数据权限控制？**
A: 使用 `DataScope` 和 `DataRule` 模型：
1. 创建数据范围（如"仅本部门"）
2. 创建数据规则（如"dept_id = 当前用户部门ID"）
3. 将数据范围关联到角色
4. 在 Service 层自动应用权限过滤

**Q: 操作日志为什么是异步写入？**
A: 操作日志通过 `OperaLogMiddleware` 拦截所有请求，写入内存队列，由后台任务异步批量写入数据库，避免阻塞主请求流程。

**Q: 如何强制用户下线？**
A: 调用 `DELETE /api/v1/monitors/online/{user_id}` 接口，会清除用户的 Redis Token 缓存。

---

## 相关文件清单

### API 路由
- `api/router.py` - 路由聚合器
- `api/v1/auth/auth.py` - 认证接口
- `api/v1/auth/captcha.py` - 验证码接口
- `api/v1/sys/user.py` - 用户管理接口
- `api/v1/sys/role.py` - 角色管理接口
- `api/v1/sys/menu.py` - 菜单管理接口
- `api/v1/sys/dept.py` - 部门管理接口
- `api/v1/sys/data_scope.py` - 数据范围接口
- `api/v1/sys/data_rule.py` - 数据规则接口
- `api/v1/sys/file.py` - 文件上传接口
- `api/v1/log/login_log.py` - 登录日志接口
- `api/v1/log/opera_log.py` - 操作日志接口
- `api/v1/monitor/server.py` - 服务器监控接口
- `api/v1/monitor/redis.py` - Redis 监控接口
- `api/v1/monitor/online.py` - 在线用户接口

### 数据模型
- `model/user.py` - 用户模型
- `model/role.py` - 角色模型
- `model/menu.py` - 菜单模型
- `model/dept.py` - 部门模型
- `model/data_scope.py` - 数据范围模型
- `model/data_rule.py` - 数据规则模型
- `model/login_log.py` - 登录日志模型
- `model/opera_log.py` - 操作日志模型
- `model/user_password_history.py` - 密码历史模型
- `model/m2m.py` - 多对多关联表

### Schema（数据传输对象）
- `schema/user.py` - 用户 Schema
- `schema/role.py` - 角色 Schema
- `schema/menu.py` - 菜单 Schema
- `schema/dept.py` - 部门 Schema
- `schema/data_scope.py` - 数据范围 Schema
- `schema/data_rule.py` - 数据规则 Schema
- `schema/token.py` - Token Schema
- `schema/captcha.py` - 验证码 Schema
- `schema/login_log.py` - 登录日志 Schema
- `schema/opera_log.py` - 操作日志 Schema

### CRUD（数据访问层）
- `crud/crud_user.py` - 用户数据访问
- `crud/crud_role.py` - 角色数据访问
- `crud/crud_menu.py` - 菜单数据访问
- `crud/crud_dept.py` - 部门数据访问
- `crud/crud_data_scope.py` - 数据范围数据访问
- `crud/crud_data_rule.py` - 数据规则数据访问
- `crud/crud_login_log.py` - 登录日志数据访问
- `crud/crud_opera_log.py` - 操作日志数据访问
- `crud/crud_user_password_history.py` - 密码历史数据访问

### Service（业务逻辑层）
- `service/auth_service.py` - 认证服务
- `service/user_service.py` - 用户服务
- `service/role_service.py` - 角色服务
- `service/menu_service.py` - 菜单服务
- `service/dept_service.py` - 部门服务
- `service/data_scope_service.py` - 数据范围服务
- `service/data_rule_service.py` - 数据规则服务
- `service/login_log_service.py` - 登录日志服务
- `service/opera_log_service.py` - 操作日志服务
- `service/plugin_service.py` - 插件服务
- `service/user_password_history_service.py` - 密码历史服务

### 工具函数
- `utils/password_security.py` - 密码安全工具
- `utils/cache.py` - 缓存工具

### 测试
- `tests/conftest.py` - 测试配置
- `tests/api_v1/test_auth.py` - 认证接口测试
- `tests/utils/db.py` - 测试数据库工具
