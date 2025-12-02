# Bisheng API 接口文档

## 概述

Bisheng 平台提供了一套完整的 RESTful API 接口，支持用户管理、工作流操作、知识库管理、聊天交互等各种功能。所有 API 接口都遵循 REST 设计原则，使用 JSON 格式进行数据交换。

## API 基础信息

### 基础URL
```
http://localhost:7860/api/v1
```

### 认证方式
所有受保护的 API 接口都需要通过 JWT Token 进行认证：
```
Authorization: Bearer <jwt_token>
```

### 响应格式
所有 API 响应都采用统一的 JSON 格式：
```json
{
  "status_code": 200,
  "status_message": "success",
  "data": {}
}
```

### 错误响应格式
```json
{
  "status_code": 400,
  "status_message": "错误描述信息"
}
```

## API 版本

### v1 版本 (稳定版)
- 基础功能API
- 用户管理
- 工作流管理
- 知识库管理
- 聊天交互
- Linsight 灵思接口

### v2 版本 (开放API / RPC风格)
v2 API 采用 RPC 风格设计，主要用于外部系统集成和开放平台场景。

**特点**：
- 简化的认证机制（支持默认操作员）
- 面向集成的接口设计
- 更灵活的参数传递

**包含模块**：
- `/api/v2/knowledge` - 知识库元数据管理
- `/api/v2/filelib` - 文件库管理
- `/api/v2/chat` - 聊天接口
- `/api/v2/rpc` - RPC调用接口
- `/api/v2/flow` - 技能流程接口
- `/api/v2/assistant` - 助手接口
- `/api/v2/workflow` - 工作流接口
- `/api/v2/workstation` - 工作站接口
- `/api/v2/llm` - LLM模型接口

## 用户管理接口

### 用户注册
```
POST /user/regist
```

**请求参数:**
```json
{
  "user_name": "用户名",
  "password": "密码",
  "captcha": "验证码",
  "captcha_key": "验证码key"
}
```

**响应示例:**
```json
{
  "status_code": 200,
  "status_message": "success",
  "data": {
    "user_id": 1,
    "user_name": "testuser",
    "nick_name": "测试用户",
    "create_time": "2023-01-01T00:00:00"
  }
}
```

### 用户登录
```
POST /user/login
```

**请求参数:**
```json
{
  "user_name": "用户名",
  "password": "密码",
  "captcha": "验证码",
  "captcha_key": "验证码key"
}
```

**响应示例:**
```json
{
  "status_code": 200,
  "status_message": "success",
  "data": {
    "user_id": 1,
    "user_name": "testuser",
    "role": "admin",
    "access_token": "jwt_token",
    "refresh_token": "refresh_token"
  }
}
```

### 获取用户信息
```
GET /user/info
```

**响应示例:**
```json
{
  "status_code": 200,
  "status_message": "success",
  "data": {
    "user_id": 1,
    "user_name": "testuser",
    "nick_name": "测试用户",
    "role": "admin",
    "web_menu": [],
    "admin_groups": []
  }
}
```

### 用户列表
```
GET /user/list
```

**查询参数:**
- name: 用户名模糊搜索
- page_size: 每页数量 (默认10)
- page_num: 页码 (默认1)
- group_id: 用户组ID列表
- role_id: 角色ID列表

**响应示例:**
```json
{
  "status_code": 200,
  "status_message": "success",
  "data": {
    "data": [
      {
        "user_id": 1,
        "user_name": "testuser",
        "nick_name": "测试用户",
        "roles": [],
        "groups": []
      }
    ],
    "total": 1
  }
}
```

### 更新用户状态
```
POST /user/update
```

**请求参数:**
```json
{
  "user_id": 1,
  "delete": 0  // 0:启用 1:禁用
}
```

## 角色管理接口

### 创建角色
```
POST /role/add
```

**请求参数:**
```json
{
  "group_id": 1,
  "role_name": "角色名称",
  "remark": "角色描述"
}
```

### 角色列表
```
GET /role/list
```

**查询参数:**
- role_name: 角色名称模糊搜索
- page: 页码
- limit: 每页数量

