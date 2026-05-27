# 卡住问题诊断与优化

本文档帮助诊断和解决系统卡住问题。

---

## 🔍 卡住症状诊断

### 症状1：主代理停在等待子代理

**表现**：
- 主代理显示"等待子代理完成"
- 长时间无响应
- 没有新的日志输出

**可能原因**：
1. 子代理执行时间过长（超过300秒）
2. 子代理会话已过期
3. 子代理返回格式不正确
4. API配额耗尽

**诊断命令**：
```bash
# 检查最近的Agent调用
tail -20 {PROJECT_ROOT}/outputs/events.jsonl

# 检查是否有超时记录
grep "timeout" {PROJECT_ROOT}/outputs/events.jsonl

# 检查Agent注册表
ls -la {PROJECT_ROOT}/outputs/agent-registry/
```

**解决方案**：
```bash
# 方案1：等待更长时间（最多7分钟）
# 系统会自动超时并继续

# 方案2：手动跳过当前批次
# 编辑checkpoint.json，增加currentBatch

# 方案3：重启代理
# 从检查点恢复
```

---

### 症状2：子代理无响应

**表现**：
- 子代理启动后无输出
- 没有错误信息
- 任务没有进展

**可能原因**：
1. 任务过于复杂
2. 文件读取问题
3. 上下文溢出
4. 资源不足

**诊断命令**：
```bash
# 检查子代理是否启动
grep "agent_spawn" {PROJECT_ROOT}/outputs/events.jsonl | tail -5

# 检查是否有错误
grep "error" {PROJECT_ROOT}/outputs/events.jsonl | tail -10

# 检查文件是否存在
ls -la {PROJECT_ROOT}/outputs/
```

**解决方案**：
```bash
# 方案1：减小任务规模
# 将BATCH_SIZE从3改为1

# 方案2：检查文件路径
# 确认所有输入文件存在

# 方案3：重启子代理
# 系统会自动超时并重启
```

---

### 症状3：修正循环卡住

**表现**：
- 修正循环一直在进行
- 测试一直失败
- 没有进展

**可能原因**：
1. 测试用例有问题
2. 修复方案不正确
3. 死循环

**诊断命令**：
```bash
# 检查修正轮次
grep "fix_round" {PROJECT_ROOT}/outputs/events.jsonl | tail -10

# 检查测试结果
grep "test_result" {PROJECT_ROOT}/outputs/events.jsonl | tail -10

# 检查降级记录
grep "degraded" {PROJECT_ROOT}/outputs/events.jsonl | tail -5
```

**解决方案**：
```bash
# 方案1：等待3轮自动降级
# 系统会自动降级并继续

# 方案2：手动跳过修正
# 编辑checkpoint.json，标记任务完成

# 方案3：检查测试报告
# 查看具体失败原因
```

---

## 🛠️ 优化方案

### 1. 超时检测机制

在主代理提示词中添加：

```markdown
#### 超时检测机制

**检测频率**：每60秒检查一次

**超时阈值**：
- 子代理：300秒（5分钟）
- 主代理：600秒（10分钟）
- 批次：1800秒（30分钟）

**超时处理流程**：
1. 检测到超时
2. 记录超时日志
3. 尝试恢复会话
4. 如果恢复失败，跳过当前任务
5. 继续执行下一个任务

**日志格式**：
```
- {yymmdd hhmm} ⚠️ Agent超时：{agent_type}（{agent_id}），超时 {duration}秒
- {yymmdd hhmm} 尝试恢复会话：{agent_id}
- {yymmdd hhmm} 恢复成功/失败：{agent_id}
```
```

---

### 2. 健康检查机制

```markdown
#### 健康检查机制

**检查频率**：每30秒

**检查项目**：
1. Agent是否响应
2. 任务是否在推进
3. 资源是否充足
4. 网络是否正常

**健康状态**：
- healthy：正常运行
- degraded：部分功能受影响
- unhealthy：无法正常运行

**不健康处理**：
1. 记录不健康状态
2. 尝试自动修复
3. 如果修复失败，暂停执行
4. 通知用户
```

---

### 3. 快速失败机制

```markdown
#### 快速失败机制

**检测点**：
- Agent启动前
- 任务执行中
- 任务完成后

**失败类型**：
- 可恢复：自动重试（最多3次）
- 不可恢复：记录并跳过
- 平台错误：等待或切换

**处理时间目标**：
- 检测：< 1秒
- 决策：< 5秒
- 恢复：< 30秒
```

---

### 4. 部分恢复机制

```markdown
#### 部分恢复机制

**恢复粒度**：
- 批次级别：跳过整个批次
- 任务级别：跳过单个任务
- Agent级别：重启单个Agent

**恢复流程**：
1. 检测失败点
2. 保存当前状态到检查点
3. 跳过失败部分
4. 继续执行剩余部分
5. 记录失败日志

**恢复时间目标**：
- 批次恢复：< 1分钟
- 任务恢复：< 30秒
- Agent恢复：< 2分钟
```

---

## 📊 监控指标

### 关键指标

```json
{
  "metrics": {
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
    }
  }
}
```

### 告警规则

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
      "name": "high_fix_rounds",
      "condition": "fix_rounds > 3",
      "severity": "warning",
      "message": "修正轮次过多"
    }
  ]
}
```

---

## 🔧 诊断命令

### 检查系统状态

```bash
# 1. 检查最新日志
tail -30 {PROJECT_ROOT}/outputs/main-log.md

