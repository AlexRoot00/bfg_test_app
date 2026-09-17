
## Архитектура
Проект состоит из двух контейнеров:
* **app** — сервис на Python 3.14 + FastAPI + Uvicorn;
* **db** — PostgreSQL 18 Alpine.

Для обоих сервисов выбран Alpine Linux с целью минимизировать размер образов и количество устанавливаемых зависимостей.
Приложение подключается к PostgreSQL по имени сервиса `db` во внутренней Docker-сети.
PostgreSQL запускается первым. Контейнер приложения начинает работу только после успешного `healthcheck` базы данных.

## Решения которые пришлось реализовать:
gthtp
### Docker
Для приложения используется:
```text
python:3.14.7-alpine3.24
```

`psycopg2` в текущем окружении устанавливается из исходников. Необходимые для сборки компилятор и dev-зависимости PostgreSQL устанавливаются только на этапе сборки образа и после установки Python-зависимостей удаляются.
`libpq` необходима приложению во время выполнения, поэтому эта runtime-зависимость остаётся в итоговом образе.
Приложение запускается от непривилегированного пользователя `app`.

### PostgreSQL
Используется:
```text
postgres:17-alpine
```
Данные PostgreSQL хранятся непосредственно на хосте:
```text
data/postgres/
```
Это позволяет сохранять данные при пересоздании контейнера.

### Healthchecks
Для PostgreSQL используется стандартная проверка:
```text
pg_isready
```
Для приложения используется endpoint:
```text
GET /healthz
```
Он проверяет жизнеспособность приложения и не обращается к базе данных.
Для проверки готовности приложения предусмотрен:
```text
GET /readyz
```
Этот endpoint дополнительно проверяет доступность PostgreSQL.
## Запуск
Из директории, содержащей `compose.yml`:
```bash
docker compose -f compose.yml up --build
```
Запуск в фоне:
```bash
docker compose -f compose.yml up --build -d
```
Проверка состояния контейнеров:
```bash
docker compose -f compose.yml ps
```
Просмотр логов:
```bash
docker compose -f compose.yml logs -f
```
## Проверка API
Проверка жизнеспособности приложения:

```bash
curl http://localhost:8000/healthz
```

Проверка готовности приложения и подключения к БД:

```bash
curl http://localhost:8000/readyz
```

## Остановка

```bash
docker compose -f compose.yml down
```

Локальные данные PostgreSQL при этом сохраняются в `./data/postgres`.

## Конфигурация

Основные параметры приложения и PostgreSQL задаются непосредственно в `compose.yml`:

```yaml
POSTGRES_HOST: db
POSTGRES_DB: notes
POSTGRES_USER: notes
POSTGRES_PASSWORD: change-me-please

APP_PORT: 8000
LOG_LEVEL: INFO
SHUTDOWN_DELAY_SECONDS: 3
```