### 更新角色
```
PATCH /role/{role_id}
```

**请求参数:**
```json
{
  "role_name": "新角色名称",
  "remark": "新角色描述"
}
```

### 删除角色
```
DELETE /role/{role_id}
```

## 工作流接口

### 创建工作流
```
POST /workflow/create
```

**请求参数:**
```json
{
  "name": "工作流名称",
  "description": "工作流描述"
}
```

**响应示例:**
```json
{
  "status_code": 200,
  "status_message": "success",
  "data": {
    "id": "flow_id",
    "name": "工作流名称",
    "description": "工作流描述",
    "status": 1,
    "version_id": 1
  }
}
```

### 工作流列表
```
GET /workflow/list
```

**查询参数:**
- name: 工作流名称模糊搜索
- status: 状态筛选
- flow_type: 类型筛选
- page_size: 每页数量
- page_num: 页码

### 获取工作流详情
```
GET /workflow/get_one_flow/{flow_id}
```

### 更新工作流
```
PATCH /workflow/update/{flow_id}
```

**请求参数:**
```json
{
  "name": "新名称",
  "description": "新描述",
  "status": 2  // 1:草稿 2:在线 3:离线
}
```

### 更新工作流状态
```
PATCH /workflow/status
```

**请求参数:**
```json
{
  "flow_id": "工作流ID",
  "version_id": 1,
  "status": 2
}
```

## 工作流版本管理接口

### 创建版本
```
POST /workflow/versions
```

**查询参数:**
- flow_id: 工作流ID

**请求参数:**
```json
{
  "name": "版本名称",
  "description": "版本描述",
  "data": "流程定义数据"
}
```

### 版本列表
```
GET /workflow/versions
```

**查询参数:**
- flow_id: 工作流ID

### 更新版本
```
PUT /workflow/versions/{version_id}
```

### 删除版本
```
DELETE /workflow/versions/{version_id}
```

### 获取版本详情
```
GET /workflow/versions/{version_id}
```

### 切换当前版本
```
POST /workflow/change_version
```

**查询参数:**
- flow_id: 工作流ID
- version_id: 版本ID

## 单节点执行
```
POST /workflow/run_once
```

**请求参数:**
```json
{
  "node_input": {},  // 节点输入参数
  "node_data": {}    // 节点数据
}
```

## 知识库接口

### 创建知识库
```
POST /knowledge/base/create
```

**请求参数:**
```json
{
  "name": "知识库名称",
  "description": "知识库描述"
}
```

### 知识库列表
```
GET /knowledge/base/list
```

**查询参数:**
- name: 知识库名称模糊搜索
- page_size: 每页数量
- page_num: 页码

### 获取知识库详情
```
GET /knowledge/base/{id}
```

### 更新知识库
```
PUT /knowledge/base/{id}
```

### 删除知识库
```
DELETE /knowledge/base/{id}
```

## 文档管理接口

### 上传文档
```
POST /knowledge/upload
```

**表单参数:**
- file: 文档文件
- knowledge_id: 知识库ID

### 文档列表
```
GET /knowledge/list
```

**查询参数:**
- knowledge_id: 知识库ID
- name: 文档名称模糊搜索
- page_size: 每页数量
- page_num: 页码

### 获取文档详情
```
GET /knowledge/{id}
```

### 删除文档
```
DELETE /knowledge/{id}
```

### 获取文档处理状态
```
GET /knowledge/{id}/status
```

## 检索接口

### 语义检索
```
POST /knowledge/search
```

**请求参数:**
```json
{
  "query": "检索查询",
  "knowledge_id": 1,
  "top_k": 10
}
```

### 关键词检索
```
POST /knowledge/keyword_search
```

### RAG问答
```
POST /knowledge/rag
```

**请求参数:**
```json
{
  "question": "问题",
  "knowledge_id": 1
}
```

## 聊天接口

### 聊天WebSocket连接
```
WebSocket /workflow/chat/{workflow_id}
```

**查询参数:**
- chat_id: 聊天ID

### 获取聊天历史消息
```
GET /chat/message/history
```

**查询参数:**
- chat_id: 聊天ID
- page_size: 每页数量
- page_num: 页码

