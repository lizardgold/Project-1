### Спецификация в формате OpenAPI 

#### Валидационные правила

##### Автоматические проверки:

- openapi версия 3.0.0 или выше
- info содержит title, version, description
- servers указаны с URL и description
- все paths имеют операции (get, post, put, delete)
- responses содержат минимум 200 и 400/500 коды
- schemas используют $ref для переиспользования
- parameters имеют description и schema
- requestBody содержит content с schema

#### Обязательные разделы

##### info - информация о проекте
```yaml
info:
  title: User Management API
  description: |
    REST API для управления пользователями системы.
    Поддерживает полный CRUD для пользователей и ролей.
  version: '1.0.0'
  contact:
    name: API Support
    email: support@example.com
  license:
    name: MIT
    url: https://opensource.org/licenses/MIT
```

##### servers - серверы API
```yaml
servers:
  - url: https://api.example.com/v1
    description: Production server
  - url: https://staging-api.example.com/v1
    description: Staging server
```

##### tags - группировка операций
```yaml
tags:
  - name: users
    description: Управление пользователями
  - name: auth
    description: Аутентификация и авторизация
```

#### Описание эндпоинтов

##### Структура операции:
```yaml
/users/{id}:
  get:
    tags: [users]
    summary: Получить пользователя по ID
    description: Возвращает детальную информацию о пользователе
    parameters:
      - name: id
        in: path
        required: true
        schema:
          type: integer
        description: Уникальный идентификатор пользователя
    responses:
      '200':
        description: Пользователь найден
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/User'
            example:
              id: 1
              email: "user@example.com"
              name: "John Doe"
      '404':
        description: Пользователь не найден
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Error'
```

##### Обязательные элементы операции:
- **tags**: группировка по функциональности
- **summary**: краткое описание (до 50 символов)
- **description**: подробное описание
- **parameters**: описание всех параметров
- **responses**: минимум 200 и error codes
- **examples**: примеры запросов и ответов

#### Компоненты и схемы

##### Переиспользуемые схемы:
```yaml
components:
  schemas:
    User:
      type: object
      required: [id, email]
      properties:
        id:
          type: integer
          description: Уникальный идентификатор
          example: 1
        email:
          type: string
          format: email
          description: Email пользователя
          example: "user@example.com"
        name:
          type: string
          description: Полное имя пользователя
          example: "John Doe"
        created_at:
          type: string
          format: date-time
          description: Дата создания
          example: "2024-01-15T10:30:00Z"
    
    UserCreateRequest:
      type: object
      required: [email, password]
      properties:
        email:
          type: string
          format: email
        password:
          type: string
          minLength: 8
        name:
          type: string
    
    Error:
      type: object
      required: [code, message]
      properties:
        code:
          type: integer
          description: Код ошибки
        message:
          type: string
          description: Описание ошибки
  
  parameters:
    PageParam:
      name: page
      in: query
      schema:
        type: integer
        minimum: 1
      description: Номер страницы для пагинации
  
  responses:
    NotFound:
      description: Ресурс не найден
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
```

#### Лучшие практики

##### Коды статусов
| Операция | Успех | Ошибка клиента | Ошибка сервера |
|----------|--------|----------------|----------------|
| **GET** | 200 | 404, 400 | 500 |
| **POST** | 201 | 400, 409 | 500 |
| **PUT** | 200 | 400, 404 | 500 |
| **DELETE** | 204 | 404 | 500 |
##### Валидация данных
```yaml
properties:
  email:
    type: string
    format: email
    maxLength: 255
  age:
    type: integer
    minimum: 0
    maximum: 150
  status:
    type: string
    enum: [active, inactive, pending]
```

##### Примеры и документация
- Добавляй `example` для каждого поля
- Используй `description` для всех элементов
- Включай реалистичные примеры запросов/ответов
- Документируй бизнес-логику в `description`

---
#### Полный пример API

