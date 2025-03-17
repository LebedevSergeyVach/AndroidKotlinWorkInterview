#  Photo, Glide, Coil
<a name="up"></a>

---

## URI
**URI** — это строка, которая идентифицирует ресурс (например, файл, изображение, веб-страницу).
В Android `URI` часто используется для работы с файлами, контентом и внешними ресурсами.

В результате запроса фото у системы мы получим объект типа `Uri`.
Он представляет собой ссылку на ресурс.

 - **file://** – ссылка на файл
 - **content://** – ссылка на контент другого приложения
 - **http://** – ссылка на удалённый ресурс

Пример `URI`: `content://com.example.provider/images/1`

### Типы URI:

- **URL (Uniform Resource Locator)** - Указывает на местоположение ресурса в сети (например, https://example.com/image.jpg).
- **URN (Uniform Resource Name)** - Уникальное имя ресурса (например, `urn:isbn:0451450523`).

В `Android` `URI` часто используется для доступа к файлам через `ContentProvider` или для передачи данных между приложениями.

---

## Intent

В `Android` доступно 2 варианта по выбору фото:

 - Реализовать работу с камерой внутри приложения
 - Попросить у системы файлы нужного формата

Работа с камерой внутри приложения потребует от нас запроса
разрешений на работу с камерой, а также написания кода.

Если камера не является ключевой функцией приложения, проще
попросить результат у системы.

---

## Контракты


### Контракт для запуска активности съемки фотографии

**TakePicture** — это контракт, который позволяет сделать фотографию и сохранить ее в файл.

В `Android` рекомендуется использовать `ActivityResultContracts` для запуска активностей и получения результатов. 
Это более современный и безопасный подход, чем `startActivityForResult`.

```kotlin
/**
 * URI для временного хранения фотографии, которая будет прикреплена к посту.
 *
 * @see createPhotoUri
 */
val photoUri: Uri = imageHelper.createPhotoUri()

/**
 * Контракт для запуска активности съемки фотографии.
 *
 * После успешного завершения съемки фотографии, URI изображения сохраняется в ViewModel.
 *
 * @see ActivityResultContracts.TakePicture
 */
val takePictureContract: ActivityResultLauncher<Uri> =
    registerForActivityResult(ActivityResultContracts.TakePicture()) { success: Boolean ->
        if (success) {
            if (isCompressionEnabled) {
                val compressedFile = imageHelper.compressImage(photoUri)
                compressedFile?.let { file: File ->
                    if (file.exists()) {
                        newPostViewModel.saveAttachmentFileType(
                            FileModel(
                                uri = Uri.fromFile(file),
                                type = AttachmentTypeFile.IMAGE
                            )
                        )
                    } else {
                        requireContext().singleVibrationWithSystemCheck(35L)

                        requireContext().showMaterialDialogWithTwoButtons(
                            title = getString(R.string.image_compression_error),
                            message = getString(R.string.image_compression_error_description),
                            cancelButtonText = getString(R.string.unplug),
                            deleteButtonText = getString(R.string.thanks),
                            onDeleteConfirmed = {
                                requireContext().singleVibrationWithSystemCheck(35L)

                                isCompressionEnabled = false
                                newPostViewModel.saveAttachmentFileType(null)
                            },
                        )
                    }
                }
            } else {
                newPostViewModel.saveAttachmentFileType(
                    FileModel(
                        uri = photoUri,
                        type = AttachmentTypeFile.IMAGE
                    )
                )
            }
        }
    }
```

```kotlin
/**
 * Создает URI для сохранения изображения во временном каталоге приложения.
 * Если каталог не существует, он будет создан.
 *
 * @return URI созданного файла изображения.
 * @see FileProvider.getUriForFile
 * @sample createPhotoUri()
 */
fun createPhotoUri(): Uri {
    val directory: File = context.cacheDir.resolve("file_picker").apply {
        mkdir()
    }

    val file: File = directory.resolve("image.jpg")

    if (BuildConfig.DEBUG) {
        LoggerHelper.d("Путь к файлу: ${file.absolutePath}")
        LoggerHelper.d("Файл существует: ${file.exists()}")
        LoggerHelper.d("Файл доступен для чтения: ${file.canRead()}")
    }

    return FileProvider.getUriForFile(
        context,
        "${BuildConfig.APPLICATION_ID}.fileprovider",
        file
    )
}
```

### Контракт для запуска активности выбора фотографии из галереи

Для выбора фотографии из галереи можно использовать контракт `ActivityResultContracts.GetContent`.

```kotlin
/**
 * Контракт для запуска активности съемки фотографии.
 *
 * После успешного завершения съемки фотографии, URI изображения сохраняется в ViewModel.
 *
 * @see ActivityResultContracts.TakePicture
 */
val takePictureContract: ActivityResultLauncher<Uri> =
    registerForActivityResult(ActivityResultContracts.TakePicture()) { success: Boolean ->
        if (success) {
            if (isCompressionEnabled) {
                val compressedFile = imageHelper.compressImage(photoUri)
                compressedFile?.let { file: File ->
                    if (file.exists()) {
                        newPostViewModel.saveAttachmentFileType(
                            FileModel(
                                uri = Uri.fromFile(file),
                                type = AttachmentTypeFile.IMAGE
                            )
                        )
                    } else {
                        requireContext().singleVibrationWithSystemCheck(35L)

                        requireContext().showMaterialDialogWithTwoButtons(
                            title = getString(R.string.image_compression_error),
                            message = getString(R.string.image_compression_error_description),
                            cancelButtonText = getString(R.string.unplug),
                            deleteButtonText = getString(R.string.thanks),
                            onDeleteConfirmed = {
                                requireContext().singleVibrationWithSystemCheck(35L)

                                isCompressionEnabled = false
                                newPostViewModel.saveAttachmentFileType(null)
                            },
                        )
                    }
                }
            } else {
                newPostViewModel.saveAttachmentFileType(
                    FileModel(
                        uri = photoUri,
                        type = AttachmentTypeFile.IMAGE
                    )
                )
            }
        }
    }
```

### GetContent

**GetContent** — это контракт, который позволяет пользователю выбрать файл или изображение из галереи или файлового менеджера.
Он возвращает `URI` выбранного файла.

В момент вызова нужно передать тип файлов, например, любые изображения:

---

## FileModel

**FileModel** — это модель данных, которая представляет файл. Обычно содержит `URI` файла и его тип.

Для хранения данных в памяти перед отправкой на сервер создадим специальный класс, шде `AttachmentType` – допустимые типы вложений:

```kotlin
/**
 * Модель данных для представления файла.
 *
 * @property uri URI файла.
 * @property type Тип файла (например, изображение, документ и т.д.).
 */
data class FileModel(
    val uri: Uri,
    val type: AttachmentTypeFile,
)
```

```kotlin
/**
 * Перечисление, представляющее тип вложения (attachment) в медиа-сети.
 * Используется для указания типа медиа-файла (изображение, видео, аудио).
 *
 * @property IMAGE Тип вложения - изображение.
 * @property VIDEO Тип вложения - видео.
 * @property AUDIO Тип вложения - аудио.
 * @see Attachment
 */
@Serializable
enum class AttachmentTypeFile {
    @SerialName("IMAGE")
    IMAGE,

    @SerialName("VIDEO")
    VIDEO,

    @SerialName("AUDIO")
    AUDIO,
}
```

---

## ContentProvider, File Provider и Paths

**ContentProvider** — это компонент `Android`, который предоставляет доступ к данным приложения другим приложениям.
**FileProvider** — это подкласс `ContentProvider`, который используется для безопасного обмена файлами между приложениями.

```kotlin
<provider
    android:name="androidx.core.content.FileProvider"
    android:authorities="${applicationId}.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">

    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths" />
</provider>
```

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<paths>
    <cache-path
        name="file_picker"
        path="./file_picker" />
</paths>

```

Чтобы сделать фото в приложение, потребуется:

1. Указать ``FileProvider`` в `AndroidManifest.xml`
2. Создать файл в директории paths `FileProvider`
3. Преобразовать файл в Uri
4. Передать `Uri` системе через контракт `TakePicture`

### Paths

Пути бывают 3 видов:
 - **cache**
 - **internal**
 - **external**

В случае не долгого хранения фото и после отправки - удаления. Поэтому `cache` – лучший кандидат.

---

## Multipart и Retrofit

Для загрузки файлов на сервер используется **Multipart** запрос. 
В `Retrofit` это можно сделать с помощью аннотации `@Multipart`.

Дословно **Multi Part** – много частей. Такой формат позволяет передавать файлы в бинарном представлении.
Для разделения частей используется параметр `boundary` в заголовке.

В `Retrofit` есть специальная аннотация `@Multipart` для всего запроса и `@Part` для каждой части.

```kotlin
/**
 * Загружает медиа-файл на сервер.
 *
 * @param file Медиа-файл, который нужно загрузить, представленный как часть MultipartBody.
 * @return Возвращает объект [MediaDto], содержащий URL загруженного файла.
 * @see MediaDto
 *
 * @throws IOException Если произошла ошибка при загрузке файла.
 */
@Multipart
@POST("api/media")
suspend fun uploadMedia(@Part file: MultipartBody.Part): MediaDto
```

```kotlin
/**
 * Преобразует изображение в формате URI в JPG и загружает его на сервер с отслеживанием прогресса.
 *
 * Репозиторий для работы с медиафайлами через сеть.
 * Этот файл содержит функцию для загрузки медиафайлов на сервер с отслеживанием прогресса.
 *
 * @param fileModel Модель файла, содержащая URI файла и другую необходимую информацию.
 * @param contentResolver [ContentResolver] для доступа к содержимому файла по URI.
 * @param onProgress Колбэк, вызываемый для обновления прогресса загрузки. Принимает целое число от 0 до 100, представляющее процент выполнения.
 * @param mediaApi API для загрузки медиафайлов на сервер.
 *
 * @return [MediaDto] Ответ сервера после успешной загрузки файла.
 *
 * @throws IOException Если файл не найден или произошла ошибка при чтении файла.
 * @throws Exception Если произошла ошибка при преобразовании изображения.
 *
 * @see MediaApi
 * @see MediaDto
 * @see FileModel
 */
suspend fun uploadMedia(
    fileModel: FileModel,
    contentResolver: ContentResolver,
    onProgress: (Int) -> Unit,
    mediaApi: MediaApi,
): MediaDto {
    val inputStream: InputStream = contentResolver.openInputStream(fileModel.uri)
        ?: throw IOException("File not found")

    val bitmap: Bitmap? = BitmapFactory.decodeStream(inputStream)
    withContext(Dispatchers.IO) {
        inputStream.close()
    }

    if (bitmap == null) {
        throw IOException("Failed to decode bitmap from input stream")
    }

    val byteArrayOutputStream = ByteArrayOutputStream()
    if (!bitmap.compress(CompressFormat.JPEG, 100, byteArrayOutputStream)) {
        throw IOException("Failed to compress bitmap to JPEG")
    }

    val fileBytes: ByteArray = byteArrayOutputStream.toByteArray()
    if (fileBytes.isEmpty()) {
        throw IOException("Failed to convert bitmap to byte array")
    }

    val fileBody: RequestBody = fileBytes.toRequestBody("image/jpeg".toMediaTypeOrNull())

    val requestBody = object : RequestBody() {
        override fun contentType(): MediaType? = fileBody.contentType()

        override fun writeTo(sink: BufferedSink) {
            val totalBytes: Long = fileBytes.size.toLong()
            var bytesWritten = 0L
            val buffer = ByteArray(DEFAULT_BUFFER_SIZE)

            val source: ByteArrayInputStream = fileBytes.inputStream()
            source.use { input: ByteArrayInputStream ->
                while (true) {
                    val bytesRead: Int = input.read(buffer)
                    if (bytesRead == -1) break

                    sink.write(buffer, 0, bytesRead)
                    bytesWritten += bytesRead

                    val progress: Int = ((bytesWritten.toFloat() / totalBytes) * 100).toInt()
                    onProgress(progress)
                }
            }
        }

        override fun contentLength(): Long = fileBytes.size.toLong()
    }

    val part = MultipartBody.Part.createFormData("file", "file.jpg", requestBody)

    return try {
        mediaApi.uploadMedia(part)
    } catch (e: IOException) {
        throw IOException("Failed to upload file: ${e.message}")
    } catch (e: Exception) {
        throw IOException("Unexpected error: ${e.message}")
    }
}
```

Для того, чтобы из `Uri` получить `MultipartBody.Part`, нам потребуется `ContentResolver`.

К сожалению, придётся прочитать все байты перед отправкой.
Поддержка стриминга для `Uri` в `OkHttp` отсутствует.

---

## Glide

**Glide** — это библиотека для загрузки и кэширования изображений.
Она проста в использовании и поддерживает асинхронную загрузку.

```kotlin
binding.skeletonAttachment.showSkeleton()

if (post.attachment != null) {
    renderingImageAttachment(attachment = post.attachment, radius = radius)
} else {
    binding.skeletonAttachment.showOriginal()
    binding.attachment.isVisible = false
}
```

```kotlin
/**
 * Отображает вложение (изображение) в элементе интерфейса, используя библиотеку Glide.
 * Если загрузка изображения завершилась ошибкой, отображается заглушка.
 * Изображение отображается с закругленными углами и эффектом перехода (cross-fade).
 *
 * @param attachment Экземпляр [Attachment], содержащий данные о вложении, включая URL изображения.
 * @param radius Радиус закругления углов изображения (в пикселях).
 *
 * @see Attachment Модель данных вложения, содержащая URL изображения.
 * @see Glide Библиотека для загрузки и отображения изображений.
 * @see RequestListener Интерфейс для обработки событий загрузки изображений.
 * @see RoundedCorners Трансформация для закругления углов изображения.
 * @see DrawableTransitionOptions Опции для анимации перехода при загрузке изображения.
 *
 * @property attachment.url URL изображения, которое необходимо загрузить.
 * @property binding.attachment ImageView для отображения вложения.
 * @property binding.skeletonAttachment Элемент интерфейса, используемый для отображения скелетона (заглушки) во время загрузки.
 */
private fun renderingImageAttachment(
    attachment: Attachment,
    radius: Int
) {
    Glide.with(binding.root)
        .load(attachment.url)
        .listener(object : RequestListener<Drawable> {
            override fun onLoadFailed(
                e: GlideException?,
                model: Any?,
                target: Target<Drawable>,
                isFirstResource: Boolean
            ): Boolean {
                binding.skeletonAttachment.showOriginal()
                binding.attachment.setImageResource(R.drawable.ic_404_24)

                return false
            }

            override fun onResourceReady(
                resource: Drawable,
                model: Any,
                target: Target<Drawable>?,
                dataSource: DataSource,
                isFirstResource: Boolean
            ): Boolean {
                binding.skeletonAttachment.showOriginal()

                return false
            }
        })
        .thumbnail(
            Glide.with(binding.root)
                .load(attachment.url)
                .override(50, 50)
                .diskCacheStrategy(DiskCacheStrategy.ALL)
        )
        .diskCacheStrategy(DiskCacheStrategy.ALL)
        .transform(RoundedCorners(radius))
        .transition(DrawableTransitionOptions.withCrossFade(500))
        .error(R.drawable.ic_404_24)
        .into(binding.attachment)
}
```

---

## Coil

**Coil** — это современная библиотека для загрузки изображений, написанная на `Kotlin`. 
Она легковесная и поддерживает корутины.

---

## ViewModel, State и Fragment

### API

```kotlin
/**
 * Сохраняет или обновляет пост.
 *
 * @param post Объект PostData, который нужно сохранить или обновить.
 * @return PostData Обновленный или сохраненный пост.
 */
@POST("api/posts")
suspend fun savePost(@Body post: PostData): PostData
```

### Repository

```kotlin
/**
 * Сохраняет или обновляет пост.
 *
 * @param postId Идентификатор поста.
 * @param content Новое содержание поста.
 *
 * @return PostData Обновленный или сохраненный пост.
 */
override suspend fun save(
    postId: Long,
    content: String,
    fileModel: FileModel?,
    contentResolver: ContentResolver,
    onProgress: (Int) -> Unit
): PostData {
    val post: PostData = fileModel?.let { file: FileModel ->
        val media: MediaDto =
            uploadMedia(
                fileModel = file,
                contentResolver = contentResolver,
                onProgress = onProgress,
                mediaApi = mediaApi,
            )

        PostData(
            id = postId,
            content = content,
            attachment = Attachment(url = media.url, type = file.type),
        )
    } ?: PostData(
        id = postId, content = content,
    )

    return postsApi.savePost(post = post)
}
```

### ViewModel

```kotlin
/**
 * Сохраняет или обновляет пост.
 *
 * Этот метод отвечает за сохранение нового поста или обновление существующего. Если идентификатор поста равен 0, создается новый пост. В противном случае обновляется существующий пост.
 *
 * @param content Содержимое поста. Текст, который будет сохранен или обновлен.
 * @param context Контекст приложения. Используется для работы с файлами и другими ресурсами.
 *
 * @see PostData Класс, представляющий данные поста.
 * @see StatusLoad Перечисление, представляющее состояние загрузки (Idle, Loading, Error).
 */
fun save(content: String, contentResolver: ContentResolver, onProgress: (Int) -> Unit) {
    _state.update { newPostState: NewPostState ->
        newPostState.copy(
            statusPost = StatusLoad.Loading
        )
    }

    viewModelScope.launch {
        try {
            val post: PostData = repository.save(
                postId = postId,
                content = content,
                fileModel = _state.value.file,
                contentResolver = contentResolver,
                onProgress = onProgress,
            )

            _state.update { newPostState: NewPostState ->
                newPostState.copy(
                    statusPost = StatusLoad.Idle,
                    post = post,
                )
            }
        } catch (e: Exception) {
            _state.update { newPostState: NewPostState ->
                newPostState.copy(
                    statusPost = StatusLoad.Error(exception = e)
                )
            }
        }
    }
}

/**
 * Сохраняет тип файла вложения.
 *
 * Этот метод обновляет состояние ViewModel, сохраняя модель файла, который будет прикреплен к посту.
 *
 * @param file Модель файла, который будет прикреплен к посту. Может быть null, если вложение отсутствует.
 *
 * @see FileModel Класс, представляющий модель файла для вложения.
 */
fun saveAttachmentFileType(file: FileModel?) {
    _state.update { stateNewPost: NewPostState ->
        stateNewPost.copy(
            file = file,
        )
    }
}
```

### Fragment

```kotlin
newPostViewModel.save(
    content = newContent,
    contentResolver = requireContext().contentResolver,
    onProgress = { progress ->
        binding.progressBar.setProgressCompat(progress, true)
    }
)
```

```kotlin
/**
 * Контракт для запуска активности выбора фотографии из галереи.
 *
 * После выбора фотографии, URI изображения сохраняется в ViewModel.
 *
 * @see ActivityResultContracts.PickVisualMedia
 */
val takePictureGalleryContract: ActivityResultLauncher<PickVisualMediaRequest> =
    registerForActivityResult(ActivityResultContracts.PickVisualMedia()) { uriOrNull: Uri? ->
        uriOrNull?.let { uri: Uri ->
            if (isCompressionEnabled) {
                val compressedFile = imageHelper.compressImage(uri)
                compressedFile?.let { file: File ->
                    if (file.exists()) {
                        newPostViewModel.saveAttachmentFileType(
                            FileModel(
                                uri = Uri.fromFile(file),
                                type = AttachmentTypeFile.IMAGE
                            )
                        )
                    } else {
                        requireContext().singleVibrationWithSystemCheck(35L)

                        requireContext().showMaterialDialogWithTwoButtons(
                            title = getString(R.string.image_compression_error),
                            message = getString(R.string.image_compression_error_description),
                            cancelButtonText = getString(R.string.unplug),
                            deleteButtonText = getString(R.string.thanks),
                            onDeleteConfirmed = {
                                requireContext().singleVibrationWithSystemCheck(35L)

                                isCompressionEnabled = false
                                newPostViewModel.saveAttachmentFileType(null)
                            },
                        )
                    }
                }
            } else {
                newPostViewModel.saveAttachmentFileType(
                    FileModel(
                        uri = uri,
                        type = AttachmentTypeFile.IMAGE
                    )
                )
            }
        }
    }
```

---

## Кратко

### 1. **URI**
- **URI** - Универсальный идентификатор ресурса, используемый для доступа к файлам, изображениям или другим данным.
- **Пример:** `content://com.example.provider/images/1`.

---

### 2. **Intent**
- **Intent** - Объект для передачи данных между компонентами приложения или запуска внешних приложений (например, камеры или галереи).
- **Типы:**
    - **Explicit Intent** - Запуск конкретного компонента.
    - **Implicit Intent** - Выполнение действия (например, открыть камеру).

---

### 3. **Контракт для съемки фотографии**
- **Контракт для съемки** - Современный способ запуска активности камеры через `ActivityResultContracts.TakePicture`.
- **Пример:**
  ```kotlin
  val takePicture = registerForActivityResult(ActivityResultContracts.TakePicture()) { success -> }
  ```

---

### 4. **Контракт для выбора фото из галереи**
- **Контракт для выбора** - Способ выбора фото из галереи через `ActivityResultContracts.GetContent`.
- **Пример:**
  ```kotlin
  val getContent = registerForActivityResult(ActivityResultContracts.GetContent()) { uri -> }
  ```

---

### 5. **GetContent**
- **GetContent** - Контракт для выбора файла или изображения из галереи или файлового менеджера.
- **Пример:**
  ```kotlin
  val getContent = registerForActivityResult(ActivityResultContracts.GetContent()) { uri -> }
  ```

---

### 6. **FileModel**
- **FileModel** - Модель данных для представления файла, содержащая URI и тип файла.
- **Пример:**
  ```kotlin
  data class FileModel(val uri: Uri, val type: AttachmentTypeFile)
  ```

---

### 7. **ContentProvider и FileProvider**
- **ContentProvider** - Компонент для предоставления доступа к данным приложения другим приложениям.
- **FileProvider** - Подкласс `ContentProvider` для безопасного обмена файлами.
- **Пример:**
  ```xml
  <provider android:name="androidx.core.content.FileProvider" ... />
  ```

---

### 8. **Создание файла и преобразование в URI**
- **Создание файла** - Процесс создания файла и получения его URI через `FileProvider`.
- **Пример:**
  ```kotlin
  val photoURI = FileProvider.getUriForFile(context, "${context.packageName}.fileprovider", file)
  ```

---

### 9. **TakePicture**
- **TakePicture** - Контракт для съемки фото и сохранения его в файл.
- **Пример:**
  ```kotlin
  val takePicture = registerForActivityResult(ActivityResultContracts.TakePicture()) { success -> }
  ```

---

### 10. **Multipart + Retrofit**
- **Multipart** - Способ загрузки файлов на сервер через `Multipart` запросы.
- **Retrofit** - Библиотека для работы с сетевыми запросами.
- **Пример:**
  ```kotlin
  @Multipart @POST("upload") suspend fun uploadImage(@Part file: MultipartBody.Part)
  ```

---

### 11. **Glide**
- **Glide** - Библиотека для загрузки и кэширования изображений.
- **Пример:**
  ```kotlin
  Glide.with(context).load(imageUrl).into(imageView)
  ```

---

### 12. **Coil**
- **Coil** - Легковесная библиотека для загрузки изображений, написанная на Kotlin.
- **Пример:**
  ```kotlin
  imageView.load(imageUrl)
  ```

---

#### [README](README.md) [UP](#up)
