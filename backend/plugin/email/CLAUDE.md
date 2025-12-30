# Email 插件 - 邮件发送

[根目录](../../../CLAUDE.md) > [backend](../../) > [plugin](../) > **email**

> 最后更新：2025-12-30 11:02:40 +0800

## 插件职责

`email` 插件提供**邮件发送**功能，支持验证码、通知等场景。

**核心功能：**
- 邮件验证码发送
- 验证码校验
- SMTP 邮件发送

---

## API 路由

`/api/v1/email/*`

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/send/code` | 发送邮件验证码 |
| POST | `/verify/code` | 验证邮件验证码 |

---

## 环境配置

`.env` 文件配置：

```env
# 邮箱账号
EMAIL_USERNAME='your_email@qq.com'
EMAIL_PASSWORD='your_smtp_password'  # QQ 邮箱需使用授权码

# SMTP 服务器
EMAIL_HOST='smtp.qq.com'
EMAIL_PORT=465
EMAIL_SSL=True

# 验证码配置
EMAIL_CAPTCHA_REDIS_PREFIX='fba:email:captcha'
EMAIL_CAPTCHA_EXPIRE_SECONDS=180  # 3 分钟
```

---

## 使用示例

```python
# 发送验证码
POST /api/v1/email/send/code
{
  "email": "user@example.com"
}

# 验证验证码
POST /api/v1/email/verify/code
{
  "email": "user@example.com",
  "code": "123456"
}
```

---

## 相关文件

- `api/router.py` - 路由聚合
- `api/v1/email.py` - 邮件接口
- `utils/send.py` - 邮件发送工具
- `plugin.toml` - 插件配置
