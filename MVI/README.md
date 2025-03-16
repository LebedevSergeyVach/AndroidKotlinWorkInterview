# MVI (Model-View-Intent)
<a name="up"></a>

---

**MVI (Model-View-Intent)** — это архитектурный паттерн, который разделяет приложение на три основные части:

- **Model** - Состояние приложения.
- **View** - Отображение состояния.
- **Intent** - Действия пользователя, которые изменяют состояние.

### **MVI** основан на однонаправленном потоке данных:

- Пользователь совершает действие (`Intent`).
- Действие преобразуется в Message, которая изменяет `State`.
- `State` обновляет `View`.
- Побочные эффекты (`Effects`) обрабатываются отдельно.

### Преимущества MVI

- **Предсказуемость** - Однонаправленный поток данных делает поведение приложения более предсказуемым.
- **Тестируемость** - Каждый компонент (`State`, `Message`, `Reducer`) можно тестировать отдельно.
- **Масштабируемость** - Легко добавлять новые функции и изменять существующие.

---

## Model

**Model** — это комбинация трех основных компонентов:

- **State** - Текущее состояние приложения, экрана/фичи.
- **Message (Intent)** - Действия, которые могут изменить состояние, намерение пользователя/сервера.
- **Effect** - Побочные эффекты, которые возникают в результате изменения состояния, команда к `Model`.

Модель представляет собой источник истины (`single source of truth`) для всего приложения.
Она описывает, как приложение должно реагировать на действия пользователя и внешние события.

### Компоненты Model

#### **State (Состояние)**

**State** — это неизменяемое (`immutable`) представление текущего состояния приложения.
Оно описывает, что должно отображаться на экране и как приложение должно себя вести в данный момент.

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
) {}
```

#### **Message (Сообщение)**

**Message** — это действия или события, которые могут изменить состояние. 
Они представляют собой намерения пользователя (например, нажатие кнопки) или события системы (например, завершение сетевого запроса).


```kotlin
/**
 * Запечатанный интерфейс, представляющий сообщения, которые могут быть отправлены в хранилище (Store)
 * для изменения состояния постов. Сообщения используются для обработки различных действий, таких как
 * загрузка постов, лайки, удаление и обработка ошибок.
 */
sealed interface PostMessage {

    /**
     * Сообщение для загрузки следующей страницы постов.
     */
    data object LoadNextPage : PostMessage

    /**
     * Сообщение для обновления списка постов (например, при pull-to-refresh).
     */
    data object Refresh : PostMessage

    /**
     * Сообщение для обновления списка постов.
     */
    data object Retry : PostMessage

    /**
     * Сообщение для лайка или снятия лайка с поста.
     *
     * @property post Пост, который нужно лайкнуть или убрать лайк.
     */
    data class Like(val post: PostUiModel) : PostMessage

    /**
     * Сообщение для удаления поста.
     *
     * @property post Пост, который нужно удалить.
     */
    data class Delete(val post: PostUiModel) : PostMessage

    /**
     * Сообщение для обработки ошибки (например, сброса состояния ошибки).
     */
    data object HandleError : PostMessage

    /**
     * Сообщение, которое возникает при ошибке удаления поста.
     *
     * @property error Информация об ошибке, включая пост и исключение.
     */
    data class DeleteError(val error: PostWithError) : PostMessage

    /**
     * Сообщение, которое возникает после попытки лайкнуть пост.
     *
     * @property result Результат операции лайка, который может быть успешным или содержать ошибку.
     */
    data class LikeResult(val result: Either<PostWithError, PostUiModel>) : PostMessage

    /**
     * Сообщение, которое возникает после загрузки начальной страницы постов.
     *
     * @property result Результат загрузки, который может быть успешным или содержать ошибку.
     */
    data class InitialLoaded(val result: Either<Throwable, List<PostUiModel>>) : PostMessage

    /**
     * Сообщение, которое возникает после загрузки следующей страницы постов.
     *
     * @property result Результат загрузки, который может быть успешным или содержать ошибку.
     */
    data class NextPageLoaded(val result: Either<Throwable, List<PostUiModel>>) : PostMessage
}

