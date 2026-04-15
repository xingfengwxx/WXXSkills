# Novelab 常用结构模板

## 新源码文件头模板

> 适用于 Kotlin / Java / Gradle Kotlin DSL 等支持块注释的源码文件。
> `date` 请替换为创建文件时的当前系统时间，格式 `yyyy/M/d HH:mm`。

```text
/**
 * author : 王星星
 * date : 2026/4/15 10:00
 * email : 1099420259@qq.com
 * description : 文件职责说明
 */
```

## Compose Feature 目录建议

```text
feature/<name>/
├── <Name>Route.kt
├── <Name>Screen.kt
├── <Name>UiState.kt
├── <Name>ViewModel.kt
└── <Name>Repository.kt
```

适用场景：书架页、阅读设置页、阅读器辅助面板等新功能。

## `UiState` 模板

```kotlin
data class ReaderSettingsUiState(
    val isLoading: Boolean = false,
    val themeMode: ThemeMode = ThemeMode.FOLLOW_SYSTEM,
    val fontScale: Float = 1.0f,
    val errorMessage: String? = null,
)
```

- 用单一数据类表达页面状态
- 避免多个互相竞争的状态源

## `ViewModel` 模板

```kotlin
@HiltViewModel
class ReaderSettingsViewModel @Inject constructor(
    private val preferencesRepository: ReaderPreferencesRepository,
) : ViewModel() {

    val uiState: StateFlow<ReaderSettingsUiState> = preferencesRepository.observePreferences()
        .map { preferences ->
            ReaderSettingsUiState(
                themeMode = preferences.themeMode,
                fontScale = preferences.fontScale,
            )
        }
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5_000),
            initialValue = ReaderSettingsUiState(isLoading = true),
        )

    fun updateThemeMode(themeMode: ThemeMode) {
        viewModelScope.launch {
            preferencesRepository.updateThemeMode(themeMode)
        }
    }
}
```

- 暴露 `StateFlow`
- 由 ViewModel 编排状态，Repository 处理数据访问

## `ThemeMode` 与偏好模型模板

```kotlin
enum class ThemeMode {
    FOLLOW_SYSTEM,
    LIGHT,
    DARK,
}

data class ReaderPreferences(
    val themeMode: ThemeMode = ThemeMode.FOLLOW_SYSTEM,
    val fontScale: Float = 1.0f,
)
```

- 新增主题相关代码时，优先朝这个模型收敛
- 不要继续放大 `darkModeEnabled: Boolean` 这种过渡状态

## Repository 分层模板

```kotlin
interface ReaderRepository {
    fun observeChapterList(bookId: String): Flow<List<Chapter>>
    suspend fun updateReadingProgress(bookId: String, chapterId: String, offset: Int)
}

class OfflineFirstReaderRepository @Inject constructor(
    private val localDataSource: ReaderLocalDataSource,
    private val remoteDataSource: ReaderRemoteDataSource,
) : ReaderRepository {
    override fun observeChapterList(bookId: String): Flow<List<Chapter>> =
        localDataSource.observeChapterList(bookId)

    override suspend fun updateReadingProgress(bookId: String, chapterId: String, offset: Int) {
        localDataSource.updateReadingProgress(bookId, chapterId, offset)
    }
}
```

- Repository 作为统一入口
- 结构化阅读数据优先走 Room / 本地数据库边界

## 国际化模板

```xml
<!-- res/values/strings.xml -->
<string name="reader_progress">Read %1$d%%</string>
<string name="reader_continue">Continue reading</string>

<!-- res/values-zh-rCN/strings.xml -->
<string name="reader_progress">已阅读 %1$d%%</string>
<string name="reader_continue">继续阅读</string>
```

- 默认英文资源必须完整
- 中文简体同步补齐
- 数量变化优先考虑 `plurals`

## Application 初始化模板

```kotlin
@HiltAndroidApp
class NovelabApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        Utils.init(this)
    }
}
```

- 如果启用 UtilCodex，统一在 `Application` 层初始化
- 不要在 Activity、Fragment 或 Composable 中重复初始化
