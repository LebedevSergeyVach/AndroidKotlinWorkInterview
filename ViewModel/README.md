# ViewModel
<a name="up"></a>

---

**ViewModel** — это одна из ключевых компонентов архитектуры `Android`, которая помогает управлять данными, связанными с `UI`, и переживать изменения конфигурации (например, поворот экрана).

### Основные задачи ViewModel:

 - **Хранение данных** - `ViewModel` хранит данные, связанные с `UI`, и предоставляет их `Activity` или `Fragment`.
 - **Переживание изменений конфигурации** - `ViewModel` сохраняет данные при повороте экрана или других изменениях конфигурации.
 - **Разделение ответственности** - `ViewModel` отделяет логику управления данными от `UI`, что делает код более поддерживаемым и тестируемым.


---

## Жизненный цикл ViewModel

**ViewModel** связана с жизненным циклом `Activity` или `Fragment`.
Она создается, когда `Activity` или Fragment создаются, и уничтожается, когда они окончательно завершают свою работу (например, когда `Activity` закрывается).

Жизненный цикл ViewModel:

 - **Создание** - Когда `Activity` или `Fragment` создаются.
 - **Использование** - Пока `Activity` или `Fragment` активны.
 - **Уничтожение** - Когда `Activity` или `Fragment` завершают свою работу.

---

## ViewModel и LiveData

**LiveData** — это `observable`-объект, который позволяет наблюдать за изменениями данных. 
Он автоматически учитывает жизненный цикл компонентов (например, `Activity` или `Fragment`).

### Пример использования LiveData в ViewModel

```kotlin
import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel

class MyViewModel : ViewModel() {
    private val _data = MutableLiveData<String>()
    val data: LiveData<String> get() = _data

    fun updateData(newData: String) {
        _data.value = newData
    }
}
```

### Наблюдение за LiveData в Activity

```kotlin
viewModel.data.observe(this, Observer { newData ->
    // Обновление UI
    textView.text = newData
})
```

---

## ViewModel и Flow

**Flow** — это асинхронный поток данных из библиотеки `Kotlin Coroutines`. Он позволяет обрабатывать последовательности данных, которые могут изменяться со временем. 
`Flow` похож на `LiveData`, но предоставляет больше гибкости и возможностей для работы с асинхронными данными.



---

## ViewModel и Kotlin Coroutines

`ViewModel` поддерживает корутины через `viewModelScope`. 
Это позволяет выполнять асинхронные операции, которые автоматически отменяются при уничтожении `ViewModel`.

### Преимущества использования Flow в ViewModel:

 - **Асинхронность** - `Flow` работает с корутинами, что делает его идеальным для асинхронных операций.
 - **Гибкость** - `Flow` поддерживает множество операторов (например, `map`, `filter`, `flatMap`), которые позволяют трансформировать данные.
 - **Реактивность** - `Flow` автоматически обновляет данные при их изменении.
 - **Интеграция с Jetpack Compose** - `Flow` легко интегрируется с `Jetpack Compose` через `collectAsState`.

### Использование StateFlow

**StateFlow** — это специальный тип `Flow`, который хранит текущее состояние и автоматически обновляет подписчиков при его изменении.

### Использование SharedFlow

**SharedFlow** — это поток, который может иметь несколько подписчиков и поддерживает кэширование данных.

### Наблюдение за Flow в UI

Для наблюдения за `Flow` в `UI` (например, в `Activity` или `Fragment`) используйте `lifecycleScope` или `repeatOnLifecycle`.

### Преобразование Flow

`Flow` поддерживает множество операторов для трансформации данных. Например, `map`, `filter`, `flatMap`.

---

## ViewModel и Dependency Injection

`Hilt` упрощает внедрение зависимостей в `ViewModel` - `@HiltViewModel`.

---

## Пример `ViewModel`, `State`, и `Fragment`

### Status

