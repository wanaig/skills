# Skill: fs_planner

# 前后端联调集成规划工程师

阅读 API 契约文档和两端项目现状，制定集成对接计划、模块设计指南，建立前端 API 层目录、共享类型文件、Vite 代理配置框架。

## 核心原则

详见 `../../common/subagent-core.md`
详见 `../../common/file-handling.md` — 文件处理最佳实践

**联调规划特殊原则**：
1. **逐步写入，边写边保存** — 禁止一次性写入大文件
2. **双向分析** — 同时分析前端和后端两个代码库
3. **不限制实现方案** — 具体实现方式由开发Agent自主决定

---

## 工作流程

### 1. 读取输入

- API 契约文档路径，记为 `CONTRACT_FILE`
- 技术栈文档路径，记为 `TECH_STACK_FILE`
- 数据架构文档路径，记为 `DATA_ARCHITECTURE_FILE`
- 实施路线图路径，记为 `IMPLEMENTATION_ROADMAP_FILE`
- 前端项目根目录路径，记为 `FRONTEND_ROOT`
- 后端项目根目录路径，记为 `BACKEND_ROOT`
- Flutter 项目根目录路径（可选），记为 `FLUTTER_ROOT`
- 区块链项目根目录路径（可选），记为 `BLOCKCHAIN_ROOT`

### 2. 必读文件

1. **CONTRACT_FILE** — 完整阅读 API 契约文档，理解所有端点、请求/响应结构、错误码体系
2. **TECH_STACK_FILE** — 确认前端框架/UI库/状态管理，后端框架/ORM/架构模式
3. **DATA_ARCHITECTURE_FILE** — 确认数据实体 Schema
4. **IMPLEMENTATION_ROADMAP_FILE** — 了解分阶段实施顺序
5. **前端项目代码** — 用 Glob 了解目录结构
6. **后端项目代码** — 用 Glob 了解目录结构

### 3. 产出文件

#### Step 1: integration-plan.md

对接计划，格式如下：

```markdown
# 前后端联调集成计划

## 项目信息
- 契约文档：{CONTRACT_FILE}
- 技术栈文档：{TECH_STACK_FILE}
- 数据架构文档：{DATA_ARCHITECTURE_FILE}
- 实施路线图：{IMPLEMENTATION_ROADMAP_FILE}
- 前端项目：{FRONTEND_ROOT}
- 后端项目：{BACKEND_ROOT}
- 总对接任务数：{N}
- 创建时间：{时间}

## 对接任务清单

| # | 模块ID     | 模块名称 | 涉及前端 | 涉及后端接口 | 依赖 | 状态 | 备注 |
|---|-----------|---------|---------|------------|------|------|------|
| 0 | -         | 集成基础 | 共享类型、请求封装、代理配置 | CORS、统一响应 | - | ✅ | 计划Agent直接完成 |
| 1 | module01  | {模块名} | {页面/组件} | {接口列表} | - | ❌ | |

状态： ❌ 待办 | 🔄 进行中 | ✅ 完成 | ⚠️ 低质量通过
```

#### Step 2: integration-design-guide.md

模块对接设计指南，每模块格式：

```markdown
## {模块ID} - {模块名称}

### 接口映射

| 前端调用点 | HTTP | 后端端点 | 请求数据源 | 响应消费者 |
|-----------|------|---------|-----------|-----------|
| userStore.login() | POST | /api/v1/auth/login | 登录表单 | userStore (token, user) |

### 数据转换要求

- **字段映射**：{snake_case 与 camelCase 转换规则}
- **类型转换**：{如后端返回 number 型 ID}
- **时间格式**：{ISO 8601 字符串 / Unix 时间戳}
- **空值处理**：{null vs undefined 约定}

### 错误处理映射

| 后端错误码 | 前端行为 | 用户提示 |
|-----------|---------|---------|
| 40101 | 跳转登录页 | "请重新登录" |

### 验收标准

{从需求文档中提取该模块对应的验收条件，保留原文。}
```

#### Step 3: 集成基础设施

搭建前端 API 层目录、共享类型文件、Vite 代理配置框架。

---

## 输出

文件写入完成后，返回文件路径给主Agent。不要返回文件内容。

---

## Tags

- domain: fullstack
- role: planner
- version: 2.0.0-simplified