### 删除聊天消息
```
DELETE /chat/message/delete
```

**请求参数:**
```json
{
  "message_id": "消息ID"
}
```

### 中断聊天处理
```
POST /chat/interrupt
```

**请求参数:**
```json
{
  "chat_id": "聊天ID"
}
```

## 会话管理接口

### 创建聊天会话
```
POST /chat/session/create
```

**请求参数:**
```json
{
  "flow_id": "流程ID",
  "chat_type": "聊天类型"
}
```

### 会话列表
```
GET /chat/session/list
```

**查询参数:**
- flow_id: 流程ID
- chat_type: 聊天类型
- page_size: 每页数量
- page_num: 页码

### 获取会话详情
```
GET /chat/session/{session_id}
```

### 更新会话信息
```
PUT /chat/session/{session_id}
```

### 删除会话
```
DELETE /chat/session/{session_id}
```

## LLM接口

### 聊天补全
```
POST /llm/chat/completion
```

**请求参数:**
```json
{
  "provider": "openai",
  "model": "gpt-3.5-turbo",
  "messages": [
    {
      "role": "user",
      "content": "你好"
    }
  ]
}
```

### 文本嵌入
```
POST /llm/embedding
```

**请求参数:**
```json
{
  "provider": "openai",
  "model": "text-embedding-ada-002",
  "texts": ["文本1", "文本2"]
}
```

### 流式聊天补全
```
POST /llm/stream/chat/completion
```

## 统计监控接口

### 使用统计
```
GET /llm/stats/usage
```

**查询参数:**
- start_time: 开始时间
- end_time: 结束时间
- provider: 提供商筛选

### 成本统计
```
GET /llm/stats/cost
```

### 调用日志
```
GET /llm/logs
```

**查询参数:**
- provider: 提供商筛选
- model: 模型筛选
- status: 状态筛选
- page_size: 每页数量
- page_num: 页码

## 系统接口

### 健康检查
```
GET /health
```

**响应示例:**
```json
{
  "status": "OK"
}
```

### 获取验证码
```
GET /user/get_captcha
```

**响应示例:**
```json
{
  "status_code": 200,
  "status_message": "success",
  "data": {
    "captcha_key": "验证码key",
    "captcha": "base64编码的验证码图片",
    "user_capthca": true
  }
}
```

### 获取RSA公钥
```
GET /user/public_key
```

**响应示例:**
```json
{
  "status_code": 200,
  "status_message": "success",
  "data": {
    "public_key": "RSA公钥"
  }
}
```

## 错误码说明

### 通用错误码
| 错误码 | 说明 |
|--------|------|
| 200 | 成功 |
| 400 | 请求参数错误 |
| 401 | 未认证 |
| 403 | 权限不足 |
| 404 | 资源不存在 |
| 500 | 服务器内部错误 |

### 业务错误码
| 错误码 | 说明 |
|--------|------|
| 10001 | 用户名已存在 |
| 10002 | 用户名或密码错误 |
| 10003 | 用户被禁用 |
| 10004 | 密码已过期 |
| 20001 | 工作流名称已存在 |
| 20002 | 工作流在线状态下不允许编辑 |
| 30001 | 知识库处理失败 |

## 请求示例

### Python 示例
```python
import requests

# 用户登录
login_data = {
    "user_name": "testuser",
    "password": "password123"
}

response = requests.post(
    "http://localhost:7860/api/v1/user/login",
    json=login_data
)

if response.status_code == 200:
    token = response.json()["data"]["access_token"]

    # 获取用户信息
    headers = {"Authorization": f"Bearer {token}"}
    user_response = requests.get(
        "http://localhost:7860/api/v1/user/info",
        headers=headers
    )
    print(user_response.json())
```

### JavaScript 示例
```javascript
// 用户登录
const loginData = {
    user_name: "testuser",
    password: "password123"
};

fetch("http://localhost:7860/api/v1/user/login", {
    method: "POST",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify(loginData)
})
.then(response => response.json())
.then(data => {
    const token = data.data.access_token;

    // 获取用户信息
    fetch("http://localhost:7860/api/v1/user/info", {
        headers: {
            "Authorization": `Bearer ${token}`
        }
    })
    .then(response => response.json())
    .then(userInfo => console.log(userInfo));
});
```

