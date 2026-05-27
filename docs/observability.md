# 可观测性增强机制

本文档定义了多智能体系统的可观测性增强机制，包括实时状态追踪、性能指标收集和异常检测。

## 1. 实时状态追踪

### 1.1 状态追踪文件

**文件路径**：`{PROJECT_ROOT}/outputs/status-tracker.json`

**状态结构**：
```json
{
  "version": "1.0",
  "lastUpdated": "yymmdd hhmm",
  "systemStatus": "running|paused|completed|failed",
  "currentPhase": "architecture|frontend|backend|flutter|blockchain|fullstack|deploy",
  "currentBatch": 3,
  "totalBatches": 7,
  "activeAgents": [
    {
      "id": "abc123",
      "type": "dg_frontend_vue_dev",
      "status": "active",
      "startedAt": "yymmdd hhmm",
      "currentTask": "开发用户列表模块"
    }
  ],
  "completedTasks": [
    {
      "task": "用户登录模块",
      "completedAt": "yymmdd hhmm",
      "durationMinutes": 15,
      "agentCalls": 4,
      "fixRounds": 1
    }
  ],
  "pendingTasks": [
    {
      "task": "用户列表模块",
      "priority": "high",
      "estimatedDuration": 20
    }
  ]
}
```

### 1.2 状态更新时机

1. **系统启动时**：初始化 status-tracker.json
2. **阶段切换时**：更新 currentPhase
3. **批次开始时**：更新 currentBatch 和 activeAgents
4. **任务完成时**：更新 completedTasks 和 pendingTasks
5. **Agent 状态变化时**：更新 activeAgents
6. **系统暂停/恢复时**：更新 systemStatus

---

## 2. 性能指标收集

### 2.1 性能指标文件

**文件路径**：`{PROJECT_ROOT}/outputs/metrics.json`

**指标结构**：
```json
{
  "version": "1.0",
  "lastUpdated": "yymmdd hhmm",
  "counters": {
    "totalAgentCalls": 45,
    "totalBatchesCompleted": 3,
    "totalTasksCompleted": 12,
    "totalFixRounds": 5,
    "totalTimeouts": 2,
    "totalSessionRefreshes": 1
  },
  "gauges": {
    "currentActiveAgents": 3,
    "currentPendingTasks": 8,
    "contextUsagePercent": 45,
    "sessionAgeMinutes": 60
  },
  "histograms": {
    "batchDurationMinutes": {
      "min": 10,
      "max": 25,
      "avg": 15,
      "p50": 14,
      "p95": 22,
      "samples": [12, 15, 18, 10, 25]
    },
    "agentCallDurationSeconds": {
      "min": 30,
      "max": 300,
      "avg": 120,
      "p50": 100,
      "p95": 250,
      "samples": [30, 60, 120, 180, 300]
    },
    "fixRoundsPerTask": {
      "min": 0,
      "max": 3,
      "avg": 1.2,
      "p50": 1,
      "p95": 3,
      "samples": [0, 1, 1, 2, 3]
    }
  },
  "rates": {
    "tasksPerHour": 4,
    "batchesPerHour": 1.5,
    "fixRate": 0.3,
    "timeoutRate": 0.04
  }
}
```

### 2.2 指标收集时机

1. **Agent 调用前后**：记录调用时长
2. **批次完成时**：更新批次时长和任务数
3. **修正循环时**：更新修正轮次
4. **超时发生时**：更新超时计数
5. **会话刷新时**：更新会话刷新计数
6. **每 10 分钟**：更新实时指标（活跃Agent数、待处理任务数等）

---

## 3. 异常检测和告警

### 3.1 异常检测规则

#### 基础异常检测

| 异常类型 | 检测条件 | 严重程度 | 处理方式 |
|---------|---------|---------|---------|
| Agent 超时 | 单次调用 > 300秒 | warning | 记录日志，继续等待 |
| 连续超时 | 同一Agent连续3次超时 | critical | 暂停该Agent，创建新会话 |
| 批次超时 | 单批次 > 60分钟 | warning | 记录日志，继续执行 |
| 高修正率 | 连续3个批次修正率 > 50% | warning | 记录日志，分析原因 |
| 上下文溢出 | 上下文使用率 > 90% | critical | 触发压缩，新建会话 |
| 会话过期 | 会话存活 > 2小时 | warning | 自动刷新会话 |
| 系统停滞 | 30分钟无进度更新 | critical | 检查系统状态，恢复执行 |

#### 高级异常检测

| 异常类型 | 检测条件 | 严重程度 | 处理方式 |
|---------|---------|---------|---------|
| 批次效率下降 | 连续3个批次时长 > 平均时长的2倍 | warning | 分析瓶颈，优化流程 |
| 修正轮次增加 | 连续3个批次平均修正轮次 > 2 | warning | 分析问题模式，优化prompt |
| Agent失败率高 | 单个Agent连续5次调用失败 | critical | 暂停该Agent，检查问题 |
| 资源竞争 | 同时活跃Agent数 > 平台限制 | warning | 降低并行度 |
| 知识库膨胀 | knowledge-base.json > 10MB | warning | 清理旧数据，归档历史 |
| 指标异常波动 | 任务完成速率变化 > 50% | warning | 分析原因，调整策略 |
| 告警积压 | 未解决告警数 > 10 | critical | 批量处理告警，恢复系统 |
| 数据不一致 | checkpoint.json 与 dev-plan.md 状态不匹配 | critical | 以 checkpoint.json 为准，恢复状态 |

