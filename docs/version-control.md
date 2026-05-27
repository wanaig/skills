# 版本控制集成

本文档定义了系统的版本控制集成机制，包括Git自动提交、分支管理、变更追踪和冲突处理。

## 1. Git集成配置

### 1.1 基础配置

```json
{
  "gitIntegration": {
    "enabled": true,
    "provider": "git",
    "autoCommit": true,
    "autoPush": false,
    "commitFrequency": "per_batch",
    "signCommits": false,
    "gpgKey": null
  }
}
```

### 1.2 提交信息格式

```json
{
  "commitMessage": {
    "format": "[{type}] {description}",
    "types": {
      "feat": "新功能",
      "fix": "修复bug",
      "docs": "文档更新",
      "style": "代码格式",
      "refactor": "重构",
      "test": "测试",
      "chore": "构建/工具"
    },
    "maxLength": 72,
    "includeBody": true,
    "includeFooter": true,
    "template": "{type}: {description}\n\n{body}\n\n{footer}"
  }
}
```

### 1.3 自动提交规则

```json
{
  "autoCommitRules": {
    "onBatchComplete": true,
    "onPhaseComplete": true,
    "onError": true,
    "onCheckpoint": false,
    "filePatterns": [
      "src/**",
      "test/**",
      "docs/**",
      "*.json",
      "*.md"
    ],
    "excludePatterns": [
      "node_modules/**",
      "dist/**",
      "build/**",
      ".env*",
      "*.log"
    ]
  }
}
```

---

## 2. 分支管理策略

### 2.1 分支类型

```json
{
  "branchStrategy": {
    "main": {
      "description": "主分支，稳定版本",
      "protection": true,
      "requirePullRequest": true,
      "requireReviews": 2,
      "requireStatusChecks": true
    },
    "develop": {
      "description": "开发分支，最新功能",
      "protection": false,
      "requirePullRequest": false,
      "requireReviews": 0,
      "requireStatusChecks": false
    },
    "feature/*": {
      "description": "功能分支",
      "naming": "feature/{module-name}",
      "lifecycle": "temporary",
      "mergeTarget": "develop"
    },
    "hotfix/*": {
      "description": "紧急修复分支",
      "naming": "hotfix/{issue-description}",
      "lifecycle": "temporary",
      "mergeTarget": "main"
    },
    "release/*": {
      "description": "发布分支",
      "naming": "release/{version}",
      "lifecycle": "temporary",
      "mergeTarget": "main"
    }
  }
}
```

### 2.2 分支创建规则

```json
{
  "branchCreation": {
    "featureBranches": {
      "createFrom": "develop",
      "namingConvention": "feature/{module-name}",
      "examples": [
        "feature/user-management",
        "feature/order-processing",
        "feature/payment-integration"
      ]
    },
    "hotfixBranches": {
      "createFrom": "main",
      "namingConvention": "hotfix/{issue-description}",
      "examples": [
        "hotfix/login-error",
        "hotfix/payment-timeout"
      ]
    },
    "releaseBranches": {
      "createFrom": "develop",
      "namingConvention": "release/{version}",
      "examples": [
        "release/1.0.0",
        "release/1.1.0"
      ]
    }
  }
}
```

### 2.3 分支合并策略

```json
{
  "mergeStrategy": {
    "featureToDevelop": {
      "method": "squash",
      "deleteBranch": true,
      "requireCI": true
    },
    "developToMain": {
      "method": "merge",
      "deleteBranch": false,
      "requireCI": true,
      "requireReviews": 2
    },
    "hotfixToMain": {
      "method": "merge",
      "deleteBranch": true,
      "requireCI": true,
      "requireReviews": 1
    }
  }
}
```

---

## 3. 变更追踪机制

### 3.1 变更记录格式

```json
{
  "changeRecord": {
    "id": "CHG-001",
    "timestamp": "yymmdd hhmm",
    "agent": {
      "id": "abc123",
      "type": "dg_frontend_vue_dev",
      "name": "前端开发Agent"
    },
    "batch": {
      "number": 3,
      "phase": "frontend_batch_dev",
      "modules": ["user-list", "user-detail"]
    },
    "changes": {
      "files": [
        {
          "path": "src/components/UserList.vue",
          "action": "modified",
          "linesAdded": 45,
          "linesDeleted": 12
        },
        {
          "path": "src/components/UserDetail.vue",
          "action": "created",
          "linesAdded": 120,
          "linesDeleted": 0
        }
      ],
      "summary": "实现用户列表和详情组件"
    },
    "commit": {
      "hash": "abc123def456",
      "message": "[feat] 实现用户列表和详情组件",
      "branch": "feature/user-management"
    },
    "metrics": {
      "duration": 420,
      "agentCalls": 4,
      "fixRounds": 1
    }
  }
}
```

