# Bisheng 项目开发文档

## 项目概述

BISHENG 是一个开源的大语言模型(LLM)应用开发运维平台，专注于企业场景。它已被众多行业领先组织和财富500强公司广泛使用。

"Bisheng"(毕升)是中国古代活字印刷术的发明者，他在促进人类知识传播方面发挥了至关重要的作用。我们希望 BISHENG 也能为智能应用的广泛实施提供强有力的支持。

## 项目特点

### 1. Linsight - 具备专家级品味的通用代理
通过 [AGL](https://github.com/dataelement/AgentGuidanceLanguage)(Agent Guidance Language) 框架，我们将领域专家的偏好、经验和业务逻辑嵌入到AI中，使代理在处理任务时能够表现出"专家级的理解"。

### 2. 独特的 BISHENG 工作流
- **独立且全面的应用编排框架**：能够在单一框架内执行各种任务（而类似产品依赖于bot调用或分别的聊天流程和工作流模块来处理不同任务）
- **人在回路**：允许用户在工作流执行过程中进行干预和提供反馈（包括多轮对话），而类似产品只能从头到尾执行工作流而不允许干预
- **强大功能**：支持循环、并行、批处理、条件逻辑以及所有逻辑组件的自由组合。还能处理复杂场景如多类型输入/输出、报告生成、内容审核等
- **用户友好直观**：类似于循环、并行和批处理等在类似产品中需要专门组件的操作，在BISHENG中可以轻松可视化为"流程图"（绘制循环形成循环，对齐元素创建并行，选择多项实现批处理）

### 3. 专为企业应用设计
文档审查、固定布局报告生成、多代理协作、政策更新对比、支持票务协助、客户服务协助、会议纪要生成、简历筛选、通话记录分析、非结构化数据治理、知识挖掘、数据分析等。

该平台支持构建**高度复杂的企业应用场景**，并提供**深度优化**，拥有数百个组件和数千个参数。

### 4. 企业级特性
安全审查、RBAC、用户组管理、按组流量控制、SSO/LDAP、漏洞扫描和修补、高可用性部署解决方案、监控、统计等功能是应用实施的根本保证。

### 5. 高精度文档解析
我们的高精度文档解析模型是在过去5年积累的大量高质量数据上训练的。它包括高精度印刷文本、手写文本和罕见字符识别模型、表格识别模型、布局分析模型和印章模型。您可以免费私有化部署。

### 6. 社区分享
各企业场景最佳实践的开放仓库。

## 技术架构

### 后端技术栈
- **编程语言**: Python
- **Web框架**: FastAPI
- **数据库**: MySQL 8.0
- **缓存**: Redis 7.0.4
- **向量数据库**: Milvus v2.5.10
- **搜索引擎**: Elasticsearch 8.12.0
- **对象存储**: MinIO
- **消息队列**: Celery with Redis
- **容器化**: Docker

### 前端技术栈
- **框架**: React + TypeScript
- **构建工具**: Vite
- **UI库**: TailwindCSS
- **状态管理**: 自定义状态管理方案

## 目录结构

```
bisheng/
├── docker/                     # Docker部署相关配置
│   ├── bisheng/               # Bisheng服务配置
│   ├── mysql/                 # MySQL配置
│   ├── redis/                 # Redis配置
│   ├── nginx/                 # Nginx配置
│   ├── docker-compose.yml     # 主要的Docker Compose配置文件
│   ├── docker-compose-ft.yml  # 微调服务配置
│   └── docker-compose-uns.yml # UNS服务配置
├── src/
│   ├── backend/               # 后端代码
│   │   ├── bisheng/           # 核心代码
│   │   │   ├── api/           # API接口层 (v1 RESTful API)
│   │   │   ├── chat/          # 聊天模块
│   │   │   ├── chat_session/  # 会话管理模块
│   │   │   ├── common/        # 公共模块（常量、错误码、模型等）
│   │   │   ├── components/    # 组件模块
│   │   │   ├── core/          # 核心模块（缓存、配置、存储等）
│   │   │   ├── custom/        # 自定义扩展
│   │   │   ├── database/      # 数据库模型（31个模型文件）
│   │   │   ├── finetune/      # 模型微调模块
│   │   │   ├── graph/         # 图结构处理
│   │   │   ├── interface/     # 接口定义
│   │   │   ├── knowledge/     # 知识库模块
│   │   │   ├── linsight/      # Linsight专家代理模块（核心特性）
│   │   │   ├── llm/           # LLM模型管理
│   │   │   ├── mcp_manage/    # MCP协议管理
│   │   │   ├── open_endpoints/# v2开放API（RPC风格）
│   │   │   ├── processing/    # 数据处理
│   │   │   ├── services/      # 业务服务层
│   │   │   ├── share_link/    # 分享链接模块
│   │   │   ├── utils/         # 工具类
│   │   │   ├── worker/        # Celery异步任务处理
│   │   │   └── workflow/      # 工作流引擎（核心特性）
│   │   ├── bisheng_langchain/ # LangChain扩展库
│   │   │   ├── agents/        # 自定义代理
│   │   │   ├── chains/        # 自定义链
│   │   │   ├── chat_models/   # 聊天模型适配
│   │   │   ├── document_loaders/ # 文档加载器
│   │   │   ├── embeddings/    # 嵌入模型
│   │   │   ├── gpts/          # GPTs相关
│   │   │   ├── linsight/      # Linsight Agent实现
│   │   │   ├── memory/        # 记忆模块
│   │   │   ├── rag/           # RAG实现
│   │   │   ├── retrievers/    # 检索器
│   │   │   ├── sql/           # SQL相关
│   │   │   └── vectorstores/  # 向量存储
│   │   └── fastapi_jwt_auth/  # JWT认证模块
│   └── frontend/              # 前端代码
│       └── platform/          # 平台前端
│           └── src/
│               ├── components/    # React组件
│               ├── controllers/   # API调用层
│               │   └── API/       # 各模块API封装
│               ├── pages/         # 页面组件
│               ├── store/         # 状态管理
│               └── workspace/     # 工作区组件
└── README.md                  # 项目说明文档
```

## 快速开始

### 系统要求
- CPU >= 4虚拟核心
- RAM >= 16GB
- Docker 19.03.9+
- Docker Compose 1.25.1+

推荐硬件配置：18虚拟核心，48GB内存。除了安装BISHENG外，默认还会安装以下第三方组件：ES、Milvus和Onlyoffice。

### 安装步骤
```bash
# 克隆项目
git clone https://github.com/dataelement/bisheng.git

# 进入安装目录
cd bisheng/docker

# 启动服务
docker compose -f docker-compose.yml -p bisheng up -d
```

启动完成后，在浏览器中访问 http://IP:3001，登录页面会出现，进行用户注册。

默认情况下，第一个注册的用户将成为系统管理员。

## 核心模块介绍

### 用户管理系统
负责用户认证、授权、角色管理等功能，采用JWT进行身份验证，支持RBAC权限控制。

### 工作流引擎
BISHENG的核心功能之一，提供可视化的流程编排能力，支持复杂业务逻辑的组合与执行。

### 知识库系统
基于向量数据库和搜索引擎构建的知识管理系统，支持文档解析、索引和检索。

### LLM集成
支持多种大语言模型的集成，提供统一的接口和服务封装。

### 聊天系统
实现实时聊天功能，支持多轮对话和上下文管理。

## 部署架构

BISHENG采用微服务架构，主要包含以下服务：
- MySQL数据库服务
- Redis缓存服务
- Elasticsearch搜索服务
- Milvus向量数据库服务
- MinIO对象存储服务
- Bisheng后端API服务
- Bisheng前端Web服务
- Celery异步任务处理服务

所有服务都通过Docker容器化部署，便于扩展和维护。