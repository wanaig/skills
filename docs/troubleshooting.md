# 卡住问题诊断与优化

本文档分析系统卡住的原因并提供解决方案。

---

## 🔍 卡住原因分析

### 1. 主代理卡住原因

#### 1.1 等待子代理超时

**症状**：主代理停在"等待子代理完成"阶段

**原因**：
- 子代理执行时间过长（超过300秒）
- 子代理会话已过期
- 子代理返回格式不正确

**诊断**：
```bash
# 检查最近的Agent调用
tail -50 {PROJECT_ROOT}/outputs/events.jsonl

# 检查是否有超时记录
grep "timeout" {PROJECT_ROOT}/outputs/events.jsonl
```

**解决方案**：
```markdown
1. 检查子代理是否还在运行
2. 检查API配额是否耗尽
3. 手动跳过当前批次继续
```

---

#### 1.2 上下文溢出

**症状**：主代理响应变慢或停止响应

**原因**：
- 对话轮次过多（超过50轮）
- 单次输入内容过长
- 累积的上下文过大

**诊断**：
```bash
# 检查对话轮次（估算）
# 如果已经进行了很多批次，可能需要压缩
```

**解决方案**：
```markdown
1. 重启OpenCode会话
2. 从检查点恢复
3. 使用更小的批次大小
```

---

#### 1.3 检查点损坏

**症状**：恢复后状态不正确或卡住

**原因**：
- 检查点文件写入不完整
- 检查点格式错误
- 检查点与实际状态不一致

**诊断**：
```bash
# 检查检查点文件
cat {PROJECT_ROOT}/outputs/checkpoint.json

# 检查是否有语法错误
python -m json.tool {PROJECT_ROOT}/outputs/checkpoint.json
```

**解决方案**：
```markdown
1. 删除损坏的检查点文件
2. 从备份恢复检查点
3. 手动修复检查点
```

---

### 2. 子代理卡住原因

#### 2.1 任务过于复杂

**症状**：子代理长时间无响应

**原因**：
- 单次任务包含太多模块
- 模块逻辑过于复杂
- 需要读取的文件过多

**诊断**：
```bash
# 检查当前批次的任务数
grep "batch_start" {PROJECT_ROOT}/outputs/events.jsonl | tail -1
```

**解决方案**：
```markdown
1. 减小BATCH_SIZE（从3改为1）
2. 拆分复杂模块
3. 简化任务描述
```

---

#### 2.2 文件读取问题

**症状**：子代理卡在"读取文件"阶段

**原因**：
- 文件路径不存在
- 文件过大
- 文件编码问题

**诊断**：
```bash
# 检查文件是否存在
ls -la {PROJECT_ROOT}/outputs/

# 检查文件大小
du -sh {PROJECT_ROOT}/outputs/*.md
```

**解决方案**：
```markdown
1. 确认文件路径正确
2. 检查文件是否存在
3. 检查文件编码（UTF-8）
```

---

#### 2.3 错误处理失败

**症状**：子代理遇到错误后卡住

**原因**：
- 错误处理逻辑有bug
- 重试次数用尽
- 错误恢复失败

**诊断**：
```bash
# 检查错误记录
grep "error" {PROJECT_ROOT}/outputs/events.jsonl | tail -10

# 检查告警记录
cat {PROJECT_ROOT}/outputs/alerts.jsonl
```

**解决方案**：
```markdown
1. 查看错误详情
2. 手动修复问题
3. 跳过失败的任务
```

---

## 🛠️ 优化方案

### 1. 超时优化

#### 1.1 调整超时时间

在 `opencode.json` 中配置：

```json
{
  "agent": {
    "frontend-main": {
      "timeout": 600,
      "subagent_timeout": 300
    }
  }
}
```

#### 1.2 添加超时检测

在主代理提示词中添加：

```markdown
#### 超时检测机制

**检测频率**：每60秒检查一次

**超时阈值**：
- 子代理：300秒
- 主代理：600秒

**超时处理**：
1. 记录超时日志
2. 尝试恢复会话
3. 如果恢复失败，跳过当前任务
4. 继续执行下一个任务
```

---

### 2. 会话保活优化

#### 2.1 心跳机制

