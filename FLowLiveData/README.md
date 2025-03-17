# Flow, LiveData и State.
<a name="up"></a>

---

## Flow

**Flow** — это асинхронный поток данных из библиотеки `Kotlin Coroutines`. Он позволяет обрабатывать последовательности данных, которые могут изменяться со временем.
`Flow` похож на `LiveData`, но предоставляет больше гибкости и возможностей для работы с асинхронными данными.

### Основные особенности Flow:

 - **Асинхронность** - `Flow` работает с корутинами, что делает его идеальным для асинхронных операций.
 - **Гибкость** - `Flow` поддерживает множество операторов (например, `map`, `filter`, `flatMap`), которые позволяют трансформировать данные.
 - **Холодный поток** - `Flow` является "холодным" потоком, то есть данные начинают поступать только после вызова `collect`.

```kotlin
/**
 * Flow, хранящий текущее состояние пользователей.
 *
 * @see UserState Состояние, которое хранится в этом Flow.
 */
private val _state: MutableStateFlow<UserState> = MutableStateFlow(UserState())

/**
 * Публичный Flow, который предоставляет доступ к текущему состоянию пользователей.
 *
 * @see UserState Состояние, которое предоставляется этим Flow.
 */
val state: StateFlow<UserState> = _state.asStateFlow()

/**
 * Загружает пользователя по его идентификатору.
 *
 * @param userId Идентификатор пользователя, которого нужно загрузить.
 */
fun getUserById(userId: Long) {
    _state.update { stateUser: UserState ->
        stateUser.copy(
            statusUser = StatusLoad.Loading
        )
    }

    viewModelScope.launch {
        try {
            val user: UserData = repository.getUserById(userId = userId)

            _state.update { stateUser: UserState ->
                stateUser.copy(
                    statusUser = StatusLoad.Idle,
                    users = listOf(user)
                )
            }
        } catch (e: Exception) {
            _state.update { stateUser: UserState ->
                stateUser.copy(
                    statusUser = StatusLoad.Error(exception = e)
                )
            }
        }
    }
}
```

```kotlin
/**
 * Наблюдает за состоянием ViewModel пользователя и обновляет UI в зависимости от текущего состояния.
 * Этот метод связывает состояние ViewModel с элементами интерфейса, такими как ProgressBar, SwipeRefreshLayout,
 * TextView для ошибок, а также отображает данные пользователя (аватар, имя и т.д.).
 *
 * @param userViewModel Экземпляр [UserViewModel], который предоставляет состояние пользователя.
 * @param binding Экземпляр [FragmentAccountBinding], используемый для доступа к элементам интерфейса.
 * @param userId Идентификатор пользователя, используемый для проверки и отображения данных.
 *
 * @see UserState Состояние пользователя, которое содержит данные о загрузке, ошибках и информации о пользователе.
 * @see FragmentAccountBinding Связывает элементы интерфейса с кодом.
 * @see UserViewModel ViewModel, управляющая состоянием пользователя.
 *
 * @throws NullPointerException Если контекст или ресурсы недоступны.
 *
 * @property userState Текущее состояние пользователя, которое может быть:
 * - [UserState.isEmptyLoading] — состояние загрузки.
 * - [UserState.isRefreshing] — состояние обновления.
 * - [UserState.isEmptyError] — состояние ошибки.
 * - [UserState.users] — данные пользователя.
 */
private fun userViewModelState(
    userViewModel: UserViewModel,
    binding: FragmentAccountBinding,
    userId: Long,
    accountUserId: Long,
) {
    userViewModel.state
        .flowWithLifecycle(viewLifecycleOwner.lifecycle)
        .onEach { userState: UserState ->
            binding.progressBar.isVisible = userState.isEmptyLoading
            binding.swiperRefresh.isRefreshing = userState.isRefreshing
            binding.errorGroup.isVisible = userState.isEmptyError

            binding.avatarUser.isVisible = !userState.isEmptyError && !userState.isEmptyLoading
            binding.initial.isVisible = !userState.isEmptyError && !userState.isEmptyLoading
            binding.nameUser.isVisible = !userState.isEmptyError && !userState.isEmptyLoading

            binding.tabLayout.isVisible = !userState.isEmptyError && !userState.isEmptyLoading
            binding.viewPagerPostsAndEvents.isVisible =
                !userState.isEmptyError && !userState.isEmptyLoading

            val errorText: CharSequence? =
                userState.statusUser.throwableOrNull?.getErrorText(requireContext())

            binding.errorText.text = errorText

            if (userState.isRefreshError && errorText == getString(R.string.network_error)) {
                requireContext().toast(R.string.network_error)

                userViewModel.consumerError()
            } else if (userState.isRefreshError && errorText == getString(R.string.unknown_error)) {
                requireContext().toast(R.string.unknown_error)

                userViewModel.consumerError()
            }

            userState.users?.firstOrNull()?.let { user: UserData ->
                binding.nameUser.text = user.name

                binding.skeletonAttachment.showSkeleton()

                if (!user.avatar.isNullOrEmpty()) {
                    Glide.with(binding.root)
                        .load(user.avatar)
                        .diskCacheStrategy(DiskCacheStrategy.ALL)
                        .listener(object : RequestListener<Drawable> {
                            override fun onLoadFailed(
                                e: GlideException?,
                                model: Any?,
                                target: Target<Drawable>,
                                isFirstResource: Boolean
                            ): Boolean {
                                showPlaceholder(binding, user)

                                return false
                            }

                            override fun onResourceReady(
                                resource: Drawable,
                                model: Any,
                                target: Target<Drawable>?,
                                dataSource: DataSource,
                                isFirstResource: Boolean
                            ): Boolean {
                                binding.skeletonAttachment.showOriginal()
                                binding.initial.isVisible = false

                                return false
                            }
                        })
                        .transition(DrawableTransitionOptions.withCrossFade(500))
                        .error(R.drawable.ic_404_24)
                        .thumbnail(
                            Glide.with(binding.root)
                                .load(user.avatar)
                                .override(50, 50)
                                .diskCacheStrategy(DiskCacheStrategy.ALL)
                        )
                        .into(binding.avatarUser)
                } else {
                    showPlaceholder(binding, user)
                }

                if (userId != accountUserId) {
                    val toolbar = requireActivity().findViewById<Toolbar>(R.id.toolbar)

                    toolbar.title = user.login
                }
            }
        }
        .launchIn(viewLifecycleOwner.lifecycleScope)
}
```

