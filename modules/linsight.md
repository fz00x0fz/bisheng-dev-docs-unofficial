# Linsight 模块详细设计文档

## 模块概述

Linsight（灵思）是 Bisheng 平台的核心特性之一，是一个具备"专家级品味"的通用代理系统。通过 AGL（Agent Guidance Language）框架，将领域专家的偏好、经验和业务逻辑嵌入到 AI 中，使代理在处理任务时能够表现出"专家级的理解"。

## 核心特性

### 1. SOP（标准操作程序）驱动
- **智能 SOP 生成**：根据用户问题自动生成执行计划
- **SOP 反馈优化**：支持用户反馈，持续优化执行方案
- **精选案例库**：支持将优秀 SOP 保存为精选案例供参考

### 2. 任务自动分解与执行
- **任务树生成**：将复杂任务分解为可执行的子任务
- **智能工具调用**：自动选择合适的工具执行任务
- **人在回路**：支持执行过程中的用户干预和反馈

### 3. 多模态支持
- **文件处理**：支持上传和解析多种格式文件
- **知识库集成**：支持个人知识库和组织知识库
- **代码执行**：内置代码解释器沙盒环境

## 模块结构

```
bisheng/linsight/                    # Linsight 核心模块
├── __init__.py
├── task_exec.py                     # 任务执行器
├── state_message_manager.py         # 状态消息管理器
├── utils.py                         # 工具函数
└── worker.py                        # 异步任务队列

bisheng/api/v1/linsight.py           # Linsight API 接口
bisheng/api/services/linsight/       # Linsight 业务服务
├── message_stream_handle.py         # 消息流处理
├── sop_manage.py                    # SOP 管理服务
└── workbench_impl.py                # 工作台实现

bisheng/database/models/             # 数据模型
├── linsight_sop.py                  # SOP 数据模型
├── linsight_session_version.py      # 会话版本模型
└── linsight_execute_task.py         # 执行任务模型

bisheng_langchain/linsight/          # LangChain 集成
├── agent.py                         # Linsight Agent
├── const.py                         # 常量定义
├── event.py                         # 事件定义
├── manage.py                        # 任务管理
├── prompt.py                        # Prompt 模板
└── utils.py                         # 工具函数
```

## 数据模型设计

### 会话版本表 (LinsightSessionVersion)

```python
class LinsightSessionVersion(SQLModelSerializable, table=True):
    id: str                          # 会话版本ID (UUID)
    session_id: str                  # 关联的消息会话ID
    user_id: int                     # 用户ID
    question: str                    # 用户问题
    title: Optional[str]             # 会话标题
    tools: Optional[List[Dict]]      # 可用工具列表
    personal_knowledge_enabled: bool # 是否启用个人知识库
    org_knowledge_enabled: bool      # 是否启用组织知识库
    files: Optional[List[Dict]]      # 上传的文件列表
    sop: Optional[str]               # SOP 内容
    output_result: Optional[Dict]    # 输出结果
    status: SessionVersionStatusEnum # 会话版本状态
    score: Optional[int]             # 会话评分 (1-5)
    execute_feedback: Optional[str]  # 执行结果反馈
    has_reexecute: bool              # 是否有重新执行
    version: datetime                # 版本时间
```

### 会话版本状态枚举

```python
class SessionVersionStatusEnum(str, Enum):
    NOT_STARTED = "not_started"      # 未执行
    IN_PROGRESS = "in_progress"      # 进行中
    COMPLETED = "completed"          # 运行完成
    FAILED = "failed"                # 运行失败
    SOP_GENERATION_FAILED = "sop_generation_failed"  # SOP 生成失败
    TERMINATED = "terminated"        # 终止
```

### SOP 模型 (LinsightSOP)

```python
class LinsightSOP(SQLModelSerializable, table=True):
    id: int                          # SOP 唯一ID
    name: str                        # SOP 名称
    description: Optional[str]       # SOP 描述
    user_id: int                     # 用户ID
    content: str                     # SOP 内容
    rating: Optional[int]            # SOP 评分 (0-5)
    showcase: bool                   # 是否作为精选案例展示
    vector_store_id: str             # 向量存储ID
    linsight_version_id: Optional[str]  # 关联的会话版本ID
```

### 执行任务表 (LinsightExecuteTask)

```python
class LinsightExecuteTask(SQLModelSerializable, table=True):
    id: str                          # 任务ID
    session_version_id: str          # 会话版本ID
    task_name: str                   # 任务名称
    task_description: str            # 任务描述
    task_type: ExecuteTaskTypeEnum   # 任务类型
    status: ExecuteTaskStatusEnum    # 任务状态
    result: Optional[Dict]           # 执行结果
    error_message: Optional[str]     # 错误信息
```

## 核心组件