```

#### **Effect (Побочный эффект)**

**Effect** — это побочные эффекты, которые не изменяют состояние напрямую, но могут вызывать дополнительные действия (например, навигация, показ `Toast`).

```kotlin
/**
 * Запечатанный интерфейс, представляющий различные эффекты (действия), которые могут быть выполнены
 * в контексте постов. Эффекты используются для обработки бизнес-логики, такой как загрузка постов,
 * лайки и удаление.
 */
sealed interface PostEffect {

    /**
     * Эффект для загрузки следующей страницы постов.
     *
     * @property id Идентификатор последнего поста, начиная с которого нужно загрузить следующую страницу.
     * @property count Количество постов, которые нужно загрузить.
     */
    data class LoadNextPage(val id: Long, val count: Int) : PostEffect

    /**
     * Эффект для загрузки начальной страницы постов.
     *
     * @property count Количество постов, которые нужно загрузить.
     */
    data class LoadInitialPage(val count: Int) : PostEffect

    /**
     * Эффект для лайка поста.
     *
     * @property post Пост, который нужно лайкнуть или убрать лайк.
     */
    data class Like(val post: PostUiModel) : PostEffect

    /**
     * Эффект для удаления поста.
     *
     * @property post Пост, который нужно удалить.
     */
    data class Delete(val post: PostUiModel) : PostEffect
}

```

---

- **Model** в **MVI** — это комбинация `State`, `Message` и `Effect`.
- **State** описывает текущее состояние приложения.
- **Message** — это действия, которые изменяют состояние.
- **Effect** — это побочные эффекты, которые не изменяют состояние напрямую.
- **Reducer** обновляет состояние на основе сообщений.
- **EffectHandler** обрабатывает побочные эффекты.

---

## Store

**Store** — это центральный компонент в `MVI`, который управляет состоянием приложения. Он:

- Хранит текущее состояние (`State`).
- Обрабатывает сообщения (`Message`) и обновляет состояние.
- Управляет побочными эффектами (`Effects`).

### Как работает Store?

- **Хранение состояния** - `Store` содержит текущее состояние приложения.
- **Обработка сообщений** - Когда приходит новое сообщение, `Store` вызывает `Reducer`, чтобы обновить состояние.
- **Обработка эффектов** - После обновления состояния `Store` может вызвать `EffectHandler` для обработки побочных эффектов.

```kotlin
/**
 * Класс, представляющий хранилище (Store) в архитектуре MVI. Хранилище управляет состоянием приложения
 * и обрабатывает сообщения, которые могут изменять состояние.
 *
 * @param reducer Редьюсер, который обрабатывает сообщения и обновляет состояние.
 * @param effectHandler Обработчик эффектов, который выполняет побочные эффекты.
 * @param initMessages Набор начальных сообщений, которые будут отправлены при запуске хранилища.
 * @param initState Начальное состояние приложения.
 */
