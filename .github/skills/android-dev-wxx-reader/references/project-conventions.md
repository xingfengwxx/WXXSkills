# Novelab 项目约束与仓库现状

## 规范优先级

1. 根目录 `AGENTS.md` 是当前仓库唯一的工作区级指令文件。
2. 如果存量代码和 `AGENTS.md` 冲突：
   - 新增代码优先遵守 `AGENTS.md`
   - 尽量做局部收敛，不顺手扩大战场
   - 在说明中明确哪些是“本次修复”，哪些是“遗留待迁移点”
3. 这个 skill 只记录 **Novelab 特有约束**，不重复 Android 通用知识。

## 产品与演进方向

- 产品定位：小说阅读器，优先关注书架、阅读、阅读进度、阅读偏好。
- 当前形态：单模块 `:app`，由它负责应用入口、导航装配、依赖注入入口和模块集成。
- 后续演进方向：
  - `:core:common`
  - `:core:model`
  - `:core:network`
  - `:core:database`
  - `:core:datastore`
  - `:feature:bookshelf`
  - `:feature:reader`
- `:reader-core` 与 `:payment` 当前只允许预留职责与接口边界，**不要**让业务层直接依赖未来旧项目的具体实现。

## 技术基线

- 语言与构建：Kotlin、Gradle Kotlin DSL、Version Catalog
- UI：Jetpack Compose + Material 3 + Navigation Compose，优先单 Activity；不要新增 XML 页面
- 架构：MVVM + immutable `UiState` + `StateFlow`
- 异步：Kotlin Coroutines + Flow；不要引入 RxJava
- 网络：OkHttp + Retrofit，统一包装错误模型、超时、拦截器和日志开关
- DI：Hilt；不要引入 Koin
- 图片：Coil
- 本地数据：Room 负责结构化数据，DataStore 负责轻量偏好
- 日志：统一使用 UtilCodex 的 `LogUtils`
- 国际化：英文默认资源 + 简体中文资源，支持继续扩语言

## 当前仓库落点

- `app/src/main/java/com/wenova/novelab/app/NovelabApp.kt`：Compose 根装配
- `app/src/main/java/com/wenova/novelab/navigation/`：导航定义与 NavHost
- `app/src/main/java/com/wenova/novelab/feature/bookshelf/`：书架基线实现
- `app/src/main/java/com/wenova/novelab/feature/reader/`：阅读页占位实现
- `app/src/main/java/com/wenova/novelab/core/datastore/`：阅读偏好仓储
- `app/src/main/java/com/wenova/novelab/core/database/`：Room 数据库与 DAO
- `app/src/main/java/com/wenova/novelab/core/network/`：网络 API
- `app/src/main/java/com/wenova/novelab/core/di/`：Hilt Modules
- `app/src/main/java/com/wenova/novelab/core/model/`：稳定业务模型
- `gradle/libs.versions.toml`：依赖和插件版本目录
- `app/src/main/java/com/wenova/novelab/NovelabApplication.kt`：Application 入口

## 已确认的存量偏差（新代码不要继续复制）

- 当前 `ReaderPreferences` 仍使用 `darkModeEnabled: Boolean`，但项目规范要求三态 `ThemeMode`：`FOLLOW_SYSTEM` / `LIGHT` / `DARK`，并通过 DataStore 持久化。
- 当前仓库缺少 `app/src/main/res/values-zh-rCN/strings.xml`；任何新增的用户可见文案都应同时补齐英文默认资源和简体中文资源。
- `NovelabApplication` 目前只有 `@HiltAndroidApp`；如果接入 UtilCodex，应在这里统一执行 `Utils.init(this)`，不要分散在 Activity、Fragment 或 Composable。
- 版本以 `gradle/libs.versions.toml` 为准；不要从外部文章抄版本号覆盖当前仓库真实依赖。

## 编码与分层规则

- 包名前缀统一为 `com.wenova.novelab`
- Compose 页面优先拆为 `Route`、`Screen`、`UiState`、`ViewModel`
- ViewModel 负责编排状态，不直接持有 Android View
- Repository 是数据访问统一入口，协调网络、本地数据库、DataStore 和未来迁移模块
- DTO、Room Entity、UI Model 不能混用；跨层前先做映射
- 用户可见文本必须通过 Android 资源系统或 locale-aware 格式化能力输出
- 业务逻辑不要直接写在 `Activity` 或 Composable 中

## 数据边界

- Room：书籍信息、章节目录、书签、阅读进度、缓存索引等结构化数据
- DataStore：主题、字号、行距、背景、翻页方式、夜间模式、轻量设置
- 不要把阅读偏好塞进 Room
- 不要把结构化阅读进度塞进 DataStore
- 远端模型落库前先映射，不要让 API DTO 直接渗透到 UI

## 主题与国际化要求

- 主题必须支持：跟随系统 / 浅色 / 深色
- 主题切换必须即时生效，不依赖重启应用
- 设置页负责修改主题偏好，业务页面只消费主题状态
- 阅读器夜间模式、背景、字号等偏好要和全局主题策略协同设计，避免出现冲突双入口
- 默认语言资源在 `res/values/strings.xml`
- 简体中文资源在 `res/values-zh-rCN/strings.xml`
- 变量文案优先使用占位符、`plurals`、locale-aware 格式化，而不是手工拼接字符串

## 依赖与构建规则

- 新依赖统一写入 `gradle/libs.versions.toml`
- 不要在模块 `build.gradle.kts` 中硬编码版本号
- 调整依赖、Gradle 或模块结构后，至少验证：
  - `./gradlew :app:assembleDebug`
  - `./gradlew :app:testDebugUnitTest`
- 仅在确有必要时才推进模块拆分；当前阶段优先保证基线稳固、边界清晰、迁移友好

## 新源码文件要求

- 新建 Kotlin / Java / Gradle Kotlin DSL 等支持块注释的源码文件时，使用统一文件头模板
- 模板见 [`templates.md`](./templates.md)
- `description` 必须写明文件职责，不能留空
