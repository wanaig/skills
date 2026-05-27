# 国际化支持

本文档定义了系统的国际化支持机制，包括界面国际化、文档国际化和日志国际化。

## 1. 界面国际化

### 1.1 i18n框架配置

```json
{
  "i18nFramework": {
    "react": {
      "react-intl": {
        "enabled": true,
        "config": {
          "locale": "zh-CN",
          "fallbackLocale": "en-US",
          "messages": {}
        }
      },
      "react-i18next": {
        "enabled": false,
        "config": {
          "lng": "zh-CN",
          "fallbackLng": "en-US"
        }
      }
    },
    "vue": {
      "vue-i18n": {
        "enabled": true,
        "config": {
          "locale": "zh-CN",
          "fallbackLocale": "en-US"
        }
      }
    }
  }
}
```

### 1.2 语言包结构

```json
{
  "languagePacks": {
    "zh-CN": {
      "common": {
        "ok": "确定",
        "cancel": "取消",
        "save": "保存",
        "delete": "删除",
        "edit": "编辑",
        "create": "创建",
        "search": "搜索",
        "loading": "加载中...",
        "no_data": "暂无数据"
      },
      "auth": {
        "login": "登录",
        "logout": "退出登录",
        "register": "注册",
        "forgot_password": "忘记密码",
        "reset_password": "重置密码"
      },
      "messages": {
        "success": "操作成功",
        "error": "操作失败",
        "confirm_delete": "确定要删除吗？",
        "unsaved_changes": "有未保存的更改"
      }
    },
    "en-US": {
      "common": {
        "ok": "OK",
        "cancel": "Cancel",
        "save": "Save",
        "delete": "Delete",
        "edit": "Edit",
        "create": "Create",
        "search": "Search",
        "loading": "Loading...",
        "no_data": "No Data"
      },
      "auth": {
        "login": "Login",
        "logout": "Logout",
        "register": "Register",
        "forgot_password": "Forgot Password",
        "reset_password": "Reset Password"
      },
      "messages": {
        "success": "Success",
        "error": "Error",
        "confirm_delete": "Are you sure you want to delete?",
        "unsaved_changes": "You have unsaved changes"
      }
    }
  }
}
```

### 1.3 语言包管理

```json
{
  "languagePackManagement": {
    "loading": {
      "lazy": true,
      "chunk": true,
      "fallback": "en-US"
    },
    "storage": {
      "type": "file",
      "path": "src/locales",
      "format": "json"
    },
    "extraction": {
      "enabled": true,
      "tool": "i18next-scanner",
      "patterns": ["src/**/*.{js,jsx,ts,tsx}"]
    }
  }
}
```

---

## 2. 语言切换

### 2.1 切换机制

```json
{
  "languageSwitching": {
    "enabled": true,
    "methods": {
      "ui_selector": {
        "enabled": true,
        "position": "header",
        "options": ["zh-CN", "en-US"]
      },
      "url_parameter": {
        "enabled": true,
        "parameter": "lang",
        "example": "/?lang=en-US"
      },
      "browser_detection": {
        "enabled": true,
        "auto_detect": true,
        "fallback": "en-US"
      },
      "user_preference": {
        "enabled": true,
        "storage": "localStorage",
        "key": "preferred_language"
      }
    }
  }
}
```

### 2.2 切换流程

```markdown
### 语言切换流程

1. **检测语言**
   - 检测URL参数
   - 检测用户偏好
   - 检测浏览器语言
   - 使用默认语言

2. **加载语言包**
   - 加载对应语言包
   - 合并默认语言包
   - 验证翻译完整性

3. **应用语言**
   - 更新界面文本
   - 更新日期格式
   - 更新数字格式
   - 更新货币格式

4. **保存偏好**
   - 保存用户选择
   - 更新用户配置
   - 同步到服务器
```

### 2.3 语言检测

