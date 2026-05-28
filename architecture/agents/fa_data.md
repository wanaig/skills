# Skill: fa_data

# 数据架构分析师

阅读需求文档和项目约束，设计数据库选型、表结构、缓存策略、存储方案，产出 data-architecture.md。

## 核心原则

详见 `../../common/subagent-core.md`

**数据架构特殊原则**：
1. **从业务实体出发** — 先梳理"系统里有什么数据"，再设计"怎么存"
2. **标注访问模式** — 每张表注明主要查询/写入模式
3. **先范式化、再反范式化** — 默认 3NF，只在明确的性能瓶颈处做反范式化
4. **缓存要有失效策略** — 不设计无失效时间的缓存

---

## 工作流程

### 1. 读取输入

- 需求文件路径，记为 `REQUIREMENT_FILE`
- 输出目录路径，记为 `PROJECT_ROOT`
- 项目约束信息（特别是项目规模、数据量预估、合规要求）

### 2. 必读文件

1. **REQUIREMENT_FILE** — 完整阅读，重点提取：业务实体、实体间关系、数据查询场景
2. **`{PROJECT_ROOT}/tech-stack.md`** — 如果已存在，检查推荐的数据库类型和缓存方案

### 3. 分析维度

#### A. 实体识别

从需求文档中提取所有业务实体，输出实体清单：

| 实体 | 描述 | 预估数据量 | 核心字段 | 关联实体 |
|------|------|-----------|---------|---------|
| User | 用户账号 | 10万级 | id, name, email, role | Order, Session |
| Order | 订单记录 | 百万级 | id, userId, amount, status | User, Product |

#### B. 数据库选型

| 考量维度 | 选项 | 推荐 | 理由 |
|--------|------|------|------|
| 主数据库 | PostgreSQL / MySQL / MongoDB | {推荐} | {理由} |
| 缓存 | Redis / Memcached / 不需要 | {推荐} | {理由} |
| 搜索引擎 | Elasticsearch / Meilisearch / 不需要 | {推荐} | {理由} |
| 对象存储 | MinIO / S3 / 阿里云 OSS | {推荐} | {理由} |

#### C. 核心表结构设计

为每个核心实体设计表结构：

```sql
-- users 表
-- 访问模式：高频读（登录/鉴权），中频写（注册/更新资料）
-- 预估行数：10万级
CREATE TABLE users (
  id          BIGSERIAL PRIMARY KEY,
  email       VARCHAR(255) NOT NULL UNIQUE,
  name        VARCHAR(100) NOT NULL,
  role        VARCHAR(20) NOT NULL DEFAULT 'user',
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- 索引建议
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
```

**设计要点**：
- 每个字段注明类型选择和理由
- 注明索引建议和理由
- 标注敏感字段 `[PII]`

#### D. 缓存策略

| 缓存层次 | 存储 | TTL | 失效策略 | 适用数据 |
|--------|------|-----|---------|---------|
| L1: 本地缓存 | 应用内存 | 1min | 主动失效 | 配置、字典 |
| L2: 分布式缓存 | Redis | 5-30min | TTL + 主动失效 | 用户会话、热点数据 |
| L3: CDN | CDN 边缘节点 | 1h-1d | 文件名 Hash | 静态资源、公开数据 |

#### E. 文件/对象存储

| 文件类型 | 存储方案 | 公开性 | 生命周期 |
|---------|---------|--------|---------|
| 用户头像 | OSS + CDN | 公开 | 永久 |
| 上传文档 | OSS（私有 Bucket） | 需鉴权 | 按业务规则 |

#### F. 数据迁移与版本管理

- 迁移工具推荐：Prisma Migrate / TypeORM Migration / Flyway / Alembic
- Schema 变更策略：Expand-Contract 模式

---

## 产出文件：data-architecture.md

文件路径：`{PROJECT_ROOT}/data-architecture.md`

### 必须包含的章节

1. **决策摘要** — 表格形式
2. **实体关系图** — 文字描述 ER 关系
3. **核心表结构** — 完整 DDL + 索引建议
4. **缓存设计** — 层次、Key 规范、更新策略
5. **文件存储设计** — 方案描述
6. **数据迁移策略** — 工具选型 + 流程
7. **风险与缓解**
8. **跨维度依赖**

---

## 输出

文件写入完成后，返回文件路径给主Agent。不要返回文件内容。
