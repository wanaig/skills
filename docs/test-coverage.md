# 测试覆盖率保障

本文档定义了系统的测试覆盖率保障机制，包括覆盖率要求、检查机制、报告生成和质量保障。

## 1. 覆盖率要求

### 1.1 覆盖率目标

```json
{
  "coverageRequirements": {
    "unit": {
      "minimum": 80,
      "target": 90,
      "critical": 95,
      "description": "单元测试覆盖率"
    },
    "integration": {
      "minimum": 60,
      "target": 70,
      "critical": 80,
      "description": "集成测试覆盖率"
    },
    "e2e": {
      "minimum": 100,
      "target": 100,
      "critical": 100,
      "scope": "core_flows",
      "description": "端到端测试覆盖率（核心流程）"
    }
  }
}
```

### 1.2 覆盖率指标

```json
{
  "coverageMetrics": {
    "lineCoverage": {
      "description": "行覆盖率",
      "formula": "执行的代码行数 / 总代码行数",
      "weight": 0.4
    },
    "branchCoverage": {
      "description": "分支覆盖率",
      "formula": "执行的分支数 / 总分支数",
      "weight": 0.3
    },
    "functionCoverage": {
      "description": "函数覆盖率",
      "formula": "调用的函数数 / 总函数数",
      "weight": 0.2
    },
    "statementCoverage": {
      "description": "语句覆盖率",
      "formula": "执行的语句数 / 总语句数",
      "weight": 0.1
    }
  }
}
```

### 1.3 覆盖率排除规则

```json
{
  "coverageExclusions": {
    "files": [
      "node_modules/**",
      "dist/**",
      "build/**",
      "**/*.test.*",
      "**/*.spec.*",
      "**/test/**",
      "**/tests/**",
      "**/__tests__/**"
    ],
    "patterns": [
      "console.log",
      "console.error",
      "console.warn",
      "debugger",
      "// istanbul ignore next",
      "// istanbul ignore file"
    ],
    "functions": [
      "constructor",
      "getters",
      "setters",
      "toString",
      "valueOf"
    ]
  }
}
```

---

## 2. 覆盖率检查机制

### 2.1 检查时机

```json
{
  "coverageCheckTiming": {
    "preCommit": {
      "enabled": true,
      "description": "提交前检查",
      "actions": ["validate", "warn", "block"]
    },
    "prePush": {
      "enabled": true,
      "description": "推送前检查",
      "actions": ["validate", "warn", "block"]
    },
    "onPR": {
      "enabled": true,
      "description": "PR创建时检查",
      "actions": ["validate", "comment", "block"]
    },
    "onMerge": {
      "enabled": true,
      "description": "合并前检查",
      "actions": ["validate", "block"]
    },
    "nightly": {
      "enabled": true,
      "description": "每晚检查",
      "actions": ["validate", "report"]
    }
  }
}
```

### 2.2 检查规则

```json
{
  "coverageCheckRules": {
    "overall": {
      "minimum": 80,
      "target": 90,
      "blockOnFailure": true,
      "warnOnBelowTarget": true
    },
    "newCode": {
      "minimum": 90,
      "target": 95,
      "blockOnFailure": true,
      "description": "新代码覆盖率要求更高"
    },
    "criticalPaths": {
      "minimum": 95,
      "target": 100,
      "blockOnFailure": true,
      "description": "关键路径覆盖率要求最高"
    },
    "regression": {
      "enabled": true,
      "threshold": 5,
      "blockOnRegression": true,
      "description": "覆盖率不能下降超过5%"
    }
  }
}
```

### 2.3 检查流程

```markdown
### 覆盖率检查流程

1. **收集覆盖率数据**
   - 运行测试套件
   - 生成覆盖率报告
   - 解析覆盖率数据

2. **验证覆盖率要求**
   - 检查整体覆盖率
   - 检查新代码覆盖率
   - 检查关键路径覆盖率
   - 检查覆盖率回归

3. **生成检查报告**
   - 覆盖率详情
   - 未覆盖区域
   - 改进建议

4. **执行检查动作**
   - 通过：继续执行
   - 失败：阻断或警告
   - 回归：阻断并通知
```

