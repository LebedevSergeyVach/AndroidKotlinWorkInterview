# JSON, Kotlinx Serialization и Data классы
<a name="up"></a>

---

**JSON (JavaScript Object Notation)** — это текстовый формат для хранения и передачи данных. 
Он широко используется в веб-приложениях для обмена данными между клиентом и сервером.

---

## Библиотека kotlinx.serialization

**kotlinx.serialization** — это официальная библиотека `Kotlin` для сериализации и десериализации данных. 
Она поддерживает различные форматы, включая `JSON`, и позволяет легко преобразовывать объекты Kotlin в `JSON` и обратно.

### Основные возможности:

 - **Сериализация** - Преобразование объектов `Kotlin` в `JSON`.
 - **Десериализация** - Преобразование `JSON` в объекты `Kotlin`.
 - **Аннотации** - Упрощают настройку процесса сериализации.

### Plugin

```kotlin
serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
```

```kotlin
plugins {
    alias(libs.plugins.serialization)
}
```

### Библиотека kotlinx.serialization

```kotlin
kotlinx-serialization-json = { module = "org.jetbrains.kotlinx:kotlinx-serialization-json", version.ref = "kotlinxSerializationJson" }
```

```kotlin
/**
 * Serialization
 *
 * https://github.com/Kotlin/kotlinx.serialization
 * https://kotlinlang.org/docs/serialization.html
 */
implementation(libs.kotlinx.serialization.json)
```


### Retrofit Serialization

```kotlin
retrofit2-kotlinx-serialization-converter = { module = "com.jakewharton.retrofit:retrofit2-kotlinx-serialization-converter", version.ref = "retrofit2KotlinxSerializationConverter" }
```

```kotlin
/**
 * Retrofit
 *
 * https://github.com/square/retrofit
 * https://square.github.io/retrofit/
 *
 * https://github.com/JakeWharton/retrofit2-kotlinx-serialization-converter
 * https://github.com/square/retrofit/tree/trunk/retrofit-converters/kotlinx-serialization
 */
implementation(libs.retrofit)
implementation(libs.retrofit2.kotlinx.serialization.converter)
```

---

## Data-классы и аннотации

### @Serializable

Аннотация `@Serializable` указывает, что класс может быть сериализован и десериализован.

### @SerialName

Аннотация `@SerialName` позволяет указать имя поля в JSON, если оно отличается от имени свойства в классе.

```kotlin
/**
 * Класс, представляющий пост в приложении.
 * Этот класс используется для хранения данных о посте, включая его идентификатор, автора, дату публикации,
 * содержание и информацию о лайках.
 *
 * @property id Уникальный идентификатор поста. По умолчанию 0L.
 * @property author Имя автора поста. По умолчанию пустая строка.
 * @property authorId Уникальный идентификатор автора поста. По умолчанию 0L.
 * @property authorAvatar Ссылка на аватар автора поста. По умолчанию пустая строка.
 * @property published Дата и время публикации поста. По умолчанию текущее время.
 * @property content Текстовое содержание поста. По умолчанию пустая строка.
 * @property likedByMe Флаг, указывающий, лайкнул ли текущий пользователь этот пост. По умолчанию false.
 * @property likeOwnerIds Множество идентификаторов пользователей, которые лайкнули этот пост. По умолчанию пустое множество.
 * @property attachment Вложение (например, изображение или видео), связанное с постом. По умолчанию null.
 *
 * @see Attachment
 * @see InstantSerializer
 */
@Serializable
data class PostData(
    @SerialName("id")
    val id: Long = 0L,
    @SerialName("author")
    val author: String = "",
    @SerialName("authorId")
    val authorId: Long = 0L,
    @SerialName("authorAvatar")
    val authorAvatar: String = "",
    @SerialName("published")
    @Serializable(with = InstantSerializer::class)
    val published: Instant = Instant.now(),
    @SerialName("content")
    val content: String = "",
    @SerialName("likedByMe")
    val likedByMe: Boolean = false,
    @SerialName("likeOwnerIds")
    val likeOwnerIds: Set<Long> = emptySet(),
    @SerialName("attachment")
    val attachment: Attachment? = null,
)
```

---

## Сериализация и десериализация

### Сериализация (объект → JSON)

Используйте метод `Json.encodeToString`, чтобы преобразовать объект в `JSON`-строку.

```kotlin
import kotlinx.serialization.encodeToString
import kotlinx.serialization.json.Json

fun main() {
    val user = User("Sergey", 18, true)
    val json = Json.encodeToString(user)
    println(json) // {"name":"Sergey","age":18,"isStudent":true}
}
```

### Десериализация (JSON → объект)

Используйте метод `Json.decodeFromString`, чтобы преобразовать `JSON`-строку в объект.

