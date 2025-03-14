# Coroutines / Корутины
<a name="up"></a>

---

**Coroutines** — это легковесные потоки, которые позволяют выполнять асинхронные операции без блокировки основного потока. 
Они предоставляют механизм для приостановки и возобновления выполнения кода.

### Легковесные потоки

Корутины в `Kotlin` иногда называют легковесными потоками, потому что они позволяют выполнять задачи параллельно или асинхронно, не создавая при этом тяжелых системных потоков (`threads`).

### Преимущества Coroutines

- **Упрощение кода**: Нет `callback hell`, код выглядит как синхронный.
- **Интеграция с жизненным циклом**: Использование `lifecycleScope` для автоматической отмены корутин.
- **Гибкость**: Поддержка параллельных запросов, таймаутов и других сложных сценариев.


---

### Suspend

**Suspend** — это ключевое слово, которое указывает, что функция может быть приостановлена и возобновлена. 
`Suspend`-функции могут вызываться только из других `suspend`-функций или корутин.
Функция может приостановить своё выполнение в определённой точке и вернуть результат или выбросить `Exception`.


---

## Continuation

**Continuation** — это объект, который представляет собой точку возобновления выполнения корутины. 
Когда корутина приостанавливается, она сохраняет свое состояние в `Continuation`, чтобы позже возобновить выполнение.

### Основные методы Continuation

**resumeWith(result: Result<T>)** - Возобновляет выполнение корутины с результатом или исключением.

```kotlin
suspend fun fetchData(): String {
    return suspendCoroutine { continuation ->
        // Имитация асинхронного запроса
        Thread {
            Thread.sleep(1000)
            continuation.resumeWith(Result.success("Data fetched"))
        }.start()
    }
}
```

---

## withContext

**withContext** — это функция, которая позволяет временно изменить контекст выполнения корутины.

Мы можем использовать функцию `withContext`, чтобы изменить политику запуска задач на определённых потоках.

```kotlin
suspend fun fetchData(): String = withContext(Dispatchers.IO) {
    delay(1000)
    "Data fetched"
}
```

---

## CoroutineContext

**CoroutineContext** — это контекст выполнения корутины, который содержит:

- **Dispatcher** - Определяет, на каком потоке выполняется корутина.
- **Job** - Управляет жизненным циклом корутины.
- **CoroutineExceptionHandler** - Обрабатывает исключения в корутине.



### Пример использования CoroutineContext

- **Создание контекста:**

```kotlin
val context = Dispatchers.IO + CoroutineName("MyCoroutine") + CoroutineExceptionHandler { _, exception ->
    println("Caught exception: $exception")
}
```

- **Запуск корутины с контекстом:**

```kotlin
val job = CoroutineScope(context).launch {
    println("Running in ${coroutineContext[CoroutineName]}")
    throw IllegalStateException("Test exception")
}
```

---

## resumeWith

**resumeWith** — это метод `Continuation`, который возобновляет выполнение корутины с результатом или исключением.

```kotlin
suspend fun fetchData(): String {
    return suspendCoroutine { continuation ->
        // Имитация асинхронного запроса
        Thread {
            try {
                val data = "Data fetched"
                continuation.resumeWith(Result.success(data))
            } catch (e: Exception) {
                continuation.resumeWith(Result.failure(e))
            }
        }.start()
    }
}
```

---

## Dispatchers

**Dispatchers** определяют, на каком потоке выполняется корутина. Основные диспетчеры:

- **Dispatchers.Main** - Основной поток (`UI`-поток в `Android`).
- **Dispatchers.IO** - Для `I/O`-операций (например, сетевые запросы, работа с файлами).
- **Dispatchers.Default** - Для вычислительных задач.
- **Dispatchers.Unconfined** - Не привязывает корутину к конкретному потоку.

---

## Sync

**Sync (синхронный)** — это выполнение кода в одном потоке, где каждая операция блокирует выполнение следующей до завершения текущей.

---

## Async

**Async** — это функция, которая запускает корутину и возвращает `Deferred`, который можно использовать для получения результата.

```kotlin
fun main() = runBlocking {
    val deferred = async {
        delay(1000)
        "Task completed"
    }
    println("Task started")
    println(deferred.await())
}
```

