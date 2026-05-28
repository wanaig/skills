# 诊断命令

## 系统状态检查

```bash
# 检查最新日志
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

## 紧急恢复

```bash
# 从检查点恢复
opencode
# 选择主代理，系统自动恢复

# 跳过卡住任务
vi {PROJECT_ROOT}/outputs/checkpoint.json
# 修改 currentBatch 增加1

# 重置状态
rm {PROJECT_ROOT}/outputs/checkpoint.json
rm -rf {PROJECT_ROOT}/outputs/agent-registry/
```
