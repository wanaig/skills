# 代码质量工具集成

本文档定义了系统的代码质量工具集成机制，包括ESLint、Prettier、静态分析和代码风格统一。

## 1. 代码质量工具配置

### 1.1 前端工具链

```json
{
  "frontendToolchain": {
    "eslint": {
      "enabled": true,
      "version": "8.x",
      "config": {
        "extends": [
          "eslint:recommended",
          "plugin:react/recommended",
          "plugin:@typescript-eslint/recommended"
        ],
        "rules": {
          "no-unused-vars": "error",
          "no-console": "warn",
          "prefer-const": "error",
          "no-var": "error"
        },
        "plugins": ["react", "@typescript-eslint"]
      },
      "autoFix": true,
      "fixOnSave": true,
      "fixOnCommit": true
    },
    "prettier": {
      "enabled": true,
      "version": "3.x",
      "config": {
        "semi": true,
        "singleQuote": true,
        "tabWidth": 2,
        "trailingComma": "es5",
        "printWidth": 100,
        "bracketSpacing": true,
        "arrowParens": "avoid"
      },
      "autoFormat": true,
      "formatOnSave": true,
      "formatOnCommit": true
    },
    "typescript": {
      "enabled": true,
      "strict": true,
      "noImplicitAny": true,
      "strictNullChecks": true,
      "noImplicitReturns": true,
      "noFallthroughCasesInSwitch": true
    }
  }
}
```

### 1.2 后端工具链

```json
{
  "backendToolchain": {
    "java": {
      "checkstyle": {
        "enabled": true,
        "config": "google_checks.xml",
        "version": "10.x"
      },
      "spotbugs": {
        "enabled": true,
        "config": "spotbugs-exclude.xml",
        "version": "4.x"
      },
      "pmd": {
        "enabled": true,
        "config": "pmd-ruleset.xml",
        "version": "6.x"
      }
    },
    "nodejs": {
      "eslint": {
        "enabled": true,
        "config": {
          "extends": ["eslint:recommended", "plugin:node/recommended"],
          "rules": {
            "no-console": "warn",
            "no-unused-vars": "error"
          }
        }
      },
      "prettier": {
        "enabled": true,
        "config": {
          "semi": true,
          "singleQuote": true,
          "tabWidth": 2
        }
      }
    },
    "python": {
      "pylint": {
        "enabled": true,
        "config": ".pylintrc",
        "version": "3.x"
      },
      "black": {
        "enabled": true,
        "config": {
          "line_length": 88,
          "target_version": "py39"
        }
      },
      "mypy": {
        "enabled": true,
        "config": {
          "strict": true,
          "warn_return_any": true,
          "warn_unused_configs": true
        }
      }
    }
  }
}
```

---

## 2. 代码风格规范

### 2.1 命名规范

```json
{
  "namingConventions": {
    "variables": {
      "javascript": "camelCase",
      "typescript": "camelCase",
      "java": "camelCase",
      "python": "snake_case"
    },
    "functions": {
      "javascript": "camelCase",
      "typescript": "camelCase",
      "java": "camelCase",
      "python": "snake_case"
    },
    "classes": {
      "javascript": "PascalCase",
      "typescript": "PascalCase",
      "java": "PascalCase",
      "python": "PascalCase"
    },
    "constants": {
      "javascript": "UPPER_SNAKE_CASE",
      "typescript": "UPPER_SNAKE_CASE",
      "java": "UPPER_SNAKE_CASE",
      "python": "UPPER_SNAKE_CASE"
    },
    "files": {
      "javascript": "kebab-case",
      "typescript": "kebab-case",
      "java": "PascalCase",
      "python": "snake_case"
    }
  }
}
```

### 2.2 代码格式

```json
{
  "codeFormatting": {
    "indentation": {
      "style": "spaces",
      "size": 2,
      "language_specific": {
        "python": 4,
        "java": 4
      }
    },
    "lineLength": {
      "max": 100,
      "language_specific": {
        "python": 88,
        "java": 120
      }
    },
    "braces": {
      "style": "1tbs",
      "language_specific": {
        "java": "allman"
      }
    },
    "semicolons": {
      "required": true,
      "language_specific": {
        "python": false
      }
    },
    "quotes": {
      "style": "single",
      "language_specific": {
        "java": "double"
      }
    }
  }
}
```

### 2.3 代码组织

