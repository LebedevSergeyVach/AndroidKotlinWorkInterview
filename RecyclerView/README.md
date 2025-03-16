# RecyclerView - ViewGroup
<a name="up"></a>

---

**RecyclerView** — это гибкий и мощный компонент для отображения списков и сеток. Он:

- Эффективно управляет отображением большого количества элементов.
- Поддерживает анимации, горизонтальные и вертикальные списки, сетки и многое другое.
- Использует паттерн `ViewHolder` для повторного использования элементов списка.

---

## Основные компоненты RecyclerView

### Adapter

Адаптер отвечает за:

- Создание `ViewHolder`.
- Связывание данных с элементами списка.
- Возвращает количество элементов в списке.

### ViewHolder

**ViewHolder** — это объект, который хранит ссылки на элементы интерфейса (например, TextView, ImageView). 
Он используется для повторного использования элементов списка.

```kotlin
/**
 * ViewHolder для отображения элемента списка событий.
 *
 * @param binding Binding для макета элемента списка.
 * @param context Контекст приложения.
 *
 * @see EventAdapter Адаптер, использующий этот ViewHolder.
 */
@SuppressLint("ClickableViewAccessibility")
class EventViewHolder(
    private val binding: CardEventBinding,
    private val context: Context
) : ViewHolder(binding.root) {
    private var lastClickTime: Long = 0

    init {
        binding.cardEvent.setOnTouchListener { _, event: MotionEvent ->
            if (event.action == MotionEvent.ACTION_DOWN) {
                val clickTime = System.currentTimeMillis()
                if (clickTime - lastClickTime < 300) {
                    onDoubleClick()
                }
                lastClickTime = clickTime
            }
            false
        }
    }

    private fun onDoubleClick() {
        binding.like.performClick()
    }

    /**
     * Привязывает данные события к элементу списка.
     *
     * @param event Событие, данные которого нужно отобразить.
     */
    @SuppressLint("SetTextI18n")
    fun bindEvent(event: EventUiModel, currentUserId: Long) {
        binding.author.text = event.author
        binding.published.text = event.published
        binding.optionConducting.text = event.optionConducting
        binding.dataEvent.text = event.dateEvent
        binding.content.text = event.content

        if (event.link.isNotEmpty()) {
            binding.link.isVisible = true
            binding.link.text = event.link
        } else {
            binding.link.isVisible = false
        }

        binding.like.text = event.likes.toString()
        binding.participate.text = event.participates.toString()

        val radius = context.resources.getDimensionPixelSize(R.dimen.radius_for_rounding_images)

        renderingUserAvatar(event = event)

        binding.skeletonAttachment.showSkeleton()

        if (event.attachment != null) {
            renderingImageAttachment(event.attachment, radius)
        } else {
            binding.skeletonAttachment.showOriginal()
            binding.attachment.isVisible = false
        }

        SpannableString(binding.link.text)
        SpannableString(binding.content.text)

        updateLike(event.likedByMe)
        updateParticipate(event.participatedByMe)

        binding.menu.isVisible = event.authorId == currentUserId

        binding.share.setOnClickListener {
            shareEvent(event)
        }

        binding.cardEvent.setOnLongClickListener {
            shareEvent(event)

            true
        }
    }

    /**
     * Привязывает данные события к элементу списка с учетом изменений.
     *
     * @param payload Изменения в событии.
     */
    @SuppressLint("SetTextI18n")
    fun bind(payload: EventPayload) {
        payload.likeByMe?.let { likeByMe: Boolean ->
            updateLike(likeByMe)

            buttonClickAnimation(
                button = binding.like,
                condition = likeByMe,
                confetti = false,
                causeVibration = true
            )
        }

        payload.likes?.let { likes: Int ->
            binding.like.text = likes.toString()
        }

        payload.participateByMe?.let { participateByMe: Boolean ->
            updateParticipate(participateByMe)

            buttonClickAnimation(
                button = binding.participate,
                condition = participateByMe,
                confetti = false,
                causeVibration = true
            )
        }

        payload.participates?.let { participates: Int ->
            binding.participate.text = participates.toString()
        }
    }

    /**
     * Обновляет состояние лайка события.
     *
     * @param likeByMe Состояние лайка (лайкнут/не лайкнут).
     */
    private fun updateLike(likeByMe: Boolean) {
        binding.like.isSelected = likeByMe
    }
}
```

