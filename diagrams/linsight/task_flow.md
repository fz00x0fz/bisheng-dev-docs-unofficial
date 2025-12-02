# Linsight任务执行流程图

展示Linsight任务从提交到完成的执行流程。

## 代码入口

| 类/函数 | 文件路径 | 说明 |
|---------|----------|------|
| `LinsightWorkflowTask` | `src/backend/bisheng/linsight/task_exec.py:49` | 任务执行器核心类 |
| `LinsightWorkflowTask._managed_execution()` | `src/backend/bisheng/linsight/task_exec.py:66` | 执行资源管理 |
| `LinsightStateMessageManager` | `src/backend/bisheng/linsight/state_message_manager.py` | 状态消息管理 |
| `LinsightWorkbenchImpl` | `src/backend/bisheng/api/services/linsight/workbench_impl.py` | 工作台实现 |
| `LinsightAgent` | `src/backend/bisheng_langchain/linsight/agent.py` | Linsight Agent |
| `linsight.py` | `src/backend/bisheng/api/v1/linsight.py:45` | API路由入口 |
| `LinsightExecuteTaskDao` | `src/backend/bisheng/database/models/linsight_execute_task.py` | 任务数据访问 |

```mermaid
flowchart TD
    A[用户提交问题] --> B[创建SessionVersion]
    B --> C[解析用户问题]
    C --> D{已有SOP?}
    D -->|否| E[调用LLM生成SOP]
    D -->|是| F[加载已有SOP]
    E --> G[用户确认/修改SOP]
    F --> G
    G --> H[SOP确认完成]
    H --> I[生成执行任务列表]
    I --> J[初始化LinsightAgent]
    J --> K[开始执行任务]
    K --> L{当前任务}
    L --> M[执行任务步骤]
    M --> N{需要工具调用?}
    N -->|是| O[调用工具]
    O --> P[获取工具结果]
    P --> Q{需要用户输入?}
    N -->|否| Q
    Q -->|是| R[暂停等待用户]
    R --> S[用户提交输入]
    S --> M
    Q -->|否| T{任务完成?}
    T -->|否| M
    T -->|是| U{还有下一任务?}
    U -->|是| L
    U -->|否| V[汇总执行结果]
    V --> W[生成最终报告]
    W --> X[返回结果给用户]
```

## 执行说明

### 执行阶段

| 阶段 | 说明 |
|------|------|
| 问题解析 | 理解用户意图 |
| SOP生成 | 生成执行计划 |
| 任务分解 | 分解为子任务 |
| 任务执行 | 逐个执行任务 |
| 结果汇总 | 汇总执行结果 |

### 人机交互

- 用户可修改SOP
- 执行中可请求用户输入
- 支持中断和恢复

### 工具调用

- 代码解释器
- 知识库检索
- 文件处理
- 自定义工具
