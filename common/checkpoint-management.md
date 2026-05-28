# 检查点管理

## 检查点文件

`{PROJECT_ROOT}/outputs/checkpoint.json`

## 检查点结构

```json
{
  "version": "1.0",
  "phase": "batch_dev",
  "lastUpdated": "yymmdd hhmm",
  "currentBatch": 3,
  "totalBatches": 7,
  "completedModules": ["module1", "module2"],
  "pendingModules": ["module3", "module4"],
  "activeSessions": {
    "dev": {"id": "abc123", "skill": "dev_skill", "createdAt": "yymmdd hhmm", "status": "active"},
    "test": {"id": "def456", "skill": "test_skill", "createdAt": "yymmdd hhmm", "status": "active"}
  },
  "currentFixRound": 0,
  "metrics": {
    "totalAgentCalls": 45,
    "startTime": "yymmdd hhmm",
    "batchDurations": [12, 15, 18]
  }
}
```

## 检查点更新时机

1. 每批次开始前：更新 currentBatch 和 pendingModules
2. 每批次完成后：更新 completedModules 和 metrics
3. Agent会话创建后：更新 activeSessions
4. Agent会话结束后：清除对应session记录

## 检查点恢复流程

1. 读取 checkpoint.json
2. 验证 activeSessions 中的会话是否仍有效
3. 无效会话：从 dev-plan.md 重新读取状态
4. 有效会话：直接 resume
