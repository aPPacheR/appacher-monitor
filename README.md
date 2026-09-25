# Appacher Monitor — мониторинг доступности сайтов на базе Uptime Kuma

Развёртывание сервиса мониторинга [Uptime Kuma](https://github.com/louislam/uptime-kuma) в Docker с автоматическим обратным прокси (nginx-proxy) и автоматическим выпуском/обновлением SSL-сертификатов Let's Encrypt (acme-companion).

## Архитектура

Три контейнера в общей Docker-сети `proxy-net`:
| Контейнер | Образ | Назначение |
|---|---|---|
| `nginx-proxy` | `nginxproxy/nginx-proxy` | Принимает весь входящий трафик на 80/443, проксирует к нужному контейнеру по домену |
| `acme-companion` | `nginxproxy/acme-companion` | Автоматически получает и обновляет SSL-сертификаты Let's Encrypt |
| `uptime-kuma` | `louislam/uptime-kuma:2` | Само приложение мониторинга |

Данные каждого сервиса хранятся в отдельных Docker volumes и переживают перезапуск/обновление контейнеров.

## Системные требования

- ОС сервера: Ubuntu 22.04/24.04 (тестировалось на Ubuntu)
- Docker Engine 24+ и Docker Compose plugin
- Домен, A-запись которого указывает на IP сервера
- Открытые порты 80 и 443 (для HTTP-валидации Let's Encrypt и HTTPS)

## Быстрый старт

1. Клонировать репозиторий:
```bash
   git clone https://github.com/appacher/appacher-monitor.git
   cd appacher-monitor
```

2. Создать файл `.env` на основе примера:
```bash
   cp .env.example .env
```
   и указать свой домен и email:

DOMAIN=your-domain.ru
LETSENCRYPT_EMAIL=your-email@example.com


3. Запустить:
```bash
   docker compose up -d
```

4. Проверить статус контейнеров:
```bash
   docker compose ps
```

5. Открыть `https://your-domain.ru` — сертификат выпустится автоматически при первом запуске (может занять 1-2 минуты).

## Остановка и обновление

Остановить всё:
```bash
docker compose down
```

Обновить образы до последних версий:
```bash
docker compose pull
docker compose up -d
```

## Переменные окружения (.env)

| Переменная | Описание |
|---|---|
| `DOMAIN` | Домен, по которому будет доступно приложение |
| `LETSENCRYPT_EMAIL` | Email для уведомлений Let's Encrypt |
