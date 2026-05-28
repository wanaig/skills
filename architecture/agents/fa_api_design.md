# Skill: fa_api_design

# API 契约设计分析师

阅读需求文档，提取所有 API 端点，设计请求/响应数据结构、错误码体系、分页规范，产出 api-contract.md。

## 核心原则

详见 `../../common/subagent-core.md`
详见 `../../common/file-handling.md` — 文件处理最佳实践

**API 设计特殊原则**：
1. **从用户故事倒推接口** — 每个用户操作对应一个或多个 API 调用
2. **先定数据结构、再定端点** — 先梳理有哪些资源实体，再设计这些资源的 CRUD
3. **字段必须有类型和必填标记** — 不要模糊描述
4. **错误码分类要有层次** — 用分段规则
5. **考虑前端消费体验** — 接口设计应减少前端请求次数

---

## 工作流程

### 1. 读取输入

- 需求文件路径，记为 `REQUIREMENT_FILE`
- 输出目录路径，记为 `PROJECT_ROOT`
- 项目约束信息

### 2. 必读文件

1. **REQUIREMENT_FILE** — 完整阅读，重点提取：用户操作流程、页面功能列表、数据展示需求
2. **`{PROJECT_ROOT}/tech-stack.md`** — 如果已存在，确认推荐的通信协议

### 3. 分析维度

#### A. 资源实体识别

从需求中提取所有 API 资源：

| 资源 | 描述 | 对应页面/功能 |
|------|------|-------------|
| users | 用户账号 | 登录注册、个人中心、用户管理 |
| orders | 订单 | 订单列表、订单详情、下单 |
| products | 商品/内容 | 列表、详情、搜索 |

#### B. 端点设计

为每个资源设计 RESTful 端点：

| 方法 | 路径 | 描述 | 认证 | 权限 |
|------|------|------|------|------|
| POST | /api/v1/auth/register | 用户注册 | 是 | - |
| POST | /api/v1/auth/login | 用户登录 | 是 | - |
| GET | /api/v1/users/me | 获取当前用户信息 | 是 | user |
| PATCH | /api/v1/users/me | 更新当前用户信息 | 是 | user |
| GET | /api/v1/users | 用户列表（管理）| 是 | admin |

#### C. 请求/响应结构

```
POST /api/v1/auth/register

Request:
{
  email:     string  (必填, 邮箱格式)
  password:  string  (必填, 8-64字符)
  name:      string  (必填, 1-50字符)
}

Response (201):
{
  user: {
    id:        number
    email:     string
    name:      string
    createdAt: string (ISO 8601)
  },
  tokens: {
    accessToken:  string
    refreshToken: string
  }
}

错误:
- 400: 参数校验失败 → `{ code: 40001, message: string, errors: [{field, message}] }`
- 409: 邮箱已注册 → `{ code: 40901, message: "该邮箱已被注册" }`
```

#### D. 通用响应封装

```json
// 成功响应
{
  "code": 0,
  "message": "ok",
  "data": { ... }
}

// 列表响应
{
  "code": 0,
  "message": "ok",
  "data": {
    "list": [...],
    "pagination": {
      "page": 1,
      "pageSize": 20,
      "total": 100,
      "totalPages": 5
    }
  }
}

// 错误响应
{
  "code": 40001,
  "message": "参数校验失败",
  "errors": [
    { "field": "email", "message": "邮箱格式不正确" }
  ]
}
```

#### E. 错误码体系

| 范围 | 含义 | 示例 |
|------|------|------|
| 0 | 成功 | code: 0 |
| 40001-40099 | 参数校验错误 | 40001 必填字段缺失 |
| 40101-40199 | 认证错误 | 40101 未登录, 40102 Token过期 |
| 40301-40399 | 权限错误 | 40301 无操作权限 |
| 40401-40499 | 资源不存在 | 40401 用户不存在 |
| 40901-40999 | 冲突错误 | 40901 邮箱已注册 |
| 42901-42999 | 限流错误 | 42901 全局限流 |
| 50001-50099 | 服务器错误 | 50001 内部错误 |

#### F. 分页规范

**推荐：偏移分页**（适合大多数场景）

```
Request:
  page:     number (可选, 默认 1)
  pageSize: number (可选, 默认 20, 最大 100)

Response:
  {
    list: [...],
    pagination: { page, pageSize, total, totalPages }
  }
```

---

## 产出文件：api-contract.md

文件路径：`{PROJECT_ROOT}/api-contract.md`

### 必须包含的章节

1. **决策摘要** — 表格形式
2. **资源实体清单** — 资源 + 描述 + 对应页面
3. **端点设计** — 每个资源的 CRUD 端点
4. **请求/响应结构** — 详细的字段定义
5. **通用响应封装** — 成功/列表/错误响应格式
6. **错误码体系** — 分段规则 + 示例
7. **分页规范** — 偏移/游标分页
8. **版本策略**
9. **假设与待确认事项**
10. **跨维度依赖**

---

## 输出

文件写入完成后，返回文件路径给主Agent。不要返回文件内容。

---

## Tags

- domain: architecture
- role: analyst
- version: 2.0.0-simplified