### 1. LinsightWorkflowTask（任务执行器）

负责管理整个任务的生命周期：

```python
class LinsightWorkflowTask:
    """工作流任务执行器"""
    
    async def async_run(self, session_version_id: str) -> None:
        """异步任务执行入口"""
        pass
    
    async def _execute_workflow(self, session_model: LinsightSessionVersion):
        """执行工作流的核心逻辑"""
        # 1. 更新会话状态为进行中
        # 2. 初始化 LLM 和工具
        # 3. 创建智能体
        # 4. 生成并保存任务
        # 5. 执行任务
        pass
```

### 2. LinsightStateMessageManager（状态消息管理器）

管理任务执行过程中的状态和消息：

```python
class LinsightStateMessageManager:
    """灵思状态与消息管理器"""
    
    async def push_message(self, message: MessageData) -> None:
        """推送消息到 Redis"""
        pass
    
    async def pop_message(self) -> Optional[MessageData]:
        """从 Redis 弹出消息"""
        pass
    
    async def set_session_version_info(self, session_version_model) -> None:
        """设置会话版本信息"""
        pass
    
    async def set_execution_tasks(self, tasks: List[LinsightExecuteTask]) -> None:
        """设置执行任务信息"""
        pass
```

### 3. LinsightAgent（智能代理）

基于 LangChain 的智能代理实现：

```python
class LinsightAgent(BaseModel):
    """Linsight 智能代理"""
    
    file_dir: Optional[str]          # 文件存储目录
    query: str                       # 用户问题
    llm: BaseLanguageModel           # 语言模型
    tools: list[BaseTool]            # 工具列表
    task_manager: Optional[TaskManage]  # 任务管理器
    
    async def generate_sop(self, sop: str, file_list: list[str] = None, 
                          knowledge_list: list[str] = None):
        """生成 SOP"""
        pass
    
    async def feedback_sop(self, sop: str, feedback: str, ...):
        """根据反馈优化 SOP"""
        pass
    
    async def generate_task(self, sop: str) -> list[dict]:
        """根据 SOP 生成任务列表"""
        pass
    
    async def ainvoke(self, tasks: list[dict], sop: str, ...):
        """执行任务"""
        pass
```

### 4. 消息事件类型

```python
class MessageEventType(str, Enum):
    TASK_START = "task_start"              # 任务开始
    TASK_GENERATE = "task_generate"        # 生成任务
    TASK_STATUS_UPDATE = "task_status_update"  # 任务状态更新
    USER_INPUT = "user_input"              # 用户输入
    USER_INPUT_COMPLETED = "user_input_completed"  # 用户输入完成
    TASK_EXECUTE_STEP = "task_execute_step"  # 任务执行步骤
    TASK_END = "task_end"                  # 任务结束
    ERROR_MESSAGE = "error_message"        # 错误消息
    FINAL_RESULT = "final_result"          # 最终结果
    TASK_TERMINATED = "task_terminated"    # 任务终止
```

## API 接口设计

### 文件管理接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/linsight/workbench/upload-file` | POST | 上传文件 |
| `/linsight/workbench/file-parsing-status` | POST | 获取文件解析状态 |

### 问题提交接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/linsight/workbench/submit` | POST | 提交用户问题（SSE） |

### SOP 管理接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/linsight/workbench/generate-sop` | POST | 生成/重新规划 SOP（SSE） |
| `/linsight/workbench/sop-modify` | POST | 修改 SOP |

### 执行控制接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/linsight/workbench/start-execute` | POST | 开始执行任务 |
| `/linsight/workbench/terminate` | POST | 终止任务执行 |
| `/linsight/workbench/user-input` | POST | 提交用户输入 |

### 结果查询接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/linsight/workbench/execute-status` | GET | 获取执行状态（SSE） |
| `/linsight/workbench/download-files` | POST | 下载执行结果文件 |
| `/linsight/workbench/session-versions` | GET | 获取会话版本列表 |

### 精选案例接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/linsight/sop/list` | GET | 获取精选案例列表 |
| `/linsight/sop/create` | POST | 创建精选案例 |
| `/linsight/sop/update` | PUT | 更新精选案例 |
| `/linsight/sop/delete` | DELETE | 删除精选案例 |

## 执行流程

### 1. 问题提交流程

```
用户提交问题
    ↓
创建消息会话 (MessageSession)
    ↓
创建会话版本 (LinsightSessionVersion)
    ↓
生成任务标题
    ↓
返回会话信息
```

### 2. SOP 生成流程

```
接收会话版本ID
    ↓
获取用户权限内的知识库列表
    ↓
调用 LLM 生成 SOP
    ↓
流式返回 SOP 内容
    ↓
保存 SOP 到会话版本
```

### 3. 任务执行流程

