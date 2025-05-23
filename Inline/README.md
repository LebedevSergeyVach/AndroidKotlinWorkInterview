# inline-функции и value-классы
<a name="up"></a>

---

Две мощные возможности `Kotlin`: `inline`-функции и `value`-классы.
Они помогают оптимизировать производительность и снижать накладные расходы.

#### [Обязательно изучить материал здесь: habr.com - Что такое inline функции, в чем их преимущество?](https://habr.com/ru/articles/736392/#Что%20такое%20inline%20функции,%20в%20чем%20их%20преимущество?)

### Как работают inline функции?

Использование анонимных функций (лямбда-выражений) в `Kotlin` приводит к дополнительным затратам памяти. 
При использовании лямбда-выражения создается объект `FunctionN` (где `N` — количество параметров в лямбда-выражении),
который содержит ссылку на само лямбда-выражение и может содержать захваченные переменные.
При передаче лямбда-выражения в качестве параметра метода также создается новый объект `FunctionN`, что приводит к дополнительным затратам памяти.

Поэтому, чтобы избежать создания дополнительных объектов при передаче лямбда-выражений в функцию в качестве параметра, можно использовать встраивание (`inline`).
Ключевое слово `inline` позволяет компилятору подставить тело функции непосредственно в место её вызова, вместо того, чтобы создавать объекты функций.
Таким образом можно уменьшить затраты на создание объектов и улучшить производительность приложения.

**Пример синтаксиса inline-функций с лямбдой:**

```kotlin
inline fun functionName(parameter1: Type1, parameter2: Type2, ..., parameterN: TypeN, block: () -> Unit): ReturnType {
    // function body
}
```

Модификатор `inline` влияет и на функцию, и на лямбду, переданную ей: они обе будут встроены в место вызова.

---

## Inline-функции в Kotlin

### 1. Зачем нужно inline?

**Проблема:**

Лямбды в `Kotlin` создают анонимные классы, что может привести к:

 - Нагрузке на `GC` (лишние объекты).
 - Оверхеду вызова (особенно в горячих участках кода).

**Решение:**
`inline`-функции встраивают код на этапе компиляции, избегая создания объектов.

### 2. Как работает inline?

```kotlin
inline fun measureTime(action: () -> Unit) {
    val start = System.currentTimeMillis()
    action()
    println("Time: ${System.currentTimeMillis() - start} ms")
}

// При компиляции вызов заменится на:
val start = System.currentTimeMillis()
println("Hello!")  // Тело лямбды встроено напрямую
println("Time: ${System.currentTimeMillis() - start} ms")
```

### 3. Ограничения

 - Нельзя использовать inline для рекурсивных функций (бесконечное встраивание).
 - Большие функции увеличивают размер бинарника (используйте для коротких функций).

### 4. noinline и crossinline

 - **noinline** — отключает встраивание для конкретной лямбды:

Если же вы хотите, чтобы некоторые лямбды, переданные `inline`-функции, не были встроены, то отметьте их модификатором `noinline`.

Разница между ними в том, что встраиваемая лямбда может быть вызвана только внутри `inline`-функции, либо может быть передана в качестве встраиваемого аргумента. 
В то время как с `noinline`-функциями можно работать без ограничений: хранить внутри полей, передавать куда-либо и т.д.

```kotlin
inline fun foo(noinline block: () -> Unit) { /* ... */ }

inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) {
    // ...
}
```

 - **crossinline** — гарантирует, что лямбда не содержит `return`:

`crossinline` — ключевое слово, которое используется для указания, что лямбда-выражение не может содержать нелокальных `return`, даже если оно передано в `inline`-функцию.
Когда мы передаем лямбда-выражение в функцию в качестве параметра, мы можем использовать оператор `return` внутри лямбды, чтобы выйти из цикла или функции, в которой вызывается лямбда. 
Однако, если мы передаем лямбда-выражение в `inline`-функцию, код лямбда-выражения может быть вставлен прямо в место вызова функции.
В этом случае, если в лямбде используется оператор `return`, это может привести к выходу из внешней функции, что не всегда желательно.

```kotlin
inline fun runAsync(crossinline block: () -> Unit) {
    thread { block() }  // Нельзя выйти из внешней функции
}
```

### 5. Пример из Android

```kotlin
inline fun View.onClick(crossinline action: () -> Unit) {
    setOnClickListener { action() }  // Избегаем создания лишнего Listener
}
```

---

## Value-классы (@JvmInline value class)

### 1. Зачем нужно?

Оптимизация для обёрток над примитивами/строками, чтобы:

 - Избегать аллокаций (объект заменяется на значение при компиляции).
 - Сохранять типобезопасность (например, `UserId` vs `Long`).

### 2. Синтаксис

```kotlin
@JvmInline
value class Password(val raw: String)  // Обёртка над String

fun login(password: Password) { /* ... */ }

// Использование
login(Password("123"))  // Тип проверяется на этапе компиляции
```

```kotlin
@JvmInline
value class UserId(val id: Int)

fun getUser(id: UserId) { /* ... */ }

// Компилируется в Java как примитив `int`!
```

### 3. Ограничения

 - Одно поле в конструкторе.
 - Не могут быть `null` (если не обёрнуты в `Nullable`).
 - Только `val` (изменяемые `var` запрещены).

---

## 4. Под капотом

**Без `value`-класса:**

```java
public final class UserId {
    private final int id;
    // Геттеры, аллокации...
}
```

**С `value`-классом:**

```java
public static final void getUser(int id) { /* ... */ }
```

---

## Комбо: Inline + Value-классы

**Сценарий: Оптимизация API-клиента**

```kotlin
@JvmInline
value class ApiKey(val raw: String)

inline fun <reified T> getData(
    key: ApiKey,
    crossinline onSuccess: (T) -> Unit
) {
    // Высокопроизводительный парсинг
    val data = parseJson<T>(fetchData(key))
    onSuccess(data)
}
```

**Что даёт:**
 - Нет аллокаций для ApiKey.
 - Нет оверхеда для лямбды onSuccess.

### Сравнение с аналогами

| **Техника**        | **Плюсы**             | **Минусы**                    |
|--------------------|-----------------------|-------------------------------|
| **inline-функции** | Убирает оверхед лямбд | Увеличивает размер кода       |
| **value-классы**   | Избегает аллокаций    | Ограниченная функциональность |
| **Обычные классы** | Полная свобода        | Нагрузка на `GC`              |

### Когда что использовать?

- **`inline`**:
    - Часто вызываемые функции с лямбдами (например, `map`, `filter`).
    - Утилиты для `Android` (например, `View.onClick`).
- **`value class`**:
    - Обёртки над примитивами (`UserId`, `Password`).
    - `DTO` для `API` (например, `JsonToken`).

---

## **🚀 Примеры из реального мира**

### 1. Inline-функции в Kotlin Stdlib

```kotlin
inline fun <T, R> Iterable<T>.map(transform: (T) -> R): List<R> {
    val result = ArrayList<R>()
    for (item in this) result.add(transform(item))
    return result
}
```

### 2. Value-классы в Android

```kotlin
@JvmInline
value class Px(val value: Int) {
    fun toDp(context: Context): Float = value / context.resources.displayMetrics.density
}
```

---

## Итог

- **`inline`** — для оптимизации лямбд.
- **`value class`** — для типобезопасности без оверхеда.
- **Комбинация** — максимум производительности в критичных участках.

**Эти инструменты особенно полезны в:**

- `Android` (`UI`, многопоточность).
- Высоконагруженных сервисах.
- Библиотеках (например, `Kotlinx.Coroutines`).

---

#### [README](README.md) [UP](#up)