---

## 3. 覆盖率报告生成

### 3.1 报告格式

```json
{
  "coverageReportFormats": {
    "html": {
      "enabled": true,
      "description": "HTML报告，可视化",
      "output": "coverage/index.html"
    },
    "json": {
      "enabled": true,
      "description": "JSON报告，程序解析",
      "output": "coverage/coverage-summary.json"
    },
    "lcov": {
      "enabled": true,
      "description": "LCOV报告，工具集成",
      "output": "coverage/lcov.info"
    },
    "text": {
      "enabled": true,
      "description": "文本报告，控制台输出",
      "output": "stdout"
    },
    "clover": {
      "enabled": false,
      "description": "Clover报告",
      "output": "coverage/clover.xml"
    },
    "cobertura": {
      "enabled": false,
      "description": "Cobertura报告",
      "output": "coverage/cobertura.xml"
    }
  }
}
```

### 3.2 报告内容

```json
{
  "coverageReportContent": {
    "summary": {
      "totalLines": 1000,
      "coveredLines": 850,
      "totalBranches": 200,
      "coveredBranches": 170,
      "totalFunctions": 100,
      "coveredFunctions": 90,
      "lineCoverage": 85,
      "branchCoverage": 85,
      "functionCoverage": 90
    },
    "byFile": [
      {
        "path": "src/components/UserList.vue",
        "lineCoverage": 90,
        "branchCoverage": 85,
        "functionCoverage": 95,
        "uncoveredLines": [45, 67, 89]
      }
    ],
    "byDirectory": [
      {
        "path": "src/components",
        "lineCoverage": 88,
        "branchCoverage": 82,
        "functionCoverage": 92
      }
    ],
    "uncoveredAreas": [
      {
        "file": "src/utils/helper.ts",
        "lines": [23, 24, 25, 45, 46],
        "reason": "边界条件未测试"
      }
    ]
  }
}
```

### 3.3 报告存储

```json
{
  "coverageReportStorage": {
    "local": {
      "enabled": true,
      "path": "{PROJECT_ROOT}/coverage",
      "retention": "7days"
    },
    "artifact": {
      "enabled": true,
      "path": "{PROJECT_ROOT}/outputs/coverage",
      "retention": "30days"
    },
    "remote": {
      "enabled": false,
      "provider": "codecov",
      "token": null
    }
  }
}
```

---

## 4. 测试质量检查

### 4.1 测试有效性检查

```json
{
  "testQualityChecks": {
    "testEffectiveness": {
      "enabled": true,
      "checks": [
        {
          "name": "assertion_count",
          "description": "断言数量检查",
          "minimum": 1,
          "warning": "测试用例至少有一个断言"
        },
        {
          "name": "test_isolation",
          "description": "测试隔离性检查",
          "checks": ["no_shared_state", "no_external_dependencies"],
          "warning": "测试应该独立运行"
        },
        {
          "name": "test_naming",
          "description": "测试命名规范",
          "pattern": "should.*when.*",
          "warning": "测试命名应该清晰描述行为"
        }
      ]
    },
    "avoidIneffectiveTests": {
      "enabled": true,
      "patterns": [
        {
          "name": "always_pass",
          "description": "总是通过的测试",
          "detection": "no_assertions",
          "action": "warn"
        },
        {
          "name": "test_nothing",
          "description": "没有实际测试的测试",
          "detection": "assert_true_true",
          "action": "warn"
        },
        {
          "name": "duplicate_test",
          "description": "重复的测试",
          "detection": "same_assertions",
          "action": "warn"
        }
      ]
    }
  }
}
```

### 4.2 边界条件覆盖

