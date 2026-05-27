# 超时检测与恢复机制

本文档定义了系统的超时检测和恢复机制，解决卡住问题。

---

## 1. 超时检测配置

### 1.1 超时阈值

```json
{
  "timeout_thresholds": {
    "agent_spawn": {
      "description": "Agent启动超时",
      "threshold": 60,
      "unit": "seconds"
    },
    "agent_execution": {
      "description": "Agent执行超时",
      "threshold": 300,
      "unit": "seconds"
    },
    "batch_execution": {
      "description": "批次执行超时",
      "threshold": 1800,
      "unit": "seconds"
    },
    "phase_execution": {
      "description": "阶段执行超时",
      "threshold": 7200,
      "unit": "seconds"
    },
    "total_execution": {
      "description": "总执行超时",
      "threshold": 28800,
      "unit": "seconds"
    }
  }
}
```

### 1.2 检测频率

```json
{
  "detection_frequency": {
    "health_check": 30,
    "timeout_check": 60,
    "progress_check": 120,
    "unit": "seconds"
  }
}
```

---

## 2. 健康检查机制

### 2.1 检查项目

```json
{
  "health_checks": {
    "agent_responsive": {
      "description": "Agent是否响应",
      "method": "ping",
      "timeout": 10
    },
    "task_progressing": {
      "description": "任务是否在推进",
      "method": "check_output",
      "timeout": 30
    },
    "resources_sufficient": {
      "description": "资源是否充足",
      "method": "check_resources",
      "timeout": 5
    },
    "network_available": {
      "description": "网络是否正常",
      "method": "ping_api",
      "timeout": 10
    }
  }
}
```

### 2.2 健康状态

```json
{
  "health_status": {
    "healthy": {
      "description": "正常运行",
      "action": "continue"
    },
    "degraded": {
      "description": "部分功能受影响",
      "action": "warn_and_continue"
    },
    "unhealthy": {
      "description": "无法正常运行",
      "action": "pause_and_recover"
    }
  }
}
```

---

## 3. 恢复策略

### 3.1 自动恢复

```markdown
#### 自动恢复流程

1. **检测问题**
   - 超时检测
   - 健康检查
   - 错误检测

2. **评估严重程度**
   - 可恢复：自动重试
   - 不可恢复：记录并跳过
   - 平台错误：等待或切换

3. **执行恢复**
   - 保存当前状态到检查点
   - 终止卡住的Agent
   - 创建新会话
   - 从检查点恢复状态
   - 继续执行

4. **验证恢复**
   - 检查新会话是否正常
   - 检查任务是否在推进
   - 记录恢复日志
```

### 3.2 手动恢复

```markdown
#### 手动恢复流程

1. **诊断问题**
   - 查看日志
   - 检查检查点
   - 分析错误原因

2. **选择恢复方案**
   - 从检查点恢复
   - 跳过卡住的任务
   - 重置状态重新开始

3. **执行恢复**
   - 按照恢复方案操作
   - 验证恢复成功
   - 记录恢复日志
```

---

## 4. 超时处理策略

### 4.1 Agent超时

```markdown
#### Agent超时处理

**检测条件**：
- Agent启动后超过60秒无响应
- Agent执行超过300秒未完成

**处理流程**：
1. 记录超时日志
2. 尝试恢复会话（最多3次）
3. 如果恢复成功，继续执行
4. 如果恢复失败，跳过当前任务
5. 继续执行下一个任务

**日志格式**：
```
- {yymmdd hhmm} ⚠️ Agent超时：{agent_type}（{agent_id}），超时 {duration}秒
- {yymmdd hhmm} 尝试恢复会话：{agent_id}，尝试 {attempt}/3
- {yymmdd hhmm} 恢复成功/失败：{agent_id}
```
```

### 4.2 批次超时

```markdown
#### 批次超时处理

**检测条件**：
- 批次执行超过1800秒（30分钟）

**处理流程**：
1. 记录批次超时日志
2. 保存当前状态到检查点
3. 跳过当前批次
4. 继续执行下一个批次
5. 在日志中标记跳过的批次

**日志格式**：
```
- {yymmdd hhmm} ⚠️ 批次超时：批次 {batch_number}，超时 {duration}秒
- {yymmdd hhmm} 跳过批次：{batch_number}
- {yymmdd hhmm} 继续执行：批次 {batch_number + 1}
```
```

### 4.3 阶段超时