---

## Launch

**Launch** — это функция, которая запускает корутину без возврата результата. Она используется для выполнения фоновых задач.

Запустить корутину можно при помощи билдеров:

 - **launch** – возвращает `Job`
 - **async** – возвращает `Deferred<T>`

Обе функции являются экстеншенами к CoroutineScope

---

## CoroutineScope

**CoroutineScope** — это область видимости для корутин. Она управляет жизненным циклом корутин и может быть отменена.

```kotlin
val scope = CoroutineScope(Dispatchers.Main)
scope.launch {
    delay(1000)
    println("Task completed")
}
scope.cancel() // Отмена всех корутин в scope
```

**CoroutineScope** - Интерфейс с одним свойством. Обязательно:

 - должен содержать в `coroutineContext` объект типа `Job`
 - должен быть отменён, когда больше не нужен

Исходя из того, что `CoroutineScope` нужно закрывать, логичнее всего привязать его к жизненному циклу компонентов.

---

## MainScope

**MainScope** — это предопределенный `CoroutineScope`, который использует `Dispatchers.Main` для выполнения корутин.

```kotlin
val mainScope = MainScope()
mainScope.launch {
    delay(1000)
    println("Task completed")
}
mainScope.cancel() // Отмена всех корутин в MainScope
```

Удобнее всего запускать корутины из главного потока. 
Для этого нам потребуется создать CoroutineScope с `Dispatchers.Main`.
Можем воспользоваться готовой функцией `MainScope`.

---

## Flow

**Flow** — это поток данных, который может `emit` (излучать) несколько значений асинхронно.
Он похож на `RxJava` `Observable`, но более легковесный.

```kotlin
fun fetchData(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(1000)
        emit(i)
    }
}

fun main() = runBlocking {
    fetchData().collect { value -> println(value) }
}
```

Интерфейс с одной функцией. Смысл которой – подписаться.

---

## FlowCollector

**FlowCollector** — это интерфейс, который используется для сбора элементов из `Flow`.

```kotlin
fun fetchData(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(1000)
        emit(i)
    }
}

fun main() = runBlocking {
    fetchData().collect { value -> println(value) }
}
```

В `FlowCollector` можно передавать данные.

---

## Hot/Cold Flow

- **Cold Flow** - Начинает `emit` элементы только после подписки. Каждый подписчик получает свою копию данных.
- **Hot Flow** - `Emit` элементы независимо от подписок. Подписчики получают только те элементы, которые были `emitted` после подписки.

По умолчанию `Flow`, созданный через `flow` билдер, является холодным.
Если нам нужна горячая версия, которая работает независимо от наличия подписчиков в одном экземпляре, то нам понадобится `SharedFlow` или `StateFlow`.

---

## SharedFlow

**SharedFlow** — это поток данных, который может иметь несколько подписчиков (`hot flow`). 
Он используется для передачи данных между корутинами.

```kotlin
val sharedFlow = MutableSharedFlow<Int>()

fun main() = runBlocking {
    launch {
        sharedFlow.collect { value -> println("Subscriber 1: $value") }
    }
    launch {
        sharedFlow.collect { value -> println("Subscriber 2: $value") }
    }
    sharedFlow.emit(1)
    sharedFlow.emit(2)
}
```

Наследуется от `Flow`.

---

## StateFlow

**StateFlow** — это поток данных, который хранит текущее состояние и уведомляет подписчиков об изменениях (`hot flow`).

```kotlin
val stateFlow = MutableStateFlow(0)

fun main() = runBlocking {
    launch {
        stateFlow.collect { value -> println("State: $value") }
    }
    stateFlow.value = 1
    stateFlow.value = 2
}
```

Является наследником `SharedFlow`. Появляется возможность получить текущее значение.

### Пример StateFlow в ViewModel

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
```

---

## MutableStateFlow

**MutableStateFlow** — это изменяемая версия `StateFlow`, которая позволяет обновлять текущее состояние.

```kotlin
val mutableStateFlow = MutableStateFlow(0)

