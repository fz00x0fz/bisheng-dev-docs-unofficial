# LLM集成模块详细设计文档

## 模块概述

LLM集成模块是Bisheng平台与各种大语言模型服务集成的核心组件，提供了统一的接口和服务封装，支持多种主流LLM提供商的服务接入。该模块实现了模型调用的抽象化、参数配置管理、调用日志记录和成本监控等功能。

## 模块结构

```
bisheng/llm/                # LLM模块
├── api/                   # LLM API接口
│   └── router.py          # 路由定义
├── const.py               # 常量定义
├── domain/                # 领域模型
│   ├── llm.py             # LLM领域模型
│   └── services.py        # LLM服务实现
├── models/                # 模型封装
└── schemas.py             # 数据模式定义

bisheng_langchain/          # LangChain扩展（模型适配器）
├── chat_models/           # 聊天模型适配器
│   ├── host_llm.py        # 自托管模型
│   ├── qwen.py            # 通义千问
│   ├── wenxin.py          # 文心一言
│   ├── zhipuai.py         # 智谱AI
│   ├── xunfeiai.py        # 讯飞星火
│   └── minimax.py         # MiniMax
└── embeddings/            # 嵌入模型适配器
    ├── host_embedding.py  # 自托管嵌入
    └── wenxin.py          # 文心嵌入
```

## 核心功能

### 1. 多模型提供商支持

#### 支持的提供商
- **OpenAI**: GPT系列模型
- **Anthropic**: Claude系列模型
- **阿里云**: 通义千问(Qwen)系列模型
- **智谱AI**: ChatGLM系列模型
- **百度**: 文心一言系列模型
- **讯飞**: 星火大模型

#### 适配器模式
采用适配器设计模式，为每个LLM提供商实现统一的接口：
```python
class LLMProviderAdapter(ABC):
    @abstractmethod
    async def chat_completion(self, messages, **kwargs):
        """聊天补全接口"""
        pass

    @abstractmethod
    async def embedding(self, texts, **kwargs):
        """文本嵌入接口"""
        pass

    @abstractmethod
    def get_model_info(self):
        """获取模型信息"""
        pass
```

### 2. 统一调用接口

#### 功能描述
提供统一的抽象接口，屏蔽不同提供商的具体实现细节。

#### 核心接口
```python
class LLMService:
    @staticmethod
    async def chat_completion(provider, model, messages, **kwargs):
        """统一聊天补全接口"""
        adapter = LLMAdapterFactory.get_adapter(provider)
        return await adapter.chat_completion(messages, model=model, **kwargs)

    @staticmethod
    async def embedding(provider, model, texts, **kwargs):
        """统一文本嵌入接口"""
        adapter = LLMAdapterFactory.get_adapter(provider)
        return await adapter.embedding(texts, model=model, **kwargs)
```

### 3. 模型参数配置

#### 功能描述
支持灵活的模型参数配置，适应不同场景需求。

#### 支持的参数
- **temperature**: 控制输出随机性 (0.0-2.0)
- **max_tokens**: 最大输出token数
- **top_p**: 核采样参数 (0.0-1.0)
- **frequency_penalty**: 频率惩罚 (-2.0-2.0)
- **presence_penalty**: 存在惩罚 (-2.0-2.0)
- **stop_sequences**: 停止序列列表

#### 配置示例
```yaml
llm:
  openai:
    api_key: "sk-xxx"
    base_url: "https://api.openai.com/v1"
    models:
      - name: "gpt-3.5-turbo"
        temperature: 0.7
        max_tokens: 2048
      - name: "gpt-4"
        temperature: 0.5
        max_tokens: 4096
  anthropic:
    api_key: "sk-ant-xxx"
    base_url: "https://api.anthropic.com"
    models:
      - name: "claude-3-haiku"
        temperature: 0.7
        max_tokens: 4096
```

### 4. 调用日志和监控

#### 功能描述
记录模型调用日志，监控调用性能和成本。

#### 日志内容
- 调用时间戳
- 模型提供商和模型名称
- 输入输出token数
- 调用耗时
- 错误信息
- 成本计算

#### 监控指标
- **调用成功率**: 成功调用占总调用的比例
- **平均响应时间**: 模型响应时间统计
- **成本统计**: 按模型和提供商的成本分析
- **使用量统计**: 各模型的调用频次统计

### 5. 成本控制和优化

#### 功能描述
提供成本控制和优化机制，帮助用户合理使用LLM服务。

#### 成本计算
根据各提供商的定价模型计算调用成本：
```python
class CostCalculator:
    @staticmethod
    def calculate_openai_cost(model, prompt_tokens, completion_tokens):
        """计算OpenAI调用成本"""
        pricing = {
            "gpt-3.5-turbo": {"input": 0.0015, "output": 0.002},
            "gpt-4": {"input": 0.03, "output": 0.06}
        }
        model_pricing = pricing.get(model, {"input": 0, "output": 0})
        return (prompt_tokens / 1000) * model_pricing["input"] + \
               (completion_tokens / 1000) * model_pricing["output"]
```