### 3.2 变更历史存储

```json
{
  "changeHistory": {
    "storage": {
      "type": "file",
      "path": "{PROJECT_ROOT}/outputs/change-history.jsonl",
      "format": "jsonl",
      "rotation": {
        "maxSize": "10MB",
        "maxAge": "90days",
        "maxFiles": 10
      }
    },
    "indexing": {
      "enabled": true,
      "fields": ["timestamp", "agent.type", "batch.phase", "commit.hash"],
      "searchable": true
    }
  }
}
```

### 3.3 变更查询能力

```json
{
  "changeQuery": {
    "filters": {
      "byAgent": "agent.type",
      "byPhase": "batch.phase",
      "byModule": "changes.files.path",
      "byTime": "timestamp",
      "byCommit": "commit.hash"
    },
    "sorting": {
      "default": "timestamp",
      "options": ["timestamp", "agent.type", "batch.number"]
    },
    "pagination": {
      "enabled": true,
      "pageSize": 20,
      "maxPageSize": 100
    }
  }
}
```

---

## 4. 冲突处理机制

### 4.1 冲突检测

```json
{
  "conflictDetection": {
    "enabled": true,
    "strategies": {
      "preCommit": {
        "description": "提交前检测",
        "actions": ["validate", "warn", "block"]
      },
      "preMerge": {
        "description": "合并前检测",
        "actions": ["validate", "warn", "block"]
      },
      "realTime": {
        "description": "实时检测",
        "actions": ["notify", "suggest"]
      }
    },
    "conflictTypes": {
      "content": {
        "description": "内容冲突",
        "detection": "diff comparison",
        "resolution": "manual or auto-merge"
      },
      "structural": {
        "description": "结构冲突",
        "detection": "file structure analysis",
        "resolution": "manual review"
      },
      "semantic": {
        "description": "语义冲突",
        "detection": "code analysis",
        "resolution": "manual review"
      }
    }
  }
}
```

### 4.2 冲突解决策略

```json
{
  "conflictResolution": {
    "autoResolve": {
      "enabled": true,
      "strategies": {
        "whitespace": {
          "description": "空白字符冲突",
          "resolution": "auto-fix",
          "confidence": "high"
        },
        "import": {
          "description": "导入语句冲突",
          "resolution": "merge-both",
          "confidence": "medium"
        },
        "formatting": {
          "description": "格式化冲突",
          "resolution": "apply-prettier",
          "confidence": "high"
        }
      }
    },
    "manualResolve": {
      "required": true,
      "strategies": {
        "content": {
          "description": "内容冲突",
          "resolution": "manual-merge",
          "tools": ["diff-viewer", "merge-tool"]
        },
        "structural": {
          "description": "结构冲突",
          "resolution": "manual-review",
          "tools": ["file-comparison"]
        }
      }
    }
  }
}
```

### 4.3 冲突预防

```json
{
  "conflictPrevention": {
    "strategies": {
      "communication": {
        "description": "Agent间通信",
        "implementation": "shared-state",
        "benefits": "减少并发冲突"
      },
      "locking": {
        "description": "文件锁定",
        "implementation": "optimistic-locking",
        "benefits": "防止同时修改"
      },
      "scheduling": {
        "description": "任务调度",
        "implementation": "sequential-when-conflict",
        "benefits": "避免冲突场景"
      }
    }
  }
}
```

---

## 5. 版本标签管理

### 5.1 标签策略

```json
{
  "taggingStrategy": {
    "enabled": true,
    "types": {
      "version": {
        "format": "v{major}.{minor}.{patch}",
        "examples": ["v1.0.0", "v1.1.0", "v1.1.1"],
        "autoCreate": true,
        "onRelease": true
      },
      "phase": {
        "format": "phase-{phase}-{batch}",
        "examples": ["phase-frontend-1", "phase-backend-3"],
        "autoCreate": true,
        "onBatchComplete": true
      },
      "milestone": {
        "format": "milestone-{name}",
        "examples": ["milestone-mvp", "milestone-release"],
        "autoCreate": false,
        "manual": true
      }
    }
  }
}
```

### 5.2 语义化版本

