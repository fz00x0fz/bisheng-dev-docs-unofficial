# Bisheng 配置说明文档

## 概述

Bisheng 平台通过配置文件来管理系统各项功能和组件的行为。配置文件采用 YAML 格式，支持环境变量注入和动态配置更新。

## 配置文件位置

主配置文件位于：
```
docker/bisheng/config/config.yaml
```

## 配置文件结构

```yaml
database_url:               # 数据库连接配置
redis_url:                  # Redis缓存配置
celery_redis_url:           # Celery消息队列配置
celery_task:                # Celery任务路由配置
vector_stores:              # 向量存储配置
object_storage:             # 对象存储配置
environment:                # 环境配置
logger_conf:                # 日志配置
password_conf:              # 密码安全配置
llm:                       # LLM配置
```

## 数据库配置

### database_url
配置 MySQL 数据库连接信息。

```yaml
database_url: "mysql+pymysql://root:password@mysql:3306/bisheng?charset=utf8mb4"
```

**配置项说明:**
- `mysql+pymysql`: 数据库驱动
- `root`: 数据库用户名
- `password`: 数据库密码（支持加密）
- `mysql`: 数据库主机
- `3306`: 数据库端口
- `bisheng`: 数据库名
- `charset=utf8mb4`: 字符集

**密码加密:**
数据库密码支持加密存储，加密方法参考官方文档。

## 缓存配置

### redis_url
配置 Redis 缓存连接信息。

```yaml
# 普通模式
redis_url: "redis://redis:6379/1"

# 集群模式
redis_url:
  mode: "cluster"
  startup_nodes:
    - {"host": "redis-node1", "port": 6001}
    - {"host": "redis-node2", "port": 6002}
  password: "redis_password"

# 哨兵模式
redis_url:
  mode: "sentinel"
  sentinel_hosts: [("sentinel1", 26379), ("sentinel2", 26379)]
  sentinel_master: "mymaster"
  sentinel_password: "sentinel_password"
  db: 1
```

## Celery 配置

### celery_redis_url
配置 Celery 消息队列的 Redis 连接。

```yaml
celery_redis_url: "redis://redis:6379/2"
```

### celery_task
配置 Celery 任务路由。

```yaml
celery_task:
  task_routers:
    bisheng.worker.knowledge.*:  # 知识库文件处理相关任务
      queue: knowledge_celery
    bisheng.worker.workflow.*:   # 工作流相关任务
      queue: workflow_celery
```

## 向量存储配置

### vector_stores
配置 Milvus 和 Elasticsearch 向量存储。

```yaml
vector_stores:
  milvus:
    connection_args: !env ${BS_MILVUS_CONNECTION_ARGS}  # 从环境变量获取
    is_partition: !env ${BS_MILVUS_IS_PARTITION}
    partition_suffix: !env ${BS_MILVUS_PARTITION_SUFFIX}
  elasticsearch:
    url: !env ${BS_ELASTICSEARCH_URL}
    ssl_verify: !env ${BS_ELASTICSEARCH_SSL_VERIFY}
```

**环境变量配置示例:**
```bash
BS_MILVUS_CONNECTION_ARGS='{"host":"milvus","port":"19530","user":"","password":"","secure":false}'
BS_MILVUS_IS_PARTITION='true'
BS_MILVUS_PARTITION_SUFFIX='1'
BS_ELASTICSEARCH_URL='http://elasticsearch:9200'
BS_ELASTICSEARCH_SSL_VERIFY='{}'
```

## 对象存储配置

### object_storage
配置 MinIO 对象存储。

```yaml
object_storage:
  type: minio
  minio:
    schema: !env ${BS_MINIO_SCHEMA}
    cert_check: !env ${BS_MINIO_CERT_CHECK}
    endpoint: !env ${BS_MINIO_ENDPOINT}
    sharepoint: !env ${BS_MINIO_SHAREPOINT}
    access_key: !env ${BS_MINIO_ACCESS_KEY}
    secret_key: !env ${BS_MINIO_SECRET_KEY}
    public_bucket: 'bisheng'   # 公共bucket
    tmp_bucket: 'tmp-dir'      # 临时bucket
```

**环境变量配置示例:**
```bash
BS_MINIO_SCHEMA='false'
BS_MINIO_CERT_CHECK='false'
BS_MINIO_ENDPOINT='minio:9000'
BS_MINIO_SHAREPOINT='minio:9000'
BS_MINIO_ACCESS_KEY='minioadmin'
BS_MINIO_SECRET_KEY='minioadmin'
```

## 环境配置

### environment
配置系统运行环境。

```yaml
environment:
  env: dev                    # 运行环境 (dev/test/prod)
  uns_support: ['png','jpg','jpeg','bmp','doc', 'docx', 'ppt', 'pptx', 'xls', 'xlsx', 'txt', 'md', 'html', 'pdf', 'csv', 'tiff']  # 支持的文件类型
```

## 日志配置

### logger_conf
配置系统日志行为。

