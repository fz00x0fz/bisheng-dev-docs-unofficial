# 用户管理模块详细设计文档

## 模块概述

用户管理模块是 Bisheng 平台的基础模块，负责用户的身份认证、权限管理、角色分配等功能。该模块基于 JWT (JSON Web Token) 实现身份认证，采用 RBAC (Role-Based Access Control) 模型进行权限控制。

## 模块结构

```
bisheng/
├── api/v1/user.py              # 用户相关API接口
├── api/v1/group.py             # 用户组API接口
├── database/models/
│   ├── user.py                 # 用户数据模型
│   ├── role.py                 # 角色数据模型
│   ├── user_role.py            # 用户角色关联模型
│   ├── group.py                # 用户组数据模型
│   └── user_group.py           # 用户组关联模型
├── api/services/user_service.py # 用户业务逻辑服务
└── common/dependencies/user_deps.py # 用户依赖注入

fastapi_jwt_auth/               # JWT认证库（内置）
├── auth_jwt.py                 # JWT认证核心
└── auth_config.py              # JWT配置
```

## 核心功能

### 1. 用户注册

#### 功能描述
允许新用户注册账户，创建用户基本信息。

#### 接口定义
- **URL**: `/user/regist`
- **Method**: POST
- **参数**:
  ```json
  {
    "user_name": "用户名",
    "password": "密码",
    "captcha": "验证码",
    "captcha_key": "验证码key"
  }
  ```

#### 实现逻辑
1. 验证码校验（如启用）
2. 检查用户名是否已存在
3. 密码加密处理
4. 判断是否为首个用户（自动设为管理员）
5. 创建用户并分配默认角色
6. 添加到默认用户组

#### 安全措施
- 用户名长度限制（最长30字符）
- 密码加密存储（MD5加密）
- 验证码防机器人注册

### 2. 用户登录

#### 功能描述
验证用户身份，生成访问令牌。

#### 接口定义
- **URL**: `/user/login`
- **Method**: POST
- **参数**:
  ```json
  {
    "user_name": "用户名",
    "password": "密码",
    "captcha": "验证码",
    "captcha_key": "验证码key"
  }
  ```

#### 实现逻辑
1. 验证码校验（如启用）
2. 用户名存在性检查
3. 密码验证
4. 账户状态检查（是否被禁用）
5. 密码过期检查
6. 生成 JWT token
7. 设置 Redis 会话信息
8. 记录登录审计日志

#### 安全措施
- 密码错误次数限制
- 账户自动锁定机制
- 密码过期策略
- JWT token 安全生成

### 3. 用户信息管理

#### 功能描述
获取和更新用户个人信息。

#### 接口定义
- **获取用户信息**: `/user/info` (GET)
- **更新用户信息**: `/user/update` (POST)

#### 实现逻辑
1. 身份认证检查
2. 权限验证
3. 数据查询/更新
4. 返回标准化结果

### 4. 角色管理

#### 功能描述
管理系统中的角色定义和分配。

#### 核心功能
- 角色创建
- 角色更新
- 角色删除
- 角色列表查询
- 用户角色分配

#### 数据模型
```python
class Role(BaseSQLModel):
    id: Optional[int] = Field(default=None, primary_key=True)
    group_id: int = Field(index=True)
    role_name: str = Field(max_length=50)
    remark: Optional[str] = Field(default=None, max_length=200)
```

### 5. 用户组管理

#### 功能描述
管理用户分组，便于批量权限管理。

#### 核心功能
- 用户组创建
- 用户组成员管理
- 用户组权限分配

## 权限控制系统

### RBAC 模型设计

Bisheng 采用基于角色的访问控制（RBAC）模型：

1. **用户(User)** - 系统的实际使用者
2. **角色(Role)** - 权限的集合
3. **权限(Permission)** - 对资源的操作权利
4. **用户组(Group)** - 用户的逻辑分组

### 权限验证流程

1. 用户登录获取 JWT token
2. 请求携带 token 访问受保护资源
3. 系统解析 token 获取用户信息
4. 根据用户ID查询所属角色
5. 根据角色查询拥有的权限
6. 验证请求是否具有相应权限

### 权限类型定义

```python
class AccessType(Enum):
    FLOW_READ = 1      # 流程读取权限
    FLOW_WRITE = 2     # 流程写入权限
    ASSISTANT_READ = 3 # 助手读取权限
    ASSISTANT_WRITE = 4 # 助手写入权限
    WORKFLOW_READ = 5  # 工作流读取权限
    WORKFLOW_WRITE = 6 # 工作流写入权限
```

## 安全机制

### 1. 密码安全

#### 加密策略
- 使用 MD5 进行密码加密
- 密码传输过程加密

