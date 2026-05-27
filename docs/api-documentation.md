# API文档自动生成

本文档定义了系统的API文档自动生成机制，包括OpenAPI集成、文档验证、发布和维护。

## 1. OpenAPI规范配置

### 1.1 OpenAPI版本配置

```json
{
  "openapi": {
    "version": "3.0.3",
    "info": {
      "title": "API Documentation",
      "description": "Auto-generated API documentation",
      "version": "1.0.0",
      "contact": {
        "name": "API Support",
        "email": "support@example.com"
      }
    },
    "servers": [
      {
        "url": "http://localhost:3000",
        "description": "Development server"
      },
      {
        "url": "https://api.example.com",
        "description": "Production server"
      }
    ]
  }
}
```

### 1.2 文档生成配置

```json
{
  "documentationGeneration": {
    "enabled": true,
    "sources": {
      "code_comments": {
        "enabled": true,
        "format": "jsdoc",
        "patterns": ["**/*.ts", "**/*.js"]
      },
      "type_definitions": {
        "enabled": true,
        "format": "typescript",
        "patterns": ["**/*.d.ts", "**/types.ts"]
      },
      "route_definitions": {
        "enabled": true,
        "format": "express",
        "patterns": ["**/routes.ts", "**/controllers/*.ts"]
      }
    },
    "output": {
      "format": "openapi",
      "path": "{PROJECT_ROOT}/docs/api",
      "filename": "openapi.json"
    }
  }
}
```

### 1.3 文档模板

```json
{
  "documentationTemplates": {
    "endpoint": {
      "summary": "简短描述",
      "description": "详细描述",
      "operationId": "唯一操作ID",
      "tags": ["标签"],
      "parameters": [],
      "requestBody": {},
      "responses": {},
      "security": []
    },
    "schema": {
      "type": "object",
      "properties": {},
      "required": [],
      "example": {}
    },
    "response": {
      "description": "响应描述",
      "content": {
        "application/json": {
          "schema": {},
          "example": {}
        }
      }
    }
  }
}
```

---

## 2. 文档生成流程

### 2.1 代码注释解析

```json
{
  "commentParsing": {
    "enabled": true,
    "formats": {
      "jsdoc": {
        "enabled": true,
        "tags": [
          "@api",
          "@apiDescription",
          "@apiParam",
          "@apiSuccess",
          "@apiError",
          "@apiExample"
        ]
      },
      "swagger": {
        "enabled": true,
        "tags": [
          "@swagger",
          "@operation",
          "@parameter",
          "@response"
        ]
      },
      "typescript": {
        "enabled": true,
        "extract": [
          "interfaces",
          "types",
          "enums"
        ]
      }
    }
  }
}
```

### 2.2 路由解析

```json
{
  "routeParsing": {
    "enabled": true,
    "frameworks": {
      "express": {
        "enabled": true,
        "patterns": [
          "app.get('/path', handler)",
          "app.post('/path', handler)",
          "router.get('/path', handler)"
        ]
      },
      "nestjs": {
        "enabled": true,
        "decorators": [
          "@Get('/path')",
          "@Post('/path')",
          "@Controller('/path')"
        ]
      },
      "fastify": {
        "enabled": false,
        "patterns": [
          "fastify.get('/path', handler)"
        ]
      }
    }
  }
}
```

### 2.3 类型解析

```json
{
  "typeParsing": {
    "enabled": true,
    "parsers": {
      "typescript": {
        "enabled": true,
        "extract": [
          "interfaces",
          "type_aliases",
          "enums",
          "classes"
        ]
      },
      "json_schema": {
        "enabled": true,
        "sources": [
          "**/*.schema.json",
          "**/schemas/*.json"
        ]
      }
    }
  }
}
```

---

## 3. 文档验证

### 3.1 验证规则

```json
{
  "validationRules": {
    "completeness": {
      "enabled": true,
      "checks": [
        {
          "name": "all_endpoints_documented",
          "description": "所有端点都有文档",
          "severity": "error"
        },
        {
          "name": "all_parameters_described",
          "description": "所有参数都有描述",
          "severity": "warning"
        },
        {
          "name": "all_responses_defined",
          "description": "所有响应都有定义",
          "severity": "error"
        }
      ]
    },
    "consistency": {
      "enabled": true,
      "checks": [
        {
          "name": "naming_convention",
          "description": "命名规范一致性",
          "severity": "warning"
        },
        {
          "name": "response_format",
          "description": "响应格式一致性",
          "severity": "error"
        }
      ]
    },
    "correctness": {
      "enabled": true,
      "checks": [
        {
          "name": "schema_valid",
          "description": "Schema格式正确",
          "severity": "error"
        },
        {
          "name": "examples_valid",
          "description": "示例数据有效",
          "severity": "warning"
        }
      ]
    }
  }
}
```

