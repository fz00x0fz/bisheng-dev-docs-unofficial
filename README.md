# Bisheng 开发文档

## 概述

本仓库包含了 Bisheng 平台的完整开发文档，涵盖了系统架构、模块设计、API接口、配置说明、部署指南和二次开发等内容，为开发者提供全面的技术参考资料。

## 文档目录结构

```
bisheng-dev-docs/
├── overview.md                    # 项目概述文档
├── README.md                      # 本文档
├── architecture/                  # 架构设计文档
│   ├── architecture.md           # 系统架构详细设计
│   └── system_architecture_diagrams.md  # 系统架构图集（已拆分到diagrams/）
├── diagrams/                      # 架构图与流程图（独立维护）
│   ├── system/                   # 系统级架构图
│   ├── user_management/          # 用户管理模块图
│   ├── workflow/                 # 工作流模块图
│   ├── knowledge/                # 知识库模块图
│   ├── chat/                     # 聊天模块图
│   ├── llm/                      # LLM集成模块图
│   └── linsight/                 # Linsight模块图
├── modules/                       # 核心模块文档
│   ├── user_management.md        # 用户管理模块
│   ├── workflow.md               # 工作流模块
│   ├── workflow_nodes.md         # 工作流节点类型详解
│   ├── knowledge_base.md         # 知识库模块
│   ├── chat.md                   # 聊天模块
│   ├── llm_integration.md        # LLM集成模块
│   ├── linsight.md               # Linsight专家代理模块
│   └── langchain_extensions.md   # LangChain扩展库
├── api/                           # API接口文档
│   └── api_documentation.md      # 完整API接口说明
├── configuration/                 # 配置说明文档
│   └── configuration_guide.md    # 系统配置详细说明
├── deployment/                    # 部署指南
│   └── deployment_guide.md       # 部署和运行指南
├── development/                   # 二次开发指南
│   └── development_guide.md      # 二次开发详细指南
└── extension/                     # 扩展开发指南
    └── extension_development.md  # 无侵入式扩展开发指南
```

## 文档内容概览

### 1. 项目概述 (overview.md)
- 项目介绍和特点
- 技术架构概览
- 目录结构说明
- 快速开始指南
- 核心模块介绍

### 2. 架构设计 (architecture/)
- **architecture.md**: 系统架构详细设计
  - 整体架构设计
  - 后端架构层次
  - 核心模块详解
  - 数据流向说明
  - 部署架构设计
  - 性能优化策略
  - 安全设计
- **system_architecture_diagrams.md**: 系统架构图集
  - 整体系统架构图
  - 模块依赖关系图
  - 技术栈全景图
  - 部署架构图
  - 数据流向图
  - 安全架构图

### 3. 核心模块文档 (modules/)
- **用户管理模块**: 用户认证、权限控制、角色管理
- **工作流模块**: 流程编排、版本管理、执行引擎
- **工作流节点详解**: 所有节点类型的详细配置和使用方法
- **知识库模块**: 文档管理、内容解析、向量索引
- **聊天模块**: 实时通信、多轮对话、会话管理
- **LLM集成模块**: 模型适配、统一接口、成本控制
- **Linsight模块**: 专家级代理、SOP驱动、任务自动执行
- **LangChain扩展库**: 自定义模型适配、文档加载器、GPTs助手

### 4. API接口文档 (api/api_documentation.md)
- API基础信息
- 用户管理接口
- 工作流接口
- 知识库接口
- 聊天接口
- LLM接口
- 系统接口
- 错误码说明

### 5. 配置说明 (configuration/configuration_guide.md)
- 配置文件结构
- 数据库配置
- 缓存配置
- 向量存储配置
- 对象存储配置
- 日志配置
- 安全配置

### 6. 部署指南 (deployment/deployment_guide.md)
- 系统要求
- 部署方式
- Docker配置详解
- 初始化配置
- 性能调优
- 高可用部署
- 监控和日志
- 备份和恢复

### 7. 二次开发指南 (development/development_guide.md)
- 开发环境搭建
- 项目结构说明
- 自定义功能开发
- 开发规范
- 测试策略
- 调试技巧
- 性能优化
- 安全开发

### 8. 扩展开发指南 (extension/extension_development.md)
- 无侵入式扩展方案
- 插件化架构设计
- 事件监听机制
- 第三方集成扩展
- 部署和维护策略
- 风险控制和解决方案

### 9. 架构图与流程图 (diagrams/)
独立维护的可视化图表，使用 Mermaid 语法编写：
- **system/**: 系统级架构图（整体架构、模块依赖、技术栈、部署、数据流、安全）
- **user_management/**: 用户管理模块（认证流程、权限验证、模块脑图）
- **workflow/**: 工作流模块（执行流程、版本管理、时序图、数据流、脑图）
- **knowledge/**: 知识库模块（文档处理、RAG检索、时序图、向量数据流、脑图）
- **chat/**: 聊天模块（WebSocket连接、消息处理、时序图、数据流、脑图）
- **llm/**: LLM集成模块（调用流程、流式输出、时序图、多模型数据流、脑图）
- **linsight/**: Linsight模块（任务执行、SOP生成、时序图、工具数据流、状态图、脑图）

## 使用说明

### 阅读顺序建议
1. 首先阅读 **项目概述** 了解整体情况
2. 查看 **架构设计** 理解系统设计思路
3. 根据关注模块阅读对应的 **核心模块文档**
4. 参考 **API接口文档** 进行接口调用
5. 查阅 **配置说明** 进行系统配置
6. 阅读 **部署指南** 进行系统部署
7. 参考 **二次开发指南** 进行功能扩展
8. 查看 **扩展开发指南** 了解无侵入式扩展方案

### 文档更新
本文档会随着 Bisheng 平台的版本更新而持续维护，建议定期查看最新版本。

## 贡献文档

如果您发现文档中的错误或有改进建议，欢迎提交 Issue 或 Pull Request。

## 版权信息

Bisheng 是开源项目，遵循相应的开源许可证。