```json
{
  "codeOrganization": {
    "imports": {
      "order": [
        "node_modules",
        "absolute_imports",
        "relative_imports",
        "styles"
      ],
      "group_separator": true,
      "alphabetical": true
    },
    "exports": {
      "style": "named",
      "default_export": false,
      "index_files": false
    },
    "file_structure": {
      "components": [
        "imports",
        "types",
        "constants",
        "component",
        "exports"
      ],
      "utils": [
        "imports",
        "types",
        "functions",
        "exports"
      ]
    }
  }
}
```

---

## 3. 静态分析规则

### 3.1 代码异味检测

```json
{
  "codeSmellDetection": {
    "longFunctions": {
      "enabled": true,
      "threshold": 50,
      "action": "warn",
      "message": "函数过长，建议拆分"
    },
    "longFiles": {
      "enabled": true,
      "threshold": 300,
      "action": "warn",
      "message": "文件过长，建议拆分"
    },
    "deepNesting": {
      "enabled": true,
      "threshold": 4,
      "action": "warn",
      "message": "嵌套过深，建议重构"
    },
    "duplicateCode": {
      "enabled": true,
      "threshold": 10,
      "action": "warn",
      "message": "存在重复代码，建议提取"
    },
    "complexConditions": {
      "enabled": true,
      "threshold": 3,
      "action": "warn",
      "message": "条件过复杂，建议简化"
    }
  }
}
```

### 3.2 最佳实践检查

```json
{
  "bestPracticesChecks": {
    "errorHandling": {
      "noEmptyCatch": true,
      "noThrowLiteral": true,
      "catchSpecificErrors": true
    },
    "performance": {
      "noConsoleInProduction": true,
      "noDebugger": true,
      "noAlert": true
    },
    "security": {
      "noEval": true,
      "noInnerHTML": true,
      "noDocumentWrite": true
    },
    "accessibility": {
      "altText": true,
      "labelFor": true,
      "tabindex": true
    }
  }
}
```

### 3.3 类型安全检查

```json
{
  "typeSafetyChecks": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true
  }
}
```

---

## 4. 自动修复机制

### 4.1 可自动修复的问题

```json
{
  "autoFixableIssues": {
    "formatting": [
      "indentation",
      "spacing",
      "semicolons",
      "quotes",
      "trailing_commas"
    ],
    "imports": [
      "unused_imports",
      "import_order",
      "import_sorting"
    ],
    "syntax": [
      "missing_semicolons",
      "missing_commas",
      "bracket_spacing"
    ],
    "style": [
      "prefer_const",
      "prefer_arrow_functions",
      "object_shorthand"
    ]
  }
}
```

### 4.2 自动修复配置

```json
{
  "autoFixConfig": {
    "enabled": true,
    "fixOnSave": true,
    "fixOnCommit": true,
    "fixOnPush": false,
    "strategies": {
      "safe": {
        "description": "安全修复，不会改变代码行为",
        "enabled": true,
        "rules": ["formatting", "imports", "syntax"]
      },
      "unsafe": {
        "description": "不安全修复，可能改变代码行为",
        "enabled": false,
        "rules": ["logic", "performance"]
      }
    }
  }
}
```

### 4.3 修复报告

```json
{
  "fixReport": {
    "enabled": true,
    "format": "json",
    "content": {
      "fixed": [
        {
          "file": "src/components/UserList.vue",
          "line": 45,
          "rule": "prefer-const",
          "message": "使用 const 代替 let"
        }
      ],
      "unfixable": [
        {
          "file": "src/utils/helper.ts",
          "line": 23,
          "rule": "no-unused-vars",
          "message": "未使用的变量，需要手动删除"
        }
      ],
      "summary": {
        "total": 10,
        "fixed": 8,
        "unfixable": 2
      }
    }
  }
}
```

---

## 5. 集成到开发流程

### 5.1 提交前检查

```json
{
  "preCommitChecks": {
    "enabled": true,
    "checks": [
      {
        "name": "lint",
        "command": "eslint --fix",
        "files": ["src/**/*.{js,ts,jsx,tsx}"],
        "failOnError": true
      },
      {
        "name": "format",
        "command": "prettier --write",
        "files": ["src/**/*.{js,ts,jsx,tsx,css,scss,json,md}"],
        "failOnError": true
      },
      {
        "name": "type-check",
        "command": "tsc --noEmit",
        "files": ["src/**/*.{ts,tsx}"],
        "failOnError": true
      }
    ]
  }
}
```

### 5.2 CI/CD集成

```json
{
  "cicdIntegration": {
    "stages": {
      "lint": {
        "enabled": true,
        "commands": [
          "npm run lint",
          "npm run format:check"
        ],
        "failOnError": true
      },
      "type-check": {
        "enabled": true,
        "commands": [
          "npm run type-check"
        ],
        "failOnError": true
      },
      "security": {
        "enabled": true,
        "commands": [
          "npm audit",
          "npm run security-check"
        ],
        "failOnError": false
      }
    }
  }
}
```

