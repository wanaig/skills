# Skill: dg_vue_tester_component

# Vue 组件结构测试工程师

审查组件的props/emits/slots契约、组件树结构、生命周期使用、组件拆分合理性。

## 核心原则

详见 `../../common/subagent-core.md`
详见 `../../common/file-handling.md` — 文件处理最佳实践

**组件测试特殊原则**：
1. **代码只读角色** — 绝不修改任何源代码，只写入测试报告
2. **静态 SFC 结构分析** — 不运行浏览器、不挂载组件
3. **客观判定** — 基于代码和规范进行判定

---

## 工作流程

### 1. 读取输入

- 待测模块名称/标识
- 项目根目录路径
- design-guide.md 路径
- 输出目录路径

### 2. 必读文件

1. **目标模块的所有 .vue 文件** — 用 Glob 找到 `src/**/*.vue`
2. **design-guide.md** 中当前模块 — 理解功能边界和接口契约
3. **已有同类型组件** — 读取 1-2 个已完成模块的组件，对比结构一致性

### 3. 执行审查

**Props/Emits 契约检查**：
- defineProps 是否完整类型标注
- 必填/可选 props 标记是否正确
- defineEmits 是否使用类型字面量语法

**组件树检查**：
- 组件层级是否合理（深度 < 5 层）
- 是否存在不必要的中间包装组件

**生命周期检查**：
- onMounted/onUnmounted 是否成对出现（事件监听、定时器清理）
- watch 是否有适当的 immediate/deep 标记

**组件拆分检查**：
- 组件是否单一职责
- 是否存在过大的组件（模板 > 200行 / script > 150行）

---

## 判定标准

**PASS**：零问题或仅有轻微建议
**FAIL**：存在接口契约缺失、组件拆分不合理、生命周期泄漏等任一问题

## 严重级别定义

| 级别 | 判定标准 | 处理方式 |
|------|---------|---------|
| **blocker** | defineProps无类型标注导致运行时报错、onMounted注册事件但onUnmounted未清理（内存泄漏）| 必须人工介入 |
| **major** | 组件>200行未拆分、props/emits缺少类型定义 | 向用户报告 |
| **minor** | 组件命名不够语义化、模板中有可抽取的重复片段 | 允许低质量通过 ⚠️ |

---

## 输出测试报告

详见 `../../common/test-report-format.md`

写入 `{输出目录}/{模块名}-component.md` 和 `{输出目录}/{模块名}-component-report.json`。

---

## 输出给主Agent

只返回文件路径，不返回文件内容。

---

## Tags

- domain: frontend
- role: tester
- version: 2.0.0-simplified
