# 错误恢复机制

本文档定义了系统的错误恢复机制，包括错误分类、智能重试、部分恢复和降级质量保障。

## 1. 错误分类体系

### 1.1 错误类型定义

```json
{
  "errorTypes": {
    "recoverable": {
      "description": "可恢复错误",
      "characteristics": [
        "临时性问题",
        "外部依赖问题",
        "资源暂时不足"
      ],
      "examples": [
        "Agent超时",
        "网络连接失败",
        "API速率限制",
        "服务暂时不可用"
      ],
      "handling": "自动重试"
    },
    "nonRecoverable": {
      "description": "不可恢复错误",
      "characteristics": [
        "逻辑错误",
        "需求理解偏差",
        "代码语法错误"
      ],
      "examples": [
        "代码编译失败",
        "测试用例失败",
        "类型检查错误",
        "业务逻辑错误"
      ],
      "handling": "人工介入"
    },
    "platform": {
      "description": "平台错误",
      "characteristics": [
        "平台服务问题",
        "配额限制",
        "权限问题"
      ],
      "examples": [
        "API配额耗尽",
        "服务维护中",
        "认证失败",
        "权限不足"
      ],
      "handling": "等待或切换方案"
    }
  }
}
```

### 1.2 错误严重程度

```json
{
  "severityLevels": {
    "critical": {
      "description": "严重错误",
      "impact": "系统无法继续运行",
      "response": "立即停止，人工介入",
      "examples": ["数据丢失", "安全漏洞", "系统崩溃"]
    },
    "major": {
      "description": "主要错误",
      "impact": "功能无法正常工作",
      "response": "暂停当前任务，尝试恢复",
      "examples": ["核心功能失败", "数据不一致"]
    },
    "minor": {
      "description": "次要错误",
      "impact": "部分功能受影响",
      "response": "记录日志，继续执行",
      "examples": ["非核心功能失败", "性能下降"]
    },
    "warning": {
      "description": "警告",
      "impact": "潜在问题",
      "response": "记录日志，监控",
      "examples": ["性能警告", "资源使用警告"]
    }
  }
}
```

---

## 2. 智能重试机制

### 2.1 重试策略配置

```json
{
  "retryStrategy": {
    "enabled": true,
    "maxRetries": 3,
    "backoff": {
      "type": "exponential",
      "initialDelay": 1000,
      "multiplier": 2,
      "maxDelay": 30000,
      "jitter": true
    },
    "retryableErrors": [
      "timeout",
      "network_error",
      "rate_limit",
      "service_unavailable",
      "connection_reset"
    ],
    "nonRetryableErrors": [
      "authentication_failed",
      "authorization_failed",
      "invalid_input",
      "not_found"
    ]
  }
}
```

### 2.2 重试执行流程

```markdown
### 重试流程

1. **错误发生**
   - 捕获错误
   - 记录错误信息

2. **错误分类**
   - 判断错误类型
   - 确定是否可重试

3. **重试决策**
   - 检查重试次数
   - 检查重试条件
   - 决定是否重试

4. **重试执行**
   - 等待退避时间
   - 执行重试操作
   - 记录重试结果

5. **结果处理**
   - 成功：继续执行
   - 失败：升级处理
```

### 2.3 退避策略

```json
{
  "backoffStrategies": {
    "exponential": {
      "description": "指数退避",
      "formula": "delay = initialDelay * (multiplier ^ retryCount)",
      "example": "1s, 2s, 4s, 8s, 16s"
    },
    "linear": {
      "description": "线性退避",
      "formula": "delay = initialDelay + (increment * retryCount)",
      "example": "1s, 2s, 3s, 4s, 5s"
    },
    "fixed": {
      "description": "固定退避",
      "formula": "delay = fixedDelay",
      "example": "5s, 5s, 5s, 5s, 5s"
    }
  }
}
```

---

## 3. 部分恢复机制

### 3.1 检查点管理

```json
{
  "checkpointManagement": {
    "enabled": true,
    "frequency": "per_batch",
    "storage": {
      "type": "file",
      "path": "{PROJECT_ROOT}/outputs/checkpoint.json",
      "backup": true,
      "backupPath": "{PROJECT_ROOT}/outputs/checkpoint_backup.json"
    },
    "statePreservation": [
      "current_phase",
      "current_batch",
      "completed_tasks",
      "in_progress_tasks",
      "pending_tasks",
      "agent_sessions",
      "test_results",
      "metrics"
    ]
  }
}
```