# 2. 检查最近事件
tail -30 {PROJECT_ROOT}/outputs/events.jsonl

# 3. 检查检查点
cat {PROJECT_ROOT}/outputs/checkpoint.json | python -m json.tool

# 4. 检查告警
cat {PROJECT_ROOT}/outputs/alerts.jsonl

# 5. 检查Agent注册表
ls -la {PROJECT_ROOT}/outputs/agent-registry/
```

### 检查Agent状态

```bash
# 检查活跃Agent
grep "agent_spawn" {PROJECT_ROOT}/outputs/events.jsonl | tail -5

# 检查完成的Agent
grep "agent_complete" {PROJECT_ROOT}/outputs/events.jsonl | tail -5

# 检查超时的Agent
grep "timeout" {PROJECT_ROOT}/outputs/events.jsonl | tail -5

# 检查失败的Agent
grep "error" {PROJECT_ROOT}/outputs/events.jsonl | tail -10
```

### 检查任务进度

```bash
# 检查批次进度
grep "batch_" {PROJECT_ROOT}/outputs/events.jsonl | tail -10

# 检查修正轮次
grep "fix_round" {PROJECT_ROOT}/outputs/events.jsonl | tail -5

# 检查测试结果
grep "test_result" {PROJECT_ROOT}/outputs/events.jsonl | tail -10

# 检查降级记录
grep "degraded" {PROJECT_ROOT}/outputs/events.jsonl | tail -5
```

### 检查性能指标

```bash
# 检查响应时间
grep "agent_complete" {PROJECT_ROOT}/outputs/events.jsonl | tail -10

# 检查批次时长
grep "batch_complete" {PROJECT_ROOT}/outputs/events.jsonl | tail -5

# 检查错误率
grep -c "error" {PROJECT_ROOT}/outputs/events.jsonl
grep -c "agent_spawn" {PROJECT_ROOT}/outputs/events.jsonl
```

---

## 🚨 紧急恢复方案

### 方案1：从检查点恢复

```bash
# 1. 检查检查点文件
cat {PROJECT_ROOT}/outputs/checkpoint.json

# 2. 确认检查点有效
python -m json.tool {PROJECT_ROOT}/outputs/checkpoint.json

# 3. 重启OpenCode
# Ctrl+C 退出，然后重新运行 opencode

# 4. 选择对应的主代理
# 系统会自动从检查点恢复
```

### 方案2：手动跳过卡住的任务

```bash
# 1. 编辑检查点文件
vi {PROJECT_ROOT}/outputs/checkpoint.json

# 2. 修改当前批次号
# 将 currentBatch 增加1

# 3. 更新任务状态
# 将卡住的任务从 pending 移到 completed

# 4. 重启代理
```

### 方案3：重置状态重新开始

```bash
# 1. 备份当前输出
cp -r {PROJECT_ROOT}/outputs {PROJECT_ROOT}/outputs_backup

# 2. 删除检查点
rm {PROJECT_ROOT}/outputs/checkpoint.json

# 3. 删除Agent注册表
rm -rf {PROJECT_ROOT}/outputs/agent-registry/

# 4. 重新启动代理
# 系统会从头开始
```

### 方案4：清理上下文重新开始

```bash
# 1. 完全退出OpenCode
# Ctrl+C 或关闭终端

# 2. 重新启动OpenCode
opencode

# 3. 选择主代理
# 系统会创建新的会话

# 4. 从检查点恢复（如果有）
# 或从头开始
```

---

## 📋 预防措施

### 1. 控制任务规模

```markdown
**建议规模**：
- 小型项目：5-10个模块
- 中型项目：10-20个模块
- 大型项目：20-30个模块（分阶段）

**批次大小**：
- 简单模块：BATCH_SIZE=3
- 复杂模块：BATCH_SIZE=1
- 混合模块：BATCH_SIZE=2
```

### 2. 定期保存检查点

```markdown
**保存频率**：
- 每批次完成后
- 每个Phase完成后
- 检测到错误时
- 手动保存（可选）
```

### 3. 监控资源使用

```markdown
**监控项目**：
- API调用次数
- 上下文大小
- 磁盘空间
- 内存使用
```

### 4. 定期清理

```markdown
**清理项目**：
- 过期的检查点
- 旧的日志文件
- 临时文件
- 备份文件
```

---

## 🎯 最佳实践

### 1. 启动前检查

```bash
# 检查磁盘空间
df -h

# 检查网络连接
ping api.openai.com

# 检查项目目录
ls -la {PROJECT_ROOT}

# 检查依赖文件
ls -la {REQUIREMENT_FILE}
ls -la {TECH_STACK_FILE}
```

### 2. 运行中监控

```bash
# 定期检查日志
tail -f {PROJECT_ROOT}/outputs/main-log.md

# 监控事件流
tail -f {PROJECT_ROOT}/outputs/events.jsonl

# 检查告警
watch -n 30 'cat {PROJECT_ROOT}/outputs/alerts.jsonl | tail -5'
```

### 3. 异常处理

```markdown
**发现异常时**：
1. 立即检查日志
2. 诊断问题原因
3. 尝试自动恢复
4. 如果失败，手动干预
5. 记录问题和解决方案
```

### 4. 持续优化

```markdown
**优化方向**：
1. 减少超时概率
2. 提高恢复速度
3. 优化上下文使用
4. 改进错误处理
```