```yaml
logger_conf:
  level: DEBUG                # 默认日志级别
  format: '<level>[{time:YYYY-MM-DD HH:mm:ss.SSSSSS}] [{level.name} process-{process.id}-{thread.id} {name}:{line}]</level> - <level>trace={extra[trace_id]} {message}</level>'
  handlers:
    - sink: "/app/data/bisheng.log"  # 日志文件路径
      level: INFO
      format: '<level>[{time:YYYY-MM-DD HH:mm:ss.SSSSSS}] [{level.name} process-{process.id}-{thread.id} {name}:{line}]</level> - <level>trace={extra[trace_id]} {message}</level>'
      rotation: "00:00"       # 每天切割
      retention: "3 Days"     # 保留3天
      enqueue: true
    - sink: "/app/data/statistic.log"
      level: INFO
      filter: "lambda record: record['level'].name == 'INFO' and record['message'].startswith('k=s')"
      format: "[{time:YYYY-MM-DD HH:mm:ss.SSSSSS}]|{level}|BISHENG|{extra[trace_id]}||{process.id}|{thread.id}|||#EX_ERR:POS={name},line {line},ERR=500,EMSG={message}"
      rotation: "00:00"
      retention: "3 Days"
      enqueue: true
```

## 密码安全配置

### password_conf
配置用户密码安全策略。

```yaml
password_conf:
  login_error_time_window: 30    # 登录错误时间窗口（分钟）
  max_error_times: 5             # 最大错误次数
  password_valid_period: 90      # 密码有效期（天）
```

## LLM 配置

### llm
配置大语言模型相关参数。

```yaml
llm:
  openai:
    api_key: !env ${OPENAI_API_KEY}
    base_url: "https://api.openai.com/v1"
    timeout: 30
    max_retries: 3
    models:
      - name: "gpt-3.5-turbo"
        temperature: 0.7
        max_tokens: 2048
      - name: "gpt-4"
        temperature: 0.5
        max_tokens: 4096

  anthropic:
    api_key: !env ${ANTHROPIC_API_KEY}
    base_url: "https://api.anthropic.com"
    timeout: 30
    max_retries: 3
    models:
      - name: "claude-3-haiku"
        temperature: 0.7
        max_tokens: 4096

  cache:
    enabled: true
    ttl: 3600
    max_size: 10000
```

## 验证码配置

### 验证码相关配置
```yaml
use_captcha: true  # 是否启用验证码
```

## 系统登录配置

### 系统登录方式
```yaml
system_login_method:
  bisheng_pro: false  # 是否启用SSO
  admin_username: "admin"  # 默认管理员用户名
```

## 环境变量注入

配置文件支持通过环境变量注入值，使用 `!env` 标签：

```yaml
database_url: !env ${DATABASE_URL}
redis_url: !env ${REDIS_URL}
```

## 配置优先级

配置加载优先级（从高到低）：
1. 环境变量
2. 配置文件
3. 默认值

## 配置热更新

部分配置支持运行时动态更新：
- 日志配置
- LLM配置
- 密码策略配置

## 安全配置建议

### 生产环境配置建议
1. **数据库密码加密**: 使用加密方式存储数据库密码
2. **Redis密码**: 配置Redis访问密码
3. **API密钥保护**: LLM API密钥通过环境变量注入
4. **日志敏感信息过滤**: 避免日志中记录敏感信息
5. **文件权限**: 配置文件设置适当访问权限

### 开发环境配置示例
```yaml
environment:
  env: dev

logger_conf:
  level: DEBUG

password_conf:
  login_error_time_window: 5
  max_error_times: 10
  password_valid_period: 365
```

## 配置验证

系统启动时会自动验证配置的正确性：
- 数据库连接测试
- Redis连接测试
- 必要配置项检查
- 配置格式验证

## 常见配置问题

### 1. 数据库连接失败
**问题**: 系统无法连接到数据库
**解决方案**:
- 检查数据库URL配置是否正确
- 验证数据库服务是否正常运行
- 确认网络连接是否通畅

### 2. Redis连接超时
**问题**: Redis连接超时或认证失败
**解决方案**:
- 检查Redis URL配置
- 验证Redis服务状态
- 确认密码是否正确

### 3. LLM API调用失败
**问题**: LLM调用返回认证错误
**解决方案**:
- 检查API密钥是否正确配置
- 验证环境变量是否正确注入
- 确认提供商服务是否正常

## 配置备份和恢复

### 备份配置
```bash
# 备份配置文件
cp config.yaml config.yaml.backup.$(date +%Y%m%d)
```

### 恢复配置
```bash
# 恢复配置文件
cp config.yaml.backup.20231201 config.yaml
```

## 多环境配置管理

### 不同环境配置文件
```
config/
├── config.dev.yaml      # 开发环境
├── config.test.yaml     # 测试环境
├── config.prod.yaml     # 生产环境
└── config.yaml         # 默认配置
```

### 环境切换
通过环境变量指定配置文件：
```bash
export BISHENG_ENV=prod
```

## 配置监控

### 配置变更监控
- 记录配置文件修改历史
- 监控关键配置项变更
- 配置变更通知机制

### 性能相关配置监控
- 数据库连接池使用情况
- Redis内存使用情况
- LLM调用配额使用情况