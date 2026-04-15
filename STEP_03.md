# **STEP_03.md — Настройка Топвизора**

Самодостаточный модуль. Загружается: CORE.md + PROJECT.md + этот файл. ~4K токенов.

---

# **██ КАРТОЧКА ШАГА**

| **Параметр**        | **Значение**                                                                 |
|---------------------|------------------------------------------------------------------------------|
| ID                  | 03                                                                           |
| Название            | Настройка Топвизора (проект, группы, запросы, регионы, расписание)            |
| Блок                | 1: Подготовительный                                                          |
| Основная роль       | B: Инженер автоматизации                                                     |
| Консультативные     | A: SEO-стратег, G: Инженер контрактов данных                                 |
| Тип                 | Скрипт                                                                       |
| HitL                | Нет (результат → мониторинг автоматически)                                   |
| Приоритет ROI       | ВЫСОКИЙ — без трекинга позиций невозможен мониторинг эффективности            |
| Зависит от          | Шаг 02 (Структура сайта) — site_tree + cluster_ids + url_pattern             |
| Передаёт в          | Шаг 33 (Конкурентная разведка), Шаг 34 (Content Decay), Шаг 40 (Авто-отчётность), Шаг 40.1 (Аномалии) |
| Режим Opus          | Не используется (тип = Скрипт)                                              |

---

# **██ ЦЕЛЬ**

Автоматически создать и настроить проект в Топвизоре: импортировать все запросы из семантического ядра, сгруппировать по кластерам и страницам из архитектуры сайта, настроить регионы и поисковые системы ([Я]/[G]), задать расписание проверок — так, чтобы с момента запуска шага позиции трекались без ручного вмешательства.

---

# **██ КОНТРАКТ ВХОДНЫХ ДАННЫХ (JSON)**

Вход из Шага 02 (Структура сайта). Шаблон 5: Стратегия. Необходимые поля:

```json
{
  "project_id": "string",
  "step_id": "02",
  "prompt_version": "string",
  "timestamp": "ISO 8601",
  "strategy_name": "string",
  "site_tree": [
    {
      "node_id": "string (N001...)",
      "page_type": "string",
      "title": "string",
      "url_pattern": "string (/path/)",
      "depth": "number",
      "parent_node_id": "string|null",
      "cluster_ids": ["string (CL001...)"],
      "primary_intent": "string",
      "engine_tag": "[Я]|[G]|[ОБА]",
      "estimated_traffic": "number",
      "priority": "critical|high|medium|low"
    }
  ],
  "url_rules": ["object"],
  "taxonomy": {
    "max_depth": "number",
    "total_pages_planned": "number"
  },
  "hitl_status": "approved",
  "qc_score": "0-100"
}
```

Дополнительно из PROJECT.md: `domain`, `geo`, `priority_engines`, `niche_cluster`, `main_competitors`.

Дополнительно из Шага 01 (через NocoDB): `items[].queries[]` — полный список запросов с volume, intent, engine.

---

# **██ КОНТРАКТ ВЫХОДНЫХ ДАННЫХ (JSON)**

Шаблон 4: Скрипт/Конфигурация. Роль G валидирует.

```json
{
  "project_id": "string",
  "step_id": "03",
  "prompt_version": "string",
  "timestamp": "ISO 8601",
  "target_system": "topvisor",
  "actions": [
    {
      "action_id": "string (A001...)",
      "type": "create|update|configure",
      "target": "string (project|group|keywords|regions|schedule|competitors)",
      "config": {
        "topvisor_project_id": "number|null",
        "name": "string|null",
        "url": "string|null",
        "keywords": ["string"]|null,
        "group_name": "string|null",
        "regions": [
          {
            "searcher": "yandex|google",
            "region_id": "number",
            "region_name": "string"
          }
        ]|null,
        "schedule": {
          "frequency": "daily|weekly|manual",
          "time": "string (HH:MM UTC)",
          "days": ["string"]|null
        }|null,
        "competitors": ["string (domain)"]|null
      },
      "validation_result": "pass|fail|skip",
      "engine_tag": "[Я]|[G]|[ОБА]"
    }
  ],
  "execution_log": ["string"],
  "summary": {
    "topvisor_project_id": "number",
    "total_keywords": "number",
    "total_groups": "number",
    "regions_configured": "number",
    "competitors_added": "number",
    "schedule_set": "boolean",
    "searchers": ["yandex", "google"]
  },
  "qc_score": "0-100"
}
```

---

# **██ ПРОМПТ ДЛЯ CLAUDE OPUS**

