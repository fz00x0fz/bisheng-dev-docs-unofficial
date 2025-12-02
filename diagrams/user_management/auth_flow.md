# 用户认证流程图

展示用户登录认证的完整流程。

## 代码入口

| 类/函数 | 文件路径 | 说明 |
|---------|----------|------|
| `UserPayload` | `src/backend/bisheng/api/services/user_service.py:30` | 用户负载类 |
| `get_login_user()` | `src/backend/bisheng/api/services/user_service.py` | 获取登录用户 |
| `UserDao` | `src/backend/bisheng/database/models/user.py` | 用户数据访问 |
| `UserRoleDao` | `src/backend/bisheng/database/models/user_role.py` | 用户角色数据 |
| `AuthJWT` | `fastapi_jwt_auth` | JWT认证库 |
| `user.py` | `src/backend/bisheng/api/v1/user.py` | 用户API路由 |
| `JWT.py` | `src/backend/bisheng/api/JWT.py` | JWT配置 |

```mermaid
flowchart TD
    A[用户发起登录请求] --> B{验证码校验}
    B -->|失败| C[返回验证码错误]
    B -->|成功| D{用户名存在?}
    D -->|否| E[返回用户不存在]
    D -->|是| F{密码正确?}
    F -->|否| G[增加错误次数]
    G --> H{超过限制?}
    H -->|是| I[锁定账户]
    H -->|否| J[返回密码错误]
    F -->|是| K{账户状态正常?}
    K -->|否| L[返回账户被禁用]
    K -->|是| M{密码过期?}
    M -->|是| N[返回密码过期提示]
    M -->|否| O[生成JWT Token]
    O --> P[设置Redis会话]
    P --> Q[记录审计日志]
    Q --> R[返回登录成功]
```

## 流程说明

### 认证步骤

1. **验证码校验** - 防止自动化攻击
2. **用户名验证** - 检查用户是否存在
3. **密码验证** - 校验密码正确性
4. **错误次数控制** - 防止暴力破解
5. **账户状态检查** - 确认账户未被禁用
6. **密码过期检查** - 强制密码更新策略
7. **Token生成** - 生成JWT访问令牌
8. **会话管理** - Redis存储会话信息
9. **审计日志** - 记录登录行为

### 安全配置

```yaml
password_conf:
  login_error_time_window: 30    # 错误统计时间窗口（分钟）
  max_error_times: 5             # 最大错误次数
  password_valid_period: 90      # 密码有效期（天）
```
