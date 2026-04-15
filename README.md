# WXXSkills

`WXXSkills` 是一个面向 GitHub Copilot / VS Code Agent 的 **Skill 集合仓库**，用于沉淀特定业务域的开发约束、参考资料、模板和交付检查项。

当前仓库聚焦 **Novelab 小说阅读器 Android 开发场景**，目标是把“项目特有知识”从通用经验里拆出来，让模型在真实项目里输出更贴近上下文、更稳定、更少跑偏的方案。

## 仓库定位

- 统一收纳可复用的 Skill 目录与说明文档
- 沉淀项目级约束、反模式、模板与实施检查清单
- 通过 `SKILL.md + references/` 的方式，兼顾**发现效率**和**上下文成本**
- 为后续扩展更多垂直领域 Skill 保持一致的组织方式

## 当前收录概览

- Skill 数量：**1**
- 参考文档数量：**4**
- 当前主题：**Android / Kotlin / Compose / 小说阅读器业务场景**

## 目录结构

```text
.
├── README.md
└── .github/
    └── skills/
        └── android-dev-wxx-reader/
            ├── SKILL.md
            └── references/
                ├── anti-patterns.md
                ├── project-conventions.md
                ├── templates.md
                └── workflow-checklist.md
```

## Skill 组织约定

每个 Skill 目录建议遵循下面的结构：

- `SKILL.md`：Skill 主入口，负责说明适用场景、核心原则、工作流和任务分流
- `description`：Skill 的“发现入口”，需要明确写出触发关键词与使用场景
- `argument-hint`：给出典型任务示例，帮助快速触发正确使用方式
- `references/`：按需拆分的详细资料，避免把所有规则都堆到主文件里

这种组织方式的好处是：**主文件足够短，便于识别；细节文档足够全，便于深挖。**

## Skill 索引

| Skill | 入口文件 | 适用场景 | 关键主题 | 参考资料 |
| --- | --- | --- | --- | --- |
| `android-dev-wxx-reader` | [`SKILL.md`](./.github/skills/android-dev-wxx-reader/SKILL.md) | Novelab 小说阅读器 Android 开发、重构、Review、排障 | Kotlin、Compose、Material 3、Hilt、Room、DataStore、Retrofit、Navigation、Gradle Kotlin DSL、MVVM、StateFlow | [`project-conventions`](./.github/skills/android-dev-wxx-reader/references/project-conventions.md)、[`anti-patterns`](./.github/skills/android-dev-wxx-reader/references/anti-patterns.md)、[`templates`](./.github/skills/android-dev-wxx-reader/references/templates.md)、[`workflow-checklist`](./.github/skills/android-dev-wxx-reader/references/workflow-checklist.md) |

## Skill 详细说明

### `android-dev-wxx-reader`

这是一个面向 **Novelab 小说阅读器项目** 的 Android 开发 Skill，重点不是重复 Android 通用知识，而是固化该项目特有的规范、约束、迁移边界和交付检查项。

#### 适用任务

- 实现或重构 `bookshelf`、`reader`、`navigation`、`theme`、`datastore`、`database`、`network`、`di`
- 生成 Kotlin / Compose / Gradle Kotlin DSL 代码
- 设计 `UiState`、`ViewModel`、`Repository`、Room、DataStore、Hilt Module
- 修改国际化文案、主题模式、阅读偏好、设置页交互
- 排查 Crash、ANR、生命周期问题、架构漂移
- 为未来 `reader-core`、`payment` 迁移预留接口与职责边界

#### 核心约束

- 目标项目中的 `AGENTS.md` 优先级高于存量实现
- UI 与架构默认走 **Compose + Material 3 + Navigation Compose + MVVM + immutable `UiState` + `StateFlow`**
- 主题模式优先采用三态 `ThemeMode(FOLLOW_SYSTEM / LIGHT / DARK)`，不要继续扩大 `darkModeEnabled: Boolean`
- `Room` 与 `DataStore` 边界要清晰：结构化数据进 Room，轻量偏好进 DataStore
- 用户可见文案需要同时考虑默认英文资源和简体中文资源
- 依赖版本统一收敛到 `gradle/libs.versions.toml`
- 日志统一使用 `LogUtils`，且禁止输出敏感信息

#### 配套参考文档

| 文档 | 作用 |
| --- | --- |
| [`project-conventions.md`](./.github/skills/android-dev-wxx-reader/references/project-conventions.md) | 记录 Novelab 项目约束、技术基线、目录落点、分层规则和演进方向 |
| [`anti-patterns.md`](./.github/skills/android-dev-wxx-reader/references/anti-patterns.md) | 汇总模型在该项目里最容易犯的高成本错误与修正方向 |
| [`templates.md`](./.github/skills/android-dev-wxx-reader/references/templates.md) | 提供文件头、Feature 结构、`UiState`、`ViewModel`、`ThemeMode`、Repository 等常用模板 |
| [`workflow-checklist.md`](./.github/skills/android-dev-wxx-reader/references/workflow-checklist.md) | 提供实施步骤、专项检查点、完成门槛与交付说明建议 |

#### 推荐触发方式

可以直接用自然语言描述任务，例如：

- “实现书架筛选”
- “补三态主题模式”
- “设计 ReaderPreferences”
- “排查阅读页崩溃”

## 如何使用这组 Skill

如果要在目标仓库中使用某个 Skill，推荐按下面的方式组织：

1. 将对应 Skill 目录放入目标仓库的 `.github/skills/<skill-name>/`
2. 保持 `SKILL.md` 为主入口，把详细内容拆到 `references/`
3. 在 `description` 中明确写出使用场景、关键词和任务类型
4. 如目标项目已有 `AGENTS.md`，应将其视为最高优先级约束来源

## 维护建议

- 新增 Skill 后，优先更新本 README 的索引和说明，避免仓库“有内容、没地图”
- 发现模型容易重复犯的错误时，优先补充到对应 Skill 的 `anti-patterns.md`
- 规范调整时，优先更新约束文档，再同步 README 中的摘要说明
- 模板或工作流变化时，优先维护 `references/`，保持 `SKILL.md` 主体简洁

## 后续扩展方向

当前仓库已经具备 Skill 集合的基础结构，后续可以继续扩展：

- 更多 Android 业务域 Skill（如支付、登录、书城、同步）
- 通用工程 Skill（如测试策略、性能优化、发布流程）
- 不同项目的专属 Skill，形成按业务或仓库分层管理的 Skill 索引

如果把 Skill 比作工具箱，这个 README 就是工具箱盖子上的标签——至少以后翻东西时，不会再靠玄学。🙂