> Тип шага = Скрипт. Секция промпта минимальна — Opus не вызывается. Вся логика реализована в скриптах Claude Code.

Не применимо. Шаг полностью автоматизирован через API Топвизора.

---

# **██ RAG-ОБОГАЩЕНИЕ**

| Коллекция Qdrant    | Запрос                      | Назначение                                              | Лимит токенов |
|---------------------|-----------------------------|---------------------------------------------------------|---------------|
| `site_structure`    | project_id, step_id=02      | Полное дерево сайта для маппинга URL → группа Топвизора | 1000          |
| `search_patterns`   | гео + язык                  | Региональные коды Топвизора (Яндекс region_id)          | 200           |

Общий лимит RAG-контекста: 1200 токенов. Используется скриптами, не Opus.

---

# **██ СКРИПТЫ CLAUDE CODE**

> Тип шага = Скрипт → основное содержание здесь.

## **Пред-шаг: Подготовка данных для Топвизора**

```
# step_03_pre_prepare_data.py
# Назначение: Сборка запросов из Шага 01 + маппинг на URL из Шага 02
# Вход: step_01_output (кластеры с запросами), step_02_output (site_tree)
# Выход: topvisor_import_data.json
#   {
#     groups: [{
#       name: "N001 — Лечение кариеса",
#       url: "/uslugi/lechenie-kariesa/",
#       engine_tag: "[ОБА]",
#       keywords: [{query: "лечение кариеса москва", engine: "both", volume: 1200}, ...]
#     }, ...],
#     domain: "domain.ru",
#     geo: "Москва",
#     priority_engines: "both",
#     competitors: ["competitor1.ru", "competitor2.ru"]
#   }
#
# Действия:
#   1. Загрузить step_01_output из NocoDB (items[] с queries[])
#   2. Загрузить step_02_output из NocoDB (site_tree[] с cluster_ids[])
#   3. Для каждой ноды site_tree:
#      a. Найти все кластеры (cluster_ids) → собрать их запросы из step_01
#      b. Сформировать группу: name = "node_id — title", url = url_pattern
#      c. engine_tag наследовать из ноды
#   4. Дедупликация запросов внутри группы (по лемме)
#   5. Фильтрация: убрать запросы с volume < 5 (шум)
#   6. Ограничение: ≤500 запросов на группу (лимит API Топвизора)
#   7. Валидация: группы без запросов → лог предупреждения
#   8. Сохранить topvisor_import_data.json
```

## **Основной скрипт: Настройка Топвизора через API**

```
# step_03_main_topvisor_setup.py
# Назначение: Полная настройка проекта в Топвизоре через REST API
# Вход: topvisor_import_data.json, PROJECT.md (credentials)
# Выход: step_03_output.json (контракт Template 4)
#
# API: https://api.topvisor.com/ (v2)
# Аутентификация: Authorization: bearer {TOPVISOR_API_KEY}
#
# Действия (последовательно):
#
#   1. СОЗДАНИЕ ПРОЕКТА
#      POST /v2/json/project/add
#      Body: {name: project.name, site: project.domain}
#      → topvisor_project_id
#      Action: {action_id:"A001", type:"create", target:"project"}
#      Если проект уже существует (поиск по домену) → update, не create.
#
#   2. НАСТРОЙКА РЕГИОНОВ
#      Для priority_engines=yandex|both:
#        POST /v2/json/project/searcher/region/add
#        Body: {project_id, searcher:0, region_key: yandex_region_id}
#        Маппинг geo → region_id:
#          Москва → 213, Санкт-Петербург → 2, Россия → 225,
#          Екатеринбург → 54, Новосибирск → 65, Казань → 43
#        Action: {action_id:"A002", type:"configure", target:"regions", engine_tag:"[Я]"}
#      Для priority_engines=google|both:
#        POST /v2/json/project/searcher/region/add
#        Body: {project_id, searcher:1, region_key: google_region_id}
#        Маппинг geo → Google region:
#          Москва → "Moscow,Moscow,Russia", Россия → "Russia"
#        Action: {action_id:"A003", type:"configure", target:"regions", engine_tag:"[G]"}
#
#   3. СОЗДАНИЕ ГРУПП И ИМПОРТ ЗАПРОСОВ
#      Для каждой группы из topvisor_import_data:
#        a. POST /v2/json/project/keywords/folders/add
#           Body: {project_id, name: group.name}
#           → folder_id
#        b. POST /v2/json/project/keywords/add
#           Body: {project_id, folder_id, keywords: group.keywords.map(k => k.query)}
#           Батчами по 100 запросов (лимит API).
#        c. Привязка URL к группе:
#           POST /v2/json/project/keywords/folders/update
#           Body: {project_id, id: folder_id, target: domain + group.url}
#        Action: {action_id:"A004-A0XX", type:"create", target:"keywords"}
#
#   4. ДОБАВЛЕНИЕ КОНКУРЕНТОВ
#      Для каждого конкурента из project.main_competitors:
#        POST /v2/json/project/competitor/add
#        Body: {project_id, name: competitor, site: competitor}
#        Action: {action_id:"A0XX", type:"create", target:"competitors", engine_tag:"[ОБА]"}
#
#   5. НАСТРОЙКА РАСПИСАНИЯ
#      POST /v2/json/project/update
#      Body: {id: project_id, positions_check: {schedule: "daily", time: "06:00"}}
#      Для commercial_local/medical/legal — daily (позиции нестабильны).
#      Для info/education — 3 раза в неделю (Пн/Ср/Пт).
#      Для ecom — daily в высокий сезон, 3x в неделю вне сезона.
#      Action: {action_id:"A0XX", type:"configure", target:"schedule", engine_tag:"[ОБА]"}
#
# Обработка ошибок:
#   - 429 (rate limit): бэкофф 5с → 15с → 45с, макс. 3 попытки
#   - 401 (auth): немедленный fail с алертом — невалидный API-ключ
#   - 5xx: retry 3x с экспоненциальным бэкоффом
#   - Дубликат запроса (ошибка API): skip, лог предупреждения
#   - Превышение лимита запросов проекта: алерт, частичный импорт
#
# Все API-вызовы логируются в execution_log[].
```

