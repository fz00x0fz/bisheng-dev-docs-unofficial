# 工作流节点类型详细文档

## 概述

本文档详细描述 Bisheng 工作流中所有节点类型的功能、配置参数和使用方法。所有节点都基于 `BaseNode` 抽象类实现。

## 节点类型枚举

```python
class NodeType(Enum):
    START = "start"              # 开始节点
    END = "end"                  # 结束节点
    INPUT = "input"              # 输入节点
    AGENT = "agent"              # 代理节点
    CODE = "code"                # 代码节点
    CONDITION = "condition"      # 条件判断节点
    LLM = "llm"                  # LLM调用节点
    OUTPUT = "output"            # 输出节点
    QA_RETRIEVER = "qa_retriever"  # QA检索节点
    RAG = "rag"                  # RAG节点
    REPORT = "report"            # 报告生成节点
    TOOL = "tool"                # 工具节点
    KNOWLEDGE_RETRIEVER = "knowledge_retriever"  # 知识检索节点
    NOTE = "note"                # 备注节点（仅显示注释）
```

## 节点基类 (BaseNode)

所有节点都继承自 `BaseNode` 抽象类：

**代码位置**: `workflow/nodes/base.py`

```python
class BaseNode(ABC):
    def __init__(self, node_data, workflow_id, user_id, 
                 graph_state, target_edges, max_steps, callback):
        self.id = node_data.id           # 节点唯一ID
        self.type = node_data.type       # 节点类型
        self.name = node_data.name       # 节点名称
        self.description = node_data.description
        self.node_params = {}            # 节点参数
        self.other_node_variable = {}    # 其他节点变量
        self.graph_state = graph_state   # 全局状态管理
        self.callback_manager = callback # 回调管理器
        self.max_steps = max_steps       # 最大执行次数
    
    @abstractmethod
    def _run(self, unique_id: str) -> Dict[str, Any]:
        """节点执行逻辑，子类必须实现"""
        pass
    
    def parse_log(self, unique_id: str, result: dict) -> Any:
        """返回节点运行日志"""
        return []
    
    def route_node(self, state: dict) -> str:
        """条件节点的路由逻辑"""
        raise NotImplementedError
    
    def get_other_node_variable(self, variable_key: str) -> Any:
        """从全局变量中获取其他节点的变量值"""
        pass
```

## 节点参数模型

```python
class NodeParams(BaseModel):
    key: str                   # 变量的key
    label: str                 # 变量描述文本
    value: Any                 # 变量的值
    type: str                  # 变量类型
    required: bool             # 是否必填
    options: Any               # 变量的选项
    placeholder: str           # 占位提示文本
```

---

## 1. 开始节点 (Start)

工作流的入口节点，定义流程的起始点和初始参数。

**代码位置**: `workflow/nodes/start/`

### 功能
- 定义工作流入口
- 设置初始变量
- 配置全局参数

### 配置参数
无特殊配置，作为流程起点。

---

## 2. 结束节点 (End)

工作流的终止节点，标记流程结束。

**代码位置**: `workflow/nodes/end/`

### 功能
- 标记流程结束
- 汇总输出结果
- 清理资源

---

## 3. 输入节点 (Input)

接收用户输入的节点，支持多种输入类型。

**代码位置**: `workflow/nodes/input/`

### 功能
- 文本输入
- 文件上传
- 表单数据收集

### 配置参数
```python
{
    "input_type": "text",      # 输入类型: text, file, form
    "required": True,          # 是否必填
    "placeholder": "请输入...", # 占位提示
    "default_value": ""        # 默认值
}
```

---

## 4. 输出节点 (Output)

向用户展示结果的节点。

**代码位置**: `workflow/nodes/output/`

### 功能
- 文本输出
- 流式输出
- 结构化数据展示

### 配置参数
```python
{
    "output_type": "text",     # 输出类型
    "stream": True             # 是否流式输出
}
```

---

## 5. LLM节点 (LLM)

调用大语言模型进行文本生成。

**代码位置**: `workflow/nodes/llm/llm.py`

### 功能
- 单次调用和批量调用
- 系统提示词和用户提示词配置
- 支持变量模板替换
- 支持图像输入（多模态）

### 配置参数
```python
{
    "model_id": "模型ID",
    "temperature": 0.3,
    "system_prompt": "系统提示词，支持 {{node_id.variable}} 模板",
    "user_prompt": "用户提示词，支持 {{node_id.variable}} 模板",
    "output_user": False,      # 是否输出给用户
    "image_prompt": []         # 图像变量列表（多模态）
}
```

### 执行模式
- **single**: 单次调用
- **batch**: 批量调用（遍历变量列表）

### 使用示例
```python
# 单次调用配置
{
    "tab": {"value": "single"},
    "model_id": "gpt-4",
    "temperature": 0.7,
    "system_prompt": "你是一个专业的助手",
    "user_prompt": "请分析以下内容：{{input_node.text}}"
}

# 批量调用配置
{
    "tab": {"value": "batch"},
    "batch_variable": ["data_node.items"],
    "output": [{"key": "results"}]
}
```

