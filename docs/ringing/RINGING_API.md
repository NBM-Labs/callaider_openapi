# Callaider Ringing API — Документація для інтеграції

Зовнішнє REST API для масового автоматичного обдзвону (авто-прозвон) з AI-асистентами, постаналізом та вебхуками.

---

## Зміст

1. [Авторизація](#авторизація)
2. [Робочий процес (Workflow)](#робочий-процес-workflow)
3. [API Reference](#api-reference)
   - [Асистенти](#асистенти)
   - [SIP-акаунти](#sip-акаунти)
   - [Групи абонентів](#групи-абонентів)
   - [Абоненти](#абоненти)
   - [Кампанії](#кампанії)
   - [Управління кампанією](#управління-кампанією)
   - [Дзвінки кампанії](#дзвінки-кампанії)
   - [Конфігурації постаналізу](#конфігурації-постаналізу)
   - [Конфігурації вебхуків](#конфігурації-вебхуків)
4. [Постаналіз дзвінків](#постаналіз-дзвінків)
5. [Вебхуки](#вебхуки)
6. [Статуси та життєвий цикл](#статуси-та-життєвий-цикл)

---

## Авторизація

Усі запити потребують **API-ключ** вашої компанії. Ключ створюється у особистому кабінеті Callaider (розділ **Інтеграції → API-ключі**).

```
Authorization: Bearer cld_ваш_api_ключ
```

**Базовий URL:** `https://api.callaider.ai/v1/ringing`

---

## Робочий процес (Workflow)

### Огляд

```
┌─────────────────────────────────────────────────────────────────────┐
│                        ПІДГОТОВКА (в особистому кабінеті)            │
│                                                                     │
│  1. Створити AI-асистента (розділ «Асистенти»)                     │
│  2. Створити конфігурацію постаналізу (розділ «Прозвон»)            │
│  3. Створити конфігурацію вебхука (розділ «Прозвон»)                │
│  4. Створити API-ключ (розділ «Інтеграції»)                        │
└─────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        ІНТЕГРАЦІЯ (через API)                       │
│                                                                     │
│  1. Отримати ID конфігурацій  →  GET /post-analysis-configs         │
│                                  GET /webhook-configs               │
│  2. Отримати доступних        →  GET /assistants                    │
│     асистентів                                                      │
│     (опційно) SIP-акаунти     →  GET /sip-accounts                  │
│  3. Створити групу абонентів  →  POST /subscriber-groups            │
│  4. Завантажити номери        →  POST /subscribers/batch            │
│  5. Створити кампанію         →  POST /campaigns                    │
│  6. Запустити кампанію        →  POST /campaigns/:id/launch         │
│  7. Моніторити виконання      →  GET /campaigns/:id/statistics      │
│  8. Отримати результати       →  GET /campaigns/:id/calls           │
└─────────────────────────────────────────────────────────────────────┘
```

> **Важливо:** Конфігурації **постаналізу** та **вебхуків** створюються та налаштовуються в **особистому кабінеті** Callaider. Через API ви можете лише отримати їх список та використати `id` при створенні кампанії. Це ж стосується **AI-асистентів** — вони створюються в кабінеті, а через API можна отримати список доступних асистентів (`GET /assistants`) та використати `id` при створенні кампанії.

---

### Крок 1: Отримати ID конфігурацій

Перед створенням кампанії потрібно знати ідентифікатори конфігурацій, створених у кабінеті.

#### Отримати доступних асистентів

```http
GET /v1/ringing/assistants
Authorization: Bearer cld_ваш_ключ
```

**Відповідь:**
```json
[
  {
    "id": 15,
    "name": "Продажі — акції",
    "description": "Асистент для обдзвону з пропозиціями знижок",
    "language": "UK",
    "isEnabled": true,
    "createdAt": "2026-03-15T08:00:00.000Z"
  }
]
```

| Поле | Тип | Опис |
|---|---|---|
| `id` | `number` | Ідентифікатор асистента — використовується як `assistantId` при створенні кампанії |
| `name` | `string` | Назва асистента |
| `description` | `string \| null` | Опис асистента |
| `language` | `string` | Мова асистента |
| `isEnabled` | `boolean` | Чи активний асистент |
| `createdAt` | `string` | Дата створення |

> Для створення кампанії використовуйте `id` активного асистента (`isEnabled: true`).

#### Отримати конфігурації постаналізу

```http
GET /v1/ringing/post-analysis-configs
Authorization: Bearer cld_ваш_ключ
```

**Відповідь:**
```json
[
  {
    "id": 1,
    "name": "Стандартна оцінка",
    "evaluationPrompt": "Оціни розмову за шкалою 1-10...",
    "responseSchema": {"score": "number", "interested": "boolean"},
    "minCallDurationForEvaluation": 10,
    "isActive": true,
    "isDefault": false,
    "createdAt": "2026-04-01T10:00:00.000Z"
  }
]
```

> Запам'ятайте `id` — він потрібен як `postAnalysisConfigId` при створенні кампанії.

#### Отримати конфігурації вебхуків

```http
GET /v1/ringing/webhook-configs
Authorization: Bearer cld_ваш_ключ
```

**Відповідь:**
```json
[
  {
    "id": 1,
    "name": "CRM Інтеграція",
    "url": "https://my-crm.example.com/api/webhook",
    "method": "POST",
    "isActive": true,
    "retryAttempts": 3,
    "retryDelay": 30,
    "createdAt": "2026-04-01T10:00:00.000Z"
  }
]
```

> Запам'ятайте `id` — він потрібен як `webhookConfigId` при створенні кампанії.

---

### Крок 2: Отримати доступних асистентів

```http
GET /v1/ringing/assistants
Authorization: Bearer cld_ваш_ключ
```

**Відповідь:**
```json
[
  {
    "id": 15,
    "name": "Продажі — акції",
    "description": "Асистент для обдзвону з пропозиціями знижок",
    "language": "UK",
    "isEnabled": true,
    "createdAt": "2026-03-15T08:00:00.000Z"
  }
]
```

> Запам'ятайте `id` активного асистента (`isEnabled: true`) — він потрібен як `assistantId` при створенні кампанії.

---

### Крок 3: Створити групу абонентів

```http
POST /v1/ringing/subscriber-groups
Authorization: Bearer cld_ваш_ключ
Content-Type: application/json

{
  "name": "Клієнти квітень 2026",
  "description": "Імпорт з CRM"
}
```

**Відповідь:**
```json
{
  "id": 1,
  "companyId": 42,
  "name": "Клієнти квітень 2026",
  "description": "Імпорт з CRM",
  "createdAt": "2026-04-03T10:00:00.000Z"
}
```

---

### Крок 4: Завантажити абонентів

```http
POST /v1/ringing/subscribers/batch
Authorization: Bearer cld_ваш_ключ
Content-Type: application/json

{
  "subscriberGroupId": 1,
  "phones": [
    "+380501234567",
    "+380671234567",
    "+380931234567"
  ],
  "extraData": [
    {"name": "Іван Петренко", "orderId": "ORD-001"},
    {"name": "Олена Коваль", "orderId": "ORD-002"},
    {"name": "Тарас Шевченко", "orderId": "ORD-003"}
  ],
  "onlyUniquePhones": true
}
```

| Поле | Тип | Обов'язкове | Опис |
|---|---|---|---|
| `phones` | `string[]` | Так | Масив номерів у міжнародному форматі (1–5000 за запит). Зайві символи видаляються автоматично |
| `extraData` | `object[]` | Ні | Додаткові дані по кожному абоненту (індекс відповідає `phones`). Передаються AI-асистенту |
| `subscriberGroupId` | `number` | Ні* | ID існуючої групи |
| `subscriberGroupName` | `string` | Ні* | Назва нової групи (буде створена) |
| `onlyUniquePhones` | `boolean` | Ні | `true` — пропускати дублікати в межах групи |

> \* Потрібен або `subscriberGroupId`, або `subscriberGroupName`. Якщо не вказано жоден — автоматично створюється група `group_YYYY-MM-DD`.

**Нормалізація номерів:**
- Номери потрібно передавати у **міжнародному форматі** (з `+` та кодом країни) - або в тому форматі, який приймає **ваша телефонія** для вихідних викликів
- Пробіли, дефіси, дужки видаляються автоматично: `+38 (050) 123-45-67` → `+380501234567`

**Відповідь:**
```json
{
  "result": "Added 3 subscribers",
  "subscriberGroup": {
    "id": 1,
    "subscribersCount": 3
  }
}
```

---

### Крок 5: Створити кампанію

```http
POST /v1/ringing/campaigns
Authorization: Bearer cld_ваш_ключ
Content-Type: application/json

{
  "name": "Акція Весна 2026",
  "description": "Обдзвон клієнтів з пропозицією знижки",
  "assistantId": 15,
  "subscriberGroupId": 1,
  "postAnalysisConfigId": 1,
  "webhookConfigId": 1,
  "sipAccountIds": [10, 11]
}
```

| Поле | Тип | Обов'язкове | Опис |
|---|---|---|---|
| `name` | `string` | Так | Назва кампанії |
| `description` | `string` | Ні | Опис кампанії |
| `assistantId` | `number` | Так | ID AI-асистента (отримати через `GET /assistants`) |
| `subscriberGroupId` | `number` | Так | ID групи абонентів |
| `postAnalysisConfigId` | `number` | Ні | ID конфігурації постаналізу (отримати через `GET /post-analysis-configs`) |
| `webhookConfigId` | `number` | Ні | ID конфігурації вебхуку (отримати через `GET /webhook-configs`) |
| `sipAccountIds` | `number[]` | Ні | Список ID SIP-акаунтів, через які ітимуть дзвінки (отримати через `GET /sip-accounts`). Якщо не вказано — використовуються всі активні SIP-акаунти компанії |
| `scheduledAt` | `string` | Ні | Запланований час запуску (ISO 8601) |

> Без `postAnalysisConfigId` — дзвінки не оцінюватимуться AI. Без `webhookConfigId` — вебхуки не відправлятимуться.

**Відповідь:**
```json
{
  "id": 1,
  "name": "Акція Весна 2026",
  "status": "created",
  "launched": false,
  "assistantId": 15,
  "subscriberGroupId": 1,
  "postAnalysisConfigId": 1,
  "webhookConfigId": 1,
  "sipAccountIds": [10, 11],
  "createdAt": "2026-04-03T12:00:00.000Z"
}
```

---

### Крок 6: Запустити кампанію

```http
POST /v1/ringing/campaigns/1/launch
Authorization: Bearer cld_ваш_ключ
Content-Type: application/json

{
  "maxCallTryCount": 3,
  "callDelay": 1
}
```

| Поле | Тип | За замовч. | Опис |
|---|---|---|---|
| `maxCallTryCount` | `number` | `3` | Максимальна кількість спроб дозвону на абонента (1–3). Зберігається при першому запуску, ігнорується при повторних |
| `callDelay` | `number` | `1` | Затримка між дзвінками в секундах (0–10) |

Після запуску статус кампанії → `pending`, потім автоматично → `in_progress` (коли з'являться записи дзвінків).

#### Повторний запуск (re-launch)

Той самий endpoint `POST /campaigns/:id/launch` підтримує повторний запуск завершених або невдалих кампаній:

- Кампанію можна перезапустити до **4 разів** (1 початковий + 3 повторних)
- При повторному запуску система знаходить абонентів, яким **не вдалося додзвонитися** (тільки `failed`, без жодного успішного дзвінка) і кількість невдалих спроб < `maxCallTryCount`
- Для знайдених абонентів створюються **нові записи дзвінків** (старі не змінюються — зберігається повний аудіт-трейл)
- Якщо eligible абонентів немає — кампанія автоматично позначається як `completed`
- `maxCallTryCount` та `callDelay` зберігаються з першого запуску і не можуть бути змінені при повторних

---

### Крок 7: Моніторинг

```http
GET /v1/ringing/campaigns/1/statistics
Authorization: Bearer cld_ваш_ключ
```

**Відповідь:**
```json
{
  "campaignId": 1,
  "status": "in_progress",
  "total": 100,
  "pending": 30,
  "in_progress": 5,
  "completed": 45,
  "failed": 10,
  "evaluated": 10,
  "evaluating": 0,
  "progress": 65,
  "startedAt": "2026-04-05T09:00:15.000Z"
}
```

---

### Крок 8: Отримати результати

```http
GET /v1/ringing/campaigns/1/calls?status=evaluated
Authorization: Bearer cld_ваш_ключ
```

Деталі конкретного дзвінка:

```http
GET /v1/ringing/calls/42
Authorization: Bearer cld_ваш_ключ
```

**Відповідь:**
```json
{
  "id": 42,
  "campaignId": 1,
  "subscriberId": 15,
  "assistantId": 15,
  "status": "evaluated",
  "callDuration": 125,
  "endReason": "normal_clearing",
  "transcript": "Асистент: Доброго дня! Олено...",
  "summary": "Клієнтка зацікавлена у пропозиції...",
  "recordingUrl": "https://storage.example.com/recording.mp3",
  "evaluationStatus": "success",
  "evaluationResult": {
    "interested": true,
    "score": 8,
    "summary": "Клієнтка виявила інтерес до продукту"
  },
  "webhookSent": true,
  "webhookSentAt": "2026-04-05T09:05:30.000Z",
  "attemptNumber": 1,
  "createdAt": "2026-04-05T09:00:20.000Z",
  "updatedAt": "2026-04-05T09:05:30.000Z"
}
```

URL запису дзвінка:

```http
GET /v1/ringing/calls/42/recording
Authorization: Bearer cld_ваш_ключ
```

---

## API Reference

### Асистенти

> Асистенти створюються та налаштовуються в **особистому кабінеті**. Через API доступне лише читання скороченої інформації.

| Метод | Шлях | Опис |
|---|---|---|
| `GET` | `/assistants` | Список доступних асистентів (скорочений формат) |

Використовуйте `id` з відповіді як `assistantId` при створенні кампанії.

**Поля відповіді:**

| Поле | Тип | Опис |
|---|---|---|
| `id` | `number` | Ідентифікатор асистента |
| `name` | `string` | Назва |
| `description` | `string \| null` | Опис |
| `language` | `string` | Мова асистента |
| `isEnabled` | `boolean` | Чи активний |
| `createdAt` | `string` | Дата створення |

---

### Групи абонентів

| Метод | Шлях | Опис |
|---|---|---|
| `POST` | `/subscriber-groups` | Створити групу |
| `GET` | `/subscriber-groups` | Список груп (з полем `subscribersCount`) |
| `GET` | `/subscriber-groups/:id` | Деталі групи |
| `PUT` | `/subscriber-groups/:id` | Оновити групу |
| `DELETE` | `/subscriber-groups/:id` | Видалити групу (каскадно видаляє абонентів!) |

**Створити групу — Body:**
```json
{
  "name": "string (обов'язкове, унікальне)",
  "description": "string"
}
```

---

### Абоненти

| Метод | Шлях | Опис |
|---|---|---|
| `POST` | `/subscribers/batch` | Завантажити пакет абонентів (до 5000) |
| `GET` | `/subscribers?groupId=1` | Список абонентів (фільтр за групою) |
| `DELETE` | `/subscribers/:id` | Видалити абонента |

---

### SIP-акаунти

> SIP-акаунти створюються та налаштовуються в **особистому кабінеті** (розділ «Телефонія»). Через API доступне лише читання скороченої інформації.

| Метод | Шлях | Опис |
|---|---|---|
| `GET` | `/sip-accounts` | Список SIP-акаунтів компанії |

**Запит:**
```http
GET /v1/ringing/sip-accounts
Authorization: Bearer cld_ваш_ключ
```

**Відповідь:**
```json
[
  {
    "id": 10,
    "name": "Київстар основний",
    "trunkType": "sip",
    "callerId": "+380441234567",
    "status": "registered",
    "isEnabled": true
  }
]
```

| Поле | Тип | Опис |
|---|---|---|
| `id` | `number` | Ідентифікатор — використовується як елемент масиву `sipAccountIds` при створенні кампанії |
| `name` | `string` | Назва акаунту |
| `trunkType` | `string` | Тип транка (`sip`, `pjsip`, тощо) |
| `callerId` | `string \| null` | Номер, що відображається викликуваному |
| `status` | `string` | Поточний статус реєстрації |
| `isEnabled` | `boolean` | Чи активний акаунт |

> Для кампанії беруть до уваги лише акаунти з `isEnabled: true`. Якщо `sipAccountIds` не передано при створенні кампанії — використовуються всі активні SIP-акаунти компанії.

---

### Кампанії

| Метод | Шлях | Опис |
|---|---|---|
| `POST` | `/campaigns` | Створити кампанію |
| `GET` | `/campaigns` | Список кампаній (`?status=` для фільтрації) |
| `GET` | `/campaigns/:id` | Деталі кампанії |
| `PUT` | `/campaigns/:id` | Оновити (тільки до запуску) |
| `DELETE` | `/campaigns/:id` | Видалити кампанію |

**Фільтрація:** `?status=created|pending|in_progress|paused|completed|failed`

---

### Управління кампанією

| Метод | Шлях | Опис |
|---|---|---|
| `POST` | `/campaigns/:id/launch` | Запустити / перезапустити кампанію (до 4 разів) |
| `POST` | `/campaigns/:id/pause` | Поставити на паузу |
| `POST` | `/campaigns/:id/resume` | Відновити після паузи |
| `GET` | `/campaigns/:id/statistics` | Статистика виконання |

---

### Дзвінки кампанії

| Метод | Шлях | Опис |
|---|---|---|
| `GET` | `/campaigns/:id/calls` | Список дзвінків (`?status=` для фільтрації) |
| `GET` | `/calls/:id` | Деталі дзвінка |
| `GET` | `/calls/:id/recording` | URL запису дзвінка |

**Фільтрація дзвінків:** `?status=pending|acquiring_slot|in_progress|completed|failed|evaluating|evaluated`

---

### Конфігурації постаналізу

> Створюються та редагуються в **особистому кабінеті**. Через API доступне лише читання.

| Метод | Шлях | Опис |
|---|---|---|
| `GET` | `/post-analysis-configs` | Список конфігурацій |
| `GET` | `/post-analysis-configs/:id` | Деталі конфігурації |

Використовуйте `id` з відповіді як `postAnalysisConfigId` при створенні кампанії.

**Поля конфігурації:**

| Поле | Тип | Опис |
|---|---|---|
| `id` | `number` | Ідентифікатор |
| `name` | `string` | Назва конфігурації |
| `evaluationPrompt` | `string` | Промпт для AI-оцінки транскрипта |
| `responseSchema` | `object` | Очікувана структура JSON-відповіді AI |
| `minCallDurationForEvaluation` | `number` | Мінімальна тривалість дзвінка (сек) для оцінки |
| `isActive` | `boolean` | Чи активна конфігурація |
| `isDefault` | `boolean` | Чи є конфігурацією за замовчуванням |

---

### Конфігурації вебхуків

> Створюються та редагуються в **особистому кабінеті**. Через API доступне лише читання та тестування.

| Метод | Шлях | Опис |
|---|---|---|
| `GET` | `/webhook-configs` | Список конфігурацій |
| `GET` | `/webhook-configs/:id` | Деталі конфігурації |
| `POST` | `/webhook-configs/:id/test` | Відправити тестовий вебхук |

Використовуйте `id` з відповіді як `webhookConfigId` при створенні кампанії.

**Тестовий вебхук — Відповідь:**
```json
{
  "success": true,
  "statusCode": 200
}
```

---

## Постаналіз дзвінків

Після завершення кожного дзвінка система автоматично виконує AI-постаналіз (якщо кампанію створено з `postAnalysisConfigId`):

1. Перевіряється мінімальна тривалість дзвінка (`minCallDurationForEvaluation`)
2. Транскрипт + промпт відправляються на AI-аналіз
3. Результат зберігається в полі `evaluationResult` (JSON)

**Приклад результату оцінки дзвінка:**
```json
{
  "evaluationStatus": "success",
  "evaluationResult": {
    "interested": true,
    "score": 8,
    "summary": "Клієнтка виявила інтерес до продукту",
    "keywords": ["знижка", "замовлення", "доставка"],
    "nextAction": "відправити пропозицію"
  }
}
```

Дзвінки коротші за `minCallDurationForEvaluation` пропускаються — отримують статус `evaluated` без оцінки.

---

## Вебхуки

Якщо кампанію створено з `webhookConfigId`, після завершення обробки кожного дзвінка на вказаний URL відправляється HTTP-запит.

### Тіло вебхука

Формат визначається конфігурацією (налаштовується в кабінеті):

**Варіант 1 — `payloadTemplate`** (кастомний маппінг):
```json
{
  "call_result": "evaluationResult",
  "phone": "subscriberId",
  "duration": "callDuration"
}
```
Підтримується dot-notation: `evaluationResult.score`, `evaluationResult.interested`.

**Варіант 2 — `includeFields`** (обрані поля):
```json
["callId", "status", "callDuration", "transcript", "evaluationResult"]
```

**Варіант 3 — за замовчуванням** (повний набір):
```json
{
  "callId": 42,
  "campaignId": 1,
  "subscriberId": 15,
  "phoneNumber": "+380501234567",
  "extraData": {"name": "Іван Петренко", "orderId": "ORD-001"},
  "status": "evaluated",
  "callDuration": 125,
  "endReason": "normal_clearing",
  "transcript": "...",
  "summary": "...",
  "evaluationResult": {"interested": true, "score": 8},
  "evaluationStatus": "success",
  "recordingUrl": "https://...",
  "attemptNumber": 1,
  "createdAt": "...",
  "updatedAt": "..."
}
```

> `extraData` — ті самі дані, що були передані при завантаженні абонентів через `POST /subscribers/batch`. Повертаються as-is для зіставлення з вашою системою.

### Retry логіка

| Параметр | За замовч. | Опис |
|---|---|---|
| `retryAttempts` | 3 | Кількість повторних спроб |
| `retryDelay` | 30 сек | Початковий інтервал |

Стратегія: Exponential backoff — `retryDelay × 2^спроба` (30с → 60с → 120с). Таймаут запиту: 30 секунд.

---

## Статуси та життєвий цикл

### Статуси кампанії

| Статус | Опис |
|---|---|
| `created` | Створено, очікує запуску |
| `pending` | Запущено, створюються записи дзвінків |
| `in_progress` | Йде обдзвон |
| `paused` | На паузі (нові дзвінки не ініціюються) |
| `completed` | Усі дзвінки оброблено |
| `failed` | Помилка |

```
created → launch() → pending → in_progress → completed
                                    │               │
                                    ├→ paused        ├→ launch() → pending (re-launch)
                                    │   │            │
                                    │   └→ resume()  └→ (до 4 запусків)
                                    │       │
                                    │       └→ in_progress
                                    └→ failed
                                         │
                                         └→ launch() → pending (re-launch)
```

> **Re-launch:** Завершені (`completed`) та невдалі (`failed`) кампанії можна перезапустити через `POST /campaigns/:id/launch`. При повторному запуску обдзвонюються тільки абоненти без успішних дзвінків (в межах ліміту `maxCallTryCount`).

### Статуси дзвінка

| Статус | Опис |
|---|---|
| `pending` | Очікує обробки |
| `acquiring_slot` | Бронюється SIP-лінія |
| `in_progress` | Дзвінок активний |
| `completed` | Завершено, очікує постаналіз |
| `failed` | Не вдався |
| `evaluating` | Виконується AI-постаналіз |
| `evaluated` | Постаналіз завершено |

### Статуси постаналізу

| Статус | Опис |
|---|---|
| `none` | Не виконувався |
| `pending` | Очікує |
| `processing` | AI обробляє |
| `success` | Успішно |
| `failure` | Помилка |