```kotlin
/**
 * Интерфейс для описания состояния загрузки данных.
 *
 * Этот интерфейс используется для представления различных состояний загрузки данных:
 * - `Idle`: Данные не загружаются.
 * - `Loading`: Данные загружаются.
 * - `Error`: Произошла ошибка при загрузке данных.
 *
 * @property throwableOrNull Возвращает исключение, если состояние `Error`, иначе `null`.
 */
interface StatusLoad {
    val throwableOrNull: Throwable?
        get() = (this as? Error)?.exception

    /**
     * Состаяние, когда загрузка произошла успешно (например данные авторизации коректны).
     */
    data object Success : StatusLoad

    /**
     * Состояние, когда данные не загружаются.
     */
    data object Idle : StatusLoad

    /**
     * Состояние, когда данные загружаются.
     */
    data object Loading : StatusLoad

    /**
     * Состояние, когда произошла ошибка при загрузке данных.
     *
     * @property exception Исключение, вызвавшее ошибку.
     */
    data class Error(val exception: Exception) : StatusLoad
}

```

### State

```kotlin
/**
 * Состояние пользователей в ViewModel.
 *
 * @property users Список пользователей [UserData]. По умолчанию пустой список.
 * @property statusUser Статус загрузки или ошибки [StatusUser]. По умолчанию Idle.
 */
data class UserState(
    val users: List<UserData>? = null,
    val statusUser: StatusLoad = StatusLoad.Idle,
) {
    /**
     * Флаг, указывающий на то, что данные обновляются (рефреш).
     */
    val isRefreshing: Boolean
        get() = statusUser == StatusLoad.Loading && users?.isNotEmpty() == true

    /**
     * Флаг, указывающий на то, что данные загружаются, но список пользователей пуст.
     */
    val isEmptyLoading: Boolean
        get() = statusUser == StatusLoad.Loading && users.isNullOrEmpty()

    /**
     * Флаг, указывающий на ошибку при обновлении данных, если список пользователей не пуст.
     */
    val isRefreshError: Boolean
        get() = statusUser is StatusLoad.Error && users?.isNotEmpty() == true

    /**
     * Флаг, указывающий на ошибку при обновлении данных, если список пользователей пуст.
     */
    val isEmptyError: Boolean
        get() = statusUser is StatusLoad.Error && users.isNullOrEmpty()
}

```

### ViewModel

```kotlin
/**
 * ViewModel для управления состоянием пользователей.
 *
 * Этот ViewModel отвечает за загрузку данных пользователей и управление их состоянием.
 *
 * @param repository Репозиторий, который предоставляет данные о пользователях.
 * @param userId Идентификатор пользователя.
 *
 * @see UserRepository Интерфейс репозитория, который используется в этом ViewModel.
 * @see UserState Состояние, которое управляется этим ViewModel.
 */
@HiltViewModel(assistedFactory = UserViewModel.ViewModelFactory::class)
class UserViewModel @AssistedInject constructor(
    private val repository: UserRepository,
    @Assisted private val userId: Long,
) : ViewModel() {

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

    init {
        getUserById(userId = userId)
    }

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

    /**
     * Обрабатывает ошибку и сбрасывает состояние загрузки.
     */
    fun consumerError() {
        _state.update { stateUser: UserState ->
            stateUser.copy(
                statusUser = StatusLoad.Idle
            )
        }
    }

    /**
     * Вызывается при очистке ViewModel.
     *
     * Этот метод освобождает все ресурсы, связанные с корутинами.
     * Он вызывается, когда ViewModel больше не используется и будет уничтожено.
     *
     * @see viewModelScope
     */
    override fun onCleared() {
        viewModelScope.cancel()
    }

    @AssistedFactory
    interface ViewModelFactory {
        fun create(userId: Long = 0L): UserViewModel
    }
}
```

### Fragment

