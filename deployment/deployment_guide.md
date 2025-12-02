# Bisheng 部署和运行指南

## 系统要求

### 硬件要求
- **最低配置**:
  - CPU: 4核虚拟CPU
  - 内存: 16GB RAM
  - 存储: 100GB 可用磁盘空间

- **推荐配置**:
  - CPU: 8核虚拟CPU
  - 内存: 32GB RAM
  - 存储: 500GB 可用磁盘空间

### 软件要求
- Docker 19.03.9 或更高版本
- Docker Compose 1.25.1 或更高版本
- 操作系统: Linux (推荐 Ubuntu 18.04+/CentOS 7+) 或 Windows 10+/macOS

## 部署方式

### 1. Docker Compose 部署（推荐）

#### 下载代码
```bash
# 克隆项目代码
git clone https://github.com/dataelement/bisheng.git

# 进入项目目录
cd bisheng/docker
```

#### 启动服务
```bash
# 启动所有服务
docker compose -f docker-compose.yml -p bisheng up -d
```

#### 验证部署
```bash
# 查看服务状态
docker compose -p bisheng ps

# 查看服务日志
docker compose -p bisheng logs -f
```

### 2. 手动部署

#### 环境准备
1. 安装 MySQL 8.0
2. 安装 Redis 7.0.4
3. 安装 Elasticsearch 8.12.0
4. 安装 Milvus 2.5.10
5. 安装 MinIO

#### 后端服务部署
```bash
# 进入后端目录
cd src/backend

# 安装依赖
pip install -r requirements.txt

# 启动API服务
python -m bisheng.main

# 启动Worker服务
celery -A bisheng.run_celery worker --loglevel=info
```

#### 前端服务部署
```bash
# 进入前端目录
cd src/frontend/platform

# 安装依赖
npm install

# 构建项目
npm run build

# 启动服务
npm run start
```

## Docker Compose 配置详解

### 主配置文件
```yaml
# docker-compose.yml
services:
  mysql:
    container_name: bisheng-mysql
    image: mysql:8.0
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: "1234"
      MYSQL_DATABASE: bisheng
      TZ: Asia/Shanghai
    volumes:
      - ./mysql/conf/my.cnf:/etc/mysql/my.cnf
      - ./mysql/data:/var/lib/mysql

  redis:
    container_name: bisheng-redis
    image: redis:7.0.4
    ports:
      - "6379:6379"
    environment:
      TZ: Asia/Shanghai
    volumes:
      - ./data/redis:/data
      - ./redis/redis.conf:/etc/redis.conf

  backend:
    container_name: bisheng-backend
    image: dataelement/bisheng-backend:v2.2.0
    ports:
      - "7860:7860"
    environment:
      TZ: Asia/Shanghai
      BS_MILVUS_CONNECTION_ARGS: '{"host":"milvus","port":"19530","user":"","password":"","secure":false}'
    volumes:
      - ./bisheng/config/config.yaml:/app/bisheng/config.yaml
      - ./bisheng/entrypoint.sh:/app/entrypoint.sh
      - ./data/bisheng:/app/data

  frontend:
    container_name: bisheng-frontend
    image: dataelement/bisheng-frontend:v2.2.0
    ports:
      - "3001:3001"
    environment:
      TZ: Asia/Shanghai
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/conf.d:/etc/nginx/conf.d

  elasticsearch:
    container_name: bisheng-es
    image: docker.io/bitnamilegacy/elasticsearch:8.12.0
    user: root
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      TZ: Asia/Shanghai
    volumes:
      - ./data/es:/bitnami/elasticsearch/data

  etcd:
    container_name: bisheng-milvus-etcd
    image: quay.io/coreos/etcd:v3.5.5

  minio:
    container_name: bisheng-milvus-minio
    image: minio/minio:RELEASE.2023-03-20T20-16-18Z

  milvus:
    container_name: bisheng-milvus-standalone
    image: milvusdb/milvus:v2.5.10
```

## 初始化配置

### 1. 数据库初始化
系统启动时会自动初始化数据库表结构。

### 2. 管理员账户
第一个注册的用户将自动成为系统管理员。

### 3. 默认配置
系统提供默认配置，可根据需要进行调整。

## 服务访问

### Web 界面
- **地址**: http://localhost:3001
- **默认端口**: 3001
- **首次访问**: 需要注册第一个管理员账户

### API 接口
- **地址**: http://localhost:7860/api/v1
- **文档**: http://localhost:7860/docs

### 数据库
- **地址**: localhost:3306
- **用户名**: root
- **密码**: 1234（默认）

### Redis
- **地址**: localhost:6379
- **数据库**: 1（应用缓存），2（Celery队列）

