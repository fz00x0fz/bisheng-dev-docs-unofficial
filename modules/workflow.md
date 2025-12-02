# 工作流模块详细设计文档

## 模块概述

工作流模块是 Bisheng 平台的核心功能之一，提供了可视化的流程编排能力，支持复杂业务逻辑的组合与执行。该模块允许用户通过拖拽方式构建复杂的AI应用流程，支持多种节点类型、条件分支、循环控制等高级功能。

## 模块结构

```
api/v1/workflow.py          # 工作流API接口
database/models/flow.py     # 流程数据模型
database/models/flow_version.py # 流程版本数据模型
services/flow_service.py    # 流程业务逻辑
services/workflow_service.py # 工作流业务逻辑
chat/                       # 聊天相关实现
```

## 核心概念

### 1. 流程 (Flow)
流程是工作流的基本单位，包含了流程的元数据信息，如名称、描述、创建者、状态等。

### 2. 流程版本 (Flow Version)
为了支持流程的迭代开发和版本管理，每个流程可以有多个版本，每个版本包含具体的流程定义数据。

### 3. 节点 (Node)
流程中的基本执行单元，每个节点代表一个具体的功能或操作。

### 4. 边 (Edge)
连接节点的有向边，表示数据流和执行顺序。

### 5. 流程状态
- **草稿状态**: 流程正在编辑中，不可执行
- **在线状态**: 流程已发布，可供执行
- **离线状态**: 流程已下线，停止执行

## 数据模型设计

### 流程表 (Flow)

```python
class Flow(BaseSQLModel):
    id: Optional[str] = Field(default=None, primary_key=True)
    name: str = Field(max_length=50)
    user_id: int = Field(index=True)
    description: Optional[str] = Field(default=None, max_length=500)
    data: Optional[str] = Field(default=None)  # 流程定义JSON
    status: int = Field(default=1)  # 1:草稿 2:在线 3:离线
    create_time: Optional[datetime] = Field(default_factory=datetime.utcnow)
    update_time: Optional[datetime] = Field(default_factory=datetime.utcnow)
    flow_type: int = Field(default=1)  # 1:flow 5:assistant 10:workflow
```

### 流程版本表 (FlowVersion)

```python
class FlowVersion(BaseSQLModel):
    id: Optional[int] = Field(default=None, primary_key=True)
    flow_id: str = Field(index=True)
    name: str = Field(max_length=50)
    data: Optional[str] = Field(default=None)  # 版本数据
    description: Optional[str] = Field(default=None, max_length=500)
    create_time: Optional[datetime] = Field(default_factory=datetime.utcnow)
    update_time: Optional[datetime] = Field(default_factory=datetime.utcnow)
    flow_type: int = Field(default=1)
```

## 核心功能

### 1. 流程创建

#### 功能描述
允许用户创建新的工作流程。

#### 接口定义
- **URL**: `/workflow/create`
- **Method**: POST
- **参数**:
  ```json
  {
    "name": "流程名称",
    "description": "流程描述"
  }
  ```

#### 实现逻辑
1. 验证流程名称唯一性
2. 创建流程基础信息
3. 初始化默认版本
4. 记录创建日志

### 2. 流程版本管理

#### 功能描述
支持流程的版本控制，允许用户创建、更新、删除和切换流程版本。

#### 核心接口
- **创建版本**: `/workflow/versions` (POST)
- **更新版本**: `/workflow/versions/{version_id}` (PUT)
- **删除版本**: `/workflow/versions/{version_id}` (DELETE)
- **获取版本**: `/workflow/versions/{version_id}` (GET)
- **版本列表**: `/workflow/versions` (GET)
- **切换版本**: `/workflow/change_version` (POST)

#### 实现逻辑
1. 版本创建：复制当前版本数据，生成新版本
2. 版本更新：修改版本的元数据或流程定义
3. 版本删除：软删除版本，保留历史记录
4. 版本切换：设置流程的当前活动版本

### 3. 流程状态管理

#### 功能描述
控制流程的生命周期状态，包括上线、下线等操作。

#### 接口定义
- **URL**: `/workflow/status`
- **Method**: PATCH
- **参数**:
  ```json
  {
    "flow_id": "流程ID",
    "version_id": "版本ID",
    "status": "状态值"
  }
  ```

#### 状态转换规则
- 草稿 → 在线：发布流程
- 在线 → 离线：下线流程
- 离线 → 在线：重新上线
- 草稿 → 离线：不允许

### 4. 流程执行

#### 功能描述
执行工作流程，处理输入数据并产生输出结果。

#### 执行方式
1. **WebSocket 实时执行**: 通过 WebSocket 连接实时执行并返回结果
2. **单节点调试**: 执行单个节点进行测试
3. **批处理执行**: 批量处理多个输入数据

#### WebSocket 接口
- **URL**: `/workflow/chat/{workflow_id}`
- **协议**: WebSocket
- **功能**: 实时交互式流程执行

#### 单节点执行接口
- **URL**: `/workflow/run_once`
- **Method**: POST
- **功能**: 调试单个节点

### 5. 流程数据管理

#### 功能描述
管理流程的定义数据和执行数据。

#### 数据结构
流程数据采用 JSON 格式存储，包含以下主要部分：
```json
{
  "nodes": [
    {
      "id": "节点ID",
      "type": "节点类型",
      "position": {"x": 0, "y": 0},
      "data": { /* 节点参数 */ }
    }
  ],
  "edges": [
    {
      "id": "连线ID",
      "source": "源节点ID",
      "target": "目标节点ID",
      "sourceHandle": "源句柄",
      "targetHandle": "目标句柄"
    }
  ]
}
```

## 节点类型

