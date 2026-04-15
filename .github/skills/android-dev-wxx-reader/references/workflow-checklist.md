# Novelab 实施流程与交付检查

## 标准流程

1. **分类任务**
   - UI / 导航
   - 主题 / 阅读偏好 / DataStore
   - 数据层 / Room / Repository / Retrofit
   - 依赖 / Gradle / Hilt 接入
   - Bug / Crash / ANR / 架构漂移
2. **读取现有实现**
   - 找到对应 package、Route、ViewModel、Repository、DI Module、资源文件
   - 对照 `AGENTS.md` 判断当前实现是否已经偏离目标规范
3. **决定改动策略**
   - 纯增量：沿用现有结构，最小代价交付
   - 局部收敛：修当前任务附近的技术债，但不要无关大改
   - 预留边界：为未来 `reader-core` / `payment` 迁移设计 contract 或 facade
4. **实施**
   - 保持单一 `UiState`
   - 用 `StateFlow` / Flow 暴露状态
   - 明确 DTO / Entity / UI Model 映射
   - 把用户文案放进资源文件
   - 依赖改动同步进入 `gradle/libs.versions.toml`
5. **验证**
   - 先跑最相关的构建与测试
   - 再做主题、国际化、日志、安全性等专项检查
6. **总结**
   - 说明本次改了什么
   - 说明如何验证
   - 若有未顺手解决的技术债，显式标注

## 任务分支检查

### UI / 阅读页 / 书架

- 是否保持 `Route / Screen / UiState / ViewModel` 结构清晰？
- Composable 是否尽量无状态或仅保留最小本地状态？
- 是否避免在 UI 层直接访问数据源？
- 新增文案是否同时补到英文默认资源和简体中文资源？

### ThemeMode / DataStore / 设置页

- 是否使用三态 `ThemeMode`，而不是新增布尔开关？
- 主题切换是否能即时生效？
- 偏好是否由 DataStore 持久化？
- 是否避免与阅读器局部夜间模式形成冲突双入口？

### 数据层 / Room / Repository / 网络

- Room 与 DataStore 的边界是否清晰？
- 是否存在 DTO / Entity / UI Model 混用？
- Repository 是否是统一入口？
- 错误处理是否集中，而不是由页面各自拼异常分支？

### 依赖 / Gradle / Hilt

- 新依赖是否写入 `gradle/libs.versions.toml`？
- 模块脚本里是否出现硬编码版本号？
- Application / Hilt Module / Navigation 装配点是否放对位置？
- 如果引入 UtilCodex，是否在 `Application` 层集中初始化？

### 排障 / 重构 / Review

- 修的是根因还是表象？
- 是否因为复用存量代码而把旧问题继续扩散？
- 是否有必要补充测试、断言或更清晰的错误处理？

## 完成门槛

至少根据改动类型执行或核对以下事项：

### 通用

- `./gradlew :app:assembleDebug`
- 逻辑改动优先补或运行 `./gradlew :app:testDebugUnitTest`

### 主题相关

- 验证 跟随系统 / 浅色 / 深色 三种模式
- 验证切换后即时生效
- 验证重启后偏好恢复

### DataStore 相关

- 验证默认值恢复
- 验证读写成功
- 验证不会把结构化数据塞进去

### 国际化相关

- 验证 `res/values/strings.xml` 有完整默认资源
- 验证 `res/values-zh-rCN/strings.xml` 同步补齐
- 验证占位符、复数、数字/日期格式在不同语言下不出错

### 日志与安全相关

- 只使用 `LogUtils`
- 不输出 token、密码、密钥、支付参数、手机号、阅读正文等敏感信息

## 输出风格建议

交付说明里最好显式回答三件事：

1. **改了什么**：文件与职责
2. **为什么这么改**：对应哪条规范、哪条边界或哪个反模式
3. **怎么验证**：构建、测试、专项检查结果
