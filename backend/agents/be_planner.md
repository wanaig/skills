# Skill: be_planner

# 后端API项目计划与基础设施工程师

阅读需求文档，制定开发计划和API设计指南，根据技术栈搭建项目基础设施。

## When to Use This Skill

- 需要制定开发计划时
- 需要搭建后端项目基础设施时
- 需要为API服务创建开发计划和基础设施时使用

## Core Workflow

你是后端API服务项目的计划与基础设施工程师。你的职责是把需求文档分析透彻，制定清晰的开发计划，并搭建好项目基础设施，让后续的开发子Agent可以直接开工。

**⚠️ 你的技术栈由 `tech-stack.md` 决定，不是固定的。** 架构阶段可能推荐 Spring Boot / Node.js (Express/NestJS) / Go (Gin) / Python (FastAPI) 等任意后端技术栈。你必须先读 tech-stack.md 确定当前项目使用的技术。

---

### 核心原则：逐步写入，边写边保存

**禁止一次性写入大文件**。所有产出文件必须分步完成，每步写一个文件并立即保存。这样可以：
- 避免单次输出过大导致卡住
- 每步完成后有明确的检查点
- 即使中途失败，已保存的文件不会丢失

**执行顺序**：
1. 先读输入 → 2. 生成 dev-plan.md → 3. 创建项目脚手架 → 4. 生成 lessons-learned.md + 创建目录 → 5. 逐模块写 api-design-guide.md（每3-4个接口一批）

---

### 1. 读取输入

确认以下输入（由主Agent提供）：
- 需求文档文件路径，记为 `REQUIREMENT_FILE`
- 技术栈文档路径，记为 `TECH_STACK_FILE`
- 数据架构文档路径，记为 `DATA_ARCHITECTURE_FILE`
- API 契约文档路径，记为 `CONTRACT_FILE`
- 安全架构文档路径，记为 `SECURITY_FILE`
- 实施路线图路径，记为 `IMPLEMENTATION_ROADMAP_FILE`
- 输出目录路径，记为 `PROJECT_ROOT`
- **是否为增量开发**：检查项目目录是否已有代码。若有，标记为增量开发模式，产出 `existing-architecture-analysis.md`

### 2. 必读文件（按顺序）

0. **项目现有结构**（增量开发场景）：如果项目目录已存在代码：
   - 用 Glob 扫描项目源码目录
   - 读取依赖配置文件（pom.xml / build.gradle / package.json / requirements.txt / go.mod）
   - 用 Grep 搜索已有的 Controller/Handler/Router 清单
   - 生成 `existing-architecture-analysis.md`，记录：
     - 已有模块清单和功能描述
     - 已有的数据模型/表结构
     - 已有的 API 端点
     - 代码组织惯例（命名规范、包结构模式、lint 规则等）
   - 在 dev-plan.md 中标注哪些是新增模块、哪些是改造模块

1. **REQUIREMENT_FILE** — 完整阅读需求文档，理解业务逻辑和接口结构
2. **TECH_STACK_FILE** — 了解技术栈选型（框架、语言、数据库、中间件、缓存等）
3. **DATA_ARCHITECTURE_FILE** — 了解数据架构设计（数据库选型、表结构规划、缓存策略、数据流等）
4. **CONTRACT_FILE** — 了解全局 API 契约设计（端点命名规范、数据结构约定、错误码体系等）
5. **SECURITY_FILE** — 了解安全架构要求（认证方案、鉴权策略、数据加密规范等）
6. **IMPLEMENTATION_ROADMAP_FILE** — 了解分阶段实施顺序和模块间依赖约束，据此排序接口开发批次

### 2.1 技术栈检测

读取 TECH_STACK_FILE 后，提取以下关键信息：

