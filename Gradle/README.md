# Gradle
<a name="up"></a>

---

**build.gradle.kts** — это файл конфигурации сборки для проектов на `Kotlin`. 
Он заменяет традиционный `build.gradle` (на `Groovy`) и использует `Kotlin DSL` (**`Domain Specific Language`**) для настройки проекта.
`Kotlin DSL` предоставляет более строгую типизацию и лучшую поддержку `IDE`.

### Основные блоки в build.gradle.kts

- **plugins** - Подключает плагины (например, `android`, `kotlin`, `ksp`).
- **android** - Настройка Android-приложения (например, `compileSdk`, `defaultConfig`).
- **dependencies** - Подключает зависимости (библиотеки).
- **repositories** - Указывает репозитории для загрузки зависимостей.

---

## Android

Блок **android** содержит настройки, специфичные для `Android`-приложений.

### Основные настройки:

- **compileSdk** - Версия `SDK`, используемая для компиляции.
- **defaultConfig** - Настройки по умолчанию для приложения.
- **applicationId** - Уникальный идентификатор приложения.
- **minSdk** - Минимальная версия `SDK`, на которой может работать приложение.
- **targetSdk** - Версия `SDK`, на которую ориентировано приложение.
- **versionCode** - Внутренний номер версии приложения.
- **versionName** - Публичное имя версии приложения.
- **buildTypes** - Настройки для разных типов сборки (например, `release` и `debug`).
- **isMinifyEnabled** - Включает минификацию кода.
- **proguardFiles** - Указывает файлы с правилами `ProGuard`/`R8`.

---

## Plugins

**Плагины** — это модули, которые добавляют функциональность в проект. 
Они могут настраивать сборку, добавлять задачи или интегрировать сторонние инструменты.

- **com.android.application** - Плагин для сборки `Android`-приложений.
- **kotlin-android** - Поддержка `Kotlin` в `Android`.
- **ksp** - Плагин для **`Kotlin Symbol Processing`** (`KSP`).


---

## Dependencies

Блок **dependencies** подключает библиотеки и другие зависимости, необходимые для проекта.

---

## Repositories

Блок **repositories** указывает, откуда загружать зависимости.
По умолчанию используется **`Maven Central`** и **`Google Maven Repository`**.

---

## KSP (Kotlin Symbol Processing)

**KSP** — это замена `kapt` (**`Kotlin Annotation Processing Tool`**). 
Он работает быстрее и поддерживает больше возможностей `Kotlin`.

---

## secrets.properties

**secrets.properties** — это файл для хранения конфиденциальных данных (например, `API`-ключей).
Он не добавляется в систему контроля версий (например, `Git`), чтобы избежать утечки данных.

```kotlin
// file("secrets.properties")
val secretsProperties = rootDir.resolve("secrets.properties")
    .bufferedReader()
    .use { buffer: BufferedReader ->
        Properties().apply {
            load(buffer)
        }
    }

buildConfigField("String", "API_KEY", secretsProperties.getProperty("API_KEY"))
buildConfigField("String", "URL_SERVER", secretsProperties.getProperty("URL_SERVER"))
```

---

## BuildConfig

**BuildConfig** — это класс, который генерируется автоматически во время сборки.
Он содержит константы, определенные в `build.gradle.kts`.

---

#### [README](README.md) [UP](#up)
