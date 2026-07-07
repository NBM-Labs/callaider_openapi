# External Conversation Bridge

Інструкція для підключення власного текстового каналу до AI-асистента Callaider через публічний API.

## Призначення

External Conversation Bridge дозволяє підключити асистента Callaider до каналу, яким керує клієнтська система. Це може бути Telegram-бот, WhatsApp, Instagram, Viber, CRM-чат, сайт, support desk або власний операторський інтерфейс.

У цьому режимі Callaider приймає повідомлення від вашого backend-сервісу, підтримує контекст діалогу і повертає текстові відповіді асистента. Доставку повідомлень у зовнішній канал виконує ваша система.

## Що залишається на стороні клієнта

External Conversation Bridge не є готовим конектором до конкретного месенджера. На стороні клієнтської системи залишаються:

- токени Telegram, WhatsApp, Instagram, Viber, CRM та інших каналів;
- webhook-и, polling і отримання подій із зовнішнього каналу;
- перетворення повідомлень у payload Callaider;
- доставка відповідей Callaider назад користувачу;
- логіка handoff між AI та оператором;
- черги, retries, таймаути, rate limiting і фільтрація повідомлень.

API-ключ Callaider не можна передавати у браузер, мобільний застосунок або кінцевим користувачам. Використовуйте цей API тільки server-to-server.

## Основна схема

```text
Користувач у зовнішньому каналі
        |
        v
Клієнтський backend / bridge server
        |
        | POST /v1/assistants/:assistantId/external-conversations/messages
        | Authorization: Bearer cld_...
        v
Callaider Public API
        |
        v
JSON response: messages[]
        |
        v
Клієнтський backend доставляє messages[] у свій канал
```

## Налаштування асистента

1. Відкрийте асистента в кабінеті Callaider.
2. Перейдіть у `Channels`.
3. Увімкніть `Зовнішній API-канал`.
4. Збережіть налаштування.
5. Скопіюйте `Ідентифікатор бота в API` — це значення використовується як `assistantId`.

Base URL для публічного API:

```text
https://api.callaider.ai/v1
```

## Авторизація

Кожен запит має містити API-ключ компанії у header `Authorization`:

```http
Authorization: Bearer cld_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

API-ключ створюється в кабінеті Callaider і прив'язаний до компанії.

Важливо:

- API-ключ Callaider не є токеном месенджера;
- API-ключ Callaider не є preview token або JWT користувача;
- API-ключ не можна комітити у frontend-код, mobile app або публічний репозиторій;
- API-ключ потрібно зберігати тільки на backend-стороні клієнта.

## Postman Collection

Для ручного тестування використовуйте готову Postman collection:

```text
postman/callaider_assistant_external_api.postman_collection.json
```

Після імпорту задайте collection variables:

| Variable | Значення |
| --- | --- |
| `base_url` | `https://api.callaider.ai/v1` |
| `api_key` | API-ключ компанії у форматі `cld_...` |
| `assistant_id` | ID асистента з поля `Ідентифікатор бота в API` |
| `external_conversation_id` | Стабільний ID тестового діалогу на стороні клієнта |
| `external_user_id` | ID тестового користувача у вашому каналі |

## Надіслати Повідомлення

```http
POST /v1/assistants/:assistantId/external-conversations/messages
Authorization: Bearer <API_KEY>
Content-Type: application/json
```

### Мінімальний Payload

```json
{
  "externalConversationId": "tg:123456",
  "externalUserId": "123456",
  "messageId": "789",
  "text": "Добрий день"
}
```

### Повний Payload

```json
{
  "externalConversationId": "tg:123456",
  "externalUserId": "123456",
  "messageId": "789",
  "text": "Подивись, будь ласка, цей рахунок",
  "attachments": [
    {
      "type": "remote_url",
      "url": "https://client.example.com/files/invoice-1001.pdf",
      "mimeType": "application/pdf",
      "filename": "invoice-1001.pdf",
      "sizeBytes": 245760,
      "sha256": "f2c7d0b4f4e8f1b7a12a4d34a9f6b7c0d1e2f3a4567890abcdef1234567890ab"
    }
  ],
  "metadata": {
    "username": "client_user",
    "chatType": "private",
    "topicId": 42,
    "rawChannelMessageId": 789
  }
}
```

### Поля Payload

| Поле | Тип | Обов'язкове | Опис |
| --- | --- | --- | --- |
| `externalConversationId` | string | Так | Стабільний ID діалогу на стороні клієнта. За ним Callaider продовжує контекст розмови. |
| `externalUserId` | string | Ні | ID користувача у зовнішньому каналі. |
| `messageId` | string | Ні, але рекомендовано | ID вхідного повідомлення. Використовуйте те саме значення при retry одного й того самого повідомлення. |
| `text` | string | Так, якщо немає `attachments` | Текст, який потрібно передати асистенту. |
| `attachments` | array | Ні | До 5 вкладень типу `remote_url`. |
| `metadata` | object | Ні | Додатковий JSON object для аудиту або інтеграції на стороні клієнта. |