```kotlin
import kotlinx.serialization.decodeFromString
import kotlinx.serialization.json.Json

fun main() {
    val json = """{"name":"Sergey","age":18,"isStudent":true}"""
    val user = Json.decodeFromString<User>(json)
    println(user) // User(name=Sergey, age=18, isStudent=true)
}
```

---

## Настройка JSON

Библиотека `kotlinx.serialization` позволяет настраивать процесс сериализации и десериализации с помощью объекта `Json`.

```kotlin
/**
 * Конфигурация JSON-сериализации.
 *
 * - `ignoreUnknownKeys = true`: Игнорирует неизвестные ключи в JSON.
 * - `coerceInputValues = true`: Приводит значения `null` к значениям по умолчанию.
 */
private val json = Json {
    ignoreUnknownKeys = true
    coerceInputValues = true
}
```

```kotlin
val json = Json {
    prettyPrint = true // Красивый вывод JSON с отступами
    ignoreUnknownKeys = true // Игнорировать неизвестные ключи в JSON
    isLenient = true // Разрешить нестрогий формат JSON (например, без кавычек)
}

val user = User("Sergey", 25, true)
val jsonString = json.encodeToString(user)
println(jsonString)
```

---

## Вложенные объекты и коллекции

### Вложенные объекты

Можно сериализовать и десериализовать вложенные объекты.

```kotlin
@Serializable
data class Address(
    val city: String,
    val street: String
)

@Serializable
data class User(
    val name: String,
    val age: Int,
    val address: Address
)

fun main() {
    val user = User("Sergey", 25, Address("Moscow", "Lenina"))
    val json = Json.encodeToString(user)
    println(json) // {"name":"Sergey","age":25,"address":{"city":"Moscow","street":"Lenina"}}
}
```

### Коллекции

Библиотека поддерживает сериализацию и десериализацию коллекций (например, списков).

```kotlin
@Serializable
data class User(
    val name: String,
    val age: Int
)

fun main() {
    val users = listOf(
        User("Sergey", 25),
        User("Anna", 30)
    )
    val json = Json.encodeToString(users)
    println(json) // [{"name":"Sergey","age":25},{"name":"Anna","age":30}]
}
```

---

## Обработка null-значений

Если свойство может быть `null`, нужно использовать `nullable-тип`.

```kotlin
@Serializable
data class User(
    val name: String,
    val age: Int? // Возраст может быть null
)

fun main() {
    val json = """{"name":"Sergey"}"""
    val user = Json.decodeFromString<User>(json)
    println(user) // User(name=Sergey, age=null)
}
```

---

## Кастомные сериализаторы

Если стандартной сериализации недостаточно, можно создать кастомный сериализатор.

```kotlin
/**
 * Сериализатор для класса [Instant].
 *
 * Этот сериализатор используется для преобразования объектов [Instant] в строку и обратно.
 * Он реализует интерфейс [KSerializer] из библиотеки Kotlinx Serialization.
 *
 * @see KSerializer Интерфейс для создания пользовательских сериализаторов.
 * @see Instant Класс, представляющий момент времени.
 */
class InstantSerializer : KSerializer<Instant> {

    /**
     * Описание сериализации для [Instant].
     *
     * Это описание указывает, что [Instant] будет сериализован как примитивный тип [PrimitiveKind.STRING].
     */
    override val descriptor: SerialDescriptor =
        PrimitiveSerialDescriptor("Instant", PrimitiveKind.STRING)

    /**
     * Метод для десериализации строки в объект [Instant].
     *
     * @param decoder Декодер, используемый для чтения данных.
     * @return Объект [Instant], полученный из строки.
     */
    override fun deserialize(decoder: Decoder): Instant =
        Instant.parse(decoder.decodeString())

    /**
     * Метод для сериализации объекта [Instant] в строку.
     *
     * @param encoder Энкодер, используемый для записи данных.
     * @param value Объект [Instant], который нужно сериализовать.
     */
    override fun serialize(encoder: Encoder, value: Instant) =
        encoder.encodeString(value.toString())
}
```

---

## Кратко

**JSON** — Текстовый формат для хранения и передачи данных.

**kotlinx.serialization** — Библиотека для сериализации и десериализации в Kotlin.

### Аннотации:

 - **@Serializable** — Для сериализуемых классов.
 - **@SerialName** — Для указания имени поля в JSON.


 - **Сериализация** - `Json.encodeToString`.
 - **Десериализация** - `Json.decodeFromString`.


 - **Настройка JSON** - использование объекта `Json` для настройки.
 - **Вложенные объекты и коллекции** - Поддерживаются автоматически.
 - **Кастомные сериализаторы** - Для нестандартных типов данных.

---

#### [README](README.md) [UP](#up)