### 1. 输入节点
- 文本输入
- 文件输入
- 数据库输入

### 2. 处理节点
- LLM 调用节点
- 数据处理节点
- 条件判断节点
- 循环控制节点

### 3. 输出节点
- 文本输出
- 文件生成
- 数据库写入
- 报告生成

### 4. 集成节点
- 外部API调用
- 数据库查询
- 消息推送

## 版本控制机制

### 版本命名规则
- 主版本号.次版本号.修订号 (如 1.0.0)
- 支持自定义版本名称

### 版本比较
- 支持可视化版本差异对比
- 支持版本回滚操作

### 版本发布流程
1. 开发阶段：草稿状态
2. 测试阶段：内部测试版本
3. 发布阶段：正式版本上线

## 执行引擎

### 执行模型
采用事件驱动的执行模型，支持：
- 顺序执行
- 并行执行
- 条件分支执行
- 循环执行

### 执行上下文
维护流程执行过程中的上下文信息：
- 输入数据
- 中间结果
- 执行状态
- 错误信息

### 错误处理
- 节点执行错误捕获
- 错误传播机制
- 重试机制
- 回滚机制

## WebSocket 通信协议

### 连接建立
```
WebSocket URL: ws://host:port/workflow/chat/{workflow_id}?chat_id={chat_id}
Headers: Authorization: Bearer {jwt_token}
```

### 消息格式
```json
{
  "type": "消息类型",
  "data": { /* 消息数据 */ },
  "trace_id": "追踪ID"
}
```

### 消息类型
- `start`: 开始执行
- `progress`: 执行进度
- `result`: 执行结果
- `error`: 执行错误
- `end`: 执行结束

## 报告生成功能

### 功能描述
支持生成Word格式的报告文档。

### 核心接口
- **获取报告文件**: `/workflow/report/file` (GET)
- **复制报告模板**: `/workflow/report/copy` (POST)
- **上传报告文件**: `/workflow/report/callback` (POST)

### 实现机制
1. 模板存储：使用 MinIO 存储报告模板文件
2. 模板编辑：集成 Office Online 进行在线编辑
3. 模板渲染：执行时根据数据渲染模板生成报告

## 权限控制

### 访问权限类型
- **读取权限**: 查看流程信息
- **写入权限**: 编辑流程定义
- **执行权限**: 执行流程
- **管理权限**: 发布/下线流程

### 权限验证
1. 用户身份验证
2. 角色权限检查
3. 资源访问控制
4. 操作权限验证

## 性能优化

### 执行优化
- 节点结果缓存
- 并行执行优化
- 资源预加载

### 存储优化
- 流程数据压缩
- 版本数据去重
- 历史数据归档

### 网络优化
- WebSocket 长连接复用
- 消息批量发送
- 数据流控制

## 监控与日志

### 执行监控
- 流程执行状态跟踪
- 节点执行时间统计
- 资源使用情况监控

### 日志记录
- 执行日志：记录流程执行过程
- 错误日志：记录执行错误信息
- 审计日志：记录用户操作行为

## API 接口清单

### 流程管理接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/workflow/create` | POST | 创建流程 |
| `/workflow/get_one_flow/{flow_id}` | GET | 获取流程详情 |
| `/workflow/update/{flow_id}` | PATCH | 更新流程 |
| `/workflow/list` | GET | 流程列表 |
| `/workflow/status` | PATCH | 更新流程状态 |

### 版本管理接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/workflow/versions` | GET | 获取版本列表 |
| `/workflow/versions` | POST | 创建新版本 |
| `/workflow/versions/{version_id}` | PUT | 更新版本 |
| `/workflow/versions/{version_id}` | DELETE | 删除版本 |
| `/workflow/versions/{version_id}` | GET | 获取版本详情 |
| `/workflow/change_version` | POST | 切换当前版本 |

### 执行接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/workflow/chat/{workflow_id}` | WebSocket | 实时执行流程 |
| `/workflow/run_once` | POST | 单节点执行 |

### 报告接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/workflow/report/file` | GET | 获取报告文件 |
| `/workflow/report/copy` | POST | 复制报告模板 |
| `/workflow/report/callback` | POST | 上传报告文件 |

## 配置项

### 相关配置
```yaml
# 工作流相关配置
workflow:
  max_nodes: 100          # 最大节点数限制
  max_execution_time: 300  # 最大执行时间（秒）
  cache_ttl: 3600          # 缓存过期时间（秒）
```

## 扩展性设计

### 插件化节点
支持自定义节点类型开发：
1. 节点定义接口
2. 节点执行逻辑
3. 节点配置界面

### 流程模板
支持流程模板创建和复用：
1. 模板市场
2. 模板导入导出
3. 模板版本管理

## 错误处理

### 常见错误类型
- 流程定义错误
- 节点执行错误
- 权限验证错误
- 系统资源错误

### 错误恢复机制
- 自动重试
- 手动恢复
- 状态回滚
- 错误隔离

## 测试策略

### 单元测试
- 节点执行逻辑测试
- 流程解析测试
- 版本管理测试

### 集成测试
- 完整流程执行测试
- WebSocket通信测试
- 权限控制测试

### 性能测试
- 大流程执行性能
- 并发执行能力
- 资源使用监控

---

## 流程图

详细的流程图和脑图已拆分到独立文件，便于维护：

- [工作流执行流程图](../diagrams/workflow/execution_flow.md)
- [工作流版本管理流程图](../diagrams/workflow/version_flow.md)
- [节点执行时序图](../diagrams/workflow/sequence.md)
- [节点数据流图](../diagrams/workflow/node_dataflow.md)
- [工作流模块脑图](../diagrams/workflow/mindmap.md)