## Attachments

External Conversation Bridge підтримує вкладення через короткоживучий публічний HTTPS URL. Файл не передається напряму в JSON.

```json
{
  "type": "remote_url",
  "url": "https://client.example.com/files/photo.jpg",
  "mimeType": "image/jpeg",
  "filename": "photo.jpg",
  "sizeBytes": 734003,
  "sha256": "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef"
}
```

| Поле | Тип | Обов'язкове | Опис |
| --- | --- | --- | --- |
| `type` | string | Так | Підтримується значення `remote_url`. |
| `url` | string | Так | Публічний HTTPS URL, доступний Callaider під час обробки повідомлення. |
| `mimeType` | string | Ні, але рекомендовано | Очікуваний MIME type. Корисно для signed URL, які повертають `application/octet-stream`. |
| `filename` | string | Ні | Назва файла для відображення у контексті діалогу. |
| `sizeBytes` | number | Ні | Очікуваний розмір файла. Якщо значення перевищує ліміт, запит буде відхилено. |
| `sha256` | string | Ні | SHA-256 checksum файла. Якщо передано, Callaider перевірить checksum після завантаження. |

Підтримувані MIME types:

- `image/png`, `image/jpeg`, `image/webp`, `image/heic`, `image/heif`, `image/gif`;
- `video/mp4`;
- `audio/wav`, `audio/x-wav`, `audio/wave`, `audio/mp3`, `audio/mpeg`, `audio/aiff`, `audio/x-aiff`, `audio/aac`, `audio/ogg`, `audio/flac`;
- `application/pdf`.

Правила безпеки для `remote_url`:

- приймається тільки `https`;
- URL не може містити username/password;
- hostname має резолвитись у публічну IP-адресу;
- private, localhost, link-local, multicast та службові IP-діапазони блокуються;
- redirect-и обмежені і перевіряються;
- розмір файла перевіряється;
- не рекомендується передавати довгоживучі URL з приватними секретами у query string.

Рекомендовано використовувати short-lived signed URL або власний одноразовий download endpoint клієнта.

## Ліміти

| Поле | Ліміт |
| --- | --- |
| `externalConversationId` | до 128 символів |
| `externalUserId` | до 128 символів |
| `messageId` | до 128 символів |
| `text` | до 20000 символів |
| `attachments` | до 5 файлів на повідомлення |
| один attachment | до 20 MB |
| `metadata` | тільки JSON object |

## Відповідь На Повідомлення

### Успішна Відповідь

```json
{
  "ok": true,
  "status": "active",
  "messages": [
    {
      "text": "Добрий день! Чим можу допомогти?"
    }
  ]
}
```

`messages` — масив відповідей, які клієнтська програма має доставити у свій канал у тому самому порядку.

### Повторне Повідомлення

Якщо `messageId` збігається з останнім обробленим повідомленням для цього `externalConversationId`, Callaider не генерує повторну відповідь:

```json
{
  "ok": true,
  "status": "active",
  "duplicate": true,
  "messages": []
}
```

Клієнтська програма має трактувати `duplicate: true` як "нічого не доставляти користувачу".

### Діалог На Паузі Або Заблокований

Якщо діалог має статус `paused` або `blocked`, асистент не відповідає:

```json
{
  "ok": true,
  "status": "paused",
  "skipped": true,
  "messages": []
}
```

Це зручно для сценаріїв, коли діалог веде оператор.

### Нове Звернення Після Закриття

Статус `closed` означає, що поточне звернення завершене. Якщо користувач пізніше напише у той самий `externalConversationId`, звичайний `POST /messages` відкриє новий контекст звернення:

```json
{
  "ok": true,
  "status": "active",
  "reopened": true,
  "messages": [
    {
      "text": "Добрий день! Чим можу допомогти?"
    }
  ]
}
```

## Змінити Стан Діалогу

```http
POST /v1/assistants/:assistantId/external-conversations/:externalConversationId/state
Authorization: Bearer <API_KEY>
Content-Type: application/json
```

### Поставити На Паузу

```json
{
  "status": "paused",
  "externalUserId": "123456",
  "metadata": {
    "reason": "operator_joined",
    "operatorId": "op-17"
  }
}
```

Відповідь:

```json
{
  "ok": true,
  "status": "paused"
}
```

### Повернути Асистенту

```json
{
  "status": "active",
  "metadata": {
    "reason": "operator_left"
  }
}
```

### Закрити Звернення

```json
{
  "status": "closed",
  "metadata": {
    "reason": "conversation_finished"
  }
}
```