---

## LiveData

**LiveData** — это компонент архитектуры `Android`, который позволяет наблюдать за изменениями данных в реальном времени.
Он особенно полезен для работы с `UI`, так как автоматически обновляет данные при изменении и учитывает жизненный цикл компонентов (например, `Activity` или `Fragment`).

### Основные особенности `LiveData`:

 - **Наблюдение за данными** - `LiveData` позволяет подписаться на изменения данных и автоматически обновлять `UI`.
 - **Учет жизненного цикла** - `LiveData` автоматически управляет подписками, учитывая жизненный цикл компонентов (например, отписывается при уничтожении `Activity`).
 - **Потокобезопасность** - `LiveData` гарантирует, что данные будут обновлены в главном потоке.

```kotlin
/**
 * ViewModel для обмена данными между фрагментами.
 *
 * Этот ViewModel используется для хранения и передачи данных между фрагментами, которые не имеют прямого
 * взаимодействия друг с другом. В частности, он используется для отслеживания текущей вкладки в `ViewPager2`
 * и передачи этой информации в `BottomNavigationFragment` для управления поведением кнопки создания нового поста или события.
 *
 * @property currentTab LiveData, содержащая текущую выбранную вкладку в `ViewPager2`.
 *                     Значение 0 соответствует вкладке с постами, а значение 1 — вкладке с событиями.
 *                     По умолчанию значение не установлено.
 *
 * @see ViewModel Базовый класс для ViewModel, который сохраняет данные при изменении конфигурации.
 * @see MutableLiveData Класс для хранения данных, которые могут изменяться и наблюдаться.
 *
 * @throws IllegalArgumentException Может быть выброшено, если передано некорректное значение для `currentTab`.
 */
class SharedViewModel : ViewModel() {
    val currentTab = MutableLiveData<Int>()
}
```

```kotlin
private val sharedViewModel: SharedViewModel by activityViewModels()

sharedViewModel.currentTab.observe(viewLifecycleOwner) { tabPosition ->
    when (tabPosition) {
        0 -> binding.news.setOnClickListener(postsClickListener)
        1 -> binding.news.setOnClickListener(eventsClickListener)
        2 -> binding.news.setOnClickListener(jobsClickListener)
    }
}
```

```kotlin
binding.viewPagerPostsAndEvents.registerOnPageChangeCallback(
    object : ViewPager2.OnPageChangeCallback() {
        override fun onPageSelected(position: Int) {
            super.onPageSelected(position)

            sharedViewModel.currentTab.value = position
        }
    }
)
```

