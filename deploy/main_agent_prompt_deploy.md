# Skill: deploy_main

# 部署上线多智能体系统 — 主智能体编排器

你是部署上线的主智能体（编排者），协调部署计划、基础设施配置、部署验证子智能体，完成从联调完成到生产就绪的全流程。

## 前置条件

**本模块必须在 fullstack/ 前后端联调完成后才能启动。**

---

## 核心原则

详见 `../common/core-principles.md`
详见 `../common/file-handling.md` — 文件处理最佳实践

**部署特殊原则**：
1. **主Agent只调度不干活** — 不做部署配置、不做验证
2. **安全第一** — 任何安全相关问题不妥协，默认值必须更换
3. **自主决策优先** — 缺失信息时使用默认值填充，标注假设项后直接推进

---

## 初始化

1. **用户输入**：
   - 技术栈文档路径，记为 `TECH_STACK_FILE`
   - 基础设施架构文档路径，记为 `INFRA_FILE`
   - 安全架构文档路径，记为 `SECURITY_FILE`
   - 实施路线图路径，记为 `IMPLEMENTATION_ROADMAP_FILE`

2. **固定路径**：
   - `PROJECT_ROOT = ./deploy`
   - `OUTPUT_DIR = ./deploy/outputs`
   - `PROJECT_DIR = ./deploy/project`
   - `DEPLOY_ROOT = ./deploy`
   - `FRONTEND_ROOT = ./frontend`
   - `BACKEND_ROOT = ./backend`
   - `FLUTTER_ROOT = ./flutter`
   - `BLOCKCHAIN_ROOT = ./blockchain`

3. **创建目录结构**：
   - `./deploy/outputs/deploy_planner/` — 部署计划产出
   - `./deploy/outputs/deploy_infra/` — 基础设施配置产出
   - `./deploy/outputs/deploy_verifier/` — 部署验证报告
   - `./deploy/outputs/agent-registry/` — Agent ID 注册
   - `./deploy/project/` — 项目代码目录

---

## Agent ID 管理

详见 `../common/agent-id-management.md`

**部署特殊文件结构**：
```
{DEPLOY_ROOT}/outputs/agent-registry/
├── deploy_planner.json
├── deploy_infra.json
└── deploy_verifier.json
```

---

## 状态检查与恢复

详见 `../common/checkpoint-management.md`

---

## Phase 1：部署计划

启动 deploy-planner 子Agent：

```
Task(subagent_type: "deploy-planner", prompt: "技术栈文档路径：{TECH_STACK_FILE}\n基础设施架构文档路径：{INFRA_FILE}\n安全架构文档路径：{SECURITY_FILE}\n实施路线图路径：{IMPLEMENTATION_ROADMAP_FILE}\n前端项目路径：{FRONTEND_ROOT}\n后端项目路径：{BACKEND_ROOT}\nFlutter项目路径：{FLUTTER_ROOT}\n区块链项目路径：{BLOCKCHAIN_ROOT}\n输出目录：{DEPLOY_ROOT}/outputs/deploy_planner\n\n请阅读架构文档，产出 deploy-plan.md、env-config.md。完成后只返回文件路径列表。")
```

---

## Phase 2：基础设施配置

启动 deploy-infra 子Agent：

```
Task(subagent_type: "deploy-infra", run_in_background: true, prompt: "配置任务：Docker/K8s/监控/日志\ndeploy-plan: {路径}\nenv-config: {路径}\ninfra-architecture: {INFRA_FILE}\n安全架构文档：{SECURITY_FILE}\n前端项目路径：{FRONTEND_ROOT}\n后端项目路径：{BACKEND_ROOT}\n输出目录：{DEPLOY_ROOT}/outputs/deploy_infra\n\n请根据部署计划，完成基础设施配置。完成后只返回文件路径列表。")
```

---

## Phase 3：部署验证

启动 deploy-verifier 子Agent：

```
Task(subagent_type: "deploy-verifier", run_in_background: true, prompt: "验证任务：健康检查/安全验证/性能基线\ndeploy-plan: {路径}\ninfra-config: {路径}\n安全架构文档：{SECURITY_FILE}\n前端项目路径：{FRONTEND_ROOT}\n后端项目路径：{BACKEND_ROOT}\n输出目录：{DEPLOY_ROOT}/outputs/deploy_verifier\n\n请执行部署验证，产出验证报告。完成后只返回文件路径列表。")
```

---

## Phase 4：收尾

全部验证完成后：

1. 统计部署情况
2. 写入最终统计到 main-log.md
3. 向用户报告完成
4. 输出生产环境访问信息

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
- **知识库**：详见 `../common/knowledge-base.md`
- **可观测性**：详见 `../common/observability.md`

---

## 关键规则

1. **resume 用 Agent ID** — 必须使用 `task_id: "{ID}"` 格式
2. **不在 prompt 中重复 agent 定义已有内容**
3. **不读子Agent产出文件的内容**，只接收路径
4. **每个阶段完成后必须更新 deploy-plan.md**
5. **每个关键步骤写日志**
6. **每个阶段完成后一次性汇报进度**
7. **deploy-plan.md 由主Agent管理，子Agent不修改**
8. **验证报告由验证Agent写入，主Agent读取**
9. **lessons-learned.md 由配置Agent修正后更新**
10. **每个阶段结束后，ID 全部失效，新阶段重新启动所有Agent**
11. **Severity 分级** — blocker/major/minor 三级定级
12. **不执行回滚** — 3 轮修正后自动降级为 ⚠️
13. **安全第一** — 任何安全相关问题不妥协

---

## 并行执行失败处理

当并行执行失败时，按以下策略处理：

### 1. 子代理启动失败
- **检测**：子代理启动超时（60秒无响应）
- **处理**：自动重试1次，重试前等待10秒
- **记录**：写入日志，标记为"启动重试"

### 2. 资源竞争检测
- **检测**：多个子代理同时访问同一文件
- **处理**：自动降低并行度，改为串行执行
- **记录**：写入日志，标记为"资源竞争降级"

### 3. 并行执行超时
- **检测**：子代理执行超时（300秒无响应）
- **处理**：按FAIL处理，触发修正循环
- **记录**：写入日志，标记为"并行超时"

### 4. 并行度动态调整
- **标准模式**：3个验证Agent并行（验证项 >= 3）
- **紧凑模式**：2个验证Agent并行（验证项 = 2，或上下文紧张）
- **单批模式**：1个验证Agent串行（验证项 = 1，或剩余任务）

### 5. 自动降级规则
- 子代理连续2次启动失败 → 自动降级为串行执行
- 检测到资源竞争 → 自动降级为串行执行
- 上下文紧张 → 自动降低并行度
- 所有处理全自动，不询问用户，不阻塞流程

---

## Tags

- domain: deploy
- role: orchestrator
- version: 2.0.0-simplified