```json
{
  "languageDetection": {
    "detection_order": [
      "url_parameter",
      "user_preference",
      "browser_language",
      "default"
    ],
    "browser_detection": {
      "enabled": true,
      "languages": ["zh-CN", "zh", "en-US", "en"]
    },
    "fallback": {
      "enabled": true,
      "language": "en-US"
    }
  }
}
```

---

## 3. 本地化格式

### 3.1 日期格式

```json
{
  "dateFormat": {
    "zh-CN": {
      "short": "YYYY-MM-DD",
      "medium": "YYYY年MM月DD日",
      "long": "YYYY年MM月DD日 HH:mm:ss",
      "relative": {
        "just_now": "刚刚",
        "minutes_ago": "{n}分钟前",
        "hours_ago": "{n}小时前",
        "days_ago": "{n}天前"
      }
    },
    "en-US": {
      "short": "MM/DD/YYYY",
      "medium": "MMMM D, YYYY",
      "long": "MMMM D, YYYY h:mm:ss A",
      "relative": {
        "just_now": "just now",
        "minutes_ago": "{n} minutes ago",
        "hours_ago": "{n} hours ago",
        "days_ago": "{n} days ago"
      }
    }
  }
}
```

### 3.2 数字格式

```json
{
  "numberFormat": {
    "zh-CN": {
      "decimal": ".",
      "thousands": ",",
      "precision": 2,
      "currency": "¥"
    },
    "en-US": {
      "decimal": ".",
      "thousands": ",",
      "precision": 2,
      "currency": "$"
    }
  }
}
```

### 3.3 货币格式

```json
{
  "currencyFormat": {
    "zh-CN": {
      "symbol": "¥",
      "position": "before",
      "format": "¥{amount}"
    },
    "en-US": {
      "symbol": "$",
      "position": "before",
      "format": "${amount}"
    }
  }
}
```

---

## 4. 文档国际化

### 4.1 文档翻译

```json
{
  "documentationTranslation": {
    "enabled": true,
    "languages": ["zh-CN", "en-US"],
    "source_language": "zh-CN",
    "translation_method": {
      "manual": {
        "enabled": true,
        "description": "人工翻译"
      },
      "machine": {
        "enabled": false,
        "provider": "google_translate",
        "api_key": null
      }
    }
  }
}
```

### 4.2 文档结构

```json
{
  "documentationStructure": {
    "zh-CN": {
      "path": "docs/zh-CN",
      "files": [
        "README.md",
        "API.md",
        "CONTRIBUTING.md"
      ]
    },
    "en-US": {
      "path": "docs/en-US",
      "files": [
        "README.md",
        "API.md",
        "CONTRIBUTING.md"
      ]
    }
  }
}
```

### 4.3 文档同步

```json
{
  "documentationSync": {
    "enabled": true,
    "sync_on_change": true,
    "notify_translators": true,
    "track_progress": true,
    "missing_translations": {
      "action": "show_warning",
      "fallback": "source_language"
    }
  }
}
```

---

## 5. 日志国际化

### 5.1 日志消息

```json
{
  "logMessages": {
    "zh-CN": {
      "system_start": "系统启动",
      "system_stop": "系统停止",
      "user_login": "用户登录",
      "user_logout": "用户退出",
      "error_occurred": "发生错误",
      "warning": "警告"
    },
    "en-US": {
      "system_start": "System started",
      "system_stop": "System stopped",
      "user_login": "User logged in",
      "user_logout": "User logged out",
      "error_occurred": "Error occurred",
      "warning": "Warning"
    }
  }
}
```

### 5.2 日志配置

```json
{
  "logConfiguration": {
    "language": "zh-CN",
    "configurable": true,
    "fallback": "en-US",
    "translation_support": true
  }
}
```

### 5.3 日志格式

