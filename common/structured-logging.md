# 结构化日志系统

## 双轨日志

同时维护两种日志格式：

### 人类可读日志（main-log.md）

- 格式：`- {yymmdd hhmm} {事件描述}`
- 用途：快速浏览、人工审查

### 机器可读日志（events.jsonl）

- 格式：每行一个JSON对象
- 用途：程序解析、状态恢复、统计分析

## events.jsonl 事件类型

```json
// 批次开始
{"ts":"yymmdd hhmm","event":"batch_start","batch":3,"modules":["module1","module2"]}

// Agent启动
{"ts":"yymmdd hhmm","event":"agent_spawn","type":"agent_type","id":"abc123","batch":3}

// Agent完成
{"ts":"yymmdd hhmm","event":"agent_complete","type":"agent_type","id":"abc123","duration_sec":420}

// 测试结果
{"ts":"yymmdd hhmm","event":"test_result","module":"module1","dimension":"component","verdict":"PASS","warnings":0}

// 修正循环
{"ts":"yymmdd hhmm","event":"fix_round","batch":3,"round":1,"modules":["module1"],"issues":["issue1"]}

// 批次完成
{"ts":"yymmdd hhmm","event":"batch_complete","batch":3,"duration_min":15,"agent_calls":4,"pass_rate":0.67}

// 检查点更新
{"ts":"yymmdd hhmm","event":"checkpoint_update","batch":4,"completed":["module1"],"pending":["module2"]}

// 会话重建
{"ts":"yymmdd hhmm","event":"session_refresh","type":"agent_type","old_id":"abc123","new_id":"xyz789","reason":"expired"}

// 阶段完成
{"ts":"yymmdd hhmm","event":"phase_complete","phase":"frontend","total_batches":7,"total_modules":20,"duration_min":180}
```