## **Пост-шаг: Валидатор**

```
# step_03_post_validate.py
# Назначение: Проверка корректности настройки Топвизора
# Вход: step_03_output.json + API Топвизора (верификация)
# Выход: validation_result {pass|fail, errors[], qc_score}
#
# Проверки:
#   1. topvisor_project_id существует и != null
#   2. Все actions[].validation_result != "fail"
#   3. total_keywords > 0
#   4. total_groups ≥ 3 (минимум 3 группы для осмысленного трекинга)
#   5. regions_configured ≥ 1
#   6. При priority_engines=both — настроены регионы И для Яндекса, И для Google
#   7. schedule_set = true
#   8. competitors_added ≥ 1 (если main_competitors не пуст)
#   9. Верификация через API: GET /v2/json/project/get → проверка project_id
#  10. Верификация: GET /v2/json/project/keywords/get → count совпадает с total_keywords (±5%)
#  11. Нет actions с type="create" и validation_result="skip" (пропущенные действия)
#  12. execution_log не содержит строк с "ERROR" или "CRITICAL"
```

## **Пост-шаг: Первая проверка позиций**

```
# step_03_post_first_check.py
# Назначение: Запуск первой проверки позиций для верификации
# Вход: topvisor_project_id
# Выход: first_check_result {status, positions_count}
# Действия:
#   1. POST /v2/json/project/positions/check/add
#      Body: {project_id: topvisor_project_id}
#   2. Ожидание 60с (позиции собираются)
#   3. GET /v2/json/project/positions/get
#      → если positions_count > 0 → всё работает
#      → если 0 → лог предупреждения (возможно, проверка ещё идёт)
#   4. Результат в execution_log
```

---

# **██ КРИТЕРИИ QC**

| **Проверка**                             | **Порог**                            | **Вес** | **При провале** |
|------------------------------------------|--------------------------------------|---------|-----------------|
| Проект создан в Топвизоре                | topvisor_project_id != null          | 20      | Retry           |
| Запросы импортированы                    | total_keywords > 0                   | 20      | Retry           |
| Группы созданы                           | total_groups ≥ 3                     | 10      | Retry           |
| Регионы настроены                        | regions_configured ≥ 1               | 10      | Retry           |
| Баланс ПС при both                      | Яндекс + Google регионы             | 10      | Retry           |
| Расписание задано                        | schedule_set = true                  | 10      | Ревью           |
| Конкуренты добавлены                     | competitors_added ≥ 1                | 5       | Ревью           |
| Нет критических ошибок в логе            | 0 строк ERROR/CRITICAL               | 10      | Retry           |
| API-верификация количества запросов      | ±5% от total_keywords                | 5       | Ревью           |

QC = сумма весов. ≥80 → автоматический переход, 60–79 → ревью, <60 → retry.

---

# **██ СТРУКТУРА N8N-ВОРКФЛОУ**