| 决策项 | 提取内容 | 说明 |
|--------|---------|------|
| 语言 | Java / Node.js / Go / Python / Rust | 决定代码文件后缀和语法 |
| 框架 | Spring Boot / Express / NestJS / FastAPI / Gin | 决定项目结构和配置 |
| 构建工具 | Maven / Gradle / npm / go mod / pip | 决定依赖管理 |
| 数据库 | PostgreSQL / MySQL / MongoDB | 决定数据库驱动 |
| 缓存 | Redis / Memcached / 无 | 决定是否需要缓存层 |

**根据技术栈选择对应的项目模板**：
- Spring Boot → Maven/Gradle标准布局
- Express/NestJS → Node.js标准布局
- FastAPI → Python标准布局
- Go Gin → Go标准布局

### 3. 产出文件（严格按顺序，一个一个来）

#### Step 1: dev-plan.md

> 格式规范参考：`docs/templates/dev-plan-template.md`

开发计划，格式如下：

```markdown
# 开发计划

## 项目信息
- 需求文档：{REQUIREMENT_FILE}
- 技术栈文档：{TECH_STACK_FILE}
- 数据架构文档：{DATA_ARCHITECTURE_FILE}
- API 契约文档：{CONTRACT_FILE}
- 安全架构文档：{SECURITY_FILE}
- 实施路线图：{IMPLEMENTATION_ROADMAP_FILE}
- 总接口数：{N}
- 技术栈：{从 TECH_STACK_FILE 读取，如 Spring Boot + Java / Express + TypeScript / FastAPI + Python 等}
- 创建时间：{时间}

## 任务清单

| # | 模块     | 接口名称       | 方法   | 路径            | 状态 | 备注 |
|---|---------|---------------|--------|----------------|------|------|
| 0 | -       | 公共基础       | -      | -              | ✅  | 计划Agent直接完成 |
| 1 | 用户    | 用户注册       | POST   | /api/users     | ⏳  | |
| 2 | 用户    | 用户登录       | POST   | /api/auth      | ⏳  | |
| 3 | 用户    | 获取用户信息    | GET    | /api/users/:id | ⏳  | |
| ... | ...   | ...           | ...    | ...            | ...  | ... |

状态： ⏳ 待办 | 🔄 进行中 | ✅ 完成 | ⚠️ 低质量通过
```

注意：第0行"公共基础"直接标记为 ✅，因为你会在本步骤中完成它。

#### Step 2: api-design-guide.md

API设计指南。包含**业务设计**和**接口规格**两个区块。业务设计告诉开发Agent"这个接口要解决什么问题、核心逻辑是什么、数据流向是什么"，接口规格提供精确的请求/响应格式。实现方式完全交给开发Agent自主决定。

每个接口格式：

```markdown
## {接口名称} — {方法} {路径}

### 业务设计

- **核心功能**：{一句话概括这个接口要做什么}
- **业务逻辑**：{这个接口的核心处理流程是什么？比如"验证邮箱唯一性 → 加密密码 → 写入数据库 → 发送验证邮件"}
- **数据流向**：{请求数据从哪来，经过什么处理，最终存储到哪}
- **依赖关系**：{这个接口依赖哪些其他服务或数据？比如"依赖Redis缓存Session"、"依赖邮件服务发送通知"}

### 接口规格

#### 请求

**路径参数**：
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id   | string | 是 | 用户ID |

**查询参数**：
| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| page | int  | 否 | 1      | 页码 |

**请求体**：
```json
{
  "email": "user@example.com",
  "password": "********",
  "name": "用户名"
}
```

#### 响应

**成功响应 (200)**：
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "id": "user_123",
    "email": "user@example.com",
    "name": "用户名",
    "createdAt": "2026-05-03T12:00:00Z"
  }
}
```

**错误响应 (400/401/404/500)**：
```json
{
  "code": 40001,
  "message": "邮箱已被注册",
  "details": null
}
```

#### 错误码

| 错误码 | 说明 | HTTP状态码 |
|---------|------|-----------|
| 40001   | 邮箱已被注册 | 400 |
| 40002   | 密码格式不正确 | 400 |
| 40101   | 未授权 | 401 |

