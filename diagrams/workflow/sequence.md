# 节点执行时序图

展示工作流节点执行过程中各组件的交互时序。

## 代码入口

| 类/函数 | 文件路径 | 说明 |
|---------|----------|------|
| `BaseNode` | `src/backend/bisheng/workflow/nodes/base.py:19` | 节点基类 |
| `BaseNode.run()` | `src/backend/bisheng/workflow/nodes/base.py:183` | 节点执行入口 |
| `BaseNode._run()` | `src/backend/bisheng/workflow/nodes/base.py:80` | 节点执行抽象方法 |
| `NodeFactory` | `src/backend/bisheng/workflow/nodes/node_manage.py:33` | 节点工厂类 |
| `NODE_CLASS_MAP` | `src/backend/bisheng/workflow/nodes/node_manage.py:16` | 节点类型映射 |
| `BaseCallback` | `src/backend/bisheng/workflow/callback/base_callback.py` | 回调处理基类 |
| `WorkFlowService.run_once()` | `src/backend/bisheng/api/services/workflow.py:111` | 单节点执行服务 |

```mermaid
sequenceDiagram
    participant User as 用户
    participant API as API层
    participant Engine as 工作流引擎
    participant Node as 节点执行器
    participant LLM as LLM服务
    participant KB as 知识库
    participant WS as WebSocket

    User->>API: 发起执行请求
    API->>Engine: 调用执行方法
    Engine->>Engine: 解析工作流定义
    Engine->>Node: 执行输入节点
    Node-->>Engine: 返回用户输入
    Engine->>Node: 执行知识检索节点
    Node->>KB: 语义检索
    KB-->>Node: 返回相关文档
    Node-->>Engine: 返回检索结果
    Engine->>Node: 执行LLM节点
    Node->>LLM: 调用大模型
    LLM-->>Node: 流式返回结果
    Node->>WS: 推送流式消息
    WS-->>User: 实时显示
    Node-->>Engine: 返回LLM结果
    Engine->>Node: 执行输出节点
    Node-->>Engine: 格式化输出
    Engine-->>API: 返回执行结果
    API-->>User: 返回最终结果
```

## 时序说明

### 参与组件

| 组件 | 职责 |
|------|------|
| 用户 | 发起请求，接收结果 |
| API层 | 接收请求，返回响应 |
| 工作流引擎 | 解析和调度节点执行 |
| 节点执行器 | 执行具体节点逻辑 |
| LLM服务 | 大模型调用 |
| 知识库 | 文档检索 |
| WebSocket | 实时消息推送 |

### 执行特点

- **流式输出**: LLM结果实时推送
- **异步处理**: 耗时操作异步执行
- **状态同步**: 执行状态实时更新
