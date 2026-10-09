# Job Search Agent — Test Documentation

## Цель тестирования

Проверить, что Job Search Agent корректно:
- хранит и обновляет вакансии;
- фильтрует их по профилю пользователя;
- классифицирует роль по title;
- не показывает повторно уже обработанные вакансии;
- не смешивает состояние разных пользователей;
- переживает отказ отдельного источника или AI;
- корректно работает после архитектурных изменений.

## Основные наборы проверок

### Matcher / taxonomy

Проверяются сценарии:
- exact title match;
- synonym match;
- close profession;
- unrelated profession;
- negative profession rules;
- seniority mismatch;
- score thresholds;
- grading A / B / C.

Регрессионные примеры:
- QA Tester Entry Level → QA / Testing;
- Senior QA Engineer для junior-профиля → понижение релевантности;
- Software Developer → не должен классифицироваться как QA;
- Payroll/Legal/Sales роли не должны становиться Marketing/Data из-за случайных слов в description.

### Geography / language

Проверяются:
- Russia → RU market;
- Russia → International;
- Outside Russia → RU market;
- Outside Russia → International;
- RU + International;
- Russian / English language combinations;
- Worldwide eligibility;
- country-restricted vacancies.

### User history

Проверяется:
- User A увидел вакансию → повторно не получает её;
- User B всё ещё может получить ту же вакансию;
- Save / Applied / Skip сохраняются;
- after-filter top results добираются из следующих кандидатов.

### Vacancy database / ingestion

Проверяется:
- запись новой вакансии;
- update существующей;
- expires_at;
- deduplication;
- одновременное чтение поиска и обновление collector;
- повторный ingestion без появления дублей.

### Telegram whitelist

Проверяется:
- загрузка telegram_sources.json;
- enabled/disabled source;
- market/language metadata;
- category filtering;
- чтение публичного канала;
- нормализация публикации;
- сохранение message_id / URL;
- повторный запуск без дублей.

### External services / fallback

Проверяется:
- один источник недоступен;
- Gemini недоступен;
- AI не вызывается для детерминированно классифицированной вакансии;
- AI используется только для ambiguous case;
- ошибка AI не должна ронять collector или бота.

## Выполненные regression/smoke проверки

В review-сборке v4:
- новый v4 test suite: 34 теста;
- 33 passed;
- 1 skipped в review-среде из-за отсутствия python-telegram-bot;
- matcher v4.1 regression — passed;
- architecture v3 — passed;
- multi-user v3 — passed;
- user jobs smoke — passed;
- salary / legitimacy smoke — passed;
- compileall — passed.

Live integrations требуют отдельного server smoke.

## Примеры smoke-flow

### Search flow
1. Создать/загрузить профиль.
2. Запустить поиск.
3. Убедиться, что collector не запускается внутри user search.
4. Проверить фильтрацию history.
5. Проверить matcher.
6. Проверить score ≥ 50.
7. Проверить сортировку.
8. Проверить выдачу максимум 6 вакансий.
9. Проверить mark_job_shown только после успешной отправки.

### Multi-user flow
1. User A выполняет поиск.
2. User A получает vacancy X.
3. User A выполняет повторный поиск — vacancy X не возвращается.
4. User B выполняет поиск — vacancy X может быть показана.

## Известные ограничения

- live Telegram collector зависит от доступности публичных t.me страниц;
- часть переформулированных дублей может не объединяться;
- полноценный universal web verification остаётся отдельным этапом развития;
- production-ready статус нельзя считать подтверждённым без live integration smoke.
