# 聊天模块详细设计文档

## 模块概述

聊天模块是 Bisheng 平台实现人机交互的核心组件，提供了实时聊天通信、多轮对话管理、上下文保持等功能。该模块基于 WebSocket 技术实现，支持与工作流、助手等多种AI应用的集成。

## 模块结构

```
bisheng/chat/               # 聊天模块核心
├── manager.py             # 聊天管理器（ChatManager）
├── client.py              # 客户端处理
├── handlers.py            # 消息处理器
├── types.py               # 类型定义（WorkType枚举）
└── utils.py               # 工具类

bisheng/chat_session/       # 聊天会话模块
├── api/
│   └── router.py          # 会话API路由
└── domain/
    └── services/          # 会话服务

bisheng/api/v1/chat.py      # 聊天API接口

bisheng/database/models/
├── message.py             # 消息数据模型（ChatMessage）
└── session.py             # 会话数据模型
```

## 核心概念

### 1. 会话 (Session)
会话是聊天的基本单位，代表一次完整的对话交互过程，包含对话的历史记录和上下文信息。

### 2. 消息 (Message)
消息是会话中的基本交互单元，包含发送者、内容、时间戳等信息。

### 3. 聊天类型 (Chat Type)
- **普通聊天**: 基础的文本聊天功能
- **工作流聊天**: 与工作流集成的聊天交互
- **助手聊天**: 与AI助手的交互

### 4. 连接管理
管理客户端与服务器之间的 WebSocket 连接，确保通信的稳定性和实时性。

## 数据模型设计

### 会话表 (Session)

```python
class Session(BaseSQLModel):
    id: Optional[str] = Field(default=None, primary_key=True)
    user_id: int = Field(index=True)
    chat_id: str = Field(index=True)
    flow_id: Optional[str] = Field(default=None, index=True)
    chat_type: str = Field(max_length=50)  # 聊天类型
    topic: Optional[str] = Field(default=None, max_length=200)  # 会话主题
    logo: Optional[str] = Field(default=None, max_length=500)   # 会话图标
    create_time: Optional[datetime] = Field(default_factory=datetime.utcnow)
    update_time: Optional[datetime] = Field(default_factory=datetime.utcnow)
```

### 消息表 (Message)

```python
class Message(BaseSQLModel):
    id: Optional[str] = Field(default=None, primary_key=True)
    chat_id: str = Field(index=True)
    flow_id: Optional[str] = Field(default=None, index=True)
    chat_type: str = Field(max_length=50)
    user_id: int = Field()
    is_bot: bool = Field(default=False)  # 是否为机器人消息
    message: Optional[str] = Field(default=None)  # 消息内容
    extra: Optional[str] = Field(default=None)   # 额外信息（JSON格式）
    source: int = Field(default=0)  # 消息来源
    create_time: Optional[datetime] = Field(default_factory=datetime.utcnow)
```

## 核心功能

### 1. 实时聊天通信

#### 功能描述
通过 WebSocket 实现客户端与服务器之间的实时双向通信。

#### 连接建立流程
1. 客户端发起 WebSocket 连接请求
2. 服务器验证 JWT Token 身份认证
3. 建立连接并初始化聊天上下文
4. 开始实时消息传输

#### WebSocket 连接地址
```
ws://host:port/chat/{flow_id}?chat_id={chat_id}
Headers: Authorization: Bearer {jwt_token}
```

#### 消息格式
```json
{
  "type": "消息类型",
  "data": { /* 消息数据 */ },
  "trace_id": "追踪ID"
}
```

#### 消息类型定义
- `start`: 开始聊天
- `message`: 普通消息
- `progress`: 处理进度
- `result`: 处理结果
- `error`: 错误信息
- `end`: 结束聊天
- `interrupt`: 中断信号

### 2. 多轮对话管理

#### 功能描述
维护对话的历史记录和上下文状态，支持多轮交互。

#### 上下文管理
1. **历史消息存储**: 将对话历史持久化存储
2. **上下文构建**: 根据历史消息构建对话上下文
3. **状态保持**: 在多次交互中保持对话状态

#### 会话生命周期
1. **创建**: 用户发起新对话
2. **活跃**: 正在进行的对话交互
3. **暂停**: 用户暂时离开的对话
4. **结束**: 对话明确结束或超时

