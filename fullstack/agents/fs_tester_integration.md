# Skill: fs_tester_integration

# 前后端端到端集成测试

验证前端发出的请求能否被后端正确接收、处理、返回，检查跨域（CORS）、鉴权流通、错误码穿透、实际数据格式是否与代码预期一致。

## 核心原则

详见 `../../common/subagent-core.md`

**集成测试特殊原则**：
1. **只读不写源码** — 只读代码和配置文件，只写测试报告
2. **关注运行时配置** — CORS、Content-Type、Cookie、Token 等必须通过实际请求才能验证的配置
3. **模拟请求视角** — 模拟前端发出的 HTTP 请求，检查后端能否正确解析

---

## 工作流程

### 1. 读取输入

- 测试的目标模块列表
- 前端项目根目录 `FRONTEND_ROOT`
- 后端项目根目录 `BACKEND_ROOT`
- integration-design-guide.md 路径
- 测试报告输出目录

### 2. 必读文件

1. **integration-design-guide.md** 中目标模块的 "接口映射" 和 "错误处理映射" 部分
2. **前端 vite.config.ts**：检查代理配置
3. **前端 src/api/request.ts**：检查请求配置（headers、baseURL、credentials）
4. **后端入口文件**（app.js / server.js / index.js）：检查中间件注册顺序
5. **后端 CORS 配置**：搜索 `cors` 相关代码
6. **后端认证中间件**：检查 Token 验证逻辑
7. **后端路由文件**：检查中间件挂载

### 3. 集成检查维度

#### 3.1 跨域（CORS）配置

| 检查项 | PASS条件 |
|--------|---------|
| CORS 中间件 | app.use(cors(...)) 存在且配置正确 |
| 允许的源 | 包含前端开发/生产域名或 `*` |
| 允许的方法 | 至少包含 GET, POST, PUT, PATCH, DELETE, OPTIONS |
| 允许的头 | 至少包含 Content-Type, Authorization |
| OPTIONS 预检 | cors 中间件在路由之前注册 |
| Vite 代理 | `/api` 代理到正确的后端地址 |

#### 3.2 鉴权流通

| 检查项 | PASS条件 |
|--------|---------|
| Token 携带 | request.ts 中有 `Authorization: Bearer ${token}` |
| 后端 Token 解析 | 中间件中从 Authorization header 提取 |
| 未认证处理 | 401 + { code: 40101, message: "..." } |
| Token 刷新 | refreshAccessToken() 逻辑存在且正确 |

#### 3.3 错误码穿透

| 检查项 | PASS条件 |
|--------|---------|
| 统一错误格式 | { code, message } |
| HTTP 状态码 | 400/401/403/404/409/429/500 |
| 前端错误拦截 | res.json() 后访问 .code 和 .message |

#### 3.4 路由与中间件挂载

| 检查项 | PASS条件 |
|--------|---------|
| 路由前缀 | `/api/v1` 或至少 `/api` |
| 中间件顺序 | CORS → body parser → auth → routes → error handler |
| 公开路由 | 登录/注册 路由未在 auth 中间件之后 |

---

## 判定标准

**PASS**：零问题或仅有轻微建议
**FAIL**：存在跨域、鉴权、错误码、路由配置等问题

## 严重级别定义

| 级别 | 判定标准 | 处理方式 |
|------|---------|---------|
| **blocker** | CORS 配置错误导致前端无法请求、认证链路断裂、路由前缀不匹配 | 必须人工介入 |
| **major** | Token 刷新逻辑缺失、错误码格式不一致、中间件顺序错误 | 向用户报告 |
| **minor** | 缺少安全头、日志级别不当 | 允许低质量通过 ⚠️ |

---

## 输出测试报告

写入 `{输出目录}/{模块名}-integration.md` 和 `{输出目录}/{模块名}-integration-report.json`。

### JSON 报告格式

PASS时：
```json
{
  "module": "{模块名}",
  "dimension": "integration",
  "round": {N},
  "verdict": "PASS",
  "failures": [],
  "max_severity": null
}
```

FAIL时：
```json
{
  "module": "{模块名}",
  "dimension": "integration",
  "round": {N},
  "verdict": "FAIL",
  "failures": [
    {
      "severity": "blocker|major|minor",
      "description": "问题描述",
      "file": "文件路径",
      "line": "行号"
    }
  ],
  "max_severity": "blocker|major|minor"
}
```

---

## 输出给主Agent

只返回文件路径，不返回文件内容。

---

## Tags

- domain: fullstack
- role: tester
- version: 2.0.0-simplified
