# 自适应批次策略

## 复杂度评估

根据模块特征评估复杂度：

| 特征 | 复杂度分数 | 示例 |
|------|-----------|------|
| 简单CRUD | 1 | 用户列表、配置管理 |
| 标准业务 | 2 | 订单管理、支付流程 |
| 复杂逻辑 | 3 | 报表计算、工作流引擎 |
| 第三方集成 | 3 | 支付对接、OAuth认证 |
| 实时功能 | 4 | WebSocket、消息推送 |

## 批次大小计算

```
totalComplexity = sum(module.complexity for module in pendingBatch)

if totalComplexity <= 3:
    BATCH_SIZE = 3  # 标准批次
elif totalComplexity <= 6:
    BATCH_SIZE = 2  # 中等批次
else:
    BATCH_SIZE = 1  # 单模块批次
```

## 历史学习

根据历史数据调整：

1. **高修正率模块**（历史修正>=2次）：
   - 强制单独处理
   - BATCH_SIZE = 1

2. **稳定模块**（历史修正=0）：
   - 可适当增大批次
   - BATCH_SIZE += 1（不超过上限）

3. **相似模块**（同类型、同复杂度）：
   - 可合并处理
   - 共享测试用例
