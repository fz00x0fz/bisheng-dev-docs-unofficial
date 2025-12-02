# LLM调用流程图

展示LLM调用的完整流程。

## 代码入口

| 类/函数 | 文件路径 | 说明 |
|---------|----------|------|
| `LLMService` | `src/backend/bisheng/llm/domain/services/llm.py:31` | LLM服务核心类 |
| `LLMService.get_bisheng_llm()` | `src/backend/bisheng/llm/domain/services/llm.py` | 获取LLM实例 |
| `LLMService.get_bisheng_embedding()` | `src/backend/bisheng/llm/domain/services/llm.py` | 获取Embedding实例 |
| `BishengLLM` | `src/backend/bisheng/llm/domain/llm/` | Bisheng LLM封装 |
| `BishengEmbedding` | `src/backend/bisheng/llm/domain/llm/` | Bisheng Embedding封装 |
| `LLMDao` | `src/backend/bisheng/llm/models/` | LLM数据访问 |
| `llm/api/router.py` | `src/backend/bisheng/llm/api/router.py` | LLM API路由 |

```mermaid
flowchart TD
    A[业务层发起LLM调用] --> B[LLM服务接收请求]
    B --> C[参数验证]
    C -->|失败| D[返回参数错误]
    C -->|成功| E{检查缓存}
    E -->|命中| F[返回缓存结果]
    E -->|未命中| G[获取模型配置]
    G --> H{选择适配器}
    H -->|OpenAI| I[OpenAI适配器]
    H -->|Anthropic| J[Anthropic适配器]
    H -->|Qwen| K[Qwen适配器]
    H -->|其他| L[其他适配器]
    I --> M[构建API请求]
    J --> M
    K --> M
    L --> M
    M --> N[发送HTTP请求]
    N --> O{请求成功?}
    O -->|否| P{重试次数<最大值?}
    P -->|是| Q[等待后重试]
    Q --> N
    P -->|否| R[返回调用失败]
    O -->|是| S[解析响应]
    S --> T[计算Token用量]
    T --> U[计算调用成本]
    U --> V[记录调用日志]
    V --> W[更新缓存]
    W --> X[返回结果]
```

## 调用说明

### 支持的提供商

| 提供商 | 模型 |
|--------|------|
| OpenAI | GPT-3.5, GPT-4 |
| Anthropic | Claude 3 |
| 阿里云 | 通义千问 |
| 智谱AI | ChatGLM |
| 百度 | 文心一言 |
| 讯飞 | 星火大模型 |
| 自托管 | Host LLM |

### 调用参数

| 参数 | 说明 |
|------|------|
| temperature | 输出随机性 (0-2) |
| max_tokens | 最大输出token |
| top_p | 核采样参数 |
| frequency_penalty | 频率惩罚 |
| presence_penalty | 存在惩罚 |

### 重试机制

- 默认最大重试次数: 3次
- 重试间隔: 指数退避
- 可配置超时时间
