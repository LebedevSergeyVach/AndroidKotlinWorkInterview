# Paging - пагинация для RecyclerView и ListAdapter, работа с разными View
<a name="up"></a>

---

**RecyclerView** — это компонент для отображения списков и сеток.
Он поддерживает эффективное отображение больших наборов данных.

**ListAdapter** — это адаптер для `RecyclerView`, который автоматически обновляет список при изменении данных. 
Он использует `DiffUtil` для определения изменений.

**DiffUtil** — это утилита, которая сравнивает два списка и определяет, какие элементы изменились. 
Это позволяет обновлять только измененные элементы, что повышает производительность.

В `RecyclerView` можно отображать элементы разных типов, например:

- Заголовки (`Header`).
- Данные (`Data`).
- Разделители (`Separator`).
- Прогресс-бар загрузки (`Loading`).
- Каждый тип элемента может иметь свой макет и логику отображения.

### Основные шаги реализации

 - **Создание модели данных** - Определяем типы элементов (например, заголовки, данные, разделители).
 - **Реализация ListAdapter** - Настраиваем адаптер для работы с разными типами `View`.
 - **Реализация DiffUtil** - Сравниваем элементы списка для эффективного обновления `RecyclerView`.
 - **Реализация пагинации** - Загружаем данные по частям и добавляем их в адаптер.
 - **Обработка прокрутки** - Определяем, когда пользователь доскроллил до конца списка, и загружаем следующую страницу данных.

---

## PagingModel и Mapper

Для передачи в адаптер нужно использовать неоднородный список из элементов.

```kotlin
/**
 * Маппер для преобразования состояния постов в модель пагинации.
 * Используется для отображения постов, ошибок и состояния загрузки в RecyclerView.
 */
object PostPagingMapper {

    /**
     * Преобразует состояние постов в список моделей пагинации.
     *
     * @param state Состояние постов.
     * @param context Контекст для доступа к строковым ресурсам.
     * @return Список моделей пагинации.
     */
    fun map(state: PostState, context: Context): List<PagingModel<PostUiModel>> {
        val posts: List<PagingModel.Data<PostUiModel>> = state.posts.map { post: PostUiModel ->
            PagingModel.Data(post)
        }

        val groupedPosts = mutableListOf<PagingModel<PostUiModel>>()
        var lastDate: String? = null

        for (post in posts) {
            val postDate = getFormattedDate(date = post.value.published, context = context)

            if (postDate != lastDate) {
                groupedPosts.add(PagingModel.DateSeparator(postDate))
                lastDate = postDate
            }

            groupedPosts.add(post)
        }

        return when (val statusValue = state.statusPost) {
            PostStatus.EmptyLoading -> List(PostReducer.PAGE_SIZE) { PagingModel.Loading }
            PostStatus.NextPageLoading -> groupedPosts + List(PostReducer.PAGE_SIZE) { PagingModel.Loading }
            is PostStatus.NextPageError -> groupedPosts + PagingModel.Error(reason = statusValue.reason)
            PostStatus.Refreshing, is PostStatus.Idle, is PostStatus.EmptyError -> groupedPosts
        }
    }

    /**
     * Преобразует состояние стены постов в список моделей пагинации.
     *
     * @param state Состояние стены постов.
     * @param context Контекст для доступа к строковым ресурсам.
     * @return Список моделей пагинации.
     */
    fun map(state: PostWallState): List<PagingModel<PostUiModel>> {
        val posts: List<PagingModel.Data<PostUiModel>> = state.posts.map { post: PostUiModel ->
            PagingModel.Data(post)
        }

        return when (val statusValue = state.statusPost) {
            PostStatus.EmptyLoading -> List(PostWallReducer.PAGE_SIZE) { PagingModel.Loading }
            PostStatus.NextPageLoading -> posts + List(PostWallReducer.PAGE_SIZE) { PagingModel.Loading }
            is PostStatus.NextPageError -> posts + PagingModel.Error(reason = statusValue.reason)
            PostStatus.Refreshing,
            is PostStatus.Idle,
            is PostStatus.EmptyError -> posts
        }
    }

    /**
     * Форматирует дату в строку с использованием строковых ресурсов.
     * Месяц отображается текстом, а не числом.
     *
     * @param date Дата в формате строки.
     * @param context Контекст для доступа к строковым ресурсам.
     * @return Отформатированная строка с датой.
     */
    @Suppress("DEPRECATION")
    private fun getFormattedDate(date: String, context: Context): String {
        val today = LocalDate.now()
        val yesterday = today.minusDays(1)
        val postDate = LocalDate.parse(date, DateTimeFormatter.ofPattern("dd.MM.yy HH:mm"))

        return when (postDate) {
            today -> context.getString(R.string.today)
            yesterday -> context.getString(R.string.yesterday)

            else -> {
                val formatter = DateTimeFormatter.ofPattern(
                    "dd MMM yyyy",
                    context.resources.configuration.locale
                )
                postDate.format(formatter)
            }
        }
    }
}
```