### 3.2 检查点数据结构

```json
{
  "checkpoint": {
    "version": "1.0",
    "timestamp": "yymmdd hhmm",
    "systemState": {
      "currentPhase": "frontend_batch_dev",
      "currentBatch": 3,
      "totalBatches": 7,
      "status": "running"
    },
    "taskState": {
      "completed": ["task1", "task2", "task3"],
      "inProgress": ["task4"],
      "pending": ["task5", "task6", "task7"]
    },
    "agentState": {
      "dev": {"id": "abc123", "status": "active", "createdAt": "yymmdd hhmm"},
      "test_component": {"id": "def456", "status": "completed"},
      "test_logic": {"id": "ghi789", "status": "active"}
    },
    "metrics": {
      "totalAgentCalls": 45,
      "totalFixRounds": 5,
      "startTime": "yymmdd hhmm"
    }
  }
}
```

### 3.3 恢复流程

```markdown
### 恢复流程

1. **检测中断**
   - 检查系统状态
   - 识别中断原因

2. **加载检查点**
   - 读取检查点文件
   - 验证数据完整性

3. **状态恢复**
   - 恢复系统状态
   - 恢复任务状态
   - 恢复Agent状态

4. **会话重建**
   - 检查会话有效性
   - 重建失效会话
   - 恢复会话上下文

5. **继续执行**
   - 从中断点继续
   - 跳过已完成任务
   - 执行待完成任务
```

---

## 4. 错误处理策略

### 4.1 错误处理决策树

```
错误发生
├── 错误分类
│   ├── 可恢复错误
│   │   ├── 检查重试次数
│   │   │   ├── 未超过限制 → 自动重试
│   │   │   └── 超过限制 → 升级为不可恢复
│   │   └── 记录重试日志
│   ├── 不可恢复错误
│   │   ├── 生成错误报告
│   │   ├── 写入 needs-human-review.md
│   │   └── 请求人工介入
│   └── 平台错误
│       ├── 检查平台状态
│       │   ├── 服务可用 → 重试
│       │   └── 服务不可用 → 等待或切换方案
│       └── 记录平台日志
└── 结果处理
    ├── 成功 → 继续执行
    └── 失败 → 降级处理
```

### 4.2 降级处理策略

```json
{
  "degradationStrategy": {
    "enabled": true,
    "levels": {
      "level1": {
        "description": "跳过失败任务",
        "conditions": ["minor_error", "non_critical_task"],
        "actions": ["skip_task", "log_warning", "continue"]
      },
      "level2": {
        "description": "降级通过",
        "conditions": ["major_error", "critical_task", "max_retries_exceeded"],
        "actions": ["mark_degraded", "generate_report", "continue"]
      },
      "level3": {
        "description": "暂停执行",
        "conditions": ["critical_error", "system_failure"],
        "actions": ["pause_execution", "notify_user", "wait_for_intervention"]
      }
    },
    "qualityGuarantee": {
      "markDegraded": true,
      "requireHumanReview": true,
      "generateReport": true,
      "blockDeployment": false,
      "notifyStakeholders": true
    }
  }
}
```

### 4.3 错误报告格式

```json
{
  "errorReport": {
    "timestamp": "yymmdd hhmm",
    "errorId": "ERR-001",
    "errorType": "recoverable",
    "severity": "major",
    "message": "Agent dg_frontend_vue_dev 超时",
    "context": {
      "agentId": "abc123",
      "agentType": "dg_frontend_vue_dev",
      "batch": 3,
      "task": "开发用户列表模块"
    },
    "stackTrace": "...",
    "retryHistory": [
      {"attempt": 1, "timestamp": "yymmdd hhmm", "result": "timeout"},
      {"attempt": 2, "timestamp": "yymmdd hhmm", "result": "timeout"},
      {"attempt": 3, "timestamp": "yymmdd hhmm", "result": "failed"}
    ],
    "resolution": {
      "action": "degrade",
      "reason": "max_retries_exceeded",
      "nextSteps": ["generate_report", "continue_execution"]
    }
  }
}
```

