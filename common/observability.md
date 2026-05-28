# 可观测性增强

## 可观测性组件

1. `status-tracker.json`：实时状态追踪
2. `metrics.json`：性能指标收集
3. `alerts.jsonl`：异常检测和告警

## 主Agent职责

1. **初始化可观测性组件**：首次启动时创建 status-tracker.json、metrics.json、alerts.jsonl
2. **更新状态追踪**：每批次开始/结束时更新 status-tracker.json
3. **收集性能指标**：每批次完成后更新 metrics.json
4. **检测异常**：每批次开始前检查异常规则，发现异常时生成告警
5. **展示可观测性数据**：在仪表盘中展示状态、指标、告警

## 告警处理流程

1. 检测异常
2. 生成告警（写入 alerts.jsonl）
3. 执行动作
4. 记录结果
5. 展示在仪表盘
