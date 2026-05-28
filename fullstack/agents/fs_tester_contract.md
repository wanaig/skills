# Skill: fs_tester_contract

# 前后端 API 契约一致性测试

验证前端的 API 类型定义、请求参数、响应结构与后端接口契约一致，不验运行时行为，只验代码层面的约定匹配。

## 核心原则

详见 `../../common/subagent-core.md`

**契约测试特殊原则**：
1. **只读不写源码** — 只读代码文件，只写测试报告
2. **以契约文档为基准** — API 契约文档是唯一的真相来源
3. **字段级精确对比** — 比对到每一个字段的类型、必填/可选、命名

---

## 工作流程

### 1. 读取输入

- 测试的目标模块列表
- 前端项目根目录 `FRONTEND_ROOT`
- 后端项目根目录 `BACKEND_ROOT`
- API 契约文档路径
- integration-design-guide.md 路径
- 测试报告输出目录

### 2. 必读文件

1. **integration-design-guide.md** 中目标模块的 "接口映射" 和 "数据转换要求" 部分
2. **API 契约文档** — 确认端点定义、请求/响应字段、错误码
3. **前端 API 类型文件** — 了解前端类型定义
4. **前端 API 调用文件** — 了解前端调用方式
5. **后端接口代码** — 了解后端实现

### 3. 契约对比维度

#### 3.1 端点路径一致性

| 检查项 | PASS条件 |
|--------|---------|
| URL 路径 | 路径完全一致 |
| HTTP 方法 | 方法一致 |
| baseURL | `/api/v1` 统一 |

#### 3.2 请求参数一致性

| 检查项 | PASS条件 |
|--------|---------|
| Body 字段 | 字段名、类型、必填/可选一致 |
| Query 参数 | 参数名、默认值一致 |
| Path 参数 | 参数命名一致 |

#### 3.3 响应结构一致性

| 检查项 | PASS条件 |
|--------|---------|
| 响应泛型 | 字段名、类型、嵌套结构一一对应 |
| 分页结构 | {page, pageSize, total, totalPages} |
| 错误响应 | code 为 number, message 为 string |

#### 3.4 类型定义一致性

| 检查项 | PASS条件 |
|--------|---------|
| 字段命名 | camelCase vs snake_case 有转换逻辑 |
| 字段类型 | string/string, number/number, boolean/boolean |
| 时间字段格式 | 统一为 string (ISO 8601) 或 number (时间戳) |

---

## 判定标准

**PASS**：零问题或仅有轻微建议
**FAIL**：存在端点不匹配、字段类型不一致、命名风格不统一等问题

## 严重级别定义

| 级别 | 判定标准 | 处理方式 |
|------|---------|---------|
| **blocker** | 端点路径完全不匹配、请求/响应字段缺失、类型定义错误 | 必须人工介入 |
| **major** | 字段命名不一致（camelCase vs snake_case）、可选/必填标记不一致、枚举值不匹配 | 向用户报告 |
| **minor** | 时间格式不一致、分页字段命名差异 | 允许低质量通过 ⚠️ |

---

## 输出测试报告

写入 `{输出目录}/{模块名}-contract.md` 和 `{输出目录}/{模块名}-contract-report.json`。

### JSON 报告格式

PASS时：
```json
{
  "module": "{模块名}",
  "dimension": "contract",
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
  "dimension": "contract",
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