### 日志输出
```python
[
    {"key": "system_prompt", "value": "实际系统提示词", "type": "params"},
    {"key": "user_prompt", "value": "实际用户提示词", "type": "params"},
    {"key": "思考内容", "value": "推理过程", "type": "params"},
    {"key": "node_id.output", "value": "输出结果", "type": "variable"}
]
```

---

## 6. 代理节点 (Agent)

基于 LangChain 的智能代理节点，支持工具调用和知识库检索。

**代码位置**: `workflow/nodes/agent/agent.py`

### 功能
- ReAct 推理模式
- Function Call 模式
- 工具调用
- 知识库检索
- SQL Agent（数据库查询）

### 配置参数
```python
{
    "model_id": "模型ID",
    "temperature": 0.3,
    "system_prompt": "系统提示词",
    "user_prompt": "用户提示词",
    "tool_list": [             # 工具列表
        {"key": "tool_id_1"},
        {"key": "tool_id_2"}
    ],
    "knowledge_id": {          # 知识库配置
        "type": "knowledge",   # knowledge: 知识库, file: 临时文件
        "value": [
            {"key": "knowledge_id_1"}
        ]
    },
    "chat_history_flag": {     # 聊天历史
        "value": 5             # 0:不使用, >0:使用N条历史
    },
    "output_user": False,      # 是否输出给用户
    "sql_agent": {             # SQL Agent配置（可选）
        "open": False,
        "database_engine": "mysql",
        "db_address": "localhost:3306",
        "db_username": "root",
        "db_password": "password",
        "db_name": "database"
    }
}
```

### 支持的数据库类型
- MySQL
- PostgreSQL
- DB2
- GaussDB
- Oracle

### Agent执行类型
- **ReAct**: 使用 ConfigurableAssistant 实现
- **Function Call**: 使用 LangGraph create_react_agent 实现

### 日志输出
```python
[
    {"key": "system_prompt", "value": "系统提示词", "type": "params"},
    {"key": "user_prompt", "value": "用户提示词", "type": "params"},
    {"key": "tool_invoke", "value": [...], "type": "tool"},
    {"key": "node_id.output", "value": "输出结果", "type": "variable"}
]
```

---

## 7. 代码节点 (Code)

执行自定义 Python 代码。

**代码位置**: `workflow/nodes/code/code.py`

### 功能
- 执行自定义 Python 代码
- 支持输入输出变量映射
- 安全的代码解析和执行

### 配置参数
```python
{
    "code_input": [            # 输入变量
        {
            "key": "var1",
            "type": "ref",     # ref: 引用其他节点, value: 固定值
            "value": "node_id.output"
        },
        {
            "key": "var2",
            "type": "value",
            "value": "固定值"
        }
    ],
    "code": """
def main(var1, var2):
    # 自定义处理逻辑
    result = process(var1, var2)
    return {'output': result}
""",
    "code_output": [           # 输出变量
        {"key": "output"}
    ]
}
```

### 注意事项
- `main` 函数是入口函数，必须定义
- 返回值必须是字典类型
- 输出 key 必须在返回字典中存在
- 代码在安全沙盒中执行

### 日志输出
```python
[
    {"key": "code_input", "value": {...}, "type": "params"},
    {"key": "code_output", "value": {...}, "type": "params"}
]
```

---

## 8. 条件判断节点 (Condition)

根据条件表达式进行分支路由。

**代码位置**: `workflow/nodes/condition/condition.py`

### 功能
- 多条件分支判断
- 支持多种比较运算符
- 支持逻辑组合（AND/OR）
- 默认分支（else）

### 配置参数
```python
{
    "condition": [
        {
            "id": "case_1",        # 分支ID，对应 sourceHandle
            "cases": [
                {
                    "variable": "node_id.output",
                    "operator": "==",
                    "value": "expected_value"
                },
                {
                    "variable": "node_id.count",
                    "operator": ">",
                    "value": 10
                }
            ],
            "logical_operator": "and"  # and / or
        },
        {
            "id": "case_2",
            "cases": [...],
            "logical_operator": "or"
        }
    ]
}
```

### 支持的运算符
| 运算符 | 说明 |
|--------|------|
| `==` | 等于 |
| `!=` | 不等于 |
| `>` | 大于 |
| `>=` | 大于等于 |
| `<` | 小于 |
| `<=` | 小于等于 |
| `contains` | 包含 |
| `not_contains` | 不包含 |
| `starts_with` | 以...开头 |
| `ends_with` | 以...结尾 |
| `is_empty` | 为空 |
| `is_not_empty` | 不为空 |

### 路由逻辑
1. 按顺序评估每个条件分支
2. 第一个满足条件的分支被选中
3. 如果没有满足的条件，使用默认分支（`right_handle`）

---

## 9. 知识检索节点 (Knowledge Retriever)