#### 优化策略
- **缓存机制**: 缓存相同请求的结果
- **模型选择**: 根据任务复杂度选择合适模型
- **批处理**: 批量处理多个请求
- **降级策略**: 高成本时自动降级到低成本模型

## 数据模型设计

### 模型配置表 (LLMModel)

```python
class LLMModel(BaseSQLModel):
    id: Optional[int] = Field(default=None, primary_key=True)
    provider: str = Field(max_length=50, index=True)  # 提供商
    name: str = Field(max_length=100)  # 模型名称
    description: Optional[str] = Field(default=None, max_length=500)
    config: Optional[str] = Field(default=None)  # JSON格式配置
    status: int = Field(default=1)  # 1:启用 2:禁用
    create_time: Optional[datetime] = Field(default_factory=datetime.utcnow)
    update_time: Optional[datetime] = Field(default_factory=datetime.utcnow)
```

### 调用日志表 (LLMCallLog)

```python
class LLMCallLog(BaseSQLModel):
    id: Optional[int] = Field(default=None, primary_key=True)
    trace_id: str = Field(max_length=50, index=True)  # 追踪ID
    provider: str = Field(max_length=50)
    model: str = Field(max_length=100)
    prompt_tokens: int = Field(default=0)
    completion_tokens: int = Field(default=0)
    total_tokens: int = Field(default=0)
    duration: float = Field()  # 调用耗时(毫秒)
    cost: float = Field(default=0.0)  # 调用成本(美元)
    status: int = Field(default=1)  # 1:成功 2:失败
    error_message: Optional[str] = Field(default=None, max_length=1000)
    create_time: Optional[datetime] = Field(default_factory=datetime.utcnow)
```

## 核心服务实现

### 1. LLM服务类

```python
class LLMService:
    @staticmethod
    async def chat_completion(trace_id: str, provider: str, model: str,
                            messages: List[Dict], **kwargs) -> Dict:
        """聊天补全服务"""
        start_time = time.time()

        try:
            # 获取适配器
            adapter = LLMAdapterFactory.get_adapter(provider)

            # 调用模型
            result = await adapter.chat_completion(
                messages=messages, model=model, **kwargs
            )

            # 记录日志
            await LLMService._log_call(
                trace_id=trace_id,
                provider=provider,
                model=model,
                prompt_tokens=result.get("usage", {}).get("prompt_tokens", 0),
                completion_tokens=result.get("usage", {}).get("completion_tokens", 0),
                duration=(time.time() - start_time) * 1000,
                status=1
            )

            return result

        except Exception as e:
            # 记录错误日志
            await LLMService._log_call(
                trace_id=trace_id,
                provider=provider,
                model=model,
                duration=(time.time() - start_time) * 1000,
                status=2,
                error_message=str(e)
            )
            raise e

    @staticmethod
    async def _log_call(trace_id: str, provider: str, model: str,
                       prompt_tokens: int = 0, completion_tokens: int = 0,
                       duration: float = 0, status: int = 1,
                       error_message: str = None):
        """记录调用日志"""
        total_tokens = prompt_tokens + completion_tokens
        cost = CostCalculator.calculate_cost(provider, model, prompt_tokens, completion_tokens)

        log_entry = LLMCallLog(
            trace_id=trace_id,
            provider=provider,
            model=model,
            prompt_tokens=prompt_tokens,
            completion_tokens=completion_tokens,
            total_tokens=total_tokens,
            duration=duration,
            cost=cost,
            status=status,
            error_message=error_message
        )

        await LLMDao.save_call_log(log_entry)
```

### 2. 适配器工厂

```python
class LLMAdapterFactory:
    _adapters = {}

    @classmethod
    def register_adapter(cls, provider: str, adapter_class):
        """注册适配器"""
        cls._adapters[provider] = adapter_class

    @classmethod
    def get_adapter(cls, provider: str) -> LLMProviderAdapter:
        """获取适配器实例"""
        if provider not in cls._adapters:
            raise ValueError(f"Unsupported provider: {provider}")

        adapter_class = cls._adapters[provider]
        return adapter_class()
```

### 3. 缓存机制

```python
class LLMCache:
    @staticmethod
    async def get_cached_result(cache_key: str) -> Optional[Dict]:
        """获取缓存结果"""
        redis_client = await get_redis_client()
        cached_result = await redis_client.aget(f"llm_cache:{cache_key}")
        if cached_result:
            return json.loads(cached_result)
        return None

    @staticmethod
    async def set_cached_result(cache_key: str, result: Dict, ttl: int = 3600):
        """设置缓存结果"""
        redis_client = await get_redis_client()
        await redis_client.aset(
            f"llm_cache:{cache_key}",
            json.dumps(result),
            expiration=ttl
        )
```

## API 接口设计

