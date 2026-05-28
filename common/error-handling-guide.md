# 常见错误处理映射表

## 用途

为开发类子智能体提供"错误→原因→修复方案"的快速映射，提高修正效率。

---

## 前端常见错误

### 编译错误

| 错误信息 | 常见原因 | 修复方案 |
|---------|---------|---------|
| `Cannot find module 'xxx'` | 依赖未安装或路径错误 | 运行 `npm install` 或检查 import 路径 |
| `Type 'xxx' is not assignable to type 'yyy'` | 类型不匹配 | 检查 Props/State 类型定义，添加类型转换 |
| `Property 'xxx' does not exist on type 'yyy'` | 访问不存在的属性 | 检查接口定义，添加可选标记 `?` |
| `Unexpected token` | 语法错误 | 检查括号、引号、分号是否匹配 |
| `Module not found: Can't resolve 'xxx'` | 路径别名配置错误 | 检查 vite.config.ts 或 tsconfig.json |

### 运行时错误

| 错误信息 | 常见原因 | 修复方案 |
|---------|---------|---------|
| `xxx is undefined` | 异步数据未加载完成 | 添加 `loading` 状态判断或可选链 `?.` |
| `Cannot read properties of null` | DOM 元素未渲染或数据为空 | 添加空值判断或 `v-if` 条件 |
| `Maximum update depth exceeded` | 无限循环更新 | 检查 useEffect 依赖数组或 computed 计算 |
| `Hydration mismatch` | SSR/CSR 内容不一致 | 检查服务端和客户端渲染逻辑 |
| `CORS error` | 跨域配置问题 | 检查 vite proxy 或后端 CORS 配置 |

### Vue 特有错误

| 错误信息 | 常见原因 | 修复方案 |
|---------|---------|---------|
| `Avoid mutating a prop directly` | 直接修改 props | 使用 emit 事件通知父组件 |
| `Invalid watch source` | watch 监听类型错误 | 确保监听 ref 或 getter 函数 |
| `Store is not defined` | Pinia store 使用错误 | 确保在 setup 中调用 `useStore()` |

---

## 后端常见错误

### 编译错误

| 错误信息 | 常见原因 | 修复方案 |
|---------|---------|---------|
| `cannot find symbol` | 类/方法未导入或不存在 | 检查 import 语句和类路径 |
| `incompatible types` | 类型不匹配 | 检查方法签名和参数类型 |
| `method does not override` | 方法签名不匹配 | 检查父类方法定义 |
| `unreported exception` | 异常未处理 | 添加 try-catch 或 throws 声明 |

### 运行时错误

| 错误信息 | 常见原因 | 修复方案 |
|---------|---------|---------|
| `NullPointerException` | 空指针访问 | 添加 null 检查或使用 Optional |
| `StackOverflowError` | 无限递归 | 检查递归终止条件 |
| `OutOfMemoryError` | 内存溢出 | 检查大对象创建和集合操作 |
| `Connection refused` | 数据库/服务连接失败 | 检查连接配置和服务状态 |
| `Duplicate entry` | 唯一约束冲突 | 检查数据唯一性或使用 INSERT IGNORE |

### Spring 特有错误

| 错误信息 | 常见原因 | 修复方案 |
|---------|---------|---------|
| `No qualifying bean` | Bean 未注册 | 检查 @Service/@Component 注解 |
| `Circular dependency` | 循环依赖 | 使用 @Lazy 或重构依赖关系 |
| `Validation failed` | 参数校验失败 | 检查 @Valid 注解和 DTO 约束 |
| `Access Denied` | 权限不足 | 检查 @PreAuthorize 配置 |

---

## 区块链常见错误

### 编译错误

| 错误信息 | 常见原因 | 修复方案 |
|---------|---------|---------|
| `ParserError` | 语法错误 | 检查 Solidity 版本和语法 |
| `TypeError` | 类型不匹配 | 检查变量类型和转换 |
| `DeclarationError` | 未声明的变量 | 检查变量作用域和声明 |

### 运行时错误

| 错误信息 | 常见原因 | 修复方案 |
|---------|---------|---------|
| `Revert` | 条件不满足 | 检查 require 语句和条件逻辑 |
| `Out of gas` | Gas 不足 | 优化循环和存储操作 |
| `Stack too deep` | 局部变量过多 | 拆分函数或使用结构体 |

---

## Flutter 常见错误

### 编译错误

| 错误信息 | 常见原因 | 修复方案 |
|---------|---------|---------|
| `The argument type 'xxx' can't be assigned` | 类型不匹配 | 检查 Widget 参数类型 |
| `The method 'xxx' isn't defined` | 方法不存在 | 检查类定义和继承关系 |
| `A value of type 'xxx' can't be returned` | 返回类型错误 | 检查方法返回类型声明 |

### 运行时错误

| 错误信息 | 常见原因 | 修复方案 |
|---------|---------|---------|
| `setState() called after dispose()` | Widget 已销毁但调用 setState | 添加 `mounted` 检查 |
| `RenderFlex overflowed` | 布局溢出 | 使用 Expanded 或 SingleChildScrollView |
| `The getter 'xxx' was called on null` | 空值访问 | 添加 null 检查或使用 `?.` |
| `No MaterialLocalizations found` | 缺少 Material 配置 | 确保 MaterialApp 包裹 Widget |

### Riverpod 特有错误

| 错误信息 | 常见原因 | 修复方案 |
|---------|---------|---------|
| `Could not find the correct Provider` | Provider 未定义 | 检查 Provider 定义和导入 |
| `Tried to read a provider from a place` | 上下文错误 | 确保在 Widget 树中使用 |

---

## 前后端联调常见错误

### HTTP 错误

| 状态码 | 含义 | 常见原因 | 修复方案 |
|--------|------|---------|---------|
| 400 | Bad Request | 请求参数格式错误 | 检查请求体格式和字段类型 |
| 401 | Unauthorized | Token 缺失或过期 | 检查 Token 携带和刷新逻辑 |
| 403 | Forbidden | 权限不足 | 检查用户角色和权限配置 |
| 404 | Not Found | 接口路径错误 | 检查 URL 路径和 HTTP 方法 |
| 409 | Conflict | 数据冲突 | 检查唯一约束和并发处理 |
| 422 | Unprocessable Entity | 业务逻辑错误 | 检查后端校验规则 |
| 429 | Too Many Requests | 请求频率过高 | 添加请求限流或重试机制 |
| 500 | Internal Server Error | 服务器内部错误 | 检查后端日志和异常处理 |

### 数据格式错误

| 问题 | 常见原因 | 修复方案 |
|------|---------|---------|
| 字段名不匹配 | 前后端命名风格不同 | 检查 camelCase/snake_case 转换 |
| 日期格式错误 | 日期序列化格式不一致 | 统一使用 ISO 8601 格式 |
| 数字精度丢失 | 大数字超出 JS 安全范围 | 使用 string 类型传输大数字 |
| 嵌套结构不一致 | 响应数据层级不同 | 检查 API 契约文档定义 |
