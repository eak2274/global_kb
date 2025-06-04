
Отлично, ты хочешь разобраться с движками таблиц **`MergeTree` и его разновидностями в ClickHouse** — это **очень важная тема**, если ты работаешь с большими объёмами данных и строишь аналитику.

---

# 📚 Введение: семейство `MergeTree`

Все движки из семейства `MergeTree`:
- Предназначены для хранения больших таблиц
- Автоматически **сливают (merge) партиции**
- Поддерживают **индексацию по первичному ключу**
- Используются для **OLAP-запросов**

➡️ Это основной класс движков для табличных данных в ClickHouse.

---

## 🔹 1. `MergeTree`

### ✅ Самый базовый и часто используемый

#### Когда использовать:
- Когда данные приходят с временными метками
- Требуется высокая скорость чтения и агрегации
- Нужна индексация по времени и первичному ключу

### Пример:

```sql
CREATE TABLE visits (
    event_date Date,
    event_time DateTime,
    user_id UInt64,
    page String,
    duration Int32
) ENGINE = MergeTree()
ORDER BY (event_date, user_id)
PARTITION BY toYYYYMM(event_date);
```

> ⚠️ `MergeTree` сам не удаляет или не объединяет дубликаты — дублирование возможно.

---

## 🔹 2. `ReplacingMergeTree`

### 🎯 Удаляет дубликаты на этапе мёрджа

#### Когда использовать:
- Данные могут содержать **дубликаты** (например, из Kafka)
- Нужно **автоматическое удаление дублей**

### Пример:

```sql
CREATE TABLE logs (
    id UInt64,
    event_time DateTime,
    message String
) ENGINE = ReplacingMergeTree()
ORDER BY (id, event_time)
```

➡️ При слиянии кусков останется только **последняя запись** для одинаковых ключей

> Можно указать столбец версии:
```sql
ENGINE = ReplacingMergeTree(version)
```
Тогда будет выбираться строка с **максимальной версией**

---

## 🔹 3. `SummingMergeTree`

### 🎯 Суммирует значения числовых полей автоматически

#### Когда использовать:
- Агрегируешь много данных
- Частые запросы типа `SELECT sum(...) GROUP BY ...`
- Хочешь уменьшить объём данных через предагрегацию

### Пример:

```sql
CREATE TABLE sales (
    event_date Date,
    product_id UInt32,
    quantity UInt32,
    revenue Decimal(10, 2)
) ENGINE = SummingMergeTree()
ORDER BY (event_date, product_id)
```

➡️ Все поля, кроме ключевых, будут суммироваться при мёрдже

> Если нужно явно указать, что суммировать:
```sql
ENGINE = SummingMergeTree(revenue)  -- суммируется только revenue
```

---

## 🔹 4. `AggregatingMergeTree`

### 🎯 Для работы с агрегирующими функциями (например, `AggregateFunction(...)`)
  
#### Когда использовать:
- Высокопроизводительные агрегации
- Нужно сохранять промежуточные состояния агрегаций

### Пример:

```sql
CREATE TABLE user_stats (
    user_id UInt64,
    event_date Date,
    pageviews SimpleAggregateFunction(sum, UInt64),
    session_durations AggregateFunction(aggState, Float32)
) ENGINE = AggregatingMergeTree()
ORDER BY (user_id, event_date)
```

> Типы:
- `SimpleAggregateFunction(...)` — для простых агрегаций (`sum`, `min`, `max`)
- `AggregateFunction(...)` — для сложных (`uniq`, `quantile`, `groupArray`)

---

## 🔹 5. `CollapsingMergeTree`

### 🎯 Для хранения событий с возможностью "схлопывания" (collapsing)

#### Когда использовать:
- Есть события, которые можно отменить (например, like / unlike)
- Нужно поддерживать актуальное состояние записей

### Пример:

```sql
CREATE TABLE actions (
    user_id UInt64,
    action_type Enum8('Add' = 1, 'Remove' = -1),
    event_date Date
) ENGINE = CollapsingMergeTree(action_type)
ORDER BY (user_id, event_date)
```

➡️ При мёрдже:
- Записи с `action_type = 1` и `action_type = -1` **схлопываются**

---

## 🔹 6. `VersionedCollapsingMergeTree`

### 🎯 То же, что и `CollapsingMergeTree`, но работает с версиями

#### Когда использовать:
- Много обновлений
- Нужно учитывать порядок обработки

### Пример:

