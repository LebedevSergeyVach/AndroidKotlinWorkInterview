# Сборщик мусора (Garbage Collector, GC)
<a name="up"></a>

---

**Сборщик мусора** — это механизм автоматического управления памятью в `JVM`, который освобождает объекты, ставшие недостижимыми.

---

## Как GC определяет, какие объекты удалять?

### Принцип достижимости (Reachability)

Объект считается живым (`reachable`), если на него есть ссылка из:

 - Стека (локальные переменные, параметры методов).
 - Статических полей класса.
 - `JNI`-ссылок (для нативного кода).
 - Других живых объектов (например, поля класса).

Если цепочка ссылок разрывается — объект становится мусором.

### Алгоритм маркировки (Marking)

1. **Корневые точки (GC Roots):** 
    - `JVM` начинает обход с "корней" (стек, статические поля и т.д.).

2. **Обход графа объектов:**
   - Все объекты, достижимые из корней, помечаются как живые.

3. **Сборка мусора:**
   - Непомеченные объекты удаляются.

```java
// Пример: объект становится мусором
Object obj = new Object(); // Создали объект
obj = null;                // Объект больше не достижим → будет удалён GC
```

---

## Типы ссылок и их влияние на GC

В `Java`/`Kotlin` есть 4 типа ссылок, которые по-разному взаимодействуют с `GC`:

| Тип ссылки                | Описание                                                                   | Когда удаляется?             |
|---------------------------|----------------------------------------------------------------------------|------------------------------|
| **Сильная (`Strong`)**    | Обычные ссылки (`val obj = Object()`).                                     | Только при недостижимости.   |
| **Мягкая (`Soft`)**       | `SoftReference` — для кэшей.                                               | При нехватке памяти.         |
| **Слабая (`Weak`)**       | `WeakReference` — для ассоциативных данных (например, `WeakHashMap`).      | В следующем цикле `GC`.      |
| **Фантомная (`Phantom`)** | `PhantomReference` — для финализации (устаревший подход, лучше `Cleaner`). | После `finalize` (если был). |

```kotlin
// SoftReference (кэш)
val softRef = SoftReference(heavyObject)

// WeakReference (ассоциативные данные)
val weakRef = WeakReference(data)
```

---

## Как GC работает в многопоточных приложениях?

###  Проблемы многопоточности

 - **Stop-The-World (STW):** На время работы `GC` все потоки приостанавливаются. → Важно минимизировать паузы (особенно для `Android UI`).
 - **Конкуренция за ресурсы:** Несколько потоков могут создавать/удалять объекты одновременно.

### Алгоритмы GC (и их многопоточная оптимизация)

1. **Serial GC (Single-Thread)**
 - **Как работает:** - Один поток выполняет сборку.
 - **Когда использовать:** - Для приложений с маленькой кучей (например, утилиты).

2. **Parallel GC (Throughput Collector)**
 - **Как работает:** Несколько потоков параллельно выполняют сборку.
 - **Плюсы:** Быстрее `Serial GC` на многопроцессорных системах.
 - **Минусы:** Всё равно требует `STW`-пауз.

3. **CMS (Concurrent Mark-Sweep)**
 - **Как работает:** Большая часть работы (маркировка) выполняется без остановки приложения.
 - **Проблемы:** Требует больше `CPU`, возможна фрагментация памяти.

4. **G1 (Garbage-First)**
 - **Как работает:** Делит кучу на регионы, собирает мусор в фоне.
 - **Плюсы:** Предсказуемые паузы (подходит для `Android`).

5. **ZGC / Shenandoah (Low-Latency)**
 - **Как работает**: Практически без `STW`-пауз (паузы < 1 мс).
 - **Когда использовать**: Для высоконагруженных серверов (не поддерживается в `Android`).

---

## Финанализация и её проблемы

### Метод `finalize()`

Вызывается перед удалением объекта (если переопределён).

**Проблемы:**

 - Не гарантируется вызов (если `GC` не соберёт объект).
 - Замедляет `GC` (объект с `finalize()` проходит два цикла сборки).
 - Может "воскресить" объект (через сильную ссылку внутри `finalize()`).

**Совет**: Вместо `finalize()` используйте `Cleaner` или `PhantomReference`.

```java
// Плохо (устаревший способ)
@Override
protected void finalize() throws Throwable {
   releaseResources();
}

// Лучше (Java 9+)
Cleaner cleaner = Cleaner.create();
cleaner.register(this, () -> releaseResources());
```

---

## Как GC взаимодействует с Kotlin Coroutines?