```json
{
  "boundaryConditionCoverage": {
    "enabled": true,
    "checks": [
      {
        "name": "null_values",
        "description": "空值处理",
        "checks": ["null", "undefined", "empty_string", "empty_array"]
      },
      {
        "name": "edge_values",
        "description": "边界值",
        "checks": ["min_value", "max_value", "zero", "negative"]
      },
      {
        "name": "type_coercion",
        "description": "类型转换",
        "checks": ["string_to_number", "number_to_string", "boolean_conversion"]
      },
      {
        "name": "error_conditions",
        "description": "错误条件",
        "checks": ["throw_error", "reject_promise", "return_error"]
      }
    ]
  }
}
```

### 4.3 Mock使用检查

```json
{
  "mockUsageChecks": {
    "enabled": true,
    "guidelines": [
      {
        "name": "appropriate_mocking",
        "description": "适当的Mock使用",
        "rules": [
          "mock_external_dependencies",
          "mock_side_effects",
          "dont_mock_implementation_details"
        ]
      },
      {
        "name": "mock_verification",
        "description": "Mock验证",
        "rules": [
          "verify_mock_calls",
          "verify_mock_arguments",
          "verify_mock_return_values"
        ]
      },
      {
        "name": "mock_cleanup",
        "description": "Mock清理",
        "rules": [
          "reset_mocks_after_test",
          "clear_mock_history",
          "restore_original_implementation"
        ]
      }
    ]
  }
}
```

---

## 5. 覆盖率工具集成

### 5.1 前端覆盖率工具

```json
{
  "frontendCoverageTools": {
    "jest": {
      "enabled": true,
      "config": {
        "collectCoverage": true,
        "coverageDirectory": "coverage",
        "coverageReporters": ["text", "lcov", "json"],
        "coverageThreshold": {
          "global": {
            "branches": 80,
            "functions": 80,
            "lines": 80,
            "statements": 80
          }
        }
      }
    },
    "vitest": {
      "enabled": false,
      "config": {
        "coverage": {
          "provider": "v8",
          "reporter": ["text", "lcov", "json"],
          "thresholds": {
            "branches": 80,
            "functions": 80,
            "lines": 80,
            "statements": 80
          }
        }
      }
    },
    "istanbul": {
      "enabled": true,
      "config": {
        "instrumentation": {
          "include": ["src/**"],
          "exclude": ["**/*.test.*", "**/*.spec.*"]
        }
      }
    }
  }
}
```

### 5.2 后端覆盖率工具

```json
{
  "backendCoverageTools": {
    "java": {
      "jacoco": {
        "enabled": true,
        "config": {
          "includes": ["com/example/**"],
          "excludes": ["com/example/test/**"],
          "reportFormats": ["html", "xml", "csv"]
        }
      }
    },
    "nodejs": {
      "nyc": {
        "enabled": true,
        "config": {
          "include": ["src/**"],
          "exclude": ["**/*.test.*", "**/*.spec.*"],
          "reporter": ["text", "lcov", "json"]
        }
      }
    },
    "python": {
      "coverage": {
        "enabled": true,
        "config": {
          "source": ["src"],
          "omit": ["**/test_*", "**/*_test.py"],
          "report": ["html", "xml", "json"]
        }
      }
    }
  }
}
```

### 5.3 覆盖率平台集成

```json
{
  "coveragePlatformIntegration": {
    "codecov": {
      "enabled": false,
      "token": null,
      "config": {
        "fail_ci_if_error": true,
        "flags": "unittests",
        "path_to_write_report": "./coverage/codecov"
      }
    },
    "coveralls": {
      "enabled": false,
      "token": null,
      "config": {
        "service_name": "github",
        "fail_ci_if_error": true
      }
    },
    "sonarqube": {
      "enabled": false,
      "config": {
        "host": "http://localhost:9000",
        "token": null,
        "project_key": "project-key"
      }
    }
  }
}
```

---

## 6. 覆盖率改进建议

### 6.1 未覆盖区域分析