### ListAdapter

**ListAdapter** — это улучшенная версия `RecyclerView.Adapter`, которая автоматически обрабатывает обновления списка с помощью `DiffUtil`. 
Он упрощает работу с изменяющимися данными.

```kotlin
/**
 * Адаптер для отображения списка событий в RecyclerView.
 *
 * Этот класс отвечает за управление списком событий и их отображение в RecyclerView.
 * Он также обрабатывает события, такие как клики на кнопки "лайк", "поделиться" и "удалить".
 *
 * @param listener Слушатель событий, который будет вызываться при кликах на элементы списка.
 *
 * @see EventViewHolder ViewHolder, используемый для отображения элементов списка.
 * @see EventItemCallback Callback для сравнения элементов списка.
 */
class EventAdapter(
    private val listener: EventListener,
    private val context: Context,
    private val currentUserId: Long
) : ListAdapter<EventUiModel, EventViewHolder>(EventItemCallback()) {}
```

### DiffUtil.ItemCallback

**DiffUtil.ItemCallback** используется для сравнения элементов списка. Он определяет:

- **`areItemsTheSame`** - Проверяет, ссылаются ли два объекта на один и тот же элемент.
- **`areContentsTheSame`** - Проверяет, одинаковы ли данные в двух элементах.

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
}8
```

### Payload

**Payload** — это механизм для частичного обновления элементов списка. 
Он позволяет обновлять только те части элемента, которые изменились, вместо полного пересоздания.

```kotlin
/**
 * Класс, представляющий изменения в посте.
 * Используется для передачи изменений в элемент списка, чтобы избежать полного обновления ViewHolder.
 *
 * @property likeByMe Состояние лайка (лайкнут/не лайкнут).
 * @property likes Количество лайков у поста.
 *
 * @see PostItemCallback Используется для передачи изменений в элемент списка.
 */
data class PostPayload (
    val likeByMe: Boolean? = null,
    val likes: Int? = null,
) {

    /**
     * Проверяет, есть ли изменения в объекте.
     *
     * @return Boolean true, если есть изменения, иначе false.
     */
    fun isNotEmpty(): Boolean = likeByMe != null || likes != null
}

```

---

## Как работает RecyclerView?

1. **Создание ViewHolder**: `RecyclerView` вызывает `onCreateViewHolder` для создания нового `ViewHolder`.
2. **Связывание данных**: `RecyclerView` вызывает `onBindViewHolder` для связывания данных с `ViewHolder`.
3. **Повторное использование**: Когда элемент прокручивается за пределы экрана, его `ViewHolder` переиспользуется для нового элемента.
4. **Обновление списка**: Если данные изменяются, `RecyclerView` обновляет только те элементы, которые изменились.

---

## Scroll Listener

Для отслеживания события скролла до низа страницы мы можем следить за элементами `RecyclerView`.

```kotlin
binding.list.addOnChildAttachStateChangeListener(
    object : RecyclerView.OnChildAttachStateChangeListener {
        override fun onChildViewAttachedToWindow(view: View) {
            val itemsCount = adapter.itemCount
            val adapterPosition = binding.list.getChildAdapterPosition(view)

            if (itemsCount - 5 <= adapterPosition) {
                viewModel.accept(message = PostMessage.LoadNextPage)
            }
        }

        override fun onChildViewDetachedFromWindow(view: View) = Unit
    }
)
```

---

#### [README](README.md) [UP](#up)
