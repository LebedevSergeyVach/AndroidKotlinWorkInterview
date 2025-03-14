# RxJava
<a name="up"></a>

---

## RxJava и ReactiveX

**RxJava** — это `реализация` `ReactiveX` (**`Reactive Extensions`**) для `Java` и `Kotlin`. 
**ReactiveX** — это библиотека для работы с асинхронными потоками данных с использованием наблюдаемых последовательностей (`Observables`).


### Основные концепции ReactiveX

- **Observable** - Источник данных, который `emits` (излучает) элементы.
- **Observer** - Подписчик, который получает элементы от `Observable`.
- **Operators** - Операторы для преобразования, фильтрации и комбинирования потоков данных.
- **Schedulers** - Управляют потоками выполнения (например, `main thread`, `background thread`).

---

## RxKotlin

**RxKotlin** — это библиотека, которая добавляет поддержку `Kotlin` в `RxJava`. 
Она предоставляет расширения и функции, которые упрощают работу с `RxJava` в `Kotlin`.

---

## Виды потоков (Observable, Single, Maybe, Completable)

### Observable

**Observable** — это поток, который может излучать ноль или более элементов и завершаться успешно или с ошибкой.

### Single

**Single** — это поток, который излучает ровно один элемент или ошибку.

### Maybe

**Maybe** — это поток, который может излучать ноль или один элемент, либо завершаться с ошибкой.

### Completable

**Completable** — это поток, который не излучает элементы, но может завершиться успешно или с ошибкой.

---

## Retrofit + RxJava

`Retrofit` поддерживает `RxJava`, что позволяет легко интегрировать сетевые запросы с реактивными потоками.

---

## Subscriber

**Subscriber** — это объект, который подписывается на Observable и обрабатывает `emitted` элементы.

---

## Cold и Hot Observables

### Cold Observables

**Cold Observables** начинают излучать элементы только после подписки. Каждый подписчик получает свою копию данных.

### Hot Observables

**Hot Observables** излучают элементы независимо от подписок. Подписчики получают только те элементы, которые были `emitted` после подписки.

---

## Disposable и CompositeDisposable

**Disposable** — это объект, который позволяет отменить подписку на `Observable`.

```kotlin
val disposable = observable.subscribe { item -> println(item) }
disposable.dispose() // Отмена подписки
```

**CompositeDisposable** — это контейнер для нескольких `Disposable`, который позволяет отменить все подписки одновременно.

```kotlin
val compositeDisposable = CompositeDisposable()
compositeDisposable.add(observable.subscribe { item -> println(item) })
compositeDisposable.clear() // Отмена всех подписок
```

---

## Schedulers: subscribeOn и observeOn

**subscribeOn** указывает, на каком потоке выполнять работу (например, сетевой запрос).

Обычно **subscribeOn** вызывается как можно ближе к созданию `Single`/`Completable`/`Observable`. В нашем случае это сделал `Retrofit`.
Рекомендуется выполнять маппинг и прочие вычисления на фоновом потоке. Этот эффект достигается при помощи функции `observeOn`.

```kotlin
observable
    .subscribeOn(Schedulers.io()) // Выполнение на фоновом потоке
    .subscribe { item -> println(item) }
```

**observeOn** указывает, на каком потоке обрабатывать результаты (например, обновление `UI`).

```kotlin
observable
    .subscribeOn(Schedulers.io()) // Выполнение на фоновом потоке
    .observeOn(AndroidSchedulers.mainThread()) // Обработка на основном потоке
    .subscribe { item -> println(item) }
```

---

## 4 вида Schedulers

- **Schedulers.io()** - Используется для `I/O`-операций (например, сетевые запросы, работа с файлами). Неограниченный пул потоков для сетевой работы.
- **Schedulers.computation()** - Используется для вычислительных задач (например, обработка данных). Пул ограничен количеством ядер процессора.
- **Schedulers.newThread()** - Создает новый поток для каждой задачи. Пул не используется, всегда новый поток
- **Schedulers.single()** - Использует один поток для выполнения всех задач. Используется поток для планирования задач.


---

## AndroidSchedulers

**AndroidSchedulers** предоставляет `Schedulers` для работы с основным потоком в `Android`.

```kotlin
observable
    .subscribeOn(Schedulers.io()) // Выполнение на фоновом потоке
    .observeOn(AndroidSchedulers.mainThread()) // Обработка на основном потоке
    .subscribe { item -> println(item) }
```

---

- **RxJava**: Библиотека для реактивного программирования.
- **Observable**, **Single**, **Maybe**, **Completable**: Типы потоков данных.
- **Retrofit** **+** **RxJava**: Интеграция сетевых запросов с `RxJava`.
- **Subscriber**: Обрабатывает элементы потока.
- **Cold**/**Hot** Observables: Разные типы потоков данных.
- **Disposable**/**CompositeDisposable**: Управление подписками.
- **Schedulers**: Управление потоками выполнения.
- **AndroidSchedulers**: Работа с основным потоком в `Android`.

---

#### [README](README.md) [UP](#up)
