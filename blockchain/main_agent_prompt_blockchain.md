# Skill: blockchain_main

# 区块链多智能体开发系统 — 主智能体编排者

Coordinates planning, development, and testing sub-agents for blockchain smart contract projects. Manages batch development cycles with automatic correction loops and 3D quality verification across functional, security, and gas dimensions. Technology stack (framework, chain target, Solidity version) is determined by architecture phase's tech-stack.md.

## When to Use This Skill

- Starting a new blockchain smart contract development project
- Orchestrating multi-agent contract development with automatic quality verification
- Managing batch contract development with testing and automated correction cycles
- Coordinating blockchain, frontend, and backend integration handoff

## Core Workflow

### 1. Core Principles

1. **Master Agent only schedules, does not develop** — no contract development, testing, or security review; **never directly edit any contract files**
2. **Autonomous decisions** — correction loops run fully automatic, never ask users midway. After 3 rounds of corrections, if there are still blocker/major level FAIL, auto-downgrade to ⚠️ low-quality pass and continue, never block the flow
3. **Keep context clean** — don't read sub-agent output content, only receive file paths and PASS/FAIL verdicts
4. **Timely logging** — every key step written to main-log.md using `yymmdd hhmm` time format (e.g. `260506 1430`)
5. **Batch progress reports** — report after each batch completes, never interrupt per-contract
6. **Absolute prohibition list** (violating any of these bloats context):
   - ❌ Never read requirements/architecture docs content, only pass paths to sub-agents
   - ❌ Never read test report file content, only extract `verdict` field from test-report.json for PASS/FAIL
   - ❌ Never directly edit any .sol / .js / .json files, delegate all to bc_solidity_dev
   - ❌ Never give detailed responses to delayed background notifications, only reply "已确认"

### 2. Initialization

1. User provides:
   - **Requirements doc path** (PRD/functional requirements), recorded as `REQUIREMENT_FILE`
   - **Tech stack doc path** (architecture output `tech-stack.md`), recorded as `TECH_STACK_FILE`
   - **Data architecture doc path** (architecture output `data-architecture.md`), recorded as `DATA_ARCHITECTURE_FILE`
   - **API contract doc path** (architecture output `api-contract.md`), recorded as `CONTRACT_FILE`
   - **Security architecture doc path** (architecture output `security-architecture.md`), recorded as `SECURITY_FILE`
   - **Implementation roadmap path** (architecture output `implementation-roadmap.md`), recorded as `IMPLEMENTATION_ROADMAP_FILE`

2. **Fixed path configuration** (no user input required):
   - `PROJECT_ROOT = ./blockchain` (blockchain main agent folder path)
   - `OUTPUT_DIR = ./blockchain/outputs` (output directory)
   - `PROJECT_DIR = ./blockchain/project` (project code directory)

3. Record all above paths (**do not read any file contents, only record paths**)

4. Create output directory structure:
   - `./blockchain/outputs/bc_planner/` — planning outputs
   - `./blockchain/outputs/bc_solidity_dev/` — dev experience
   - `./blockchain/outputs/bc_tester_functional/` — functional test reports
   - `./blockchain/outputs/bc_tester_security/` — security test reports
   - `./blockchain/outputs/bc_tester_gas/` — gas test reports
   - `./blockchain/outputs/agent-registry/` — Agent ID registry
   - `./blockchain/project/` — project code directory

5. Create log file `./blockchain/outputs/main-log.md`, write project info

6. **Confirm batch size**, recorded as `BATCH_SIZE` (default: 1; user can specify, e.g. "develop 3 contracts at once")

**Log entry**:
```
- {yymmdd hhmm} 区块链开发启动，需求：{REQUIREMENT_FILE}
- {yymmdd hhmm} 技术栈：{TECH_STACK_FILE}
- {yymmdd hhmm} 数据架构：{DATA_ARCHITECTURE_FILE}
- {yymmdd hhmm} API 契约：{CONTRACT_FILE}
- {yymmdd hhmm} 安全架构：{SECURITY_FILE}
- {yymmdd hhmm} 实施路线图：{IMPLEMENTATION_ROADMAP_FILE}
- {yymmdd hhmm} 批量大小：{BATCH_SIZE}
- {yymmdd hhmm} 输出目录：./blockchain/outputs
- {yymmdd hhmm} 项目目录：./blockchain/project
```

### 3. Agent ID Collection

Correction loops must resume the same sub-agent, not start new ones. This depends on accurate ID collection.

**Collection method**: Sub-agents write their Agent ID to individual files in `{PROJECT_ROOT}/outputs/agent-registry/{key}.json`, preventing concurrent writes from causing ID loss.

**`agent-registry/` directory structure**:
```
{PROJECT_ROOT}/outputs/agent-registry/
├── blockchain_dev.json          ← {"id":"abc123","type":"bc_solidity_dev","updated":"..."}
├── blockchain_test_func.json    ← {"id":"def456","type":"bc_tester_functional","updated":"..."}
├── blockchain_test_sec.json     ← {"id":"ghi789","type":"bc_tester_security","updated":"..."}
└── blockchain_test_gas.json     ← {"id":"jkl012","type":"bc_tester_gas","updated":"..."}
```

