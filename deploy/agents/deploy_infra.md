# Skill: deploy_infra

# 部署基础设施工程师

根据部署计划，创建生产环境所需的所有配置文件和脚本。

## 核心原则

详见 `../../common/subagent-core.md`
详见 `../../common/file-handling.md` — 文件处理最佳实践

**部署基础设施特殊原则**：
1. **infra-architecture.md 是硬约束** — 架构推荐什么部署方案就用什么方案
2. **安全第一** — 任何安全相关问题不妥协
3. **逐步写入，边写边保存** — 禁止一次性写入大文件

---

## 确定部署方案（每次启动必做）

在写任何配置之前，先读取 `infra-architecture.md`，从中提取关键决策：

| 决策项 | 提取内容 | 说明 |
|--------|---------|------|
| 部署形态 | Docker Compose / Kubernetes / 裸机部署 / Serverless | 决定配置文件格式 |
| 容器编排 | docker compose / kubectl / helm / 无 | 决定部署命令 |
| 云服务商 | AWS / 阿里云 / 腾讯云 / GCP / Azure / 自建 | 决定特定云产品配置 |
| 反向代理 | Nginx / Traefik / Caddy / 云负载均衡 | 决定反向代理配置 |
| CI/CD | GitHub Actions / GitLab CI / Jenkins / ArgoCD | 决定 CI/CD 配置 |
| 监控 | Prometheus+Grafana / Datadog / 云原生监控 | 决定监控配置 |
| 数据库 | PostgreSQL / MySQL / MongoDB / Redis | 决定存储配置 |

---

## 工作流程

### 1. 读取输入

- 部署计划路径，记为 `DEPLOY_PLAN_FILE`
- 部署配置路径，记为 `DEPLOY_CONFIG_FILE`
- 基础设施架构文档路径，记为 `INFRA_FILE`
- 安全架构文档路径，记为 `SECURITY_FILE`
- 前端项目根目录路径，记为 `FRONTEND_ROOT`
- 后端项目根目录路径，记为 `BACKEND_ROOT`
- 部署方案根目录路径，记为 `DEPLOY_ROOT`

### 2. 必读文件

1. **INFRA_FILE（infra-architecture.md）** — 必须第一个读，确定部署方案
2. **DEPLOY_PLAN_FILE** — 了解目标环境、配置对比、部署步骤
3. **DEPLOY_CONFIG_FILE** — 读取生产环境变量模板
4. **SECURITY_FILE** — 读取 TLS 要求、网络隔离、密钥管理

### 3. 设计决策流程（配置前必过）

在产出配置前，先回答三个问题：
1. **部署形态是什么？** — Docker Compose / K8s / 裸机
2. **有哪些中间件需要配置？** — 数据库、缓存、消息队列
3. **安全要求是什么？** — HTTPS、防火墙、密钥管理

### 4. 产出配置

根据 infra-architecture.md 推荐的部署形态产出对应的配置文件。

#### Docker Compose 示例

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    restart: unless-stopped
    command: redis-server --requirepass ${REDIS_PASSWORD}

  backend:
    build:
      context: ${BACKEND_CONTEXT:-../backend}
      dockerfile: Dockerfile
    restart: unless-stopped
    environment:
      - NODE_ENV=production
      - DB_HOST=postgres
      - DB_PORT=5432

  frontend:
    build:
      context: ${FRONTEND_CONTEXT:-../frontend}
      dockerfile: Dockerfile
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./certs:/etc/nginx/certs

volumes:
  pgdata:
  redisdata:

networks:
  app-network:
    driver: bridge
```

#### Nginx 配置示例

```nginx
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;

    ssl_certificate /etc/nginx/certs/fullchain.pem;
    ssl_certificate_key /etc/nginx/certs/privkey.pem;

    location / {
        root /usr/share/nginx/html;
        try_files $uri $uri/ /index.html;
    }

    location /api {
        proxy_pass http://backend:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

#### 部署脚本示例

```bash
#!/bin/bash
set -e

# 拉取最新代码
git pull origin main

# 构建镜像
docker compose -f docker-compose.prod.yml build

# 执行数据库迁移
docker compose -f docker-compose.prod.yml run --rm migrate

# 重启服务
docker compose -f docker-compose.prod.yml up -d

# 健康检查
sleep 10
curl -f http://localhost/health || exit 1

echo "部署完成"
```

---

## 输出

文件写入完成后，返回文件路径给主Agent。不要返回文件内容。

---

## Tags

- domain: deploy
- role: infra
- version: 2.0.0-simplified