```json
{
  "uncoveredAreaAnalysis": {
    "enabled": true,
    "analysis": {
      "criticalPaths": {
        "description": "关键路径未覆盖",
        "priority": "high",
        "action": "立即添加测试"
      },
      "complexLogic": {
        "description": "复杂逻辑未覆盖",
        "priority": "high",
        "action": "添加边界条件测试"
      },
      "errorHandling": {
        "description": "错误处理未覆盖",
        "priority": "medium",
        "action": "添加错误场景测试"
      },
      "edgeCases": {
        "description": "边界情况未覆盖",
        "priority": "medium",
        "action": "添加边界值测试"
      },
      "utilityFunctions": {
        "description": "工具函数未覆盖",
        "priority": "low",
        "action": "添加单元测试"
      }
    }
  }
}
```

### 6.2 改进建议生成

```json
{
  "improvementSuggestions": {
    "enabled": true,
    "suggestions": [
      {
        "type": "add_test",
        "description": "为未覆盖的函数添加测试",
        "priority": "high",
        "template": "describe('{functionName}', () => { it('should {behavior}', () => { ... }); });"
      },
      {
        "type": "add_boundary",
        "description": "为边界条件添加测试",
        "priority": "high",
        "template": "it('should handle {boundaryCondition}', () => { ... });"
      },
      {
        "type": "add_error",
        "description": "为错误场景添加测试",
        "priority": "medium",
        "template": "it('should throw error when {condition}', () => { ... });"
      },
      {
        "type": "refactor",
        "description": "重构复杂函数以提高可测试性",
        "priority": "medium",
        "template": "将 {functionName} 拆分为更小的函数"
      }
    ]
  }
}
```

---

## 7. 集成到主代理

### 7.1 主代理覆盖率检查流程

```markdown
### 覆盖率检查流程

1. **测试执行**
   - 运行测试套件
   - 收集覆盖率数据
   - 生成覆盖率报告

2. **覆盖率验证**
   - 检查整体覆盖率
   - 检查新代码覆盖率
   - 检查关键路径覆盖率
   - 检查覆盖率回归

3. **质量检查**
   - 检查测试有效性
   - 检查边界条件覆盖
   - 检查Mock使用

4. **报告生成**
   - 生成详细报告
   - 标记未覆盖区域
   - 提供改进建议

5. **结果处理**
   - 通过：继续执行
   - 失败：阻断或警告
   - 回归：阻断并通知
```

### 7.2 主代理提示词更新

在主代理提示词中添加测试覆盖率保障：

```markdown
#### 测试覆盖率保障

**覆盖率要求**：
- 单元测试：≥80%
- 集成测试：≥60%
- 端到端测试：核心流程100%

**检查机制**：
- 提交前检查覆盖率
- 覆盖率不达标阻断提交
- 生成覆盖率报告

**质量检查**：
- 检查测试有效性
- 检查边界条件覆盖
- 检查Mock使用

**改进建议**：
- 分析未覆盖区域
- 生成改进建议
- 跟踪改进效果
```

---

## 8. 最佳实践

### 8.1 覆盖率目标设定

1. **合理目标**：根据项目实际情况设定目标
2. **渐进提升**：逐步提高覆盖率要求
3. **重点关注**：关键路径覆盖率优先
4. **持续监控**：定期检查覆盖率变化
5. **团队共识**：团队对覆盖率目标达成共识

### 8.2 测试编写实践

1. **测试先行**：TDD实践，先写测试再写代码
2. **独立测试**：每个测试独立运行
3. **清晰命名**：测试命名清晰描述行为
4. **边界覆盖**：覆盖所有边界条件
5. **错误场景**：覆盖错误处理场景

### 8.3 覆盖率维护

1. **定期检查**：定期检查覆盖率报告
2. **及时修复**：覆盖率下降及时修复
3. **持续改进**：持续改进测试质量
4. **工具辅助**：使用工具辅助提高覆盖率
5. **团队协作**：团队协作提高覆盖率

### 8.4 覆盖率文化

1. **质量意识**：培养质量意识
2. **责任明确**：明确覆盖率责任
3. **激励机制**：建立激励机制
4. **知识分享**：分享测试最佳实践
5. **持续学习**：持续学习测试技术
