# Bisheng 二次开发指南

## 概述

本指南为希望在 Bisheng 平台基础上进行二次开发的开发者提供详细的指导，包括开发环境搭建、代码结构说明、自定义功能开发、测试和部署等环节。

## 开发环境搭建

### 1. 系统要求
- Python 3.8+
- Node.js 16+
- Docker (可选，用于运行依赖服务)
- Git 版本控制工具

### 2. 后端开发环境
```bash
# 克隆项目代码
git clone https://github.com/dataelement/bisheng.git
cd bisheng/src/backend

# 创建虚拟环境
python -m venv venv
source venv/bin/activate  # Linux/Mac
# 或
venv\Scripts\activate     # Windows

# 安装依赖
pip install -r requirements.txt
```

### 3. 前端开发环境
```bash
# 进入前端目录
cd ../frontend/platform

# 安装依赖
npm install

# 启动开发服务器
npm run dev
```

### 4. 依赖服务
使用 Docker 启动依赖服务：
```bash
# 进入 docker 目录
cd ../../docker

# 启动依赖服务
docker compose -f docker-compose.yml up -d mysql redis elasticsearch milvus minio
```

## 项目结构说明

### 后端项目结构
```
src/backend/
├── bisheng/                    # 核心代码
│   ├── api/                    # API接口层
│   │   ├── v1/                 # API v1版本
│   │   └── router.py           # 路由配置
│   ├── chat/                   # 聊天模块
│   ├── common/                 # 公共模块
│   ├── components/             # 组件模块
│   ├── core/                   # 核心模块
│   ├── database/               # 数据库模型
│   ├── knowledge/              # 知识库模块
│   ├── llm/                    # LLM集成模块
│   ├── services/               # 业务服务层
│   ├── utils/                  # 工具类
│   ├── worker/                 # 异步任务处理
│   ├── main.py                 # 应用入口
│   └── config.yaml             # 配置文件
├── bisheng_langchain/          # LangChain集成
├── tests/                      # 测试代码
├── requirements.txt            # Python依赖
└── pyproject.toml             # 项目配置
```

### 前端项目结构
```
src/frontend/platform/
├── src/
│   ├── components/             # React组件
│   ├── pages/                  # 页面组件
│   ├── services/               # API服务
│   ├── store/                  # 状态管理
│   ├── utils/                  # 工具函数
│   ├── routes/                 # 路由配置
│   └── App.tsx                # 应用入口
├── public/                     # 静态资源
├── package.json               # Node.js依赖
└── vite.config.mts            # 构建配置
```

## 自定义功能开发

### 1. 添加新的API接口

#### 步骤1: 创建API路由
在 `api/v1/` 目录下创建新的路由文件：
```python
# api/v1/custom_api.py
from fastapi import APIRouter, Depends
from bisheng.api.v1.schemas import resp_200

router = APIRouter(prefix='/custom', tags=['Custom'])

@router.get('/hello')
async def hello_world():
    return resp_200(data={'message': 'Hello World'})
```

#### 步骤2: 注册路由
在 `api/router.py` 中注册新路由：
```python
from bisheng.api.v1.custom_api import router as custom_router

router.include_router(custom_router)
```

### 2. 添加新的数据库模型

#### 步骤1: 定义模型
在 `database/models/` 目录下创建模型文件：
```python
# database/models/custom_model.py
from sqlmodel import Field, SQLModel
from typing import Optional
from datetime import datetime

class CustomModel(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    name: str = Field(max_length=100)
    description: Optional[str] = Field(default=None)
    create_time: Optional[datetime] = Field(default_factory=datetime.utcnow)
```

#### 步骤2: 创建数据访问对象
```python
# database/dao/custom_dao.py
from sqlmodel import select
from bisheng.core.database import get_sync_db_session
from bisheng.database.models.custom_model import CustomModel

class CustomDao:
    @staticmethod
    def create_custom_model(model: CustomModel) -> CustomModel:
        with get_sync_db_session() as session:
            session.add(model)
            session.commit()
            session.refresh(model)
            return model

    @staticmethod
    def get_custom_model_by_id(id: int) -> Optional[CustomModel]:
        with get_sync_db_session() as session:
            return session.get(CustomModel, id)
```

