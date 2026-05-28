# Skill: fs_tester_dataflow

# 前后端数据流完整性测试

验证从前端发起请求到后端处理再返回前端渲染的完整数据链路，检查数据转换、状态流转、错误传播、加载态处理是否正确。

## 核心原则

详见 `../../common/subagent-core.md`

**数据流测试特殊原则**：
1. **只读不写源码** — 只读代码文件，只写测试报告
2. **追踪完整链路** — 从 UI 事件到 API 响应再到 DOM 更新，一个环节不能少
3. **关注状态转换** — loading → success/error → data 的每个状态都要覆盖

---

## 工作流程

### 1. 读取输入

- 测试的目标模块列表
- 前端项目根目录 `FRONTEND_ROOT`
- 后端项目根目录 `BACKEND_ROOT`
- integration-design-guide.md 路径
- 测试报告输出目录

### 2. 必读文件

1. **integration-design-guide.md** 中目标模块的 "接口映射""数据转换要求""错误处理映射" 部分
2. **前端 Store 文件** — 了解状态管理逻辑
3. **前端 API 调用文件** — 了解 API 调用方式
4. **前端页面/组件文件** — 了解 UI 如何使用 store 数据
5. **后端控制器/服务文件** — 了解数据如何被处理和返回

### 3. 数据流检查维度

#### 3.1 请求发起链路

从用户操作追踪到 API 调用：

```
用户操作 (click/submit/mounted)
  → 组件方法调用
    → Store action 调用
      → API 函数调用
        → request() 发出 HTTP 请求
```

| 检查项 | PASS条件 |
|--------|---------|
| 触发点 | 组件中能追踪到明确的调用链 |
| 参数组装 | 参数来源可追溯 |
| 调用前状态 | loading.value = true |

#### 3.2 响应处理链路

从 HTTP 响应追踪到 UI 更新：

```
HTTP 响应
  → request.ts 响应拦截器
    → API 函数返回
      → Store action 处理响应
        → 更新 store 状态
          → 组件模板响应式更新
```

| 检查项 | PASS条件 |
|--------|---------|
| 成功处理 | data.value = res.data.xxx |
| loading 重置 | finally 中有 loading.value = false |
| 空数据处理 | 模板中有空态展示 |

#### 3.3 错误处理链路

| 检查项 | PASS条件 |
|--------|---------|
| try-catch | 有明确的 catch 块 |
| 错误状态 | error.value = err.message |
| Token 过期 | request.ts 拦截器中有 401 处理 |

#### 3.4 数据转换链路

| 检查项 | PASS条件 |
|--------|---------|
| snake_case→camelCase | request.ts 拦截器或 store 中明确处理 |
| 时间格式化 | 组件中用 `new Date()` 或日期库格式化 |
| 空值统一 | 使用了 `??` 或 `\|\|` 兜底 |

---

## 判定标准

**PASS**：零问题或仅有轻微建议
**FAIL**：存在数据流断裂、状态转换错误、错误处理缺失等问题

## 严重级别定义

| 级别 | 判定标准 | 处理方式 |
|------|---------|---------|
| **blocker** | API调用无错误处理导致静默失败、loading状态未重置导致UI卡住、Token过期无处理 | 必须人工介入 |
| **major** | 三态未完整覆盖、数据转换缺失导致字段不一致、错误信息未展示 | 向用户报告 |
| **minor** | 空值处理不一致、非关键路径的状态管理优化 | 允许低质量通过 ⚠️ |

---

## 输出测试报告

写入 `{输出目录}/{模块名}-dataflow.md` 和 `{输出目录}/{模块名}-dataflow-report.json`。

### JSON 报告格式

PASS时：
```json
{
  "module": "{模块名}",
  "dimension": "dataflow",
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
  "dimension": "dataflow",
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