## 性能调优

### 1. 数据库优化
```bash
# MySQL 配置优化
[mysqld]
innodb_buffer_pool_size = 2G
max_connections = 200
query_cache_size = 128M
```

### 2. Redis 优化
```bash
# Redis 配置优化
maxmemory 2gb
maxmemory-policy allkeys-lru
```

### 3. Elasticsearch 优化
```bash
# JVM 堆内存设置
-Xms2g
-Xmx2g
```

### 4. Milvus 优化
```bash
# Milvus 配置优化
cache:
  cache_size: 4GB
```

## 高可用部署

### 1. 数据库主从复制
配置 MySQL 主从复制提高数据库可用性。

### 2. Redis 集群
使用 Redis 集群模式提高缓存可用性。

### 3. 负载均衡
使用 Nginx 或 HAProxy 实现负载均衡。

### 4. 容器编排
使用 Kubernetes 进行容器编排和管理。

## 监控和日志

### 1. 系统监控
- CPU 使用率
- 内存使用率
- 磁盘使用率
- 网络流量

### 2. 服务监控
- API 响应时间
- 数据库查询性能
- Redis 缓存命中率
- Elasticsearch 搜索性能

### 3. 日志管理
```bash
# 查看后端日志
docker logs bisheng-backend

# 查看前端日志
docker logs bisheng-frontend

# 查看数据库日志
docker logs bisheng-mysql
```

## 备份和恢复

### 1. 数据库备份
```bash
# MySQL 备份
mysqldump -u root -p bisheng > bisheng_backup_$(date +%Y%m%d).sql

# MySQL 恢复
mysql -u root -p bisheng < bisheng_backup_20231201.sql
```

### 2. 配置文件备份
```bash
# 备份配置文件
cp -r bisheng/config bisheng/config.backup.$(date +%Y%m%d)
```

### 3. 数据文件备份
```bash
# 备份数据文件
tar -czf bisheng_data_$(date +%Y%m%d).tar.gz ./data
```

## 故障排除

### 1. 服务启动失败
```bash
# 查看服务状态
docker compose -p bisheng ps

# 查看服务日志
docker compose -p bisheng logs service_name

# 重启服务
docker compose -p bisheng restart service_name
```

### 2. 数据库连接问题
```bash
# 测试数据库连接
docker exec -it bisheng-mysql mysql -u root -p -e "SHOW DATABASES;"

# 检查网络连接
docker exec -it bisheng-backend ping mysql
```

### 3. 内存不足
```bash
# 查看容器资源使用
docker stats

# 调整容器资源限制
docker update --memory=4g bisheng-backend
```

## 安全配置

### 1. 修改默认密码
```bash
# 修改数据库密码
docker exec -it bisheng-mysql mysqladmin -u root -p1234 password new_password

# 修改Redis密码
# 在 redis.conf 中添加 requirepass your_password
```

### 2. 启用 HTTPS
```nginx
# Nginx HTTPS 配置
server {
    listen 443 ssl;
    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;
}
```

### 3. 访问控制
```bash
# 防火墙配置
ufw allow 3001/tcp
ufw allow 7860/tcp
ufw deny 3306/tcp
```

## 升级指南

### 1. 备份当前版本
```bash
# 备份配置和数据
cp -r bisheng bisheng_backup_$(date +%Y%m%d)
```

### 2. 拉取新版本
```bash
# 拉取最新代码
git pull origin main

# 重新构建镜像
docker compose -p bisheng build
```

### 3. 执行升级
```bash
# 停止当前服务
docker compose -p bisheng down

# 启动新版本服务
docker compose -p bisheng up -d
```

### 4. 验证升级
```bash
# 检查服务状态
docker compose -p bisheng ps

# 验证功能
curl http://localhost:7860/health
```

## 常见问题

### 1. 端口被占用
```bash
# 查看端口占用
netstat -tlnp | grep :3001

# 修改端口映射
# 在 docker-compose.yml 中修改 ports 配置
```

### 2. 磁盘空间不足
```bash
# 查看磁盘使用情况
df -h

# 清理日志文件
docker system prune -a
```

### 3. 内存溢出
```bash
# 查看容器内存使用
docker stats

# 调整内存限制
docker update --memory=8g bisheng-backend
```

## 最佳实践

### 1. 生产环境部署
- 使用独立服务器部署
- 配置监控和告警
- 定期备份数据
- 启用安全防护

### 2. 性能优化
- 合理配置资源限制
- 使用 SSD 存储
- 优化数据库索引
- 启用缓存机制

### 3. 安全加固
- 修改默认密码
- 启用 HTTPS
- 配置防火墙
- 定期更新组件