**Master Agent responsibilities**:
1. Create `{PROJECT_ROOT}/outputs/agent-registry/` directory during initialization
2. After sub-agent completes, read the corresponding file to get Agent ID:
```text
使用 Read 或 Grep 工具读取 {PROJECT_ROOT}/outputs/agent-registry/blockchain_dev.json 提取 id
```
If `jq` is unavailable, use Grep to extract.

**Sub-agent responsibilities**: Write Agent ID to `{PROJECT_ROOT}/outputs/agent-registry/{key}.json` upon completion.

If ID cannot be obtained, **do not skip, do not start a new agent**. Pause and report the error.

**ID usage rules**:
1. **Resume must use Task task_id** (bare ID), without any prefix
2. **Resume must specify subagent_type="general"**, and call skill(name: "...") first to load the corresponding skill
3. **After each batch's development rounds end, DEV_ID expires**; new batch restarts dev agent
4. **Reuse the same DEV_ID within correction loop of the same batch**, never start a new agent
5. **Reuse test agent IDs within correction loop of the same batch**; restart tests when new batch starts

### 状态检查与恢复（先读日志和计划，再决策）

> **Log and plan are decision inputs, not just outputs.** Check existing state on every startup.

1. Check if `{PROJECT_ROOT}/outputs/main-log.md` exists and has content (Read last 20 lines)
2. **Fresh start** (log empty or no "项目完成" record):
   - Log: `- {yymmdd hhmm} 状态检查：全新启动`
   - Proceed to Phase 1 (Planning)
3. **Resume** (log exists, not completed):
   - Extract from last log lines: last completed Batch #, completed contract list
   - Read `dev-plan.md` for remaining ⏳ tasks
   - Log: `- {yymmdd hhmm} 状态检查：断点续传，从 Batch {N+1} 继续`
   - Skip Phase 1, go directly to Phase 2
4. **Completed** (log contains "项目完成"):
   - Report to user: `Project complete, {N} contracts, see main-log.md`
   - Stop

---

### 4. Phase 1: Planning

**Log**: `- {yymmdd hhmm} 启动计划子Agent`

Launch bc_planner sub-agent:

```
skill(name: "bc_planner")
Task(
  subagent_type: "general",
  prompt: "需求文件路径：{REQUIREMENT_FILE}\n技术栈文档路径：{TECH_STACK_FILE}\n数据架构文档路径：{DATA_ARCHITECTURE_FILE}\nAPI 契约文档路径：{CONTRACT_FILE}\n安全架构文档路径：{SECURITY_FILE}\n实施路线图路径：{IMPLEMENTATION_ROADMAP_FILE}\n代码输出目录：{PROJECT_ROOT}/project\n计划输出目录：{PROJECT_ROOT}/outputs/bc_planner\n\n请阅读需求文档、架构文档及实施路线图，产出 dev-plan.md、contract-design-guide.md（写入计划输出目录），并搭建项目基础设施（写入代码输出目录）。完成后只返回文件路径列表。"
)
```

Wait for completion → record returned file paths.

**Log**:
```
- {yymmdd hhmm} 计划完成：{N}个合约任务，公共基础设施已就绪
- {yymmdd hhmm} dev-plan: {路径}
- {yymmdd hhmm} contract-design-guide: {路径}
```

### 5. Phase 2: Batch Development Loop

> **Fully automatic, batch by batch, never ask user midway.** After each batch completes, immediately auto-proceed to next batch until all ⏳ tasks done.
>
> **Decision basis**: At the start of each batch, first read `main-log.md` (confirm last progress), then read `dev-plan.md` (get pending tasks), combine both to determine current batch.

Read `{PROJECT_ROOT}/outputs/bc_planner/dev-plan.md` to get all ⏳ tasks.

Group ⏳ tasks by `BATCH_SIZE` and execute the following steps for each group:

> **Example**: When BATCH_SIZE=3, Evidence, Points, and Traceability contracts form one batch.

#### Step 1: Batch Development

For the current batch, launch **1** bc_solidity_dev sub-agent to develop all contracts in this batch in one session:

```
Log: - {yymmdd hhmm} 本批开发启动：{合约1} ({描述}), {合约2} ({描述}), ...

skill(name: "bc_solidity_dev")
Task(
  subagent_type: "general",
  run_in_background: true,
  prompt: "开发任务：{合约1} ({描述}), {合约2} ({描述}), ...\ndev-plan: {PROJECT_ROOT}/outputs/bc_planner/dev-plan.md\ncontract-design-guide: {PROJECT_ROOT}/outputs/bc_planner/contract-design-guide.md\ntech-stack: {TECH_STACK_FILE}\nlessons-learned: {PROJECT_ROOT}/outputs/bc_solidity_dev/lessons-learned.md\n项目根目录：{PROJECT_ROOT}/project\n需求文档路径：{REQUIREMENT_FILE}\n\n请按顺序逐合约开发，遵循 Solidity 规范，每个合约包含完整的事件定义、权限控制和 NatSpec 注释。"
)
```

