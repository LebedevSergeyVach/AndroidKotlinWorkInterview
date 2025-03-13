# Коллекции и Лямбда-выражения
<a name="up"></a>

---

## Коллекции в Kotlin

**Коллекции** — это структуры данных, которые хранят набор элементов.

### В `Kotlin` коллекции делятся на:

- **Неизменяемые** (**`Immutable`**) — нельзя изменять после создания.
- **Изменяемые** (**`Mutable`**) — можно изменять после создания.

### Основные типы коллекций:

- **Списки** (**`List`**) — упорядоченные коллекции с доступом по индексу.
- **Множества** (**`Set`**) — неупорядоченные коллекции уникальных элементов.
- **Ассоциативные массивы** (**`Map`**) — пары ключ-значение.

### Примеры

- `listOf` — создает неизменяемый список. Элементы нельзя добавлять, удалять или изменять.
- `mutableListOf` — создает изменяемый список. Элементы можно добавлять, удалять и изменять.


- `setOf` — создает неизменяемое множество. Хранит только уникальные элементы.
- `mutableSetOf` — создает изменяемое множество.


- `mapOf` — создает неизменяемый Map. Пары ключ-значение.
- `mutableMapOf` — создает изменяемый Map.

---

## Лямбда-выражения

**Лямбда-выражения** — это анонимные функции`*`, которые можно передавать как аргументы или сохранять в переменных. 
Они упрощают код и делают его более читаемым.

`*` **Анонимные функции** - не имеют имени и определяются в том месте, где используются.=

```
{ параметр -> тело }
```

```kotlin
val sum = { a: Int, b: Int -> a + b }
println(sum(2, 3)) // 5
```

---

## Лямбды и коллекции

**Лямбда-выражения** часто используются для обработки коллекций. 

---

`*` **Функции высшего порядка**:
- принимают функцию как аргумент
- возвращают функцию

В `Kotlin` функции могут быть переданы как объекты, что делает их очень гибкими и мощными.

#### Функция, принимающая другую функцию как аргумент

Рассмотрим пример функции, которая принимает лямбду (анонимную функцию) как аргумент:

```kotlin
fun operateOnNumbers(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}
```

- `operation: (Int, Int) -> Int` — это параметр функции, который представляет собой лямбду. Она принимает два `Int` и возвращает `Int`.
- `operation(a, b)` — вызов переданной лямбды.

```kotlin
val sum = operateOnNumbers(5, 3) { x, y -> x + y }
println(sum) // 8

val multiply = operateOnNumbers(5, 3) { x, y -> x * y }
println(multiply) // 15
```

#### Функция, возвращающая другую функцию

Теперь рассмотрим пример функции, которая возвращает лямбду:

```kotlin
fun createMultiplier(factor: Int): (Int) -> Int {
    return { number -> number * factor }
}
```

- `createMultiplier` возвращает лямбду, которая умножает число на `factor`.

```kotlin
val double = createMultiplier(2)
println(double(5)) // 10

val triple = createMultiplier(3)
println(triple(5)) // 15
```

---

Вот основные функции высшего`*` порядка (`higher-order functions`), которые работают с коллекциями:

- **`forEach`** - Применяет лямбду к каждому элементу коллекции.

```kotlin
val numbers = listOf(1, 2, 3)
numbers.forEach { println(it) } // 1 2 3
```

- **`map`** - Преобразует каждый элемент коллекции.

```kotlin
val doubled = numbers.map { it * 2 }
println(doubled) // [2, 4, 6]
```

- **`filter`** - Фильтрует элементы по условию.

```kotlin
val evenNumbers = numbers.filter { it % 2 == 0 }
println(evenNumbers) // [2]
```

- **`reduce`** - Сворачивает коллекцию в одно значение.

```kotlin
val sum = numbers.reduce { acc, number -> acc + number }
println(sum) // 6
```

- **`sortedBy`** - Сортирует коллекцию по заданному критерию.

```kotlin
val names = listOf("Alice", "Bob", "Charlie")
val sortedNames = names.sortedBy { it.length }
println(sortedNames) // [Bob, Alice, Charlie]
```

---

### Лямбды с несколькими параметрами

Если лямбда принимает несколько параметров, их можно указать в скобках:

```kotlin
val multiply: (Int, Int) -> Int = { a, b -> a * b }
println(multiply(3, 4)) // 12
```

---

### Лямбды как аргументы функций

Лямбды можно передавать в функции как аргументы:

```kotlin
fun operateOnNumbers(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

val result = operateOnNumbers(5, 3) { x, y -> x + y }
println(result) // 8
```

---

### Полезные функции для работы с коллекциями

- **`any`** - Проверяет, есть ли хотя бы один элемент, удовлетворяющий условию.

```kotlin
val hasEven = numbers.any { it % 2 == 0 }
println(hasEven) // true
```

- **`all`** - Проверяет, все ли элементы удовлетворяют условию.

```kotlin
val allEven = numbers.all { it % 2 == 0 }
println(allEven) // false
```

- **`none`** - Проверяет, что ни один элемент не удовлетворяет условию.

```kotlin
val noZeros = numbers.none { it == 0 }
println(noZeros) // true
```

- **`flatMap`** - Преобразует каждый элемент в коллекцию и объединяет результаты.

```kotlin
val nestedList = listOf(listOf(1, 2), listOf(3, 4))
val flatList = nestedList.flatMap { it }
println(flatList) // [1, 2, 3, 4]
```

---

#### [README](README.md) [UP](#up)
