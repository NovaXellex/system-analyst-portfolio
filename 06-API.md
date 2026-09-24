# 06. API

REST API системы подачи заявок на финансирование.

API предоставляет доступ к ресурсам системы и операциям над ними: 
* создание и редактирование заявок
* проведение экспертиз
* согласование
* работа с участниками и документами
* получение текущего состояния и истории процесса

Для защиты API используется аутентификация пользователя и авторизация операций с учётом его роли и этапа заявки.

Базовый URL:

```text
/api/v1
```
Пример:

```http
GET /api/v1/financial-requests/123
```
---

## 6.1. Определение API

API построено на следующих принципах:

* REST;
* JSON в качестве основного формата обмена;
* версионирование API через URL;
* разделение ресурсных операций и бизнес-операций;
* запрет прямого изменения `status` и `stage` через API;
* проверка прав пользователя перед выполнением операции;
* проверка допустимости операции относительно текущего статуса/этапа заявки.

---

## 6.2. Ресурсы API

Основные ресурсы API:

| Ресурс             | Назначение                                        |
| ------------------ | ------------------------------------------------- |
| `FinancialRequest` | Заявка на финансирование                          |
| `Project`          | Проект, для которого запрашивается финансирование |
| `Participant`      | Участник заявки и его роль                        |
| `Document`         | Документ, связанный с заявкой                     |
| `ExpertReview`     | Заключение эксперта                               |
| `FinancialReview`  | Заключение финансового специалиста                |
| `Process`          | Текущее состояние процесса                        |
| `ProcessHistory`   | История переходов процесса                        |

Например, `RequestProcessContext` и `RequestProcessHistory` используются для хранения состояния и истории процесса, клиент работает с ними через API-представления:

```http
GET /api/v1/financial-requests/{id}/process
GET /api/v1/financial-requests/{id}/history
```

---

## 6.3. Конечные точки и методы

### Заявки

| Метод   | Endpoint                   | Назначение              |
| ------- | -------------------------- | ----------------------- |
| `POST`  | `/financial-requests`      | Создание заявки         |
| `GET`   | `/financial-requests`      | Получение списка заявок |
| `GET`   | `/financial-requests/{id}` | Получение заявки        |
| `PATCH` | `/financial-requests/{id}` | Изменение заявки        |

### Бизнес-операции

| Метод  | Endpoint                                 | Назначение            |
| ------ | ---------------------------------------- | --------------------- |
| `POST` | `/financial-requests/{id}/submit`        | Подача заявки         |
| `POST` | `/financial-requests/{id}/approve`       | Согласование          |
| `POST` | `/financial-requests/{id}/return`        | Возврат на доработку  |
| `POST` | `/financial-requests/{id}/reject`        | Отклонение            |
| `POST` | `/financial-requests/{id}/final-approve` | Финальное утверждение |

### Экспертизы

| Метод  | Endpoint                                     | Назначение                        |
| ------ | -------------------------------------------- | --------------------------------- |
| `POST` | `/financial-requests/{id}/expert-reviews`    | Добавление заключения эксперта    |
| `GET`  | `/financial-requests/{id}/expert-reviews`    | Получение заключений эксперта     |
| `POST` | `/financial-requests/{id}/financial-reviews` | Добавление финансового заключения |
| `GET`  | `/financial-requests/{id}/financial-reviews` | Получение финансовых заключений   |

### Участники

| Метод    | Endpoint                                                | Назначение           |
| -------- | ------------------------------------------------------- | -------------------- |
| `GET`    | `/financial-requests/{id}/participants`                 | Получение участников |
| `POST`   | `/financial-requests/{id}/participants`                 | Добавление участника |
| `DELETE` | `/financial-requests/{id}/participants/{participantId}` | Удаление участника   |

### Документы

| Метод    | Endpoint                                          | Назначение           |
| -------- | ------------------------------------------------- | -------------------- |
| `GET`    | `/financial-requests/{id}/documents`              | Получение документов |
| `POST`   | `/financial-requests/{id}/documents`              | Добавление документа |
| `DELETE` | `/financial-requests/{id}/documents/{documentId}` | Удаление документа   |

### Процесс

| Метод | Endpoint                           | Назначение                            |
| ----- | ---------------------------------- | ------------------------------------- |
| `GET` | `/financial-requests/{id}/process` | Получение текущего состояния процесса |
| `GET` | `/financial-requests/{id}/history` | Получение истории процесса            |

---

## 6.4. Параметры

### Path-параметры

Идентификатор заявки передаётся в URL:

```http
GET /api/v1/financial-requests/{id}
```

| Параметр | Тип       | Описание             |
| -------- | --------- | -------------------- |
| `id`     | `integer` | Идентификатор заявки |

Для операций с участниками и документами используются дополнительные идентификаторы:

```http
DELETE /api/v1/financial-requests/{id}/participants/{participantId}
DELETE /api/v1/financial-requests/{id}/documents/{documentId}
```

### Query-параметры

Для получения списка заявок используются параметры фильтрации:

| Параметр      | Тип       | Описание               |
| ------------- | --------- | ---------------------- |
| `status`      | `string`  | Фильтр по статусу      |
| `stage`       | `string`  | Фильтр по этапу        |
| `authorId`    | `integer` | Фильтр по автору       |
| `projectId`   | `integer` | Фильтр по проекту      |
| `createdFrom` | `date`    | Начальная дата периода |
| `createdTo`   | `date`    | Конечная дата периода  |

Пример:

```http
GET /api/v1/financial-requests?stage=EXPERT_REVIEW&status=CONSIDERATION
```

### Body-параметры

Данные создания и изменения заявки передаются в теле HTTP-запроса в формате JSON.

Пример:

```json
{
  "title": "Финансирование проекта X",
  "purpose": "Финансирование реализации проекта X",
  "projectId": 42,
  "expectedStartDate": "2026-10-01",
  "expectedEndDate": "2027-09-30",
  "requestedSum": 15000000,
  "comment": "Дополнительная информация по заявке"
}
```

---

## 6.5. Примеры запросов

### Создание заявки

```http
POST /api/v1/financial-requests
Content-Type: application/json
Authorization: Bearer <access-token>
```

```json
{
  "title": "Финансирование проекта X",
  "purpose": "Финансирование реализации проекта X",
  "projectId": 42,
  "expectedStartDate": "2026-10-01",
  "expectedEndDate": "2027-09-30",
  "requestedSum": 15000000,
  "comment": "Дополнительная информация по заявке"
}
```

### Изменение заявки

```http
PATCH /api/v1/financial-requests/123
Content-Type: application/json
Authorization: Bearer <access-token>
```

```json
{
  "title": "Обновлённое название",
  "purpose": "Обновлённое назначение",
  "requestedSum": 17000000,
  "comment": "Дополнительный комментарий"
}
```

`status` и `stage` не передаются в запросе на изменение заявки.

### Подача заявки

```http
POST /api/v1/financial-requests/123/submit
Authorization: Bearer <access-token>
```

### Добавление экспертного заключения

```http
POST /api/v1/financial-requests/123/expert-reviews
Content-Type: application/json
Authorization: Bearer <access-token>
```

```json
{
  "result": "Проект соответствует установленным требованиям",
  "comment": "Дополнительные замечания эксперта"
}
```

### Согласование

```http
POST /api/v1/financial-requests/123/approve
Content-Type: application/json
Authorization: Bearer <access-token>
```

```json
{
  "comment": "Заявка согласована."
}
```

### Возврат на доработку

```http
POST /api/v1/financial-requests/123/return
Content-Type: application/json
Authorization: Bearer <access-token>
```

```json
{
  "comment": "Требуется уточнить финансовые показатели"
}
```

### Финальное утверждение

```http
POST /api/v1/financial-requests/123/final-approve
Content-Type: application/json
Authorization: Bearer <access-token>
```

```json
{
  "comment": "Заявка утверждена"
}
```

---

## 6.6. Примеры и схемы ответов

### Заявка

```http
GET /api/v1/financial-requests/123
```

```json
{
  "id": 123,
  "number": "FR-2026-00123",
  "title": "Финансирование проекта X",
  "purpose": "Финансирование реализации проекта X",
  "projectId": 42,
  "expectedStartDate": "2026-10-01",
  "expectedEndDate": "2027-09-30",
  "requestedSum": 15000000,
  "comment": "Дополнительная информация по заявке",
  "status": "CONSIDERATION",
  "stage": "EXPERT_REVIEW",
  "createdAt": "2026-09-24T10:15:00Z",
  "submittedAt": "2026-09-24T11:00:00Z",
  "approvedAt": null,
  "authorId": 15
}
```

### Состояние процесса

```http
GET /api/v1/financial-requests/123/process
```

```json
{
  "requestId": 123,
  "stage": "EXPERT_REVIEW",
  "status": "CONSIDERATION",
  "currentResponsibleId": 25,
  "startedAt": "2026-09-24T11:00:00Z",
  "updatedAt": "2026-09-24T12:30:00Z"
}
```

### История процесса

```http
GET /api/v1/financial-requests/123/history
```

```json
[
  {
    "fromStage": "PREPARATION",
    "toStage": "EXPERT_REVIEW",
    "fromStatus": "DRAFT",
    "toStatus": "CONSIDERATION",
    "action": "SUBMIT",
    "responsibleId": 15,
    "createdAt": "2026-09-24T11:00:00Z",
    "endedAt": "2026-09-24T14:30:00Z",
    "comment": null
  }
]
```

---

## 6.7. Бизнес-операции и управление состоянием заявки

`status` и `stage` не изменяются клиентом напрямую.

Например, запрос:

```http
PATCH /api/v1/financial-requests/123
```

с телом:

```json
{
  "status": "APPROVED",
  "stage": "FINAL_APPROVAL"
}
```

не допускается.

Переход выполняется соответствующей бизнес-операцией:

```http
POST /api/v1/financial-requests/123/approve
```

или:

```http
POST /api/v1/financial-requests/123/final-approve
```

При выполнении бизнес-операции система проверяет текущий статус, текущий этап, роль пользователя и допустимость перехода

[← Предыдущий раздел: База данных](05-База%20данных.md) · [Следующий раздел: Интеграции →](07-Интеграции.md)

[К портфолио →](00-Портфолио.md)
