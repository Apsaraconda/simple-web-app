# simple-web-app Backend + Nginx Reverse Proxy

Простое веб-приложение, где `nginx` выступает в роли reverse proxy, проксируя запросы к backend-сервису на Python.

## Как запустить

1. Убедитесь, что у вас установлены:
    - Docker
    - Docker Compose

2. Клонируйте репозиторий (если не сделано):
```bash
git clone https://github.com/Apsaraconda/simple-web-app.git
```

3. Запустите приложение:
```bash
sudo docker compose up -d
```
## Проверка работоспособности

После запуска выполните:
```bash
curl http://localhost
```

Ожидаемый ответ:
```txt
Hello from Effective Mobile!
```

## Архитектура
```txt
[curl http://localhost]
    🠃
[docker-контейнер `em-nginx` (порт 80)]
    🠃
[docker-контейнер `em-backend` (порт 8080)]
```
- **Nginx** принимает HTTP-запросы на порт 80 хоста.
- Все запросы на `/` проксируются к сервису `backend` по внутренней Docker-сети.
- **Backend** работает на Python `http.server`, слушает порт 8080, но не публикует его наружу.
- Взаимодействие происходит через отдельную Docker-сеть `em-net`.
- Только порт 80 (nginx) доступен снаружи.

## Использованные технологии

- Docker-образ `python:3.13-alpine` для компиляции
- Минималистичный образ `alpine:3.23` в качестве рабочей среды сервера em-backend
- Docker-образ `nginx:1.25-alpine`
- Docker & Docker Compose
- Безопасность: 
    - сервер работает от non-root пользователя appuser, 
    - реализован healthcheck, 
    - минимальный образ `alpine:3.23`

## Сеть

- Backend недоступен с хоста напрямую.
- Взаимодействие — только внутри изолированной сети Docker `em-net`.
- Никаких других портов наружу не проброшено, кроме 80.

## Структура проекта

```txt
├── backend/
│ ├── Dockerfile
│ └── app.py
├── nginx/
│ └── nginx.conf
├── docker-compose.yml
└── README.md
```
* Backend на Python, слушает 8080, не публикует порт
* Nginx проксирует / на backend по имени сервиса
* Конфиг nginx внешний, подключен через volume
* Docker Compose:
    * отдельная сеть
    * проброшен только порт 80
    * проименованные контейнеры
    * реализован healthcheck

*Примечание: если `curl` возвращает пустоту — дождитесь полной инициализации (`healthcheck` может занять 10–15 сек).*

## Безопасность: 
* non-root пользователь, 
* минималистичные образы, 
* без `latest` версий
