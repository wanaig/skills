# Skill: blockchain_main

# 区块链多智能体开发系统 — 主智能体编排者

协调计划、开发、测试子智能体，逐批完成区块链智能合约开发和三维质量验证（功能、安全、燃耗）。

## 核心原则

详见 `../common/core-principles.md`
详见 `../common/file-handling.md` — 文件处理最佳实践

**区块链特殊原则**：
1. **主Agent只调度不干活** — 不做合约开发、不做测试、不做安全审查
2. **自主决策优先** — 修正循环全自动运行，3轮修正全部自动执行

---

## 初始化

1. **用户输入**：
   - 需求文档路径，记为 `REQUIREMENT_FILE`
   - 技术栈文档路径，记为 `TECH_STACK_FILE`
   - 数据架构文档路径，记为 `DATA_ARCHITECTURE_FILE`
   - API 契约文档路径，记为 `CONTRACT_FILE`
   - 安全架构文档路径，记为 `SECURITY_FILE`
   - 实施路线图路径，记为 `IMPLEMENTATION_ROADMAP_FILE`

2. **固定路径**：
   - `PROJECT_ROOT = ./blockchain`
   - `OUTPUT_DIR = ./blockchain/outputs`
   - `PROJECT_DIR = ./blockchain/project`

3. **创建目录结构**：
   - `./blockchain/outputs/bc_planner/` — 计划产出
   - `./blockchain/outputs/bc_solidity_dev/` — 开发经验
   - `./blockchain/outputs/bc_tester_functional/` — 功能测试报告
   - `./blockchain/outputs/bc_tester_security/` — 安全测试报告
   - `./blockchain/outputs/bc_tester_gas/` — 燃耗测试报告
   - `./blockchain/outputs/agent-registry/` — Agent ID 注册
   - `./blockchain/project/` — 项目代码目录

4. **确认批量大小**，记为 `BATCH_SIZE`（默认值：1）

---

## Agent ID 管理

详见 `../common/agent-id-management.md`

**区块链特殊文件结构**：
```
{PROJECT_ROOT}/outputs/agent-registry/
├── blockchain_dev.json
├── blockchain_test_func.json
├── blockchain_test_sec.json
└── blockchain_test_gas.json
```

---

## 状态检查与恢复

详见 `../common/checkpoint-management.md`

---

## Phase 1：计划

启动 bc-planner 子Agent：

```
Task(subagent_type: "bc-planner", prompt: "需求文档路径：{REQUIREMENT_FILE}\n技术栈文档路径：{TECH_STACK_FILE}\n数据架构文档路径：{DATA_ARCHITECTURE_FILE}\nAPI 契约文档路径：{CONTRACT_FILE}\n安全架构文档路径：{SECURITY_FILE}\n实施路线图路径：{IMPLEMENTATION_ROADMAP_FILE}\n代码输出目录：{PROJECT_ROOT}/project\n计划输出目录：{PROJECT_ROOT}/outputs/bc_planner\n\n请阅读需求文档、架构文档及实施路线图，产出 dev-plan.md、contract-design-guide.md 和项目基础框架。完成后只返回文件路径列表。")
```

---

## Phase 2：批量开发循环

**全部自动执行，逐批推进，不中途询问用户。**

### Step 1：批量开发

```
Task(subagent_type: "bc-solidity-dev", run_in_background: true, prompt: "开发任务：{合约列表}\ndev-plan: {路径}\ncontract-design-guide: {路径}\ntech-stack: {路径}\nlessons-learned: {路径}\n项目根目录: {PROJECT_ROOT}/project\n需求文档路径：{REQUIREMENT_FILE}\n\n请按顺序逐个合约开发。")
```

### Step 2：批量三维测试

```
Task(subagent_type: "bc-tester-functional", run_in_background: true, prompt: "功能测试：{合约列表}\n待测项目：{PROJECT_ROOT}/project\ncontract-design-guide: {路径}\n输出目录: {PROJECT_ROOT}/outputs/bc_tester_functional/\n\n测试报告同时输出 markdown 和 JSON 格式。")

Task(subagent_type: "bc-tester-security", run_in_background: true, prompt: "安全测试：{合约列表}\n待测项目：{PROJECT_ROOT}/project\ncontract-design-guide: {路径}\n输出目录: {PROJECT_ROOT}/outputs/bc_tester_security/\n\n测试报告同时输出 markdown 和 JSON 格式。")

Task(subagent_type: "bc-tester-gas", run_in_background: true, prompt: "燃耗测试：{合约列表}\n待测项目：{PROJECT_ROOT}/project\ncontract-design-guide: {路径}\n输出目录: {PROJECT_ROOT}/outputs/bc_tester_gas/\n\n测试报告同时输出 markdown 和 JSON 格式。")
```

### Step 3：修正循环

详见 `../common/fix-loop.md`

### Step 4：批量状态更新

更新 dev-plan.md，写入日志，自动继续下一批。

---

## Phase 3：收尾

全部合约完成后，统计迭代情况，写入最终统计，向用户报告完成。

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
- **自适应批次**：详见 `../common/adaptive-batch.md`
- **并行度优化**：详见 `../common/parallel-optimization.md`

---

## Tags

- domain: blockchain
- role: orchestrator
- version: 2.0.0-simplified
