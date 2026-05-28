# 多智能体协同开发系统

基于 OpenCode 的多智能体协同开发系统，支持全栈项目从需求分析到部署上线的全流程自动化。

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

### 3. 进入项目目录并启动

```bash
cd /path/to/skills
opencode
```

## 系统架构

### 开发阶段

| 阶段 | 说明 | 主智能体提示词 |
|------|------|---------------|
| PRD 设计 | 产品需求文档设计 | `prd/main_agent_prompt_prd.md` |
| 架构设计 | 技术架构设计 | `architecture/main_agent_prompt_fs_architect.md` |
| 前端开发 | Vue/React 前端开发 | `frontend/main_agent_prompt_vue.md` |
| 后端开发 | API 接口开发 | `backend/main_agent_prompt.md` |
| Flutter 开发 | 跨端应用开发 | `flutter/main_agent_prompt_flutter.md` |
| 区块链开发 | 智能合约开发 | `blockchain/main_agent_prompt_blockchain.md` |
| 前后端联调 | 接口对接与集成 | `fullstack/main_agent_prompt_fullstack.md` |
| 部署上线 | 生产环境部署 | `deploy/main_agent_prompt_deploy.md` |

### 工作原理

主智能体通过 `general` 内置代理并行调度多个任务，每个任务会读取对应的专业提示词文件：

```markdown
Task(subagent_type: "general", prompt: "请先阅读文件：{file:./frontend/agents/dg_vue_planner.md}，然后执行任务...")
```

## 目录结构

```
skills/
├── prd/                          # PRD 设计模块
│   ├── main_agent_prompt_prd.md  # 主智能体提示词
│   └── agents/                   # 专业角色提示词
│       ├── prd_business.md
│       ├── prd_user.md
│       ├── prd_functional.md
│       └── prd_technical.md
├── architecture/                 # 架构设计模块
│   ├── main_agent_prompt_fs_architect.md
│   └── agents/
│       ├── fa_techstack.md
│       ├── fa_data.md
│       ├── fa_infra.md
│       ├── fa_security.md
│       ├── fa_api_design.md
│       └── fa_uiux.md
├── frontend/                     # 前端开发模块
│   ├── main_agent_prompt_vue.md
│   └── agents/
│       ├── dg_vue_planner.md
│       ├── dg_frontend_vue_dev.md
│       ├── dg_vue_tester_component.md
│       ├── dg_vue_tester_logic.md
│       └── dg_vue_tester_style.md
├── backend/                      # 后端开发模块
│   ├── main_agent_prompt.md
│   └── agents/
│       ├── be_planner.md
│       ├── be_api_dev.md
│       ├── be_tester_functional.md
│       ├── be_tester_performance.md
│       └── be_tester_security.md
├── flutter/                      # Flutter 开发模块
├── blockchain/                   # 区块链开发模块
├── fullstack/                    # 前后端联调模块
├── deploy/                       # 部署上线模块
└── common/                       # 公共文档
    ├── core-principles.md        # 核心原则
    ├── parallel-optimization.md  # 并行度优化
    ├── timeout-recovery.md       # 超时恢复
    ├── session-management.md     # 会话管理
    └── context-management.md     # 上下文管理
```

## 使用方式

### 使用主智能体

直接告诉主智能体您的需求，它会自动协调子任务：

```
请帮我设计一个电商系统的架构
```

### 使用专业角色

通过 `@` 提及调用专业角色：

```
@fa-techstack 请分析这个项目的技术栈选型
```

## 核心原则

1. **主Agent只调度不干活** - 不做开发、不做测试、不做代码审查
2. **自主决策优先** - 缺失信息时使用默认值填充，不阻塞流程
3. **并行执行** - 多个任务同时启动，提高效率
4. **自动恢复** - 超时自动重试，失败自动降级

## 相关文档

- [核心原则](./common/core-principles.md)
- [并行度优化](./common/parallel-optimization.md)
- [超时恢复](./common/timeout-recovery.md)
- [会话管理](./common/session-management.md)
- [上下文管理](./common/context-management.md)