class Store<State, Message, Effect>(
    private val reducer: Reducer<State, Effect, Message>,
    private val effectHandler: EffectHandler<Effect, Message>,
    private val initMessages: Set<Message> = emptySet(),
    initState: State,
) {

    /**
     * Приватный поток для хранения и обновления текущего состояния приложения.
     * Используется для управления состоянием в архитектуре MVI.
     *
     * @property _state Хранит текущее состояние приложения, которое может включать данные (например, список постов), статус загрузки, ошибки и другие параметры. Состояние обновляется через редьюсер в ответ на сообщения (Intents).
     * @see MutableStateFlow Поток, который позволяет обновлять и читать текущее состояние.
     * @see State Состояние приложения, которое может быть сериализовано и восстановлено.
     */
    private val _state = MutableStateFlow(initState)

    /**
     * Публичный поток для предоставления текущего состояния компонентам UI.
     * Компоненты UI могут подписаться на этот поток и обновлять интерфейс в соответствии с текущим состоянием.
     *
     * @property state Предоставляет доступ к текущему состоянию приложения. Является неизменяемым потоком данных, что предотвращает изменение состояния извне.
     * @see StateFlow Поток, который предоставляет доступ только для чтения к текущему состоянию.
     * @see State Состояние приложения, которое может быть сериализовано и восстановлено.
     */
    val state: StateFlow<State> = _state.asStateFlow()

    /**
     * Приватный поток для обработки пользовательских действий (Intents).
     * Используется для передачи намерений пользователя (например, лайк или удаление поста) в ViewModel,
     * где они обрабатываются и преобразуются в изменения состояния.
     *
     * @property messages Поток сообщений, которые могут быть отправлены пользователем. Сообщения обрабатываются асинхронно, что позволяет избежать блокировки основного потока.
     * @see MutableSharedFlow Поток, который позволяет отправлять и получать сообщения асинхронно.
     * @see Message Тип сообщений, которые могут быть отправлены пользователем.
     */
    private val messages = MutableSharedFlow<Message>(extraBufferCapacity = 64)

    /**
     * Приватный поток для выполнения побочных эффектов (например, загрузка данных с сервера, показ уведомлений).
     * Используется для выполнения действий, которые не связаны напрямую с изменением состояния, но необходимы
     * для корректной работы приложения.
     *
     * @property effects Поток побочных эффектов, которые могут быть выполнены. Эффекты обрабатываются асинхронно, что позволяет избежать блокировки основного потока.
     * @see MutableSharedFlow Поток, который позволяет отправлять и получать эффекты асинхронно.
     * @see Effect Тип побочных эффектов, которые могут быть выполнены.
     */
    private val effects = MutableSharedFlow<Effect>(extraBufferCapacity = 64)

    /**
     * Принимает сообщение и добавляет его в поток сообщений для обработки.
     *
     * @param message Сообщение, которое нужно обработать.
     */
    fun accept(message: Message) {
        messages.tryEmit(message)
    }

    /**
     * Запускает хранилище и начинает обработку сообщений и эффектов.
     */
    suspend fun connect() = coroutineScope {
        launch {
            effectHandler.connect(effects)
                .collect(messages::tryEmit)
        }
        launch {
            listOf(
                initMessages.asFlow(),
                messages,
            )
                .merge()
                .map { message: Message ->
                    reducer.reduce(_state.value, message)
                }
                .collect { reducerResult: ReducerResult<State, Effect> ->
                    _state.value = reducerResult.newState
                    reducerResult.effects.forEach(effects::tryEmit)
                }
        }
    }
}
```

```kotlin
/**
 * Псевдоним типа для хранилища (Store), специализированного для работы с постами.
 * Этот тип упрощает использование хранилища, связанного с состоянием постов, сообщениями и эффектами.
 *
 * @see Store Базовый класс хранилища, который управляет состоянием и эффектами.
 * @see PostState Состояние, связанное с постами.
 * @see PostMessage Сообщения, которые могут изменять состояние постов.
 * @see PostEffect Эффекты, которые выполняются в ответ на сообщения.
 */
typealias PostStore = Store<PostState, PostMessage, PostEffect>

```

---

## Reducer

**Reducer** — это функция, которая принимает текущее состояние и сообщение, а затем возвращает новое состояние и возможные эффекты.

Интерфейс описывает чистую функцию, которая принимает на вход текущий `State` и `Message`, а возвращает новый `State` и набор `Effect`.

- Принимает состояние и сообщение.
- Возвращает новое состояние и набор эффектов.

```kotlin
/**
 * Интерфейс для редьюсера. Редьюсер отвечает за обработку сообщений и обновление состояния приложения.
 *
 * @param State Тип состояния приложения.
 * @param Effect Тип эффекта, который может быть вызван в результате обработки сообщения.
 * @param Message Тип сообщения, которое будет обработано.
 */
interface Reducer<State, Effect, Message> {

    /**
     * Обрабатывает сообщение и возвращает новое состояние и набор эффектов, которые нужно выполнить.
     *
     * @param old Текущее состояние приложения.
     * @param message Сообщение, которое нужно обработать.
     * @return Результат обработки сообщения, содержащий новое состояние и набор эффектов.
     */
    fun reduce(old: State, message: Message): ReducerResult<State, Effect>
}
```

```kotlin
/**
 * Класс, представляющий результат работы редьюсера. Содержит новое состояние и набор эффектов,
 * которые нужно выполнить.
 *
 * @param newState Новое состояние приложения.
 * @param effects Набор эффектов, которые нужно выполнить.
 */