从知识库中检索相关内容。

**代码位置**: `workflow/nodes/knowledge_retriever/knowledge_retriever.py`

### 功能
- 语义检索
- 支持多知识库
- 支持 Rerank 重排序
- 返回结构化检索结果

### 配置参数
```python
{
    "knowledge_id": [          # 知识库ID列表
        {"key": "1", "label": "知识库1"},
        {"key": "2", "label": "知识库2"}
    ],
    "user_question": "{{input_node.question}}",  # 检索问题
    "top_k": 5,                # 返回数量
    "score_threshold": 0.5,    # 相似度阈值
    "rerank_model_id": null,   # Rerank模型ID（可选）
    "retrieved_result": [      # 输出变量
        {"key": "result"}
    ]
}
```

### 返回结果结构
```python
[
    {
        "text": "检索到的文本内容",
        "metadata": {
            "chunk_index": 0,
            "knowledge_id": 1,
            "document_id": "doc_id",
            "document_name": "文档名称",
            "upload_time": "2024-01-01 00:00:00",
            "update_time": "2024-01-02 00:00:00",
            "uploader": "上传者",
            "updater": "更新者",
            "user_metadata": {}
        }
    }
]
```

---

## 10. RAG节点 (RAG)

检索增强生成节点，结合知识检索和 LLM 生成。

**代码位置**: `workflow/nodes/rag/rag.py`

### 功能
- 知识库检索
- 上下文构建
- LLM 生成回答
- 支持多轮对话

### 配置参数
```python
{
    "knowledge_id": [],        # 知识库ID列表
    "model_id": "模型ID",
    "user_question": "{{input_node.question}}",
    "system_prompt": "基于以下内容回答问题：\n{context}",
    "top_k": 5,
    "score_threshold": 0.5,
    "rerank_model_id": null
}
```

### 执行流程
1. 根据用户问题检索相关文档
2. 构建上下文（将检索结果拼接）
3. 调用 LLM 生成回答
4. 返回答案和引用来源

---

## 11. QA检索节点 (QA Retriever)

专门用于问答对检索的节点。

**代码位置**: `workflow/nodes/qa_retriever/`

### 功能
- QA对检索
- 精确匹配
- 语义匹配

---

## 12. 报告生成节点 (Report)

生成 Word 格式报告文档。

**代码位置**: `workflow/nodes/report/`

### 功能
- 模板渲染
- 动态内容填充
- 文件生成和下载

### 配置参数
```python
{
    "template_id": "模板文件ID",
    "variables": {             # 模板变量映射
        "title": "{{node_id.title}}",
        "content": "{{node_id.content}}",
        "date": "{{node_id.date}}"
    }
}
```

### 模板语法
使用 Jinja2 模板语法：
- `{{ variable }}`: 变量替换
- `{% for item in list %}`: 循环
- `{% if condition %}`: 条件判断

---

## 13. 工具节点 (Tool)

调用外部工具或 API。

**代码位置**: `workflow/nodes/tool/`

### 功能
- HTTP API 调用
- 自定义工具执行
- 参数映射

### 配置参数
```python
{
    "tool_key": "tool_unique_id",
    "parameters": {
        "param1": "{{node_id.value1}}",
        "param2": "固定值"
    }
}
```

---

## 14. 备注节点 (Note)

仅用于显示注释，不参与实际执行。

### 功能
- 流程说明
- 文档注释
- 不影响执行流程

---

## 变量引用语法

节点之间通过变量引用传递数据：

### 语法格式
```
{{node_id.variable_key}}
```

### 示例
```python
# 引用输入节点的文本
"{{input_node.text}}"

# 引用LLM节点的输出
"{{llm_node.output}}"

# 引用代码节点的结果
"{{code_node.result}}"
```

### 变量解析
使用 `PromptTemplateParser` 类进行变量解析：
```python
from bisheng.workflow.nodes.prompt_template import PromptTemplateParser

template = PromptTemplateParser(template="Hello {{name}}")
variables = template.extract()  # ['name']
result = template.format({"name": "World"})  # "Hello World"
```

---

## 自定义节点开发

### 步骤1: 创建节点类
```python
from bisheng.workflow.nodes.base import BaseNode

class CustomNode(BaseNode):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        # 初始化节点参数
        self._custom_param = self.node_params.get('custom_param')
    
    def _run(self, unique_id: str):
        # 实现节点逻辑
        result = self._process()
        return {'output': result}
    
    def parse_log(self, unique_id: str, result: dict):
        return [[
            {"key": "output", "value": result['output'], "type": "variable"}
        ]]
```

### 步骤2: 注册节点类型
在 `NodeType` 枚举中添加新类型：
```python
class NodeType(Enum):
    CUSTOM = "custom"
```

### 步骤3: 注册节点工厂
在节点管理器中注册：
```python
# workflow/nodes/node_manage.py
NODE_CLASSES = {
    'custom': CustomNode,
}
```