### 模型管理接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/llm/models` | GET | 获取可用模型列表 |
| `/llm/models/{id}` | GET | 获取模型详情 |
| `/llm/models` | POST | 创建模型配置 |
| `/llm/models/{id}` | PUT | 更新模型配置 |
| `/llm/models/{id}` | DELETE | 删除模型配置 |

### 调用接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/llm/chat/completion` | POST | 聊天补全 |
| `/llm/embedding` | POST | 文本嵌入 |
| `/llm/stream/chat/completion` | POST | 流式聊天补全 |

### 监控接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/llm/stats/usage` | GET | 获取使用统计 |
| `/llm/stats/cost` | GET | 获取成本统计 |
| `/llm/logs` | GET | 获取调用日志 |
| `/llm/logs/{id}` | GET | 获取日志详情 |

## 适配器实现示例

### OpenAI 适配器

```python
class OpenAIAdapter(LLMProviderAdapter):
    def __init__(self):
        self.client = AsyncOpenAI(
            api_key=settings.llm.openai.api_key,
            base_url=settings.llm.openai.base_url
        )

    async def chat_completion(self, messages, **kwargs):
        """OpenAI聊天补全"""
        model = kwargs.pop("model", "gpt-3.5-turbo")

        response = await self.client.chat.completions.create(
            model=model,
            messages=messages,
            **kwargs
        )

        return {
            "id": response.id,
            "choices": [
                {
                    "message": choice.message.dict(),
                    "finish_reason": choice.finish_reason
                }
                for choice in response.choices
            ],
            "usage": response.usage.dict() if response.usage else {},
            "model": response.model
        }

    async def embedding(self, texts, **kwargs):
        """OpenAI文本嵌入"""
        model = kwargs.pop("model", "text-embedding-ada-002")

        response = await self.client.embeddings.create(
            model=model,
            input=texts,
            **kwargs
        )

        return {
            "data": [
                {
                    "embedding": emb.embedding,
                    "index": emb.index
                }
                for emb in response.data
            ],
            "model": response.model,
            "usage": response.usage.dict() if response.usage else {}
        }
```

## 性能优化

### 1. 连接池管理
- 复用HTTP连接减少建立开销
- 控制并发连接数避免资源耗尽
- 自动重试机制处理网络波动

### 2. 批处理优化
- 支持批量文本嵌入处理
- 合并小请求减少调用次数
- 异步并发处理提高吞吐量

### 3. 缓存策略
- 热点数据缓存减少重复调用
- 缓存失效策略保证数据新鲜度
- 分布式缓存支持集群部署

## 安全机制

### 1. 认证安全
- API密钥加密存储
- 请求签名验证
- 访问频率限制

### 2. 数据安全
- 敏感信息脱敏处理
- 传输过程加密
- 日志敏感信息过滤

### 3. 调用安全
- 参数合法性校验
- 输出内容安全检查
- 恶意使用防护

## 配置项

### 相关配置
```yaml
# LLM配置
llm:
  openai:
    api_key: "${OPENAI_API_KEY}"
    base_url: "https://api.openai.com/v1"
    timeout: 30  # 请求超时时间(秒)
    max_retries: 3  # 最大重试次数

  anthropic:
    api_key: "${ANTHROPIC_API_KEY}"
    base_url: "https://api.anthropic.com"
    timeout: 30
    max_retries: 3

  cache:
    enabled: true  # 是否启用缓存
    ttl: 3600      # 缓存过期时间(秒)
    max_size: 10000 # 最大缓存条目数
```

## 扩展性设计

### 1. 插件化适配器
支持自定义LLM提供商适配器：
1. 实现LLMProviderAdapter接口
2. 注册到适配器工厂
3. 配置相关信息

### 2. 动态模型发现
- 支持运行时动态加载模型
- 自动发现新模型版本
- 配置热更新支持

### 3. 多租户支持
- 按用户/组织隔离配置
- 独立的成本统计
- 细粒度的权限控制

## 监控与运维

### 1. 性能监控
- 调用延迟分布统计
- 成功率趋势监控
- 资源使用情况跟踪

### 2. 成本监控
- 实时成本统计
- 预算预警机制
- 成本分析报表

### 3. 错误监控
- 错误类型分类统计
- 异常调用告警
- 错误根因分析

## 测试策略

### 1. 单元测试
- 适配器功能测试
- 服务逻辑测试
- 缓存机制测试

### 2. 集成测试
- 端到端调用测试
- 多提供商兼容性测试
- 错误处理测试

### 3. 性能测试
- 高并发调用测试
- 响应时间压测
- 成本计算准确性测试

---

## 流程图

详细的流程图和脑图已拆分到独立文件，便于维护：

- [LLM调用流程图](../diagrams/llm/call_flow.md)
- [流式输出流程图](../diagrams/llm/stream_flow.md)
- [LLM调用时序图](../diagrams/llm/sequence.md)
- [多模型调用数据流图](../diagrams/llm/multi_model_dataflow.md)
- [LLM集成模块脑图](../diagrams/llm/mindmap.md)