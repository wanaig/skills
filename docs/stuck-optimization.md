# 卡住问题优化方案

本文档提供解决卡住问题的具体优化方案。

---

## 🔍 卡住原因分析

### 1. 主代理卡住原因

| 原因 | 症状 | 解决方案 |
|------|------|----------|
| 等待子代理超时 | 停在"等待子代理完成" | 添加超时检测 |
| 上下文溢出 | 响应变慢或停止 | 压缩上下文 |
| 检查点损坏 | 恢复后状态不正确 | 修复检查点 |
| 会话过期 | 无响应 | 刷新会话 |
| 错误处理失败 | 卡在错误状态 | 改进错误处理 |

### 2. 子代理卡住原因

| 原因 | 症状 | 解决方案 |
|------|------|----------|
| 任务过于复杂 | 长时间无响应 | 减小任务规模 |
| 文件读取问题 | 卡在读取阶段 | 检查文件路径 |
| 上下文溢出 | 响应变慢 | 新建会话 |
| 错误处理失败 | 卡在错误状态 | 跳过失败任务 |

---

## 🛠️ 优化方案

### 方案1：添加超时检测

在主代理提示词中添加超时检测机制：

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
3. 尝试恢复会话（最多3次）
4. 如果恢复失败，跳过当前任务
5. 继续执行下一个任务
```

### 方案2：添加健康检查

```markdown
#### 健康检查机制

**检查频率**：每30秒

**检查项目**：
1. Agent是否响应
2. 任务是否在推进
3. 资源是否充足
4. 网络是否正常

**不健康处理**：
1. 记录不健康状态
2. 尝试自动修复
3. 如果修复失败，暂停执行
4. 通知用户
```

### 方案3：改进错误恢复

```markdown
#### 错误恢复机制

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
```

### 方案4：优化上下文管理

```markdown
#### 上下文管理优化

**压缩策略**：
- 每3个批次压缩一次
- 保留摘要，丢弃细节
- 防止上下文溢出

**预算控制**：
- 主Agent：保留最近30轮对话
- 子Agent：每批次新建会话
```

---

## 📋 具体实施步骤

### 步骤1：更新主代理提示词

在主代理提示词中添加以下内容：

```markdown
### 卡住预防机制

#### 超时检测

**检测频率**：每60秒

**超时阈值**：
- 子代理：300秒
- 批次：1800秒

**超时处理**：
1. 记录超时日志
2. 尝试恢复（3次）
3. 恢复失败则跳过
4. 继续执行

#### 健康检查

**检查频率**：每30秒

**检查项目**：
- Agent响应性
- 任务进度
- 资源状态
- 网络状态

**处理策略**：
- healthy：继续
- degraded：警告并继续
- unhealthy：暂停并恢复

#### 快速失败

**检测点**：
- Agent启动前
- 任务执行中
- 任务完成后

**失败类型**：
- 可恢复：自动重试
- 不可恢复：跳过
- 平台错误：等待

#### 部分恢复

**恢复粒度**：
- 批次级别
- 任务级别
- Agent级别

**恢复时间**：
- 批次：< 1分钟
- 任务：< 30秒
- Agent：< 2分钟
```

### 步骤2：添加诊断命令

```bash
# 检查系统状态
tail -20 {PROJECT_ROOT}/outputs/main-log.md

# 检查最近事件
tail -20 {PROJECT_ROOT}/outputs/events.jsonl

# 检查超时记录
grep "timeout" {PROJECT_ROOT}/outputs/events.jsonl

# 检查错误记录
grep "error" {PROJECT_ROOT}/outputs/events.jsonl

# 检查Agent状态
grep "agent_spawn\|agent_complete" {PROJECT_ROOT}/outputs/events.jsonl | tail -10
```

### 步骤3：添加恢复命令

```bash
# 从检查点恢复
opencode
# 选择主代理，系统自动恢复

# 手动跳过卡住任务
vi {PROJECT_ROOT}/outputs/checkpoint.json
# 修改 currentBatch 增加1

# 重置状态重新开始
rm {PROJECT_ROOT}/outputs/checkpoint.json
rm -rf {PROJECT_ROOT}/outputs/agent-registry/
```

---

## 🎯 预防措施

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
```

---

## 📊 监控指标

### 关键指标

```json
{
  "metrics": {
    "agent_response_time": {
      "threshold": 60,
      "unit": "seconds"
    },
    "batch_duration": {
      "threshold": 1800,
      "unit": "seconds"
    },
    "error_rate": {
      "threshold": 10,
      "unit": "percent"
    },
    "timeout_rate": {
      "threshold": 5,
      "unit": "percent"
    }
  }
}
```

### 告警规则

```json
{
  "alerts": [
    {
      "name": "agent_timeout",
      "condition": "agent_response_time > 300",
      "severity": "warning"
    },
    {
      "name": "batch_timeout",
      "condition": "batch_duration > 1800",
      "severity": "critical"
    },
    {
      "name": "high_error_rate",
      "condition": "error_rate > 10",
      "severity": "critical"
    }
  ]
}
```

---

## 🚨 紧急处理

### 处理流程

```markdown
1. **发现卡住**
   - 检查日志
   - 诊断原因
   - 选择方案

2. **尝试恢复**
   - 从检查点恢复
   - 跳过卡住任务
   - 重启代理

3. **验证恢复**
   - 检查任务进度
   - 确认系统正常
   - 记录恢复日志

4. **预防再次**
   - 分析根本原因
   - 调整配置
   - 更新文档
```

### 恢复命令

```bash
# 方案1：从检查点恢复
opencode
# 选择主代理

# 方案2：跳过卡住任务
vi {PROJECT_ROOT}/outputs/checkpoint.json
# 修改 currentBatch

# 方案3：重置状态
rm {PROJECT_ROOT}/outputs/checkpoint.json
rm -rf {PROJECT_ROOT}/outputs/agent-registry/
```

---

## 📚 相关文档

- `docs/troubleshooting.md` - 通用故障排除
- `docs/stuck-diagnosis.md` - 卡住问题诊断
- `docs/timeout-recovery.md` - 超时检测与恢复
- `docs/error-recovery.md` - 错误恢复机制
- `docs/monitoring-alerting.md` - 监控告警系统
