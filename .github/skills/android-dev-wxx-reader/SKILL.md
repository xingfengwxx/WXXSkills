---
name: android-dev-wxx-reader
description: 'Novelab 小说阅读器 Android 开发 skill。用于 Kotlin、Compose、Material 3、Hilt、Room、DataStore、Retrofit、OkHttp、Navigation、Gradle Kotlin DSL 相关任务，以及书架、阅读页、阅读偏好、ThemeMode、国际化、Version Catalog、代码评审、崩溃排查和架构演进。遇到 AGENTS.md 规范、StateFlow/MVVM/Repository、LogUtils、reader-core/payment 预留边界等场景时使用。'
argument-hint: '描述你的 Novelab Android 任务，例如：实现书架筛选、补三态主题、设计 ReaderPreferences、排查崩溃'
---

# Android Dev WXX Reader

这个 skill 的目标不是重复 Android 通用常识，而是把 **Novelab 项目特有的规范、反模式、迁移边界和交付检查** 固化下来，让模型在这个仓库里输出更稳、更贴项目实际的方案。

## 何时使用

- 实现或重构 `bookshelf`、`reader`、`navigation`、`theme`、`datastore`、`database`、`network`、`di`
- 生成 Kotlin / Compose / Gradle Kotlin DSL 代码
- 设计 `UiState`、`ViewModel`、`Repository`、Room、DataStore、Hilt Module
- 修改 `strings.xml`、国际化文案、主题模式、阅读偏好
- 排查 Crash、ANR、生命周期问题、架构漂移
- 为未来 `reader-core`、`payment` 迁移预留接口与边界

## 核心原则

1. **只写模型默认不知道的内容**：项目规范、架构约束、历史坑、迁移方向优先；泛泛的“遵循最佳实践”不值得占上下文。
2. **`AGENTS.md` 高于存量实现**：如果当前代码落后于规范，新增代码优先朝规范收敛，但要控制改动范围，并明确说明迁移点。
3. **正文保持简洁**：需要细节时再按需读取 `references/`，避免把所有规则堆进主文件。

## 先看什么

- 项目约束与仓库现状：[`project-conventions.md`](./references/project-conventions.md)
- 常见错误与禁区：[`anti-patterns.md`](./references/anti-patterns.md)
- 实施流程与交付检查：[`workflow-checklist.md`](./references/workflow-checklist.md)
- 常用结构模板：[`templates.md`](./references/templates.md)

## 标准工作流

1. 先判断任务属于哪类：UI/导航、主题与偏好、数据层、网络层、Gradle/依赖、排障与重构。
2. 读取相关现有文件，确认当前实现、命名、目录与接入点。
3. 对照 `AGENTS.md` 和 [`project-conventions.md`](./references/project-conventions.md)，决定是沿用现状还是顺手收敛技术债。
4. 编码时优先保持 `Route / Screen / UiState / ViewModel / Repository` 结构，确保 UI 状态单一来源、跨层模型显式映射。
5. 遇到以下场景时必须额外检查：
   - 用户可见文案 → 同步英文默认资源与中文简体资源
   - 主题/阅读偏好 → DataStore 持久化，并避免新增与全局主题冲突的入口
   - 依赖/构建调整 → 更新 `gradle/libs.versions.toml`，不要把版本写死到模块脚本
   - 日志 → 只用 `LogUtils`，且不能打印敏感信息
6. 完成后按 [`workflow-checklist.md`](./references/workflow-checklist.md) 做构建、测试和专项验证。

## 任务分流

- **书架 / 阅读页 / Compose 页面** → 先读 `project-conventions.md` + `templates.md`
- **ThemeMode / ReaderPreferences / 设置页** → 先读 `project-conventions.md` + `anti-patterns.md`
- **Room / Repository / DTO 映射 / Retrofit** → 先读 `project-conventions.md` + `templates.md`
- **Crash / 生命周期 / ANR / 架构问题** → 先读 `anti-patterns.md` + `workflow-checklist.md`
- **Gradle / 依赖接入 / 多模块预留** → 先读 `project-conventions.md`

## 维护约定

- AI 再犯新错误时，优先补充到 `anti-patterns.md`
- 结构或模板变化时，更新 `templates.md`
- 项目规范变化时，以 `AGENTS.md` 为源头同步修订 references
- 依赖细节、检查项和案例放在 references 中维护，不回填到 SKILL 主体里
