# Novelab 常见反模式

## 使用原则

当任务涉及重构、代码生成、Review、排障时，优先对照本清单。这里列的是 **模型在本仓库里最容易犯、而且代价比较高的错误**。

## 架构与状态管理

### 不要这样做

- 在 `Activity` 或 Composable 里直接编排业务流程、访问 Repository、拼接复杂状态
- 新功能继续引入 `LiveData` 作为主要状态出口
- 在多个层级散落可变状态，导致 UI 状态没有单一来源
- ViewModel 直接持有 View、Binding、Fragment、Activity
- Repository 直接返回 DTO、Entity 或 `LiveData`

### 应该这样做

- 用 `UiState` 数据类表达页面状态，由 `StateFlow` 对外暴露
- ViewModel 负责状态编排，Repository 负责数据访问协调
- DTO / Entity / UI Model 分层映射后再跨层传递

## 项目演进与模块边界

### 不要这样做

- 让 `:app`、feature、ViewModel 直接依赖未来旧项目里的阅读器或支付具体类
- 为了“看起来高级”提前把单模块项目拆成一堆小模块
- 看到存量实现就照抄，即使它已经落后于 `AGENTS.md`

### 应该这样做

- 为 `reader-core`、`payment` 只预留 contract / facade / adapter 边界
- 优先稳住 `:app` 装配根和现有特性目录，再考虑按规模拆分
- 当前实现和规范冲突时，以规范为目标态，做局部收敛

## 主题与偏好

### 不要这样做

- 继续把应用级主题建模成单个 `darkModeEnabled: Boolean`
- 把主题模式落到 Room
- 单独再造一个与全局主题冲突的阅读器夜间模式入口
- 把结构化阅读进度、书签、目录等数据塞进 DataStore

### 应该这样做

- 应用级主题统一抽象为 `ThemeMode(FOLLOW_SYSTEM/LIGHT/DARK)`
- 主题和轻量阅读偏好使用 DataStore 持久化
- 结构化阅读数据使用 Room
- 阅读器偏好设计时先检查是否会与全局主题策略冲突

## 文案与国际化

### 不要这样做

- 在 Kotlin / Compose 里硬编码中文或英文
- 只改 `values/strings.xml`，不补 `values-zh-rCN/strings.xml`
- 直接把服务端返回的枚举值、字段值、文案当 UI 文案展示
- 手工拼接数字、日期、章节数、百分比字符串

### 应该这样做

- 所有用户可见文本都进入 Android 资源系统
- 默认英文资源完整可用，中文简体同步补齐
- 带变量文案使用占位符、`plurals` 或 locale-aware 格式化
- 服务端字段进入 UI 前先映射到本地化文案

## 依赖、日志与构建

### 不要这样做

- 在 `build.gradle.kts` 里直接写版本号
- 引入 Koin、RxJava、`android.util.Log`、`println`
- 在日志里输出 token、密码、密钥、支付参数、手机号、阅读正文等敏感信息
- 因为文章示例里有某个依赖版本，就覆盖当前仓库的真实版本目录

### 应该这样做

- 所有依赖和插件版本统一进入 `gradle/libs.versions.toml`
- 日志统一使用 `LogUtils`
- 任何依赖调整都先对齐当前仓库版本目录，再决定是否升级

## UI 技术选型

### 不要这样做

- 新增 XML 页面、ViewBinding 流程或 Fragment-heavy 页面结构
- 在 Composable 中直接发网络请求、读写数据库
- 把业务文案、格式化逻辑、导航常量散落在多个页面里

### 应该这样做

- 继续沿用单 Activity + Compose Navigation
- UI 层保持无状态或最小状态
- 导航、状态、字符串、数据访问各自归位

## 提交前的快速反查

如果本次改动涉及以下任一主题，完成前至少回看一次本清单：

- `ThemeMode` / DataStore / 设置页
- `strings.xml` / 本地化 / 格式化文案
- `ViewModel` / `UiState` / Repository / 映射层
- `Room` / 阅读进度 / 书签 / 阅读偏好
- `gradle/libs.versions.toml` / 新依赖 / 日志