```markdown
#### 心跳检测

**检测间隔**：每30秒

**心跳内容**：
- 检查会话是否活跃
- 检查Agent是否在运行
- 检查是否有进展

**失活处理**：
1. 记录失活日志
2. 创建新会话
3. 从检查点恢复
4. 继续执行
```

#### 2.2 会话刷新策略

```markdown
#### 会话刷新

**刷新条件**：
- 会话存活超过90分钟
- 会话无响应超过60秒
- 检测到会话错误

**刷新流程**：
1. 保存当前状态到检查点
2. 结束当前会话
3. 创建新会话
4. 从检查点恢复状态
5. 继续执行
```

---

### 3. 上下文优化

#### 3.1 上下文压缩

```markdown
#### 上下文压缩策略

**压缩频率**：每3个批次

**压缩内容**：
- 已完成批次 → 仅保留统计摘要
- 已解决问题 → 仅保留数量
- 中间状态 → 合并为最终状态

**压缩后**：
- 写入 context-summary.md
- 后续会话读取此文件恢复上下文
```

#### 3.2 上下文预算

```markdown
#### 上下文预算

**主Agent**：保留最近30轮对话
**子Agent**：每批次新建会话

**监控指标**：
- 对话轮次
- Token使用量
- 上下文大小

**溢出处理**：
- 自动触发压缩
- 子Agent强制新建
- 主Agent保留最小工作集
```

---

### 4. 错误恢复优化

#### 4.1 快速失败

```markdown
#### 快速失败机制

**检测点**：
- Agent启动前
- 任务执行中
- 任务完成后

**失败类型**：
- 可恢复：自动重试
- 不可恢复：记录并跳过
- 平台错误：等待或切换

**处理时间**：
- 检测：< 1秒
- 决策：< 5秒
- 恢复：< 30秒
```

#### 4.2 部分恢复

```markdown
#### 部分恢复机制

**恢复粒度**：
- 批次级别：跳过整个批次
- 任务级别：跳过单个任务
- Agent级别：重启单个Agent

**恢复流程**：
1. 检测失败点
2. 保存当前状态
3. 跳过失败部分
4. 继续执行剩余部分
5. 记录失败日志
```

---

### 5. 监控优化

#### 5.1 实时监控

```markdown
#### 实时监控

**监控指标**：
- Agent状态
- 任务进度
- 错误率
- 响应时间

**监控频率**：每10秒

**告警阈值**：
- 响应时间 > 60秒：警告
- 响应时间 > 300秒：严重
- 错误率 > 10%：严重
```

#### 5.2 健康检查

```markdown
#### 健康检查

**检查频率**：每60秒

**检查项目**：
- Agent是否响应
- 任务是否在推进
- 资源是否充足
- 网络是否正常

**不健康处理**：
1. 记录不健康状态
2. 尝试自动修复
3. 如果修复失败，暂停执行
4. 通知用户
```

---

## 🔧 实时诊断命令

### 检查系统状态

```bash
# 1. 检查最新日志
tail -20 {PROJECT_ROOT}/outputs/main-log.md

# 2. 检查最近事件
tail -20 {PROJECT_ROOT}/outputs/events.jsonl

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
```

### 检查任务进度

```bash
# 检查批次进度
grep "batch_" {PROJECT_ROOT}/outputs/events.jsonl | tail -10

# 检查修正轮次
grep "fix_round" {PROJECT_ROOT}/outputs/events.jsonl | tail -5

# 检查测试结果
grep "test_result" {PROJECT_ROOT}/outputs/events.jsonl | tail -10
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
opencode

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

### 1. 定期保存检查点

```markdown
**保存频率**：
- 每批次完成后
- 每个Phase完成后
- 检测到错误时
- 手动保存（可选）
```

### 2. 监控资源使用

```markdown
**监控项目**：
- API调用次数
- 上下文大小
- 磁盘空间
- 内存使用
```

### 3. 控制任务规模

```markdown
**建议规模**：
- 小型项目：5-10个模块
- 中型项目：10-20个模块
- 大型项目：20-30个模块（分阶段）

**批次大小**：
- 简单模块：BATCH_SIZE=3
- 复杂模块：BATCH_SIZE=1
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

# 检查API配额
# （在OpenCode中查看）

# 检查项目目录
ls -la {PROJECT_ROOT}
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
