# Bisheng 架构图与流程图

本目录包含 Bisheng 平台的所有架构图、流程图、脑图等可视化文档。

## 目录结构

```
diagrams/
├── README.md                      # 本文档
├── system/                        # 系统级架构图
│   ├── overall_architecture.md    # 整体系统架构图
│   ├── module_dependencies.md     # 模块依赖关系图
│   ├── tech_stack.md              # 技术栈全景图
│   ├── deployment.md              # 部署架构图
│   ├── data_flow.md               # 数据流向图
│   └── security.md                # 安全架构图
├── user_management/               # 用户管理模块图
│   ├── auth_flow.md               # 认证流程图
│   ├── permission_flow.md         # 权限验证流程图
│   └── mindmap.md                 # 模块脑图
├── workflow/                      # 工作流模块图
│   ├── execution_flow.md          # 执行流程图
│   ├── version_flow.md            # 版本管理流程图
│   ├── sequence.md                # 节点执行时序图
│   ├── node_dataflow.md           # 节点数据流图
│   └── mindmap.md                 # 模块脑图
├── knowledge/                     # 知识库模块图
│   ├── document_flow.md           # 文档处理流程图
│   ├── rag_flow.md                # RAG检索流程图
│   ├── sequence.md                # 创建时序图
│   ├── vector_dataflow.md         # 向量检索数据流图
│   └── mindmap.md                 # 模块脑图
├── chat/                          # 聊天模块图
│   ├── websocket_flow.md          # WebSocket连接流程图
│   ├── message_flow.md            # 消息处理流程图
│   ├── sequence.md                # 会话时序图
│   ├── message_dataflow.md        # 消息流转数据流图
│   └── mindmap.md                 # 模块脑图
├── llm/                           # LLM集成模块图
│   ├── call_flow.md               # LLM调用流程图
│   ├── stream_flow.md             # 流式输出流程图
│   ├── sequence.md                # 调用时序图
│   ├── multi_model_dataflow.md    # 多模型调用数据流图
│   └── mindmap.md                 # 模块脑图
└── linsight/                      # Linsight模块图
    ├── task_flow.md               # 任务执行流程图
    ├── sop_flow.md                # SOP生成流程图
    ├── sequence.md                # 执行时序图
    ├── tool_dataflow.md           # 工具调用数据流图
    ├── state_diagram.md           # 状态流转图
    └── mindmap.md                 # 模块脑图
```

## 图表类型说明

| 类型 | 语法 | 用途 |
|------|------|------|
| 流程图 | `flowchart` | 展示业务流程、决策逻辑 |
| 时序图 | `sequenceDiagram` | 展示组件间交互顺序 |
| 脑图 | `mindmap` | 展示模块结构、概念关系 |
| 状态图 | `stateDiagram-v2` | 展示状态流转 |
| 通用图 | `graph` | 展示架构、依赖关系 |

## 代码溯源

每个图表文档都包含 **代码入口** 表格，列出了与该图表相关的核心代码类和文件路径，方便开发者快速定位源代码：

```markdown
## 代码入口

| 类/函数 | 文件路径 | 说明 |
|---------|----------|------|
| `ClassName` | `src/backend/bisheng/module/file.py:行号` | 功能说明 |
```

路径格式说明：
- 路径相对于项目根目录 `bisheng/`
- 行号格式为 `文件路径:行号`（可选）
- 可直接在IDE中跳转到对应位置

## 查看方式

这些图表使用 **Mermaid** 语法编写，可以在以下环境中查看：
- GitHub / GitLab（原生支持）
- VS Code（安装 Mermaid 插件）
- Typora
- 其他支持 Mermaid 的 Markdown 编辑器

## 快速导航

### 系统架构
- [整体系统架构图](system/overall_architecture.md)
- [模块依赖关系图](system/module_dependencies.md)
- [技术栈全景图](system/tech_stack.md)
- [部署架构图](system/deployment.md)

### 核心模块
- [用户管理模块](user_management/)
- [工作流模块](workflow/)
- [知识库模块](knowledge/)
- [聊天模块](chat/)
- [LLM集成模块](llm/)
- [Linsight模块](linsight/)