```
Нода 1:  Триггер (после утверждения HitL Шага 02)
Нода 2:  Загрузка PROJECT.md из NocoDB
Нода 3:  Загрузка output Шага 02 из NocoDB (site_tree)
Нода 4:  Загрузка output Шага 01 из NocoDB (кластеры + запросы)
Нода 5:  step_03_pre_prepare_data.py — сборка данных для импорта
Нода 6:  step_03_main_topvisor_setup.py — основной скрипт настройки
         → при ошибке API → retry с бэкоффом (макс. 3)
Нода 7:  step_03_post_validate.py — валидация настройки
         → провал → Нода 6 (макс. 3 retry)
Нода 8:  step_03_post_first_check.py — запуск первой проверки позиций
Нода 9:  NocoDB — сохранение результата + prompt_version
Нода 10: Grafana — метрики (total_keywords, groups, qc_score)
Нода 11: Переход к Шагу 04 или стоп
```

## **Обработка ошибок**

```
• API Топвизора 429: бэкофф 5с→15с→45с, макс. 3 попытки
• API Топвизора 401: немедленный fail + алерт Telegram (невалидный ключ)
• API Топвизора 5xx: retry 3x с экспоненциальным бэкоффом
• NocoDB недоступен: retry 3x, затем алерт
• QC провал 3x: алерт Telegram с логом ошибок
• Превышение лимита запросов Топвизора: частичный импорт + алерт
```

---

# **██ ПРИМЕР FEW-SHOT**

```json
{
  "project_id": "dental_moscow_001",
  "step_id": "03",
  "prompt_version": "step03_v1.0",
  "timestamp": "2025-06-16T08:00:00Z",
  "target_system": "topvisor",
  "actions": [
    {
      "action_id": "A001",
      "type": "create",
      "target": "project",
      "config": {
        "topvisor_project_id": 1234567,
        "name": "dental_moscow_001 — ДентаПлюс",
        "url": "dentaplus.ru"
      },
      "validation_result": "pass",
      "engine_tag": "[ОБА]"
    },
    {
      "action_id": "A002",
      "type": "configure",
      "target": "regions",
      "config": {
        "regions": [
          {"searcher": "yandex", "region_id": 213, "region_name": "Москва"},
          {"searcher": "google", "region_id": 1, "region_name": "Moscow, Russia"}
        ]
      },
      "validation_result": "pass",
      "engine_tag": "[ОБА]"
    },
    {
      "action_id": "A003",
      "type": "create",
      "target": "keywords",
      "config": {
        "group_name": "N002 — Услуги стоматологии",
        "url": "dentaplus.ru/uslugi/",
        "keywords": [
          "стоматология москва",
          "стоматологическая клиника",
          "зубной врач москва"
        ]
      },
      "validation_result": "pass",
      "engine_tag": "[ОБА]"
    },
    {
      "action_id": "A004",
      "type": "create",
      "target": "keywords",
      "config": {
        "group_name": "N003 — Лечение кариеса",
        "url": "dentaplus.ru/uslugi/lechenie-kariesa/",
        "keywords": [
          "лечение кариеса москва",
          "лечение кариеса цена",
          "кариес стоматология",
          "лечение зубов без боли"
        ]
      },
      "validation_result": "pass",
      "engine_tag": "[ОБА]"
    },
    {
      "action_id": "A005",
      "type": "create",
      "target": "keywords",
      "config": {
        "group_name": "N007 — Стоматология в Бутово",
        "url": "dentaplus.ru/uslugi/stomatologiya-butovo/",
        "keywords": [
          "стоматология бутово",
          "зубной врач бутово",
          "стоматологическая клиника южное бутово"
        ]
      },
      "validation_result": "pass",
      "engine_tag": "[Я]"
    },
    {
      "action_id": "A006",
      "type": "create",
      "target": "competitors",
      "config": {
        "competitors": ["konkurrent1.ru", "konkurrent2.ru", "konkurrent3.ru"]
      },
      "validation_result": "pass",
      "engine_tag": "[ОБА]"
    },
    {
      "action_id": "A007",
      "type": "configure",
      "target": "schedule",
      "config": {
        "schedule": {
          "frequency": "daily",
          "time": "06:00",
          "days": null
        }
      },
      "validation_result": "pass",
      "engine_tag": "[ОБА]"
    }
  ],
  "execution_log": [
    "2025-06-16T08:00:01Z INFO Проект создан: topvisor_id=1234567",
    "2025-06-16T08:00:03Z INFO Регион Яндекс добавлен: Москва (213)",
    "2025-06-16T08:00:04Z INFO Регион Google добавлен: Moscow, Russia",
    "2025-06-16T08:00:06Z INFO Группа создана: N002 — Услуги стоматологии (3 запроса)",
    "2025-06-16T08:00:08Z INFO Группа создана: N003 — Лечение кариеса (4 запроса)",
    "2025-06-16T08:00:10Z INFO Группа создана: N007 — Стоматология в Бутово (3 запроса)",
    "2025-06-16T08:00:15Z INFO Конкуренты добавлены: 3 шт",
    "2025-06-16T08:00:16Z INFO Расписание: daily 06:00 UTC",
    "2025-06-16T08:01:20Z INFO Первая проверка запущена, позиции собираются",
    "2025-06-16T08:02:30Z INFO Верификация: 10 запросов в Топвизоре, ожидалось 10. OK"
  ],
  "summary": {
    "topvisor_project_id": 1234567,
    "total_keywords": 10,
    "total_groups": 3,
    "regions_configured": 2,
    "competitors_added": 3,
    "schedule_set": true,
    "searchers": ["yandex", "google"]
  },
  "qc_score": 95
}
```

