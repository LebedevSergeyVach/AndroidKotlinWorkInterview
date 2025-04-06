# Generics
<a name="up"></a>

---

**Дженерики (Generics)** — это мощный инструмент в `Kotlin` (и `Java`), который позволяет создавать обобщённые классы, функции и интерфейсы, работающие с разными типами данных.
Они обеспечивают типобезопасность и избегают дублирования кода.

---

## Зачем нужны дженерики?

### Проблема без дженериков:

```kotlin
class Box {
    private var item: Any? = null

    fun put(item: Any?) {
        this.item = item
    }

    fun get(): Any? = item
}

val box = Box()
box.put("Hello")
val value: String = box.get() as String // Небезопасное приведение!
```

**Минусы**:

 - Нужно вручную приводить типы (`as String`).
 - Риск ошибок (`ClassCastException`).

### Решение с дженериками:

```kotlin
class Box<T> {
    private var item: T? = null

    fun put(item: T?) {
        this.item = item
    }

    fun get(): T? = item
}

val box = Box<String>() // Указываем тип явно
box.put("Hello")
val value: String? = box.get() // Безопасно!
```

--

## Основные понятия

### Обобщённые классы

Класс, который работает с произвольным типом `T`:

```kotlin
class Container<T>(private val item: T) {
    fun getItem(): T = item
}

val intContainer = Container(42) // Тип выводится как Int
val stringContainer = Container("Kotlin") // Тип String
```

### Обобщённые функции

Функции тоже могут быть обобщёнными:

```kotlin
fun <T> printItem(item: T) {
    println(item)
}

printItem(10) // T = Int
printItem("Text") // T = String
```

### Ограничения типов (Upper Bounds)

Можно указать, что тип `T` должен быть наследником определённого класса/интерфейса:

```kotlin
fun <T : Number> sum(a: T, b: T): Double {
    return a.toDouble() + b.toDouble()
}

println(sum(5, 10)) // 15.0
// sum("A", "B") // Ошибка: String не является Number
```

Несколько ограничений (через `where`):

```kotlin
fun <T> compareItems(a: T, b: T) where T : Comparable<T>, T : Any {
    println(a.compareTo(b))
}
```

---

## Вариантность (Variance)

### Инвариантность (Invariant)

По умолчанию обобщённые типы в `Kotlin` инвариантны: `Box<Int>` и `Box<Number>` — это разные типы, даже если `Int` наследуется от `Number`.

```kotlin
class Box<T>(val item: T)

val intBox = Box(42)
// val numberBox: Box<Number> = intBox // Ошибка!
```

### Ковариантность (Covariant)

Если `Box<Int>` можно присвоить `Box<Number>`, это ковариантность. Используется `out`:

```kotlin
class Box<out T>(val item: T)

val intBox = Box(42)
val numberBox: Box<Number> = intBox // OK, так как T — только на выходе
```

**Правило**: `T` может быть только возвращаемым типом (`producer`).

### Контравариантность (Contravariant)

Если `Box<Number>` можно присвоить `Box<Int>`, это контравариантность. Используется `in`:

```kotlin
class Box<in T> {
    fun consume(item: T) { /*...*/ }
}

val numberBox = Box<Number>()
val intBox: Box<Int> = numberBox // OK, так как T — только на входе
```

**Правило**: `T` может быть только параметром (`consumer`).

---

## Звёздный проекции (*)

Когда тип неизвестен, но нужно работать с обобщённым классом:

```kotlin
fun printItems(list: List<*>) {
    for (item in list) {
        println(item)
    }
}

printItems(listOf(1, 2, 3))
printItems(listOf("A", "B"))
```

---

## Reified типы

В обычных функциях тип `T` стирается во время выполнения, но с `reified` его можно использовать внутри функции:

```kotlin
inline fun <reified T> checkType(item: Any) {
    if (item is T) {
        println("Это ${T::class.simpleName}")
    }
}

checkType<String>("Hello") // Это String
checkType<Int>(10) // Это Int
```

**Условия**:

 - Функция должна быть `inline`.
 - Тип должен быть `reified`.

---

## Примеры из стандартной библиотеки

