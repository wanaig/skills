# 知识库和可观测性组件初始化脚本

本文档提供了初始化知识库和可观测性组件的脚本和说明。

## 1. 初始化脚本

### 1.1 创建知识库文件

**文件路径**：`{PROJECT_ROOT}/outputs/knowledge-base.json`

**初始内容**：
```json
{
  "version": "1.0",
  "lastUpdated": "yymmdd hhmm",
  "patterns": [],
  "antiPatterns": [],
  "fixStrategies": [],
  "statistics": {
    "totalPatterns": 0,
    "totalAntiPatterns": 0,
    "totalFixStrategies": 0,
    "mostCommonCategory": "",
    "averageConfidence": 0
  }
}
```

### 1.2 创建状态追踪文件

**文件路径**：`{PROJECT_ROOT}/outputs/status-tracker.json`

**初始内容**：
```json
{
  "version": "1.0",
  "lastUpdated": "yymmdd hhmm",
  "systemStatus": "running",
  "currentPhase": "architecture",
  "currentBatch": 0,
  "totalBatches": 0,
  "activeAgents": [],
  "completedTasks": [],
  "pendingTasks": []
}
```

### 1.3 创建性能指标文件

**文件路径**：`{PROJECT_ROOT}/outputs/metrics.json`

**初始内容**：
```json
{
  "version": "1.0",
  "lastUpdated": "yymmdd hhmm",
  "counters": {
    "totalAgentCalls": 0,
    "totalBatchesCompleted": 0,
    "totalTasksCompleted": 0,
    "totalFixRounds": 0,
    "totalTimeouts": 0,
    "totalSessionRefreshes": 0
  },
  "gauges": {
    "currentActiveAgents": 0,
    "currentPendingTasks": 0,
    "contextUsagePercent": 0,
    "sessionAgeMinutes": 0
  },
  "histograms": {
    "batchDurationMinutes": {
      "min": 0,
      "max": 0,
      "avg": 0,
      "p50": 0,
      "p95": 0,
      "samples": []
    },
    "agentCallDurationSeconds": {
      "min": 0,
      "max": 0,
      "avg": 0,
      "p50": 0,
      "p95": 0,
      "samples": []
    },
    "fixRoundsPerTask": {
      "min": 0,
      "max": 0,
      "avg": 0,
      "p50": 0,
      "p95": 0,
      "samples": []
    }
  },
  "rates": {
    "tasksPerHour": 0,
    "batchesPerHour": 0,
    "fixRate": 0,
    "timeoutRate": 0
  }
}
```

### 1.4 创建告警文件

**文件路径**：`{PROJECT_ROOT}/outputs/alerts.jsonl`

**初始内容**：空文件

---

## 2. 初始化流程

### 2.1 主Agent初始化流程

在主Agent启动时，执行以下步骤：

1. **检查输出目录**：确认 `{PROJECT_ROOT}/outputs/` 目录存在
2. **检查知识库文件**：如果 `knowledge-base.json` 不存在，则创建初始文件
3. **检查状态追踪文件**：如果 `status-tracker.json` 不存在，则创建初始文件
4. **检查性能指标文件**：如果 `metrics.json` 不存在，则创建初始文件
5. **检查告警文件**：如果 `alerts.jsonl` 不存在，则创建空文件

### 2.2 初始化代码示例

```markdown
### 初始化可观测性组件

1. 检查 `{PROJECT_ROOT}/outputs/` 目录是否存在，如不存在则创建
2. 检查 `{PROJECT_ROOT}/outputs/knowledge-base.json` 是否存在，如不存在则创建初始文件
3. 检查 `{PROJECT_ROOT}/outputs/status-tracker.json` 是否存在，如不存在则创建初始文件
4. 检查 `{PROJECT_ROOT}/outputs/metrics.json` 是否存在，如不存在则创建初始文件
5. 检查 `{PROJECT_ROOT}/outputs/alerts.jsonl` 是否存在，如不存在则创建空文件
```

---

## 3. 知识库结构说明

### 3.1 Patterns（问题模式）

**用途**：记录已知的问题模式和解决方案

**结构**：
```json
{
  "id": "PAT-001",
  "category": "vue_component",
  "subcategory": "props_validation",
  "problem": "Props缺少类型验证导致运行时错误",
  "solution": "使用TypeScript接口定义Props类型",
  "confidence": 0.95,
  "occurrences": 5,
  "firstSeen": "batch_2_fix_1",
  "lastSeen": "batch_5"
}
```

**字段说明**：
- `id`：唯一标识符
- `category`：问题类别（如 vue_component、api_design、state_management）
- `subcategory`：问题子类别
- `problem`：问题描述
- `solution`：解决方案
- `confidence`：置信度（0-1）
- `occurrences`：出现次数
- `firstSeen`：首次出现时间
- `lastSeen`：最后出现时间

