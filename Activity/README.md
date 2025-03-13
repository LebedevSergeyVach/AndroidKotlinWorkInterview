# Activity и ViewBinding

<a name="up"></a>

---

**Activity** — это компонент `Android`, который представляет собой один экран с пользовательским интерфейсом.
Каждое приложение состоит из одной или нескольких `Activity`, которые взаимодействуют с пользователем.

---

## Жизненный цикл Activity

**Жизненный цикл Activity** — это набор состояний, через которые проходит `Activity` от создания до уничтожения.
Понимание жизненного цикла важно для правильного управления ресурсами и поведением приложения.

### Основные состояния Activity

- **onCreate()**: Вызывается при создании `Activity`. Здесь происходит инициализация `UI` и данных.
- **onStart()**: Вызывается, когда `Activity` становится видимой для пользователя.
- **onResume()**: Вызывается, когда `Activity` переходит в активное состояние и готово к взаимодействию с пользователем.
- **onPause()**: Вызывается, когда `Activity` теряет фокус (например, при открытии другого `Activity`).
- **onStop()**: Вызывается, когда `Activity` больше не видно пользователю.
- **onDestroy()**: Вызывается перед уничтожением `Activity`.
- **onRestart()**: Вызывается, когда `Activity` возвращается из состояния `onStop()`.

### Схема жизненного цикла

```plaintext
onCreate() -> onStart() -> onResume() -> [Active] -> onPause() -> onStop() -> onDestroy()
```

- После `onPause()` `Activity` может вернуться в состояние `onResume()`.
- После `onStop()` `Activity` может вернуться в состояние `onRestart()`.

---

## Навигация между Activity

**Навигация между Activity** — это переход от одного экрана к другому. Это можно сделать с помощью **`Intent`**.

### Переход к новой Activity

```kotlin
val intent = Intent(this, SecondActivity::class.java)
startActivity(intent)
```

### Передача данных между Activity

Из первой Activity:

```kotlin
val intent = Intent(this, SecondActivity::class.java)
intent.putExtra("KEY_NAME", "Сергей")
intent.putExtra("KEY_AGE", 18)
startActivity(intent)
```

Во второй Activity:

```kotlin
class SecondActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_second)

        val name = intent.getStringExtra("KEY_NAME")
        val age = intent.getIntExtra("KEY_AGE", 0)

        Log.d("SecondActivity", "Name: $name, Age: $age")
    }
}
```

###  Возврат результата из Activity

Запуск Activity с ожиданием результата:

```kotlin
val intent = Intent(this, SecondActivity::class.java)
startActivityForResult(intent, REQUEST_CODE)
```

Обработка результата в первой Activity:

```kotlin
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)
    if (requestCode == REQUEST_CODE && resultCode == RESULT_OK) {
        val result = data?.getStringExtra("RESULT_KEY")
        Log.d("MainActivity", "Result: $result")
    }
}
```

Возврат результата из второй Activity:

```kotlin
val intent = Intent()
intent.putExtra("RESULT_KEY", "Данные возвращены")
setResult(RESULT_OK, intent)
finish()
```

---

## ViewBinding

**ViewBinding** — это механизм, который позволяет легко и безопасно обращаться к элементам `UI` в коде. 
Он генерирует классы для каждого `XML`-файла макета, что позволяет избежать использования `findViewById`.

### Преимущества ViewBinding

- **Безопасность** - Исключает ошибки, связанные с неправильными `ID`.
- **Удобство** - Упрощает доступ к элементам `UI`.
- **Производительность** - Не использует рефлексию, в отличие от `findViewById`.


- **ActivityMainBinding** - Класс, сгенерированный `ViewBinding` для `activity_main.xml`.
- **binding.root** - Корневой элемент макета.
- **binding.textView**, **binding.button** - Элементы `UI`, к которым можно обращаться напрямую.

```kotlin
buildFeatures {
    // Для ViewBinding вместо findViewById
    viewBinding = true
}
```

---

#### [README](README.md) [UP](#up)