#### 安全策略
- 密码错误次数限制（默认5次）
- 超过限制自动锁定账户
- 密码过期策略（可配置）
- 密码强度要求

### 2. JWT Token 安全

#### Token 结构
- 访问 Token (Access Token)
- 刷新 Token (Refresh Token)

#### 安全特性
- Token 过期时间控制
- Token 黑名单机制
- Token 加密签名

### 3. 验证码机制

#### 功能
- 防止自动化攻击
- 防止暴力破解

#### 实现
- 图形验证码生成
- Redis 存储验证码
- 验证码时效控制（5分钟）

## 数据模型设计

### 用户表 (User)

| 字段名 | 类型 | 描述 |
|--------|------|------|
| user_id | int | 用户ID（主键） |
| user_name | varchar | 用户名 |
| password | varchar | 密码（加密存储） |
| nick_name | varchar | 昵称 |
| delete | tinyint | 删除状态（0正常，1禁用） |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |

### 角色表 (Role)

| 字段名 | 类型 | 描述 |
|--------|------|------|
| id | int | 角色ID（主键） |
| group_id | int | 所属用户组ID |
| role_name | varchar | 角色名称 |
| remark | varchar | 角色备注 |

### 用户角色关联表 (UserRole)

| 字段名 | 类型 | 描述 |
|--------|------|------|
| id | int | 主键 |
| user_id | int | 用户ID |
| role_id | int | 角色ID |

## 业务逻辑实现

### UserService 类

```python
class UserService:
    @staticmethod
    def decrypt_md5_password(password: str) -> str:
        """解密并MD5加密密码"""
        pass

    @staticmethod
    def create_user(request, admin_user, req) -> User:
        """创建用户"""
        pass
```

### 核心服务方法

1. **用户认证**: `authenticate_user()`
2. **JWT生成**: `gen_user_jwt()`
3. **角色生成**: `gen_user_role()`
4. **权限检查**: `check_access()`

## API 接口清单

### 用户相关接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/user/regist` | POST | 用户注册 |
| `/user/login` | POST | 用户登录 |
| `/user/logout` | POST | 用户登出 |
| `/user/info` | GET | 获取用户信息 |
| `/user/list` | GET | 用户列表 |
| `/user/update` | POST | 更新用户信息 |

### 角色相关接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/role/add` | POST | 添加角色 |
| `/role/{role_id}` | PATCH | 更新角色 |
| `/role/list` | GET | 角色列表 |
| `/role/{role_id}` | DELETE | 删除角色 |

### 权限相关接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/role_access/refresh` | POST | 刷新角色权限 |
| `/role_access/list` | GET | 权限列表 |

## 审计日志

### 日志记录内容

1. 用户登录/登出
2. 用户信息修改
3. 角色创建/修改/删除
4. 权限变更
5. 密码重置

### 日志结构

```python
class AuditLogService:
    @staticmethod
    def user_login(user_payload, ip_address):
        """记录用户登录日志"""
        pass

    @staticmethod
    def forbid_user(admin_user, ip_address, target_user):
        """记录用户禁用日志"""
        pass
```

## 配置项

### 相关配置

```yaml
# 密码安全配置
password_conf:
  login_error_time_window: 30    # 登录错误时间窗口（分钟）
  max_error_times: 5             # 最大错误次数
  password_valid_period: 90      # 密码有效期（天）

# 验证码配置
use_captcha: true                # 是否启用验证码
```

## 扩展性设计

### 插件化支持

1. **认证方式扩展**: 支持LDAP、OAuth2等多种认证方式
2. **角色模型扩展**: 支持自定义角色属性
3. **权限模型扩展**: 支持更细粒度的权限控制

### 性能优化

1. **缓存机制**: Redis 缓存用户权限信息
2. **数据库优化**: 索引优化，查询优化
3. **并发控制**: 连接池管理，并发访问控制

## 错误处理

### 常见错误码

| 错误码 | 描述 |
|--------|------|
| UserNotPasswordError | 用户未设置密码 |
| UserPasswordExpireError | 密码已过期 |
| UserValidateError | 用户验证失败 |
| UserPasswordError | 密码错误 |
| UserForbiddenError | 用户被禁用 |

### 异常处理机制

1. 统一异常捕获
2. 错误信息标准化
3. 安全的错误信息返回
4. 详细的日志记录

---

## 流程图

详细的流程图和脑图已拆分到独立文件，便于维护：

- [用户认证流程图](../diagrams/user_management/auth_flow.md)
- [权限验证流程图](../diagrams/user_management/permission_flow.md)
- [用户管理模块脑图](../diagrams/user_management/mindmap.md)