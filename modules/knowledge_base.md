# 知识库模块详细设计文档

## 模块概述

知识库模块是 Bisheng 平台的重要组成部分，负责文档管理、内容解析、向量索引构建和语义检索等功能。该模块基于 Milvus 向量数据库和 Elasticsearch 搜索引擎，实现了高效的文档处理和检索能力。

## 模块结构

```
bisheng/knowledge/          # 知识库模块
├── api/                   # 知识库API接口
│   └── router.py          # 路由定义（qa_router, knowledge_router）
├── domain/                # 领域模型
│   ├── schemas/           # 数据模式
│   └── services/          # 领域服务
│       ├── knowledge_service.py      # 知识库服务
│       └── knowledge_file_service.py # 文件服务
├── rag/                   # RAG相关实现
│   ├── retriever.py       # 检索器
│   └── rerank.py          # 重排序
└── utils/                 # 工具类

bisheng/database/models/   # 数据模型
├── knowledge.py           # 知识库模型
└── knowledge_file.py      # 知识库文件模型

bisheng_langchain/         # 文档处理扩展
├── document_loaders/      # 文档加载器
│   ├── elem_pdf.py        # PDF加载器
│   ├── elem_image.py      # 图像加载器
│   └── elem_html.py       # HTML加载器
└── text_splitter.py       # 文本分割器
```

## 核心功能

### 1. 文档管理

#### 功能描述
支持多种格式文档的上传、存储、管理和删除。

#### 支持的文档格式
- 文本文件：txt, md
- 办公文档：doc, docx, xls, xlsx, ppt, pptx
- PDF文档：pdf
- 图像文件：png, jpg, jpeg, bmp, tiff
- 结构化数据：csv

#### 文档存储
使用 MinIO 对象存储服务进行文档存储，支持：
- 公开访问的持久化文件存储
- 临时文件存储（带过期时间）
- 文件版本管理

### 2. 文档解析

#### 功能描述
将各种格式的文档解析为结构化文本内容，提取有用信息。

#### 解析流程
1. 文档格式识别
2. 内容提取
3. 布局分析
4. 表格识别
5. 图像文字识别（OCR）
6. 结构化输出

#### 核心技术
- **高精度OCR**: 基于深度学习的印刷文本、手写文本和罕见字符识别
- **表格识别**: 专业的表格结构识别和内容提取
- **布局分析**: 文档页面布局分析和内容区域划分
- **印章识别**: 专业印章检测和识别

### 3. 向量索引

#### 功能描述
将文档内容转换为向量表示，并构建向量索引以支持语义检索。

#### 向量化流程
1. 文本预处理
2. 文本分块（Chunking）
3. 向量编码（Embedding）
4. 向量存储
5. 索引构建

#### 技术实现
- **向量数据库**: Milvus v2.5.10
- **向量编码器**: 支持多种 Embedding 模型
- **相似度计算**: 余弦相似度、欧氏距离等

### 4. 语义检索

#### 功能描述
基于向量相似度的语义检索功能，支持自然语言查询。

#### 检索流程
1. 查询文本向量化
2. 向量相似度计算
3. 相似文档召回
4. 结果排序和过滤
5. 上下文增强（RAG）

#### 检索模式
- **密集检索**: 基于向量相似度的检索
- **稀疏检索**: 基于关键词的传统检索
- **混合检索**: 密集检索和稀疏检索的结合

## 数据模型设计

### 知识库表 (KnowledgeBase)

```python
class KnowledgeBase(BaseSQLModel):
    id: Optional[int] = Field(default=None, primary_key=True)
    name: str = Field(max_length=100)
    description: Optional[str] = Field(default=None, max_length=500)
    user_id: int = Field(index=True)
    group_id: Optional[int] = Field(default=None, index=True)
    status: int = Field(default=1)  # 1:正常 2:处理中 3:错误
    create_time: Optional[datetime] = Field(default_factory=datetime.utcnow)
    update_time: Optional[datetime] = Field(default_factory=datetime.utcnow)
```

### 文档表 (Document)

