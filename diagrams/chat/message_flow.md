# 消息处理流程图

展示聊天消息的接收和处理流程。

## 代码入口

| 类/函数 | 文件路径 | 说明 |
|---------|----------|------|
| `Handler` | `src/backend/bisheng/chat/handlers.py` | 消息处理器核心类 |
| `Handler.dispatch_task()` | `src/backend/bisheng/chat/handlers.py:41` | 消息分发入口 |
| `Handler.process_message()` | `src/backend/bisheng/chat/handlers.py:201` | 处理普通消息 |
| `Handler.process_file()` | `src/backend/bisheng/chat/handlers.py:313` | 处理文件消息 |
| `Handler.process_autogen()` | `src/backend/bisheng/chat/handlers.py:373` | 处理AutoGen消息 |
| `Handler.process_stop()` | `src/backend/bisheng/chat/handlers.py:59` | 处理停止消息 |
| `WorkType` | `src/backend/bisheng/chat/types.py` | 聊天业务类型枚举 |

```mermaid
flowchart TD
    A[接收客户端消息] --> B[解析消息JSON]
    B --> C{消息类型判断}
    C -->|start| D[初始化聊天上下文]
    C -->|message| E[处理用户消息]
    C -->|interrupt| F[中断当前处理]
    C -->|其他| G[未知消息类型]
    
    E --> H{聊天类型判断}
    H -->|workflow| I[工作流聊天处理]
    H -->|assistant| J[助手聊天处理]
    H -->|flow| K[技能聊天处理]
    
    I --> L[调用工作流引擎]
    J --> M[调用助手服务]
    K --> N[调用技能服务]
    
    L --> O[流式返回结果]
    M --> O
    N --> O
    
    O --> P[推送消息到客户端]
    P --> Q[保存消息记录]
    Q --> R[更新会话状态]
```

## 消息说明

### 消息类型

| 类型 | 说明 |
|------|------|
| start | 开始聊天 |
| message | 普通消息 |
| interrupt | 中断处理 |
| progress | 处理进度 |
| result | 处理结果 |
| error | 错误信息 |
| end | 结束聊天 |

### 聊天类型

| 类型 | 说明 |
|------|------|
| workflow | 工作流聊天 |
| assistant | 助手聊天 |
| flow | 技能聊天 |
| linsight | Linsight聊天 |

### 消息格式

```json
{
  "type": "message",
  "data": {
    "input": "用户输入内容"
  },
  "trace_id": "追踪ID"
}
```
