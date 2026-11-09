---
name: android-architecture
description: >-
  Use when designing module structure, choosing architecture patterns
  (MVVM/MVI), setting up dependency injection, or defining layer
  boundaries in an Android project.
---

## 프로젝트 범위와 실행 근거

기존 구조, DI, pinned 버전과 요청된 작업을 먼저 확인한다. SDK, emulator, Hilt와 앱 의존성을 자동 설치하거나 전체 앱을 재구성하지 않는다. 원본 예제의 버전과 API가 불명확하거나 프로젝트와 다르면 [find-docs](../find-docs/SKILL.md)에 라이브러리, 정확한 버전과 질문을 전달해 공식 API 근거를 확인한다. 도구가 없거나 지원 API를 확인하지 못하면 미확인으로 보고하며 추측한 API를 구현 완료로 제시하지 않는다. 필요한 테스트 종류와 명령만 선택하고 SDK, 기기, 권한이 없으면 실행 미검증으로 남긴다. 정적 코드 확인과 지침 모델 테스트는 Android 빌드, instrumented 테스트나 실기기 성공의 근거가 아니다.

다른 skill 링크는 해당 작업이 필요할 때만 읽는 선택 참고다. 미선택 upstream skill은 배포되지 않았으며 링크 열람은 설치나 실행 승인이 아니다.

# Android Architecture

## Overview

Good architecture makes Android apps testable, maintainable, and scalable. This skill guides architectural decisions: module structure, layer separation (UI → Domain → Data), dependency injection with Hilt, state management with Coroutines and Flow, and patterns that survive configuration changes and process death.

## When to Use

- Starting a new Android project or module
- Adding a feature module to an existing multi-module project
- Choosing between MVVM and MVI for a screen
- Setting up or refactoring dependency injection
- Defining boundaries between layers
- Debugging lifecycle or state management issues

**Skip when:** Making a small change within an already-architected feature.

## Core Process

### Step 1: Module Structure

1. **Inspect the existing app and requested scope first.** Preserve a small single-module app when it meets the requirements. Use the following layer/feature structure only when justified by coupling, ownership or build evidence:

```
:app                          # Application, navigation, DI entry point
:feature:home                 # Home screen (UI + ViewModel)
:feature:profile              # Profile screen (UI + ViewModel)
:feature:settings             # Settings screen (UI + ViewModel)
:core:data                    # Repositories, data sources, API services
:core:domain                  # Use cases, domain models (pure Kotlin)
:core:database                # Room database, entities, DAOs
:core:network                 # Retrofit services, API models, interceptors
:core:ui                      # Shared composables, theme, design tokens
:core:common                  # Utilities, extensions, constants
:core:testing                 # Shared test utilities, fakes
```

2. **Dependency rules:**
   - `:feature:*` depends on `:core:domain`, `:core:ui`, `:core:common`
   - `:core:data` depends on `:core:domain`, `:core:database`, `:core:network`
   - `:core:domain` depends on **nothing** (pure Kotlin module)
   - `:app` depends on all `:feature:*` and `:core:data` (for DI wiring)
   - **Never:** feature → feature, core → feature, domain → data

### Step 2: Layer Separation

3. **Use the existing architecture as evidence.** The following three-layer structure is an option; do not create empty layers or use cases without a concrete need:

| Layer | Contains | Depends On |
|-------|----------|-----------|
| **UI** | Composables, ViewModels, UiState | Domain |
| **Domain** | Use cases, domain models, repository interfaces | Nothing |
| **Data** | Repository implementations, data sources, mappers | Domain (interfaces) |

4. **Data flows one direction:** UI ← Domain ← Data

```kotlin
// Domain layer: pure Kotlin, no Android dependencies
interface TaskRepository {
    fun getTasks(): Flow<List<Task>>
    suspend fun addTask(task: Task)
}

data class Task(val id: String, val title: String, val completed: Boolean)

class GetTasksUseCase @Inject constructor(
    private val repository: TaskRepository
) {
    operator fun invoke(): Flow<List<Task>> = repository.getTasks()
}
```

### Step 3: ViewModel and State Management

5. **MVVM with UiState pattern:**

```kotlin
// Sealed interface for exhaustive state handling
sealed interface TaskListUiState {
    data object Loading : TaskListUiState
    data class Success(val tasks: List<Task>) : TaskListUiState
    data class Error(val message: String) : TaskListUiState
}

@HiltViewModel
class TaskListViewModel @Inject constructor(
    private val getTasks: GetTasksUseCase
) : ViewModel() {

    val uiState: StateFlow<TaskListUiState> = getTasks()
        .map<List<Task>, TaskListUiState> { TaskListUiState.Success(it) }
        .catch { emit(TaskListUiState.Error(it.message ?: "Unknown error")) }
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5_000),
            initialValue = TaskListUiState.Loading
        )
}
```

