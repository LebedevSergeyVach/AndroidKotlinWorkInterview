# Service — Сервисы в Android
<a name="up"></a>

---

**Service** — это компонент `Android`, который выполняет задачи в фоновом режиме без пользовательского интерфейса.
Сервисы используются для выполнения долгосрочных операций, таких как воспроизведение музыки, загрузка файлов или обработка данных.

---

## Типы Service

### **Started Service (Запущенный сервис)**

**Started Service** — это сервис, который запускается с помощью `startService()` и работает в фоновом режиме, даже если приложение закрыто. 
Он не взаимодействует напрямую с компонентами приложения.

 - Запускается с помощью `startService()`.
 - Работает в фоновом режиме, даже если приложение закрыто.
 - Останавливается самостоятельно после выполнения задачи или с помощью `stopSelf()`.
 - Не предоставляет интерфейс для взаимодействия с другими компонентами.
 - Начиная с `Android 8.0` (`API 26`), `Started Service` ограничен в фоновом режиме.

### Когда использовать?

 - Для выполнения одноразовых задач, которые не требуют взаимодействия с `UI`.
 - Например, загрузка файлов, синхронизация данных.

### **Bound Service (Связанный сервис)**

**Bound Service** — это сервис, который связывается с компонентами приложения (например, `Activity` или `Fragment`) через `bindService()`. 
Он предоставляет интерфейс для взаимодействия.

 - Запускается с помощью `bindService()`.
 - Обеспечивает взаимодействие между компонентами (например, `Activity` и `Service`).
 - Останавливается, когда все клиенты отключаются.
 - Работает только пока связан хотя бы один клиент.
 - Использует `Binder` или `Messenger` для взаимодействия.
 - Подходит для задач, которые должны быть синхронизированы с `UI`.

### Когда использовать?

- Для задач, которые требуют взаимодействия с `UI`.
- Например, воспроизведение музыки, передача данных между компонентами.

### **Foreground Service (Сервис переднего плана)**

**Foreground Service** — это сервис, который работает с уведомлением в статус-баре.
Он используется для задач, которые должны быть видны пользователю.

 - Работает как `Started Service`, но с уведомлением в статус-баре.
 - Используется для задач, которые должны быть видны пользователю (например, воспроизведение музыки).
 - Требует отображения уведомления (обязательно с `Android 8.0`).
 - Не ограничен фоновыми ограничениями.
 - Подходит для задач, которые должны быть выполнены даже если приложение закрыто.

### Когда использовать?

 - Для задач, которые требуют длительного выполнения и должны быть видны пользователю.
 - Например, воспроизведение музыки, загрузка файлов.

---

## Жизненный цикл Service

### Started Service:

- **onCreate()** — Вызывается при создании сервиса.
- **onStartCommand()** — Вызывается при запуске сервиса.
- **onDestroy()** — Вызывается при остановке сервиса.

### Bound Service:

- **onCreate()** — Вызывается при создании сервиса.
- **onBind()** — Вызывается при связывании сервиса с клиентом.
- **onUnbind()** — Вызывается при отключении клиента.
- **onDestroy()** — Вызывается при остановке сервиса.

---

## Примеры

### Пример Started Service

```kotlin
class MyService : Service() {

    override fun onBind(intent: Intent?): IBinder? {
        return null // Не используется в Started Service
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        // Выполнение задачи в фоновом режиме
        doBackgroundWork()
        return START_STICKY // Сервис будет перезапущен после завершения
    }

    private fun doBackgroundWork() {
        // Логика фоновой задачи
    }

    override fun onDestroy() {
        super.onDestroy()
        // Очистка ресурсов
    }
}
```

```kotlin
val intent = Intent(this, MyService::class.java)
startService(intent)
```

### Пример Bound Service

```kotlin
class MyBoundService : Service() {

    private val binder = LocalBinder()

    inner class LocalBinder : Binder() {
        fun getService(): MyBoundService = this@MyBoundService
    }

    override fun onBind(intent: Intent?): IBinder {
        return binder
    }

    fun performTask() {
        // Логика задачи
    }
}
```

```kotlin
val connection = object : ServiceConnection {
    override fun onServiceConnected(name: ComponentName?, service: IBinder?) {
        val binder = service as MyBoundService.LocalBinder
        val myService = binder.getService()
        myService.performTask()
    }

    override fun onServiceDisconnected(name: ComponentName?) {
        // Обработка отключения
    }
}

val intent = Intent(this, MyBoundService::class.java)
bindService(intent, connection, Context.BIND_AUTO_CREATE)
```

### Foreground Service

**Foreground Service** требует отображения уведомления. Это обязательно начиная с `Android 8.0` (`API 26`).

```kotlin
class MyForegroundService : Service() {

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        val notification = createNotification()
        startForeground(1, notification)
        doBackgroundWork()
        return START_STICKY
    }

    private fun createNotification(): Notification {
        val channelId = "my_channel_id"
        val notificationManager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                channelId,
                "My Channel",
                NotificationManager.IMPORTANCE_DEFAULT
            )
            notificationManager.createNotificationChannel(channel)
        }

        return NotificationCompat.Builder(this, channelId)
            .setContentTitle("My Foreground Service")
            .setContentText("Service is running")
            .setSmallIcon(R.drawable.ic_notification)
            .build()
    }

    private fun doBackgroundWork() {
        // Логика фоновой задачи
    }

    override fun onBind(intent: Intent?): IBinder? {
        return null
    }
}
```

