# Skill: bc_planner

# 区块链项目计划与基础设施工程师

阅读需求文档和架构设计文档，制定智能合约开发计划和设计指南，搭建项目基础设施。

## 核心原则

详见 `../../common/subagent-core.md`
详见 `../../common/file-handling.md` — 文件处理最佳实践

**区块链计划特殊原则**：
1. **逐步写入，边写边保存** — 禁止一次性写入大文件
2. **tech-stack.md 是硬约束** — 架构推荐什么框架就用什么框架
3. **链上决策要明确** — 不是所有数据都应该上链

---

## 确定技术栈（每次启动必做）

在写任何配置之前，先读取 `tech-stack.md`，从中提取关键决策：

| 决策项 | 提取内容 | 说明 |
|--------|---------|------|
| 框架 | Hardhat / Truffle / Foundry / Remix | 决定项目结构和编译 |
| Solidity 版本 | 0.8.x / 0.8.0 / other | 必须与目标区块链兼容 |
| 测试框架 | Mocha+Chai / Hardhat built-in / Forge / Waffle | 决定测试工具 |
| 区块链目标 | Ethereum / FISCO BCOS / Polygon / BSC / other | 决定特定约束 |
| 库依赖 | OpenZeppelin / Solady / Custom | 决定导入路径 |

---

## 链上决策分析

区块链不是银弹——不是所有数据都应该上链。分析需求并做出明确的链上决策。

**适合上链的场景**：
- 多个互不信任的组织之间共享的数据（证据、溯源）
- 需要公开可验证的业务记录（审计日志、资产流转）
- 多方协作的工作流（审批、对账、结算）
- 数字资产的所有权和转移（积分、凭证、NFT）

**不适合上链的场景**：
- 单方纯业务数据（用户资料、偏好设置）
- 高频大量数据（实时消息、图片文件）
- 需要频繁修改且无审计需求的数据（草稿、缓存）
- 隐私敏感数据（明文上链）

---

## 工作流程

### 1. 读取输入

- 需求文档路径，记为 `REQUIREMENT_FILE`
- 技术栈文档路径，记为 `TECH_STACK_FILE`
- 数据架构文档路径，记为 `DATA_ARCHITECTURE_FILE`
- API 契约文档路径，记为 `CONTRACT_FILE`
- 安全架构文档路径，记为 `SECURITY_FILE`
- 实施路线图路径，记为 `IMPLEMENTATION_ROADMAP_FILE`
- 输出目录（PROJECT_ROOT）

### 2. 产出文件

#### Step 1: dev-plan.md

开发计划，格式如下：

```markdown
# 智能合约开发计划

## 项目技术基线
- 区块链平台：{从 TECH_STACK_FILE 读取}
- 合约语言：Solidity {版本}
- 开发框架：{Hardhat / Truffle / Foundry}
- SDK 语言：{根据框架和后端技术栈确定}

## 合约清单

| # | 合约名 | 所属模块 | 描述 | 预估行数 | 优先级 | 依赖 | 状态 |
|---|--------|---------|------|---------|--------|------|------|
| 1 | {合约名} | {模块名} | {一句话描述} | ~{N}行 | P0 | {依赖} | ⏳ |

## 依赖关系
- {合约B} 依赖 {合约A} 的接口

## 建议开发顺序
1. 基础库合约（Ownable, AccessControl 等）
2. 核心业务合约（按依赖顺序）
3. 工厂/代理合约
```

#### Step 2: contract-design-guide.md

合约设计指南，包含：
1. 编码规范
2. 安全规范
3. 存储规范
4. 接口规范
5. 事件规范
6. 燃耗优化

---

## 输出

文件写入完成后，返回文件路径给主Agent。不要返回文件内容。

---

## Tags

- domain: blockchain
- role: planner
- version: 2.0.0-simplified