### 3. 消息处理

#### 功能描述
处理不同类型的消息，包括文本、文件、命令等。

#### 消息处理流程
1. 接收客户端消息
2. 消息类型识别
3. 消息内容解析
4. 业务逻辑处理
5. 响应消息构造
6. 发送响应给客户端

#### 支持的消息类型
- **文本消息**: 普通文本内容
- **文件消息**: 文件上传和处理
- **命令消息**: 系统命令和控制指令
- **状态消息**: 系统状态和进度信息

### 4. 会话管理

#### 功能描述
管理系统中的会话资源，包括创建、查询、删除等操作。

#### 会话操作接口
- **创建会话**: `/chat/session/create` (POST)
- **获取会话**: `/chat/session/{session_id}` (GET)
- **会话列表**: `/chat/session/list` (GET)
- **删除会话**: `/chat/session/{session_id}` (DELETE)
- **更新会话**: `/chat/session/{session_id}` (PUT)

#### 会话分页查询
支持按用户、时间、类型等条件进行会话分页查询。

## 聊天管理器 (ChatManager)

### 功能职责
聊天管理器是聊天模块的核心组件，负责管理所有活跃的聊天连接和会话。

### 核心方法
```python
class ChatManager:
    async def dispatch_client(self, websocket, flow_id, chat_id, user, chat_type, websocket_instance):
        """分发客户端连接"""
        pass

    async def send_message(self, chat_id, message):
        """发送消息到指定会话"""
        pass

    async def disconnect_client(self, chat_id):
        """断开客户端连接"""
        pass

    async def get_active_sessions(self):
        """获取活跃会话列表"""
        pass
```

### 连接管理
1. **连接注册**: 将新连接注册到管理器
2. **心跳检测**: 定期检测连接活性
3. **异常处理**: 处理连接异常断开
4. **资源清理**: 清理无效连接和资源

## 消息处理器 (MessageHandler)

### 功能职责
处理具体的聊天消息，执行相应的业务逻辑。

### 核心方法
```python
class MessageHandler:
    async def handle_message(self, chat_id, message_data):
        """处理聊天消息"""
        pass

    async def process_text_message(self, chat_id, text):
        """处理文本消息"""
        pass

    async def process_command_message(self, chat_id, command):
        """处理命令消息"""
        pass
```

### 消息路由
根据不同消息类型路由到相应的处理函数：
1. 文本消息 → 文本处理逻辑
2. 命令消息 → 命令执行逻辑
3. 文件消息 → 文件处理逻辑

## 会话状态管理

### 状态定义
- **初始化**: 会话刚创建
- **进行中**: 正在处理用户请求
- **等待输入**: 等待用户进一步输入
- **已完成**: 处理完成
- **已中断**: 用户中断处理
- **已结束**: 会话结束

### 状态转换
```
初始化 → 进行中 → 等待输入 ↔ 进行中 → 已完成 → 已结束
              ↘ 已中断 ↗
```

### 状态持久化
将会话状态信息存储到数据库，支持会话恢复和状态查询。

## 多聊天类型支持

### 1. 普通聊天 (NORMAL)
基础的文本聊天功能，适用于简单问答场景。

### 2. 工作流聊天 (WORKFLOW)
与工作流集成的聊天交互，支持复杂业务流程的执行和监控。

### 3. 助手聊天 (ASSISTANT)
与AI助手的交互，提供智能化的对话体验。

### 4. Linsight聊天 (LINSIGHT)
与Linsight专家代理的交互，提供专业领域的智能服务。

## 实时通信协议

### WebSocket 生命周期
1. **连接建立**: 客户端发起连接，服务器验证身份
2. **消息交换**: 双向实时消息传输
3. **心跳维持**: 定期发送心跳包保持连接
4. **连接关闭**: 正常或异常关闭连接

### 消息传输优化
- **消息压缩**: 对大消息进行压缩传输
- **批量发送**: 将多个小消息合并发送
- **优先级控制**: 重要消息优先传输

## 错误处理与恢复

### 错误类型
- **网络错误**: 连接中断、超时等
- **认证错误**: Token过期、权限不足等
- **业务错误**: 处理逻辑错误、资源不足等
- **系统错误**: 服务器内部错误

