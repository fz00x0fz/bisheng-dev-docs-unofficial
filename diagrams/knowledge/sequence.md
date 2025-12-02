# 知识库创建时序图

展示知识库创建和文档上传过程中各组件的交互时序。

## 代码入口

| 类/函数 | 文件路径 | 说明 |
|---------|----------|------|
| `KnowledgeDao` | `src/backend/bisheng/knowledge/domain/models/knowledge.py` | 知识库数据访问 |
| `KnowledgeFileDao` | `src/backend/bisheng/knowledge/domain/models/knowledge_file.py` | 文件数据访问 |
| `KnowledgeService` | `src/backend/bisheng/api/services/knowledge.py` | 知识库服务 |
| `KnowledgeRag` | `src/backend/bisheng/knowledge/domain/knowledge_rag.py` | RAG初始化 |
| `knowledge.py` | `src/backend/bisheng/knowledge/api/endpoints/knowledge.py` | API路由 |

```mermaid
sequenceDiagram
    participant User as 用户
    participant API as API层
    participant Service as 知识库服务
    participant DB as 数据库
    participant Milvus as Milvus
    participant MinIO as MinIO

    User->>API: 创建知识库请求
    API->>Service: 调用创建方法
    Service->>DB: 创建知识库记录
    DB-->>Service: 返回知识库ID
    Service->>Milvus: 创建Collection
    Milvus-->>Service: Collection创建成功
    Service->>MinIO: 创建存储Bucket
    MinIO-->>Service: Bucket创建成功
    Service-->>API: 返回创建结果
    API-->>User: 知识库创建成功

    User->>API: 上传文档
    API->>Service: 处理上传
    Service->>MinIO: 存储文件
    MinIO-->>Service: 文件URL
    Service->>DB: 创建文档记录
    Service->>Service: 异步处理任务
    Service-->>API: 返回上传成功
    API-->>User: 文档上传成功，处理中
```

## 时序说明

### 知识库创建

1. 用户发起创建请求
2. 创建数据库记录
3. 创建Milvus Collection
4. 创建MinIO Bucket
5. 返回创建结果

### 文档上传

1. 用户上传文档
2. 存储文件到MinIO
3. 创建文档记录
4. 发送异步处理任务
5. 返回上传成功

### 涉及组件

| 组件 | 职责 |
|------|------|
| 数据库 | 存储知识库和文档元数据 |
| Milvus | 存储文档向量 |
| MinIO | 存储原始文档文件 |
| Celery | 异步处理文档 |
