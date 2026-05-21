# Backlog

Поток работы — Trunk Based Development: ветка живёт 1-2 дня, один PR в `main` за шаг. Подробнее — `AGENTS.md` §9 «Флоу разработки сервиса».

## Todo

### Шаг 1 — API-контракт и README

- [ ] Написать OpenAPI 3.x / AsyncAPI 3.0 спецификацию (`api-specification/`)
- [ ] Зафиксировать «Карту режимов отказа» в README (5xx с `error.code` для синхронного API, события для асинхронного)
- [ ] Создать README по структуре из `AGENTS.md` §«Структура постановки задачи в README»
- [ ] Зафиксировать в `devlog/01-api-contract.md`
- [ ] Merge в main

### Шаг 2.0 — Каркас компонентных тестов

Раннер godog (или эквивалент) запускается **в Docker**, не с хоста. Цель — обвязка для Шага 2: чтобы sonnet/оператор писал `.feature`-файлы под готовые степы, а не выдумывал их в воздухе.

- [ ] Создать `component-tests/` как отдельный модуль (раннер + степ-фреймворк)
- [ ] `Dockerfile.runtime` для раннера (godog в контейнере)
- [ ] `docker-compose.test.yml` — SUT + раннер + bind-mount под фикстуры режимов отказа
- [ ] Placeholder-сервис (отдаёт `501 Not Implemented` для всех эндпоинтов, `200` на `/health`) — Шаг 2 пишет красные тесты под него
- [ ] Базовые степы: HTTP-вызов, проверка статуса/заголовков/JSON-полей, инфраструктура для режимов отказа
- [ ] Smoke-сценарий end-to-end через `scripts/run-tests.sh`
- [ ] Зафиксировать в `devlog/02.0-component-tests-template.md`
- [ ] Merge в main

### Шаг 2 — Компонентные тесты (Gherkin)

Процедура — `skills/component-tests/SKILL.md`. Формула: `N = N_эндпоинтов + Σ режимов_отказа_интеграции`. Контракт первичен: режимы отказа берутся из OpenAPI/README, не выводятся из кода.

- [ ] Сгенерировать `.feature`-файлы по SKILL — один файл на ресурс/тему
- [ ] Прогнать чек-лист SKILL: число файлов = число ресурсов; число сценариев = по формуле; каждый failure-сценарий привязан к одному эндпоинту
- [ ] Зафиксировать в `devlog/02-gherkin.md`
- [ ] Merge в main

### Шаг 3 — Реализация (slice-by-slice)

Реализация идёт **вертикальными срезами** (slice), не плоским списком модулей. Каждый slice = один эндпоинт (или одно событие) от ингресс-адаптера до I/O. Цикл одного slice'а:

1. **Дизайн** на ветке `feat/design-<slice>` — opus на скилле `skills/program-design/SKILL.md`. Выход: пакет `docs/design/<service>/slices/NN-<slice>.md` + дополнения `messages.md`/`contracts-graph.md`/`infrastructure.md` + тикет с DoD и хендофф-чеклистом в `docs/design/<service>/backlog.md`. PR в main, последняя строка чеклиста — аппрув оператора с GitHub-handle и датой. **Мерж дизайн-PR = аппрув**.
2. **Реализация** на ветке `feat/slice-<slice>` — sonnet на скилле `skills/program-implementation/SKILL.md`. Один тикет = один slice = одна ветка = один PR.
3. **Devlog** `docs/design/<service>/devlog.md` дополняется блоком после каждого слайса.

- [ ] Создать `docs/architecture.md` (иерархия модулей + блок-схема entry point) — заполняется по мере добавления слайсов
- [ ] Инициализировать модуль и структуру директорий (`cmd/`, `internal/slice/<slice>/`, `internal/db/migrations/`, `internal/app/`)
- [ ] Спроектировать первый slice (см. шаги выше) → реализовать → смержить
- [ ] Повторить для каждого slice'а
- [ ] Все компонентные сценарии зелёные
- [ ] Зафиксировать в `devlog/03-implementation.md`
- [ ] Merge финального slice'а

### Шаг 4 — CI на PR

Запускается **после Шагов 1-3** (контракт, тесты, реализация). До этого CI проверял бы только smoke от placeholder'а.

- [ ] Валидация OpenAPI/AsyncAPI на PR (GitHub Actions + `redocly/cli` или эквивалент)
- [ ] Сборка сервиса на PR
- [ ] Прогон компонентных тестов на PR (профиль `healthy` всегда; профили отказов — по применимости)
- [ ] Зафиксировать в `devlog/04-ci.md`
- [ ] Merge в main

## Done

- [x] Шаг 0 — Репозиторий, AGENTS.md, CLAUDE.md, intent
