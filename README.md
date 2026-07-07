# Callaider OpenAPI Специфікація

OpenAPI 3.0.3 специфікація для [Callaider](https://callaider.ai) API — платформи AI-дзвінків і AI-асистентів.

- **Базовий URL:** `https://api.callaider.ai`
- **Поточна версія:** 1.1.0

## Реалізовані модулі

| Модуль | Опис | Документація | Postman |
|--------|------|--------------|---------|
| **Прозвон (Ringing)** | Масовий автоматичний обдзвон з AI-асистентами, постаналізом та вебхуками | [docs/ringing/RINGING_API.md](docs/ringing/RINGING_API.md) | [postman/callaider_api.postman_collection.json](postman/callaider_api.postman_collection.json) |
| **External Conversation Bridge** | Server-to-server підключення власних текстових каналів до AI-асистентів Callaider | [docs/assistants/EXTERNAL_CONVERSATION_BRIDGE.md](docs/assistants/EXTERNAL_CONVERSATION_BRIDGE.md) | [postman/callaider_assistant_external_api.postman_collection.json](postman/callaider_assistant_external_api.postman_collection.json) |

## Швидкий старт

Валідація специфікації:

```bash
npx @redocly/cli lint spec/v1/openapi.json
```

Прев'ю документації локально:

```bash
npx @redocly/cli preview
```

Згенерувати HTML-документацію:

```bash
npx @redocly/cli build-docs spec/v1/openapi.json -o index.html
```

## Використання специфікації

Імпортуйте `spec/v1/openapi.json` у:

- [Postman](https://www.postman.com/) — імпорт як OpenAPI-колекція
- [openapi-generator](https://openapi-generator.tech/) — генерація клієнтських бібліотек
- [Swagger UI](https://swagger.io/tools/swagger-ui/) — інтерактивний API-браузер

Або використовуйте готові Postman-колекції з папки [postman](postman).

## Авторизація

Кожен запит потребує заголовок `Authorization` з Bearer-токеном:

```http
Authorization: Bearer cld_ваш_api_ключ
```

API-ключ створюється в особистому кабінеті Callaider (розділ **Інтеграції → API-ключі**).
Формат ключа: `cld_` + 32 hex-символи.

## Структура проєкту

```text
├── spec/v1/openapi.json                                      # OpenAPI 3.0.3 специфікація
├── postman/callaider_api.postman_collection.json             # Postman-колекція Ringing
├── postman/callaider_assistant_external_api.postman_collection.json # Postman-колекція External Conversation Bridge
├── docs/ringing/RINGING_API.md                               # Документація модуля Прозвон
├── docs/assistants/EXTERNAL_CONVERSATION_BRIDGE.md           # Документація External Conversation Bridge
├── .github/workflows/validate.yml                            # CI — валідація специфікації
├── .redocly.yaml                                             # Конфігурація Redocly
├── CHANGELOG.md                                              # Історія змін
└── LICENCE                                                   # MIT
```

## Ліцензія

[MIT](LICENCE)