```json
{
  "semanticVersioning": {
    "enabled": true,
    "rules": {
      "major": "不兼容的API变更",
      "minor": "向后兼容的功能性新增",
      "patch": "向后兼容的问题修正"
    },
    "autoIncrement": {
      "enabled": true,
      "triggers": {
        "major": ["breaking-change"],
        "minor": ["new-feature"],
        "patch": ["bug-fix"]
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
  "codeReview": {
    "enabled": true,
    "rules": {
      "requireReviews": 2,
      "requireApprovals": 1,
      "dismissStaleReviews": true,
      "requireCodeOwnerReviews": false
    },
    "reviewers": {
      "autoAssign": true,
      "strategy": "round-robin",
      "pool": ["team-member-1", "team-member-2", "team-member-3"]
    }
  }
}
```

### 6.2 审查检查项

```json
{
  "reviewChecklist": {
    "codeQuality": [
      "代码风格一致",
      "无重复代码",
      "函数职责单一",
      "命名规范"
    ],
    "functionality": [
      "功能正确实现",
      "边界条件处理",
      "错误处理完整",
      "性能考虑"
    ],
    "security": [
      "无安全漏洞",
      "输入验证",
      "权限检查",
      "敏感数据处理"
    ],
    "testing": [
      "测试覆盖充分",
      "测试用例有效",
      "边界条件测试",
      "错误场景测试"
    ]
  }
}
```

---

## 7. 集成到主代理

### 7.1 主代理Git操作流程

```markdown
### Git集成流程

1. **初始化检查**
   - 检查Git仓库状态
   - 验证远程仓库配置
   - 确认分支策略

2. **批次开始**
   - 创建功能分支（如需要）
   - 切换到工作分支
   - 同步最新代码

3. **批次执行**
   - 记录变更文件
   - 跟踪变更内容
   - 监控冲突

4. **批次完成**
   - 暂存变更文件
   - 生成提交信息
   - 执行自动提交
   - 创建版本标签

5. **错误处理**
   - 提交失败处理
   - 冲突检测和解决
   - 回滚能力
```

### 7.2 主代理提示词更新

在主代理提示词中添加版本控制集成：

```markdown
#### 版本控制集成

**Git配置**：
- 自动提交：每批次完成后
- 提交格式：[type] description
- 分支策略：feature/develop/main

**变更追踪**：
- 记录所有文件变更
- 记录Agent和批次信息
- 支持变更历史查询

**冲突处理**：
- 自动检测冲突
- 简单冲突自动解决
- 复杂冲突提示人工

**版本标签**：
- 版本标签：v{major}.{minor}.{patch}
- 阶段标签：phase-{phase}-{batch}
- 里程碑标签：milestone-{name}
```

---

## 8. 工具集成

### 8.1 Git工具

```json
{
  "gitTools": {
    "cli": "git",
    "libraries": {
      "node": "simple-git",
      "python": "GitPython",
      "java": "JGit"
    },
    "gui": {
      "enabled": false,
      "tool": "git-gui"
    }
  }
}
```

### 8.2 代码托管平台

```json
{
  "codeHosting": {
    "platform": "github",
    "alternatives": ["gitlab", "bitbucket", "gitea"],
    "integration": {
      "api": true,
      "webhooks": true,
      "actions": true,
      "packages": true
    }
  }
}
```

### 8.3 CI/CD集成

```json
{
  "cicdIntegration": {
    "enabled": true,
    "platforms": {
      "github": {
        "actions": true,
        "workflows": true
      },
      "gitlab": {
        "ci": true,
        "pipelines": true
      }
    },
    "triggers": {
      "onPush": true,
      "onPullRequest": true,
      "onTag": true,
      "onSchedule": false
    }
  }
}
```

---

## 9. 最佳实践

### 9.1 提交规范

1. **原子提交**：每次提交只做一件事
2. **清晰信息**：提交信息清晰描述变更
3. **频繁提交**：小批量频繁提交
4. **测试通过**：提交前确保测试通过
5. **代码审查**：重要变更进行代码审查

### 9.2 分支管理

1. **主分支稳定**：main分支始终可用
2. **功能分支隔离**：每个功能独立分支
3. **及时合并**：功能完成后及时合并
4. **删除临时分支**：合并后删除功能分支
5. **分支命名规范**：遵循命名约定

### 9.3 变更追踪

1. **完整记录**：记录所有变更
2. **关联任务**：变更关联到具体任务
3. **可追溯性**：支持变更历史查询
4. **审计日志**：保留完整审计日志
5. **定期清理**：定期清理过期记录

### 9.4 冲突处理

1. **预防为主**：通过良好实践预防冲突
2. **及时解决**：发现冲突立即解决
3. **沟通协调**：冲突时及时沟通
4. **工具辅助**：使用工具辅助解决
5. **记录学习**：记录冲突原因和解决方案
