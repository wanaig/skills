# 项目仪表盘模板

> 最后更新：{yymmdd hhmm}

## 📊 总体进度

| 阶段 | 状态 | 进度 | 预计剩余 |
|------|------|------|---------|
| 架构设计 | {status} | {progress}% | {remaining} |
| 前端开发 | {status} | {progress}% | {remaining} |
| 后端开发 | {status} | {progress}% | {remaining} |
| 联调 | {status} | {progress}% | {remaining} |
| 部署 | {status} | {progress}% | {remaining} |

**状态说明**：
- ⏳ 等待
- 🔄 进行中
- ✅ 完成
- ⚠️ 降级通过
- ❌ 失败

---

## 📈 质量指标

| 指标 | 值 | 趋势 | 说明 |
|------|-----|------|------|
| 首次通过率 | {pass_rate}% | {trend} | 无需修正的模块占比 |
| 平均修正轮次 | {avg_fix_rounds} | {trend} | 越低越好 |
| 降级模块数 | {degraded_count} | - | 3轮修正后降级的模块 |
| blocker问题 | {blocker_count} | - | 需人工介入的问题 |
| 总Agent调用 | {total_calls} | - | 开发+测试+修正 |

---

## 🔄 当前批次状态

**批次 {current_batch}/{total_batches}**：{batch_items}

| 模块 | 开发 | 测试1 | 测试2 | 测试3 | 状态 |
|------|------|-------|-------|-------|------|
| {module1} | {dev_status} | {test1_status} | {test2_status} | {test3_status} | {overall_status} |
| {module2} | {dev_status} | {test1_status} | {test2_status} | {test3_status} | {overall_status} |
| {module3} | {dev_status} | {test1_status} | {test2_status} | {test3_status} | {overall_status} |

**状态图标**：
- ⏳ 待开始
- 🔄 进行中
- ✅ 通过
- ❌ 失败
- ⚠️ 降级通过

---

## 📝 最近事件

<!-- 从 events.jsonl 自动提取最近10条事件 -->

- {timestamp} {event_description}
- {timestamp} {event_description}
- {timestamp} {event_description}
- {timestamp} {event_description}
- {timestamp} {event_description}

---

## ⚠️ 待处理事项

### 需人工介入（blocker）
<!-- 从 needs-human-review.md 提取 -->
- [ ] {issue_description} - {discovery_time}

### 技术债务（major）
<!-- 从 technical-debt.md 提取 -->
- [ ] {issue_description} - {discovery_time} - {impact}

### 警告（minor）
- {warning_description}

---

## 🔗 快速链接

- [检查点文件](./checkpoint.json)
- [完整日志](./main-log.md)
- [结构化日志](./events.jsonl)
- [技术债务](./technical-debt.md)
- [人工审核](./needs-human-review.md)
- [上下文摘要](./context-summary.md)

---

## 📊 批次历史

| 批次 | 模块数 | 耗时 | Agent调用 | 通过率 | 修正轮次 |
|------|--------|------|-----------|--------|----------|
| 1 | {count} | {duration}min | {calls} | {pass_rate}% | {fix_rounds} |
| 2 | {count} | {duration}min | {calls} | {pass_rate}% | {fix_rounds} |
| 3 | {count} | {duration}min | {calls} | {pass_rate}% | {fix_rounds} |

---

## 🎯 预测

- **预计完成时间**：{estimated_completion}
- **预计总Agent调用**：{estimated_calls}
- **预计通过率**：{estimated_pass_rate}%

---

## 📊 可观测性数据

### 实时状态

**系统状态**：{system_status}
**当前阶段**：{current_phase}
**当前批次**：{current_batch}/{total_batches}
**活跃Agent数**：{active_agents_count}
**待处理任务数**：{pending_tasks_count}

### 性能指标

| 指标 | 值 | 说明 |
|------|-----|------|
| 任务完成速率 | {tasks_per_hour} 个/小时 | 每小时完成的任务数 |
| 批次完成速率 | {batches_per_hour} 个/小时 | 每小时完成的批次数 |
| 修正率 | {fix_rate}% | 需要修正的任务占比 |
| 超时率 | {timeout_rate}% | 超时的Agent调用占比 |
| 平均批次时长 | {avg_batch_duration} 分钟 | 批次平均耗时 |
| 平均Agent调用时长 | {avg_agent_duration} 秒 | Agent调用平均耗时 |

### 异常告警

<!-- 从 alerts.jsonl 提取未解决的告警 -->

| 时间 | 类型 | 严重程度 | 描述 | 状态 |
|------|------|---------|------|------|
| {timestamp} | {type} | {severity} | {message} | {resolved} |

**告警统计**：
- 未解决告警数：{unresolved_alerts_count}
- 严重告警数：{critical_alerts_count}
- 警告告警数：{warning_alerts_count}

### 知识库统计

| 指标 | 值 | 说明 |
|------|-----|------|
| 已知问题模式数 | {patterns_count} | 已识别的问题模式 |
| 反模式数 | {antipatterns_count} | 已识别的反模式 |
| 修复策略数 | {fix_strategies_count} | 已验证的修复策略 |
| 最常见问题类型 | {most_common_category} | 出现最多的问题类型 |

---

## 📋 使用说明

此仪表盘由主Agent在以下时机自动更新：
1. 每批次开始前
2. 每批次完成后
3. 每个Phase完成后
4. 检测到重大状态变化时

数据来源：
- `checkpoint.json`：进度和会话状态
- `events.jsonl`：事件历史和统计
- `dev-plan.md` / `integration-plan.md`：任务列表
- `test-report.json`：测试结果
- `technical-debt.md`：技术债务
- `needs-human-review.md`：人工审核事项
- `status-tracker.json`：实时状态追踪
- `metrics.json`：性能指标
- `alerts.jsonl`：异常告警
- `knowledge-base.json`：经验知识图谱