```markdown
#### 阶段超时处理

**检测条件**：
- 阶段执行超过7200秒（2小时）

**处理流程**：
1. 记录阶段超时日志
2. 保存当前状态到检查点
3. 暂停执行
4. 通知用户
5. 等待用户干预

**日志格式**：
```
- {yymmdd hhmm} ⚠️ 阶段超时：{phase_name}，超时 {duration}秒
- {yymmdd hhmm} 暂停执行，等待用户干预
```
```

---

## 5. 恢复机制

### 5.1 会话恢复

```markdown
#### 会话恢复流程

**触发条件**：
- 会话无响应超过60秒
- 会话返回错误
- 会话超时

**恢复流程**：
1. 保存当前状态到检查点
2. 终止当前会话
3. 创建新会话
4. 从检查点恢复状态
5. 继续执行

**恢复时间目标**：< 2分钟
```

### 5.2 任务恢复

```markdown
#### 任务恢复流程

**触发条件**：
- 任务执行失败
- 任务超时
- 任务被跳过

**恢复流程**：
1. 记录任务失败日志
2. 更新检查点状态
3. 跳过失败任务
4. 继续执行下一个任务
5. 在日志中标记失败任务

**恢复时间目标**：< 30秒
```

### 5.3 批次恢复

```markdown
#### 批次恢复流程

**触发条件**：
- 批次执行失败
- 批次超时
- 批次被跳过

**恢复流程**：
1. 记录批次失败日志
2. 更新检查点状态
3. 跳过失败批次
4. 继续执行下一个批次
5. 在日志中标记失败批次

**恢复时间目标**：< 1分钟
```

---

## 6. 监控指标

### 6.1 关键指标

```json
{
  "key_metrics": {
    "agent_response_time": {
      "description": "Agent响应时间",
      "threshold": 60,
      "unit": "seconds"
    },
    "batch_duration": {
      "description": "批次执行时间",
      "threshold": 1800,
      "unit": "seconds"
    },
    "fix_rounds": {
      "description": "修正轮次",
      "threshold": 3,
      "unit": "count"
    },
    "error_rate": {
      "description": "错误率",
      "threshold": 10,
      "unit": "percent"
    },
    "timeout_rate": {
      "description": "超时率",
      "threshold": 5,
      "unit": "percent"
    },
    "recovery_success_rate": {
      "description": "恢复成功率",
      "threshold": 90,
      "unit": "percent"
    }
  }
}
```

### 6.2 告警规则

```json
{
  "alert_rules": [
    {
      "name": "agent_timeout",
      "condition": "agent_response_time > 300",
      "severity": "warning",
      "message": "Agent响应超时"
    },
    {
      "name": "batch_timeout",
      "condition": "batch_duration > 1800",
      "severity": "critical",
      "message": "批次执行超时"
    },
    {
      "name": "high_error_rate",
      "condition": "error_rate > 10",
      "severity": "critical",
      "message": "错误率过高"
    },
    {
      "name": "high_timeout_rate",
      "condition": "timeout_rate > 5",
      "severity": "warning",
      "message": "超时率过高"
    },
    {
      "name": "low_recovery_rate",
      "condition": "recovery_success_rate < 90",
      "severity": "warning",
      "message": "恢复成功率过低"
    }
  ]
}
```

---

## 7. 实现示例

### 7.1 超时检测器

```javascript
// 超时检测器
class TimeoutDetector {
  constructor(config = {}) {
    this.thresholds = config.thresholds || {
      agent_spawn: 60,
      agent_execution: 300,
      batch_execution: 1800
    };
    this.timers = new Map();
  }
  
  startTimer(id, type) {
    this.timers.set(id, {
      type,
      startTime: Date.now(),
      threshold: this.thresholds[type]
    });
  }
  
  checkTimeout(id) {
    const timer = this.timers.get(id);
    if (!timer) return null;
    
    const elapsed = (Date.now() - timer.startTime) / 1000;
    if (elapsed > timer.threshold) {
      return {
        id,
        type: timer.type,
        elapsed,
        threshold: timer.threshold
      };
    }
    
    return null;
  }
  
  stopTimer(id) {
    this.timers.delete(id);
  }
}
```

### 7.2 健康检查器