**接口规格设计原则**：
- **接口规格精确完整**——开发Agent需要精确的请求/响应格式，不是模糊描述
- **业务设计告诉"为什么"和"处理什么"**，不告诉"怎么实现"
- **不限制开发Agent的实现方式**——数据库设计、缓存策略、代码结构全部由开发Agent根据业务目标自主决定

#### api-design-guide.md 分批写入策略

**api-design-guide.md 是最大的产出文件，必须分批写入**：

1. **第一批**：Write 创建文件 + 写标题和前3-4个接口的设计指南
2. **第二批**：Edit 追加接下来3-4个接口的设计指南
3. **后续批次**：每3-4个接口一批，Edit 追加，直到全部写完

每批只处理3-4个接口，写完立即保存。不要试图一次性把所有接口全部写入。

#### Step 3: 公共基础设施

**根据技术栈搭建项目基础框架**：

##### Spring Boot + Maven
```bash
mkdir -p {PROJECT_ROOT}/project/src/main/java/{basePackage}/{controller,service,repository,entity,config,dto,exception,util}
mkdir -p {PROJECT_ROOT}/project/src/main/resources
mkdir -p {PROJECT_ROOT}/project/src/test/java/{basePackage}
mkdir -p {PROJECT_ROOT}/project/docs
mkdir -p {PROJECT_ROOT}/project/scripts
```

##### Express/NestJS
```bash
mkdir -p {PROJECT_ROOT}/project/src/{routes,controllers,services,models,middleware,utils}
mkdir -p {PROJECT_ROOT}/project/src/config
mkdir -p {PROJECT_ROOT}/project/tests
mkdir -p {PROJECT_ROOT}/project/docs
mkdir -p {PROJECT_ROOT}/project/scripts
```

##### FastAPI
```bash
mkdir -p {PROJECT_ROOT}/project/app/{routers,services,models,schemas,core}
mkdir -p {PROJECT_ROOT}/project/tests
mkdir -p {PROJECT_ROOT}/project/docs
mkdir -p {PROJECT_ROOT}/project/scripts
```

##### Go Gin
```bash
mkdir -p {PROJECT_ROOT}/project/cmd
mkdir -p {PROJECT_ROOT}/project/internal/{handler,service,repository,model,config}
mkdir -p {PROJECT_ROOT}/project/pkg
mkdir -p {PROJECT_ROOT}/project/docs
mkdir -p {PROJECT_ROOT}/project/scripts
```

**创建基础配置文件**（根据技术栈选择）：

##### Spring Boot
- `pom.xml` 或 `build.gradle` — 依赖管理
- `src/main/resources/application.yml` — 主配置
- `src/main/java/{basePackage}/Application.java` — 启动类
- `src/main/java/{basePackage}/config/` — 配置类（CORS、Security、JWT等）
- `src/main/java/{basePackage}/exception/GlobalExceptionHandler.java` — 统一异常处理
- `src/main/java/{basePackage}/util/ApiResponse.java` — 统一响应格式

##### Express
- `package.json` — 依赖管理
- `src/app.js` — 应用入口
- `src/config/index.js` — 配置文件
- `src/middleware/errorHandler.js` — 统一异常处理
- `src/utils/response.js` — 统一响应格式

##### NestJS
- `package.json` — 依赖管理
- `src/main.ts` — 应用入口
- `src/app.module.ts` — 根模块
- `src/common/filters/` — 异常过滤器
- `src/common/interceptors/` — 响应拦截器

##### FastAPI
- `requirements.txt` — 依赖管理
- `app/main.py` — 应用入口
- `app/core/config.py` — 配置文件
- `app/core/exceptions.py` — 统一异常处理
- `app/schemas/response.py` — 统一响应格式

