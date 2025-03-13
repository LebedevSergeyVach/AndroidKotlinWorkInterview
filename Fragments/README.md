# Fragments
<a name="up"></a>

---

## Fragments Androidx

**Fragment** — это компонент, который представляет собой часть пользовательского интерфейса (`UI`) в `Activity`. Fragment:

- **Модульность**: Разделение `UI` на независимые части.
- **Адаптивность**: Упрощение создания интерфейсов для разных устройств.
- **Повторное использование**: Один `Fragment` можно использовать в нескольких `Activity`.

## Основные особенности Fragments Androidx

- **Жизненный цикл** - `Fragment` имеет собственный жизненный цикл, который синхронизируется с жизненным циклом `Activity`.
- **FragmentManager** - Управляет `Fragments` в `Activity`.
- **FragmentTransaction** - Позволяет добавлять, удалять и заменять `Fragments`.
- **Back Stack** - Поддержка стека обратного перехода.

---

## Navigation Component

**Navigation Component** — это библиотека, которая упрощает навигацию между `Fragments` и `Activity`. Она предоставляет:

- **Граф навигации** - Визуальное представление переходов между экранами.
- **FragmentManager** - Управление `Fragments`.
- **NavController** - Управляет навигацией.
- **NavOptions** - Позволяет настроить анимации и поведение при навигации.
- **Аргументы** - Передача данных между `Fragments`.

---

## FragmentManager

**FragmentManager** — это класс, который управляет `Fragments` в `Activity`. Он отвечает за:

- Добавление, удаление и замену `Fragments`.
- Управление стеком обратного перехода (`back stack`).

### Основные методы FragmentManager

- **beginTransaction()** - Начинает транзакцию для работы с Fragments.
- **add()** - Добавляет Fragment в контейнер.
- **replace()** - Заменяет текущий Fragment на новый.
- **remove()** - Удаляет Fragment.
- **commit()** - Применяет транзакцию.

---

## Fragment Container

**Fragment Container** — это `View` (например, `FrameLayout` или `FragmentContainerView`), который используется для отображения `Fragments`.


### Пример Fragment Container в XML:

```xml
<FrameLayout
    android:id="@+id/fragment_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

---

## Жизненный цикл / Lifecycle Fragment

**Жизненный цикл Fragment** — это набор состояний, через которые проходит `Fragment` от создания до уничтожения. 
Он похож на жизненный цикл `Activity`, но имеет дополнительные методы.

###  Основные методы жизненного цикла Fragment

- **onAttach()** - Вызывается, когда `Fragment` связывается с `Activity`.
- **onCreate()** - Вызывается при создании `Fragment`.
- **onCreateView()** - Вызывается для создания `UI` `Fragment`.
- **onViewCreated()** - Вызывается после создания `UI`.
- **onStart()** - Вызывается, когда `Fragment` становится видимым.
- **onResume()** - Вызывается, когда `Fragment` становится активным.
- **onPause()** - Вызывается, когда `Fragment` теряет фокус.
- **onStop()** - Вызывается, когда `Fragment` больше не виден.
- **onDestroyView()** - Вызывается перед уничтожением `UI`.
- **onDestroy()** - Вызывается перед уничтожением `Fragment`.
- **onDetach()** - Вызывается, когда `Fragment` отвязывается от `Activity`.

---


## Arguments

**Arguments** — это способ передачи данных между `Fragments`. Данные передаются в виде `Bundle`.

Принимающий:

```kotlin
companion object {
    const val POST_ID = "POST_ID"
    const val POST_CONTENT = "POST_CONTENT"
    const val IS_UPDATE = "IS_UPDATE"
    const val POST_CREATED_OR_UPDATED_KEY = "POST_CREATED_OR_UPDATED_KEY"
}
```

```kotlin
val postId = arguments?.getLong(POST_ID) ?: 0L
val content = arguments?.getString(POST_CONTENT) ?: ""
val isUpdate = arguments?.getBoolean(IS_UPDATE, false) ?: false
```

Передающий:

```kotlin
override fun onUpdateClicked(post: PostUiModel) {
    requireParentFragment().requireParentFragment().findNavController()
        .navigate(
            R.id.action_BottomNavigationFragment_to_newOrUpdatePostFragment,
            bundleOf(
                NewOrUpdatePostFragment.POST_ID to post.id,
                NewOrUpdatePostFragment.POST_CONTENT to post.content,
                NewOrUpdatePostFragment.IS_UPDATE to true,
            ),
            NavOptions.Builder()
                .setEnterAnim(R.anim.slide_in_right)
                .setExitAnim(R.anim.slide_out_left)
                .setPopEnterAnim(R.anim.slide_in_left)
                .setPopExitAnim(R.anim.slide_out_right)
                .build()
        )
}
```

---

## Main Navigation - `main_navigation.xml` (`root_navigation.xml`)

**main_navigation.xml** — это файл, который описывает граф навигации в приложении. Он используется в `Navigation Component`.

- **app:startDestination** - Указывает стартовый `Fragment`.
- **<fragment>** - Описывает `Fragment` в графе навигации.
- **<action>** - Описывает переход между `Fragments`.

---

## Навигация с помощью `requireParentFragment().requireParentFragment().findNavController()`

Этот код используется для навигации между `Fragments`, когда текущий `Fragment` вложен в другой `Fragment` (например, в `BottomNavigationFragment`).

- **requireParentFragment()** - Возвращает родительский `Fragment` текущего `Fragment`.
- **requireParentFragment().requireParentFragment()** - Возвращает родительский `Fragment` родительского `Fragment` (например, `BottomNavigationFragment`).
- **findNavController()** - Возвращает `NavController`, связанный с родительским `Fragment`.
- **navigate()** - Выполняет навигацию с указанными параметрами.

---

## Bundle и bundleOf

**Bundle** — это контейнер для хранения данных в виде пар `ключ-значение`. 
Он используется для передачи данных между компонентами `Android`, такими как `Activity`, `Fragment` и `Service`.

**bundleOf** — это функция из библиотеки `Android KTX`, которая упрощает создание `Bundle`.
Вместо того чтобы вручную создавать `Bundle` и добавлять в него данные, можно использовать `bundleOf` для более компактного и читаемого кода.

- `bundleOf` создает `Bundle` и добавляет в него данные.
- Ключи и значения могут быть разных типов (`String`, `Int`, `Boolean` и т.д.).

---

#### [README](README.md) [UP](#up)