data class ReducerResult<State, Effect>(
    val newState: State,
    val effects: Set<Effect>,
) {

    /**
     * Альтернативный конструктор для создания результата с одним эффектом.
     *
     * @param newState Новое состояние приложения.
     * @param action Единственный эффект, который нужно выполнить.
     */
    constructor(newState: State, action: Effect? = null) : this(newState, setOfNotNull(action))
}
```

```kotlin
/**
 * Редьюсер для постов. Этот класс реализует интерфейс [Reducer] и отвечает за обработку сообщений,
 * связанных с постами, и обновление состояния приложения.
 *
 * @see Reducer Интерфейс, который реализует этот класс.
 * @see PostState Состояние, связанное с постами на стене пользователя.
 * @see PostMessage Сообщения, которые могут изменять состояние постов.
 * @see PostEffect Эффекты, которые выполняются в ответ на сообщения.
 */
class PostReducer @Inject constructor() : Reducer<PostState, PostEffect, PostMessage> {

    companion object {
        /**
         * Размер страницы для пагинации. Определяет, сколько постов загружается за один запрос.
         */
        const val PAGE_SIZE: Int = 10
    }

    /**
     * Обрабатывает сообщение и возвращает новое состояние и набор эффектов, которые нужно выполнить.
     *
     * @param old Текущее состояние приложения.
     * @param message Сообщение, которое нужно обработать.
     * @return Результат обработки сообщения, содержащий новое состояние и набор эффектов.
     */
    override fun reduce(
        old: PostState,
        message: PostMessage
    ): ReducerResult<PostState, PostEffect> =
        when (message) {
            is PostMessage.Delete -> ReducerResult(
                newState = old.copy(posts = old.posts.filter { post: PostUiModel ->
                    post.id != message.post.id
                }),

                action = PostEffect.Delete(message.post),
            )

            is PostMessage.DeleteError -> ReducerResult(
                newState = old.copy(
                    posts = buildList(old.posts.size + 1) {
                        val deletedPost: PostUiModel = message.error.post

                        addAll(old.posts.filter { postInState: PostUiModel ->
                            postInState.id > deletedPost.id
                        })

                        add(deletedPost)

                        addAll(old.posts.filter { postInState: PostUiModel ->
                            postInState.id < deletedPost.id
                        })
                    }
                )
            )

            PostMessage.HandleError -> ReducerResult(
                newState = old.copy(
                    singleError = null
                )
            )

            is PostMessage.InitialLoaded -> ReducerResult(
                newState = when (
                    val messageResult: Either<Throwable, List<PostUiModel>> = message.result
                ) {
                    is Either.Left -> {
                        if (old.posts.isNotEmpty()) {
                            old.copy(
                                singleError = messageResult.value,
                                statusPost = PostStatus.Idle(),
                            )
                        } else {
                            old.copy(
                                statusPost = PostStatus.EmptyError(reason = messageResult.value)
                            )
                        }
                    }

                    is Either.Right -> old.copy(
                        posts = messageResult.value,
                        statusPost = PostStatus.Idle(),
                    )
                }
            )

            is PostMessage.Like -> ReducerResult(
                newState = old.copy(
                    posts = old.posts.map { post: PostUiModel ->
                        if (post.id == message.post.id) {
                            post.copy(
                                likedByMe = !post.likedByMe,
                                likes = if (post.likedByMe) post.likes - 1 else post.likes + 1
                            )
                        } else {
                            post
                        }
                    }
                ),

                action = PostEffect.Like(message.post)
            )

            is PostMessage.LikeResult -> ReducerResult(
                newState = when (val result = message.result) {
                    is Either.Left -> {
                        val value: PostWithError = result.value
                        val postLiked: PostUiModel = value.post

                        old.copy(
                            posts = old.posts.map { post: PostUiModel ->
                                if (post.id == postLiked.id) {
                                    postLiked
                                } else {
                                    post
                                }
                            },

                            singleError = value.throwable,
                        )
                    }

                    is Either.Right -> {
                        val postLiked: PostUiModel = result.value

                        old.copy(
                            posts = old.posts.map { post: PostUiModel ->
                                if (post.id == postLiked.id) {
                                    postLiked
                                } else {
                                    post
                                }
                            }
                        )
                    }
                }
            )

            PostMessage.LoadNextPage -> {
                val loadingFinished: Boolean =
                    (old.statusPost as? PostStatus.Idle)?.loadingFinished == true

                val status: PostStatus =
                    if (loadingFinished || old.statusPost !is PostStatus.Idle) {
                        old.statusPost
                    } else {
                        PostStatus.NextPageLoading
                    }

                val effect: PostEffect.LoadNextPage? = if (loadingFinished) {
                    null
                } else {
                    PostEffect.LoadNextPage(
                        id = old.posts.last().id,
                        count = PAGE_SIZE
                    )
                }

                ReducerResult(
                    newState = old.copy(
                        statusPost = status
                    ),

                    action = effect
                )
            }

            is PostMessage.NextPageLoaded -> ReducerResult(
                newState = when (val messageResult = message.result) {
                    is Either.Right -> {
                        val postUiModels: List<PostUiModel> = messageResult.value
                        val loadingFinished: Boolean = postUiModels.size < PAGE_SIZE

                        old.copy(
                            statusPost = PostStatus.Idle(loadingFinished = loadingFinished),
                            posts = old.posts + messageResult.value
                        )
                    }

                    is Either.Left -> {
                        if (BuildConfig.DEBUG) {
                            LoggerHelper.e("NextPageLoaded: statusPost = ${PostStatus.NextPageError(reason = messageResult.value)}")
                        }

                        old.copy(
                            statusPost = PostStatus.NextPageError(reason = messageResult.value)
                        )
                    }
                }
            )

            PostMessage.Retry -> {
                val nextId: Long? = old.posts.lastOrNull()?.id

                if (nextId == null) {
                    ReducerResult(old)
                } else {
                    ReducerResult(
                        newState = old.copy(
                            statusPost = PostStatus.NextPageLoading,
                        ),

                        action = PostEffect.LoadNextPage(
                            id = nextId,
                            count = PAGE_SIZE
                        )
                    )
                }
            }

            PostMessage.Refresh -> ReducerResult(
                newState = old.copy(
                    statusPost = if (old.posts.isNotEmpty()) {
                        PostStatus.Refreshing
                    } else {
                        PostStatus.EmptyLoading
                    }
                ),

                action = PostEffect.LoadInitialPage(count = PAGE_SIZE)
            )
        }
}
```

---

## EffectHandler

**EffectHandler** — это компонент, который обрабатывает побочные эффекты (например, навигация, показ `Toast`).

- Принимает эффект.
- Выполняет действие (например, показывает `Toast`).

Интерфейс для обработки всех сайд-эффектов. Принимает поток `Effect`, возвращает поток `Message`.

```kotlin
/**
 * Интерфейс для обработки эффектов. Эффекты представляют собой действия, которые должны быть выполнены
 * в ответ на сообщения, такие как загрузка данных, лайки и т.д.
 *
 * @param Effect Тип эффекта, который будет обрабатываться.
 * @param Message Тип сообщения, которое будет возвращено после обработки эффекта.
 */
