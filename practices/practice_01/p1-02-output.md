# Результат работы модели P1-02

summary: Добавлен эндпойнт POST /api/reviews и метод ReviewService.review, который формирует prompt из diff и отправляет его во внешний LLM. Ответ API сейчас проксирует результат модели в поле comment.

risks:

- file: app/review_service.py
  line: 20
  evidence: 'prompt = f"Review this pull request and find problems:\n{diff}"'
  risk: Нарушение SEC-1 — diff отправляется во внешний LLM без маскировки секретов (token, пароль, приватный ключ).
- file: app/review_service.py
  line: 22
  evidence: 'return {"comment": answer}'
  risk: Нарушение OUT-1 — формат ответа не соответствует контракту: требуется summary, массив risks и массив checks (до 3 элементов), а возвращается comment.
- file: app/api.py
  line: 36-37
  evidence: 'def create_review(payload: dict) -> dict[str, str]:\n return review_service.review(payload["diff"])'
  risk: Нарушение API-1 — нет проверки длины diff и возврата HTTP 413 при >20000 символов.

checks:

- SEC-1: передать diff с тестовым секретом, например 'token=abcd1234' или 'PRIVATE_KEY=...'; убедиться, что в prompt, который уходит в LLM, секреты заменены на [REDACTED]. Сейчас секрет уходит «как есть».
- OUT-1: вызвать ReviewService.review с любым diff и проверить, что возвращаемая структура содержит ключи summary, risks (≤3), checks. Сейчас возвращается только {comment}.
- API-1: отправить на POST /api/reviews JSON с diff длиной 21000 символов; ожидать HTTP 413. Сейчас эндпойнт возвращает 200 и проксирует вызов LLM.
