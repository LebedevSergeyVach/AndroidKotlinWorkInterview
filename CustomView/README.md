# Custom View
<a name="up"></a>

---

**Custom View** — это пользовательский компонент, который вы создаете сами, чтобы реализовать уникальное поведение или внешний вид, недоступные в стандартных виджетах `Android`.

### Когда использовать Custom View?

 - **Уникальный дизайн** - Когда стандартные виджеты не могут реализовать нужный вам дизайн.
 - **Сложная логика** - Когда требуется сложная логика отрисовки или обработки событий.
 - **Повторное использование** - Когда вы хотите создать компонент, который можно использовать в нескольких местах.

---

## Создание Custom View

### Шаги создания Custom View:

 - Создайте класс, который наследуется от `View` или другого стандартного виджета (например, `Button`, `TextView`).
 - Переопределите методы для отрисовки (`onDraw`) и обработки событий (`onTouchEvent`).
 - Добавьте атрибуты для настройки `Custom View` через `XML`.
 - Используйте `Custom View` в `XML`-разметке или программно.

### Создание класса Custom View

```kotlin
import android.content.Context
import android.graphics.Canvas
import android.graphics.Color
import android.graphics.Paint
import android.util.AttributeSet
import android.view.View

class CircleView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {

    private val paint = Paint().apply {
        color = Color.RED
        isAntiAlias = true
        style = Paint.Style.FILL
    }

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        val radius = (width.coerceAtMost(height) / 2).toFloat()
        canvas.drawCircle(width / 2f, height / 2f, radius, paint)
    }
}
```

### Использование Custom View в XML

```xml
<com.example.myapp.CircleView
    android:layout_width="100dp"
    android:layout_height="100dp"
    android:layout_margin="16dp" />
```

---

## Добавление атрибутов

### Шаги:

 - Создайте файл атрибутов в `res`/`values`/`attrs.xml`.
 - Определите атрибуты для `Custom View`.
 - Используйте атрибуты в `XML` и обрабатывайте их в коде.

### Создание файла `attrs.xml`

```xml
<declare-styleable name="CircleView">
    <attr name="circleColor" format="color" />
</declare-styleable>
```

### Обработка атрибутов в Custom View

```kotlin
class CircleView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {

    private val paint = Paint().apply {
        isAntiAlias = true
        style = Paint.Style.FILL
    }

    init {
        val typedArray = context.obtainStyledAttributes(attrs, R.styleable.CircleView)
        val circleColor = typedArray.getColor(R.styleable.CircleView_circleColor, Color.RED)
        typedArray.recycle()

        paint.color = circleColor
    }

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        val radius = (width.coerceAtMost(height) / 2).toFloat()
        canvas.drawCircle(width / 2f, height / 2f, radius, paint)
    }
}
```

### Использование атрибута в XML

```xml
<com.example.myapp.CircleView
    android:layout_width="100dp"
    android:layout_height="100dp"
    android:layout_margin="16dp"
    app:circleColor="@color/blue" />
```

---

## Обработка событий

Можно переопределить метод `onTouchEvent`, чтобы обрабатывать касания.

### Пример: Изменение цвета при касании

```kotlin
override fun onTouchEvent(event: MotionEvent): Boolean {
    when (event.action) {
        MotionEvent.ACTION_DOWN -> {
            paint.color = Color.GREEN
            invalidate() // Перерисовать View
            return true
        }
        MotionEvent.ACTION_UP -> {
            paint.color = Color.RED
            invalidate() // Перерисовать View
            return true
        }
    }
    return super.onTouchEvent(event)
}
```

---

## Оптимизация Custom View

1. **Используйте invalidate() и postInvalidate()**

 - `invalidate()` — Перерисовывает `View` в главном потоке.
 - `postInvalidate()` — Перерисовывает `View` в фоновом потоке.

2. **Минимизируйте вызовы onDraw**

 - Избегайте сложных вычислений в onDraw.
 - Используйте кэширование для часто используемых значений.

3. **Используйте Canvas и Paint эффективно**

 - Создавайте объекты Paint один раз и переиспользуйте их.
 - Используйте методы Canvas для оптимизации отрисовки.

---

## Пример: ProgressBar с текстом

### Custom View:

```kotlin
class TextProgressBar @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {

    private val backgroundPaint = Paint().apply {
        color = Color.LTGRAY
        style = Paint.Style.FILL
    }

    private val progressPaint = Paint().apply {
        color = Color.BLUE
        style = Paint.Style.FILL
    }

    private val textPaint = Paint().apply {
        color = Color.WHITE
        textSize = 40f
        textAlign = Paint.Align.CENTER
    }

    var progress: Int = 50
        set(value) {
            field = value.coerceIn(0, 100)
            invalidate()
        }

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)

        // Отрисовка фона
        canvas.drawRect(0f, 0f, width.toFloat(), height.toFloat(), backgroundPaint)

        // Отрисовка прогресса
        val progressWidth = (width * progress / 100f)
        canvas.drawRect(0f, 0f, progressWidth, height.toFloat(), progressPaint)

        // Отрисовка текста
        val text = "$progress%"
        val x = width / 2f
        val y = height / 2f - (textPaint.descent() + textPaint.ascent()) / 2f
        canvas.drawText(text, x, y, textPaint)
    }
}
```

### Использование в XML:

```xml
<com.example.myapp.TextProgressBar
    android:layout_width="200dp"
    android:layout_height="30dp"
    android:layout_margin="16dp" />
```

---

## Кратко

**Custom View** — Пользовательский компонент для уникального дизайна и поведения.

### Основные шаги:

 - Создайте класс, наследуемый от View.
 - Переопределите onDraw для отрисовки.
 - Добавьте атрибуты через attrs.xml.
 - Обрабатывайте события через onTouchEvent.

### Оптимизация:

 - Используйте `invalidate()` и `postInvalidate()`.
 - Минимизируйте вызовы `onDraw`.

---

#### [README](README.md) [UP](#up)