#### 业务异常检测

| 异常类型 | 检测条件 | 严重程度 | 处理方式 |
|---------|---------|---------|---------|
| 模块依赖阻塞 | 模块A依赖模块B，但模块B未完成 | warning | 调整批次顺序 |
| 测试覆盖率下降 | 连续2个批次测试覆盖率 < 80% | warning | 增加测试用例 |
| 代码质量下降 | 连续2个批次代码审查问题 > 5 | warning | 加强代码审查 |
| 性能指标恶化 | 接口响应时间 > 500ms | warning | 优化性能 |
| 安全漏洞 | 检测到安全漏洞 | critical | 立即修复，暂停部署 |

#### 平台异常检测

| 异常类型 | 检测条件 | 严重程度 | 处理方式 |
|---------|---------|---------|---------|
| API配额耗尽 | API调用次数接近配额限制 | warning | 优化调用频率 |
| 存储空间不足 | 存储空间使用率 > 90% | critical | 清理临时文件，扩展存储 |
| 网络连接问题 | 网络延迟 > 5秒 | warning | 检查网络状态 |
| 平台服务异常 | 平台API返回错误 | critical | 等待平台恢复，记录日志 |

### 3.2 告警文件

**文件路径**：`{PROJECT_ROOT}/outputs/alerts.jsonl`

**告警格式**：
```json
{
  "timestamp": "yymmdd hhmm",
  "type": "timeout|high_fix_rate|context_overflow|session_expired|system_stall",
  "severity": "warning|critical",
  "message": "Agent dg_frontend_vue_dev 连续3次超时",
  "context": {
    "agentId": "abc123",
    "agentType": "dg_frontend_vue_dev",
    "timeoutCount": 3,
    "lastTimeoutDuration": 350
  },
  "action": "暂停该Agent，创建新会话",
  "resolved": false
}
```

### 3.3 告警处理流程

1. **检测异常**：根据规则检测异常
2. **生成告警**：写入 alerts.jsonl
3. **执行动作**：根据严重程度执行相应动作
4. **记录结果**：更新告警的 resolved 状态
5. **汇总报告**：在仪表盘中展示未解决的告警

---

## 4. 集成到主代理

### 4.1 主代理职责

1. **初始化可观测性组件**：
   - 创建 status-tracker.json
   - 创建 metrics.json
   - 创建 alerts.jsonl

2. **定期更新状态**：
   - 每批次开始/结束时更新状态
   - 每个关键步骤后更新指标

3. **检测异常**：
   - 每批次开始前检查异常规则
   - 发现异常时生成告警

4. **展示可观测性数据**：
   - 在仪表盘中展示状态、指标、告警
   - 提供快速链接到详细文件

### 4.2 子代理职责

1. **记录执行时长**：
   - 开始执行时记录开始时间
   - 完成执行时计算时长并记录

2. **报告异常**：
   - 遇到超时、错误等情况时报告
   - 提供详细的上下文信息

3. **更新指标**：
   - 完成任务后更新相关指标
   - 记录修正轮次等信息

---

## 5. 可观测性仪表盘集成

### 5.1 仪表盘数据源

仪表盘从以下文件读取数据：

1. `status-tracker.json`：实时状态
2. `metrics.json`：性能指标
3. `alerts.jsonl`：告警信息
4. `checkpoint.json`：检查点状态
5. `events.jsonl`：事件日志

### 5.2 仪表盘展示内容

1. **系统概览**：
   - 当前状态（运行中/暂停/完成/失败）
   - 当前阶段和批次
   - 活跃Agent数量
   - 待处理任务数量

2. **性能指标**：
   - 任务完成速率
   - 批次完成速率
   - 修正率
   - 超时率

3. **告警信息**：
   - 未解决的告警列表
   - 告警趋势图
   - 严重程度分布

4. **预测信息**：
   - 预计完成时间
   - 预计剩余任务数
   - 预计总Agent调用次数

---

## 6. 实现示例

### 6.1 主代理初始化代码

```markdown
### 初始化可观测性组件

1. 创建 status-tracker.json，初始化系统状态
2. 创建 metrics.json，初始化计数器和指标
3. 创建 alerts.jsonl，初始化告警文件
4. 在仪表盘中添加可观测性数据源链接
```

### 6.2 批次开始时更新

```markdown
### 批次开始时更新状态

1. 读取 status-tracker.json
2. 更新 currentBatch 和 activeAgents
3. 记录批次开始时间
4. 检查异常规则
5. 写回 status-tracker.json
```

### 6.3 批次完成时更新

```markdown
### 批次完成时更新状态

1. 读取 status-tracker.json 和 metrics.json
2. 更新 completedTasks 和 pendingTasks
3. 计算批次时长并更新指标
4. 更新速率指标
5. 检查异常规则
6. 写回文件
```

---

## 7. 最佳实践

1. **定期更新**：每批次至少更新一次状态和指标
2. **及时告警**：发现异常立即生成告警
3. **清理旧数据**：定期清理超过7天的告警和事件
4. **备份关键文件**：定期备份 checkpoint.json 和 metrics.json
5. **监控资源使用**：注意文件大小，避免过大影响性能
