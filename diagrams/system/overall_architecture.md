# 系统整体架构图

展示Bisheng平台的整体系统架构，包含各层级组件及其关系。

## 代码入口

| 层级 | 目录/文件 | 说明 |
|------|----------|------|
| **应用入口** | `src/backend/bisheng/main.py` | FastAPI应用入口 |
| **API层** | `src/backend/bisheng/api/` | REST API定义 |
| **服务层** | `src/backend/bisheng/api/services/` | 业务逻辑服务 |
| **数据层** | `src/backend/bisheng/database/` | 数据库模型和访问 |
| **核心模块** | `src/backend/bisheng/chat/` | 聊天模块 |
| **工作流** | `src/backend/bisheng/workflow/` | 工作流引擎 |
| **知识库** | `src/backend/bisheng/knowledge/` | 知识库模块 |
| **LLM** | `src/backend/bisheng/llm/` | LLM集成模块 |
| **Linsight** | `src/backend/bisheng/linsight/` | Linsight模块 |
| **LangChain扩展** | `src/backend/bisheng_langchain/` | LangChain扩展库 |

```mermaid
graph TB
    subgraph "客户端层 Client Layer"
        WEB[Web Browser<br/>React + TypeScript]
        API_CLIENT[API Client<br/>REST/WebSocket]
    end

    subgraph "接入层 Gateway Layer"
        NGINX[Nginx<br/>反向代理/负载均衡]
        WS_GATEWAY[WebSocket Gateway]
    end

    subgraph "应用层 Application Layer"
        subgraph "API Services"
            USER_API[用户管理 API]
            FLOW_API[技能/工作流 API]
            KNOWLEDGE_API[知识库 API]
            CHAT_API[聊天 API]
            LLM_API[LLM API]
            LINSIGHT_API[Linsight API]
            ASSISTANT_API[助手 API]
        end
        
        subgraph "Core Services"
            AUTH_SVC[认证服务<br/>JWT Auth]
            WORKFLOW_ENGINE[工作流引擎]
            CHAT_MANAGER[聊天管理器]
            RAG_SERVICE[RAG服务]
            LINSIGHT_ENGINE[Linsight引擎]
        end
    end

    subgraph "业务逻辑层 Business Layer"
        USER_SVC[用户服务]
        FLOW_SVC[流程服务]
        KNOWLEDGE_SVC[知识库服务]
        LLM_SVC[LLM服务]
        TASK_SVC[任务服务]
    end

    subgraph "数据访问层 Data Access Layer"
        DAO[DAO Layer<br/>SQLModel/SQLAlchemy]
        CACHE[缓存层<br/>Redis Client]
        VECTOR_CLIENT[向量库客户端]
        STORAGE_CLIENT[对象存储客户端]
    end

    subgraph "基础设施层 Infrastructure Layer"
        MYSQL[(MySQL<br/>关系数据库)]
        REDIS[(Redis<br/>缓存/消息队列)]
        MILVUS[(Milvus<br/>向量数据库)]
        ES[(Elasticsearch<br/>全文搜索)]
        MINIO[(MinIO<br/>对象存储)]
        CELERY[Celery<br/>异步任务队列]
    end

    subgraph "外部服务 External Services"
        OPENAI[OpenAI API]
        ANTHROPIC[Anthropic API]
        QWEN[通义千问 API]
        ZHIPU[智谱 API]
        OTHER_LLM[其他LLM服务]
    end

    WEB --> NGINX
    API_CLIENT --> NGINX
    NGINX --> USER_API
    NGINX --> FLOW_API
    NGINX --> KNOWLEDGE_API
    NGINX --> CHAT_API
    NGINX --> LLM_API
    NGINX --> LINSIGHT_API
    NGINX --> ASSISTANT_API
    WS_GATEWAY --> CHAT_MANAGER

    USER_API --> AUTH_SVC
    USER_API --> USER_SVC
    FLOW_API --> WORKFLOW_ENGINE
    FLOW_API --> FLOW_SVC
    KNOWLEDGE_API --> RAG_SERVICE
    KNOWLEDGE_API --> KNOWLEDGE_SVC
    CHAT_API --> CHAT_MANAGER
    LLM_API --> LLM_SVC
    LINSIGHT_API --> LINSIGHT_ENGINE
    ASSISTANT_API --> WORKFLOW_ENGINE

    USER_SVC --> DAO
    FLOW_SVC --> DAO
    KNOWLEDGE_SVC --> DAO
    KNOWLEDGE_SVC --> VECTOR_CLIENT
    KNOWLEDGE_SVC --> STORAGE_CLIENT
    LLM_SVC --> CACHE
    TASK_SVC --> CELERY

    DAO --> MYSQL
    CACHE --> REDIS
    VECTOR_CLIENT --> MILVUS
    VECTOR_CLIENT --> ES
    STORAGE_CLIENT --> MINIO

    LLM_SVC --> OPENAI
    LLM_SVC --> ANTHROPIC
    LLM_SVC --> QWEN
    LLM_SVC --> ZHIPU
    LLM_SVC --> OTHER_LLM
```

## 架构说明

### 分层设计

| 层级 | 职责 | 技术实现 |
|------|------|----------|
| 客户端层 | 用户界面、API调用 | React, TypeScript, Axios |
| 接入层 | 负载均衡、反向代理、WebSocket | Nginx |
| 应用层 | API接口、核心服务 | FastAPI |
| 业务逻辑层 | 业务处理、数据转换 | Python Services |
| 数据访问层 | 数据持久化、缓存 | SQLModel, Redis |
| 基础设施层 | 数据存储、消息队列 | MySQL, Redis, Milvus |
| 外部服务 | LLM模型调用 | OpenAI, Anthropic等 |