---

## 5. 实现示例

### 5.1 错误分类器实现

```javascript
// 错误分类器
class ErrorClassifier {
  constructor() {
    this.recoverableErrors = [
      'timeout',
      'network_error',
      'rate_limit',
      'service_unavailable',
      'connection_reset'
    ];
    
    this.nonRecoverableErrors = [
      'authentication_failed',
      'authorization_failed',
      'invalid_input',
      'not_found',
      'logic_error'
    ];
    
    this.platformErrors = [
      'api_quota_exceeded',
      'service_maintenance',
      'platform_error'
    ];
  }
  
  classify(error) {
    const errorType = error.type || error.code;
    
    if (this.recoverableErrors.includes(errorType)) {
      return 'recoverable';
    }
    
    if (this.nonRecoverableErrors.includes(errorType)) {
      return 'nonRecoverable';
    }
    
    if (this.platformErrors.includes(errorType)) {
      return 'platform';
    }
    
    // 默认根据错误消息判断
    if (error.message.includes('timeout')) return 'recoverable';
    if (error.message.includes('network')) return 'recoverable';
    if (error.message.includes('syntax')) return 'nonRecoverable';
    
    return 'unknown';
  }
  
  getSeverity(error) {
    const errorType = this.classify(error);
    
    switch (errorType) {
      case 'recoverable':
        return 'minor';
      case 'nonRecoverable':
        return 'major';
      case 'platform':
        return 'warning';
      default:
        return 'minor';
    }
  }
}
```

### 5.2 重试管理器实现

```javascript
// 重试管理器
class RetryManager {
  constructor(config = {}) {
    this.maxRetries = config.maxRetries || 3;
    this.initialDelay = config.initialDelay || 1000;
    this.multiplier = config.multiplier || 2;
    this.maxDelay = config.maxDelay || 30000;
    this.jitter = config.jitter || true;
  }
  
  async executeWithRetry(fn, context = {}) {
    let lastError;
    
    for (let attempt = 0; attempt <= this.maxRetries; attempt++) {
      try {
        const result = await fn();
        return {
          success: true,
          result,
          attempts: attempt + 1
        };
      } catch (error) {
        lastError = error;
        
        // 检查是否可重试
        if (!this.isRetryable(error)) {
          throw error;
        }
        
        // 最后一次尝试失败
        if (attempt === this.maxRetries) {
          break;
        }
        
        // 等待退避时间
        const delay = this.calculateDelay(attempt);
        await this.sleep(delay);
      }
    }
    
    return {
      success: false,
      error: lastError,
      attempts: this.maxRetries + 1
    };
  }
  
  isRetryable(error) {
    const retryableErrors = [
      'timeout',
      'network_error',
      'rate_limit',
      'service_unavailable'
    ];
    
    return retryableErrors.some(type => 
      error.type === type || error.message.includes(type)
    );
  }
  
  calculateDelay(attempt) {
    let delay = this.initialDelay * Math.pow(this.multiplier, attempt);
    
    // 添加抖动
    if (this.jitter) {
      delay = delay * (0.5 + Math.random());
    }
    
    // 限制最大延迟
    return Math.min(delay, this.maxDelay);
  }
  
  sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

### 5.3 检查点管理器实现

```javascript
// 检查点管理器
class CheckpointManager {
  constructor(config = {}) {
    this.checkpointPath = config.checkpointPath || './checkpoint.json';
    this.backupPath = config.backupPath || './checkpoint_backup.json';
    this.autoSave = config.autoSave || true;
  }
  
  async save(state) {
    const checkpoint = {
      version: '1.0',
      timestamp: new Date().toISOString(),
      state: state
    };
    
    // 备份现有检查点
    await this.backup();
    
    // 保存新检查点
    await this.writeFile(this.checkpointPath, JSON.stringify(checkpoint, null, 2));
    
    return checkpoint;
  }
  