```python
class Document(BaseSQLModel):
    id: Optional[str] = Field(default=None, primary_key=True)
    knowledge_id: int = Field(index=True)
    name: str = Field(max_length=200)
    source: str = Field(max_length=500)  # 文件路径或URL
    chunk_size: int = Field(default=1000)
    status: int = Field(default=1)  # 1:待处理 2:处理中 3:已完成 4:错误
    create_time: Optional[datetime] = Field(default_factory=datetime.utcnow)
    update_time: Optional[datetime] = Field(default_factory=datetime.utcnow)
```

### 文档片段表 (DocumentChunk)

```python
class DocumentChunk(BaseSQLModel):
    id: Optional[str] = Field(default=None, primary_key=True)
    document_id: str = Field(index=True)
    chunk_index: int = Field()
    content: str = Field()
    metadata: Optional[str] = Field(default=None)  # JSON格式元数据
    embedding: Optional[str] = Field(default=None)  # 向量数据
    create_time: Optional[datetime] = Field(default_factory=datetime.utcnow)
```

## 核心服务

### 1. 文档处理服务

#### 功能
- 文档上传和存储
- 文档格式转换
- 内容提取和清洗
- 元数据提取

#### 实现类
```python
class DocumentProcessingService:
    @staticmethod
    async def process_document(document_id: str) -> bool:
        """处理文档"""
        pass

    @staticmethod
    async def extract_content(file_path: str, file_type: str) -> str:
        """提取文档内容"""
        pass
```

### 2. 向量化服务

#### 功能
- 文本分块
- 向量编码
- 向量存储

#### 实现类
```python
class VectorizationService:
    @staticmethod
    async def vectorize_document(document_id: str) -> bool:
        """向量化文档"""
        pass

    @staticmethod
    async def generate_embeddings(text_chunks: List[str]) -> List[List[float]]:
        """生成向量嵌入"""
        pass
```

### 3. 检索服务

#### 功能
- 向量相似度检索
- 关键词检索
- 混合检索
- 结果排序和过滤

#### 实现类
```python
class RetrievalService:
    @staticmethod
    async def semantic_search(query: str, knowledge_id: int, top_k: int = 10) -> List[SearchResult]:
        """语义检索"""
        pass

    @staticmethod
    async def keyword_search(query: str, knowledge_id: int, top_k: int = 10) -> List[SearchResult]:
        """关键词检索"""
        pass
```

## RAG (检索增强生成) 实现

### 功能描述
结合检索和大语言模型生成的能力，提供更准确和丰富的回答。

### 实现流程
1. 用户提出问题
2. 知识库中检索相关内容
3. 构造 Prompt 上下文
4. 调用 LLM 生成答案
5. 返回结果给用户

### 核心组件
```python
class RAGService:
    @staticmethod
    async def generate_answer(question: str, knowledge_id: int) -> str:
        """基于检索的问答生成"""
        # 1. 检索相关文档片段
        chunks = await RetrievalService.semantic_search(question, knowledge_id)

        # 2. 构造上下文
        context = "\n".join([chunk.content for chunk in chunks])

        # 3. 调用LLM生成答案
        prompt = f"基于以下内容回答问题：{question}\n\n内容：{context}"
        answer = await LLMService.generate(prompt)

        return answer
```

## API 接口设计

### 文档管理接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/knowledge/upload` | POST | 上传文档 |
| `/knowledge/list` | GET | 文档列表 |
| `/knowledge/{id}` | GET | 获取文档详情 |
| `/knowledge/{id}` | DELETE | 删除文档 |
| `/knowledge/{id}/status` | GET | 获取文档处理状态 |

### 检索接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/knowledge/search` | POST | 语义检索 |
| `/knowledge/keyword_search` | POST | 关键词检索 |
| `/knowledge/rag` | POST | RAG问答 |

### 知识库管理接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/knowledge/base/create` | POST | 创建知识库 |
| `/knowledge/base/list` | GET | 知识库列表 |
| `/knowledge/base/{id}` | GET | 获取知识库详情 |
| `/knowledge/base/{id}` | PUT | 更新知识库 |
| `/knowledge/base/{id}` | DELETE | 删除知识库 |

## 异步处理机制

### 文档处理流水线
1. **文档上传**: 用户上传文档到 MinIO
2. **任务创建**: 创建文档处理任务
3. **异步处理**: Celery Worker 处理文档
4. **内容提取**: 解析文档内容
5. **向量化**: 生成向量嵌入
6. **索引构建**: 存储到 Milvus
7. **状态更新**: 更新处理状态

