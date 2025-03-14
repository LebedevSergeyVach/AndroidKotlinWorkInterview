# Okhttp
<a name="up"></a>

---

**OkHttp** — это `HTTP`-клиент, разработанный компанией `Square`. 
Он используется для выполнения `HTTP`-запросов и обработки ответов. `OkHttp` поддерживает:

- Синхронные и асинхронные запросы.
- Кэширование.
- Перехватчики (`Interceptors`).
- Поддержку `HTTP`/`2` и `WebSockets`.

---

## Основные компоненты OkHttp

**OkHttpClient** — это основной класс, который управляет `HTTP`-запросами и ответами. 
Он настраивает соединения, кэширование, перехватчики и другие параметры.

```kotlin
@Singleton
@Provides
fun provideOkHttpClient(authInterceptor: AuthInterceptor): OkHttpClient =
    OkHttpClient.Builder()
        .connectTimeout(120, TimeUnit.SECONDS)
        .writeTimeout(120, TimeUnit.SECONDS)
        .readTimeout(120, TimeUnit.SECONDS)
        .addInterceptor(authInterceptor)
        .let { clientOkHttp: OkHttpClient.Builder ->
            if (BuildConfig.DEBUG) {
                clientOkHttp.addInterceptor(
                    HttpLoggingInterceptor().apply {
                        level = HttpLoggingInterceptor.Level.BODY
                    }
                )
            } else {
                clientOkHttp
            }
        }
        .addInterceptor { chain: Interceptor.Chain ->
            chain.proceed(
                chain.request().newBuilder()
                    .build()
            )
        }
        .dns(DnsSelectorHelper())
        .build()
```

### Interceptors

**Interceptors** — это механизм для перехвата и модификации `HTTP`-запросов и ответов. Они могут использоваться для:

- Логирования.
- Добавления заголовков.
- Обработки ошибок.

---

## Request

**Request** — это объект, который описывает HTTP-запрос. Он содержит:

- URL.
- Метод (`GET`, `POST`, `PUT`, `DELETE` и т.д.).
- Заголовки.
- Тело запроса (для `POST` и `PUT`).

---

## Response

**Response** — это объект, который содержит ответ на `HTTP`-запрос. Он включает:

- Код статуса (`200`, `404`, `500` и т.д.).
- Заголовки.
- Тело ответа.

---

## Call

**Call** — это объект, который представляет собой выполнение `HTTP`-запроса. 
Он может быть выполнен синхронно или асинхронно.

___

## Callback

**Callback** — это интерфейс, который используется для обработки асинхронных запросов. Он содержит два метода:

- **onFailure** - Вызывается, если запрос завершился с ошибкой.
- **onResponse** - Вызывается, если запрос выполнен успешно.

---

- **OkHttp** — это мощный `HTTP`-клиент для `Android` и `Java`.
- **OkHttpClient** управляет запросами и ответами.
- **Request** и Response описывают запросы и ответы.
- **Callback** используется для асинхронной обработки запросов.
- **Interceptors** позволяют перехватывать и модифицировать запросы и ответы.
- **Кэширование** уменьшает количество сетевых запросов.
- **Обработка** **ошибок** помогает управлять сетевыми сбоями и ошибками сервера.

---

#### [README](README.md) [UP](#up)
