# Компоненты Android
<a name="up"></a>

---

**Компоненты Android** — это ключевые строительные блоки любого приложения. 
Каждый компонент имеет свою роль и жизненный цикл.

---

## Activity (Активность)

**Activity** — это один экран приложения с пользовательским интерфейсом (`UI`).

### Особенности:

 - Имеет жизненный цикл (`onCreate`, `onStart`, `onResume`, `onPause`, `onStop`, `onDestroy`).
 - Управляется через `Intent` (для перехода между `Activity`).
 - Может содержать `Fragments`.

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
```

---

## Fragment (Фрагмент)

**Fragment** — это часть `UI`, которая может быть повторно использована в разных `Activity`.

Особенности:

 - Имеет собственный жизненный цикл, но зависит от `Activity`.
 - Позволяет создавать гибкие интерфейсы для разных размеров экрана.

```kotlin
class MyFragment : Fragment() {
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_my, container, false)
    }
}
```

---

## Service (Сервис)

**Service** — компонент для выполнения фоновых задач без `UI`. 

Типы сервисов:

 - **Started Service** — запускается и работает, пока не завершит задачу.
 - **Bound Service** — связан с компонентами (например, `Activity`) и уничтожается, когда все отключаются.
 - **Foreground Service** — работает с уведомлением (например, музыкальный плеер).

```kotlin
class MyService : Service() {
    override fun onBind(intent: Intent?): IBinder? = null

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        // Фоновая задача
        return START_STICKY
    }
}
```

---

## BroadcastReceiver (Приёмник широковещательных сообщений)

**BroadcastReceiver** реагирует на системные или кастомные события.
Например: изменение уровня заряда батареи, получение `SMS`.

Особенности:

 - Может быть зарегистрирован в манифесте или динамически в коде.
 - Работает даже если приложение не запущено (если объявлен в манифесте).

```kotlin
class MyReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        if (intent.action == Intent.ACTION_BATTERY_LOW) {
            Toast.makeText(context, "Батарея разряжена!", Toast.LENGTH_SHORT).show()
        }
    }
}
```

---

## ContentProvider (Провайдер контента)

**ContentProvider** предоставляет доступ к данным между приложениями.
Например: контакты, календарь.

Особенности:

 - Использует `URI` для идентификации данных.
 - Работает через `Cursor` или `Room` (если используется с базами данных).

```kotlin
class MyProvider : ContentProvider() {
    override fun query(
        uri: Uri,
        projection: Array<out String>?,
        selection: String?,
        selectionArgs: Array<out String>?,
        sortOrder: String?
    ): Cursor? {
        // Возвращаем данные
        return null
    }
}
```

---

## ViewModel (Модель представления)

**ViewModel** хранит и управляет данными для `UI`, переживая изменения конфигурации.

Особенности:

 - Не зависит от `UI` (`Activity`/`Fragment`).
 - Может использоваться с `LiveData` или `Flow` для реактивного программирования.

```kotlin
class MyViewModel : ViewModel() {
    private val _data = MutableLiveData<String>()
    val data: LiveData<String> get() = _data

    fun loadData() {
        _data.value = "Hello, ViewModel!"
    }
}
```

---

## WorkManager (Фоновые задачи)

**WorkManager** выполняет отложенные или периодические фоновые задачи, учитывая состояние устройства (заряд батареи, интернет).

Особенности:

 - Гарантирует выполнение (даже после перезагрузки).
 - Поддерживает цепочки задач и ограничения (например, только при `Wi-Fi`).

```kotlin
class MyWorker(context: Context, params: WorkerParameters) : Worker(context, params) {
    override fun doWork(): Result {
        // Фоновая задача
        return Result.success()
    }
}

// Запуск
val workRequest = OneTimeWorkRequest.Builder(MyWorker::class.java).build()
WorkManager.getInstance(context).enqueue(workRequest)
```

---

## Сравнение компонентов

## **Сравнение компонентов**
| Компонент             | Назначение                       | Жизненный цикл          | Пример использования        |
|-----------------------|----------------------------------|-------------------------|-----------------------------|
| **Activity**          | Экран приложения                 | Зависит от пользователя | Главный экран, форма входа  |
| **Fragment**          | Часть UI                         | Зависит от Activity     | Список + детали             |
| **Service**           | Фоновая задача без UI            | Долгий                  | Воспроизведение музыки      |
| **BroadcastReceiver** | Реакция на системные события     | Кратковременный         | Уведомление о низком заряде |
| **ContentProvider**   | Обмен данными между приложениями | Долгий                  | Доступ к контактам          |
| **ViewModel**         | Хранение данных для UI           | Зависит от Activity     | Данные для списка           |
| **WorkManager**       | Гарантированная фоновая задача   | Зависит от задачи       | Синхронизация данных        |

---

## Как компоненты взаимодействуют?
1. **Activity** запускает **Fragment**.
2. **Fragment** использует **ViewModel** для данных.
3. **ViewModel** вызывает **WorkManager** для фоновых задач.
4. **BroadcastReceiver** реагирует на событие и обновляет **Activity**.
5. **ContentProvider** предоставляет данные другим приложениям.

---

#### [README](README.md) [UP](#up)
