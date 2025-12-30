# OAuth2 插件 - 第三方登录

[根目录](../../../CLAUDE.md) > [backend](../../) > [plugin](../) > **oauth2**

> 最后更新：2025-12-30 11:02:40 +0800

## 插件职责

`oauth2` 插件提供**第三方登录**功能，支持主流 OAuth2 提供商。

**支持的登录方式：**
- **GitHub** 登录
- **Google** 登录
- **LinuxDo** 登录

**核心功能：**
- OAuth2 授权流程管理
- 用户社交账号绑定/解绑
- 社交账号信息同步
- State 参数防 CSRF 攻击

---

## API 路由

`/api/v1/oauth2/*`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/github/authorize` | GitHub 授权跳转 |
| GET | `/github/callback` | GitHub 授权回调 |
| GET | `/google/authorize` | Google 授权跳转 |
| GET | `/google/callback` | Google 授权回调 |
| GET | `/linux-do/authorize` | LinuxDo 授权跳转 |
| GET | `/linux-do/callback` | LinuxDo 授权回调 |
| GET | `/user_socials` | 查询用户社交账号列表 |
| GET | `/user_socials/{pk}` | 获取社交账号详情 |
| DELETE | `/user_socials/{pk}` | 解绑社交账号 |

---

## 环境配置

`.env` 文件配置：

```env
# GitHub OAuth2
OAUTH2_GITHUB_CLIENT_ID='your_client_id'
OAUTH2_GITHUB_CLIENT_SECRET='your_client_secret'
OAUTH2_GITHUB_REDIRECT_URI='http://127.0.0.1:8000/api/v1/oauth2/github/callback'

# Google OAuth2
OAUTH2_GOOGLE_CLIENT_ID='your_client_id'
OAUTH2_GOOGLE_CLIENT_SECRET='your_client_secret'
OAUTH2_GOOGLE_REDIRECT_URI='http://127.0.0.1:8000/api/v1/oauth2/google/callback'

# LinuxDo OAuth2
OAUTH2_LINUX_DO_CLIENT_ID='your_client_id'
OAUTH2_LINUX_DO_CLIENT_SECRET='your_client_secret'
OAUTH2_LINUX_DO_REDIRECT_URI='http://127.0.0.1:8000/api/v1/oauth2/linux-do/callback'

# 前端回调地址
OAUTH2_FRONTEND_LOGIN_REDIRECT_URI='http://localhost:5173/oauth2/callback'
OAUTH2_FRONTEND_BINDING_REDIRECT_URI='http://localhost:5173/profile'

# State 参数配置
OAUTH2_STATE_REDIS_PREFIX='fba:oauth2:state'
OAUTH2_STATE_EXPIRE_SECONDS=180  # 3 分钟
```

---

## 数据模型

### 用户社交账号表 (`sys_user_social`)

```python
class UserSocial(Base):
    id: int
    user_id: int                           # 关联用户 ID
    source: str                            # 来源（github/google/linux_do）
    open_id: str                           # 第三方唯一标识
    username: str | None                   # 第三方用户名
    nickname: str | None                   # 第三方昵称
    email: str | None                      # 第三方邮箱
    avatar: str | None                     # 第三方头像
    access_token: str | None               # 访问令牌
    refresh_token: str | None              # 刷新令牌
```

---

## 使用示例

### 前端接入流程

```javascript
// 1. 前端跳转到授权页面
window.location.href = 'http://127.0.0.1:8000/api/v1/oauth2/github/authorize';

// 2. 用户授权后，GitHub 回调到 /oauth2/github/callback
// 3. 后端处理完成后，重定向到前端回调地址，携带参数：
//    - access_token: JWT Token
//    - is_new: 是否新用户
//    http://localhost:5173/oauth2/callback?access_token=xxx&is_new=false

// 4. 前端接收参数并保存 Token
const params = new URLSearchParams(window.location.search);
const token = params.get('access_token');
const isNew = params.get('is_new') === 'true';
```

---

## 相关文件

- `api/router.py` - 路由聚合
- `api/v1/github.py` - GitHub 登录接口
- `api/v1/google.py` - Google 登录接口
- `api/v1/linux_do.py` - LinuxDo 登录接口
- `api/v1/user_social.py` - 社交账号管理接口
- `model/user_social.py` - 数据模型
- `schema/user_social.py` - Schema
- `crud/crud_user_social.py` - CRUD
- `service/oauth2_service.py` - OAuth2 核心服务
- `service/user_social_service.py` - 社交账号服务
- `enums.py` - 枚举定义
- `plugin.toml` - 插件配置
