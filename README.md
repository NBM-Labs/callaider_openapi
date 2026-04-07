# Callaider OpenAPI Специфікація

OpenAPI 3.0.3 специфікація для [Callaider](https://callaider.ai) API — платформи AI-дзвінків.

- **Базовий URL:** `https://api.callaider.ai`
- **Поточна версія:** 1.0.0

## Реалізовані модулі

| Модуль | Опис | Документація |
|--------|------|-------------|
| **Прозвон (Ringing)** | Масовий автоматичний обдзвон з AI-асистентами, постаналізом та вебхуками | [docs/ringing/RINGING_API.md](docs/ringing/RINGING_API.md) |

> Нові модулі будуть додаватися в міру розвитку публічного API.

## Швидкий старт

Валідація специфікації:

```bash
npx @redocly/cli lint spec/v1/openapi.json
```

Превʼю документації локально:

```bash
npx @redocly/cli preview spec/v1/openapi.json
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

Або використовуйте готову [Postman-колекцію](postman/callaider_api.postman_collection.json) з покроковим workflow.

## Авторизація

Кожен запит потребує заголовок `Authorization` з Bearer-токеном:

```
Authorization: Bearer cld_ваш_api_ключ
```

API-ключ створюється в особистому кабінеті CallAIder (розділ **Інтеграції → API-ключі**).
Формат ключа: `cld_` + 32 hex-символи.

## Структура проєкту

```
├── spec/v1/openapi.json                          # OpenAPI 3.0.3 специфікація
├── postman/callaider_api.postman_collection.json  # Postman-колекція
├── docs/
│   └── ringing/RINGING_API.md                     # Документація модуля Прозвон
├── .github/workflows/validate.yml                 # CI — валідація специфікації
├── .redocly.yaml                                  # Конфігурація Redocly
├── CHANGELOG.md                                   # Історія змін
└── LICENCE                                        # MIT
```

## Ліцензія

[MIT](LICENCE)