6. **MVI for complex screens** (multiple user actions, side effects):

```kotlin
// Events from UI → ViewModel
sealed interface TaskListEvent {
    data class ToggleTask(val taskId: String) : TaskListEvent
    data class DeleteTask(val taskId: String) : TaskListEvent
    data object RefreshTasks : TaskListEvent
}

// One-time effects (navigation, snackbar)
sealed interface TaskListEffect {
    data class ShowSnackbar(val message: String) : TaskListEffect
    data class NavigateToDetail(val taskId: String) : TaskListEffect
}
```

### Step 4: Dependency Injection (Hilt example only)

7. **Keep the existing DI approach, including manual DI.** Add or migrate a framework only within an explicit requested scope and with project evidence. The following module setup applies only when Hilt is already selected:

```kotlin
// In :core:data
@Module
@InstallIn(SingletonComponent::class)
abstract class DataModule {
    @Binds
    abstract fun bindTaskRepository(impl: TaskRepositoryImpl): TaskRepository
}

// In :core:network
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    @Provides
    @Singleton
    fun provideRetrofit(): Retrofit = Retrofit.Builder()
        .baseUrl(BuildConfig.API_BASE_URL)
        .addConverterFactory(Json.asConverterFactory("application/json".toMediaType()))
        .build()
}
```

8. **Rules when Hilt is selected:**
   - Use `@Inject constructor` (not field injection)
   - Scope to the narrowest lifecycle: `@Singleton` only for truly app-wide instances
   - Use `@Binds` for interface→implementation, `@Provides` for third-party classes
   - Test with `@UninstallModules` + `@BindValue` or Hilt test rules

### Step 5: Coroutines and Flow

9. **Structured concurrency:**

```kotlin
// Use viewModelScope — never GlobalScope
viewModelScope.launch {
    repository.syncTasks()
}

// Use withContext for thread switching
suspend fun readFromDisk(): Data = withContext(Dispatchers.IO) {
    file.readText().parseData()
}

// Expose Flow from repositories, collect in ViewModels
fun getTasks(): Flow<List<Task>> = taskDao.observeAll()
    .map { entities -> entities.map { it.toDomain() } }
```

10. **Error handling:**

```kotlin
// Catch in the ViewModel, not in the repository
viewModelScope.launch {
    try {
        repository.syncTasks()
    } catch (e: IOException) {
        _uiState.update { it.copy(error = "Network unavailable") }
    }
}
```

## Common Rationalizations

| Shortcut | Why It Fails |
|----------|-------------|
| Small app without modules | Keep its structure unless measured coupling, ownership or build needs justify modularization. |
| Use-case layer | Introduce use cases when shared or complex domain behavior benefits; do not add boilerplate to meet a template. |
| Logic placement | Keep the existing architecture when suitable. Extract shared or complex domain logic into use cases only when a concrete need justifies them; a small app need not add a domain layer. |
| Manual DI | A valid choice, including small existing apps; change only for demonstrated project needs and requested scope. |
| "GlobalScope is fine for fire-and-forget" | GlobalScope survives activity destruction, leaking resources. Use structured concurrency. |

## Red Flags

- Feature module depends on another feature module
- A domain layer intended to be pure Kotlin imports Android framework classes
- ViewModel directly accesses Room DAOs or Retrofit services
- `GlobalScope.launch` anywhere in production code
- Field injection (`@Inject lateinit var`) instead of constructor injection
- Mutable state exposed from ViewModel (`MutableStateFlow` made public)
- Business logic in Composables

## Verification

- [ ] Module dependencies follow the chosen project architecture; apply the illustrated layer rules only when that architecture is selected
- [ ] If a pure domain layer is part of the chosen architecture, verify it has no Android dependencies; do not introduce a domain layer solely for this check
- [ ] ViewModels expose immutable state according to the existing stack; migrate LiveData only when justified and requested
- [ ] Dependencies follow the selected project DI approach; constructor injection can be manual or framework-assisted
- [ ] No `GlobalScope` usage in production code
- [ ] Repositories return `Flow` for observable data
- [ ] Where the chosen architecture uses use cases for concrete needs, verify they are independently testable; no use-case layer is required otherwise
- [ ] `./gradlew build` succeeds with no circular dependency errors
