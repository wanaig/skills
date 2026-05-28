# 文件处理最佳实践

## 核心原则

**先搜索，后读取** — 使用 Grep 定位目标内容，再有针对性地读取文件片段。

## 分块读取指南

### 何时使用分块读取

1. 文件超过 200 行
2. 需要读取特定部分
3. 避免上下文溢出

### 分块读取方法

```bash
# 读取前100行
Read(filePath, offset=0, limit=100)

# 读取第101-200行
Read(filePath, offset=100, limit=100)

# 读取第201-300行
Read(filePath, offset=200, limit=100)
```

### 推荐参数

| 场景 | offset | limit | 说明 |
|------|--------|-------|------|
| 小文件 | 0 | 200 | 一次性读取 |
| 大文件 | 0 | 100 | 分块读取 |
| 特定位置 | N | 100 | 从第N行开始 |

## Grep 搜索指南

### 何时使用 Grep

1. 查找特定函数/类/变量
2. 搜索关键词
3. 定位错误信息
4. 查找特定模式

### Grep 搜索方法

```bash
# 搜索函数定义
Grep(pattern="function\s+\w+", path="文件路径")

# 搜索类定义
Grep(pattern="class\s+\w+", path="文件路径")

# 搜索变量
Grep(pattern="const\s+\w+", path="文件路径")

# 搜索导入
Grep(pattern="import\s+", path="文件路径")

# 搜索错误
Grep(pattern="error|Error", path="文件路径")
```

### 常用正则表达式

| 模式 | 说明 | 示例 |
|------|------|------|
| `\w+` | 匹配单词字符 | `function\s+\w+` |
| `\d+` | 匹配数字 | `line\s+\d+` |
| `.*` | 匹配任意字符 | `TODO.*` |
| `[abc]` | 匹配字符集 | `[eE]rror` |
| `\s+` | 匹配空白字符 | `\s+function` |

## Glob 查找指南

### 何时使用 Glob

1. 查找特定类型的文件
2. 搜索特定目录下的文件
3. 查找匹配模式的文件

### Glob 查找方法

```bash
# 查找所有 .vue 文件
Glob(pattern="**/*.vue")

# 查找所有 .ts 文件
Glob(pattern="**/*.ts")

# 查找特定目录下的文件
Glob(pattern="src/**/*.vue")

# 查找特定名称的文件
Glob(pattern="**/UserList.vue")
```

### 常用模式

| 模式 | 说明 | 示例 |
|------|------|------|
| `**/*` | 所有文件 | `**/*` |
| `*.ext` | 特定扩展名 | `*.vue` |
| `**/*.ext` | 递归查找 | `**/*.ts` |
| `dir/**` | 特定目录 | `src/**` |

## 最佳实践

### 1. 先搜索，后读取

```bash
# 步骤1：使用 Grep 搜索目标
Grep(pattern="function\s+fetchUsers", path="src/**/*.ts")

# 步骤2：根据搜索结果读取文件
Read("src/api/user.ts", offset=10, limit=50)
```

### 2. 分块读取大文件

```bash
# 不推荐：一次性读取整个大文件
Read("large-file.md")

# 推荐：分块读取
Read("large-file.md", offset=0, limit=100)
Read("large-file.md", offset=100, limit=100)
```

### 3. 使用 Glob 过滤文件

```bash
# 步骤1：使用 Glob 找到目标文件
Glob(pattern="src/**/*.vue")

# 步骤2：对每个文件使用 Grep 搜索
Grep(pattern="loading", path="src/views/UserList.vue")

# 步骤3：读取相关部分
Read("src/views/UserList.vue", offset=20, limit=30)
```

## 常见问题解决方案

### 问题1：文件过大导致卡住

**解决方案**：使用分块读取

```bash
# 分块读取
Read("large-file.md", offset=0, limit=100)
Read("large-file.md", offset=100, limit=100)
```

### 问题2：找不到目标内容

**解决方案**：使用 Grep 搜索

```bash
# 搜索关键词
Grep(pattern="关键词", path="文件路径")

# 搜索函数
Grep(pattern="function\s+\w+", path="文件路径")
```

### 问题3：需要读取多个文件

**解决方案**：使用 Glob 过滤，再有针对性地读取

```bash
# 步骤1：使用 Glob 找到所有相关文件
Glob(pattern="src/**/*.vue")

# 步骤2：对每个文件使用 Grep 搜索
Grep(pattern="loading", path="src/views/UserList.vue")

# 步骤3：只读取需要的文件
Read("src/views/UserList.vue", offset=0, limit=100)
```

### 问题4：需要查找特定代码段

**解决方案**：结合 Grep 和 Read

```bash
# 步骤1：使用 Grep 找到目标行
Grep(pattern="function\s+fetchUsers", path="src/api/user.ts")

# 步骤2：读取目标行附近的代码
Read("src/api/user.ts", offset=10, limit=20)
```

## 注意事项

1. **避免读取整个大文件** — 始终使用 limit 参数
2. **先搜索，后读取** — 使用 Grep 定位目标
3. **分块读取** — 每次读取不超过200行
4. **使用 Glob 过滤** — 先找到目标文件，再读取
5. **注意上下文限制** — 避免同时读取太多内容