```sql
CREATE TABLE events (
    key UInt32,
    sign Int8,
    version UInt32,
    value String
) ENGINE = VersionedCollapsingMergeTree(sign, version)
ORDER BY key
```

> Здесь `sign` определяет тип строки (`+1` / `-1`)  
> `version` — номер версии, чтобы правильно выбрать последнее состояние

---

## 🔹 7. `GraphiteMergeTree`

### 🎯 Оптимизирован для хранения и агрегации графитных метрик

#### Когда использовать:
- РаЍ이터 графитных метрик (например, с графиками нагрузки)
- Нужно агрегировать метрики по времени

### Пример:

```sql
CREATE TABLE graphite_data (
    Path String,
    Time UInt32,
    Value Float64,
    Version UInt32
) ENGINE = GraphiteMergeTree('graphite_rollup')
ORDER BY Path
```

➡️ Требует конфигурацию `graphite_rollup` в `config.xml`  
➡️ Позволяет сжимать данные, например, за день/неделю/месяц

---

## 🔁 8. `ReplicatedMergeTree` и репликация

Все вышеперечисленные движки имеют **реплицированную версию**:
- `ReplicatedMergeTree`
- `ReplicatedReplacingMergeTree`
- `ReplicatedSummingMergeTree`

#### Пример:

```sql
CREATE TABLE replicated_visits (
    event_date Date,
    user_id UInt64,
    page String
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{shard}/visits', '{replica}')
ORDER BY (event_date, user_id)
```

> Репликация позволяет создавать отказоустойчивые кластеры

---

# 🧠 Сравнение движков

| Движок | Удаление дублей | Агрегация | Отмена записей | Версии | Репликация |
|--------|------------------|------------|----------------|----------|-------------|
| `MergeTree` | ❌ | ❌ | ❌ | ❌ | ✅ Да |
| `ReplacingMergeTree` | ✅ Да | ❌ | ❌ | ❌ | ✅ Да |
| `SummingMergeTree` | ❌ | ✅ Да | ❌ | ❌ | ✅ Да |
| `AggregatingMergeTree` | ❌ | ⚠️ Только через `AggregateFunction` | ❌ | ❌ | ✅ Да |
| `CollapsingMergeTree` | ❗ Схлопывает по `sign` | ❌ | ✅ Да | ❌ | ✅ Да |
| `VersionedCollapsingMergeTree` | ❗ По `sign` и `version` | ❌ | ✅ Да | ✅ Да | ✅ Да |
| `GraphiteMergeTree` | ❌ | ✅ Да (графитные метрики) | ❌ | ❌ | ✅ Да |

---

## 🔄 Как выбрать?

| Что тебе нужно | Рекомендуемый движок |
|----------------|-----------------------|
| Обычная таблица с данными | ✅ `MergeTree` |
| Убрать дубликаты | ✅ `ReplacingMergeTree` |
| Суммировать данные | ✅ `SummingMergeTree` |
| Сохранить агрегационное состояние | ✅ `AggregatingMergeTree` |
| События, которые можно отменить | ✅ `CollapsingMergeTree` |
| Сложные случаи с версиями | ✅ `VersionedCollapsingMergeTree` |
| Графиковые метрики | ✅ `GraphiteMergeTree` |
| Репликация | ✅ Почти все движки можно сделать `Replicated...MergeTree` |

---

## 🧪 Практическое задание

Попробуй создать таблицу с `ReplacingMergeTree`, вставить дублированные записи и проверь, как они удаляются после мёрджа:

```sql
CREATE TABLE users_log (
    user_id UInt64,
    name String
) ENGINE = ReplacingMergeTree()
ORDER BY user_id;

INSERT INTO users_log VALUES (1, 'Alice'), (1, 'Alicia');

OPTIMIZE TABLE users_log FINAL;  -- принудительно запускаем мёрдж

SELECT * FROM users_log;
```

---

## 📝 Советы:

| Совет | Описание |
|-------|----------|
| Используй `ORDER BY`, а не `PRIMARY KEY` | Чтобы не запутаться |
| Используй `PARTITION BY` по дате | Логично делить данные по дням или месяцам |
| Добавляй `OPTIMIZE TABLE` для тестирования | Но в продакшене он не нужен — всё происходит автоматически |
| Для агрегаций используй `AggregatingMergeTree` + `*-State` функции | Например, `sumState`, `uniqState` и т.д. |
| Репликация требует ZooKeeper / Keeper | Настройка репликации — отдельная тема |
