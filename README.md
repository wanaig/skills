# Skill: harness_overview

# Harness Engineering — 多智能体协同开发系统

基于 AI Agent 的**多智能体协同开发框架**，7 个主智能体调度 34 个子智能体协作，从 PRD 直达生产部署。

```
architecture/ → frontend/ + backend/ + flutter/ + blockchain/ → fullstack/ → deploy/
   (Phase 0)           (Phase 1, 并行)                           (Phase 2)    (Phase 3)
```

## When to Use This Skill

- 首次了解项目结构和完整使用流程
- 需要快速启动全流程开发
- 查阅各阶段的输入参数和产出文件

---

## 快速开始

### 1. 准备

```text
# 设置以下路径变量（替换为你的实际路径）
REQUIREMENT_FILE=/path/to/prd.md              # 必填：PRD 需求文档
ARCH_ROOT=/path/to/architecture/outputs        # 必填：架构输出目录
FRONTEND_ROOT=/path/to/frontend               # 必填：前端项目目录
BACKEND_ROOT=/path/to/backend                 # 必填：后端项目目录
FLUTTER_ROOT=/path/to/flutter                 # 可选：Flutter 项目目录
BLOCKCHAIN_ROOT=/path/to/blockchain           # 可选：区块链项目目录
DEPLOY_ROOT=/path/to/deploy                   # 必填：部署方案目录
```

### 2. 五步走完完整流程

---

## Phase 0：架构设计（先执行，约 15-30 分钟）

加载技能文件 `architecture/main_agent_prompt_fs_architect.md`，提供参数：

```text
REQUIREMENT_FILE
PROJECT_ROOT={ARCH_ROOT}
```

主Agent 自动执行：PRD 预检 → 6 维度并行分析 → 自审核 → 跨维度一致性检查 → 深度评审 → 产出文档和可执行制品。

**产出**（位于 `{ARCH_ROOT}/outputs/`）：

| 文件 | 内容 |
|------|------|
| `architecture-design.md` | 总架构文档（含 ADR、遗留问题、不做清单） |
| `tech-stack.md` | 技术栈选型（前端/后端/通信/测试/部署） |
| `data-architecture.md` | 数据架构（实体/表结构/缓存/存储/DDL） |
| `infra-architecture.md` | 基础设施（拓扑/CI-CD/监控/成本估算） |
| `security-architecture.md` | 安全架构（认证/鉴权/加密/合规） |
| `api-contract.md` | API 契约（端点/字段/错误码/OpenAPI 骨架） |
| `ui-ux-architecture.md` | UI/UX 架构（路由/组件树/设计Token/页面状态） |
| `implementation-roadmap.md` | 分阶段实施路线图 |
| `artifacts/` | 可执行制品（openapi.yaml / schema.sql / docker-compose.yml 等） |

---

## Phase 1：并行开发（架构完成后可同时启动）

前端和后端互不依赖，可并行执行。Flutter 和区块链按需启动。

### 前端（Vue 3）

加载 `frontend/main_agent_prompt_vue.md`：

```text
PROJECT_ROOT={FRONTEND_ROOT}
REQUIREMENT_FILE
TECH_STACK_FILE={ARCH_ROOT}/outputs/tech-stack.md
CONTRACT_FILE={ARCH_ROOT}/outputs/api-contract.md
SECURITY_FILE={ARCH_ROOT}/outputs/security-architecture.md
UI_UX_FILE={ARCH_ROOT}/outputs/ui-ux-architecture.md
IMPLEMENTATION_ROADMAP_FILE={ARCH_ROOT}/outputs/implementation-roadmap.md
BATCH_SIZE=1
```

`BATCH_SIZE` 控制每次开发的模块数，默认 1（开发 1 个 → 测试 → 修复，循环）。

### 后端（Spring Boot）

加载 `backend/main_agent_prompt.md`：

```text
PROJECT_ROOT={BACKEND_ROOT}
REQUIREMENT_FILE
TECH_STACK_FILE={ARCH_ROOT}/outputs/tech-stack.md
DATA_ARCHITECTURE_FILE={ARCH_ROOT}/outputs/data-architecture.md
CONTRACT_FILE={ARCH_ROOT}/outputs/api-contract.md
SECURITY_FILE={ARCH_ROOT}/outputs/security-architecture.md
IMPLEMENTATION_ROADMAP_FILE={ARCH_ROOT}/outputs/implementation-roadmap.md
BATCH_SIZE=1
```

