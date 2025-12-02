# Linsight执行时序图

展示Linsight执行过程中各组件的交互时序。

## 代码入口

| 类/函数 | 文件路径 | 说明 |
|---------|----------|------|
| `LinsightWorkflowTask` | `src/backend/bisheng/linsight/task_exec.py` | 任务执行器 |
| `LinsightStateMessageManager` | `src/backend/bisheng/linsight/state_message_manager.py` | 状态消息管理 |
| `MessageStreamHandle` | `src/backend/bisheng/api/services/linsight/message_stream_handle.py` | 消息流处理 |
| `LinsightAgent` | `src/backend/bisheng_langchain/linsight/agent.py` | Agent执行 |
| `ToolServices` | `src/backend/bisheng/api/services/tool.py` | 工具服务 |
| `LinsightSessionVersionDao` | `src/backend/bisheng/database/models/linsight_session_version.py` | 会话版本数据 |
| `worker.py` | `src/backend/bisheng/linsight/worker.py` | 后台Worker |

```mermaid
sequenceDiagram
    participant User as 用户
    participant API as Linsight API
    participant Task as TaskExecutor
    participant Agent as LinsightAgent
    participant LLM as LLM服务
    participant Tool as 工具服务
    participant State as StateManager
    participant WS as WebSocket

    User->>API: 提交问题
    API->>Task: 创建执行任务
    Task->>State: 初始化状态
    Task->>Agent: 生成SOP
    Agent->>LLM: 调用LLM
    LLM-->>Agent: 返回SOP
    Agent-->>Task: SOP生成完成
    Task->>WS: 推送SOP给用户
    WS-->>User: 显示SOP

    User->>API: 确认SOP
    API->>Task: 开始执行
    
    loop 执行任务
        Task->>Agent: 执行当前任务
        Agent->>LLM: 分析任务
        LLM-->>Agent: 返回执行计划
        
        opt 需要工具
            Agent->>Tool: 调用工具
            Tool-->>Agent: 返回结果
        end
        
        opt 需要用户输入
            Agent->>WS: 请求用户输入
            WS-->>User: 显示输入请求
            User->>API: 提交输入
            API->>Agent: 传递用户输入
        end
        
        Agent->>State: 更新状态
        Agent->>WS: 推送进度
        WS-->>User: 显示进度
    end
    
    Task->>Agent: 汇总结果
    Agent->>LLM: 生成报告
    LLM-->>Agent: 返回报告
    Agent-->>Task: 执行完成
    Task->>WS: 推送最终结果
    WS-->>User: 显示结果
```

## 时序说明

### 参与组件

| 组件 | 职责 |
|------|------|
| 用户 | 提交问题，确认SOP，提供输入 |
| Linsight API | 接收请求，返回响应 |
| TaskExecutor | 任务执行调度 |
| LinsightAgent | 智能代理执行 |
| LLM服务 | 大模型调用 |
| 工具服务 | 工具调用 |
| StateManager | 状态管理 |
| WebSocket | 实时消息推送 |

### 交互特点

- SOP生成后需用户确认
- 执行中可请求用户输入
- 实时推送执行进度
- 最终生成执行报告