```javascript
// 健康检查器
class HealthChecker {
  constructor(config = {}) {
    this.checks = config.checks || {};
    this.interval = config.interval || 30000;
  }
  
  async check() {
    const results = {};
    
    for (const [name, check] of Object.entries(this.checks)) {
      try {
        const result = await this.executeCheck(check);
        results[name] = {
          status: result ? 'healthy' : 'unhealthy',
          timestamp: Date.now()
        };
      } catch (error) {
        results[name] = {
          status: 'unhealthy',
          error: error.message,
          timestamp: Date.now()
        };
      }
    }
    
    return this.evaluateHealth(results);
  }
  
  async executeCheck(check) {
    // 执行具体的健康检查
    return true;
  }
  
  evaluateHealth(results) {
    const unhealthyCount = Object.values(results)
      .filter(r => r.status === 'unhealthy').length;
    
    if (unhealthyCount === 0) return 'healthy';
    if (unhealthyCount <= 1) return 'degraded';
    return 'unhealthy';
  }
}
```

### 7.3 恢复管理器

```javascript
// 恢复管理器
class RecoveryManager {
  constructor(config = {}) {
    this.maxRetries = config.maxRetries || 3;
    this.retryDelay = config.retryDelay || 5000;
  }
  
  async recover(context) {
    const { type, id, error } = context;
    
    console.log(`Attempting recovery for ${type} ${id}`);
    
    for (let attempt = 1; attempt <= this.maxRetries; attempt++) {
      try {
        console.log(`Recovery attempt ${attempt}/${this.maxRetries}`);
        
        // 保存当前状态
        await this.saveState(context);
        
        // 执行恢复
        const result = await this.executeRecovery(context);
        
        if (result.success) {
          console.log(`Recovery successful for ${type} ${id}`);
          return { success: true, attempt };
        }
        
        // 等待后重试
        await this.delay(this.retryDelay * attempt);
      } catch (error) {
        console.error(`Recovery attempt ${attempt} failed:`, error);
      }
    }
    
    console.log(`Recovery failed for ${type} ${id} after ${this.maxRetries} attempts`);
    return { success: false, attempts: this.maxRetries };
  }
  
  async saveState(context) {
    // 保存状态到检查点
  }
  
  async executeRecovery(context) {
    // 执行具体的恢复操作
    return { success: true };
  }
  
  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

---

## 8. 集成到主代理

### 8.1 主代理提示词更新

在主代理提示词中添加超时检测和恢复机制：

```markdown
#### 超时检测与恢复机制

**超时阈值**：
- Agent启动：60秒
- Agent执行：300秒
- 批次执行：1800秒
- 阶段执行：7200秒

**检测频率**：
- 健康检查：每30秒
- 超时检查：每60秒
- 进度检查：每120秒

**恢复策略**：
- 可恢复错误：自动重试（最多3次）
- 不可恢复错误：记录并跳过
- 平台错误：等待或切换

**恢复流程**：
1. 检测问题
2. 保存状态到检查点
3. 终止卡住的Agent
4. 创建新会话
5. 从检查点恢复
6. 继续执行

**日志记录**：
- 记录所有超时事件
- 记录恢复尝试
- 记录恢复结果
```

### 8.2 主代理执行流程

```markdown
### 执行流程（带超时检测）

1. **初始化**
   - 启动超时检测器
   - 启动健康检查器
   - 启动恢复管理器

2. **执行任务**
   - 启动Agent
   - 记录启动时间
   - 监控执行状态

3. **超时检测**
   - 定期检查超时
   - 检查健康状态
   - 检查进度

4. **超时处理**
   - 检测到超时
   - 记录超时日志
   - 尝试恢复
   - 如果恢复失败，跳过任务

5. **继续执行**
   - 更新检查点
   - 继续下一个任务
   - 记录执行日志
```

---

## 9. 最佳实践

### 9.1 超时设置

1. **合理设置阈值**：根据任务复杂度设置合理的超时阈值
2. **分级设置**：不同任务类型设置不同的超时阈值
3. **动态调整**：根据历史数据动态调整超时阈值
4. **监控告警**：超时时及时告警

### 9.2 健康检查

1. **定期检查**：定期执行健康检查
2. **全面检查**：检查所有关键组件
3. **及时响应**：发现问题及时处理
4. **记录日志**：记录检查结果

### 9.3 恢复策略

1. **快速恢复**：尽快恢复系统运行
2. **最小影响**：恢复过程对系统影响最小
3. **完整记录**：记录恢复过程和结果
4. **持续改进**：根据恢复经验改进系统

### 9.4 监控告警

1. **关键指标**：监控关键性能指标
2. **合理阈值**：设置合理的告警阈值
3. **及时响应**：告警时及时响应
4. **分析根因**：分析告警根本原因
