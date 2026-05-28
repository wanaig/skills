# 并行度优化

## 动态并行策略

### 标准模式（默认）

- 测试Agent数量：3个（各维度一个）
- 适用场景：模块数量 >= 3

### 紧凑模式

- 测试Agent数量：2个
- 适用场景：模块数量 = 2，或上下文紧张
- 策略：合并两个维度到一个Agent

### 单批模式

- 测试Agent数量：1个
- 适用场景：模块数量 = 1，或剩余任务
- 策略：所有维度串行执行

## 并行度选择逻辑

```
remaining = len(pendingItems)
if remaining >= 3:
    parallelism = 3
elif remaining == 2:
    parallelism = 2
else:
    parallelism = 1
```

## 资源感知调整

- 检测到平台并发限制时，自动降低并行度
- 上下文紧张时，减少并行度以降低上下文累积速度
