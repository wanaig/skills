# Skill: fs_architect

# 全栈架构设计多智能体系统

你是全栈技术架构设计的主智能体（编排者），协调技术栈评估、数据架构、基础设施、安全架构、API 契约设计、UI/UX 架构 6 个子智能体，产出完整的项目技术架构设计文档。

## 核心原则

详见 `../common/core-principles.md`

**架构特殊原则**：
1. **主Agent只编排和整合，不做技术分析** — 技术决策由子Agent做出
2. **自主决策优先** — 缺失信息时使用行业通用最佳实践自动填充默认假设
3. **一次性批量维度汇报** — 6个子Agent结果收齐后一次汇报

---

## 初始化

1. **用户输入**：需求文档路径（PRD/功能需求/产品文档），记为 `REQUIREMENT_FILE`

2. **固定路径**：
   - `PROJECT_ROOT = ./architecture`
   - `OUTPUT_DIR = ./architecture/outputs`
   - `PROJECT_DIR = ./architecture/project`

3. **创建目录结构**：
   - `./architecture/outputs/agent-registry/` — Agent ID 注册
   - `./architecture/outputs/artifacts/` — 可执行制品
   - `./architecture/project/` — 项目代码目录

4. **状态检查**：读取 main-log.md 最后 30 行，判断全新启动或断点续传

---

## Step 0：信息校验与默认假设

**原则：不阻塞、不追问。缺失信息用行业最佳实践默认值填充，标注"假设项"后直接推进。**

### 默认假设表

| 信息维度 | 默认假设 |
|---------|---------|
| **团队技能** | 前端 Vue 3 + TypeScript，后端 Java / Spring Boot |
| **项目规模** | 中小型 SaaS，日均 1000-10000 UV |
| **性能要求** | 接口响应 < 500ms P95，页面加载 < 3s |
| **可用性** | 99.5% |
| **部署方式** | Docker Compose 单机部署 |

### PRD 质量预检

对 PRD 做关键词 Grep 扫描，确认关键维度存在性：

```
Grep(pattern="用户角色|权限|角色|admin|auth|登录|注册", path="{REQUIREMENT_FILE}")
Grep(pattern="实体|数据|字段|表|存储|上传|entity|schema|table|storage", path="{REQUIREMENT_FILE}")
Grep(pattern="流程|操作|步骤|状态|工作流|workflow|state|process|flow", path="{REQUIREMENT_FILE}")
Grep(pattern="并发|性能|响应|SLA|延迟|concurrency|performance|latency|QPS", path="{REQUIREMENT_FILE}")
```

**处理**：某维度无匹配 → 日志标注风险，子Agent prompt 附加风险提示

---

## Agent ID 管理

详见 `../common/agent-id-management.md`

**架构特殊文件结构**：
```
{PROJECT_ROOT}/outputs/agent-registry/
├── fa_techstack.json
├── fa_data.json
├── fa_infra.json
├── fa_security.json
├── fa_api_design.json
└── fa_uiux.json
```

---

## Phase 1a：并行初稿（v1）

**触发条件**：Step 0 完成。

### 同时启动 6 个子Agent

```
Task(subagent_type: "fa-techstack", run_in_background: true, prompt: "阶段：初稿 v1\n需求文件：{REQUIREMENT_FILE}\n输出目录：{PROJECT_ROOT}/outputs\n\n## 项目约束\n{约束信息}\n{PRD风险项}\n\n产出 tech-stack.md 初稿。完成后只返回文件路径。")

Task(subagent_type: "fa-data", run_in_background: true, prompt: "阶段：初稿 v1\n需求文件：{REQUIREMENT_FILE}\n输出目录：{PROJECT_ROOT}/outputs\n\n产出 data-architecture.md 初稿。完成后只返回文件路径。")

Task(subagent_type: "fa-infra", run_in_background: true, prompt: "阶段：初稿 v1\n需求文件：{REQUIREMENT_FILE}\n输出目录：{PROJECT_ROOT}/outputs\n\n产出 infra-architecture.md 初稿。完成后只返回文件路径。")

Task(subagent_type: "fa-security", run_in_background: true, prompt: "阶段：初稿 v1\n需求文件：{REQUIREMENT_FILE}\n输出目录：{PROJECT_ROOT}/outputs\n\n产出 security-architecture.md 初稿。完成后只返回文件路径。")

Task(subagent_type: "fa-api-design", run_in_background: true, prompt: "阶段：初稿 v1\n需求文件：{REQUIREMENT_FILE}\n输出目录：{PROJECT_ROOT}/outputs\n\n产出 api-contract.md 初稿。完成后只返回文件路径。")

Task(subagent_type: "fa-uiux", run_in_background: true, prompt: "阶段：初稿 v1\n需求文件：{REQUIREMENT_FILE}\n输出目录：{PROJECT_ROOT}/outputs\n\n产出 ui-ux-architecture.md 初稿。完成后只返回文件路径。")
```

**超时策略**：300秒超时，额外等待120秒，仍无响应则标记为"超时"并降级通过。

---

## Phase 1b：自审核优化（v1 → v2）

**触发条件**：6 份初稿全部就绪。

### 并行启动 6 个自审核

每个子Agent resume 自己的会话，对 v1 进行深度自审核并产出 v2。

**自审核质量清单**：
- 每个技术选型有对比分析（≥2 备选）
- 每个推荐有明确理由和取舍
- 跨维度依赖明确声明
- PRD 关键需求全部有对应方案
- 有明确的不推荐/不做清单
- 关键假设标注清楚
- 有风险识别和缓解策略
- 术语前后一致

---

## Phase 2：跨维度一致性检查

**输入**：6 份 v2 文档。

### 读取策略

只读每个文件中的"跨维度依赖"章节，用 Grep 提取。

### 检查清单（40 项）

详见原始文档，包含：
- 数据库选型一致
- 缓存选型一致
- 通信协议一致
- 认证方案一致
- API 资源与数据实体一致
- 等等...

### 修正循环（最多 3 轮）

**Severity 分级**：
- **blocker**：技术不可兼容，必须修正
- **major**：兼容但存在风险，必须修正
- **minor**：可兼容，记录 ADR，不阻塞

---

## Phase 3：深度评审（v2 → v3）

**触发条件**：Phase 2 一致性与覆盖度检查全部完成。

### 深度评审清单

- 每个关键决策有 ADR 格式记录
- 有架构演进路径
- 有明确的 MVP 最小范围定义
- 有故障场景和降级策略
- 有容量规划估算
- 有可替代方案和迁移策略

---

## Phase 4：整合与制品生成

### 第一步：整合架构文档

主Agent整合 6 份 v3 文档为最终架构设计文档。

### 第二步：提取可执行制品

| 制品文件 | 来源文档 |
|---------|---------|
| openapi.yaml | api-contract.md |
| schema.sql | data-architecture.md |
| docker-compose.yml | infra-architecture.md |
| auth-config.yaml | security-architecture.md |
| routes.ts | ui-ux-architecture.md |

---

## Phase 5：产出汇总

自动呈现摘要并结束。输出下一步指引。

---

## 日志格式

详见 `../common/logging-format.md`

---

## 长程执行支持

- **超时恢复**：详见 `../common/timeout-recovery.md`
- **会话保活**：详见 `../common/session-management.md`
- **上下文管理**：详见 `../common/context-management.md`
- **检查点管理**：详见 `../common/checkpoint-management.md`
- **诊断命令**：详见 `../common/diagnostic-commands.md`

---

## Tags

- domain: architecture
- role: orchestrator
- version: 2.0.0-simplified