interface EffectHandler<Effect, Message> {

    /**
     * Подключает поток эффектов к потоку сообщений.
     *
     * @param effects Поток эффектов, которые нужно обработать.
     * @return Поток сообщений, которые будут отправлены в хранилище (Store).
     */
    fun connect(effects: Flow<Effect>): Flow<Message>
}
```

```kotlin
/**
 * Обработчик эффектов для постов. Этот класс реализует интерфейс [EffectHandler] и отвечает за
 * обработку различных эффектов, таких как загрузка постов, лайки и удаление.
 *
 * @param repository Репозиторий для работы с постами в сети.
 * @param mapper Маппер для преобразования моделей данных в UI-модели.
 */
class PostEffectHandler @Inject constructor(
    private val repository: PostRepository,
    private val mapper: PostUiModelMapper,
) : EffectHandler<PostEffect, PostMessage> {

    /**
     * Подключает поток эффектов к потоку сообщений. Этот метод объединяет обработчики для всех типов эффектов.
     *
     * @param effects Поток эффектов, которые нужно обработать.
     * @return Поток сообщений, которые будут отправлены в хранилище (Store).
     */
    override fun connect(effects: Flow<PostEffect>): Flow<PostMessage> =
        listOf(
            handleNextPage(effects = effects),
            handleInitialPage(effects = effects),
            handleLikePost(effects = effects),
            handleDeletePost(effects = effects),
        )
            .merge()

    /**
     * Обрабатывает эффект загрузки следующей страницы постов.
     *
     * @param effects Поток эффектов, из которого фильтруются эффекты типа [PostEffect.LoadNextPage].
     * @return Поток сообщений, содержащий результат загрузки следующей страницы.
     */
    @OptIn(ExperimentalCoroutinesApi::class)
    private fun handleNextPage(effects: Flow<PostEffect>) =
        effects.filterIsInstance<PostEffect.LoadNextPage>()
            .mapLatest { postEffect: PostEffect.LoadNextPage ->
                PostMessage.NextPageLoaded(
                    try {
                        repository.getBeforePosts(
                            id = postEffect.id,
                            count = postEffect.count
                        )
                            .map(mapper::map)
                            .right()
                    } catch (e: Exception) {
                        if (e is CancellationException) throw e
                        e.left()
                    }
                )
            }

    /**
     * Обрабатывает эффект загрузки начальной страницы постов.
     *
     * @param effects Поток эффектов, из которого фильтруются эффекты типа [PostEffect.LoadInitialPage].
     * @return Поток сообщений, содержащий результат загрузки начальной страницы.
     */
    @OptIn(ExperimentalCoroutinesApi::class)
    private fun handleInitialPage(effects: Flow<PostEffect>) =
        effects.filterIsInstance<PostEffect.LoadInitialPage>()
            .mapLatest { postEffect: PostEffect.LoadInitialPage ->
                PostMessage.InitialLoaded(
                    try {
                        repository.getLatestPosts(
                            count = postEffect.count
                        )
                            .map(mapper::map)
                            .right()
                    } catch (e: Exception) {
                        if (e is CancellationException) throw e
                        e.left()
                    }
                )
            }

    /**
     * Обрабатывает эффект лайка поста.
     *
     * @param effects Поток эффектов, из которого фильтруются эффекты типа [PostEffect.Like].
     * @return Поток сообщений, содержащий результат лайка поста.
     */
    @OptIn(ExperimentalCoroutinesApi::class)
    private fun handleLikePost(effects: Flow<PostEffect>) =
        effects.filterIsInstance<PostEffect.Like>()
            .mapLatest { postEffect: PostEffect.Like ->
                PostMessage.LikeResult(
                    try {
                        Either.Right(
                            mapper.map(
                                repository.likeById(
                                    postId = postEffect.post.id,
                                    likedByMe = postEffect.post.likedByMe
                                )
                            )
                        )
                    } catch (e: Exception) {
                        if (e is CancellationException) throw e
                        Either.Left(PostWithError(post = postEffect.post, throwable = e))
                    }
                )
            }

    /**
     * Обрабатывает эффект удаления поста.
     *
     * @param effects Поток эффектов, из которого фильтруются эффекты типа [PostEffect.Delete].
     * @return Поток сообщений, содержащий ошибку удаления, если она произошла.
     */
    @OptIn(ExperimentalCoroutinesApi::class)
    private fun handleDeletePost(effects: Flow<PostEffect>) =
        effects.filterIsInstance<PostEffect.Delete>()
            .mapLatest { postEffect: PostEffect.Delete ->
                try {
                    repository.deleteById(postId = postEffect.post.id)
                } catch (e: Exception) {
                    if (e is CancellationException) throw e
                    PostMessage.DeleteError(
                        PostWithError(
                            post = postEffect.post, throwable = e
                        )
                    )
                }
            }
            .filterIsInstance<PostMessage.DeleteError>()
}
```

---

## Store<State, Effect, Message>

**Store<State, Effect, Message>** — это обобщенный класс, который управляет состоянием, эффектами и сообщениями. Он:

- Хранит текущее состояние.
- Обрабатывает сообщения и обновляет состояние.
- Управляет побочными эффектами.

```kotlin
/**
 * Псевдоним типа для хранилища (Store), специализированного для работы с постами.
 * Этот тип упрощает использование хранилища, связанного с состоянием постов, сообщениями и эффектами.
 *
 * @see Store Базовый класс хранилища, который управляет состоянием и эффектами.
 * @see PostState Состояние, связанное с постами.
 * @see PostMessage Сообщения, которые могут изменять состояние постов.
 * @see PostEffect Эффекты, которые выполняются в ответ на сообщения.
 */