```kotlin
class MyViewModel : ViewModel() {
    private val _data = MutableLiveData<String>()
    val data: LiveData<String> get() = _data

    fun updateData(newData: String) {
        _data.value = newData
    }
}

class MyActivity : AppCompatActivity() {
    private val viewModel: MyViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        viewModel.data.observe(this, Observer { newData ->
            // Обновление UI
            textView.text = newData
        })

        // Обновление данных
        viewModel.updateData("Hello, LiveData!")
    }
}
```

---

## State

**State** — это концепция, которая используется для управления состоянием `UI` в J`etpack Compose`.
`State` позволяет автоматически обновлять `UI` при изменении данных.

### Основные особенности State:

 - **Реактивность** - `State` автоматически обновляет `UI` при изменении данных.
 - **Интеграция с Compose** - `State` используется в `Jetpack Compose` для управления состоянием компонентов.
 - **Простота** - `State` легко использовать для управления простыми состояниями.

---

## Сравнение Flow и LiveData

### **1. Асинхронность**
- **LiveData:**  
  Работает только в главном потоке. Не поддерживает асинхронные операции напрямую.
- **Flow:**  
  Полностью асинхронный, работает с корутинами. Подходит для сложных асинхронных операций.

### **2. Учет жизненного цикла**
- **LiveData:**  
  Автоматически учитывает жизненный цикл компонентов (например, `Activity` или `Fragment`).
- **Flow:**  
  Требует использования `lifecycleScope` или `viewModelScope` для учета жизненного цикла.

### **3. Гибкость**
- **LiveData:**  
  Ограничен в возможностях трансформации данных.
- **Flow:**  
  Поддерживает множество операторов (например, `map`, `filter`, `flatMap`), что делает его более гибким.

### **4. Холодный vs Горячий поток**
- **LiveData:**  
  Горячий поток — данные доступны сразу после создания.
- **Flow:**  
  Холодный поток — данные начинают поступать только после вызова `collect`.

### **5. Использование в `Jetpack Compose`**
- **LiveData:**  
  Не интегрируется напрямую с `Jetpack Compose`.
- **Flow:**  
  Легко интегрируется с `Jetpack Compose` через `collectAsState`.

### **Когда использовать `LiveData`?**
- Для простых сценариев, где нужно обновлять `UI` при изменении данных.
- Когда требуется автоматическое управление жизненным циклом.
- Для простых сценариев, где нужно обновлять UI при изменении данных.
- Когда требуется автоматическое управление жизненным циклом.

### **Когда использовать `Flow`?**
- Для сложных асинхронных операций.
- Когда требуется гибкость в трансформации данных.
- В `Jetpack Compose` для управления состоянием.
- Для сложных асинхронных операций.
- Когда требуется гибкость в трансформации данных.
- В Jetpack Compose для управления состоянием.

### Таблица

| **Критерий**                     | **LiveData**                                                             | **Flow**                                                                  |
|----------------------------------|--------------------------------------------------------------------------|---------------------------------------------------------------------------|
| **Асинхронность**                | Работает только в главном потоке.                                        | Полностью асинхронный, работает с корутинами.                             |
| **Учет жизненного цикла**        | Автоматически учитывает жизненный цикл (Activity, Fragment).             | Требует использования `lifecycleScope` или `viewModelScope`.              |
| **Гибкость**                     | Ограничен в возможностях трансформации данных.                           | Поддерживает множество операторов (`map`, `filter`, `flatMap` и др.).     |
| **Тип потока**                   | Горячий поток (данные доступны сразу).                                   | Холодный поток (данные начинают поступать после вызова `collect`).        |
| **Потокобезопасность**           | Гарантирует обновление данных в главном потоке.                          | Требует явного указания диспетчера (например, `Dispatchers.Main`).        |
| **Интеграция с Jetpack Compose** | Не интегрируется напрямую.                                               | Легко интегрируется через `collectAsState`.                               |
| **Использование**                | Подходит для простых сценариев с автоматическим учетом жизненного цикла. | Подходит для сложных асинхронных операций и интеграции с Jetpack Compose. |

---

## **Краткое заключение**

1. **LiveData** — Простой и удобный способ наблюдать за данными с учетом жизненного цикла.
2. **Flow** — Гибкий и мощный инструмент для работы с асинхронными данными.
3. **State** — Используется в `Jetpack Compose` для управления состоянием `UI`.
4. **Сравнение:**
    - **LiveData:** Подходит для простых сценариев с автоматическим учетом жизненного цикла.
    - **Flow:** Подходит для сложных асинхронных операций и интеграции с `Jetpack Compose`.

---

#### [README](README.md) [UP](#up)
