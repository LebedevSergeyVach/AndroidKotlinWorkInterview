# ViewGroup, View, Layouts
<a name="up"></a>

---

## Базовая иерархия

```plantuml
View (базовый класс)
├── ViewGroup (контейнер)
│   ├── LinearLayout
│   ├── RelativeLayout
│   ├── ConstraintLayout
│   └── ...
└── TextView, Button, ImageView (виджеты)
```

---

## Основные методы View

### Жизненный цикл View

**1. Конструктор**

Вызывается при создании `View` из кода или инфлейта `XML`.

```java
public View(Context context) { ... }
public View(Context context, AttributeSet attrs) { ... } // Для XML
```

**2. onAttachedToWindow()**

`View` добавлена в иерархию окна (можно запускать анимации).

**3. onMeasure(widthMeasureSpec, heightMeasureSpec)**

Определяет размеры `View`.

```java
override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
    val width = MeasureSpec.getSize(widthMeasureSpec)
    val height = (width * 0.75).toInt() // Пример: 4:3 аспект
    setMeasuredDimension(width, height)
}
```

**4. onLayout(changed: Boolean, left: Int, top: Int, right: Int, bottom: Int)**

Позиционирует дочерние элементы (для `ViewGroup`).

**5. onDraw(canvas: Canvas)**

Отрисовка контента.


```kotlin
override fun onDraw(canvas: Canvas) {
    super.onDraw(canvas)
    canvas.drawCircle(width / 2f, height / 2f, radius, paint)
}
```

**6. onDetachedFromWindow()**
        
`View` удалена из иерархии (очищайте ресурсы!).

### Работа с фокусом и событиями

 - `isFocusable() / setFocusable()`
 - `onTouchEvent(event: MotionEvent): Boolean`
 - `dispatchTouchEvent() (для ViewGroup)`

**Пример обработки касания:**

```kotlin
override fun onTouchEvent(event: MotionEvent): Boolean {
    return when (event.action) {
        MotionEvent.ACTION_DOWN -> true
        MotionEvent.ACTION_MOVE -> { /* Движение пальца */ true }
        else -> super.onTouchEvent(event)
    }
}
```

### Изменение состояния

 - **setVisibility()**

```kotlin
view.visibility = View.VISIBLE   // Видима
view.visibility = View.INVISIBLE // Невидима, но занимает место
view.visibility = View.GONE      // Скрыта и не занимает место
```

 - **setEnabled() —** включает/выключает взаимодействие.
 - **invalidate() —** перерисовывает View (onDraw).
 - **requestLayout() —** пересчёт layout (onMeasure → onLayout).

---

## ViewGroup: Управление дочерними View

### Ключевые методы

 - **addView(child: View)** / **removeView(child: View)**
 - **getChildAt(index: Int)**
 - **measureChildWithMargins()** — измерение дочернего `View` с учётом `margin`.
 - **onLayout(changed: Boolean, l: Int, t: Int, r: Int, b: Int)**

```kotlin
override fun onLayout(changed: Boolean, l: Int, t: Int, r: Int, b: Int) {
    val child = getChildAt(0)
    child.layout(0, 0, child.measuredWidth, child.measuredHeight)
}
```

### Custom ViewGroup пример

Создаём вертикальный `layout` с фиксированным отступом:

```kotlin
class VerticalLayout(context: Context, attrs: AttributeSet?) : ViewGroup(context, attrs) {
    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        var totalHeight = 0
        for (i in 0 until childCount) {
            val child = getChildAt(i)
            measureChild(child, widthMeasureSpec, heightMeasureSpec)
            totalHeight += child.measuredHeight + 20 // +20px отступ
        }
        setMeasuredDimension(widthMeasureSpec, totalHeight)
    }

    override fun onLayout(changed: Boolean, l: Int, t: Int, r: Int, b: Int) {
        var top = 0
        for (i in 0 until childCount) {
            val child = getChildAt(i)
            child.layout(0, top, child.measuredWidth, top + child.measuredHeight)
            top += child.measuredHeight + 20
        }
    }
}
```

---

## XML-разметка: ключевые атрибуты

### Общие атрибуты

```xml
<View
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="@color/red"
    android:padding="16dp"
    android:layout_margin="8dp"
    android:visibility="visible" />
```

### Особенности ViewGroup

```xml
<LinearLayout
    android:orientation="vertical"
    android:gravity="center"
    android:divider="@drawable/divider"
    android:showDividers="middle">

    <TextView android:text="Item 1" />
    <TextView android:text="Item 2" />
</LinearLayout>
```

---

## Полезные методы View

| Метод                        | Описание                                   |
|------------------------------|--------------------------------------------|
| `getX()` / `getY()`          | Координаты относительно родителя.          |
| `getWidth()` / `getHeight()` | Размеры после layout.                      |
| `setTag()` / `getTag()`      | Хранение произвольных данных.              |
| `post(Runnable)`             | Выполнение кода после прикрепления к окну. |
| `animate()`                  | Простая анимация свойства.                 |

**Пример анимации:**

```kotlin
view.animate()
    .alpha(0.5f)
    .translationX(100f)
    .setDuration(300)
    .start()
```

---

## Итог

 - **View —** базовый элемент `UI` с методами `onMeasure`, `onDraw`, `onTouchEvent`.
 - **ViewGroup —** контейнер, управляющий `layout` дочерних элементов.
 - **XML —** декларативное описание иерархии (избегайте глубокой вложенности).
 - **Оптимизация —** `ConstraintLayout`, `RecyclerView`, `include`.

**Для кастомных View:**

 - Переопределяйте `onMeasure` и `onDraw`.
 - Используйте `invalidate()` для перерисовки.
 - Очищайте ресурсы в `onDetachedFromWindow()`.

---

#### [README](README.md) [UP](#up)
