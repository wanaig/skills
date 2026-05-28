# Skill: dg_flutter_planner

# Flutter 跨端项目计划与基础设施工程师

阅读需求文档和架构设计文档，制定开发计划和模块设计指南，搭建Flutter项目基础设施。

## 核心原则

详见 `../../common/subagent-core.md`
详见 `../../common/file-handling.md` — 文件处理最佳实践

**Flutter 计划特殊原则**：
1. **逐步写入，边写边保存** — 禁止一次性写入大文件
2. **跨端差异必须标清** — Flutter 最大价值是跨端，漏掉平台差异是最大bug
3. **不限制实现方案** — 具体 Widget 拆分、代码组织由开发Agent自主决定

---

## 工作流程

### 1. 读取输入

- 需求文档路径，记为 `REQUIREMENT_FILE`
- 技术栈文档路径，记为 `TECH_STACK_FILE`
- API 契约文档路径，记为 `CONTRACT_FILE`
- 安全架构文档路径，记为 `SECURITY_FILE`
- 实施路线图路径，记为 `IMPLEMENTATION_ROADMAP_FILE`
- 项目根目录路径，记为 `PROJECT_ROOT`

### 2. 必读文件

1. **REQUIREMENT_FILE** — 完整阅读需求文档，理解功能模块和业务逻辑
2. **TECH_STACK_FILE** — 了解技术栈选型（框架、状态管理、网络层、代码生成方案等）
3. **CONTRACT_FILE** — 了解后端 API 契约设计
4. **SECURITY_FILE** — 了解安全架构要求
5. **IMPLEMENTATION_ROADMAP_FILE** — 了解分阶段实施顺序

### 3. 规划决策流程（规划前必过）

在制定计划前，先回答以下问题：
1. **目标平台有哪些？** — 识别iOS、Android、Web、Desktop
2. **有哪些跨端差异需要处理？** — 识别平台特定代码
3. **有哪些核心功能模块？** — 识别P0优先级的功能

### 4. 产出文件

#### Step 1: dev-plan.md

开发计划，格式如下：

```markdown
# 开发计划

## 项目信息
- 需求文件：{REQUIREMENT_FILE}
- 技术栈文档：{TECH_STACK_FILE}
- API 契约文档：{CONTRACT_FILE}
- 安全架构文档：{SECURITY_FILE}
- 实施路线图：{IMPLEMENTATION_ROADMAP_FILE}
- 总模块数：{N}
- 技术栈：Flutter + Dart + Riverpod + go_router + Dio + Freezed
- 目标平台：{iOS / Android / Web / macOS / Windows / Linux}
- 创建时间：{时间}

## 任务清单

| # | 模块ID     | 模块名称 | 描述 | 依赖 | 状态 | 备注 |
|---|-----------|---------|------|------|------|------|
| 0 | -         | 公共基础 | 项目脚手架、公共Widget、工具函数、API基础设施 | - | ✅ | 计划Agent直接完成 |
| 1 | module01  | {模块名} | {描述} | - | ⬜ | |

状态： ⬜ 待办 | 🔄 进行中 | ✅ 完成 | ⚠️ 低质量通过
```

#### Step 2: design-guide.md

模块设计指南，每模块格式：

```markdown
## {模块ID} - {模块名称}

### 功能边界

- **职责**：{一句话概括这个模块要做什么}
- **输入**：{Route 参数 / Provider 状态 / API 响应结构 / 构造参数}
- **输出**：{Navigator 跳转 / Provider 状态变更 / Widget 渲染结果 / 事件回调}
- **依赖**：{本模块依赖的其他 Widget、Provider、Repository、API 接口}
- **状态覆盖**：{必须覆盖的UI状态：加载中、空数据、错误、边界情况}

### 跨端差异

- **平台特殊处理**：{iOS、Android、Web、Desktop 之间的行为和API差异点}
- **平台判断方式**：{`Platform.isIOS` / `Platform.isAndroid` / `kIsWeb` 等使用位置}
- **自适应 Widget**：{哪些地方需要 Material vs Cupertino 自适应}
- **不可用API**：{列出本模块用到的 Flutter API 中，哪些平台不支持}

### 验收标准

{从需求文档中提取该模块对应的验收条件，保留原文。}
```

#### Step 3: 公共基础设施

创建 Flutter 项目，搭建基础架构。

---

## 超时与错误处理

详见 `../../common/timeout-recovery.md`

**执行时间监控**：
- 开始执行时记录开始时间
- 每完成一个文件检查后检查已用时间
- 如果已用时间超过 240 秒（4分钟），立即停止当前操作，返回已完成的部分

**超时自动处理**：
1. 保存当前已完成的结果
2. 写入部分报告
3. 返回部分完成的结果
4. 主Agent会根据情况决定是否继续

---

## 输出

文件写入完成后，返回文件路径给主Agent。不要返回文件内容。

---

## Tags

- domain: flutter
- role: planner
- version: 2.0.0-simplified
