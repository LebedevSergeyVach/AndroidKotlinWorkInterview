# Файлы Jetpack Preferences DataStore
<a name="up"></a>

---

##  Внутреннее хранилище (Internal Storage)

**Внутреннее хранилище** — это приватная папка приложения, доступная только этому приложению. 
Данные сохраняются в файловой системе устройства.

---

## Внешнее хранилище (External Storage)

**Внешнее хранилище** — это общая папка, доступная всем приложениям и пользователю. 
Для работы с внешним хранилищем нужно запросить разрешения.

---

## SharedPreferences

**SharedPreferences** — это легковесное хранилище для простых данных (например, настроек). Данные хранятся в формате ключ-значение.

---

## DataStore

**`DataStore`** — это современная замена `SharedPreferences`. Он предоставляет более безопасный и гибкий способ хранения данных. 
`DataStore` поддерживает два типа:

- **Preferences DataStore** - Хранит данные в формате ключ-значение (аналогично `SharedPreferences`).
- **Proto DataStore** - Хранит данные в виде объектов (использует `Protocol Buffers`).

### Preferences DataStore

```kotlin
implementation("androidx.datastore:datastore-preferences:1.0.0")
```

### Proto DataStore

```kotlin
implementation("androidx.datastore:datastore:1.0.0")
implementation("com.google.protobuf:protobuf-javalite:3.18.0")
```

Необходимо работать с файлом `app_prefs.proto`.

---

#### [README](README.md) [UP](#up)
