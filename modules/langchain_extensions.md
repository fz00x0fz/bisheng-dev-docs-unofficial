# Bisheng LangChain 扩展库文档

## 概述

`bisheng_langchain` 是 Bisheng 平台基于 LangChain 框架开发的扩展库，提供了自定义的聊天模型、嵌入模型、文档加载器、检索器等组件，以支持更多的模型提供商和特定业务场景。

## 模块结构

```
bisheng_langchain/
├── agents/              # 自定义代理
├── autogen_role/        # AutoGen 角色集成
├── chains/              # 自定义链
├── chat_models/         # 聊天模型适配器
├── document_loaders/    # 文档加载器
├── embeddings/          # 嵌入模型适配器
├── gpts/                # GPTs 助手实现
├── input_output/        # 输入输出处理
├── linsight/            # Linsight Agent 实现
├── memory/              # 记忆模块
├── rag/                 # RAG 实现
├── retrievers/          # 检索器
├── sql/                 # SQL 相关工具
├── utils/               # 工具函数
├── vectorstores/        # 向量存储
└── text_splitter.py     # 文本分割器
```

---

## 聊天模型 (chat_models)

提供多种国内外 LLM 提供商的适配器。

### 支持的模型

| 模型 | 文件 | 说明 |
|------|------|------|
| Host LLM | `host_llm.py` | 自托管模型代理 |
| Proxy LLM | `proxy_llm.py` | 代理模式 LLM |
| 通义千问 | `qwen.py` | 阿里云通义千问 |
| 文心一言 | `wenxin.py` | 百度文心一言 |
| 智谱AI | `zhipuai.py` | 智谱 GLM 系列 |
| 讯飞星火 | `xunfeiai.py` | 讯飞星火认知大模型 |
| MiniMax | `minimax.py` | MiniMax 模型 |
| 商汤 | `sensetime.py` | 商汤日日新 |

### BaseHostChatLLM

自托管模型的基础类，支持通过 HTTP API 调用本地部署的模型。

```python
from bisheng_langchain.chat_models import BaseHostChatLLM

llm = BaseHostChatLLM(
    host_base_url="http://localhost:8000",
    model="your-model-name",
    temperature=0.7,
    max_tokens=2048,
    request_timeout=60
)

response = llm.invoke("你好，请介绍一下自己")
```

### 配置参数

```python
class BaseHostChatLLM(BaseChatModel):
    model: str                          # 模型名称
    host_base_url: str                  # 服务地址
    temperature: float = 0.7            # 温度参数
    max_tokens: int = 2048              # 最大token数
    request_timeout: float = 60         # 请求超时
    max_retries: int = 3                # 最大重试次数
    streaming: bool = False             # 是否流式输出
```

---

## 嵌入模型 (embeddings)

提供多种嵌入模型的适配器。

### 支持的模型

| 模型 | 文件 | 说明 |
|------|------|------|
| Host Embedding | `host_embedding.py` | 自托管嵌入模型 |
| HuggingFace GTE | `huggingfacegte.py` | GTE 系列模型 |
| HuggingFace Multilingual | `huggingfacemultilingual.py` | 多语言模型 |
| 文心 Embedding | `wenxin.py` | 百度文心嵌入 |

### HostEmbeddings

自托管嵌入模型的适配器。

```python
from bisheng_langchain.embeddings import HostEmbeddings

embeddings = HostEmbeddings(
    host_base_url="http://localhost:8000",
    model="embedding-model",
    embedding_ctx_length=6144
)

# 单文本嵌入
vector = embeddings.embed_query("这是一段测试文本")

# 批量嵌入
vectors = embeddings.embed_documents([
    "文本1",
    "文本2",
    "文本3"
])
```

### 配置参数

```python
class HostEmbeddings(BaseModel, Embeddings):
    model: str = 'embedding-host'       # 模型名称
    host_base_url: str                  # 服务地址
    embedding_ctx_length: int = 6144    # 最大上下文长度
    max_retries: int = 6                # 最大重试次数
    request_timeout: float = 200        # 请求超时
```

---

## 文档加载器 (document_loaders)

提供多种文档格式的加载器。

### 支持的加载器