typealias PostStore = Store<PostState, PostMessage, PostEffect>
```

---

## Either

**Either** — это тип, который может содержать либо успешный результат, либо ошибку.
Он часто используется для обработки асинхронных операций.

Чтобы не создавать x2 классов был создан интерфейс `Either` (либо).
`Left` обычно содержит некорректный (альтернативный) результат, а `Right` ожидаемый.

```kotlin
sealed class Either<out L, out R> {
    data class Left<out L>(val value: L) : Either<L, Nothing>()
    data class Right<out R>(val value: R) : Either<Nothing, R>()
}
```

```kotlin
fun fetchData(): Either<String, List<String>> {
    return try {
        Either.Right(repository.fetchData())
    } catch (e: Exception) {
        Either.Left(e.message ?: "Unknown error")
    }
}
```

---

## Intent

**Intent** — это действия пользователя, которые преобразуются в `Message`.

---

## Connect и Accept

**Connect** и **Accept** — это функции, которые связывают `View` с `Store`. 
Они позволяют `View` подписываться на изменения состояния и отправлять сообщения.

- **Accept** - Функция для передачи Message извне
- **Connect** - Функция нужна для запуска всех подписок

### Accept

```kotlin
/**
 * Принимает сообщение и добавляет его в поток сообщений для обработки.
 *
 * @param message Сообщение, которое нужно обработать.
 */