### Коллекции

```kotlin
val list: List<String> = listOf("A", "B") // Обобщённый List
val map: Map<Int, String> = mapOf(1 to "One") // Обобщённый Map
```

### Функции высшего порядка

```kotlin
fun <T> List<T>.filter(predicate: (T) -> Boolean): List<T> {
    val result = mutableListOf<T>()
    for (item in this) {
        if (predicate(item)) result.add(item)
    }
    return result
}

val numbers = listOf(1, 2, 3, 4)
val even = numbers.filter { it % 2 == 0 } // List<Int>
```

---

## Что означают `T`, `R`, `E`, `K`, `V` в дженериках?

В дженериках `T`, `R` и другие буквы — это просто **условные обозначения типов**, которые помогают сделать код более читаемым. 
Они не имеют специального смысла в языке, но есть негласные соглашения среди разработчиков.

Это **параметры типов** (`type parameters`), и их выбор зависит от контекста:

| Буква   | Обычное значение                 | Пример использования           |
|---------|----------------------------------|--------------------------------|
| **`T`** | **Type** (основной тип)          | `class Box<T>`                 |
| **`R`** | **Return type** (тип результата) | `fun <T, R> map(item: T): R`   |
| **`E`** | **Element** (элемент коллекции)  | `interface List<E>`            |
| **`K`** | **Key** (ключ в мапе)            | `class Map<K, V>`              |
| **`V`** | **Value** (значение в мапе)      | `class Map<K, V>`              |
| **`U`** | Второй тип (если `T` уже занят)  | `fun <T, U> merge(a: T, b: U)` |

### Пример с `T` и `R`

Допустим, у нас есть функция, которая преобразует тип `T` в тип `R`:

```kotlin
fun <T, R> transform(input: T, converter: (T) -> R): R {
    return converter(input)
}

val number = 42
val text = transform(number) { it.toString() } // T = Int, R = String
println(text) // "42"
```

**Здесь**:

- `T` — входной тип (`Int`).
- `R` — тип результата (`String`).

### **2. Почему именно такие буквы?**

- **`T`** — исторически сложилось как сокращение от **Type** (начиная с ``Java``).
- **`R`** — **Result** или **Return**, чтобы отличать от входного типа.
- **`E`**, **`K`**, **`V`** — соглашения из стандартных коллекций (например, `List<E>`, `Map<K, V>`).

### Примеры из стандартной библиотеки

 - **Функция `map` в коллекциях**

```kotlin
fun <T, R> Iterable<T>.map(transform: (T) -> R): List<R>
```

- `T` — тип элемента в исходной коллекции.
- `R` — тип элемента в новой коллекции.

 - **Класс `Pair`****

```kotlin
data class Pair<out A, out B>(val first: A, val second: B)
```

- `A` и `B` — типы первого и второго элементов.

### Важные нюансы

1. **Порядок имеет значение**  

   Например, в `fun <T, R> foo()`:
    - `T` — первый тип,
    - `R` — второй тип.

2. **Можно использовать слова вместо букв**  

   Например, `<InputType, OutputType>` — но это редко встречается.

3. **Ограничения**  

   Если тип должен реализовывать интерфейс, указываем:

   ```kotlin
   fun <T : Comparable<T>> sort(list: List<T>) { /*...*/ }
   ```
---

### Итог: ключевые моменты

| Концепция              | Пример                           | Описание                                  |
|------------------------|----------------------------------|-------------------------------------------|
| **Обобщённый класс**   | `class Box<T>`                   | Работает с любым типом `T`.               |
| **Обобщённая функция** | `fun <T> print(item: T)`         | Функция принимает любой тип.              |
| **Ограничения**        | `<T : Number>`                   | `T` должен быть подтипом `Number`.        |
| **Ковариантность**     | `class Box<out T>`               | `Box<Int>` можно присвоить `Box<Number>`. |
| **Контравариантность** | `class Box<in T>`                | `Box<Number>` можно присвоить `Box<Int>`. |
| **Reified**            | `inline fun <reified T> check()` | Сохраняет тип `T` во время выполнения.    |

---

#### [README](README.md) [UP](#up)