```json
{
  "logFormat": {
    "zh-CN": {
      "timestamp": "YYYY-MM-DD HH:mm:ss",
      "level": "级别",
      "message": "消息",
      "context": "上下文"
    },
    "en-US": {
      "timestamp": "YYYY-MM-DD HH:mm:ss",
      "level": "Level",
      "message": "Message",
      "context": "Context"
    }
  }
}
```

---

## 6. 翻译管理

### 6.1 翻译流程

```markdown
### 翻译流程

1. **提取文本**
   - 扫描代码中的文本
   - 提取需要翻译的文本
   - 生成翻译模板

2. **翻译文本**
   - 人工翻译
   - 机器翻译（可选）
   - 审核翻译质量

3. **集成翻译**
   - 更新语言包
   - 验证翻译完整性
   - 测试翻译效果

4. **维护翻译**
   - 监控翻译变更
   - 更新过时翻译
   - 补充缺失翻译
```

### 6.2 翻译工具

```json
{
  "translationTools": {
    "extraction": {
      "tool": "i18next-scanner",
      "config": {
        "input": ["src/**/*.{js,jsx,ts,tsx}"],
        "output": "src/locales/{{lng}}/{{ns}}.json"
      }
    },
    "management": {
      "tool": "crowdin",
      "config": {
        "project_id": null,
        "api_key": null
      }
    },
    "validation": {
      "tool": "i18next-parser",
      "config": {
        "check_missing": true,
        "check_unused": true
      }
    }
  }
}
```

### 6.3 翻译质量

```json
{
  "translationQuality": {
    "checks": {
      "completeness": {
        "enabled": true,
        "threshold": 95
      },
      "consistency": {
        "enabled": true,
        "check_terminology": true
      },
      "accuracy": {
        "enabled": true,
        "review_required": true
      }
    },
    "reporting": {
      "enabled": true,
      "format": "json",
      "output": "translation-report.json"
    }
  }
}
```

---

## 7. 集成到主代理

### 7.1 主代理国际化流程

```markdown
### 国际化流程

1. **初始化**
   - 检测用户语言
   - 加载语言包
   - 应用语言设置

2. **文本处理**
   - 提取需要翻译的文本
   - 翻译文本
   - 应用翻译

3. **格式处理**
   - 格式化日期
   - 格式化数字
   - 格式化货币

4. **文档处理**
   - 翻译文档
   - 同步文档
   - 维护文档

5. **质量保证**
   - 验证翻译完整性
   - 检查翻译质量
   - 修复翻译问题
```

### 7.2 主代理提示词更新

在主代理提示词中添加国际化支持：

```markdown
#### 国际化支持

**界面国际化**：
- 支持中英文切换
- 提取硬编码文本
- 使用i18n框架

**文档国际化**：
- 中英文版本
- 自动翻译辅助
- 同步更新

**日志国际化**：
- 多语言日志
- 可配置语言
- 日志翻译

**本地化格式**：
- 日期格式
- 数字格式
- 货币格式
```

---

## 8. 最佳实践

### 8.1 国际化设计

1. **早期规划**：项目初期考虑国际化
2. **分离文本**：文本与代码分离
3. **使用框架**：使用成熟的i18n框架
4. **统一管理**：统一管理翻译资源
5. **持续维护**：持续维护翻译质量

### 8.2 翻译管理

1. **专业翻译**：使用专业翻译人员
2. **质量审核**：翻译质量审核
3. **术语一致**：保持术语一致性
4. **上下文信息**：提供翻译上下文
5. **及时更新**：及时更新翻译

### 8.3 本地化实践

1. **文化适应**：适应目标文化
2. **格式规范**：遵循本地格式规范
3. **法律合规**：遵守当地法律法规
4. **用户习惯**：考虑用户使用习惯
5. **测试验证**：本地化测试验证

### 8.4 技术实现

1. **性能优化**：语言包懒加载
2. **缓存策略**：翻译缓存策略
3. **回退机制**：翻译缺失回退机制
4. **监控告警**：翻译问题监控
5. **持续改进**：持续改进国际化支持