### 3.2 验证流程

```markdown
### 文档验证流程

1. **解析文档**
   - 读取OpenAPI文档
   - 解析文档结构
   - 提取关键信息

2. **执行验证**
   - 检查完整性
   - 检查一致性
   - 检查正确性

3. **生成报告**
   - 生成验证报告
   - 标记问题区域
   - 提供修复建议

4. **处理结果**
   - 通过：继续执行
   - 失败：修复后重试
   - 严重：阻断执行
```

### 3.3 验证报告

```json
{
  "validationReport": {
    "timestamp": "yymmdd hhmm",
    "summary": {
      "total_checks": 50,
      "passed": 45,
      "warnings": 3,
      "errors": 2
    },
    "issues": [
      {
        "type": "error",
        "path": "/api/users",
        "message": "Missing response schema for 400 status",
        "suggestion": "Add error response schema"
      },
      {
        "type": "warning",
        "path": "/api/orders",
        "message": "No example provided for request body",
        "suggestion": "Add request body example"
      }
    ]
  }
}
```

---

## 4. 文档发布

### 4.1 发布配置

```json
{
  "documentationPublishing": {
    "enabled": true,
    "formats": {
      "html": {
        "enabled": true,
        "tool": "swagger-ui",
        "config": {
          "deepLinking": true,
          "displayRequestDuration": true
        }
      },
      "markdown": {
        "enabled": true,
        "tool": "openapi-to-markdown",
        "config": {
          "includeExamples": true
        }
      },
      "pdf": {
        "enabled": false,
        "tool": "openapi-to-pdf"
      }
    },
    "destinations": {
      "local": {
        "enabled": true,
        "path": "{PROJECT_ROOT}/docs/api"
      },
      "remote": {
        "enabled": false,
        "provider": "stoplight",
        "config": {
          "project_id": null,
          "api_key": null
        }
      }
    }
  }
}
```

### 4.2 发布流程

```markdown
### 文档发布流程

1. **生成文档**
   - 从代码生成OpenAPI
   - 验证文档完整性
   - 生成发布版本

2. **格式转换**
   - 转换为HTML
   - 转换为Markdown
   - 转换为PDF（可选）

3. **发布部署**
   - 部署到本地
   - 部署到远程
   - 更新版本标签

4. **通知用户**
   - 发布通知
   - 更新变更日志
   - 通知相关人员
```

### 4.3 版本管理

```json
{
  "documentationVersioning": {
    "enabled": true,
    "strategy": "semantic",
    "versioning": {
      "major": "不兼容的API变更",
      "minor": "向后兼容的功能性新增",
      "patch": "向后兼容的问题修正"
    },
    "storage": {
      "current": "docs/api/latest",
      "versions": "docs/api/v{version}",
      "changelog": "docs/api/CHANGELOG.md"
    }
  }
}
```

---

## 5. 文档维护

### 5.1 自动更新

```json
{
  "autoUpdate": {
    "enabled": true,
    "triggers": {
      "on_code_change": true,
      "on_merge": true,
      "on_release": true
    },
    "actions": {
      "regenerate": true,
      "validate": true,
      "publish": true,
      "notify": true
    }
  }
}
```

### 5.2 变更追踪

```json
{
  "changeTracking": {
    "enabled": true,
    "tracking": {
      "endpoints_added": [],
      "endpoints_removed": [],
      "endpoints_modified": [],
      "schemas_changed": []
    },
    "changelog": {
      "enabled": true,
      "format": "markdown",
      "path": "docs/api/CHANGELOG.md"
    }
  }
}
```

### 5.3 文档质量

