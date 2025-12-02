# 消息流转数据流图

展示聊天消息在系统中的流转过程。

## 代码入口

| 类/函数 | 文件路径 | 说明 |
|---------|----------|------|
| `ChatManager` | `src/backend/bisheng/chat/manager.py` | 消息管理 |
| `ChatHistory` | `src/backend/bisheng/chat/manager.py:39` | 历史记录管理 |
| `ChatMessage` | `src/backend/bisheng/api/v1/schemas.py` | 消息模型 |
| `ChatMessageDao` | `src/backend/bisheng/database/models/message.py` | 消息数据访问 |
| `MessageSessionDao` | `src/backend/bisheng/database/models/session.py` | 会话数据访问 |

```mermaid
flowchart LR
    subgraph "客户端"
        USER[用户输入]
        DISPLAY[消息展示]
    end

    subgraph "WebSocket层"
        WS_IN[消息接收]
        WS_OUT[消息发送]
    end

    subgraph "处理层"
        PARSE[消息解析]
        ROUTE[消息路由]
        HANDLER[消息处理器]
    end

    subgraph "执行层"
        WORKFLOW[工作流引擎]
        ASSISTANT[助手服务]
        FLOW[技能服务]
    end

    subgraph "存储层"
        REDIS[(Redis缓存)]
        MYSQL[(MySQL存储)]
    end

    USER --> WS_IN
    WS_IN --> PARSE
    PARSE --> ROUTE
    ROUTE --> HANDLER
    HANDLER --> WORKFLOW
    HANDLER --> ASSISTANT
    HANDLER --> FLOW
    WORKFLOW --> WS_OUT
    ASSISTANT --> WS_OUT
    FLOW --> WS_OUT
    WS_OUT --> DISPLAY
    HANDLER --> REDIS
    HANDLER --> MYSQL
```

## 数据流说明

### 消息流入

| 阶段 | 说明 |
|------|------|
| 用户输入 | 用户在界面输入消息 |
| 消息接收 | WebSocket接收消息 |
| 消息解析 | 解析JSON格式 |
| 消息路由 | 根据类型路由 |
| 消息处理 | 执行业务逻辑 |

### 消息流出

| 阶段 | 说明 |
|------|------|
| 执行结果 | 业务执行返回结果 |
| 消息发送 | WebSocket发送消息 |
| 消息展示 | 界面展示消息 |

### 数据存储

| 存储 | 用途 |
|------|------|
| Redis | 会话状态、临时数据 |
| MySQL | 消息记录、会话历史 |