| 加载器 | 文件 | 说明 |
|------|------|------|
| ElemPDFLoader | `elem_pdf.py` | PDF 文档加载 |
| ElemImageLoader | `elem_image.py` | 图像文档加载 |
| ElemHTMLLoader | `elem_html.py` | HTML 文档加载 |
| ElemUnstructuredLoader | `elem_unstrcutured_loader.py` | 通用非结构化加载 |
| CustomKVLoader | `custom_kv.py` | 自定义键值对加载 |
| UniversalKVLoader | `universal_kv.py` | 通用键值对加载 |

### ElemPDFLoader

高精度 PDF 文档加载器，支持表格识别、布局分析等。

```python
from bisheng_langchain.document_loaders import ElemPDFLoader

loader = ElemPDFLoader(
    file_path="document.pdf",
    unstructured_api_url="http://localhost:10001",
    start_page=0,
    end_page=None,
    extract_images=True
)

documents = loader.load()
```

---

## GPTs 助手 (gpts)

提供 GPTs 风格的助手实现。

### ConfigurableAssistant

可配置的助手类，支持多种 Agent 执行器类型。

```python
from bisheng_langchain.gpts.assistant import ConfigurableAssistant
from bisheng_langchain.gpts.load_tools import load_tools

# 加载工具
tools = load_tools({"web_search": {}, "calculator": {}})

# 创建助手
assistant = ConfigurableAssistant(
    agent_executor_type="get_react_agent_executor",
    tools=tools,
    llm=llm,
    assistant_message="你是一个有帮助的助手",
    recursion_limit=50
)

# 执行对话
result = assistant.invoke({"messages": [HumanMessage(content="今天天气怎么样？")]})
```

### BishengAssistant

基于 YAML 配置的助手类。

```python
from bisheng_langchain.gpts.assistant import BishengAssistant

# 从配置文件创建助手
assistant = BishengAssistant(yaml_path="assistant_config.yaml")

# 执行对话
result = assistant.run("请帮我查询一下最新的新闻")
```

### 配置文件示例

```yaml
assistant:
  prompt_type: "default_prompt"
  llm:
    type: "ChatOpenAI"
    model: "gpt-4"
    temperature: 0.7
    openai_proxy: null
  tools:
    - type: "web_search"
      api_key: "your-api-key"
    - type: "calculator"
  agent_executor:
    type: "get_react_agent_executor"
    recursion_limit: 50
```

### 工具加载

```python
from bisheng_langchain.gpts.load_tools import load_tools, get_all_tool_names

# 获取所有可用工具名称
available_tools = get_all_tool_names()

# 加载指定工具
tools = load_tools({
    "web_search": {"api_key": "xxx"},
    "calculator": {},
    "code_interpreter": {"sandbox_url": "http://localhost:8080"}
})
```

---

## Linsight Agent (linsight)

Linsight 专家代理的核心实现。

### LinsightAgent

```python
from bisheng_langchain.linsight.agent import LinsightAgent

agent = LinsightAgent(
    query="帮我分析这份报告",
    llm=llm,
    tools=tools,
    file_dir="/tmp/workspace"
)

# 生成 SOP
async for chunk in agent.generate_sop(sop="", file_list=["report.pdf"]):
    print(chunk.content)

# 生成任务
tasks = await agent.generate_task(sop=sop_content)

# 执行任务
async for event in agent.ainvoke(tasks, sop=sop_content):
    print(event)
```

### 事件类型

```python
from bisheng_langchain.linsight.event import (
    BaseEvent,
    TaskStart,
    TaskEnd,
    ExecStep,
    NeedUserInput,
    GenerateSubTask
)

# 任务开始事件
class TaskStart(BaseEvent):
    task_id: str
    task_name: str

# 任务结束事件
class TaskEnd(BaseEvent):
    task_id: str
    status: TaskStatus
    result: Any

# 执行步骤事件
class ExecStep(BaseEvent):
    step_id: str
    step_name: str
    tool_name: str
    tool_input: dict
    tool_output: Any

# 需要用户输入事件
class NeedUserInput(BaseEvent):
    prompt: str
    input_type: str
```

### 任务管理

```python
from bisheng_langchain.linsight.manage import TaskManage

# 任务管理器
task_manager = TaskManage(
    tasks=tasks,
    llm=llm,
    tools=tools
)

# 执行任务
async for event in task_manager.execute():
    handle_event(event)
```

---

## AutoGen 角色 (autogen_role)