### Лямбды и замыкания

Лямбды корутин могут захватывать контекст, что иногда приводит к утечкам:

```kotlin
scope.launch {
    val data = fetchData() // Захватывает 'scope'
    println(data)
}
```

Решение: Используйте `coroutineScope` или `viewModelScope` для автоматической отмены.

### DisposableHandle

Для ресурсов, которые нужно освобождать:

```kotlin
val job = scope.launch { ... }
job.invokeOnCompletion { releaseResources() }
```

---

# Сборщик мусора (GC) в Android/Kotlin: Особенности работы с корутинами

В `Android` используется виртуальная машина `ART` (`Android Runtime`) с улучшенным `GC` по сравнению с классической `JVM`. 

---

## Особенности GC в Android

### Тип сборщика мусора

 - **До Android 5.0 (Lollipop):** `CMS` (`Concurrent Mark-Sweep`) с длительными `STW`-паузами.
 - **Android 5.0+ (ART):** `Generational GC` (похож на `G1` в `JVM`):
 - **Молодая генерация (Young Generation):** Частая сборка (алгоритм `Copying`).
 - **Старая генерация (Old Generation):** Редкая сборка (`Mark-Sweep` или `Mark-Compact`).

### Проблемы GC в Android

 - **Stop-The-World (STW) паузы:** При сборке старой генерации `UI` может "зависать".
 - **Трассировка ссылок:** `GC` должен обходить все живые объекты, включая корутины.

---

## Корутины и GC: Как избежать утечек

Корутины могут неявно удерживать ссылки на объекты, что приводит к утечкам памяти.

### 1. Захват контекста в лямбдах

**Проблема:** Лямбда корутины захватывает внешний класс (например, `Activity`), продлевая его жизнь.

```kotlin
class MyActivity : AppCompatActivity() {
    override fun onCreate() {
        lifecycleScope.launch {
            val data = fetchData() // Лямбда захватывает MyActivity!
            updateUI(data)         // Если Activity уничтожена — утечка!
        }
    }
}
```

**Решение:** Используйте `viewModelScope` или `lifecycleScope` + проверку `isActive`:

```kotlin
lifecycleScope.launch {
    if (!isActive) return@launch // Проверка перед долгой операцией
    val data = fetchData()
    withContext(Dispatchers.Main) {
        if (isActive) updateUI(data) // Дополнительная проверка
    }
}
```

### 2. Job и отмена корутин

Неотменённые корутины удерживают ссылки на свои контексты.

**Плохо:**

```kotlin
val job = CoroutineScope(Dispatchers.IO).launch {
    heavyOperation() // Корутина живёт даже после закрытия Activity
}
```

**Хорошо:**

```kotlin
private val scope = CoroutineScope(Dispatchers.IO + SupervisorJob())

override fun onDestroy() {
    scope.cancel() // Явная отмена при уничтожении
    super.onDestroy()
}
```

### WeakReference для колбэков

Если корутина принимает колбэки из `Activity`, используйте `WeakReference`:

```kotlin
class MyViewModel : ViewModel() {
    private var callback: WeakReference<MyCallback>? = null

    fun registerCallback(cb: MyCallback) {
        callback = WeakReference(cb)
    }

    fun fetchData() {
        viewModelScope.launch {
            val data = repo.loadData()
            callback?.get()?.onDataLoaded(data) // Не удерживает Activity
        }
    }
}
```

### Лучший способ привязать корутины и ViewModel к жизненному циклу и области

```kotlin
viewModel.state
   .flowWithLifecycle(lifecycle = viewLifecycleOwner.lifecycle)
   .onEach { state: State ->
   }
    .launchIn(scope = viewLifecycleOwner.lifecycleScope)
```

---

## Как GC обрабатывает корутины?

### Структура корутины в памяти

 - **Continuation:** Объект, хранящий состояние корутины (локальные переменные, точка возобновления).
 - **Context:** Ссылки на `Dispatcher`, `Job`, `CoroutineScope`.

**Что делает GC:**

 - Если корутина завершена или отменена — её объекты помечаются как мусор.
 - Если корутина активна — все её объекты достижимы из корней (`Job`, `Dispatcher`).

###  Особенности для Dispatchers

 - **Dispatchers.Main (UI):** Корутины здесь имеют высокий приоритет, `GC` старается не трогать их во время работы.
 - **Dispatchers.IO:** Долгие операции могут создавать много временных объектов → частые сборки в молодой генерации.

---

#### [README](README.md) [UP](#up)