### 错误恢复机制
1. **自动重连**: 网络断开后自动尝试重连
2. **消息重发**: 未确认消息自动重发
3. **状态恢复**: 重新连接后恢复会话状态
4. **降级处理**: 系统异常时提供降级服务

## 性能优化

### 连接优化
- **连接池**: 复用WebSocket连接
- **负载均衡**: 多实例间负载均衡
- **连接限制**: 控制最大并发连接数

### 消息处理优化
- **异步处理**: 非阻塞消息处理
- **批量处理**: 批量处理消息队列
- **缓存机制**: 热点数据缓存

### 资源管理
- **内存优化**: 及时释放无用对象
- **连接回收**: 定期清理无效连接
- **资源监控**: 实时监控资源使用情况

## 安全机制

### 身份认证
- **JWT Token**: 基于JWT的身份验证
- **Token刷新**: 自动刷新过期Token
- **权限验证**: 细粒度权限控制

### 数据安全
- **传输加密**: WebSocket TLS加密
- **数据加密**: 敏感数据加密存储
- **访问控制**: 基于角色的访问控制

### 防攻击措施
- **频率限制**: 限制消息发送频率
- **内容过滤**: 过滤恶意内容
- **连接限制**: 限制单用户连接数

## API 接口清单

### 聊天接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/chat/{flow_id}` | WebSocket | 聊天WebSocket连接 |
| `/chat/message/history` | GET | 获取聊天历史消息 |
| `/chat/message/delete` | DELETE | 删除聊天消息 |
| `/chat/interrupt` | POST | 中断聊天处理 |

### 会话接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/chat/session/create` | POST | 创建聊天会话 |
| `/chat/session/list` | GET | 获取会话列表 |
| `/chat/session/{session_id}` | GET | 获取会话详情 |
| `/chat/session/{session_id}` | PUT | 更新会话信息 |
| `/chat/session/{session_id}` | DELETE | 删除会话 |

### 统计接口

| 接口 | 方法 | 描述 |
|------|------|------|
| `/chat/statistics` | GET | 聊天统计信息 |
| `/chat/messages/count` | GET | 消息数量统计 |

## 配置项

### 相关配置
```yaml
# 聊天配置
chat:
  max_message_length: 10000     # 最大消息长度
  max_history_messages: 50      # 最大历史消息数
  heartbeat_interval: 30        # 心跳间隔（秒）
  connection_timeout: 300       # 连接超时时间（秒）
  max_concurrent_sessions: 1000 # 最大并发会话数
```

## 扩展性设计

### 插件化消息处理器
支持自定义消息处理器扩展：
1. 处理器接口定义
2. 动态加载机制
3. 配置化路由

### 多协议支持
- **WebSocket**: 主要通信协议
- **HTTP长轮询**: 兼容性备选方案
- **gRPC**: 高性能内部通信

### 多语言支持
- **国际化**: 支持多语言消息
- **本地化**: 适配不同地区习惯

## 监控与运维

### 性能监控
- **连接数**: 实时连接数量监控
- **消息吞吐**: 消息处理速率监控
- **延迟统计**: 消息传输延迟统计

### 错误监控
- **错误率**: 系统错误率统计
- **异常告警**: 异常情况及时告警
- **日志分析**: 错误日志分析

### 资源监控
- **CPU使用率**: 处理器使用情况
- **内存占用**: 内存使用情况
- **网络流量**: 网络IO统计

## 测试策略

### 单元测试
- 消息处理器测试
- 会话管理测试
- 连接管理测试

### 集成测试
- 完整聊天流程测试
- 多用户并发测试
- 异常场景测试

### 性能测试
- 高并发连接测试
- 大消息传输测试
- 长时间稳定性测试

---

## 流程图

详细的流程图和脑图已拆分到独立文件，便于维护：

- [WebSocket连接流程图](../diagrams/chat/websocket_flow.md)
- [消息处理流程图](../diagrams/chat/message_flow.md)
- [聊天会话时序图](../diagrams/chat/sequence.md)
- [消息流转数据流图](../diagrams/chat/message_dataflow.md)
- [聊天模块脑图](../diagrams/chat/mindmap.md)