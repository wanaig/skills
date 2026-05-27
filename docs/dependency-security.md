# 依赖安全检查

本文档定义了系统的依赖安全检查机制，包括漏洞扫描、版本管理、许可证检查和依赖审计。

## 1. 漏洞扫描配置

### 1.1 扫描工具配置

```json
{
  "vulnerabilityScanning": {
    "enabled": true,
    "tools": {
      "npm_audit": {
        "enabled": true,
        "config": {
          "audit_level": "moderate",
          "production_only": false,
          "json_output": true
        }
      },
      "snyk": {
        "enabled": false,
        "config": {
          "severity_threshold": "medium",
          "fail_on_issues": true
        }
      },
      "owasp_dependency_check": {
        "enabled": false,
        "config": {
          "suppression_file": "dependency-check-suppressions.xml",
          "fail_on_cvss": 7
        }
      }
    },
    "frequency": {
      "on_install": true,
      "on_update": true,
      "daily": true,
      "weekly": false
    }
  }
}
```

### 1.2 漏洞严重程度

```json
{
  "severityLevels": {
    "critical": {
      "description": "严重漏洞",
      "cvss_range": "9.0-10.0",
      "action": "立即修复",
      "block_deployment": true
    },
    "high": {
      "description": "高危漏洞",
      "cvss_range": "7.0-8.9",
      "action": "尽快修复",
      "block_deployment": true
    },
    "medium": {
      "description": "中危漏洞",
      "cvss_range": "4.0-6.9",
      "action": "计划修复",
      "block_deployment": false
    },
    "low": {
      "description": "低危漏洞",
      "cvss_range": "0.1-3.9",
      "action": "评估风险",
      "block_deployment": false
    }
  }
}
```

### 1.3 扫描流程

```markdown
### 漏洞扫描流程

1. **依赖收集**
   - 读取package.json
   - 读取package-lock.json
   - 收集所有依赖

2. **漏洞扫描**
   - 运行扫描工具
   - 收集漏洞信息
   - 分类漏洞严重程度

3. **风险评估**
   - 评估漏洞影响
   - 确定修复优先级
   - 生成修复建议

4. **报告生成**
   - 生成扫描报告
   - 标记漏洞位置
   - 提供修复方案

5. **执行动作**
   - 告警通知
   - 阻断部署（严重漏洞）
   - 记录历史
```

---

## 2. 版本管理

### 2.1 版本锁定

```json
{
  "versionLocking": {
    "enabled": true,
    "strategies": {
      "exact": {
        "description": "精确版本",
        "format": "1.2.3",
        "use_for": ["production_dependencies"]
      },
      "range": {
        "description": "版本范围",
        "format": "^1.2.3",
        "use_for": ["development_dependencies"]
      },
      "latest": {
        "description": "最新版本",
        "format": "latest",
        "use_for": ["tools"]
      }
    },
    "lock_file": {
      "enabled": true,
      "format": "package-lock.json",
      "commit": true,
      "verify": true
    }
  }
}
```

### 2.2 版本更新策略

```json
{
  "versionUpdateStrategy": {
    "automatic": {
      "enabled": true,
      "rules": [
        {
          "type": "patch",
          "description": "补丁更新",
          "auto_update": true,
          "test_required": true
        },
        {
          "type": "minor",
          "description": "次要更新",
          "auto_update": false,
          "test_required": true
        },
        {
          "type": "major",
          "description": "主要更新",
          "auto_update": false,
          "test_required": true,
          "review_required": true
        }
      ]
    },
    "manual": {
      "enabled": true,
      "workflow": {
        "create_pr": true,
        "run_tests": true,
        "require_review": true,
        "merge_after_approval": true
      }
    }
  }
}
```

### 2.3 版本兼容性检查

```json
{
  "compatibilityChecking": {
    "enabled": true,
    "checks": [
      {
        "name": "node_version",
        "description": "Node.js版本兼容性",
        "min_version": "16.0.0",
        "max_version": "20.x"
      },
      {
        "name": "peer_dependencies",
        "description": "依赖兼容性",
        "check_conflicts": true,
        "check_versions": true
      },
      {
        "name": "engine_compatibility",
        "description": "引擎兼容性",
        "check_node": true,
        "check_npm": true
      }
    ]
  }
}
```

