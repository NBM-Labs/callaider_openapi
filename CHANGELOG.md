# Changelog

## [1.2.0] - 2026-07-27

### Додано
- OpenAPI endpoint `GET /v1/ringing/post-call-workflows` для отримання безпечного довідника воркфлоу з контекстом `ringing.post_call`.
- Поле `postCallWorkflowId` у створенні, оновленні та відповіді кампанії.
- Приклади використання воркфлоу після дзвінка в документації Ringing API та Postman-колекції.

## [1.1.0] - 2026-07-07

### Додано
- Розділ **External Conversation Bridge** для роботи з текстовими AI-асистентами через власні зовнішні канали.
- OpenAPI endpoint `POST /v1/assistants/{assistantId}/external-conversations/messages`.
- OpenAPI endpoint `POST /v1/assistants/{assistantId}/external-conversations/{externalConversationId}/state`.
- Схеми запитів і відповідей для повідомлень, вкладень `remote_url`, idempotency, статусів `active`, `paused`, `closed`, `blocked`.
- Документацію [docs/assistants/EXTERNAL_CONVERSATION_BRIDGE.md](docs/assistants/EXTERNAL_CONVERSATION_BRIDGE.md).
- Postman-колекцію [postman/callaider_assistant_external_api.postman_collection.json](postman/callaider_assistant_external_api.postman_collection.json).

## [1.0.0] - 2026-04-07

### Додано
- Початкова публікація OpenAPI 3.0.3 специфікації Callaider API.
- Розділ **Прозвон (Ringing)**: кампанії, групи абонентів, абоненти, дзвінки, постаналіз, вебхуки, асистенти.
- Postman-колекція з покроковим workflow.
- Документація для інтеграції [docs/ringing/RINGING_API.md](docs/ringing/RINGING_API.md).