fun main() = runBlocking {
    launch {
        mutableStateFlow.collect { value -> println("State: $value") }
    }
    mutableStateFlow.value = 1
    mutableStateFlow.value = 2
}
```

---

## Lifecycle Coroutines

**Lifecycle Coroutines** — это интеграция корутин с жизненным циклом `Android`-компонентов (например, `Activity`, `Fragment`).
Это позволяет автоматически отменять корутины при уничтожении компонента.

---

## Coroutines в ViewModel

**Coroutines в ViewModel** позволяют выполнять асинхронные операции, которые автоматически отменяются при очистке `ViewModel`.

### viewModel.state

**viewModel.state** — это `StateFlow`, который хранит текущее состояние (`state`) в `ViewModel`. 
Обычно это неизменяемый поток данных, который обновляется в `ViewModel` и может быть подписан в `Fragment` или `Activity`.

### .flowWithLifecycle(viewLifecycleOwner.lifecycle)

**flowWithLifecycle** — это функция из библиотеки `androidx.lifecycle:lifecycle-runtime-ktx`, которая позволяет подписываться на `Flow` только тогда, когда жизненный цикл `Fragment` или `Activity` находится в активном состоянии (например, `STARTED` или `RESUMED`). 
Это предотвращает утечки памяти и ненужные обновления `UI`, когда `Fragment` не `виден`.

- Если **Fragment** переходит в состояние `STARTED` или `RESUMED`, подписка на `Flow` активируется.
- Если **Fragment** переходит в состояние `STOPPED` или `DESTROYED`, подписка автоматически отменяется.

**viewLifecycleOwner.lifecycle** — это жизненный цикл `Fragment`, который управляет подпиской.

### .onEach { state -> ... }

**.onEach** - это оператор `Flow`, который вызывается каждый раз, когда `Flow` излучает новое значение.
Внутри этого блока можно обрабатывать изменения состояния (`state`).

**state** — это текущее состояние, которое пришло из `ViewModel`.
В зависимости от состояния (`Loading`, `Success`, `Error`) обновляется `UI`.

### .launchIn(viewLifecycleOwner.lifecycleScope)

**launchIn** — это функция, которая запускает `Flow` в указанной `CoroutineScope`. 
В данном случае используется `lifecycleScope` `Fragment`, который автоматически отменяет корутину, когда `Fragment` уничтожается.

Как это работает?

- `launchIn` запускает `Flow` в фоновом потоке (если не указан другой `Dispatcher`).
- Подписка на `Flow` будет активна до тех пор, пока Fragment находится в активном состоянии.
- Когда `Fragment` уничтожается, подписка автоматически отменяется, предотвращая утечки памяти.

Здесь `viewLifecycleOwner.lifecycleScope` — это `CoroutineScope`, привязанный к жизненному циклу `Fragment`.

```kotlin
viewModel.state
    .flowWithLifecycle(viewLifecycleOwner.lifecycle)
    .onEach { state ->
        // Работа с потоком состояния StateFlow в View Model
    }
    .launchIn(viewLifecycleOwner.lifecycleScope)
```

---

- **Coroutines**: Легковесные потоки для асинхронного программирования.
- **Suspend**: Функции, которые могут быть приостановлены и возобновлены.
- **Continuation**: Объект, представляющий точку возобновления корутины.
- **withContext**: Временное изменение контекста выполнения.
- **CoroutineContext**: Контекст выполнения корутины, включающий Dispatcher, Job и ExceptionHandler.
- **resumeWith**: Метод для возобновления корутины с результатом или исключением.
- **Dispatchers**: Управление потоками выполнения.
- **Launch/Async**: Запуск корутин.
- **CoroutineScope/MainScope**: Области видимости для корутин.
- **Flow**: Поток данных, который может emit несколько значений.
- **FlowCollector**: Сбор элементов из Flow.
- **Hot/Cold Flow**: Разные типы потоков данных.
- **SharedFlow/StateFlow**: Потоки данных с поддержкой нескольких подписчиков.
- **MutableStateFlow**: Изменяемая версия StateFlow.
- **Lifecycle Coroutines**: Интеграция корутин с жизненным циклом Android-компонентов.
- **Coroutines в ViewModel**: Выполнение асинхронных операций в ViewModel с автоматической отменой.
- **Retrofit + Coroutines**: Интеграция сетевых запросов с корутинами.

---

#### [README](README.md) [UP](#up)