### Заблокувати

```json
{
  "status": "blocked",
  "metadata": {
    "reason": "spam_or_abuse"
  }
}
```

## Підтримувані Статуси

| Вхідне значення | Збережений статус | Поведінка |
| --- | --- | --- |
| `active` | `active` | Асистент може відповідати. |
| `ai_active` | `active` | Alias для `active`. |
| `paused` | `paused` | Асистент не відповідає. |
| `handoff` | `paused` | Alias для `paused`. |
| `human_handoff` | `paused` | Alias для `paused`. |
| `closed` | `closed` | Поточне звернення завершене. Наступне повідомлення з тим самим `externalConversationId` відкриє новий контекст. |
| `blocked` | `blocked` | Асистент не відповідає. |

## Conversation ID

`externalConversationId` визначає, який контекст діалогу буде використано. Значення має бути стабільним для одного діалогу у вашому каналі.

Приклади:

| Сценарій | Приклад `externalConversationId` |
| --- | --- |
| Один діалог на Telegram-користувача | `tg:user:123456` |
| Один діалог на Telegram-чат | `tg:chat:-1001234567890` |
| Окремий діалог на Telegram forum topic | `tg:-1001234567890:topic:42` |
| CRM ticket | `crm:ticket:ABC-1001` |
| Support chat | `support:conversation:7841` |

Різні асистенти можуть використовувати однаковий `externalConversationId` без конфлікту, оскільки `assistantId` передається в URL endpoint-а.

Якщо `externalConversationId` містить reserved URL-символи у `/state` endpoint, передавайте його URL-encoded.

## Handoff З Оператором

Типовий сценарій:

1. Користувач пише у Telegram, WhatsApp, CRM або інший канал.
2. Клієнтська програма відправляє повідомлення в Callaider через `POST /messages`.
3. Callaider повертає `messages[]`.
4. Клієнтська програма доставляє відповідь користувачу.
5. Оператор бере діалог.
6. Клієнтська програма викликає `/state` зі статусом `paused` або `handoff`.
7. Нові повідомлення користувача можуть продовжувати надходити в Callaider, але відповідь міститиме `skipped: true`.
8. Якщо оператор повертає діалог асистенту, клієнтська програма викликає `/state` зі статусом `active`.
9. Якщо оператор закрив питання, клієнтська програма викликає `/state` зі статусом `closed`.
10. Коли користувач пізніше напише у той самий чат, клієнтська програма надсилає звичайний `POST /messages`, і Callaider починає новий контекст звернення.

Приклад pause:

```bash
curl -X POST \
  "https://api.callaider.ai/v1/assistants/123/external-conversations/tg%3A123456/state" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <API_KEY>" \
  -d '{
    "status": "handoff",
    "metadata": {
      "operatorId": "op-17"
    }
  }'
```

Приклад resume:

```bash
curl -X POST \
  "https://api.callaider.ai/v1/assistants/123/external-conversations/tg%3A123456/state" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <API_KEY>" \
  -d '{
    "status": "active"
  }'
```

Приклад close:

```bash
curl -X POST \
  "https://api.callaider.ai/v1/assistants/123/external-conversations/tg%3A123456/state" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <API_KEY>" \
  -d '{
    "status": "closed",
    "metadata": {
      "reason": "operator_closed_issue"
    }
  }'
```

## Приклади Викликів

### Просте Повідомлення

```bash
curl -X POST \
  "https://api.callaider.ai/v1/assistants/123/external-conversations/messages" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <API_KEY>" \
  -d '{
    "externalConversationId": "tg:123456",
    "externalUserId": "123456",
    "messageId": "789",
    "text": "Добрий день"
  }'
```

### Telegram Forum Topic

```bash
curl -X POST \
  "https://api.callaider.ai/v1/assistants/123/external-conversations/messages" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <API_KEY>" \
  -d '{
    "externalConversationId": "tg:-1001234567890:topic:42",
    "externalUserId": "123456",
    "messageId": "789",
    "text": "Потрібна допомога з оплатою",
    "metadata": {
      "chatId": "-1001234567890",
      "messageThreadId": 42,
      "username": "client_user"
    }
  }'
```

### Повідомлення З Attachment URL

```bash
curl -X POST \
  "https://api.callaider.ai/v1/assistants/123/external-conversations/messages" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <API_KEY>" \
  -d '{
    "externalConversationId": "crm:ticket:ABC-1001",
    "externalUserId": "customer:555",
    "messageId": "crm-msg-9002",
    "text": "Перевір, будь ласка, цей рахунок",
    "attachments": [
      {
        "type": "remote_url",
        "url": "https://client.example.com/downloads/invoice-1001.pdf",
        "mimeType": "application/pdf",
        "filename": "invoice-1001.pdf",
        "sizeBytes": 245760
      }
    ]
  }'
```