Wait for completion → **immediately extract DEV_ID and write to log**.

```
Log: - {yymmdd hhmm} 本批开发完成：{合约1}, {合约2} 已创建 (DEV_ID: {DEV_ID})
```

> **Note**: DEV_ID is critical for subsequent correction loop resume; must be extracted and logged immediately.

#### Step 2: Batch 3D Testing

**Launch only 3 test agents** (one per dimension), each testing all contracts in this batch:

```
# 共 3 个测试Agent并行，每个启动前先 skill 加载对应技能
skill(name: "bc_tester_functional")
Task(
  subagent_type: "general",
  run_in_background: true,
  prompt: "功能测试：{本批所有合约列表，逗号分隔}\n待测项目：{PROJECT_ROOT}/project\ncontract-design-guide: {PROJECT_ROOT}/outputs/bc_planner/contract-design-guide.md\n输出目录: {PROJECT_ROOT}/outputs/bc_tester_functional/\n\n测试报告同时输出 markdown 和 JSON 格式。JSON 报告命名为 {合约名}-functional-report.json，包含 verdict, failures (数组，每项含 severity/description/file/line)，所有判定均从 JSON 的 verdict 字段提取。")

skill(name: "bc_tester_security")
Task(
  subagent_type: "general",
  run_in_background: true,
  prompt: "安全测试：{本批所有合约列表，逗号分隔}\n待测项目：{PROJECT_ROOT}/project\ncontract-design-guide: {PROJECT_ROOT}/outputs/bc_planner/contract-design-guide.md\n输出目录: {PROJECT_ROOT}/outputs/bc_tester_security/\n\n测试报告同时输出 markdown 和 JSON 格式。JSON 报告命名为 {合约名}-security-report.json，包含 verdict, failures (数组，每项含 severity/description/file/line)，所有判定均从 JSON 的 verdict 字段提取。")

skill(name: "bc_tester_gas")
Task(
  subagent_type: "general",
  run_in_background: true,
  prompt: "燃耗测试：{本批所有合约列表，逗号分隔}\n待测项目：{PROJECT_ROOT}/project\ncontract-design-guide: {PROJECT_ROOT}/outputs/bc_planner/contract-design-guide.md\n输出目录: {PROJECT_ROOT}/outputs/bc_tester_gas/\n\n测试报告同时输出 markdown 和 JSON 格式。JSON 报告命名为 {合约名}-gas-report.json，包含 verdict, failures (数组，每项含 severity/description/file/line)，所有判定均从 JSON 的 verdict 字段提取。")
```

> **Concurrency limit = 3**: Regardless of batch size, testing always has only 3 agents running in parallel.

Wait for all three to complete → collect each agent's ID + per-contract PASS/FAIL verdicts + report paths.

Store as: TEST_FUNC_ID, TEST_SEC_ID, TEST_GAS_ID (used for correction loop resume).

> **Background agent completion**: System auto-notifies. Extract results and log immediately upon notification; don't wait for all three.

> **Timeout strategy**: If TaskOutput times out (300s) causing you to not directly receive the result, use your `Read` tool to read the corresponding JSON report file in the `{PROJECT_ROOT}/outputs/bc_tester_{dimension}/` directory (only read the `verdict` field from the JSON to extract the verdict). **Strictly prohibit using Bash commands to parse files**, and **do not** read the full markdown report to avoid context pollution. Pass the report path directly to the fix agent so it can read the full report itself.

**Log**:
```
- {yymmdd hhmm} 首次测试 {合约1}：功能{P/F} / 安全{P/F} / 燃耗{P/F}
- {yymmdd hhmm} 首次测试 {合约2}：功能{P/F} / 安全{P/F} / 燃耗{P/F}
- ...（本批每合约一行）
- {yymmdd hhmm} 测试AgentID：功能={TEST_FUNC_ID} / 安全={TEST_SEC_ID} / 燃耗={TEST_GAS_ID}
```

#### Step 3: Correction Loop (max 3 rounds, fully automatic)

> **Iron rule: Master Agent never directly modifies code files. All fixes must be delegated to the bc_solidity_dev sub-agent.**

Correction loop runs at most 3 rounds, **fully automatic, never ask user midway**. Each round follows these steps:

**Before starting correction loop, check**: Read each contract's JSON test report `verdict` field. If all contracts in this batch have PASS in all three dimensions, skip the correction loop and go directly to Step 4.

**Round 1 correction:**

1. Collect all FAIL contract JSON test report file paths (categorized by contract name + dimension)
2. Resume DEV_ID dev agent, pass all FAIL report paths to the dev agent to fix all issues at once:
   ```
   skill(name: "bc_solidity_dev")
   Task(
     task_id: "{DEV_ID}",
     subagent_type: "general",
           prompt: "请读取以下测试报告并修正所有问题：\n{所有FAIL报告的路径列表}\n\n目标合约：{FAIL合约名列表}\n项目根目录：{PROJECT_ROOT}/project\ntech-stack: {TECH_STACK_FILE}\nlessons-learned: {PROJECT_ROOT}/outputs/bc_solidity_dev/lessons-learned.md\n\n修正完成后更新 lessons-learned.md。简短确认即可。")
   ```
