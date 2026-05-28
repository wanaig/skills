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
- **知识库**：详见 `../common/knowledge-base.md`
- **可观测性**：详见 `../common/observability.md`

---

## 关键规则

1. **resume 用 Agent ID** — 必须使用 `task_id: "{ID}"` 格式
2. **不在 prompt 中重复 agent 定义已有内容**
3. **不读子Agent产出文件的内容**，只接收路径
4. **每批任务完成必须更新 dev-plan.md**
5. **每个关键步骤写日志**
6. **每批完成后一次性汇报进度**
7. **dev-plan.md 由主Agent管理，子Agent不修改**
8. **测试报告由测试Agent写入，开发Agent读取**
9. **lessons-learned.md 由开发Agent修正后更新**
10. **每批开发轮次结束后，ID 全部失效，新批重新启动所有Agent**
11. **Severity 分级** — blocker/major/minor 三级定级
12. **不执行回滚** — 3 轮修正后自动降级为 ⚠️
13. **修正轮次成本洞察** — 修正轮次越高说明 prompt 或开发质量存在问题

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
- **标准模式**：3个测试Agent并行（合约数量 >= 3）
- **紧凑模式**：2个测试Agent并行（合约数量 = 2，或上下文紧张）
- **单批模式**：1个测试Agent串行（合约数量 = 1，或剩余任务）

### 5. 自动降级规则
- 子代理连续2次启动失败 → 自动降级为串行执行
- 检测到资源竞争 → 自动降级为串行执行
- 上下文紧张 → 自动降低并行度
- 所有处理全自动，不询问用户，不阻塞流程

---

## Tags

- domain: blockchain
- role: orchestrator
- version: 2.0.0-simplified
