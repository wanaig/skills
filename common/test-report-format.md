# 测试报告格式规范

## 报告格式

测试报告同时输出 markdown 和 JSON 格式。

## JSON 报告结构

### PASS 时

```json
{
  "module": "{模块名}",
  "dimension": "{测试维度}",
  "round": {轮次},
  "verdict": "PASS",
  "failures": [],
  "max_severity": null
}
```

### FAIL 时

```json
{
  "module": "{模块名}",
  "dimension": "{测试维度}",
  "round": {轮次},
  "verdict": "FAIL",
  "max_severity": "blocker|major|minor",
  "failures": [
    {
      "severity": "blocker|major|minor",
      "category": "{维度类别}",
      "file": "{文件路径}",
      "line": {行号},
      "reason": "{问题描述}",
      "suggestion": "{修改建议}"
    }
  ]
}
```

## 字段说明

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| module | string | 是 | 模块名称 |
| dimension | string | 是 | 测试维度（component/logic/style/functional/performance/security/gas/contract/dataflow/integration） |
| round | number | 是 | 测试轮次（从 1 开始） |
| verdict | string | 是 | PASS 或 FAIL |
| max_severity | string | FAIL时必填 | 最高严重级别（blocker/major/minor） |
| failures | array | 是 | 问题列表 |

### failures 数组字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| severity | string | 是 | blocker/major/minor |
| category | string | 是 | 问题所属维度类别 |
| file | string | 是 | 问题文件路径 |
| line | number | 是 | 问题所在行号 |
| reason | string | 是 | 问题原因描述 |
| suggestion | string | 是 | 修改建议 |

## 判定规则

- **PASS**：无 blocker/major 级别失败
- **FAIL**：存在 blocker 或 major 级别失败

## Severity 分级

| 级别 | 定义 | 处理策略 |
|------|------|---------|
| blocker | 功能不可用/安全漏洞 | 必须修复，不允许降级 |
| major | 核心功能缺陷/性能不达标 | 必须修复，允许降级 |
| minor | 可接受的优化项 | 记录技术债务，不阻塞 |

## 报告命名规则

`{模块名}-{维度}-report.json`

## 报告存放路径

`{PROJECT_ROOT}/outputs/{tester_type}/`