### 3.2 AntiPatterns（反模式）

**用途**：记录应避免的反模式

**结构**：
```json
{
  "id": "ANTI-001",
  "category": "vue_component",
  "pattern": "在模板中使用复杂表达式",
  "impact": "major",
  "detectedBy": "dg_vue_tester_component",
  "fixSuggestion": "提取为computed属性"
}
```

**字段说明**：
- `id`：唯一标识符
- `category`：反模式类别
- `pattern`：反模式描述
- `impact`：影响程度（critical/major/minor）
- `detectedBy`：检测者（测试Agent类型）
- `fixSuggestion`：修复建议

### 3.3 FixStrategies（修复策略）

**用途**：记录已验证的修复策略

**结构**：
```json
{
  "problemType": "type_mismatch",
  "successfulFixes": 8,
  "avgRounds": 1.5,
  "bestApproach": "先修复类型定义，再修复实现"
}
```

**字段说明**：
- `problemType`：问题类型
- `successfulFixes`：成功修复次数
- `avgRounds`：平均修正轮次
- `bestApproach`：最佳修复方法

---

## 4. 可观测性组件说明

### 4.1 StatusTracker（状态追踪器）

**用途**：实时追踪系统状态

**更新时机**：
1. 系统启动时：初始化状态
2. 阶段切换时：更新 currentPhase
3. 批次开始时：更新 currentBatch 和 activeAgents
4. 任务完成时：更新 completedTasks 和 pendingTasks
5. Agent 状态变化时：更新 activeAgents
6. 系统暂停/恢复时：更新 systemStatus

### 4.2 Metrics（性能指标）

**用途**：收集和统计性能指标

**指标类型**：
1. **计数器（Counters）**：单调递增的指标
   - totalAgentCalls：总Agent调用次数
   - totalBatchesCompleted：总完成批次数
   - totalTasksCompleted：总完成任务数
   - totalFixRounds：总修正轮次
   - totalTimeouts：总超时次数
   - totalSessionRefreshes：总会话刷新次数

2. **仪表（Gauges）**：可增可减的指标
   - currentActiveAgents：当前活跃Agent数
   - currentPendingTasks：当前待处理任务数
   - contextUsagePercent：上下文使用率
   - sessionAgeMinutes：会话存活时间

3. **直方图（Histograms）**：统计分布
   - batchDurationMinutes：批次时长分布
   - agentCallDurationSeconds：Agent调用时长分布
   - fixRoundsPerTask：每任务修正轮次分布

4. **速率（Rates）**：单位时间内的指标
   - tasksPerHour：每小时完成任务数
   - batchesPerHour：每小时完成批次数
   - fixRate：修正率
   - timeoutRate：超时率

### 4.3 Alerts（告警管理器）

**用途**：异常检测和告警

**告警类型**：
1. **timeout**：Agent超时
2. **high_fix_rate**：高修正率
3. **context_overflow**：上下文溢出
4. **session_expired**：会话过期
5. **system_stall**：系统停滞

**严重程度**：
1. **warning**：警告，不影响主流程
2. **critical**：严重，可能影响主流程

---

## 5. 使用示例

### 5.1 添加新模式到知识库

```json
{
  "id": "PAT-002",
  "category": "api_design",
  "subcategory": "input_validation",
  "problem": "缺少输入参数验证导致运行时错误",
  "solution": "使用DTO+装饰器进行参数验证",
  "confidence": 0.9,
  "occurrences": 3,
  "firstSeen": "batch_3_fix_2",
  "lastSeen": "batch_5"
}
```

### 5.2 记录新告警

```json
{
  "timestamp": "260527 1430",
  "type": "timeout",
  "severity": "warning",
  "message": "Agent dg_frontend_vue_dev 超时（350秒）",
  "context": {
    "agentId": "abc123",
    "agentType": "dg_frontend_vue_dev",
    "timeoutDuration": 350
  },
  "action": "继续等待",
  "resolved": false
}
```

### 5.3 更新性能指标

```json
{
  "counters": {
    "totalAgentCalls": 46,
    "totalBatchesCompleted": 4,
    "totalTasksCompleted": 15,
    "totalFixRounds": 6,
    "totalTimeouts": 1,
    "totalSessionRefreshes": 0
  },
  "rates": {
    "tasksPerHour": 4.5,
    "batchesPerHour": 1.8,
    "fixRate": 0.25,
    "timeoutRate": 0.02
  }
}
```

---

## 6. 最佳实践

1. **定期备份**：定期备份 knowledge-base.json 和 metrics.json
2. **清理旧数据**：定期清理超过30天的告警和事件
3. **监控文件大小**：确保文件不会过大影响性能
4. **验证数据完整性**：定期验证JSON文件格式正确
5. **及时更新**：确保知识库和指标及时更新