```kotlin
val intent = Intent(this, MyForegroundService::class.java)
startService(intent)
```

---

## IntentService (Устарел)

**IntentService** — это упрощенный способ создания `Started Service`, который обрабатывает задачи в отдельном потоке.
Однако он устарел начиная с `Android 8.0` (`API 26`). 
Вместо него рекомендуется использовать `WorkManager` или `JobIntentService`.

---

## WorkManager — Управление фоновыми задачами

**WorkManager** — это современный способ выполнения фоновых задач, который учитывает версию `Android` и состояние устройства.

**WorkManager** — это часть `Android Jetpack`, которая предоставляет `API` для выполнения отложенных, периодических и гарантированных фоновых задач. 
Он подходит для задач, которые должны быть выполнены даже после закрытия приложения или перезагрузки устройства.

**WorkManager** — это мощная библиотека для выполнения фоновых задач в `Android`, которая учитывает состояние устройства (например, заряд батареи, доступность сети) и гарантирует выполнение задач даже после перезапуска приложения.

### Когда использовать?

- Для задач, которые должны быть выполнены даже если приложение закрыто или устройство перезагружено.
- Например, синхронизация данных, отправка логов на сервер.

### Основные особенности WorkManager

 - **Гарантированное выполнение** - `WorkManager` гарантирует выполнение задач, даже если приложение закрыто или устройство перезагружено.
 - **Гибкость** - Поддерживает одноразовые (`OneTimeWorkRequest`) и периодические (`PeriodicWorkRequest`) задачи.
 - **Учет состояния устройства** - `WorkManager` учитывает заряд батареи, доступность сети и другие факторы для оптимизации выполнения задач.
 - **Совместимость** - Работает на всех версиях `Android`, начиная с `API 14` (`Android 4.0`).
 - **Интеграция с другими компонентами** - Легко интегрируется с `LiveData`, `RxJava`, `Coroutines` и другими библиотеками.

### Основные компоненты WorkManager

 - **Worker** - Класс, который выполняет фоновую задачу. Переопределите метод `doWork()`, чтобы определить логику задачи.
 - **WorkRequest** - Запрос на выполнение задачи. Бывает двух типов:

 - - **OneTimeWorkRequest** — для одноразовых задач.
 - - **PeriodicWorkRequest** — для периодических задач (минимальный промежуток 15 минут).

 - **WorkManager** - Основной класс для постановки задач в очередь и управления их выполнением.
 - **WorkInfo** - Класс, который предоставляет информацию о состоянии задачи (например, выполнена, в процессе, отменена).

### Кратко

**WorkManager** — Библиотека для выполнения фоновых задач с гарантированным выполнением.

Основные компоненты:

 - **Worker** — Выполняет задачу.
 - **WorkRequest** — Запрос на выполнение задачи.
 - **WorkManager** — Управляет задачами.
 - **WorkInfo** — Отслеживает состояние задачи.

Типы задач:

 - **OneTimeWorkRequest** — Одноразовая задача.
 - **PeriodicWorkRequest** — Периодическая задача.

Особенности:

- Учет состояния устройства (сеть, заряд батареи).
- Поддержка цепочек задач.
- Передача данных через Data.

---

## Сравнение сервисов и WorkManager

### Когда использовать сервисы?

**Started Service:**

- Для одноразовых задач, которые не требуют взаимодействия с `UI`.
- Например, загрузка файлов, синхронизация данных.
- Ограничение: Начиная с `Android 8.0`, `Started Service` ограничен в фоновом режиме.

**Bound Service:**

- Для задач, которые требуют взаимодействия с `UI`.
- Например, воспроизведение музыки, передача данных между компонентами.

**Foreground Service:**

- Для задач, которые должны быть видны пользователю и выполняться даже если приложение закрыто.
- Например, воспроизведение музыки, загрузка файлов.

### Когда использовать WorkManager?

- Для задач, которые должны быть выполнены даже если приложение закрыто или устройство перезагружено.
- Для задач, которые требуют учета состояния устройства (сеть, заряд батареи).
- Для периодических задач (например, синхронизация данных каждые 24 часа).
- Для задач, которые не требуют немедленного выполнения (например, отправка логов на сервер).

| **Критерий**                   | **Сервисы**                                | **WorkManager**                    |
|--------------------------------|--------------------------------------------|------------------------------------|
| **Гарантированное выполнение** | Нет (**Started Service** ограничен)        | Да                                 |
| **Учет состояния устройства**  | Нет                                        | Да                                 |
| **Периодические задачи**       | Нет                                        | Да                                 |
| **Взаимодействие с UI**        | Да (**Bound Service**)                     | Нет                                |
| **Видимость для пользователя** | Да (**Foreground Service**)                | Нет                                |
| **Совместимость**              | Требует `API 26+` для `Foreground Service` | Работает на всех версиях `Android` |


### **Рекомендации**

1. **Используйте сервисы, если:**
    - Вам нужно взаимодействие с UI (Bound Service).
    - Вам нужно выполнить задачу, которая должна быть видна пользователю (Foreground Service).

2. **Используйте WorkManager, если:**
    - Вам нужно гарантированное выполнение задачи.
    - Вам нужно выполнить задачу с учетом состояния устройства.
    - Вам нужно выполнить периодическую задачу.

---

#### [README](README.md) [UP](#up)