  async restore() {
    try {
      const data = await this.readFile(this.checkpointPath);
      const checkpoint = JSON.parse(data);
      
      // 验证检查点
      if (!this.validate(checkpoint)) {
        throw new Error('Invalid checkpoint');
      }
      
      return checkpoint;
    } catch (error) {
      // 尝试从备份恢复
      console.warn('Failed to restore from checkpoint, trying backup...');
      return await this.restoreFromBackup();
    }
  }
  
  async backup() {
    try {
      const exists = await this.fileExists(this.checkpointPath);
      if (exists) {
        const data = await this.readFile(this.checkpointPath);
        await this.writeFile(this.backupPath, data);
      }
    } catch (error) {
      console.warn('Failed to backup checkpoint:', error);
    }
  }
  
  async restoreFromBackup() {
    try {
      const data = await this.readFile(this.backupPath);
      return JSON.parse(data);
    } catch (error) {
      throw new Error('Failed to restore from backup');
    }
  }
  
  validate(checkpoint) {
    return checkpoint.version && checkpoint.timestamp && checkpoint.state;
  }
}
```

---

## 6. 集成到主代理

### 6.1 主代理错误处理流程

```markdown
### 错误处理流程

1. **捕获错误**
   - 在所有子Agent调用处添加try-catch
   - 记录错误详情

2. **分类错误**
   - 使用ErrorClassifier分类
   - 确定严重程度

3. **处理错误**
   - 可恢复：使用RetryManager重试
   - 不可恢复：生成报告，请求人工
   - 平台：等待或切换方案

4. **记录日志**
   - 写入events.jsonl
   - 更新metrics.json
   - 生成alerts.jsonl

5. **恢复执行**
   - 成功：继续执行
   - 失败：降级处理
```

### 6.2 主代理提示词更新

在主代理提示词中添加错误恢复机制：

```markdown
#### 错误恢复机制

**错误分类**：
- 可恢复：超时、网络问题、API限制
- 不可恢复：逻辑错误、代码错误
- 平台错误：服务不可用、配额耗尽

**重试策略**：
- 最大重试次数：3次
- 退避策略：指数退避（1s, 2s, 4s）
- 可重试错误：自动重试
- 不可重试错误：记录并请求人工

**部分恢复**：
- 检查点保存：每批次完成后
- 恢复能力：从最后检查点恢复
- 状态保持：完整系统状态

**降级处理**：
- 跳过失败任务（minor）
- 降级通过（major）
- 暂停执行（critical）
```

---

## 7. 监控和告警

### 7.1 错误监控指标

```json
{
  "errorMetrics": {
    "totalErrors": 0,
    "errorsByType": {
      "recoverable": 0,
      "nonRecoverable": 0,
      "platform": 0
    },
    "errorsBySeverity": {
      "critical": 0,
      "major": 0,
      "minor": 0,
      "warning": 0
    },
    "retrySuccessRate": 0,
    "averageRetries": 0,
    "recoveryTime": 0
  }
}
```

### 7.2 告警规则

```json
{
  "alertRules": [
    {
      "name": "high_error_rate",
      "condition": "errors.total > 10",
      "severity": "warning",
      "message": "错误率过高"
    },
    {
      "name": "critical_error",
      "condition": "errors.critical > 0",
      "severity": "critical",
      "message": "发生严重错误"
    },
    {
      "name": "low_retry_success",
      "condition": "retry.successRate < 0.5",
      "severity": "warning",
      "message": "重试成功率过低"
    }
  ]
}
```

---

## 8. 最佳实践

### 8.1 错误处理原则

1. **快速失败**：尽早检测错误，避免无效工作
2. **优雅降级**：在错误发生时保持系统可用
3. **完整日志**：记录所有错误和恢复操作
4. **及时告警**：严重错误立即通知
5. **持续改进**：分析错误模式，优化处理策略

### 8.2 重试策略建议

1. **指数退避**：避免重试风暴
2. **抖动添加**：避免同步重试
3. **最大限制**：防止无限重试
4. **错误分类**：只重试可恢复错误
5. **监控告警**：跟踪重试成功率

### 8.3 检查点管理建议

1. **定期保存**：每批次完成后保存
2. **数据验证**：保存前验证数据完整性
3. **备份机制**：保留历史检查点
4. **恢复测试**：定期测试恢复能力
5. **清理策略**：定期清理过期检查点