---

## 3. 许可证检查

### 3.1 许可证配置

```json
{
  "licenseChecking": {
    "enabled": true,
    "allowed_licenses": [
      "MIT",
      "Apache-2.0",
      "BSD-2-Clause",
      "BSD-3-Clause",
      "ISC",
      "0BSD"
    ],
    "blocked_licenses": [
      "GPL-3.0",
      "AGPL-3.0",
      "LGPL-3.0"
    ],
    "review_required": [
      "GPL-2.0",
      "LGPL-2.1",
      "MPL-2.0"
    ]
  }
}
```

### 3.2 许可证检查流程

```markdown
### 许可证检查流程

1. **依赖扫描**
   - 收集所有依赖
   - 读取依赖的package.json
   - 提取许可证信息

2. **许可证分类**
   - 分类为允许/阻止/需审查
   - 检查许可证兼容性
   - 识别潜在风险

3. **风险评估**
   - 评估许可证风险
   - 检查许可证冲突
   - 生成风险报告

4. **处理建议**
   - 提供替代方案
   - 建议许可证变更
   - 记录审查结果
```

### 3.3 许可证报告

```json
{
  "licenseReport": {
    "timestamp": "yymmdd hhmm",
    "summary": {
      "total_dependencies": 150,
      "allowed": 140,
      "blocked": 2,
      "review_required": 8
    },
    "licenses": {
      "MIT": 80,
      "Apache-2.0": 30,
      "BSD-3-Clause": 20,
      "ISC": 10
    },
    "issues": [
      {
        "package": "some-package",
        "license": "GPL-3.0",
        "severity": "critical",
        "action": "寻找替代方案"
      }
    ]
  }
}
```

---

## 4. 依赖审计

### 4.1 审计配置

```json
{
  "dependencyAudit": {
    "enabled": true,
    "checks": {
      "unused_dependencies": {
        "enabled": true,
        "action": "warn"
      },
      "outdated_dependencies": {
        "enabled": true,
        "threshold": "30days",
        "action": "warn"
      },
      "duplicate_dependencies": {
        "enabled": true,
        "action": "warn"
      },
      "size_impact": {
        "enabled": true,
        "threshold": "1MB",
        "action": "warn"
      }
    }
  }
}
```

### 4.2 审计报告

```json
{
  "auditReport": {
    "timestamp": "yymmdd hhmm",
    "summary": {
      "total_dependencies": 150,
      "unused": 5,
      "outdated": 10,
      "duplicate": 3,
      "oversized": 2
    },
    "issues": [
      {
        "type": "unused",
        "package": "unused-package",
        "action": "移除"
      },
      {
        "type": "outdated",
        "package": "old-package",
        "current": "1.0.0",
        "latest": "2.0.0",
        "action": "更新"
      }
    ]
  }
}
```

---

## 5. 依赖可视化

### 5.1 依赖树

```json
{
  "dependencyTree": {
    "enabled": true,
    "format": "tree",
    "depth": 3,
    "show_versions": true,
    "show_licenses": false,
    "output": "dependency-tree.txt"
  }
}
```

### 5.2 依赖图

```json
{
  "dependencyGraph": {
    "enabled": true,
    "format": "graphviz",
    "output": "dependency-graph.dot",
    "visualize": {
      "tool": "graphviz",
      "format": "png",
      "output": "dependency-graph.png"
    }
  }
}
```

### 5.3 依赖统计

```json
{
  "dependencyStats": {
    "enabled": true,
    "metrics": {
      "total_packages": 150,
      "direct_dependencies": 30,
      "dev_dependencies": 20,
      "peer_dependencies": 5,
      "transitive_dependencies": 95
    },
    "size_analysis": {
      "total_size": "50MB",
      "node_modules_size": "200MB",
      "largest_packages": [
        {"name": "package-a", "size": "10MB"},
        {"name": "package-b", "size": "8MB"}
      ]
    }
  }
}
```

---

## 6. CI/CD集成

### 6.1 CI/CD配置

