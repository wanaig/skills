# 经验知识图谱集成

## 知识库文件

`{PROJECT_ROOT}/outputs/knowledge-base.json`

## 主Agent职责

1. **初始化知识库**：首次启动时创建空的 knowledge-base.json 结构
2. **传递知识库路径**：将 knowledge-base.json 路径传递给开发子Agent
3. **读取知识库摘要**：每批次开始前，读取 knowledge-base.json 中的 patterns 和 antiPatterns 数量，了解已知问题
4. **不直接修改知识库**：知识库由开发子Agent维护，主Agent只读取摘要信息

## 知识库应用

1. **批次规划时**：参考历史修正数据，调整批次大小
2. **测试结果分析时**：识别是否为已知问题模式
3. **生成报告时**：引用知识库中的统计数据

## 知识库摘要读取

```markdown
读取 knowledge-base.json，提取：
- patterns 数量：{count}
- antiPatterns 数量：{count}
- 最常见的 fixStrategy：{problemType} - {bestApproach}
- 高频问题类型：{category}
```
