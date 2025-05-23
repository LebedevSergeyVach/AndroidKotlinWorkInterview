# Clean Architecture
<a name="up"></a>

---

**Clean Architecture в Android** — это методология разработки программного обеспечения, которая позволяет создавать легко поддерживаемые и масштабируемые приложения.
Это рекомендации по организации системной архитектуры. Они были предложены **Робертом С. Мартином** (известным также как Дядя Боб).
Метод разработки программного обеспечения, при котором код разделён на несколько уровней. При этом внутренний уровень не должен зависеть от каких-либо внешних уровней. 
Это означает, что зависимости должны указываться внутри каждого уровня, чтобы не было зависимостей между уровнями (слоями).

---

## Основные принципы Clean Architecture:

 - **Единственная ответственность -** Каждый слой (модуль), класс или функция выполняет только одну задачу.
 - **Разделение на слои -** Приложение разбивается на уровни, у каждого слоя своя зона ответственности. Обычно выделяют следующие уровни: `presentation`, `domain`, `data`:
 - - **Presentation —** отвечает за отображение пользовательского интерфейса и реагирование на его события.
 - - **Domain —** бизнес-логика, изолированная от деталей реализации, определяет правила и операции, как приложение должно взаимодействовать с данными.
 - - **Data —** хранилище данных.

 - **Инверсия зависимостей -** Вместо прямой зависимости слоя верхнего уровня от компонентов слоя нижнего используется общий контракт, такой как интерфейс или абстрактный класс.

### Некоторые преимущества Clean Architecture:

 - код проще тестировать;
 - удобно структурированная структура пакета;
 - легко поддерживать проект в рабочем состоянии.

---

### Основная цель Clean Architecture — создание систем, которые:

 - Просты в понимании и модификации. Разработчики могут вносить изменения с минимальным риском появления ошибок.
 - Удобны в тестировании. Для различных частей системы можно создавать автоматизированные тесты.
 - Независимы от фреймворков. Фреймворки можно заменять с минимальным влиянием на систему.
 - Независимы от пользовательского интерфейса. Интерфейс пользователя может меняться без влияния на базовую логику или бизнес-правила.
 - Независимы от базы данных. Механизмы хранения и извлечения данных могут меняться без влияния на систему.
 - Независимы от какого-либо внешнего воздействия. Бизнес-правила не привязаны к какой-либо конкретной детали реализации, что делает систему адаптируемой к изменениям.
 - Ключевой принцип `Clean Architecture` — разделение приложения на уровни, каждый из которых выполняет свои задачи и управляет своей ответственностью.

### Основные принципы Clean Architecture:

 - Независимость от фреймворков. `Clean Architecture` избегает привязки к конкретным фреймворкам, что позволяет легко заменять или обновлять их без влияния на остальную часть приложения.
 - Разделение на слои. Приложение делится на четыре основных слоя: представление, бизнес-логика, хранилище данных и внешние источники данных. Каждый слой имеет свою специфическую функцию и ответственность.
 - Принцип единственной ответственности (`Single Responsibility Principle`). Каждый компонент в `Clean Architecture` имеет одну и только одну причину для изменения, что облегчает поддержку и модификацию приложения.
 - Зависимости внутрь. В `Clean Architecture` зависимости направлены внутрь, то есть более высокоуровневые слои не зависят от низкоуровневых слоёв, что позволяет изолировать каждый слой и тестировать их независимо.

### Приложение в чистой архитектуре обычно имеет три уровня:

 - Внешний (уровень реализации). Здесь описывается основная структура приложения. Сюда входит любое содержимое `Android`, такое как создание операций и фрагментов, отправка намерений, и другой структурный код наподобие сетевого кода и кода базы данных.
 - Средний (уровень интерфейса). Обеспечивает взаимодействие между уровнем реализации и уровнем бизнес-логики.
 - Внутренний (уровень бизнес-логики). Здесь решается поставленная задача, ради которой создавалось приложение.
 - Каждый уровень, расположенный выше основного уровня, отвечает за преобразование моделей в модели нижнего уровня, перед тем как нижний уровень сможет их использовать.

---

## Материалы

 - [**Clean Architecture - habr.com**](https://habr.com/ru/companies/otus/articles/732178/)

---

#### [README](README.md) [UP](#up)