```json
{
  "cicdIntegration": {
    "stages": {
      "dependency_audit": {
        "enabled": true,
        "when": "on_push",
        "commands": [
          "npm audit",
          "npm outdated"
        ],
        "fail_on_critical": true
      },
      "license_check": {
        "enabled": true,
        "when": "on_push",
        "commands": [
          "npm run license-check"
        ],
        "fail_on_blocked": true
      },
      "vulnerability_scan": {
        "enabled": true,
        "when": "on_merge",
        "commands": [
          "npm audit --audit-level=moderate"
        ],
        "fail_on_high": true
      }
    }
  }
}
```

### 6.2 质量门禁

```json
{
  "qualityGates": {
    "enabled": true,
    "gates": [
      {
        "name": "no_critical_vulnerabilities",
        "condition": "vulnerabilities.critical == 0",
        "action": "block"
      },
      {
        "name": "no_blocked_licenses",
        "condition": "licenses.blocked == 0",
        "action": "block"
      },
      {
        "name": "outdated_threshold",
        "condition": "outdated.percentage < 20",
        "action": "warn"
      }
    ]
  }
}
```

---

## 7. 自动修复

### 7.1 自动更新

```json
{
  "autoUpdate": {
    "enabled": true,
    "strategies": {
      "security_patches": {
        "enabled": true,
        "auto_merge": true,
        "test_required": true
      },
      "patch_updates": {
        "enabled": true,
        "auto_merge": false,
        "test_required": true
      },
      "minor_updates": {
        "enabled": false,
        "auto_merge": false,
        "test_required": true,
        "review_required": true
      }
    }
  }
}
```

### 7.2 自动修复流程

```markdown
### 自动修复流程

1. **检测问题**
   - 运行漏洞扫描
   - 检查版本更新
   - 识别安全问题

2. **评估风险**
   - 评估更新风险
   - 检查兼容性
   - 确定修复策略

3. **执行修复**
   - 更新依赖版本
   - 运行测试验证
   - 生成修复报告

4. **验证修复**
   - 验证漏洞修复
   - 验证功能正常
   - 记录修复历史
```

---

## 8. 集成到主代理

### 8.1 主代理依赖检查流程

```markdown
### 依赖安全检查流程

1. **依赖扫描**
   - 收集所有依赖
   - 运行漏洞扫描
   - 检查许可证

2. **风险评估**
   - 评估漏洞风险
   - 检查版本兼容性
   - 识别潜在问题

3. **生成报告**
   - 生成安全报告
   - 标记问题依赖
   - 提供修复建议

4. **执行修复**
   - 自动修复漏洞
   - 更新依赖版本
   - 验证修复结果

5. **监控更新**
   - 监控依赖更新
   - 检查新漏洞
   - 持续安全监控
```

### 8.2 主代理提示词更新

在主代理提示词中添加依赖安全检查：

```markdown
#### 依赖安全检查

**漏洞扫描**：
- 集成npm audit/snyk
- 定期扫描依赖漏洞
- 高危漏洞自动报警

**版本管理**：
- 锁定依赖版本
- 定期更新依赖
- 测试兼容性

**许可证检查**：
- 检查依赖许可证
- 避免许可证冲突
- 生成许可证报告

**依赖审计**：
- 检查未使用依赖
- 检查过时依赖
- 检查重复依赖
```

---

## 9. 最佳实践

### 9.1 依赖管理

1. **最小依赖**：只安装必要的依赖
2. **版本锁定**：锁定依赖版本
3. **定期更新**：定期更新依赖
4. **安全扫描**：定期扫描漏洞
5. **许可证检查**：检查依赖许可证

### 9.2 安全实践

1. **及时更新**：及时更新安全补丁
2. **风险评估**：评估依赖安全风险
3. **监控告警**：监控依赖安全状态
4. **应急响应**：建立应急响应机制
5. **安全审计**：定期进行安全审计

### 9.3 版本管理

1. **语义化版本**：遵循语义化版本规范
2. **版本锁定**：锁定生产依赖版本
3. **兼容性检查**：检查版本兼容性
4. **更新策略**：制定更新策略
5. **回滚能力**：保持回滚能力

### 9.4 许可证管理

1. **许可证合规**：确保许可证合规
2. **许可证兼容**：检查许可证兼容性
3. **许可证审计**：定期审计许可证
4. **文档记录**：记录许可证信息
5. **风险控制**：控制许可证风险