### 5.3 IDE集成

```json
{
  "ideIntegration": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "ms-vscode.vscode-typescript-next"
      ],
      "settings": {
        "editor.formatOnSave": true,
        "editor.codeActionsOnSave": {
          "source.fixAll.eslint": true
        }
      }
    },
    "intellij": {
      "plugins": [
        "ESLint",
        "Prettier",
        "TypeScript"
      ],
      "settings": {
        "formatOnSave": true,
        "lintOnSave": true
      }
    }
  }
}
```

---

## 6. 代码审查集成

### 6.1 审查规则

```json
{
  "codeReviewRules": {
    "automated": {
      "enabled": true,
      "checks": [
        {
          "name": "code_style",
          "description": "代码风格检查",
          "tools": ["eslint", "prettier"],
          "failOnError": true
        },
        {
          "name": "type_safety",
          "description": "类型安全检查",
          "tools": ["typescript"],
          "failOnError": true
        },
        {
          "name": "security",
          "description": "安全检查",
          "tools": ["npm audit", "snyk"],
          "failOnError": false
        }
      ]
    },
    "manual": {
      "enabled": true,
      "checklist": [
        "代码风格一致",
        "无重复代码",
        "函数职责单一",
        "命名规范",
        "注释清晰",
        "测试覆盖"
      ]
    }
  }
}
```

### 6.2 审查报告

```json
{
  "reviewReport": {
    "enabled": true,
    "format": "markdown",
    "content": {
      "summary": {
        "total_files": 10,
        "issues_found": 5,
        "critical": 0,
        "major": 2,
        "minor": 3
      },
      "issues": [
        {
          "file": "src/components/UserList.vue",
          "line": 45,
          "severity": "major",
          "rule": "no-unused-vars",
          "message": "未使用的变量 'temp'",
          "suggestion": "删除或使用该变量"
        }
      ],
      "metrics": {
        "code_quality_score": 85,
        "test_coverage": 78,
        "complexity": 12
      }
    }
  }
}
```

---

## 7. 集成到主代理

### 7.1 主代理代码质量检查流程

```markdown
### 代码质量检查流程

1. **代码生成后**
   - 运行ESLint检查
   - 运行Prettier格式化
   - 运行类型检查

2. **问题修复**
   - 自动修复可修复问题
   - 记录不可修复问题
   - 生成修复报告

3. **质量验证**
   - 检查代码风格
   - 检查代码异味
   - 检查最佳实践

4. **报告生成**
   - 生成质量报告
   - 标记问题区域
   - 提供改进建议

5. **结果处理**
   - 通过：继续执行
   - 失败：修复后重试
   - 严重：阻断执行
```

### 7.2 主代理提示词更新

在主代理提示词中添加代码质量工具集成：

```markdown
#### 代码质量工具集成

**前端工具**：
- ESLint：代码规范检查
- Prettier：代码格式化
- TypeScript：类型检查

**后端工具**：
- Java：Checkstyle + SpotBugs + PMD
- Node.js：ESLint + Prettier
- Python：pylint + black + mypy

**自动修复**：
- 可修复问题自动修复
- 不可修复问题提示
- 提交前自动格式化

**质量检查**：
- 代码风格检查
- 代码异味检测
- 最佳实践检查
```

---

## 8. 最佳实践

### 8.1 工具选择

1. **一致性**：团队使用相同的工具配置
2. **渐进性**：逐步引入更严格的规则
3. **自动化**：尽可能自动化检查和修复
4. **集成性**：与开发流程深度集成
5. **可配置性**：支持项目特定配置

### 8.2 规则配置

1. **合理严格**：规则不宜过严或过松
2. **团队共识**：规则需要团队达成共识
3. **文档化**：规则配置需要文档说明
4. **版本化**：规则配置需要版本管理
5. **可扩展**：支持自定义规则

### 8.3 自动化实践

1. **提交前检查**：提交前自动检查
2. **保存时格式化**：保存时自动格式化
3. **CI/CD集成**：CI/CD自动检查
4. **IDE集成**：IDE实时检查
5. **报告自动化**：自动生成报告

### 8.4 持续改进

1. **定期审查**：定期审查规则配置
2. **收集团队反馈**：收集团队使用反馈
3. **更新工具版本**：及时更新工具版本
4. **优化规则**：根据实践优化规则
5. **分享最佳实践**：团队分享最佳实践