### Flutter 跨端（可选）

加载 `flutter/main_agent_prompt_flutter.md`：

```text
PROJECT_ROOT={FLUTTER_ROOT}
REQUIREMENT_FILE
TECH_STACK_FILE={ARCH_ROOT}/outputs/tech-stack.md
CONTRACT_FILE={ARCH_ROOT}/outputs/api-contract.md
SECURITY_FILE={ARCH_ROOT}/outputs/security-architecture.md
UI_UX_FILE={ARCH_ROOT}/outputs/ui-ux-architecture.md
IMPLEMENTATION_ROADMAP_FILE={ARCH_ROOT}/outputs/implementation-roadmap.md
BATCH_SIZE=1
```

### 区块链智能合约（可选）

加载 `blockchain/main_agent_prompt_blockchain.md`：

```text
PROJECT_ROOT={BLOCKCHAIN_ROOT}
REQUIREMENT_FILE
TECH_STACK_FILE={ARCH_ROOT}/outputs/tech-stack.md
DATA_ARCHITECTURE_FILE={ARCH_ROOT}/outputs/data-architecture.md
CONTRACT_FILE={ARCH_ROOT}/outputs/api-contract.md
SECURITY_FILE={ARCH_ROOT}/outputs/security-architecture.md
IMPLEMENTATION_ROADMAP_FILE={ARCH_ROOT}/outputs/implementation-roadmap.md
BATCH_SIZE=1
```

---

## Phase 2：前后端联调（Phase 1 全部完成后执行）

加载 `fullstack/main_agent_prompt_fullstack.md`：

```text
FRONTEND_ROOT
BACKEND_ROOT
FLUTTER_ROOT={若有 Flutter 项目}
BLOCKCHAIN_ROOT={若有区块链项目}
UI_UX_FILE={ARCH_ROOT}/outputs/ui-ux-architecture.md
CONTRACT_FILE={ARCH_ROOT}/outputs/api-contract.md
TECH_STACK_FILE={ARCH_ROOT}/outputs/tech-stack.md
DATA_ARCHITECTURE_FILE={ARCH_ROOT}/outputs/data-architecture.md
INFRA_FILE={ARCH_ROOT}/outputs/infra-architecture.md
SECURITY_FILE={ARCH_ROOT}/outputs/security-architecture.md
IMPLEMENTATION_ROADMAP_FILE={ARCH_ROOT}/outputs/implementation-roadmap.md
FRONTEND_LESSONS={FRONTEND_ROOT}/outputs/dg_frontend_vue_dev/lessons-learned.md
BACKEND_LESSONS={BACKEND_ROOT}/outputs/be_api_dev/lessons-learned.md
FLUTTER_LESSONS={FLUTTER_ROOT}/outputs/dg_flutter_dev/lessons-learned.md
BLOCKCHAIN_LESSONS={BLOCKCHAIN_ROOT}/outputs/bc_solidity_dev/lessons-learned.md
BLOCKCHAIN_ABI_DIR={BLOCKCHAIN_ROOT}/artifacts/contracts/
BATCH_SIZE=1
```

---

## Phase 3：部署上线（联调完成后执行）

加载 `deploy/main_agent_prompt_deploy.md`：

```text
TECH_STACK_FILE={ARCH_ROOT}/outputs/tech-stack.md
INFRA_FILE={ARCH_ROOT}/outputs/infra-architecture.md
SECURITY_FILE={ARCH_ROOT}/outputs/security-architecture.md
IMPLEMENTATION_ROADMAP_FILE={ARCH_ROOT}/outputs/implementation-roadmap.md
FRONTEND_ROOT
BACKEND_ROOT
FLUTTER_ROOT={若有 Flutter 项目}
BLOCKCHAIN_ROOT={若有区块链项目}
BLOCKCHAIN_ABI_DIR={BLOCKCHAIN_ROOT}/artifacts/contracts/
DEPLOY_ROOT
```

---

## 关键参数说明

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `BATCH_SIZE` | 每批开发的模块数，开发完一批后立即测试 | 1 |
| `REQUIREMENT_FILE` | PRD 需求文档路径 | — |
| `ARCH_ROOT` | 架构阶段输出目录（= Phase 0 的 PROJECT_ROOT） | — |

