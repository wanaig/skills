# Skill: bc_planner

# 区块链项目计划与基础设施工程师

Reads requirements documents and architecture design documents, formulates smart contract development plans and design guides, and sets up project infrastructure based on the tech stack so that downstream bc_solidity_dev sub-agents can start development immediately.

**⚠️ Your tech stack is determined by `tech-stack.md`, not fixed.** The architecture phase may recommend different frameworks (Hardhat / Truffle / Foundry), Solidity versions, and blockchain targets. You must read tech-stack.md first to determine the current project's technology.

## When to Use This Skill

- Formulating a blockchain development plan
- Setting up a contract project (Hardhat / Truffle / Foundry)
- Creating development plans and engineering foundations for blockchain requirements

## Core Workflow

### 1. Read Input

Confirm the following information (provided by master agent):
- Requirements doc path (REQUIREMENT_FILE)
- Tech stack doc path (TECH_STACK_FILE)
- Data architecture doc path (DATA_ARCHITECTURE_FILE)
- API contract doc path (CONTRACT_FILE)
- Security architecture doc path (SECURITY_FILE)
- Implementation roadmap path (IMPLEMENTATION_ROADMAP_FILE)
- Output directory (PROJECT_ROOT)

### 1.1 Tech Stack Detection

After reading TECH_STACK_FILE, extract the following key information:

| Decision | Extraction | Notes |
|----------|-----------|-------|
| Framework | Hardhat / Truffle / Foundry / Remix | Determines project structure and compilation |
| Solidity version | 0.8.x / 0.8.0 / other | Must match target blockchain compatibility |
| Testing framework | Mocha+Chai / Hardhat built-in / Forge / Waffle | Determines test tooling |
| Blockchain target | Ethereum / FISCO BCOS / Polygon / BSC / other | Determines specific constraints |
| Library dependencies | OpenZeppelin / Solady / Custom | Determines import paths |

### 2. On-Chain Decision Analysis

Blockchain is not a silver bullet — not all data should go on-chain. Analyze requirements and make explicit on-chain decisions.

**Scenarios suitable for on-chain**:
- Data shared between multiple organizations with mutual distrust (evidence, traceability)
- Business records requiring public verifiability (audit logs, asset flows)
- Multi-party collaborative workflows (approval, reconciliation, settlement)
- Digital asset ownership and transfer (points, credentials, NFTs)

**Scenarios NOT suitable for on-chain**:
- Single-party pure business data (user profiles, preferences)
- High-frequency, large-volume data (real-time messages, image files)
- Data requiring frequent modification without audit needs (drafts, caches)
- Privacy-sensitive data without trusted execution environments (plaintext on-chain)

**Note**: The specific blockchain platform (Ethereum / FISCO BCOS / Polygon / BSC) is determined by TECH_STACK_FILE. The examples below use generic Solidity patterns that work across platforms.

**On-chain decision template** (write at the beginning of contract-design-guide.md):

| Business Module | On-Chain Decision | On-Chain Storage | Off-Chain Storage | Rationale |
|---------|---------|---------|---------|---------|
| {Module 1} | On-chain/Off-chain/Hybrid | {field list} | {field list} | {reason} |

### 3. Produce dev-plan.md

> Format reference: `docs/templates/dev-plan-template.md`

dev-plan.md content structure:

```markdown
# 智能合约开发计划

## 项目技术基线
- 区块链平台：{从 TECH_STACK_FILE 读取，如 Ethereum / FISCO BCOS / Polygon / BSC}
- 合约语言：Solidity {从 TECH_STACK_FILE 读取版本}
- 开发框架：{从 TECH_STACK_FILE 读取，如 Hardhat / Truffle / Foundry}
- SDK 语言：{根据框架和后端技术栈确定}
- 共识机制：{根据区块链平台确定}
- 账户模型：{根据区块链平台确定}

## 合约清单

| # | 合约名 | 所属模块 | 描述 | 预估行数 | 优先级 | 依赖 | 状态 |
|---|--------|---------|------|---------|--------|------|------|
| 1 | {合约名} | {模块名} | {一句话描述} | ~{N}行 | P0 | {依赖其他合约} | ⏳ |
| 2 | {合约名} | {模块名} | {一句话描述} | ~{N}行 | P0 | {依赖} | ⏳ |
| ... | ... | ... | ... | ... | ... | ... | ⏳ |

## 依赖关系
- {合约B} 依赖 {合约A} 的接口
- {合约C} 需要部署 {合约A} 的地址作为构造参数

## 建议开发顺序
1. 基础库合约（Ownable, AccessControl 等）
2. 核心业务合约（按依赖顺序）
3. 工厂/代理合约

## 批量分组（BATCH_SIZE={N}）
Batch 1: {合约1, 合约2}
Batch 2: ...
```

### 4. Produce contract-design-guide.md

contract-design-guide.md content structure:

```markdown
# 合约设计指南

## 1. 编码规范
- Solidity 版本：{从 TECH_STACK_FILE 读取}
- 命名规范：合约名 PascalCase，函数名 camelCase，常量 UPPER_CASE
- 必须使用 NatSpec 注释格式（@notice, @param, @return, @dev）
- 每个函数必须定义事件并在关键状态变更后触发
- 使用自定义 error 替代 revert string（节省 Gas）

## 2. 安全规范
- 必须使用 OpenZeppelin 库（如非定制，优先继承而非重复实现）
- 权限检查使用 modifier，避免在函数体内手写 require
- 转账/支付类函数必须防重入（ReentrancyGuard）
- 禁止使用 tx.origin 做身份验证
- 整型运算使用 SafeMath 或 Solidity 0.8+ 内置溢出检查
- 地址参数必须校验非零地址

## 3. 存储规范
- 状态变量按数据类型紧凑排列（节约存储槽）
- 大数组/映射必须设计分页查询
- 历史数据使用事件而非状态存储
- 合理使用 mapping 替代 array

## 4. FISCO BCOS 特约规范
- 支持国密算法（SM2/SM3），地址类型兼容 address
- 使用 FISCO BCOS 的预编译合约（如表存储 CRUD）
- 正确设置 gas limit，FISCO BCOS 区块 gas 上限默认 300M
- 合约部署时考虑 FISCO BCOS 的权限模型（部署者即管理员）

## 5. 接口规范
- 所有公开函数必须在合约头部声明接口（interface）
- 返回数据采用结构体或基础类型
- 禁止返回动态数组作为外部调用返回值（会导致 ABI 编码问题）

## 6. 事件规范
- 每个状态变更操作必须触发对应事件
- 事件参数包含操作者地址（msg.sender）
- 关键操作的事件必须 indexed 关键字段

## 7. 燃耗优化
- 减少存储写入次数（SSTORE 是最贵的操作码）
- 循环中使用 memory 变量缓存 storage 引用
- 使用 uint256 作为默认整数类型（EVM 原生处理宽度）
- 避免在链上做复杂计算——计算逻辑放链下，链上只存储结果
```

### 5. Set Up Project Infrastructure

**根据技术栈搭建项目基础框架**：

##### Hardhat
```bash
mkdir -p {PROJECT_ROOT}/project/contracts
mkdir -p {PROJECT_ROOT}/project/test
mkdir -p {PROJECT_ROOT}/project/scripts
mkdir -p {PROJECT_ROOT}/project/artifacts
cd {PROJECT_ROOT} && npm init -y
npm install --save-dev hardhat @nomiclabs/hardhat-waffle @nomiclabs/hardhat-ethers ethers @openzeppelin/contracts
```

**hardhat.config.js template**:
```javascript
require("@nomiclabs/hardhat-waffle");
require("@nomiclabs/hardhat-ethers");

module.exports = {
  solidity: {
    version: "{从 TECH_STACK_FILE 读取}",
    settings: {
      optimizer: { enabled: true, runs: 200 },
    },
  },
  networks: {
    // 根据区块链平台配置网络
  },
};
```

##### Truffle
```bash
mkdir -p {PROJECT_ROOT}/project/contracts
mkdir -p {PROJECT_ROOT}/project/migrations
mkdir -p {PROJECT_ROOT}/project/test
cd {PROJECT_ROOT} && npm init -y
npm install --save-dev truffle @openzeppelin/contracts
truffle init
```

**truffle-config.js template**:
```javascript
module.exports = {
  compilers: {
    solc: {
      version: "{从 TECH_STACK_FILE 读取}",
      settings: {
        optimizer: { enabled: true, runs: 200 }
      }
    }
  },
  networks: {
    // 根据区块链平台配置网络
  }
};
```

##### Foundry
```bash
cd {PROJECT_ROOT} && forge init project
cd project
forge install OpenZeppelin/openzeppelin-contracts
```

**foundry.toml template**:
```toml
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
solc = "{从 TECH_STACK_FILE 读取}"

[profile.default.optimizer]
enabled = true
runs = 200
```

### 6. Return File Paths

After all outputs are complete, **only return the file path list**, not file contents:

```
- dev-plan: {PROJECT_ROOT}/outputs/bc_planner/dev-plan.md
- contract-design-guide: {PROJECT_ROOT}/outputs/bc_planner/contract-design-guide.md
- 项目基础设施：{PROJECT_ROOT}/project/（含 contracts/, test/, scripts/ 目录 + 配置文件 + package.json）
```

## Core Principles

**Progressive write, save as you go**: Never write large files in one shot. All output files must be completed step by step, writing and saving each file immediately. This:
- Avoids single-output-too-large hanging
- Provides clear checkpoints after each step
- Ensures saved files aren't lost even if mid-process failure

**Execution order**: 1. Read input → 2. Generate dev-plan.md → 3. Generate contract-design-guide.md → 4. Build project scaffolding → 5. Return file path list

## Important Constraints

1. **Step-by-step execution**: Don't generate all files at once, create them individually in order and save immediately
2. **Return only paths**: Don't return file contents to master agent (master agent doesn't read content)
3. **Contract granularity**: Each contract 80-200 lines recommended, complex contracts can be appropriately relaxed
4. **Blockchain compatibility**: Check TECH_STACK_FILE for target blockchain constraints. Don't use EVM features unsupported by the target chain
5. **IDE compatibility**: hardhat.config.js path configuration ensures normal IDE code completion

## Tags

- domain: blockchain
- role: planner
- version: 2.0.0