```kotlin
/**
 * ViewModel для управления данными о пользователе.
 * Создается с использованием фабрики, которая принимает идентификатор пользователя.
 *
 * @see UserViewModel ViewModel, который управляет данными о пользователе.
 * @see viewModels Делегат для получения ViewModel, привязанного к фрагменту.
 */
val userViewModel by viewModels<UserViewModel>(
    extrasProducer = {
        defaultViewModelCreationExtras.withCreationCallback<UserViewModel.ViewModelFactory> { factory ->
            factory.create(userId = userId)
        }
    }
)
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

```kotlin
/**
 * Загружает данные о пользователе, его постах, событиях и местах работы из ViewModel.
 * Используется для инициализации данных при создании фрагмента или обновлении данных.
 *
 * @param userId Идентификатор пользователя, для которого загружаются данные.
 * @param userViewModel ViewModel, связанная с данными о пользователе.
 * @param postViewModel ViewModel для управления постами пользователя.
 * @param eventViewModel ViewModel для управления событиями пользователя.
 * @param jobViewModel ViewModel для управления местами работы пользователя.
 * @param causeVibration Вызов вибрации (по умолчанию = true).
 *
 * @see UserViewModel.getUserById
 * @see PostWallViewModel.accept
 * @see EventWallViewModel.loadEventsByAuthor
 * @see JobViewModel.load
 */
private fun loadingDataFromTheViewModel(
    userId: Long,
    userViewModel: UserViewModel,
    postViewModel: PostWallViewModel,
    eventViewModel: EventWallViewModel,
    jobViewModel: JobViewModel,
    causeVibration: Boolean = true
) {
    if (causeVibration) requireContext().singleVibrationWithSystemCheck(35)

    userViewModel.getUserById(userId = userId)
    postViewModel.accept(message = PostWallMessage.Refresh)
    eventViewModel.loadEventsByAuthor(authorId = userId)
    jobViewModel.getJobsByUserId(userId = userId)
}
```

### Дополнительные примеры

```kotlin
/**
 * Класс, представляющий состояние постов в приложении.
 * Этот класс хранит список постов, текущий статус загрузки и информацию об ошибках.
 *
 * @property posts Список постов, отображаемых в UI.
 * @property statusPost Текущий статус загрузки постов.
 * @property singleError Исключение, которое произошло при выполнении операции (например, лайк или удаление).
 */
data class PostState(
    val posts: List<PostUiModel> = emptyList(),
    val statusPost: PostStatus = PostStatus.Idle(),
    val singleError: Throwable? = null,
) {

    /**
     * Флаг, указывающий, произошла ли ошибка при начальной загрузке постов.
     *
     * @return `true`, если статус загрузки — [PostStatus.EmptyError], иначе `false`.
     */
    val isEmptyError: Boolean = statusPost is PostStatus.EmptyError

    /**
     * Флаг, указывающий, выполняется ли в данный момент обновление списка постов.
     *
     * @return `true`, если статус загрузки — [PostStatus.Refreshing], иначе `false`.
     */
    val isRefreshing: Boolean = statusPost == PostStatus.Refreshing

    /**
     * Исключение, которое произошло при начальной загрузке постов.
     *
     * @return Исключение, если статус загрузки — [PostStatus.EmptyError], иначе `null`.
     */
    val emptyError: Throwable? = (statusPost as? PostStatus.EmptyError)?.reason

    /**
     * Флаг, указывающий, выполняется ли в данный момент начальная загрузка постов, и список постов пуст.
     *
     * @return `true`, если статус загрузки — [PostStatus.EmptyLoading], иначе `false`.
     */
    val isEmptyLoading: Boolean = statusPost == PostStatus.EmptyLoading
}
```

```kotlin
/**
 * ViewModel для управления состоянием постов.
 * Этот класс взаимодействует с [PostStore] для получения состояния постов и обработки сообщений,
 * которые изменяют это состояние.
 *
 * @property postStore Хранилище (Store), которое управляет состоянием постов и эффектами.
 * @see ViewModel
 */