### 任务队列设计
```python
class KnowledgeTask:
    @staticmethod
    @celery.task
    def process_document_task(document_id: str):
        """文档处理任务"""
        pass

    @staticmethod
    @celery.task
    def vectorize_document_task(document_id: str):
        """文档向量化任务"""
        pass
```

## 存储架构

### 对象存储 (MinIO)
- **公共桶**: 存储平台上的持久化文件，设置为可公开访问
- **临时桶**: 存储临时文件，设置有效期自动清理

### 向量数据库 (Milvus)
- **集合设计**: 每个知识库对应一个集合
- **分区策略**: 支持按时间或业务维度分区
- **索引类型**: 支持多种向量索引算法

### 搜索引擎 (Elasticsearch)
- **索引设计**: 存储文档元数据和关键词信息
- **搜索优化**: 支持复合查询和聚合分析

## 性能优化

### 文本分块策略
- **固定大小分块**: 按字符数分割
- **语义分块**: 按句子或段落分割
- **重叠分块**: 相邻块之间保留重叠内容

### 向量缓存
- **热点向量缓存**: Redis 缓存高频查询向量
- **批量处理**: 批量生成和存储向量

### 检索优化
- **近似最近邻搜索**: 使用 ANN 算法提高检索速度
- **多索引融合**: 结合多种索引提高检索质量
- **结果缓存**: 缓存常见查询结果

## 安全机制

### 访问控制
- **知识库权限**: 基于 RBAC 的访问控制
- **文档权限**: 继承知识库权限设置
- **操作权限**: 读取、写入、删除等细粒度控制

### 数据安全
- **传输加密**: HTTPS 传输保护
- **存储加密**: 敏感数据加密存储
- **访问日志**: 记录所有访问行为

## 配置项

### 相关配置
```yaml
# 知识库配置
knowledge:
  chunk_size: 1000              # 文本分块大小
  chunk_overlap: 200            # 分块重叠大小
  vector_dim: 768               # 向量维度
  similarity_threshold: 0.8     # 相似度阈值
  max_search_results: 10        # 最大检索结果数

# 支持的文件类型
uns_support: ['png','jpg','jpeg','bmp','doc', 'docx', 'ppt', 'pptx', 'xls', 'xlsx', 'txt', 'md', 'html', 'pdf', 'csv', 'tiff']
```

## 扩展性设计

### 插件化文档解析器
支持自定义文档解析器扩展：
1. 解析器接口定义
2. 格式适配器开发
3. 动态加载机制

### 多模态支持
- **图像处理**: OCR 和图像理解
- **音频处理**: 语音识别和处理
- **视频处理**: 视频内容提取

### 多语言支持
- **多语言文档解析**: 支持多种语言的OCR和NLP处理
- **多语言检索**: 支持跨语言检索

## 监控与运维

### 性能监控
- **处理时间**: 文档处理耗时统计
- **检索延迟**: 查询响应时间监控
- **系统资源**: CPU、内存、磁盘使用率

### 错误处理
- **错误分类**: 文档格式错误、处理错误、存储错误等
- **重试机制**: 自动重试和人工干预
- **告警机制**: 异常情况及时告警

### 日志记录
- **操作日志**: 用户操作记录
- **处理日志**: 文档处理过程日志
- **检索日志**: 查询行为日志

## 测试策略

### 单元测试
- 文档解析器测试
- 向量化服务测试
- 检索服务测试

### 集成测试
- 完整文档处理流程测试
- 检索准确性测试
- RAG效果测试

### 性能测试
- 大规模文档处理性能
- 高并发检索能力
- 向量存储和查询性能

## 流程图

详细的流程图和脑图已拆分到独立文件，便于维护：

- [文档处理流程图](../diagrams/knowledge/document_flow.md)
- [RAG检索流程图](../diagrams/knowledge/rag_flow.md)
- [知识库创建时序图](../diagrams/knowledge/sequence.md)
- [向量检索数据流图](../diagrams/knowledge/vector_dataflow.md)
- [知识库模块脑图](../diagrams/knowledge/mindmap.md)