```json
{
  "documentationQuality": {
    "metrics": {
      "completeness": {
        "description": "文档完整性",
        "target": 95,
        "current": 85
      },
      "accuracy": {
        "description": "文档准确性",
        "target": 99,
        "current": 95
      },
      "freshness": {
        "description": "文档新鲜度",
        "target": 100,
        "current": 90
      }
    },
    "improvement": {
      "suggestions": true,
      "auto_fix": false,
      "manual_review": true
    }
  }
}
```

---

## 6. 工具集成

### 6.1 文档生成工具

```json
{
  "documentationTools": {
    "swagger-jsdoc": {
      "enabled": true,
      "config": {
        "definition": {
          "openapi": "3.0.0",
          "info": {
            "title": "API",
            "version": "1.0.0"
          }
        },
        "apis": ["./src/routes/*.ts"]
      }
    },
    "tsoa": {
      "enabled": false,
      "config": {
        "entryFile": "src/app.ts",
        "outputDirectory": "build",
        "controllerPathGlobs": ["src/controllers/*.ts"]
      }
    },
    "openapi-generator": {
      "enabled": false,
      "config": {
        "inputSpec": "docs/api/openapi.json",
        "outputDir": "docs/api/generated"
      }
    }
  }
}
```

### 6.2 文档查看工具

```json
{
  "documentationViewers": {
    "swagger-ui": {
      "enabled": true,
      "config": {
        "url": "/api/docs/openapi.json",
        "deepLinking": true,
        "displayRequestDuration": true
      }
    },
    "redoc": {
      "enabled": false,
      "config": {
        "specUrl": "/api/docs/openapi.json",
        "theme": {
          "colors": {
            "primary": {
              "main": "#1976d2"
            }
          }
        }
      }
    },
    "stoplight": {
      "enabled": false,
      "config": {
        "projectId": null
      }
    }
  }
}
```

### 6.3 文档测试工具

```json
{
  "documentationTesting": {
    "enabled": true,
    "tools": {
      "dredd": {
        "enabled": true,
        "config": {
          "server": "http://localhost:3000",
          "reporter": "apiary"
        }
      },
      "schemathesis": {
        "enabled": false,
        "config": {
          "url": "http://localhost:3000/api/docs/openapi.json"
        }
      }
    }
  }
}
```

---

## 7. 集成到主代理

### 7.1 主代理API文档流程

```markdown
### API文档生成流程

1. **代码解析**
   - 解析代码注释
   - 解析路由定义
   - 解析类型定义

2. **文档生成**
   - 生成OpenAPI文档
   - 验证文档完整性
   - 生成示例数据

3. **文档验证**
   - 检查文档完整性
   - 检查文档一致性
   - 检查文档正确性

4. **文档发布**
   - 生成HTML版本
   - 生成Markdown版本
   - 部署到文档站点

5. **文档维护**
   - 监控文档变更
   - 自动更新文档
   - 生成变更日志
```

### 7.2 主代理提示词更新

在主代理提示词中添加API文档自动生成：

```markdown
#### API文档自动生成

**文档生成**：
- 从代码注释生成
- 支持OpenAPI/Swagger
- 文档与代码同步

**文档验证**：
- 检查文档完整性
- 验证接口定义
- 生成示例代码

**文档发布**：
- 自动生成文档站点
- 支持在线测试
- 版本管理

**文档维护**：
- 监控文档变更
- 自动更新文档
- 生成变更日志
```

---

## 8. 最佳实践

### 8.1 文档编写

1. **及时更新**：代码变更时及时更新文档
2. **详细描述**：提供详细的接口描述
3. **完整示例**：提供完整的请求响应示例
4. **错误处理**：文档化所有错误响应
5. **版本管理**：文档版本与API版本同步

### 8.2 文档组织

1. **逻辑分组**：按功能模块分组
2. **清晰命名**：使用清晰的命名规范
3. **易于导航**：提供清晰的导航结构
4. **搜索功能**：支持文档搜索
5. **多语言支持**：支持多语言文档

### 8.3 文档质量

1. **准确性**：确保文档与代码一致
2. **完整性**：覆盖所有API端点
3. **时效性**：及时更新过时文档
4. **可读性**：使用清晰的语言
5. **示例丰富**：提供丰富的示例

### 8.4 文档维护

1. **自动化**：尽可能自动化文档生成
2. **持续集成**：集成到CI/CD流程
3. **版本控制**：文档纳入版本控制
4. **审查流程**：文档变更需要审查
5. **反馈机制**：收集用户反馈并改进