集成 AutoGen 多代理框架。

### 支持的角色

| 角色 | 文件 | 说明 |
|------|------|------|
| Assistant | `assistant.py` | 助手角色 |
| User | `user.py` | 用户代理角色 |
| Custom | `custom.py` | 自定义角色 |
| GroupChatManager | `groupchat_manager.py` | 群聊管理器 |

```python
from bisheng_langchain.autogen_role import Assistant, User, GroupChatManager

# 创建助手
assistant = Assistant(
    name="assistant",
    llm_config={"model": "gpt-4"}
)

# 创建用户代理
user = User(name="user")

# 创建群聊管理器
manager = GroupChatManager(
    agents=[assistant, user],
    max_round=10
)
```

---

## 自定义链 (chains)

### LoaderOutputChain

文档加载输出链。

```python
from bisheng_langchain.chains import LoaderOutputChain

chain = LoaderOutputChain(
    loader=pdf_loader,
    output_key="documents"
)

result = chain.invoke({})
```

### TransformChain

数据转换链。

```python
from bisheng_langchain.chains import TransformChain

chain = TransformChain(
    input_variables=["text"],
    output_variables=["processed_text"],
    transform_func=lambda x: x["text"].upper()
)
```

---

## 文本分割器 (text_splitter)

提供多种文本分割策略。

```python
from bisheng_langchain.text_splitter import (
    RecursiveCharacterTextSplitter,
    ChineseTextSplitter
)

# 递归字符分割
splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50,
    separators=["\n\n", "\n", "。", "！", "？"]
)

chunks = splitter.split_text(long_text)

# 中文文本分割
chinese_splitter = ChineseTextSplitter(
    chunk_size=500,
    chunk_overlap=50
)
```

---

## 输入输出处理 (input_output)

### Input

处理用户输入。

```python
from bisheng_langchain.input_output import Input

input_handler = Input(
    input_type="text",
    required=True,
    placeholder="请输入问题"
)
```

### Output

处理输出结果。

```python
from bisheng_langchain.input_output import Output

output_handler = Output(
    output_type="text",
    stream=True
)
```

---

## 扩展开发指南

### 添加新的聊天模型

1. 创建新文件 `chat_models/your_model.py`
2. 继承 `BaseChatModel` 类
3. 实现必要的方法

```python
from langchain.chat_models.base import BaseChatModel
from langchain.schema import ChatResult

class YourChatModel(BaseChatModel):
    model: str
    api_key: str
    
    def _generate(self, messages, stop=None, run_manager=None, **kwargs):
        # 实现生成逻辑
        response = self._call_api(messages)
        return ChatResult(generations=[...])
    
    async def _agenerate(self, messages, stop=None, run_manager=None, **kwargs):
        # 实现异步生成逻辑
        pass
    
    @property
    def _llm_type(self):
        return "your-model"
```

### 添加新的嵌入模型

```python
from langchain.embeddings.base import Embeddings

class YourEmbeddings(Embeddings):
    model: str
    api_key: str
    
    def embed_documents(self, texts: List[str]) -> List[List[float]]:
        # 实现批量嵌入
        pass
    
    def embed_query(self, text: str) -> List[float]:
        # 实现单文本嵌入
        pass
```

### 添加新的工具

```python
from langchain.tools import BaseTool

class YourTool(BaseTool):
    name: str = "your_tool"
    description: str = "工具描述"
    
    def _run(self, query: str) -> str:
        # 实现工具逻辑
        return result
    
    async def _arun(self, query: str) -> str:
        # 实现异步逻辑
        pass
```

---

## 最佳实践

### 1. 模型选择

- 对于中文场景，优先使用国内模型（通义千问、文心一言等）
- 对于多语言场景，使用支持多语言的模型
- 对于私有部署，使用 Host LLM/Embedding

### 2. 错误处理

```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=4, max=10)
)
def call_with_retry():
    return llm.invoke(messages)
```

### 3. 流式输出

```python
# 启用流式输出
llm = BaseHostChatLLM(streaming=True)

for chunk in llm.stream("请讲一个故事"):
    print(chunk.content, end="", flush=True)
```

### 4. 异步调用

```python
import asyncio

async def async_call():
    response = await llm.ainvoke("你好")
    return response

result = asyncio.run(async_call())
```
