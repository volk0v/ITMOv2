# Integration-проверки

| Связь компонентов                            | Что может сломаться      | Как воспроизводим                  | Ожидаемый результат                          | Evidence                        |
| -------------------------------------------- | ------------------------ | ---------------------------------- | -------------------------------------------- | ------------------------------- |
| FastAPI -> ReviewService -> LLMClient        | Нарушение OUT-1 формата  | POST /api/reviews с валидным diff  | JSON содержит summary, risks<=3, checks      | CASE.md OUT-1, TRAINING_PR.diff |
| FastAPI -> Validator                         | Нет 413 при длинном diff | POST /api/reviews с 21000 символов | HTTP 413                                     | CASE.md API-1                   |
| ReviewService -> SecretRedactor -> LLMClient | Утечка секретов          | diff содержит token=abcd           | В запросе к LLM секрет заменён на [REDACTED] | CASE.md SEC-1                   |
| LLMClient (timeout)                          | Подвисание без ответа    | стаб LLM задерживает >10с          | Контролируемая ошибка, без 500               | CASE.md REL-1                   |

## Как использовали AI

- Строка в [`prompts.md`](prompts.md): P1-03.
- Что проверили и исправили сами: дополнили негативными сценариями (413, timeout) и маскировкой секретов.