---

## 各 Agent 职责与执行流程

每个领域内部遵循统一的三阶段流水线：

```
计划(Planner) → 批量开发-测试循环(Dev + 3 Testers) → 收尾(Wrap-up)
```

| 领域 | 主Agent | Planner | Dev | Tester × 3 |
|------|---------|---------|-----|------------|
| architecture | fs_architect | — | 6 分析Agent | Phase 1b 自审核 + Phase 2 交叉检查 |
| frontend | dg_vue | dg_vue_planner | dg_frontend_vue_dev | component / logic / style |
| backend | be | be_planner | be_api_dev | functional / performance / security |
| flutter | dg_flutter | dg_flutter_planner | dg_flutter_dev | crossplatform / logic / style |
| blockchain | bc | bc_planner | bc_solidity_dev | functional / security / gas |
| fullstack | fs | fs_planner | fs_api_dev | contract / dataflow / integration |
| deploy | deploy | deploy_planner | deploy_infra | deploy_verifier(×1) |

**纠错闭环**：开发 → 三维测试 → FAIL 则 resume 修复 Agent → 重测，最多 3 轮。3 轮后 blocker/major 自动降级为 ⚠️。

---

## 验证收尾

1. 查看各领域 `outputs/main-log.md`，确认所有模块为 ✅
2. 查看各领域的测试报告，确认 PASS
3. 查看 `lessons-learned.md`，确认经验已沉淀

---

## 目录结构

```
├── architecture/       # Phase 0: 技术架构设计
│   ├── agents/         # 6 个子Agent 提示词
│   └── outputs/        # 架构文档 + 可执行制品
│
├── frontend/           # Phase 1a: Vue 3 前端
│   ├── agents/         # 5 个子Agent 提示词
│   ├── outputs/        # 计划/测试报告/经验
│   └── project/        # 前端代码
│
├── backend/            # Phase 1b: Spring Boot 后端
│   ├── agents/
│   ├── outputs/
│   └── project/
│
├── flutter/            # Phase 1c: Flutter 跨端（可选）
│   ├── agents/
│   ├── outputs/
│   └── project/
│
├── blockchain/         # Phase 1d: Solidity 合约（可选）
│   ├── agents/
│   ├── outputs/
│   └── project/
│
├── fullstack/          # Phase 2: 前后端联调
│   ├── agents/
│   ├── outputs/
│   └── project/
│
├── deploy/             # Phase 3: 生产部署
│   ├── agents/
│   ├── outputs/
│   └── project/
│
└── docs/               # 详细文档
```

---

## 核心设计理念

| 理念 | 说明 |
|------|------|
| **主-从协同** | 主Agent 只调度整合，子Agent 专注分析/编码/测试 |
| **文件即记忆** | 所有输出持久化到文件，解决长对话上下文丢失 |
| **隔离即规范** | 子Agent 只接收明确分发的路径和参数，避免信息污染 |
| **纠错闭环** | 开发 → 测试 → 不通过则 resume 修复 → 重测，最多 3 轮 |
| **不阻塞原则** | 缺失信息用默认假设推进，全程不询问用户 |

---

## 目标技术栈

| 领域 | 主要技术 |
|------|---------|
| 前端 | Vue 3 + TypeScript + Vite + Pinia + Vue Router |
| 后端 | Java / Spring Boot |
| 数据库 | PostgreSQL + Redis |
| API | RESTful + OpenAPI 3.0 |
| 鉴权 | JWT (Access + Refresh Token) |
| 跨平台 | Flutter |
| 区块链 | FISCO BCOS v3.x + Solidity + Hardhat |
| 部署 | Docker + K8s / Cloud |

---

## 文档索引

- [系统架构](docs/architecture.md) — Agent 角色总览、执行顺序、数据流
- [设计原理](docs/design_principles.md) — 方法论、上下文策略、流程设计

## 项目定位

本项目是一套 Prompt 模板集合，需在支持 Agent/Task/Skill 调用的 AI 编程环境中作为技能加载使用。设计理念参考 B站 @费曼学徒冬瓜 的《Ralph+多智能体协同》视频。

## Tags

- domain: software-engineering
- type: overview
- version: 2.0.0