---

# **██ СТРАТЕГИЯ ОТКАТА**

```
Откат A: Частичный импорт.
  Если API Топвизора отвечает с ошибками на часть запросов — импортировать
  доступные, логировать пропущенные, пометить в NocoDB partial_import=true.
  Повторить импорт пропущенных при следующем запуске.

Откат B: Альтернативный трекер.
  Если Топвизор полностью недоступен (>3 подряд 5xx) — переключиться на
  DataForSEO Position Tracking API как fallback. Формат output идентичен,
  target_system="dataforseo". Лог: "FALLBACK: Topvisor → DataForSEO".

Откат C: Ручная настройка.
  Если оба API недоступны — сформировать CSV-файл для ручного импорта:
  query;group;url;engine. Отправить через Telegram + алерт.
  Пометка в NocoDB: manual_setup_required=true.
```

---

# **██ АДАПТАЦИИ ПОД КЛАСТЕР НИШИ**

| Кластер              | Особенности настройки Топвизора                                                                        |
|----------------------|--------------------------------------------------------------------------------------------------------|
| `commercial_local`   | Несколько регионов Яндекса (город + район). Конкуренты из 2GIS/Яндекс.Бизнес. Расписание: daily.       |
| `commercial_national`| Один регион (Россия/страна). Больше запросов (до лимита). Расписание: daily.                            |
| `ecom`               | Группы по категориям каталога. Товарные запросы отдельно. Расписание: daily (цены меняются часто).      |
| `saas`               | Мультиязычные группы (если i18n). Конкуренты международные. Расписание: 3x в неделю.                   |
| `info`               | Группы по topical clusters. Длиннохвостые запросы. Расписание: 3x в неделю (позиции стабильнее).       |
| `medical`            | Группы по услугам + заболеваниям. YMYL-запросы отдельно. Расписание: daily (высокая конкуренция).       |
| `legal`              | Группы по практикам + ситуациям. Геозапросы по судам/регионам. Расписание: daily.                       |
| `education`          | Группы по программам. Сезонные запросы помечены. Расписание: daily в сезон, 2x вне сезона.              |

---

# **██ МЕТРИКИ УСПЕХА**

| Тип               | Метрика                                              | Как измерять                                                |
|--------------------|------------------------------------------------------|-------------------------------------------------------------|
| Опережающая        | Покрытие запросов (%)                                | total_keywords / запросов из Шага 01 * 100                   |
| Опережающая        | Покрытие групп (%)                                   | total_groups / нод site_tree из Шага 02 * 100                |
| Опережающая        | Конфигурация регионов                                | regions_configured ≥ 1 для каждого ПС из priority_engines    |
| Запаздывающая      | Первая проверка позиций успешна                      | positions_count > 0 через 24 часа после настройки            |
| Запаздывающая      | Стабильность трекинга (7 дней)                       | ≥6 из 7 дней проверка прошла без ошибок                      |
| Контрольная        | Время выполнения шага                                | ≤5 минут (включая первую проверку)                           |

---

# **██ ИСТОРИЯ ИЗМЕНЕНИЙ**

| **Версия** | **Дата**  | **Изменения**                                                                                    |
|------------|-----------|-------------------------------------------------------------------------------------------------|
| v1.0       | 2025-01   | Базовая настройка: создание проекта, импорт запросов, регионы                                   |
| v2.0       | 2025-06   | Группировка по site_tree, engine теги, конкуренты, расписание по кластеру ниши                   |
| v3.0       | 2025-12   | Fallback на DataForSEO, первая проверка позиций, батчевый импорт, расширенный QC, адаптации      |
