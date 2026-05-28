# 日志格式规范

## 时间格式

使用 `yymmdd hhmm` 格式（如 `260506 1430`），精确到分钟。每次写日志时取当前时间。

## 日志位置

追加到 `{PROJECT_ROOT}/outputs/main-log.md`，每行以 `- ` 开头。

## 标准模板

```markdown
- {yymmdd hhmm} 项目启动，需求：{REQUIREMENT_FILE}
- {yymmdd hhmm} 输出目录：{OUTPUT_DIR}
- {yymmdd hhmm} 项目目录：{PROJECT_DIR}

- {yymmdd hhmm} ── Batch {N}: {模块列表} ──
- {yymmdd hhmm} 本批开发完成：{模块列表} (ID: {ID})
- {yymmdd hhmm} 首次测试 {模块}：{维度1}{P/F} / {维度2}{P/F} / {维度3}{P/F}
- {yymmdd hhmm} 第{N}轮修正：{模块列表}(ID:{ID})
- {yymmdd hhmm} {模块} 完成，迭代{N}次

- {yymmdd hhmm} ──── 项目完成 ────
- {yymmdd hhmm} 全部 {N} 个模块开发完成
- {yymmdd hhmm} 迭代统计：1次通过{X}个 / 2次通过{Y}个 / 3次通过{Z}个 / 自动降级{W}个
```

## 异常事件日志格式

### Agent 超时
```
- {yymmdd hhmm} Agent超时：{agent_type}（{agent_id}），超时批次 {batch}
```

### Agent Registry 读取失败
```
- {yymmdd hhmm} ⚠️ agent-registry/{key}.json 读取失败，{Agent名} 降级通过
```

### Agent 会话过期
```
- {yymmdd hhmm} ⚠️ {Agent名} 会话过期（ID: {agent_id}），无法 resume，降级通过
```

### 修正循环降级
```
- {yymmdd hhmm} ⚠️ {模块列表} 3轮修正后仍有 blocker/major FAIL，自动降级通过
```
