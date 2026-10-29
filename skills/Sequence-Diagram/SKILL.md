### Sequence диаграмма

#### Структура диаграммы

##### Заголовок и настройки
```plantuml
@startuml
autonumber "<b><color:DarkSlateBlue>.</color></b> " 
```

##### Участники (строгая типизация)
```plantuml
actor User as "Роль из User Story"
participant Browser as "Browser"
participant "Web Server" as WebServer
participant "Application Server" as AppServer
participant "Database Server" as DBServer
```

##### Группировка этапов
```plantuml
== Название логического этапа ==
```

##### Взаимодействия с протоколами
```plantuml
User -> Browser : Бизнес-действие
Browser -> WebServer : HTTP GET/POST /endpoint
WebServer -> AppServer : REST API: метод
AppServer -> DBServer : JDBC: SELECT/INSERT
```


#### Валидационные правила

##### Автоматические проверки:
```
- Начинается с @startuml, заканчивается @enduml
- Роль актора соответствует User Story
- Участники присутствуют в архитектурной диаграмме
- Каждый этап имеет название в формате "== Название =="
- Протоколы указаны для технических взаимодействий
- Синхронные/асинхронные стрелки используются корректно
- Есть минимум 1 альтернативный поток (alt/opt/loop)
```


#### Базовый шаблон

```plantuml
@startuml
autonumber "<b><color:DarkSlateBlue>.</color></b> " 

actor User as "[Роль из User Story]"
participant Browser as "Browser"
participant "Web Server" as WebServer
participant "Application Server" as AppServer
participant "Database Server" as DBServer

== Инициация действия ==
User -> Browser : [Бизнес-действие]
Browser -> WebServer : HTTP [метод] /[endpoint]

== Обработка запроса ==
WebServer -> AppServer : REST API: [метод]

== Работа с данными ==
AppServer -> DBServer : JDBC: [SQL операция]
DBServer --> AppServer : [Результат]

== Возврат результата ==
AppServer --> WebServer : JSON: [данные]
WebServer --> Browser : HTTP 200 OK
Browser --> User : [Отображение результата]

@enduml
```


#### Типы взаимодействий

##### Протоколы и синтаксис:
| Тип | Синтаксис | Пример |
|-----|-----------|--------|
| **HTTP** | `HTTP [метод] /endpoint` | `HTTP GET /api/users` |
| **REST API** | `REST API: [операция]` | `REST API: getUserData` |
| **База данных** | `JDBC: [SQL]` | `JDBC: SELECT * FROM users` |
| **Message Queue** | `MQ: [операция]` | `MQ: publish userCreated` |
| **gRPC** | `gRPC: [метод]` | `gRPC: GetUserProfile` |

##### Типы стрелок:
- `->` и `-->` - синхронные вызовы/ответы
- `->>` и `-->>` - асинхронные вызовы/ответы
- `->` на себя - внутренняя обработка

##### Управляющие конструкции:
```plantuml
alt Успешный сценарий
    // основной поток
else Ошибка
    // обработка ошибки
end

opt Условное выполнение
    // опциональные действия
end

loop Повторение
    // циклические действия
end
```

#### Интеграция с артефактами

##### Связь с User Story:
- **Актор диаграммы** = роль из US
- **Основной поток** = описание действий из US
- **Результат** = ожидаемая выгода из US

##### Связь с Use Case:
- **Основной сценарий UC** = главная последовательность
- **Альтернативные потоки UC** = alt/opt блоки в диаграмме
- **Исключения UC** = error handling блоки

##### Связь с архитектурой:
- **Участники sequence** = компоненты из архитектуры
- **Взаимодействия** = связи между компонентами
- **Протоколы** = технологии интеграции


#### Стандартные этапы и названия

##### Типовые группы:
1. **Инициация**: "Пользователь инициирует действие"
2. **Аутентификация**: "Проверка прав доступа"
3. **Валидация**: "Проверка входных данных"
4. **Обработка**: "Бизнес-логика и вычисления"
5. **Сохранение**: "Работа с базой данных"
6. **Уведомления**: "Отправка сообщений"
7. **Ответ**: "Возврат результата пользователю"

##### Примеры конкретных названий:
- "== Загрузка списка заказов == "
- "== Проверка корректности платежных данных == "
- "== Формирование отчета по продажам == "


#### Обработка ошибок

##### Обязательные сценарии ошибок:
```plantuml
alt Успешное выполнение
    AppServer -> DBServer : SELECT query
    DBServer --> AppServer : Data returned
else Ошибка подключения к БД
    AppServer -> DBServer : SELECT query
    DBServer --> AppServer : Error: Connection timeout
    AppServer --> WebServer : HTTP 500 Internal Error
    WebServer --> Browser : Страница ошибки
else Ошибка валидации данных
    AppServer -> AppServer : Validate input
    AppServer --> WebServer : HTTP 400 Bad Request
    WebServer --> Browser : Сообщение об ошибке
end
```

---
#### Чек-лист качества

##### Структурная проверка:
- Файл начинается с `@startuml` и заканчивается `@enduml`
- Используется autonumber для нумерации шагов
- Актор соответствует роли из User Story
- Все участники есть в архитектурной диаграмме

##### Логическая проверка:
- 3-7 логических этапов с понятными названиями
- Последовательность шагов соответствует Use Case
- Есть альтернативные потоки (alt/opt/loop)
- Обработка минимум 2 типов ошибок

##### Техническая проверка:
- Протоколы указаны для всех технических вызовов
- HTTP методы и endpoints конкретизированы
- Синхронные/асинхронные вызовы корректны
