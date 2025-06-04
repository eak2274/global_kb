# Chproxy: HTTP-прокси и балансировщик нагрузки для ClickHouse

## Что такое chproxy?

**Chproxy** — это open-source инструмент, написанный на Go, который выступает в роли HTTP-прок [HTTP-прокси](https://www.chproxy.org/) и балансировщика нагрузки для базы данных ClickHouse. Он предназначен для упрощения взаимодействия между клиентами (приложениями, аналитическими инструментами) и кластерами ClickHouse в распределенной инфраструктуре, обеспечивая:

- **Балансировку нагрузки**: равномерное распределение запросов (SELECT и INSERT) между узлами кластера.
- **Безопасность**: управление доступом и сокрытие учетных данных ClickHouse.
- **Оптимизацию производительности**: кэширование ответов и ограничение нагрузки.
- **Гибкость**: маршрутизация запросов к разным кластерам в зависимости от пользователя или типа запроса.

Chproxy не является официальной частью ClickHouse, а представляет собой сторонний проект сообщества, активно используемый в продакшене для масштабируемых систем.[](https://www.chproxy.org/)[](https://github.com/ContentSquare/chproxy)

---

## Основные возможности chproxy

Chproxy предоставляет широкий набор функций, которые делают его полезным для распределенной инфраструктуры ClickHouse:

1. **Маршрутизация запросов**:
    
    - Может направлять запросы к разным кластерам ClickHouse в зависимости от пользователя. Например, запросы от `appserver` могут идти в кластер `stats-raw`, а от `reportserver` — в `stats-aggregate`.[](https://www.chproxy.org/)
    - Поддерживает маршрутизацию INSERT-запросов напрямую к шардам (per-shard tables), минуя распределенные таблицы, что снижает нагрузку на узлы.[](https://golangexample.com/clickhouse-http-proxy-and-load-balancer/)
2. **Балансировка нагрузки**:
    
    - Распределяет запросы между узлами кластера в режиме `round-robin` или `least-loaded`, пропуская недоступные или нездоровые узлы.[](https://www.chproxy.org/)[](https://www.chproxy.org/getting_started/)
    - Позволяет распределять SELECT-запросы по всем шардам с одинаковыми распределенными таблицами, предотвращая перегрузку одного узла.[](https://golangexample.com/clickhouse-http-proxy-and-load-balancer/)
3. **Управление пользователями**:
    
    - Сопоставляет вක: **Маппинг пользователей**: позволяет скрывать реальные учетные данные ClickHouse, маппинг нескольких внешних пользователей на одного пользователя ClickHouse.[](https://www.chproxy.org/)
    - Ограничивает доступ по IP/IP-маске для HTTP/HTTPS.[](https://www.chproxy.org/)
4. **Ограничение нагрузки**:
    
    - Ограничивает количество одновременных запросов, длительность запросов и частоту запросов для каждого пользователя. Превышающие лимиты запросы завершаются через `KILL QUERY`.[](https://www.chproxy.org/)
    - Пример: ограничение до 6 одновременных запросов и максимум 1 минута на выполнение.[](https://golangexample.com/clickhouse-http-proxy-and-load-balancer/)
5. **Кэширование**:
    
    - Кხ: Кэширует ответы на SELECT-запросы в локальном (на файловой системе) или распределенном (Redis) кэше.[](https://www.chproxy.org/configuration/caching/)
    - Поддерживает пространства имен кэша (`cache_namespace`) для мгновенного сброса кэша.[](https://www.chproxy.org/configuration/caching/)
    - Возвращает заголовок `X-Cache` (`HIT`, `MISS`, `N/A`) для отслеживания кэширования.[](https://www.chproxy.org/configuration/caching/)
6. **Безопасность**:
    
    - Поддерживает HTTP и HTTPS (с автоматическим обновлением SSL-сертификатов через Let’s Encrypt).[](https://content.clickhouse.tech/docs/en/interfaces/third-party/proxy/)
    - Добавляет в заголовок `User-Agent` информацию о пользователе и адресе для логов (`system.query_log.http_user_agent`).[](https://www.chproxy.org/)
7. **Мониторинг**:
    
    - Экспортирует метрики в формате Prometheus для мониторинга производительности и ошибок.[](https://www.chproxy.org/)
8. **Гибкость конфигурации**:
    
    - Конфигурация обновляется без перезапуска через сигнал `SIGHUP`.[](https://www.chproxy.org/)
    - Поддерживает настройку кэша, сетевых ограничений и маршрутизации через YAML-файл.[](https://www.chproxy.org/getting_started/)

---

## Пример использования в распределенной инфраструктуре

Chproxy особенно полезен в сценариях, где ClickHouse работает в кластере с несколькими узлами (шардами и репликами). Вот типичный пример конфигурации:

### Сценарий: Балансировка INSERT и SELECT

- **Проблема**: INSERT-запросы через распределенную таблицу на одном узле увеличивают нагрузку на него, так как узел парсит и перенаправляет данные на шарды. SELECT-запросы также могут перегружать один узел.
- **Решение с chproxy**:
    - Настройка chproxy для маршрутизации INSERT-запросов напрямую к шардам, минуя распределенную таблицу.
    - Распределение SELECT-запросов по всем узлам с одинаковыми распределенными таблицами.

#### Пример конфигурации (для INSERT):

```yaml
server:
  http:
    listen_addr: ":9090"
    allowed_networks: ["10.10.1.0/24"] # Сети серверов приложений
users:
  - name: "insert"
    to_cluster: "stats-raw"
    to_user: "default"
clusters:
  - name: "stats-raw"
    nodes: [
      "10.10.10.1:8123",
      "10.10.10.2:8123",
      "10.10.10.3:8123",
      "10.10.10.4:8123"
    ]
```

[](https://golangexample.com/clickhouse-http-proxy-and-load-balancer/)

#### Пример конфигурации (для SELECT):

```yaml
server:
  http:
    listen_addr: ":9090"
    allowed_networks: ["10.10.2.0/24"] # Сети отчетных серверов
users:
  - name: "report"
    to_cluster: "stats-aggregate"
    to_user: "readonly"
    max_concurrent_queries: 6
    max_execution_time: 1m
clusters:
  - name: "stats-aggregate"
    nodes: [
      "10.10.20.1:8123",
      "10.10.20.2:8123"
    ]
    users:
      - name: "readonly"
        password: "****"
```

[](https://golangexample.com/clickhouse-http-proxy-and-load-balancer/)

### Результат:

- INSERT-запросы распределяются по шардам, снижая нагрузку на один узел.
- SELECT-запросы равномерно распределяются по всем узлам, предотвращая перегрузку.
- Ограничения (например, 6 одновременных запросов) защищают кластер от перегрузки.

---

## Установка и запуск

1. **Скачивание**:
    
    - Скачайте последнюю версию chproxy с [GitHub Releases](https://github.com/Vertamedia/chproxy/releases).
    - Пример для Linux:
        
        ```bash
        mkdir -p /data/chproxy
        cd /data/chproxy
        wget https://github.com/Vertamedia/chproxy/releases/download/v1.14.0/chproxy-linux-amd64-v1.14.0.tar.gz
        tar -xzvf chproxy-*.gz
        ```
        
    
    [](https://www.programmersought.com/article/53536651997/)
    
2. **Конфигурация**:
    
    - Создайте файл `config.yml` с настройками, как в примерах выше.
    - Для отключения проверок безопасности (не рекомендуется) добавьте `hack_me_please: true`.[](https://www.programmersought.com/article/53536651997/)
3. **Запуск**:
    
    - Выполните команду:
        
        ```bash
        ./chproxy -config /data/chproxy/config.yml
        ```
        
    - Или используйте Docker:
        
        ```bash
        docker run -d -v $(pwd)/config.yml:/opt/config.yml -p 9090:9090 chproxy-test -config /opt/config.yml
        ```
        
    
    [](https://morioh.com/a/3792b889c5be/chproxy-open-source-clickhouse-http-proxy-and-load-balancer)
    
4. **Проверка**:
    
    - Отправьте тестовый запрос:
        
        ```bash
        echo "SELECT * FROM tutorial.views LIMIT 1000 FORMAT Pretty" | curl 'http://default:password@localhost:9090/' --data-binary @-
        ```
        
    
    [](https://www.chproxy.org/getting_started/)
    
    - Проверьте логи ClickHouse или метрики Prometheus для подтверждения балансировки.

---

## Преимущества в распределенной инфраструктуре

- **Снижение нагрузки**: Распределение запросов предотвращает перегрузку отдельных узлов.[](https://www.chproxy.org/getting_started/)
- **Масштабируемость**: Легко добавлять новые узлы в кластер, chproxy автоматически распределяет нагрузку.[](https://medium.com/walmartglobaltech/interactive-analytics-at-scale-c8e32dd0e910)
- **Безопасность**: Скрытие учетных данных и ограничение доступа по IP.[](https://www.chproxy.org/)
- **Производительность**: Кэширование SELECT-запросов снижает количество запросов к ClickHouse.[](https://www.chproxy.org/configuration/caching/)
- **Надежность**: Пропуск нездоровых узлов и поддержка HTTPS.[](https://content.clickhouse.tech/docs/en/interfaces/third-party/proxy/)

---

## Ограничения

- **Только HTTP**: Chproxy не поддерживает нативный протокол ClickHouse, что может быть менее эффективно для массовых INSERT-запросов (бенчмарки показывают, что нативный протокол быстрее, но chproxy использует менее 20% CPU при 1 Гбит/с INSERT-трафика).[](https://www.chproxy.org/faq/)
- **Кэширование только для SELECT**: INSERT, UPDATE и DELETE не кэшируются.[](https://www.chproxy.org/configuration/caching/)
- **Зависимость от конфигурации**: Неправильная настройка (например, слишком большой кэш) может привести к увеличению потребления памяти.[](https://www.chproxy.org/configuration/caching/)
- **Ограничения распределенных таблиц**: Прямое использование распределенных таблиц без chproxy может привести к неравномерной нагрузке и проблемам при масштабировании (например, при добавлении шардов).[](https://www.programmersought.com/article/53536651997/)

---

## Альтернативы chproxy

1. **HAProxy**:
    - TCP-балансировщик, подходит для нативного протокола ClickHouse.
    - Ограничение: не понимает протокол ClickHouse, не может разделять запросы внутри одной сессии. Требуется закрытие соединения после каждого запроса (`idle_connection_timeout=0`).[](https://kb.altinity.com/altinity-kb-setup-and-maintenance/load-balancers/)
2. **Nginx**:
    - Может работать как TCP- или HTTP-балансировщик.
    - Ограничения аналогичны HAProxy, плюс меньшая гибкость для ClickHouse-специфичных функций.[](https://kb.altinity.com/altinity-kb-setup-and-maintenance/load-balancers/)
3. **KittenHouse**:
    - Локальный прокси для буферизации INSERT-запросов.
    - Поддерживает маршрутизацию по таблицам и проверку здоровья узлов, но менее функционален для SELECT-запросов.[](https://content.clickhouse.tech/docs/en/interfaces/third-party/proxy/)
4. **ClickHouse-Bulk**:
    - Простой коллектор INSERT-запросов.
    - Поддерживает группировку запросов и отправку по расписанию, но не подходит для сложной маршрутизации или SELECT.[](https://content.clickhouse.tech/docs/en/interfaces/third-party/proxy/)
5. **Распределенные таблицы ClickHouse**:
    - Встроенный механизм ClickHouse для распределения данных.
    - Проблема: высокая нагрузка на узел с распределенной таблицей, особенно при INSERT.[](https://golangexample.com/clickhouse-http-proxy-and-load-balancer/)

---

## Рекомендации для продакшена

- **Используйте HTTPS**: Настройте HTTPS с Let’s Encrypt для безопасного доступа, особенно если кластер доступен из внешних сетей.[](https://golangexample.com/clickhouse-http-proxy-and-load-balancer/)
- **Мониторинг**: Подключите метрики chproxy к Prometheus/Grafana для отслеживания нагрузки и ошибок.[](https://www.chproxy.org/)
- **Тестирование**: Перед продакшеном протестируйте конфигурацию на небольшой нагрузке, особенно кэширование и ограничения.[](https://www.chproxy.org/configuration/caching/)
- **Ограничения по пользователям**: Настройте лимиты (`max_concurrent_queries`, `max_execution_time`) для предотвращения перегрузки.[](https://golangexample.com/clickhouse-http-proxy-and-load-balancer/)
- **Проверка шардов**: Убедитесь, что все узлы кластера имеют одинаковые распределенные таблицы для SELECT-запросов.[](https://www.chproxy.org/getting_started/)
- **Регулярное обновление**: Следите за новыми версиями chproxy на [GitHub](https://github.com/Vertamedia/chproxy) для исправления багов и новых функций.[](https://github.com/ContentSquare/chproxy)

---

## Заключение

**Chproxy** — мощный инструмент для балансировки нагрузки и управления доступом в распределенной инфраструктуре ClickHouse. Он решает ключевые проблемы, такие как неравномерная нагрузка на узлы, безопасность учетных данных и перегрузка кластера, особенно в сценариях с большим количеством INSERT и SELECT-запросов. По сравнению с альтернативами (HAProxy, Nginx, встроенные распределенные таблицы), chproxy предлагает более гибкую маршрутизацию, кэширование и ограничения, специфичные для ClickHouse, что делает его популярным выбором для продакшен-систем.[](https://www.chproxy.org/)[](https://golangexample.com/clickhouse-http-proxy-and-load-balancer/)[](https://content.clickhouse.tech/docs/en/interfaces/third-party/proxy/)

Для вашего случая, если вы строите распределенную инфраструктуру ClickHouse, chproxy идеально подойдет для:

- Балансировки INSERT-запросов по шардам.
- Распределения SELECT-запросов для аналитики.
- Обеспечения безопасности и ограничения нагрузки.

Если у вас есть конкретные детали вашей инфраструктуры (количество узлов, тип запросов, объем данных), я могу предложить более точную конфигурацию или примеры.

# Docker-образы для chproxy

## Обзор

**Chproxy** — это HTTP-прокси и балансировщик нагрузки для ClickHouse, и для него доступны готовые Docker-образы в публичном репозитории на **Docker Hub**. Эти образы упрощают развертывание и управление chproxy в распределенной инфраструктуре, особенно в контейнеризированных средах. Образы поддерживаются сообществом, а начиная с версии 1.17.1, chproxy следует семантическому версионированию, что минимизирует риски breaking changes между минорными версиями.[](https://www.chproxy.org/changelog/)

- **Репозиторий**: [contentsquareplatform/chproxy](https://hub.docker.com/r/contentsquareplatform/chproxy)
- **Поддерживаемые архитектуры**: В основном x86_64. Обратите внимание, что образы для ARM64 и ARM64v8 недоступны для некоторых версий.[](https://www.chproxy.org/changelog/)
- **Текущая версия (на май 2025)**: Последние версии можно проверить на Docker Hub, например, `v1.20.0` или выше.

---

# Docker-образы для chproxy

## Обзор

**Chproxy** — это HTTP-прокси и балансировщик нагрузки для ClickHouse, и для него доступны готовые Docker-образы в публичном репозитории на **Docker Hub**. Эти образы упрощают развертывание и управление chproxy в распределенной инфраструктуре, особенно в контейнеризированных средах. Образы поддерживаются сообществом, а начиная с версии 1.17.1, chproxy следует семантическому версионированию, что минимизирует риски breaking changes между минорными версиями.[](https://www.chproxy.org/changelog/)

- **Репозиторий**: [contentsquareplatform/chproxy](https://hub.docker.com/r/contentsquareplatform/chproxy)
- **Поддерживаемые архитектуры**: В основном x86_64. Обратите внимание, что образы для ARM64 и ARM64v8 недоступны для некоторых версий.[](https://www.chproxy.org/changelog/)
- **Текущая версия (на май 2025)**: Последние версии можно проверить на Docker Hub, например, `v1.20.0` или выше.

---

## Использование Docker-образа

### 1. Запуск контейнера

Для запуска chproxy с использованием Docker-образа требуется указать конфигурационный файл (`config.yml`) и пробросить его в контейнер. Пример команды для запуска:

```bash
docker run -d -v $(pwd)/config.yml:/opt/config.yml -p 9090:9090 contentsquareplatform/chproxy:v1.20.0 -config /opt/config.yml
```

- **Опции**:
    - `-d`: Запуск контейнера в фоновом режиме.
    - `-v $(pwd)/config.yml:/opt/config.yml`: Проброс локального конфигурационного файла в контейнер.
    - `-p 9090:9090`: Публикация порта 9090 (по умолчанию для chproxy).
    - `contentsquareplatform/chproxy:v1.20.0`: Имя и тег образа.
    - `-config /opt/config.yml`: Указание пути к конфигурации внутри контейнера.

> **Примечание**: Начиная с версии 1.20.0, аргумент `BINARY=chproxy` больше не требуется для запуска образа, что упрощает команду.[](https://www.chproxy.org/changelog/)

#### Пример конфигурации (config.yml):

```yaml
server:
  http:
    listen_addr: ":9090"
    allowed_networks: ["10.0.0.0/8"]
users:
  - name: "default"
    password: "secret"
    to_cluster: "clickhouse-cluster"
    to_user: "default"
clusters:
  - name: "clickhouse-cluster"
    nodes: ["clickhouse1:8123", "clickhouse2:8123"]
```

Сохраните этот файл как `config.yml` в текущей директории перед запуском контейнера.

---

### 2. Проверка работоспособности

После запуска контейнера проверьте, что chproxy работает, отправив тестовый запрос:

```bash
echo "SELECT 1" | curl 'http://default:secret@localhost:9090/' --data-binary @-
```

- Если запрос успешен, вы получите ответ от ClickHouse (например, `1`).
- Проверьте логи контейнера для диагностики:

```bash
docker logs <container_id>
```

---

### 3. Обновление конфигурации без перезапуска

Chproxy поддерживает обновление конфигурации без перезапуска контейнера. После изменения `config.yml` отправьте сигнал `SIGHUP`:

```bash
docker kill --signal=SIGHUP <container_id>
```

---

### 4. Пример с Docker Compose

Для управления chproxy в кластере или интеграции с другими сервисами (например, ClickHouse, Grafana) используйте `docker-compose.yml`. Пример:

```yaml
version: '3'
services:
  chproxy:
    image: contentsquareplatform/chproxy:v1.20.0
    ports:
      - "9090:9090"
    volumes:
      - ./config.yml:/opt/config.yml
    restart: always
  clickhouse-server:
    image: yandex/clickhouse-server:latest
    ports:
      - "8123:8123"
      - "9000:9000"
    volumes:
      - ./data/clickhouse:/var/lib/clickhouse
```

Запустите с помощью:

```bash
docker-compose up -d
```

- Это развернет chproxy и ClickHouse в одной сети.
- Для масштабирования chproxy измените `docker-compose.yml` и выполните `docker-compose up -d` снова.[](https://github.com/ContentSquare/chproxy/issues/19)

---

## Особенности и ограничения

1. **Архитектуры**:
    
    - Образы для ARM64/ARM64v8 могут быть недоступны. Проверьте поддерживаемые теги на Docker Hub.[](https://www.chproxy.org/changelog/)
    - Если требуется ARM, возможно, придется собрать образ самостоятельно (см. ниже).
2. **TLS/HTTPS**:
    
    - Начиная с версии 1.20.0, chproxy требует TLS 1.2 или выше для HTTPS-соединений. Убедитесь, что клиенты поддерживают этот протокол.[](https://www.chproxy.org/changelog/)
    - Для настройки HTTPS добавьте SSL-сертификаты в конфигурацию или используйте Let’s Encrypt.
3. **Кэширование**:
    
    - С версии 1.20.0 кэш по умолчанию разделен между пользователями, но можно включить общий кэш с `shared_with_all_users: true`.[](https://www.chproxy.org/changelog/)
    - Кэш хранится локально или в Redis (требуется отдельная конфигурация).
4. **Ограничения производительности**:
    
    - Chproxy работает только с HTTP/HTTPS, что может быть менее эффективно для массовых INSERT-запросов по сравнению с нативным протоколом ClickHouse.[](https://morioh.com/a/3792b889c5be/chproxy-open-source-clickhouse-http-proxy-and-load-balancer)

---

## Сборка собственного образа

Если готовый образ не подходит (например, требуется ARM или кастомная версия), вы можете собрать Docker-образ самостоятельно. Для этого нужен бинарный файл chproxy.

1. **Скачайте или соберите chproxy**:
    
    - Скачайте бинарник с [GitHub Releases](https://github.com/Vertamedia/chproxy/releases).
    - Или соберите из исходников:
        
        ```bash
        git clone https://github.com/Vertamedia/chproxy.git
        cd chproxy
        go build
        ```
        
2. **Создайте Dockerfile**:
    
    ```Dockerfile
    FROM alpine:latest
    COPY chproxy /usr/bin/chproxy
    CMD ["chproxy", "-config", "/opt/config.yml"]
    ```
    
3. **Соберите и запустите**:
    
    ```bash
    docker build -t chproxy-custom .
    docker run -d -v $(pwd)/config.yml:/opt/config.yml -p 9090:9090 chproxy-custom
    ```
    

---

## Полезные ресурсы

- **Docker Hub**: [contentsquareplatform/chproxy](https://hub.docker.com/r/contentsquareplatform/chproxy)[](https://www.chproxy.org/install/)
- **Документация chproxy**: [chproxy.org](https://www.chproxy.org/)[](https://www.chproxy.org/changelog/)
- **GitHub**: [Vertamedia/chproxy](https://github.com/Vertamedia/chproxy)[](https://github.com/ContentSquare/chproxy/issues/19)
- **Примеры конфигураций**: Папка `config/examples` в репозитории GitHub.

---

## Заключение

Готовые Docker-образы chproxy доступны на Docker Hub (`contentsquareplatform/chproxy`) и подходят для быстрого развертывания в продакшене. Они упрощают управление конфигурацией и масштабирование, особенно в кластерах с ClickHouse. Основные шаги: загрузить образ, подготовить `config.yml`, запустить контейнер и настроить уведомления или мониторинг. Если нужны специфические архитектуры или кастомизации, можно собрать собственный образ. Для продакшена рекомендуется настроить HTTPS и мониторинг через Prometheus.

Если у вас есть конкретные требования (например, интеграция с существующим кластером ClickHouse или настройка кэширования), могу предложить более детальную конфигурацию.