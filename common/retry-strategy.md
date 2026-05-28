# 智能重试策略

## 分级处理策略

| 问题级别 | 处理方式 | 降级条件 |
|---------|---------|---------|
| blocker | 必须修复，不允许降级 | 3轮后暂停，请求人工介入 |
| major | 必须修复，允许降级 | 3轮后降级，记录待跟进 |
| minor | 记录技术债务 | 不阻塞，直接跳过 |

## blocker级问题处理

1. 第3轮仍有blocker → 暂停自动流程
2. 生成详细的问题报告：
   - 问题描述
   - 已尝试的修复方案
   - 相关代码位置
   - 建议的人工处理方向
3. 写入 `{PROJECT_ROOT}/outputs/needs-human-review.md`
4. 向用户报告，等待人工决策

## major级问题处理

1. 第3轮仍有major → 自动降级为⚠️
2. 记录到 `{PROJECT_ROOT}/outputs/technical-debt.md`
3. 格式：
   ```markdown
   - [MAJOR] {模块名} - {问题描述}
     - 发现时间：{yymmdd hhmm}
     - 测试维度：{dimension}
     - 影响范围：{描述}
     - 建议修复：{建议}
   ```