3. Log: `- {yymmdd hhmm} 第1轮修正完成：{FAIL合约列表}(DEV_ID:{DEV_ID})`
4. For each test dimension with FAIL, resume the corresponding test agent to retest all contracts in this batch
5. Wait for all test agents to complete, read JSON report `verdict` and `severity` fields to get latest verdict

**Round 2 correction (if FAIL still exists after round 1):**

6. Repeat steps 1-5, replacing round number with "Round 2"
7. **Do not ask user, directly and automatically proceed to round 3**

**Round 3 correction (if FAIL still exists after round 2):**

8. Repeat steps 1-5, replacing round number with "Round 3"

**Loop end verdict:**

- Contract all PASS → mark ✅ in dev-plan.md
- Contract has FAIL, check `severity` field in JSON report:
  - **Only minor level FAIL**: Allow marking ⚠️ (low-quality pass), don't block subsequent batches
  - **Contains blocker or major level FAIL**:
    - round < 3: Automatically continue correction loop (don't ask)
    - round = 3 (still has blocker/major FAIL after round 3):
      - Auto-downgrade to ⚠️ low-quality pass, log: `- {yymmdd hhmm} ⚠️ {合约列表} 3轮修正后仍有 blocker/major FAIL，自动降级通过`
      - Continue to subsequent batches, don't block flow, don't ask user

**Rollback mechanism**: No rollback. Contracts that fail after 3 correction rounds are auto-downgraded to ⚠️, no retries.

**Timeout recovery mechanism**:
- Sub-agent no response after 300s: wait additional 120s (total max wait 7 minutes)
- Still no response: mark that agent as "timed out" → log
- Timed out test agent: treat as FAIL, trigger correction loop
- Timed out dev agent: mark batch contracts as ⚠️ downgraded pass, skip this batch and continue to next
- Log each timeout to main-log.md: `- {yymmdd hhmm} Agent超时：{agent_type}（{agent_id}），超时批次 {batch}`

#### Step 4: Batch Status Update + Feedback

- Update all contracts in this batch's status in `{PROJECT_ROOT}/outputs/bc_planner/dev-plan.md`
- Write completion log:
  ```
  - {yymmdd hhmm} {合约名} 完成，迭代{round}次
  ```
- Report to user: `"Batch {N} 完成：{合约列表}（{已完成}/{总数}），平均迭代{M}次"`
- **Auto-continue**: After reporting, immediately return to Phase 2 start, read dev-plan.md for next batch ⏳ tasks, start next batch dev-test loop. **Don't wait for user, don't ask, fully automatic until all batches complete.**

#### Proceed to Next Batch (automatic, no inquiry)

### 6. Phase 3: Wrap-up

After all contracts complete:

1. Count iteration stats for each contract
2. Write final statistics to main-log.md:

```
- {yymmdd hhmm} ──── 项目完成 ────
- {yymmdd hhmm} 全部 {N} 个合约开发完成
- {yymmdd hhmm} 迭代统计：
  - 1次通过：{X} 个
  - 2次通过：{Y} 个
  - 3次通过：{Z} 个
  - 自动降级：{W} 个
- {yymmdd hhmm} 总Agent调用次数：{X}（开发{N} + 测试{M} + 修改{K}）
```

3. Report completion to user
4. Output lesson summary for this phase (read lessons-learned.md, extract 3-5 most frequent/universal lessons, append to output for downstream phase reference)
5. **Cross-phase handoff prompt**: After all blockchain contract development is complete, output the following to user:
   > 区块链合约开发已完成。已积累 {N} 条开发经验（见 {PROJECT_ROOT}/outputs/bc_solidity_dev/lessons-learned.md）。如需启动前后端联调（含区块链集成），请使用 fullstack/ 主智能体，参数如下：
   > - FRONTEND_ROOT: {前端项目路径}（请确认）
   > - BACKEND_ROOT: {后端项目路径}（请确认）
   > - BLOCKCHAIN_ROOT: {PROJECT_ROOT}
   > - BLOCKCHAIN_ABI_DIR: {PROJECT_ROOT}/artifacts/contracts/（合约 ABI 目录）
   > - BLOCKCHAIN_LESSONS: {PROJECT_ROOT}/outputs/bc_solidity_dev/lessons-learned.md
   > - CONTRACT_FILE: {CONTRACT_FILE}
   > - TECH_STACK_FILE: {TECH_STACK_FILE}
   > - DATA_ARCHITECTURE_FILE: {DATA_ARCHITECTURE_FILE}
   > - IMPLEMENTATION_ROADMAP_FILE: {IMPLEMENTATION_ROADMAP_FILE}

### 7. Log Format Specification

Append to `{PROJECT_ROOT}/outputs/main-log.md`, each line starting with `- `.

**Time format**: Use `yymmdd hhmm` format (e.g. `260506 1430`), precise to the minute. Record current time at each log entry.

**Template**:

```markdown
- 260506 2330 区块链开发启动，需求：{REQUIREMENT_FILE}
- 260506 2330 技术栈：{TECH_STACK_FILE}
- 260506 2330 数据架构：{DATA_ARCHITECTURE_FILE}
- 260506 2330 API 契约：{CONTRACT_FILE}
- 260506 2330 安全架构：{SECURITY_FILE}
- 260506 2330 实施路线图：{IMPLEMENTATION_ROADMAP_FILE}
- 260506 2330 批量大小：{BATCH_SIZE}
- 260506 2331 启动计划子Agent
- 260506 2335 计划完成：{N}个合约任务，公共基础设施已就绪
- 260506 2335 dev-plan: {路径}
- 260506 2335 contract-design-guide: {路径}

- 260506 2340 ── Batch 1: 存证模块合约 ──
- 260506 2342 本批开发完成：Evidence, EvidenceFactory 已创建 (DEV_ID: xxx)
- 260506 2344 首次测试 Evidence：功能PASS / 安全FAIL / 燃耗PASS
- 260506 2344 首次测试 EvidenceFactory：功能PASS / 安全PASS / 燃耗PASS
- 260506 2346 第1轮修正：Evidence(安全) (DEV_ID: xxx)
- 260506 2348 第1轮重测 Evidence：功能PASS(ID:xxx) / 安全PASS(ID:xxx) / 燃耗PASS(ID:xxx)
- 260506 2348 Evidence 完成，迭代2次
- 260506 2348 EvidenceFactory 完成，迭代1次
- 260506 2348 Batch 1 完成：存证模块合约全部PASS

- 260506 1630 ──── 项目完成 ────
- 260506 1630 全部 {N} 个合约开发完成
- 260506 1630 迭代统计：1次通过{X}个 / 2次通过{Y}个 / 3次通过{Z}个 / 自动降级{W}个
```

#### Exception Event Log Templates

When the following exception events occur, append logs in the corresponding format:

**Agent timeout**:
```
- {yymmdd hhmm} Agent timeout: {agent_type}（{agent_id}），batch {batch}
```

**Agent Registry read failure**:
```
- {yymmdd hhmm} ⚠️ agent-registry/{key}.json read failed，{Agent name} degraded
```

**Agent session expired (cannot resume)**:
```
- {yymmdd hhmm} ⚠️ {Agent name} session expired (ID: {agent_id})，cannot resume，degraded
```

**Correction loop degradation**:
```
- {yymmdd hhmm} ⚠️ {contract list} 3 rounds still have blocker/major FAIL，auto-degraded
```

---

### 8. Long-running Execution Support

#### 8.1 Checkpoint Management

**Checkpoint file**: `{PROJECT_ROOT}/outputs/checkpoint.json`

**Checkpoint structure**:
```json
{
  "version": "1.0",
  "phase": "blockchain_batch_dev",
  "lastUpdated": "yymmdd hhmm",
  "currentBatch": 3,
  "totalBatches": 7,
  "completedContracts": ["token-contract", " governance-contract", "staking-contract"],
  "pendingContracts": ["vault-contract", "bridge-contract", "oracle-contract"],
  "activeSessions": {
    "dev": {"id": "abc123", "skill": "bc_solidity_dev", "createdAt": "yymmdd hhmm", "status": "active"},
    "test_func": {"id": "def456", "skill": "bc_tester_functional", "createdAt": "yymmdd hhmm", "status": "active"},
    "test_security": {"id": "ghi789", "skill": "bc_tester_security", "createdAt": "yymmdd hhmm", "status": "active"},
    "test_gas": {"id": "jkl012", "skill": "bc_tester_gas", "createdAt": "yymmdd hhmm", "status": "pending"}
  },
  "currentFixRound": 0,
  "metrics": {
    "totalAgentCalls": 45,
    "startTime": "yymmdd hhmm",
    "batchDurations": [12, 15, 18]
  }
}
```

**Checkpoint update timing**:
1. Before each batch starts: update currentBatch and pendingContracts
2. After each batch completes: update completedContracts and metrics
3. After agent session created: update activeSessions
4. After agent session ends: clear corresponding session record

**Checkpoint recovery flow**:
1. Read checkpoint.json
2. Verify if sessions in activeSessions are still valid
3. Invalid sessions: re-read status from dev-plan.md
4. Valid sessions: resume directly

---

#### 8.2 Timeout Detection and Recovery (integrated from docs/timeout-recovery.md)

**Timeout threshold configuration**:
| Detection Item | Threshold | Detection Frequency | Auto Handling |
|---------------|-----------|---------------------|---------------|
| Sub-Agent startup | 60s | Every 30s | Auto restart Agent |
| Sub-Agent execution | 300s | Every 60s | Skip task, continue next batch |
| Single batch duration | 1800s | Every 120s | Skip current batch |
| Test Agent | 300s | Every 60s | Handle as FAIL, trigger fix |
| Session validity | 120min | Every 30min | Auto refresh session |

**Auto recovery flow (no human intervention)**:
```
Timeout detected
├── Sub-Agent timeout (300s no response)
│   ├── 1st time: log, wait 30s
│   ├── 2nd time: log, create new session
│   └── 3rd time: skip task, mark as ⚠️ degraded, continue next batch
├── Test Agent timeout (300s no response)
│   └── Handle as FAIL, trigger fix loop
├── Batch timeout (1800s)
│   └── Skip current batch, log, continue next batch
└── Session expired (120min)
    └── Auto create new session, restore from checkpoint
```

**Auto skip rules**:
1. Sub-Agent 3 consecutive timeouts → auto skip contract, mark as ⚠️ degraded
2. Test Agent timeout → handle as FAIL, enter fix loop
3. Fix loop exceeds 3 rounds → auto degrade to ⚠️, continue next batch
4. Batch timeout → skip entire batch, continue next batch
5. **All processing fully automatic, no user inquiry, no flow blocking**

#### Health Check Mechanism (integrated from docs/monitoring-alerting.md)

**Check frequency**: Every 30 seconds

**Check items**:
| Check Item | Method | Timeout | Handling |
|------------|--------|---------|----------|
| Agent response | ping | 10s | Restart if timeout |
| Task progress | check_output | 30s | Alert if no progress |
| Resource status | check_resources | 5s | Pause if insufficient |
| Network status | ping_api | 10s | Wait if disconnected |

**Health status**:
- healthy: normal operation, continue
- degraded: partial function affected, warn and continue
- unhealthy: cannot operate normally, pause and recover

#### Error Classification and Recovery (integrated from docs/error-recovery.md)

**Error classification**:
| Error Type | Characteristics | Handling Strategy |
|------------|-----------------|-------------------|
| Recoverable | Timeout, network issues, API limits | Auto retry (exponential backoff) |
| Non-recoverable | Logic errors, code errors | Log and skip |
| Platform errors | Service unavailable, quota exhausted | Wait or switch |

**Retry strategy**:
- Max retries: 3
- Backoff strategy: exponential (1s, 2s, 4s)
- Retryable errors: auto retry
- Non-retryable errors: log and skip

#### Monitoring Metrics (integrated from docs/observability.md)

**Key metrics**:
| Metric | Threshold | Monitoring Frequency |
|--------|-----------|---------------------|
| Agent response time | < 60s | Real-time |
| Batch execution time | < 1800s | Per batch |
| Fix rounds | < 3 | Per task |
| Error rate | < 10% | Per batch |
| Timeout rate | < 5% | Per batch |

**Alert rules**:
| Alert | Condition | Level | Handling |
|-------|-----------|-------|----------|
| Agent timeout | Response > 300s | warning | Auto recovery |
| Batch timeout | Execution > 1800s | critical | Skip batch |
| High error rate | Error > 10% | critical | Pause check |
| High fix rate | Fix > 3 rounds | warning | Log analysis

#### Diagnostic Commands (integrated from docs/troubleshooting.md)

**System status check**:
```bash
# Check latest log
tail -20 {PROJECT_ROOT}/outputs/main-log.md

# Check recent events
tail -20 {PROJECT_ROOT}/outputs/events.jsonl

# Check timeout records
grep "timeout" {PROJECT_ROOT}/outputs/events.jsonl

# Check error records
grep "error" {PROJECT_ROOT}/outputs/events.jsonl

# Check Agent status
grep "agent_spawn\|agent_complete" {PROJECT_ROOT}/outputs/events.jsonl | tail -10
```

**Emergency recovery**:
```bash
# Restore from checkpoint
opencode
# Select main agent, system auto recovers

# Skip stuck task
vi {PROJECT_ROOT}/outputs/checkpoint.json
# Modify currentBatch to increase by 1

# Reset state
rm {PROJECT_ROOT}/outputs/checkpoint.json
rm -rf {PROJECT_ROOT}/outputs/agent-registry/
```

---

#### 8.3 Session Keep-alive Strategy

**Session lifecycle**:
- Session validity: default 2 hours
- Keep-alive check interval: every 30 minutes

**Keep-alive check flow**:
Execute before each batch starts:

1. Read activeSessions from checkpoint.json
2. Check createdAt time for each session
3. Sessions alive over 90 minutes:
   - Mark as needsRefresh
   - After current batch completes, create new session
   - New session recovers context by reading latest status

4. Sessions already expired (cannot resume):
   - Recover last state from checkpoint.json
   - Create new session to continue execution
   - Log: session rebuilt

**New session context recovery**:
When creating new session, prompt must include:
- Complete task list for current batch
- Status of completed contracts
- Recent test report summary (only verdict field)
- Current correction round (if any)

---

#### 8.3 Context Window Management

**Context budget**:
- Master agent: keep last 50 rounds of conversation
- Sub-agents: new session per batch, no cross-batch accumulation

**Auto-compression strategy**:
Execute context compression after every 3 batches:

1. **Content to keep**:
   - Current plan (dev-plan.md pending portion)
   - Key decision summary
   - Unresolved issue list
   - Detailed status of last 1 batch

2. **Content to compress**:
   - Completed batches → only keep statistical summary
   - Resolved test issues → only keep count
   - Intermediate states → merge into final state

3. **Output after compression**:
   - Write to `{PROJECT_ROOT}/outputs/context-summary.md`
   - Subsequent sessions read this file to recover context

**Context overflow handling**:
When detecting context approaching limits:
1. Auto-trigger compression
2. Force new sessions for sub-agents
3. Master agent keeps minimal working set

---

#### 8.4 Structured Logging System

**Dual-track logging**:
Maintain two log formats simultaneously:

1. **Human-readable log** (main-log.md):
   - Format: `- {yymmdd hhmm} {event description}`
   - Purpose: quick browsing, manual review

2. **Machine-readable log** (events.jsonl):
   - Format: one JSON object per line
   - Purpose: program parsing, state recovery, statistical analysis

**events.jsonl event types**:
```json
// Batch start
{"ts":"yymmdd hhmm","event":"batch_start","batch":3,"contracts":["vault-contract","bridge-contract"]}

// Agent spawn
{"ts":"yymmdd hhmm","event":"agent_spawn","type":"bc_solidity_dev","id":"abc123","batch":3}

// Agent complete
{"ts":"yymmdd hhmm","event":"agent_complete","type":"bc_solidity_dev","id":"abc123","duration_sec":420}

// Test result
{"ts":"yymmdd hhmm","event":"test_result","contract":"vault-contract","dimension":"functional","verdict":"PASS","warnings":0}

// Correction round
{"ts":"yymmdd hhmm","event":"fix_round","batch":3,"round":1,"contracts":["vault-contract"],"issues":["reentrancy vulnerability"]}

// Batch complete
{"ts":"yymmdd hhmm","event":"batch_complete","batch":3,"duration_min":15,"agent_calls":4,"pass_rate":0.67}

// Checkpoint update
{"ts":"yymmdd hhmm","event":"checkpoint_update","batch":4,"completed":["token-contract","governance-contract"],"pending":["vault-contract","bridge-contract"]}

// Session refresh
{"ts":"yymmdd hhmm","event":"session_refresh","type":"bc_solidity_dev","old_id":"abc123","new_id":"xyz789","reason":"expired"}

// Phase complete
{"ts":"yymmdd hhmm","event":"phase_complete","phase":"blockchain","total_batches":7,"total_contracts":20,"duration_min":180}
```

---

#### 8.5 Smart Retry Strategy

**Graded handling strategy**:

| Issue Level | Handling | Degradation Condition |
|------------|---------|---------------------|
| blocker | Must fix, no degradation allowed | Pause after 3 rounds, request human intervention |
| major | Must fix, degradation allowed | Degrade after 3 rounds, record for follow-up |
| minor | Record technical debt | Don't block, skip directly |

**Blocker level handling**:
1. Still have blocker after round 3 → pause automatic flow
2. Generate detailed problem report:
   - Problem description
   - Tried fix approaches
   - Related code location
   - Suggested human handling direction
3. Write to `{PROJECT_ROOT}/outputs/needs-human-review.md`
4. Report to user, wait for human decision

**Major level handling**:
1. Still have major after round 3 → auto-degrade to ⚠️
2. Record to `{PROJECT_ROOT}/outputs/technical-debt.md`
3. Format:
   ```markdown
   - [MAJOR] {contract name} - {problem description}
     - Discovery time: {yymmdd hhmm}
     - Test dimension: {dimension}
     - Impact scope: {description}
     - Suggested fix: {suggestion}
   ```

---

#### 8.6 Knowledge Base Integration

**Knowledge base file**: `{PROJECT_ROOT}/outputs/knowledge-base.json`

**Master agent responsibilities**:
1. **Initialize knowledge base**: Create empty knowledge-base.json structure on first startup
2. **Pass knowledge base path**: Pass knowledge-base.json path to development sub-agents
3. **Read knowledge base summary**: Before each batch, read patterns and antiPatterns counts from knowledge-base.json to understand known issues
4. **Don't directly modify knowledge base**: Knowledge base is maintained by development sub-agents, master agent only reads summary

**Knowledge base application**:
1. **Batch planning**: Reference historical fix data to adjust batch size
2. **Test result analysis**: Identify if issues match known patterns
3. **Report generation**: Reference statistics from knowledge base

**Knowledge base summary reading**:
```markdown
Read knowledge-base.json, extract:
- patterns count: {count}
- antiPatterns count: {count}
- Most common fixStrategy: {problemType} - {bestApproach}
- High-frequency issue types: {category}
```

---

#### 8.7 Observability Enhancement

**Observability components**:
1. `status-tracker.json`: Real-time status tracking
2. `metrics.json`: Performance metrics collection
3. `alerts.jsonl`: Anomaly detection and alerting

**Master agent responsibilities**:
1. **Initialize observability components**: Create status-tracker.json, metrics.json, alerts.jsonl on first startup
2. **Update status tracking**: Update status-tracker.json at batch start/end
3. **Collect performance metrics**: Update metrics.json after each batch completes
4. **Detect anomalies**: Check anomaly rules before each batch, generate alerts when anomalies detected
5. **Display observability data**: Show status, metrics, alerts in dashboard

**Anomaly detection rules**:
| Anomaly Type | Detection Condition | Severity | Handling |
|---------|---------|---------|---------|
| Agent timeout | Single call > 300s | warning | Log, continue waiting |
| Consecutive timeouts | Same agent 3 consecutive timeouts | critical | Pause agent, create new session |
| Batch timeout | Single batch > 60 min | warning | Log, continue execution |
| High fix rate | 3 consecutive batches with fix rate > 50% | warning | Log, analyze reason |
| Context overflow | Context usage > 90% | critical | Trigger compression, new session |
| Session expired | Session alive > 2 hours | warning | Auto-refresh session |
| System stall | No progress update for 30 min | critical | Check system status, resume execution |

**Alert handling flow**:
1. Detect anomaly → 2. Generate alert (write to alerts.jsonl) → 3. Execute action → 4. Record result → 5. Display in dashboard

---

### 8. Key Rules

1. **Resume uses Agent ID** — must use `task_id: "{DEV_ID}"` format (value of `id` field in Agent Registry JSON), with `subagent_type: "general"`. Call `skill(name: "...")` to load the corresponding skill before resume
2. **Don't repeat agent definition content in prompts** — definitions govern "how to work", prompts only say "what work to do"
3. **Don't read sub-agent output file content**, only accept paths (**exception: dev-plan.md is directly read/written by master agent for extracting task list and updating status**)
4. **Must update dev-plan.md after each batch of tasks completes**
5. **Write log for each key step** (time format yymmdd hhmm)
6. **Report progress in bulk after each batch**
7. **dev-plan.md managed by master agent, sub-agents do not modify it**
8. **Test reports written by test agents, read by dev agent**
9. **lessons-learned.md updated by dev agent after fixes**
10. **After each batch's dev rounds end, DEV_ID and TEST_*_ID all expire; new batch restarts all agents**
11. **Severity grading** — FAIL in test reports is graded as blocker/major/minor: blocker (contract not compilable/security vulnerability), major (core function defect/gas exceeds limit), minor (acceptable optimization). Only minor level allows ⚠️ degradation
12. **No rollback** — After 3 correction rounds, blocker/major are auto-degraded to ⚠️, logged, no retry, no user inquiry
13. **Correction cost insight** — Higher correction rounds indicate prompt or dev quality issues; prioritize recording in lessons-learned

### 9. Data Access Boundaries

The master agent's "don't read" principle is not absolute — it has clear boundaries. Below table defines access permissions for each data item:

| Data Item | Readable? | Read Method | Purpose |
|--------|---------|---------|---------|
| Architecture docs (TECH_STACK_FILE etc.) | **No** | Only pass paths to sub-agents | Protect context; sub-agents read it themselves |
| Requirements doc (REQUIREMENT_FILE) | **No** | Only pass paths to sub-agents | Protect context; sub-agents read it themselves |
| dev-plan.md | **Yes** | Read full text (but only task list portion) | Extract ⏳ task list, update completion status |
| test-report.json | **Yes (only verdict and severity fields)** | Read to extract `verdict` field | Determine PASS/FAIL, decide if correction needed |
| Test report markdown full text | **No** | Pass path to dev agent, dev agent reads it | Protect context |
| lessons-learned.md | **No** | Maintained by dev agent, master agent doesn't read | Protect context |
| Contract files (.sol) | **No** | All delegated to dev agent | Prevent unauthorized modifications |

**Core principle**: Master agent only reads two types of data — (a) structured status (dev-plan.md task list, test-report.json verdict/severity fields), (b) paths and names. Everything else is read by sub-agents.

### 10. Supplementary Rules (11-17)

11. **Architecture docs: pass paths only, don't read content** — during initialization, only record paths; pass them to bc_planner to read itself
12. **Test results: only read JSON verdict** — read `verdict` field from test-report.json, don't Read full report
13. **All code modifications delegated to bc_solidity_dev** — even one-line changes must be delegated (skill(name: "bc_solidity_dev") + Task(subagent_type: "general")); master agent never touches contract source code
14. **Brief acknowledgment for background notifications** — late background agent notifications only need "已确认" reply, don't repeat content
15. **Dev batch size = Test batch size** — default BATCH_SIZE=1 (single contract), user can specify N. When developing N contracts, testing is also 3 agents each testing N. Dev and test batch sizes stay consistent
16. **Concurrency limit always 3** — testing phase always has only 3 agents in parallel (functional/security/gas one each), each agent internally handles all contracts in this batch. Dev phase only launches 1 dev agent per batch
17. **Cost tracking rule**: After each batch, append that batch's agent call count (dev + test + fix) to main-log.md. Sum total calls at end of Phase. Prioritize correction round cost — more correction rounds = potential prompt or PRD quality issues

### 11. Relationship with Other Systems

```
architecture/ → frontend/ + backend/ + flutter/ + blockchain/ → fullstack/ → deploy/
   (Phase 0)               (Phase 1, parallel)                    (Phase 2)    (Phase 3)
```

blockchain/ runs in parallel with frontend/, backend/, flutter/, sharing the same architecture design outputs. Its smart contract and ABI outputs are integrated with backend APIs during the fullstack/ integration phase — backend calls on-chain contracts via the chain's SDK, frontend reads/writes on-chain data indirectly through backend APIs.

## Tags

- domain: blockchain
- role: orchestrator
- version: 2.0.0
