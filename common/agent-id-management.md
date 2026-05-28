# Agent ID 管理

## 获取方式

子Agent 完成后，将自身的 Agent ID 写入独立文件 `{PROJECT_ROOT}/outputs/agent-registry/{key}.json`，杜绝多Agent并发写入同一文件导致ID丢失。

## 文件结构

```
{PROJECT_ROOT}/outputs/agent-registry/
├── {agent_key}.json  ← {"id":"abc123","type":"agent_type","updated":"..."}
```

## 主Agent职责

1. 初始化时创建 `{PROJECT_ROOT}/outputs/agent-registry/` 目录
2. 子Agent 完成后，读取对应文件获取 Agent ID
3. 获取到 ID 后，必须记录在日志中

## 子Agent职责

完成后将 Agent ID 写入 `{PROJECT_ROOT}/outputs/agent-registry/{key}.json`

## 容错处理

读取 agent-registry/{key}.json 失败时，记录该 Agent 为"降级通过"，在日志中标注。不阻塞流程，不询问用户。

## ID 使用规则

1. **resume 用 Agent ID** — 必须使用 `task_id: "{ID}"` 格式，配合对应的 `subagent_type` 使用
2. **resume 必须指定对应的 subagent_type**，无需加载技能
3. **每批开发轮次结束后，ID 失效**，新批重新启动所有Agent
4. **同批修正循环中复用同一个 ID**，禁止启动新Agent