### 3. 添加新的业务服务

#### 步骤1: 创建服务类
```python
# services/custom_service.py
from bisheng.database.dao.custom_dao import CustomDao
from bisheng.database.models.custom_model import CustomModel

class CustomService:
    @staticmethod
    async def create_custom_item(name: str, description: str) -> CustomModel:
        model = CustomModel(name=name, description=description)
        return CustomDao.create_custom_model(model)

    @staticmethod
    async def get_custom_item(id: int) -> CustomModel:
        return CustomDao.get_custom_model_by_id(id)
```

### 4. 添加新的前端页面

#### 步骤1: 创建页面组件
```tsx
// src/pages/CustomPage.tsx
import React from 'react';

const CustomPage: React.FC = () => {
  return (
    <div>
      <h1>自定义页面</h1>
      <p>这是一个自定义页面示例</p>
    </div>
  );
};

export default CustomPage;
```

#### 步骤2: 添加路由
```tsx
// src/routes/index.tsx
import CustomPage from '../pages/CustomPage';

const routes = [
  {
    path: '/custom',
    element: <CustomPage />
  }
];
```

### 5. 添加新的节点类型（工作流）

#### 步骤1: 定义节点组件
```python
# components/custom_node.py
from bisheng.components.base import BaseComponent

class CustomNode(BaseComponent):
    def __init__(self):
        super().__init__()
        self.name = "CustomNode"
        self.description = "自定义节点"

    def build(self):
        # 定义节点输入输出
        pass

    def run(self, input_data):
        # 实现节点逻辑
        return {"result": "custom result"}
```

#### 步骤2: 注册节点
在组件注册文件中添加新节点：
```python
# components/__init__.py
from bisheng.components.custom_node import CustomNode

COMPONENTS = {
    "CustomNode": CustomNode,
    # 其他组件...
}
```

## 开发规范

### 1. 代码规范

#### Python 代码规范
- 遵循 PEP 8 代码规范
- 使用类型注解
- 编写文档字符串
- 单元测试覆盖率不低于 80%

#### TypeScript 代码规范
- 遵循 Airbnb TypeScript 规范
- 使用 TypeScript 类型系统
- 组件化开发
- 状态管理规范化

### 2. API 设计规范
- 遵循 RESTful 设计原则
- 统一的响应格式
- 详细的错误处理
- API 文档完整性

### 3. 数据库设计规范
- 第三范式设计
- 合理的索引策略
- 字段命名规范
- 数据完整性约束

## 测试策略

### 1. 单元测试
```python
# tests/test_custom_service.py
import pytest
from bisheng.services.custom_service import CustomService

class TestCustomService:
    def test_create_custom_item(self):
        result = CustomService.create_custom_item("test", "description")
        assert result.name == "test"
        assert result.description == "description"
```

### 2. 集成测试
```python
# tests/integration/test_api.py
import pytest
from fastapi.testclient import TestClient
from bisheng.main import app

client = TestClient(app)

def test_custom_api():
    response = client.get("/api/v1/custom/hello")
    assert response.status_code == 200
    assert response.json()["data"]["message"] == "Hello World"
```

### 3. 端到端测试
使用 Cypress 或 Playwright 进行前端端到端测试。

## 调试技巧

### 1. 后端调试
```python
# 在代码中添加调试信息
import logging
logger = logging.getLogger(__name__)
logger.debug("调试信息: %s", variable)
```

### 2. 前端调试
```javascript
// 使用 console.log 调试
console.log('调试信息:', variable);

// 使用 React Developer Tools
// 在浏览器中安装 React Developer Tools 扩展
```

### 3. 数据库调试
```sql
-- 查看表结构
DESCRIBE table_name;

-- 查看数据
SELECT * FROM table_name LIMIT 10;
```

## 性能优化

### 1. 数据库优化
- 添加合适的索引
- 优化查询语句
- 使用连接池
- 分页查询大数据集