```kotlin
/**
 * Псевдоним типа для модели пагинации постов.
 * Используется для упрощения работы с пагинацией постов.
 */
typealias PostPagingModel = PagingModel<PostUiModel>

```

---

## DiffCallback

Необходимо реализовать общий `DiffUtil.ItemCallback` для сравнения элементов всего списка `RecyclerView`.

Каждая функция `PostPagingDiffCallback` сравнивает объекты на соответствие.
Если оба объекта посты, то делегируем работу на `postDiffCallback`.

```kotlin
/**
 * Callback для сравнения элементов списка постов с поддержкой пагинации.
 * Используется для определения изменений в списке постов и оптимизации обновлений RecyclerView.
 *
 * @see DiffUtil.ItemCallback Базовый класс для сравнения элементов списка.
 */
class PostPagingItemCallback : DiffUtil.ItemCallback<PostPagingModel>() {

    /**
     * Делегат для сравнения элементов списка постов.
     * Используется для обработки сравнения данных внутри элементов списка.
     *
     * @see PostItemCallback Callback для сравнения элементов списка постов.
     */
    private val delegate = PostItemCallback()

    /**
     * Проверяет, являются ли элементы одним и тем же объектом.
     *
     * @param oldItem Старый элемент.
     * @param newItem Новый элемент.
     *
     * @return Boolean true, если элементы одинаковы, иначе false.
     */
    override fun areItemsTheSame(oldItem: PostPagingModel, newItem: PostPagingModel): Boolean {
        if (oldItem::class != newItem::class) {
            return false
        }

        return if (oldItem is PagingModel.Data && newItem is PagingModel.Data) {
            delegate.areItemsTheSame(oldItem.value, newItem.value)
        } else {
            oldItem == newItem
        }
    }

    /**
     * Проверяет, содержат ли элементы одинаковые данные.
     *
     * @param oldItem Старый элемент.
     * @param newItem Новый элемент.
     *
     * @return Boolean true, если данные элементов одинаковы, иначе false.
     */
    override fun areContentsTheSame(oldItem: PostPagingModel, newItem: PostPagingModel): Boolean =
        oldItem == newItem

    /**
     * Возвращает объект, содержащий изменения в элементе.
     *
     * @param oldItem Старый элемент.
     * @param newItem Новый элемент.
     *
     * @return Any? Объект, содержащий изменения, или null, если изменений нет.
     */
    override fun getChangePayload(oldItem: PostPagingModel, newItem: PostPagingModel): Any? {
        if (oldItem::class != newItem::class) {
            return null
        }

        return if (oldItem is PagingModel.Data && newItem is PagingModel.Data) {
            delegate.getChangePayload(oldItem.value, newItem.value)
        } else {
            null
        }
    }
}
```

```kotlin
/**
 * Callback для сравнения элементов списка постов.
 * Используется для определения изменений в списке постов и оптимизации обновлений RecyclerView.
 *
 * @see DiffUtil.ItemCallback Базовый класс для сравнения элементов списка.
 */
class PostItemCallback : DiffUtil.ItemCallback<PostUiModel>() {

    /**
     * Проверяет, являются ли элементы одним и тем же объектом.
     *
     * @param oldItem Старый элемент.
     * @param newItem Новый элемент.
     *
     * @return Boolean true, если элементы одинаковы, иначе false.
     */
    override fun areItemsTheSame(oldItem: PostUiModel, newItem: PostUiModel): Boolean =
        oldItem.id == newItem.id

    /**
     * Проверяет, содержат ли элементы одинаковые данные.
     *
     * @param oldItem Старый элемент.
     * @param newItem Новый элемент.
     *
     * @return Boolean true, если данные элементов одинаковы, иначе false.
     */
    override fun areContentsTheSame(oldItem: PostUiModel, newItem: PostUiModel): Boolean =
        oldItem == newItem

    /**
     * Возвращает объект, содержащий изменения в элементе.
     *
     * @param oldItem Старый элемент.
     * @param newItem Новый элемент.
     *
     * @return Any? Объект, содержащий изменения, или null, если изменений нет.
     */
    override fun getChangePayload(oldItem: PostUiModel, newItem: PostUiModel): Any? =
        PostPayload(
            likeByMe = newItem.likedByMe.takeIf { likeByMe: Boolean ->
                likeByMe != oldItem.likedByMe
            },
            likes = newItem.likes.takeIf { likes: Int ->
                likes != oldItem.likes
            }
        )
            .takeIf { postPayload: PostPayload ->
                postPayload.isNotEmpty()
            }
}
```

