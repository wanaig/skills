# main-log.md 标准模板

所有领域主Agent 的日志文件格式标准。日志文件位于各领域 `{PROJECT_ROOT}/outputs/main-log.md`。

---

## 格式规范

| 规则 | 说明 |
|------|------|
| 行前缀 | 每行以 `- ` 开头（短横+空格） |
| 时间格式 | `yymmdd hhmm`（如 `260520 1430`），精确到分钟，每次写入取当前时间 |
| 追加写入 | 只追加不覆盖，保留完整历史 |
| 日志级别 | 普通事件不加标记，异常/警告用 `⚠️` 前缀 |

---

## 模板结构

### 第 1 部分：启动日志

项目启动时写入，记录所有输入参数和初始配置。

```markdown
- {yymmdd hhmm} 项目启动，需求：{REQUIREMENT_FILE}
- {yymmdd hhmm} 技术栈：{TECH_STACK_FILE}
- {yymmdd hhmm} 数据架构：{DATA_ARCHITECTURE_FILE}（如有）
- {yymmdd hhmm} API 契约：{CONTRACT_FILE}（如有）
- {yymmdd hhmm} 安全架构：{SECURITY_FILE}（如有）
- {yymmdd hhmm} UI/UX 架构：{UI_UX_FILE}（如有）
- {yymmdd hhmm} 实施路线图：{IMPLEMENTATION_ROADMAP_FILE}（如有）
- {yymmdd hhmm} 前端项目：{FRONTEND_ROOT}（如有）
- {yymmdd hhmm} 后端项目：{BACKEND_ROOT}（如有）
- {yymmdd hhmm} Flutter 项目：{FLUTTER_ROOT}（如有，无则 N/A）
- {yymmdd hhmm} 区块链项目：{BLOCKCHAIN_ROOT}（如有，无则 N/A）
- {yymmdd hhmm} 批量大小：{BATCH_SIZE}
- {yymmdd hhmm} 成本追踪：本轮预计调用 {N} 个Agent
```

### 第 2 部分：Phase 日志

每个关键步骤完成后立即写入，不延迟、不批量。

**计划阶段**：
```markdown
- {yymmdd hhmm} 启动计划子Agent
- {yymmdd hhmm} 计划完成：{N} 个任务，基础已就绪
- {yymmdd hhmm} dev-plan: {路径}
- {yymmdd hhmm} design-guide: {路径}
- {yymmdd hhmm} 项目框架: {路径}
```

**每批开发/联调日志**：
```markdown
- {yymmdd hhmm} ── Batch {N}: {批次主题} ──
- {yymmdd hhmm} 本批开发启动：{任务1} ({描述1}), {任务2} ({描述2}), ...
- {yymmdd hhmm} 本批开发完成：{任务列表} 已创建 (DEV_ID: {xxx})
- {yymmdd hhmm} 首次测试 {任务1}：维度1{P/F} / 维度2{P/F} / 维度3{P/F}
- {yymmdd hhmm} 首次测试 {任务2}：维度1{P/F} / 维度2{P/F} / 维度3{P/F}
- {yymmdd hhmm} 测试AgentID：维度1={ID1} / 维度2={ID2} / 维度3={ID3}
- {yymmdd hhmm} 第1轮修正：{FAIL任务列表} (DEV_ID: {xxx})
- {yymmdd hhmm} 第1轮重测 {任务1}：维度1{P/F}(ID:{xxx}) / 维度2{P/F}(ID:{xxx})
- {yymmdd hhmm} {任务1} 完成，迭代{round}次
- {yymmdd hhmm} {任务2} 完成，迭代{round}次
- {yymmdd hhmm} Batch {N} 完成：全部{PASS/⚠️降级}，成本={N}次调用
```

### 第 3 部分：收尾日志

```markdown
- {yymmdd hhmm} ──── 项目完成 ────
- {yymmdd hhmm} 全部 {N} 个任务完成
- {yymmdd hhmm} 迭代统计：1次通过 {X} 个 / 2次通过 {Y} 个 / 3次通过 {Z} 个 / 自动降级 {W} 个
- {yymmdd hhmm} 总Agent调用次数：{开发{N}+测试{M}+修正{K}}
- {yymmdd hhmm} 成本追踪汇总：Plan{N}次 / Dev{N}次 / Test{M}次 / Fix{K}次，修正轮次合计{R}轮
```

### 第 4 部分：异常事件日志

当异常发生时立即写入。所有异常不阻塞流程，记录后继续推进。

**Agent 超时**：
```markdown
- {yymmdd hhmm} Agent超时：{agent_type}（{agent_id}），超时批次 {batch}，降级通过
```

**Agent Registry 读取失败**：
```markdown
- {yymmdd hhmm} ⚠️ agent-registry/{key}.json 读取失败，无法获取 {Agent名} ID，降级通过
```

**Agent 会话过期（无法 resume）**：
```markdown
- {yymmdd hhmm} ⚠️ {Agent名} 会话过期（ID: {agent_id}），无法 resume，保留当前版本，降级通过
```

**修正循环降级**：
```markdown
- {yymmdd hhmm} ⚠️ {任务列表} 3轮修正后仍有 blocker/major FAIL，自动降级为 ⚠️ 通过
```

**一致性冲突（仅 architecture）**：
```markdown
- {yymmdd hhmm} 一致性冲突：{维度A} ↔ {维度B}，{冲突描述}，severity={blocker/major/minor}
- {yymmdd hhmm} 一致性第{N}轮修正：{修正的维度列表}，minor 跳过：{N}个
```

**制品提取跳过（仅 architecture）**：
```markdown
- {yymmdd hhmm} 制品 {制品名} 跳过：{来源文档} 未产出对应章节
```

---

## 各领域 adaption 指南

| 领域 | 任务单元 | 测试维度 | 计划文件 | 特有日志项 |
|------|---------|---------|---------|-----------|
| architecture | 维度 | 交叉检查 40 项 | — | 一致性冲突、制品提取、PRD 预检、ADR 记录 |
| frontend | 模块 | component/logic/style | dev-plan.md | design-guide |
| backend | 接口 | functional/performance/security | dev-plan.md | api-design-guide |
| flutter | 模块 | crossplatform/logic/style | dev-plan.md | design-guide |
| blockchain | 合约 | functional/security/gas | dev-plan.md | contract-design-guide |
| fullstack | 接口 | contract/dataflow/integration | integration-plan.md | integration-design-guide |
| deploy | 配置项 | 安全/完整性/一致性 | deploy-plan.md | deploy-checklist |
