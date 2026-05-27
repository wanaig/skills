# Harness Engineering 在 OpenCode 中的使用指南

## 概述

Harness Engineering 多智能体协同开发系统已适配 OpenCode，可通过代理（Agents）方式使用。

## 快速开始

### 1. 安装 OpenCode

```bash
curl -fsSL https://opencode.ai/install | bash
```

### 2. 配置 API 密钥

```bash
opencode
/connect
```

选择提供商并配置 API 密钥。

### 3. 进入项目目录

```bash
cd /path/to/skills
```

### 4. 启动 OpenCode

```bash
opencode
```

## 代理使用方式

### 主代理（Primary Agents）

主代理可通过 **Tab** 键切换，用于不同开发阶段：

| 代理名称 | 用途 | 使用场景 |
|---------|------|----------|
| `prd-designer` | PRD设计 | Phase -1: 产品需求文档设计 |
| `fs-architect` | 全栈架构设计 | Phase 0: 技术架构设计 |
| `frontend-main` | 前端开发 | Phase 1a: Vue 3 前端开发 |
| `backend-main` | 后端开发 | Phase 1b: Spring Boot 后端开发 |
| `flutter-main` | Flutter 跨端开发 | Phase 1c: Flutter 跨端开发（可选） |
| `blockchain-main` | 区块链开发 | Phase 1d: 智能合约开发（可选） |
| `fullstack-main` | 前后端联调 | Phase 2: 前后端集成 |
| `deploy-main` | 部署上线 | Phase 3: 生产部署 |

### 子代理（Subagents）

子代理通过 **@** 提及调用，或由主代理自动调用：

#### PRD设计阶段子代理
- `@prd-business` - 业务分析
- `@prd-user` - 用户研究
- `@prd-functional` - 功能设计
- `@prd-technical` - 技术评估

#### 架构阶段子代理
- `@fa-techstack` - 技术栈评估
- `@fa-data` - 数据架构设计
- `@fa-infra` - 基础设施架构
- `@fa-security` - 安全架构设计
- `@fa-api-design` - API 契约设计
- `@fa-uiux` - UI/UX 架构设计

#### 前端子代理
- `@dg-vue-planner` - Vue 前端项目规划
- `@dg-frontend-vue-dev` - Vue 前端开发
- `@dg-vue-tester-component` - 组件结构测试
- `@dg-vue-tester-logic` - 逻辑测试
- `@dg-vue-tester-style` - 样式测试

#### 后端子代理
- `@be-planner` - 后端项目规划
- `@be-api-dev` - 后端 API 开发
- `@be-tester-functional` - 功能测试
- `@be-tester-performance` - 性能测试
- `@be-tester-security` - 安全测试

#### Flutter 子代理
- `@dg-flutter-planner` - Flutter 项目规划
- `@dg-flutter-dev` - Flutter 开发
- `@dg-flutter-tester-crossplatform` - 跨端兼容测试
- `@dg-flutter-tester-logic` - 逻辑测试
- `@dg-flutter-tester-style` - 样式测试

#### 区块链子代理
- `@bc-planner` - 区块链项目规划
- `@bc-solidity-dev` - Solidity 合约开发
- `@bc-tester-functional` - 功能测试
- `@bc-tester-security` - 安全测试
- `@bc-tester-gas` - 燃耗测试

#### 联调子代理
- `@fs-planner` - 联调规划
- `@fs-api-dev` - 接口对接开发
- `@fs-tester-contract` - 契约测试
- `@fs-tester-dataflow` - 数据流测试
- `@fs-tester-integration` - 集成测试

#### 部署子代理
- `@deploy-planner` - 部署计划
- `@deploy-infra` - 基础设施部署
- `@deploy-verifier` - 部署验证

## 使用流程

### Phase 0: 架构设计

1. 切换到 `fs-architect` 主代理（Tab 键）
2. 提供需求文档路径：
   ```
   REQUIREMENT_FILE=/path/to/prd.md
   PROJECT_ROOT=/path/to/architecture-output
   ```
3. 主代理自动协调 6 个子代理并行分析

### Phase 1: 并行开发

1. **前端开发**：切换到 `frontend-main`
   ```
   PROJECT_ROOT=/path/to/frontend
   REQUIREMENT_FILE=/path/to/prd.md
   TECH_STACK_FILE=/path/to/architecture-output/tech-stack.md
   CONTRACT_FILE=/path/to/architecture-output/api-contract.md
   ```

2. **后端开发**：切换到 `backend-main`
   ```
   PROJECT_ROOT=/path/to/backend
   REQUIREMENT_FILE=/path/to/prd.md
   TECH_STACK_FILE=/path/to/architecture-output/tech-stack.md
   DATA_ARCHITECTURE_FILE=/path/to/architecture-output/data-architecture.md
   ```

### Phase 2: 前后端联调

切换到 `fullstack-main`，提供前端和后端项目路径。

### Phase 3: 部署上线

切换到 `deploy-main`，提供部署配置。

## 权限配置

默认权限配置：
- **主代理**：禁止编辑文件，允许执行 Bash 命令（需确认）
- **开发子代理**：允许编辑文件和执行 Bash 命令
- **测试子代理**：禁止编辑文件，允许执行 Bash 命令
- **架构子代理**：禁止编辑文件和执行 Bash 命令

可在 `opencode.json` 中自定义权限。

## 配置文件说明

- `opencode.json`：主配置文件，包含所有代理定义和权限配置
- 各领域 `main_agent_prompt*.md`：主代理提示词文件
- 各领域 `agents/*.md`：子代理提示词文件

## 故障排除

### 代理未显示
1. 检查 `opencode.json` 语法是否正确
2. 确认代理名称符合规范（小写字母、数字、连字符）
3. 检查权限配置是否隐藏了代理

### 权限问题
1. 检查 `permission` 配置
2. 确认文件路径是否正确
3. 查看 OpenCode 日志

### 提示词加载失败
1. 确认提示词文件路径正确
2. 检查文件编码（UTF-8）
3. 验证文件内容是否完整

## 高级配置

### 自定义模型

在 `opencode.json` 中为特定代理配置模型：

```json
{
  "agent": {
    "fs-architect": {
      "model": "anthropic/claude-sonnet-4-20250514"
    }
  }
}
```

### 温度控制

```json
{
  "agent": {
    "fa-techstack": {
      "temperature": 0.3
    }
  }
}
```

### 最大步数限制

```json
{
  "agent": {
    "be-api-dev": {
      "steps": 20
    }
  }
}
```

## 相关链接

- [OpenCode 文档](https://opencode.ai/docs)
- [Harness Engineering 项目](./README.md)
- [设计原理](./docs/design_principles.md)
- [系统架构](./docs/architecture.md)