fun accept(message: Message) {
    messages.tryEmit(message)
}
```

```kotlin
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
```

```kotlin
override fun onLikeClicked(post: PostUiModel) {
    viewModel.accept(message = PostMessage.Like(post = post))
}
```

### Connect

```kotlin
/**
 * Интерфейс для обработки эффектов. Эффекты представляют собой действия, которые должны быть выполнены
 * в ответ на сообщения, такие как загрузка данных, лайки и т.д.
 *
 * @param Effect Тип эффекта, который будет обрабатываться.
 * @param Message Тип сообщения, которое будет возвращено после обработки эффекта.
 */
interface EffectHandler<Effect, Message> {

    /**
     * Подключает поток эффектов к потоку сообщений.
     *
     * @param effects Поток эффектов, которые нужно обработать.
     * @return Поток сообщений, которые будут отправлены в хранилище (Store).
     */
    fun connect(effects: Flow<Effect>): Flow<Message>
}
```

```kotlin
/**
 * Запускает хранилище и начинает обработку сообщений и эффектов.
 */
suspend fun connect() = coroutineScope {
    launch {
        effectHandler.connect(effects)
            .collect(messages::tryEmit)
    }
    launch {
        listOf(
            initMessages.asFlow(),
            messages,
        )
            .merge()
            .map { message: Message ->
                reducer.reduce(_state.value, message)
            }
            .collect { reducerResult: ReducerResult<State, Effect> ->
                _state.value = reducerResult.newState
                reducerResult.effects.forEach(effects::tryEmit)
            }
    }
}
```

```kotlin
/**
 * Подключает поток эффектов к потоку сообщений. Этот метод объединяет обработчики для всех типов эффектов.
 *
 * @param effects Поток эффектов, которые нужно обработать.
 * @return Поток сообщений, которые будут отправлены в хранилище (Store).
 */
override fun connect(effects: Flow<PostEffect>): Flow<PostMessage> =
    listOf(
        handleNextPage(effects = effects),
        handleInitialPage(effects = effects),
        handleLikePost(effects = effects),
        handleDeletePost(effects = effects),
    )
        .merge()
```

```kotlin
/**
 * Инициализатор ViewModel.
 * При создании ViewModel запускается подключение к хранилищу (Store) для обработки сообщений и эффектов.
 */
init {
    viewModelScope.launch {
        postStore.connect()
    }
}
```

---

 - **MVI** - Архитектурный паттерн с однонаправленным потоком данных.
 - **Store** - Центральный компонент, который управляет состоянием и эффектами.
 - **Message** - Действия, которые изменяют состояние.
 - **Effect** - Побочные эффекты, которые не изменяют состояние напрямую.
 - **Reducer** - Функция, которая обновляет состояние на основе сообщений.
 - **EffectHandler** - Обрабатывает побочные эффекты.
 - **Either** - Тип для обработки успешных результатов и ошибок.
 - **Intent** - Действия пользователя, которые преобразуются в сообщения.
 - **Connect/Accept** - Связывает `View` с `Store`.

---

#### [README](README.md) [UP](#up)