## WebSocket 连接示例

### JavaScript WebSocket 示例
```javascript
const workflowId = "your_workflow_id";
const token = "your_jwt_token";

const ws = new WebSocket(
    `ws://localhost:7860/api/v1/workflow/chat/${workflowId}`
);

ws.onopen = function(event) {
    // 发送认证信息
    ws.send(JSON.stringify({
        type: "auth",
        token: token
    }));

    // 发送聊天消息
    ws.send(JSON.stringify({
        type: "message",
        data: {
            input: "你好，帮我分析一下这个文档"
        }
    }));
};

ws.onmessage = function(event) {
    const message = JSON.parse(event.data);
    console.log("收到消息:", message);
};

ws.onclose = function(event) {
    console.log("连接已关闭");
};
```

## 限流和配额

### API 限流
- 用户级别限流：每分钟1000次请求
- IP级别限流：每分钟10000次请求
- 特殊接口限流：根据业务需求单独设置

### LLM 调用配额
- 按用户/组织设置月度调用配额
- 超出配额后自动降级或拒绝服务
- 支持配额预警和通知

## 版本兼容性

### 向后兼容
- API v1 保证长期向后兼容
- 重大变更提前3个月通知
- 提供迁移指南和工具

### 废弃策略
- 废弃的接口标记为 deprecated
- 提供替代方案和迁移时间表
- 废弃接口保留6个月后移除

---

## v2 API 详细说明 (开放API)

v2 API 位于 `/api/v2` 路径下，采用 RPC 风格设计，主要用于外部系统集成。

### 知识库元数据接口

#### 添加元数据字段
```
POST /api/v2/knowledge/add_metadata_fields
```

**请求参数:**
```json
{
  "knowledge_id": 1,
  "fields": [
    {
      "field_name": "author",
      "field_type": "string",
      "description": "作者"
    }
  ]
}
```

#### 修改元数据字段
```
PUT /api/v2/knowledge/modify_metadata_fields
```

#### 删除元数据字段
```
DELETE /api/v2/knowledge/delete_metadata_fields
```

**请求参数:**
```json
{
  "knowledge_id": 1,
  "field_names": ["author", "category"]
}
```

#### 获取元数据字段
```
GET /api/v2/knowledge/get_metadata_fields/{knowledge_id}
```

### 文件库接口

#### 上传文件
```
POST /api/v2/filelib/upload
```

#### 文件列表
```
GET /api/v2/filelib/list
```

#### 删除文件
```
DELETE /api/v2/filelib/delete
```

### 聊天接口

#### WebSocket 聊天
```
WebSocket /api/v2/chat/ws/{flow_id}
```

**查询参数:**
- `chat_id`: 会话ID（可选）
- `tweak`: 参数调整（可选）
- `knowledge_id`: 知识库ID（可选）

#### 消息反馈
```
POST /api/v2/chat/liked
```

```
POST /api/v2/chat/solved
```

```
POST /api/v2/chat/comment
```

### 工作流接口

#### 执行工作流
```
POST /api/v2/workflow/execute
```

#### 获取执行状态
```
GET /api/v2/workflow/status/{execution_id}
```

### 助手接口

#### 创建会话
```
POST /api/v2/assistant/session
```

#### 发送消息
```
POST /api/v2/assistant/message
```

### LLM 接口

#### 模型列表
```
GET /api/v2/llm/models
```

#### 调用模型
```
POST /api/v2/llm/invoke
```

### v2 API 认证

v2 API 支持两种认证方式：

1. **JWT Token 认证**（与 v1 相同）
```
Authorization: Bearer <jwt_token>
```

2. **默认操作员模式**
通过系统配置的默认操作员身份执行操作，适用于系统集成场景。

### v2 API 使用场景

- **外部系统集成**: 与第三方系统对接
- **批量操作**: 批量导入/导出数据
- **自动化流程**: 无人值守的自动化任务
- **开放平台**: 提供给合作伙伴的开放接口