@HiltViewModel
class PostViewModel @Inject constructor(
    private val postStore: PostStore,
) : ViewModel() {

    /**
     * Публичный Flow, который предоставляет доступ к текущему состоянию постов.
     *
     * @see PostState Состояние, которое предоставляется этим Flow.
     */
    val state: StateFlow<PostState> = postStore.state

    /**
     * Инициализатор ViewModel.
     * При создании ViewModel запускается подключение к хранилищу (Store) для обработки сообщений и эффектов.
     */
    init {
        viewModelScope.launch {
            postStore.connect()
        }
    }

    /**
     * Принимает сообщение и передает его в хранилище (Store) для обработки.
     * Этот метод используется для отправки сообщений, которые изменяют состояние постов,
     * таких как загрузка постов, лайки, удаление и т.д.
     *
     * @param message Сообщение, которое нужно обработать.
     * @see PostMessage Типы сообщений, которые могут быть отправлены в хранилище.
     */
    fun accept(message: PostMessage) {
        postStore.accept(message)
    }

    /**
     * Вызывается при очистке ViewModel.
     *
     * Этот метод освобождает все ресурсы, связанные с корутинами.
     * Он вызывается, когда ViewModel больше не используется и будет уничтожено.
     *
     * @see viewModelScope
     */
    override fun onCleared() {
        viewModelScope.cancel()
    }
}
```

```kotlin
/**
 * Настраивает наблюдение за состоянием [ViewModel] и обновляет UI в соответствии с текущим состоянием.
 * Метод связывает [PostState] с элементами UI, такими как [ProgressBar], [SwipeRefreshLayout] и [RecyclerView].
 *
 * @param binding [FragmentPostsBinding] - объект binding, который предоставляет доступ к элементам UI.
 * @param adapter [PostAdapter] - адаптер, который управляет отображением списка постов.
 *
 * @see [PostState] - класс, который представляет состояние экрана с постами.
 * @see [PostMessage] - класс, который представляет сообщения, отправляемые во ViewModel.
 * @see [PostPagingMapper] - класс, который преобразует состояние в список данных для адаптера
 */
private fun postViewModelState(
    binding: FragmentPostsBinding,
    adapter: PostAdapter,
) {
    viewModel.state
        .flowWithLifecycle(viewLifecycleOwner.lifecycle)
        .onEach { postState: PostState ->
            binding.errorGroup.isVisible = postState.isEmptyError

            val errorText: CharSequence? =
                postState.emptyError?.getErrorText(requireContext())

            binding.errorText.text = errorText

            binding.progressBar.isVisible = postState.isEmptyLoading

            binding.swiperRefresh.isRefreshing = postState.isRefreshing

            if (postState.singleError != null) {
                val singleErrorText =
                    postState.singleError.getErrorText(requireContext())

                requireContext().toast(singleErrorText.toString())

                viewModel.accept(message = PostMessage.HandleError)
            }

            adapter.submitList(
                PostPagingMapper.map(
                    state = postState,
                    context = requireContext()
                )
            )
        }
        .launchIn(viewLifecycleOwner.lifecycleScope)
}

/**
 * Прокручивает RecyclerView на самый верх и обновляет данные.
 */
private fun scrollToTopAndRefresh(binding: FragmentPostsBinding) {
    viewModel.accept(message = PostMessage.Refresh)
    binding.list.smoothScrollToPosition(0)
}
```

---

## Кратко


**ViewModel** — Компонент для управления данными, связанными с UI.

### Основные задачи:

 - Хранение данных.
 - Переживание изменений конфигурации.
 - Разделение логики и `UI`.

### Использование:

 - Создайте класс, наследуемый от `ViewModel`.
 - Используйте `ViewModelProvider` для получения экземпляра.

**LiveData** — Для наблюдения за изменениями данных.
**Flow** — Асинхронный поток данных, который идеально подходит для работы с `ViewModel`.
**StateFlow** — Хранит текущее состояние и автоматически обновляет подписчиков.
**SharedFlow** — Поддерживает несколько подписчиков и кэширование данных.
**Корутины** — Для асинхронных операций через `viewModelScope`.
**SavedStateHandle** — Для сохранения данных при изменении конфигурации.
**Hilt** — Для внедрения зависимостей в `ViewModel`.

---

#### [README](README.md) [UP](#up)