### CRM-Чат

```bash
curl -X POST \
  "https://api.callaider.ai/v1/assistants/123/external-conversations/messages" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <API_KEY>" \
  -d '{
    "externalConversationId": "crm:ticket:ABC-1001",
    "externalUserId": "customer:555",
    "messageId": "crm-msg-9001",
    "text": "Я хочу змінити дату запису"
  }'
```

## Приклад Клієнтської Інтеграції

```ts
type BridgeMessage = {
  externalConversationId: string;
  externalUserId?: string;
  messageId?: string;
  text: string;
  metadata?: Record<string, unknown>;
};

async function askCallaider(message: BridgeMessage) {
  const response = await fetch(
    `${process.env.CALLAIDER_API_BASE_URL}/assistants/${process.env.CALLAIDER_ASSISTANT_ID}/external-conversations/messages`,
    {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${process.env.CALLAIDER_API_KEY}`,
      },
      body: JSON.stringify(message),
    },
  );

  if (!response.ok) {
    throw new Error(`Callaider request failed: ${response.status}`);
  }

  return response.json() as Promise<{
    ok: boolean;
    status: string;
    duplicate?: boolean;
    skipped?: boolean;
    reopened?: boolean;
    messages: Array<{ text: string }>;
  }>;
}

async function handleIncomingText(input: {
  chatId: string;
  userId: string;
  messageId: string;
  text: string;
}) {
  const result = await askCallaider({
    externalConversationId: `tg:${input.chatId}`,
    externalUserId: input.userId,
    messageId: input.messageId,
    text: input.text,
  });

  if (result.skipped || result.duplicate) {
    return;
  }

  for (const message of result.messages) {
    if (message.text.trim()) {
      await sendTextToExternalChannel(input.chatId, message.text);
    }
  }
}
```

## Retry, Idempotency Та Таймаути

Рекомендації для клієнтської системи:

- завжди передавайте стабільний `messageId`;
- не генеруйте новий `messageId` при retry одного й того самого повідомлення;
- при `duplicate: true` не доставляйте нічого користувачу;
- встановіть HTTP timeout не менше 30-60 секунд;
- використовуйте чергу або lock на один `externalConversationId`, якщо канал очікує послідовний діалог;
- логувати request id або власний correlation id на стороні клієнта;
- retry виконуйте тільки з тим самим `messageId`.

Endpoint `/messages` відповідає синхронно: після обробки повідомлення він повертає готовий масив `messages[]`. Асинхронний callback-режим у цьому API не використовується.

## Помилки

| HTTP status | Причина |
| --- | --- |
| `400` | Некоректний payload: не передано `externalConversationId`, `text` або `attachments`, або передано некоректний `status`. |
| `401` | API-ключ відсутній або некоректний. |
| `403` | Зовнішній API-канал вимкнений, асистент вимкнений або доступ до API недоступний для компанії. |
| `404` | Асистента або активний зовнішній API-канал не знайдено. |
| `500` | Тимчасова помилка на стороні Callaider. Повторіть запит з тим самим `messageId`, якщо це retry того самого повідомлення. |

## Безпека

Обов'язково:

- використовуйте HTTPS;
- викликайте API тільки з backend-сервісу;
- не викликайте API напряму з браузера кінцевого користувача;
- не логувати API-ключ у відкриті логи;
- не передавайте Callaider токени Telegram, WhatsApp, CRM або інших зовнішніх систем;
- обмежте доступ до власного bridge server;
- додайте rate limiting на стороні reverse proxy або клієнтського bridge server;
- використовуйте short-lived URL для attachments.

## Перевірка Інтеграції

1. Увімкнути `Зовнішній API-канал` у налаштуваннях асистента.
2. Скопіювати `Ідентифікатор бота в API`.
3. Імпортувати Postman collection і задати `base_url`, `api_key`, `assistant_id`.
4. Надіслати тестовий `POST /messages` з унікальним `externalConversationId` і `messageId`.
5. Переконатися, що відповідь містить `ok: true`, `status: active`, `messages[]`.
6. Повторити той самий запит з тим самим `messageId`.
7. Переконатися, що відповідь містить `duplicate: true`, `messages: []`.
8. Викликати `/state` зі статусом `paused`.
9. Надіслати нове повідомлення.
10. Переконатися, що відповідь містить `skipped: true`, `messages: []`.
11. Викликати `/state` зі статусом `active`.
12. Надіслати нове повідомлення і перевірити, що асистент знову відповідає.
13. Викликати `/state` зі статусом `closed`.
14. Надіслати нове повідомлення з тим самим `externalConversationId` і новим `messageId`.
15. Переконатися, що відповідь містить `status: active`, `reopened: true` і нові `messages[]`.
