

# URL Shortener

**URL Shortener** — это сервис для сокращения длинных ссылок с возможностью задания собственного алиаса. Проект реализован на Go, использует SQLite для хранения данных и предоставляет HTTP API.

## Возможности

- Сокращение длинных URL с генерацией случайного или пользовательского алиаса.
- Редирект по короткой ссылке.
- Basic Auth для защиты эндпоинта создания ссылок.
- Хранение данных в SQLite.
- Гибкая настройка через YAML-конфиг и переменные окружения.

## Быстрый старт

### Требования

- Go 1.24+
- SQLite (библиотека подключается автоматически)
- Make (опционально, для удобного запуска)

### Установка зависимостей

```sh
go mod download
```

### Конфигурация

1. Создайте файл `.env` в корне проекта (если требуется).
2. Укажите путь к конфигу через переменную окружения:

```sh
export CONFIG_PATH=./config/local.yaml
```

Пример содержимого `config/local.yaml`:
```yaml
env: local
storage_path: ./storage/storage.db
http_server:
  port: 8080
  timeout: 4s
  idle_timeout: 60s
  user: admin
  password: 123456
```

### Запуск

```sh
make run
```
или вручную:
```sh
go run cmd/url-shortner/main.go
```

## API

### 1. Сокращение ссылки

**POST** `/url`

- Требует Basic Auth (логин и пароль из конфига).
- Content-Type: `application/json`

#### Тело запроса:
```json
{
  "url": "https://example.com/very/long/url",
  "alias": "myalias" // не обязательно, если не указан — будет сгенерирован случайный
}
```

#### Пример успешного ответа:
```json
{
  "status": "ok",
  "alias": "myalias"
}
```

#### Ошибки:
- Некорректный запрос: `{"status":"error","error":"invalid request"}`
- URL уже существует: `{"status":"error","error":"url already exists"}`

### 2. Редирект по короткой ссылке

**GET** `/{alias}`

- Перенаправляет на исходный URL (HTTP 302 Found).
- Если алиас не найден — возвращает JSON с ошибкой:
```json
{"status":"error","error":"not found"}
```

## Переменные окружения

- `CONFIG_PATH` — путь к YAML-файлу конфигурации (обязательно).

## Структура проекта

- `cmd/url-shortner/main.go` — точка входа.
- `internal/` — основная бизнес-логика, обработчики, работа с хранилищем.
- `config/` — примеры конфигов.
- `storage/` — база данных SQLite.

## Зависимости

- [go-chi/chi](https://github.com/go-chi/chi) — роутер.
- [go-chi/render](https://github.com/go-chi/render) — рендеринг JSON.
- [go-playground/validator](https://github.com/go-playground/validator) — валидация.
- [mattn/go-sqlite3](https://github.com/mattn/go-sqlite3) — драйвер SQLite.
- [ilyakaznacheev/cleanenv](https://github.com/ilyakaznacheev/cleanenv) — загрузка конфигов.
- [joho/godotenv](https://github.com/joho/godotenv) — поддержка .env.