---

### ListAdapter

Нужно научить `ListAdapter` работать с разными `View`.

### Разные типы View

 - **getItemViewType** - Определяет тип элемента на основе его позиции.
 - **onCreateViewHolder** - Создает `ViewHolder` для каждого типа элемента.
 - **onBindViewHolder** - Привязывает данные к `ViewHolder`.

```kotlin
/**
 * Адаптер для отображения списка постов в RecyclerView.
 *
 * Этот класс отвечает за управление списком постов и их отображение в RecyclerView.
 * Он также обрабатывает события, такие как клики на кнопки "лайк", "поделиться" и "удалить".
 *
 * @param listener Слушатель событий, который будет вызываться при кликах на элементы списка.
 * @param context Контекст приложения.
 * @param currentUserId ID текущего пользователя для определения прав на редактирование постов.
 *
 * @see PostViewHolder ViewHolder, используемый для отображения элементов списка.
 * @see PostItemCallback Callback для сравнения элементов списка.
 */
class PostAdapter(
    private val listener: PostListener,
    private val context: Context,
    private val currentUserId: Long
) : ListAdapter<PostPagingModel, RecyclerView.ViewHolder>(PostPagingItemCallback()) {

    private companion object {
        private const val TYPE_POST = 0
        private const val TYPE_ERROR = 1
        private const val TYPE_LOADING = 2
        private const val TYPE_DATE_SEPARATOR = 3
    }

    /**
     * Возвращает тип View для элемента списка на основе его позиции.
     *
     * @param position Позиция элемента в списке.
     * @return Int Идентификатор макета для элемента списка.
     */
    override fun getItemViewType(position: Int): Int =
        when (getItem(position)) {
            is PagingModel.Data -> TYPE_POST
            is PagingModel.Error -> TYPE_ERROR
            is PagingModel.Loading -> TYPE_LOADING
            is PagingModel.DateSeparator -> TYPE_DATE_SEPARATOR
        }

    /**
     * Создает новый ViewHolder для отображения элемента списка.
     *
     * @param parent Родительский ViewGroup.
     * @param viewType Тип View.
     *
     * @return RecyclerView.ViewHolder Новый ViewHolder.
     */
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder =
        when (viewType) {
            TYPE_POST -> createPostViewHolder(parent = parent)
            TYPE_ERROR -> createItemErrorViewHolder(parent = parent)
            TYPE_LOADING -> createItemSkeletonViewHolder(parent = parent)
            TYPE_DATE_SEPARATOR -> createItemDateSeparatorViewHolder(parent = parent)
            else -> error("PostAdapter.onCreateViewHolder: Unknown viewType $viewType")
        }

    /**
     * Привязывает данные к существующему ViewHolder.
     *
     * @param holder ViewHolder, к которому привязываются данные.
     * @param position Позиция элемента в списке.
     */
    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        when (val item = getItem(position)) {
            is PagingModel.Data -> (holder as PostViewHolder).bindPost(
                post = item.value,
                currentUserId = currentUserId
            )

            is PagingModel.Error -> (holder as ErrorViewHolder).bind(
                error = item.reason
            )

            is PagingModel.Loading -> (holder as SkeletonPostViewHolder).bind()

            is PagingModel.DateSeparator -> (holder as DateSeparatorViewHolder).bind(item.date)
        }
    }

    /**
     * Привязывает данные к существующему ViewHolder с учетом изменений.
     *
     * @param holder ViewHolder, к которому привязываются данные.
     * @param position Позиция элемента в списке.
     * @param payloads Список изменений.
     */
    override fun onBindViewHolder(
        holder: RecyclerView.ViewHolder,
        position: Int,
        payloads: List<Any>
    ) {
        if (payloads.isNotEmpty()) {
            payloads.forEach { post ->
                if (post is PostPayload) {
                    (holder as? PostViewHolder)?.bind(payload = post)
                }
            }
        } else {
            onBindViewHolder(holder, position)
        }
    }
}
```

---

#### [README](README.md) [UP](#up)