### 2. 缓存策略
```python
# 使用 Redis 缓存
from bisheng.core.cache import get_redis_client

async def get_cached_data(key):
    redis_client = await get_redis_client()
    cached_data = await redis_client.aget(key)
    if cached_data:
        return json.loads(cached_data)

    # 获取数据并缓存
    data = fetch_data_from_db()
    await redis_client.aset(key, json.dumps(data), expiration=3600)
    return data
```

### 3. 异步处理
```python
# 使用 Celery 异步处理耗时任务
from celery import Celery

@celery.task
def process_large_file(file_path):
    # 处理大文件
    pass
```

## 安全开发

### 1. 输入验证
```python
from pydantic import BaseModel, validator

class UserCreate(BaseModel):
    username: str
    email: str

    @validator('username')
    def validate_username(cls, v):
        if len(v) < 3:
            raise ValueError('用户名长度不能少于3个字符')
        return v
```

### 2. 权限控制
```python
from fastapi import Depends
from bisheng.api.services.user_service import get_login_user

@router.get('/protected')
async def protected_endpoint(login_user = Depends(get_login_user)):
    # 检查用户权限
    if not login_user.has_permission('read_custom_data'):
        raise HTTPException(status_code=403, detail='权限不足')
    return {'message': '访问成功'}
```

### 3. 数据加密
```python
# 敏感数据加密存储
from cryptography.fernet import Fernet

def encrypt_data(data):
    key = Fernet.generate_key()
    cipher = Fernet(key)
    encrypted_data = cipher.encrypt(data.encode())
    return encrypted_data, key
```

## 扩展开发

### 1. 插件系统
Bisheng 支持插件化扩展，可以通过以下方式添加插件：

#### 步骤1: 创建插件目录
```
plugins/
└── custom_plugin/
    ├── __init__.py
    ├── plugin.py
    └── config.yaml
```

#### 步骤2: 实现插件接口
```python
# plugins/custom_plugin/plugin.py
from bisheng.plugins.base import BasePlugin

class CustomPlugin(BasePlugin):
    def __init__(self):
        super().__init__()
        self.name = "CustomPlugin"
        self.version = "1.0.0"

    def initialize(self):
        # 插件初始化逻辑
        pass

    def execute(self, data):
        # 插件执行逻辑
        return {"result": "plugin executed"}
```

### 2. 自定义节点开发
#### 步骤1: 定义节点配置
```yaml
# nodes/custom_node.yaml
name: CustomNode
category: Custom
description: 自定义节点
input:
  - name: input_data
    type: string
    required: true
output:
  - name: result
    type: string
```

#### 步骤2: 实现节点逻辑
```python
# nodes/custom_node.py
class CustomNode:
    def __init__(self, config):
        self.config = config

    def execute(self, input_data):
        # 节点处理逻辑
        result = f"处理结果: {input_data}"
        return {"result": result}
```

## 贡献指南

### 1. 提交代码
- Fork 项目仓库
- 创建功能分支
- 编写代码和测试
- 提交 Pull Request

### 2. 代码审查
- 遵循代码规范
- 编写清晰的提交信息
- 通过所有测试
- 等待代码审查

### 3. 文档更新
- 更新相关文档
- 添加使用示例
- 保持文档同步

## 常见问题

### 1. 依赖安装失败
```bash
# 清理缓存重新安装
pip cache purge
pip install -r requirements.txt --no-cache-dir
```

### 2. 数据库迁移问题
```bash
# 手动执行数据库迁移
alembic upgrade head
```

### 3. 前端构建失败
```bash
# 清理构建缓存
npm cache clean --force
rm -rf node_modules
npm install
npm run build
```

## 社区支持

### 1. 官方文档
- 项目 Wiki
- API 文档
- 部署指南

### 2. 社区交流
- GitHub Issues
- 微信交流群
- 技术论坛

### 3. 贡献项目
- Bug 修复
- 功能增强
- 文档完善
- 测试用例

## 版本管理

### 1. 版本发布流程
- 代码冻结
- 测试验证
- 版本标记
- 镜像构建
- 文档更新

### 2. 兼容性保证
- API 向后兼容
- 数据库迁移脚本
- 升级指南文档
- 弃用策略说明

通过本指南，开发者可以快速上手 Bisheng 平台的二次开发工作，根据具体业务需求进行功能扩展和定制化开发。