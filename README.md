# Solution

## Звіт

1. Першу проблему побачив в логах при запуску docker - compose:
app-1  |   File "/app/main.py", line 4, in <module>

app-1  |     ENV = os.environ["APP_ENV"]

app-1  |           ~~~~~~~~~~^^^^^^^^^^^

app-1  |   File "<frozen os>", line 714, in __getitem__

app-1  | KeyError: 'APP_ENV'

    Після дослідження додатку замітив, що в файлі compose.yaml використовується пермінна APP_MODE, тому рішенням було замінити цю перемінну в /app/main.py
    ENV = os.environ["APP_ENV"] - > ENV = os.environ["APP_MODE"]
2. Наступна проблему також показали логи:
app-1    | [2026-07-14 09:24:42 +0000] [1] [ERROR] Worker (pid:7) exited with code 4

app-1    | [2026-07-14 09:24:42 +0000] [1] [ERROR] Shutting down: Master

app-1    | [2026-07-14 09:24:42 +0000] [1] [ERROR] Reason: App failed to load.

    Проаналізував головний Dockerfile і побачив, що в команді запуску Gunicorn неправильно вказана папка для Flask-додатку. Замінив її
    CMD ["gunicorn", "-b", "0.0.0.0:8080", "main:application"] -> CMD ["gunicorn", "-b", "0.0.0.0:8080", "main:app"]
3. Помилок з запуском веб - застосунку більше не спостерігав, але при відкритті http://localhost/ , отримував помилку Bad Gateway
   
   Помилка була з портами в файлі compose.yaml. Сaddy використовує в роботі стандартний http порт, тому замінив порт апки 8080 на 80
   
   - traefik.http.services.caddy.loadbalancer.server.port=8080 - > - traefik.http.services.caddy.loadbalancer.server.port=80

   А сам застосунок використовує порт 8080 і роутиться через traefic, тому замінив для нього порт також
   ["CMD", "curl", "-f", "http://localhost:80/api/health"] -> ["CMD", "curl", "-f", "http://localhost:8080/api/health"]

## Запуск

```bash
docker compose up --build
```

## Очікуваний результат

Після виправлення всіх проблем:

- `http://localhost/` => HTML-сторінка
- `http://localhost/api/` => JSON
- `http://localhost/api/health` => `{"status":"OK"}`

Скріншоти в корені проекту