```
开始执行
    ↓
初始化执行环境
    ├── 获取 LLM 实例
    ├── 初始化工具列表
    └── 创建文件目录
    ↓
创建 LinsightAgent
    ↓
根据 SOP 生成任务树
    ↓
循环执行任务
    ├── 执行当前任务
    ├── 处理工具调用
    ├── 处理用户输入请求
    └── 更新任务状态
    ↓
生成最终结果
    ↓
清理资源
```

### 4. 人在回路机制

```
任务执行中
    ↓
需要用户输入 (NeedUserInput 事件)
    ↓
暂停执行，等待用户响应
    ↓
用户提交输入
    ↓
继续执行任务
```

## 工具集成

### 内置工具

Linsight 支持多种内置工具：

1. **代码解释器** (`bisheng_code_interpreter`)
   - 支持 Python 代码执行
   - 沙盒环境隔离
   - 自动清理资源

2. **知识库检索** (`knowledge_retriever`)
   - 支持个人知识库
   - 支持组织知识库
   - 语义检索能力

3. **文件处理工具**
   - 文件读取
   - 文件写入
   - 格式转换

### 自定义工具扩展

```python
from langchain_core.tools import BaseTool

class CustomTool(BaseTool):
    name: str = "custom_tool"
    description: str = "自定义工具描述"
    
    def _run(self, input: str) -> str:
        # 实现工具逻辑
        return result
```

## 配置项

### Linsight 配置

```yaml
linsight:
  default_temperature: 0.7          # 默认温度参数
  max_knowledge_num: 10             # 最大知识库数量
  max_file_num: 5                   # 最大文件数量
  task_timeout: 3600                # 任务超时时间（秒）
  user_input_timeout: 300           # 用户输入超时时间（秒）
```

### 邀请码配置

```yaml
linsight_invitation_code: false     # 是否启用邀请码限制
```

## 状态管理

### Redis 键设计

```
linsight_tasks:{session_version_id}:session_version_info  # 会话版本信息
linsight_tasks:{session_version_id}:messages              # 消息队列
linsight_tasks:{session_version_id}:execution_tasks:      # 执行任务信息
```

### 状态同步机制

1. **双写策略**：同时写入 Redis 和数据库
2. **缓存优先**：优先从 Redis 读取，失败时回退到数据库
3. **自动过期**：Redis 数据设置过期时间（默认 3600 秒）

## 错误处理

### 异常类型

```python
class TaskExecutionError(Exception):
    """任务执行异常"""
    pass

class UserTerminationError(Exception):
    """用户主动终止异常"""
    pass

class TaskAlreadyInProgressError(Exception):
    """任务已在进行中异常"""
    pass
```

### 错误码

| 错误码 | 描述 |
|--------|------|
| LinsightQuestionError | 问题提交错误 |
| LinsightUseUpError | 使用次数用尽 |
| LinsightModifySopError | SOP 修改错误 |
| LinsightStartTaskError | 任务启动错误 |
| LinsightSessionVersionRunningError | 会话版本正在运行 |

## 监控与日志

### 日志记录

```python
with logger.contextualize(trace_id=session_version_id):
    logger.info(f"开始执行任务: session_version_id={session_version_id}")
```

### 关键监控指标

- 任务执行时间
- 任务成功率
- LLM 调用次数
- 工具调用统计
- 用户输入等待时间

## 扩展开发

### 添加新工具

1. 实现 `BaseTool` 接口
2. 在 `ToolServices.init_linsight_tools()` 中注册
3. 更新工具描述供 LLM 理解

### 自定义 SOP 模板

1. 修改 `bisheng_langchain/linsight/prompt.py`
2. 调整 `SopPrompt` 模板内容
3. 测试生成效果

### 集成外部服务

1. 创建服务适配器
2. 封装为 LangChain Tool
3. 配置服务连接参数

## 最佳实践

### 1. SOP 编写建议

- 步骤清晰，逻辑明确
- 包含必要的条件判断
- 预留人工干预点
- 定义明确的输出格式

### 2. 工具使用建议

- 选择合适的工具组合
- 控制工具调用频率
- 处理工具调用异常
- 记录工具调用日志

### 3. 性能优化建议

- 合理设置超时时间
- 使用流式输出
- 缓存常用数据
- 异步处理耗时操作

---

## 流程图

详细的流程图和脑图已拆分到独立文件，便于维护：

- [Linsight任务执行流程图](../diagrams/linsight/task_flow.md)
- [SOP生成流程图](../diagrams/linsight/sop_flow.md)
- [Linsight执行时序图](../diagrams/linsight/sequence.md)
- [工具调用数据流图](../diagrams/linsight/tool_dataflow.md)
- [状态流转图](../diagrams/linsight/state_diagram.md)
- [Linsight模块脑图](../diagrams/linsight/mindmap.md)
