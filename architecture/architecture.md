# Bisheng 架构设计文档

## 整体架构

Bisheng 采用前后端分离的微服务架构设计，后端基于 Python 的 FastAPI 框架构建，前端使用 React + TypeScript 技术栈。整个系统通过 Docker 容器化部署，便于扩展和维护。

## 后端架构层次

### 1. 接入层 (API Layer)
- **技术实现**: FastAPI
- **功能职责**:
  - 提供 RESTful API 接口
  - 请求参数验证和响应格式化
  - 身份认证和权限控制
  - 请求日志记录和监控

### 2. 业务逻辑层 (Service Layer)
- **功能职责**:
  - 实现具体的业务逻辑
  - 数据处理和转换
  - 第三方服务集成
  - 业务规则验证

### 3. 数据访问层 (Data Access Layer)
- **技术实现**: SQLAlchemy + SQLModel
- **功能职责**:
  - 数据库实体映射
  - 数据持久化操作
  - 数据查询优化
  - 事务管理

### 4. 基础设施层 (Infrastructure Layer)
- **功能职责**:
  - 缓存管理 (Redis)
  - 文件存储 (MinIO)
  - 消息队列 (Celery)
  - 向量数据库 (Milvus)
  - 搜索引擎 (Elasticsearch)

## 核心模块详解

### 1. 用户管理模块 (User Management)

#### 功能特性
- 用户注册、登录、注销
- 用户角色和权限管理
- 用户组管理
- 密码安全策略
- JWT Token 认证机制

#### 架构设计
```
api/v1/user.py          # 用户相关API接口
database/models/user.py # 用户数据模型
services/user_service.py # 用户业务逻辑
```

#### 关键设计点
- 使用 JWT 进行身份认证
- 支持 RBAC 权限控制
- 密码加密存储
- 登录失败次数限制
- 密码过期策略

### 2. 工作流引擎模块 (Workflow Engine)

#### 功能特性
- 可视化流程编排
- 多种节点类型支持
- 流程版本管理
- 在线/离线状态控制
- 实时执行监控

#### 架构设计
```
api/v1/workflow.py          # 工作流API接口
database/models/flow.py     # 流程数据模型
database/models/flow_version.py # 流程版本数据模型
services/flow_service.py    # 流程业务逻辑
workflow/                   # 工作流引擎核心
├── callback/              # 回调处理
├── common/                # 公共组件
├── edges/                 # 边定义
├── graph/                 # 图结构
└── nodes/                 # 节点实现（14种节点类型）
```

#### 关键设计点
- 支持流程的版本控制
- 提供 WebSocket 实时通信
- 支持单节点调试运行
- 流程状态管理和变更控制

### 3. 知识库模块 (Knowledge Base)

#### 功能特性
- 文档上传和管理
- 文档解析和处理
- 向量索引构建
- 语义搜索和检索
- 知识问答功能

#### 架构设计
```
knowledge/                  # 知识库模块
├── api/                   # 知识库API
├── domain/                # 领域模型
├── rag/                   # RAG相关实现
└── services/              # 知识库服务
```

#### 关键设计点
- 集成 Milvus 向量数据库
- 支持多种文档格式解析
- 实现 RAG (Retrieval-Augmented Generation) 检索增强生成
- 支持语义搜索和关键词搜索

### 4. 聊天模块 (Chat)

#### 功能特性
- 实时聊天通信
- 多轮对话管理
- 上下文保持
- 消息历史记录
- 多种消息类型支持

#### 架构设计
```
chat/                       # 聊天模块
├── manager.py             # 聊天管理器
├── client.py              # 客户端处理
├── message.py             # 消息处理
└── session.py             # 会话管理
```

#### 关键设计点
- 使用 WebSocket 实现实时通信
- 支持多种聊天类型（普通聊天、工作流聊天等）
- 消息持久化存储
- 会话状态管理

### 5. 大语言模型集成模块 (LLM Integration)

#### 功能特性
- 多种 LLM 模型支持
- 统一的模型调用接口
- 模型参数配置管理
- 调用日志和监控
- 成本控制和优化

#### 架构设计
```
llm/                        # LLM模块
├── api/                   # LLM API接口
├── providers/             # 模型提供商适配
├── models/                # 模型封装
└── services/              # LLM服务
```

#### 关键设计点
- 支持主流 LLM 提供商（OpenAI、Anthropic等）
- 统一的抽象接口设计
- 模型调用成本监控
- 缓存机制优化性能

## 数据流向

### 用户请求处理流程
1. 用户发起 HTTP 请求到 API 层
2. API 层进行参数验证和身份认证
3. 调用相应的 Service 层处理业务逻辑
4. Service 层访问 Database 层进行数据操作
5. 返回处理结果给客户端

### 异步任务处理流程
1. 用户触发耗时操作
2. API 层将任务发送到 Celery 队列
3. Worker 进程从队列获取任务并执行
4. 执行结果存储到数据库或缓存
5. 通过 WebSocket 或轮询通知客户端

## 部署架构

### 容器化部署
所有服务都通过 Docker 容器化部署，主要包括：
- 数据库服务 (MySQL)
- 缓存服务 (Redis)
- 搜索服务 (Elasticsearch)
- 向量数据库 (Milvus)
- 对象存储 (MinIO)
- 应用服务 (Backend/Frontend)
- 异步任务服务 (Celery Worker)

### 服务间通信
- HTTP/REST: 用于服务间同步调用
- Redis Pub/Sub: 用于实时消息推送
- Celery Queue: 用于异步任务处理
- WebSocket: 用于实时双向通信

## 性能优化策略

### 缓存策略
- Redis 缓存热点数据
- 数据库查询结果缓存
- Session 缓存
- 用户权限缓存

### 数据库优化
- 索引优化
- 查询优化
- 连接池管理
- 读写分离

### 异步处理
- 耗时操作异步化
- 批量处理优化
- 并发控制

## 安全设计

### 认证授权
- JWT Token 认证
- RBAC 权限控制
- CSRF 防护
- XSS 防护

### 数据安全
- 敏感数据加密存储
- 数据传输加密 (HTTPS)
- 数据库访问控制
- 文件访问权限控制

### 应用安全
- 输入验证和过滤
- 输出编码
- 安全日志记录
- 安全审计