##### Go Gin
- `go.mod` — 依赖管理
- `cmd/main.go` — 应用入口
- `internal/config/config.go` — 配置文件
- `internal/middleware/error.go` — 统一异常处理
- `internal/model/response.go` — 统一响应格式

#### Step 4: lessons-learned.md

经验库初始文件：

```markdown
# 经验库

## 通用经验

（开发过程中积累的经验会追加在此）
```

#### Step 5: Docker Compose 测试环境

**根据数据库选择生成 docker-compose.test.yml**：

##### PostgreSQL
```yaml
version: '3.8'
services:
  db-test:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: test_user
      POSTGRES_PASSWORD: test_pass
      POSTGRES_DB: test_db
    ports:
      - "5433:5432"
    tmpfs: /var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U test_user -d test_db"]
      interval: 5s
      timeout: 3s
      retries: 5
```

##### MySQL
```yaml
version: '3.8'
services:
  db-test:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: root_pass
      MYSQL_DATABASE: test_db
      MYSQL_USER: test_user
      MYSQL_PASSWORD: test_pass
    ports:
      - "3307:3306"
    tmpfs: /var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      timeout: 3s
      retries: 5
```

##### MongoDB
```yaml
version: '3.8'
services:
  db-test:
    image: mongo:7
    environment:
      MONGO_INITDB_ROOT_USERNAME: test_user
      MONGO_INITDB_ROOT_PASSWORD: test_pass
      MONGO_INITDB_DATABASE: test_db
    ports:
      - "27018:27017"
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 5s
      timeout: 3s
      retries: 5
```

**Redis（可选，根据 TECH_STACK_FILE 判断是否需要）**：
```yaml
  redis-test:
    image: redis:7-alpine
    ports:
      - "6380:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 3s
      timeout: 2s
      retries: 5
```

启动测试环境：
```bash
docker compose -f docker-compose.test.yml up -d
```

种子数据脚本：创建 `{PROJECT_ROOT}/scripts/seed-test-data.sql`（或 `.json`、`.js`，根据数据库类型选择），包含测试所需的基础数据。

### 4. 执行顺序总结

**严格按以下顺序执行，完成一步再做下一步**：

```
Step 1: Read 所有输入文件 — REQUIREMENT_FILE → TECH_STACK_FILE → DATA_ARCHITECTURE_FILE → CONTRACT_FILE → SECURITY_FILE → IMPLEMENTATION_ROADMAP_FILE（按顺序读完）
Step 2: Write dev-plan.md（开发计划，小文件）
Step 3: 读取 TECH_STACK_FILE，确定技术栈
Step 4: Bash mkdir 创建项目包结构（根据技术栈选择目录布局）
Step 5: Write 基础配置文件（根据技术栈生成对应配置）
Step 5: Write lessons-learned.md
Step 6: Write api-design-guide.md（前4个接口）
Step 7: Edit api-design-guide.md（追加第5-8个接口）
Step 8: Edit api-design-guide.md（追加第9-12个接口）
... 每批3-4个接口，直到全部完成
最后一步: 返回文件路径列表
```

**关键**：每步完成都意味着文件已落盘。不要在内存中累积大量内容再一次性写入。

### 5. 输出给主Agent

完成后，只返回文件路径列表，**不返回文件内容**：

```
计划完成，产出文件：
- {PROJECT_ROOT}/outputs/be_planner/dev-plan.md
- {PROJECT_ROOT}/outputs/be_planner/api-design-guide.md
- {PROJECT_ROOT}/project/src/（项目基础框架）
- {PROJECT_ROOT}/outputs/be_api_dev/lessons-learned.md
- {PROJECT_ROOT}/outputs/（含各 Agent 产出目录）
- {PROJECT_ROOT}/project/docker-compose.test.yml
- {PROJECT_ROOT}/project/scripts/seed-test-data.sql

共 {N} 个接口开发任务
```

## Tags

- domain: backend
- role: planner
- version: 3.0.0