```yaml
openapi: 3.0.3
info:
  title: User Management API
  description: REST API для управления пользователями и доставки webhook
  version: '1.0.0'
  contact:
    name: API Support
    email: support@example.com

servers:
  - url: https://api.example.com/v1
    description: Production server

tags:
  - name: users
    description: Операции с пользователями
  - name: webhooks
    description: Доставка webhook и обработка событий

paths:
  /users:
    get:
      tags: [users]
      summary: Получить список пользователей
      parameters:
        - $ref: '#/components/parameters/PageParam'
        - name: limit
          in: query
          description: Количество пользователей на странице
          schema:
            type: integer
            minimum: 1
            maximum: 100
            default: 20
      responses:
        '200':
          description: Список пользователей
          content:
            application/json:
              schema:
                type: object
                properties:
                  users:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  total:
                    type: integer

    post:
      tags: [users]
      summary: Создать пользователя
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UserCreateRequest'
      responses:
        '201':
          description: Пользователь создан
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          $ref: '#/components/responses/BadRequest'

  /users/{id}:
    get:
      tags: [users]
      summary: Получить пользователя
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: Пользователь найден
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          $ref: '#/components/responses/NotFound'

  /webhooks/deliveries/{deliveryId}:
    post:
      tags: [webhooks]
      summary: Доставить webhook
      description: |
        Доставка события получателю webhook.
        Запрос должен содержать timestamp и уникальный ключ события.
        При временных ошибках выполняются повторные попытки согласно
        политике доставки. Повторная обработка события не должна
        приводить к дублированию бизнес-операции.
      parameters:
        - name: deliveryId
          in: path
          required: true
          description: Идентификатор доставки
          schema:
            type: string
            format: uuid
        - name: X-Webhook-Timestamp
          in: header
          required: true
          description: Unix timestamp создания подписанного запроса
          schema:
            type: integer
            format: int64
        - name: Idempotency-Key
          in: header
          required: true
          description: Уникальный ключ для защиты от повторной обработки события
          schema:
            type: string
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [eventId, eventType, occurredAt, data]
              properties:
                eventId:
                  type: string
                  format: uuid
                eventType:
                  type: string
                occurredAt:
                  type: string
                  format: date-time
                data:
                  type: object
                  additionalProperties: true
      responses:
        '200':
          description: Событие успешно принято получателем
        '400':
          description: Некорректный запрос. Повторная доставка не выполняется
        '401':
          description: Ошибка аутентификации или подписи. Повторная доставка не выполняется
        '403':
          description: Недостаточно прав. Повторная доставка не выполняется
        '409':
          description: Конфликт идемпотентности или событие уже обработано
        '429':
          description: Превышен лимит запросов. Повторить с учётом Retry-After
        '500':
          description: Внутренняя ошибка получателя. Повторить доставку
        '502':
          description: Ошибка шлюза. Повторить доставку
        '503':
          description: Получатель временно недоступен. Повторить доставку
        '504':
          description: Истёк таймаут ожидания ответа. Повторить доставку
      x-delivery-policy:
        timestamp:
          allowedClockSkewSeconds: 300
          rejectOlderThanSeconds: 300
          rejectFutureTimestamp: true
        retries:
          maxAttempts: 8
          strategy: exponential-backoff
          initialDelaySeconds: 5
          maxDelaySeconds: 3600
          retryOnStatusCodes: [429, 500, 502, 503, 504]
          doNotRetryOnStatusCodes: [400, 401, 403, 404, 409, 422]
          respectRetryAfter: true
        deliveryDeadlineSeconds: 86400
        exhaustedRetries:
          action: mark_as_failed
          requireManualRedelivery: true

components:
  schemas:
    User:
      type: object
      required: [id, email]
      properties:
        id:
          type: integer
          example: 1
        email:
          type: string
          format: email
          example: "user@example.com"
        name:
          type: string
          example: "John Doe"

    UserCreateRequest:
      type: object
      required: [email, password]
      properties:
        email:
          type: string
          format: email
        password:
          type: string
          minLength: 8
        name:
          type: string

  parameters:
    PageParam:
      name: page
      in: query
      required: false
      description: Номер страницы
      schema:
        type: integer
        minimum: 1
        default: 1

  responses:
    BadRequest:
      description: Некорректный запрос
      content:
        application/json:
          schema:
            type: object
            properties:
              message:
                type: string

    NotFound:
      description: Ресурс не найден
      content:
        application/json:
          schema:
            type: object
            properties:
              message:
                type: string
```

---
#### Чек-лист качества

##### Структурная проверка:
- openapi версия 3.0.0+
- info содержит title, version, description
- servers указаны с description
- tags определены для группировки

##### Проверка эндпоинтов:
- Все CRUD операции описаны
- Эндпоинты имеют версионирование
- Каждая операция имеет summary и description
- parameters содержат schema и description
- responses покрывают success и error cases

##### Проверка схем:
- Результат проверь по правилам описанным в `linter.md`

##### Качественная проверка:
- Покрыты все бизнес-операции
- Валидация соответствует бизнес-правилам
- Коды ошибок логически обоснованы
- Документация понятна разработчикам

**Цель**: Создавать OpenAPI спецификации, готовые для генерации клиентского кода и документации.

---
#### Рекомендации по валидации

##### Инструменты проверки:
- [Swagger Editor](https://editor.swagger.io/) - онлайн валидатор
- `knowledge/linter.md` - линтер для OpenAPI
##### Примеры качественной документации:
- "Возвращает список пользователей с пагинацией"
- "Создает нового пользователя с валидацией email"
- "Ошибка 409 при дублировании email"
##### Примеры плохой документации:
- "Получает данные"
- "Создает объект"
- "Возвращает ошибку"