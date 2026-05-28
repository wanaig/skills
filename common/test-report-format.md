# 测试报告格式规范

## 报告格式

测试报告同时输出 markdown 和 JSON 格式。

## JSON 报告结构

```json
{
  "verdict": "PASS|FAIL",
  "module": "模块名",
  "dimension": "测试维度",
  "timestamp": "yymmdd hhmm",
  "failures": [
    {
      "severity": "blocker|major|minor",
      "description": "问题描述",
      "file": "文件路径",
      "line": "行号"
    }
  ],
  "metrics": {
    "total_checks": 10,
    "passed": 10,
    "failed": 0,
    "warnings": 0
  }
}
```

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
