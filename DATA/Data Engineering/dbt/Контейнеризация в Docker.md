Создаем Dockerfile в корне проекта. Пример содержимого:
```bash
# Используем официальный образ Python как базу
FROM python:3.11-slim

# Устанавливаем рабочую директорию
WORKDIR /usr/app

# Копируем проект dbt
COPY . /usr/app

# Устанавливаем зависимости системы (например, для PostgreSQL)
RUN apt-get update && apt-get install -y \
    gcc \
    libpq-dev \
    git \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Устанавливаем dbt и адаптер PostgreSQL
RUN pip install --no-cache-dir --upgrade pip \
    && pip install dbt-core dbt-postgres

# Устанавливаем зависимости проекта (если есть requirements.txt)
RUN if [ -f requirements.txt ]; then pip install -r requirements.txt; fi

# Устанавливаем зависимости dbt (если есть packages.yml)
RUN dbt deps --profiles-dir /usr/app/profiles

# Указываем профили dbt
ENV DBT_PROFILES_DIR=/usr/app/profiles

# Указываем точку входа
# ENTRYPOINT ["dbt"]
```
**Объяснение**:
- FROM python:3.11-slim: Использует лёгкий образ Python 3.11.
- COPY . /usr/app: Копирует весь проект dbt в контейнер.
- apt-get: Устанавливает зависимости для PostgreSQL (если адаптер требует).
- pip install: Устанавливает dbt и адаптер.
- dbt deps: Устанавливает пакеты, если они указаны в packages.yml.
- ENV DBT_PROFILES_DIR: Указывает, где искать profiles.yml.
- ENTRYPOINT ["dbt"]: Позволяет запускать dbt-команды напрямую.

Переходим в корневую папку проекта dbt и создаем образ:
```
docker build -t my-dbt-project .
```

Для проверки списка образов используем команду
```
docker images
```

Автоматизировать сборку образа можно с помощью скрипта под названием, например, rebuild_dbt.ps1 со следующим содержимым:
```
docker build -t my-dbt-project .
Write-Host "Docker image my-dbt-project rebuilt successfully."
```

Запуск скрипта осуществляется командой
```
.\rebuild_dbt.ps1
```

Для запуска команд dbt:

debug
```
docker run --rm -it --env-file .env --network host my-dbt-project dbt debug
```

compile
```
docker run --rm -it --env-file .env --network host my-dbt-project dbt compile
```

run
```
docker run --rm -it --env-file .env --network host my-dbt-project dbt run
```

