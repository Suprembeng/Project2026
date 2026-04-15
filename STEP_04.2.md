# STEP_04.2.md — Сиротские страницы

Самодостаточный модуль. Загружается: CORE.md + PROJECT.md + этот файл. ~4K токенов.

---

## ██ КАРТОЧКА ШАГА

| **Параметр** | **Значение** |
| --- | --- |
| ID | 04.2 |
| Название | Обнаружение и приоритизация сиротских страниц |
| Блок | 2: Структура и контент |
| Основная роль | B: Инженер автоматизации |
| Консультативные | A: SEO-стратег, G: Инженер контрактов данных |
| Тип | Скрипт |
| HitL | Нет (результат → Шаг 07 автоматически) |
| Приоритет ROI | ВЫСОКИЙ — сиротские страницы не получают краулинговый бюджет, теряют позиции |
| Зависит от | Шаг 04.1 (Каннибализация — список URL с внутренними связями), PROJECT.md |
| Передаёт в | Шаг 07 (Перелинковка), Шаг 14 (Robots + Sitemap), Шаг 23 (Управление индексацией) |
| Режим Opus | JSON-вывод: temperature 0.15, top_p 0.9, max_tokens 8192 |

---

## ██ ЦЕЛЬ

Обнаружить все страницы сайта, не имеющие ни одной входящей внутренней ссылки (сиротские), сопоставив данные краулера, Sitemap.xml, Яндекс.Вебмастера и GSC, и приоритизировать их по трафику, позициям и стратегической ценности — для последующей интеграции в структуру перелинковки (Шаг 07) или деиндексации (Шаг 23).

---

## ██ КОНТРАКТ ВХОДНЫХ ДАННЫХ (JSON)

Вход формируется из output шага 04.1 (Каннибализация) + данные PROJECT.md + результаты pre-scripts.

```json
{
  "project_id": "string",
  "domain": "string",
  "priority_engines": "yandex|google|both",
  "niche_cluster": "string",
  "cannibalization_items": [
    {
      "id": "string",
      "title": "string",
      "priority": "critical|high|medium|low",
      "action_required": "string",
      "engine_tag": "[Я]|[G]|[ОБА]",
      "estimated_impact": "1-10",
      "effort": "1-10",
      "urls": ["string"]
    }
  ],
  "crawl_data": {
    "source": "screaming_frog|custom_crawler",
    "all_crawled_urls": ["string"],
    "internal_links_map": {
      "url": ["incoming_url_1", "incoming_url_2"]
    },
    "crawl_date": "ISO 8601"
  },
  "sitemap_urls": ["string"],
  "gsc_urls": ["string"],
  "ym_urls": ["string"],
  "analytics_data": {
    "url_sessions_90d": {
      "url": "number"
    },
    "url_positions": {
      "url": {
        "avg_position": "number",
        "clicks_90d": "number",
        "impressions_90d": "number"
      }
    }
  }
}
```

**Обязательные поля:** `project_id`, `domain`, `priority_engines`, `crawl_data`, `sitemap_urls`.

**Опциональные поля:** `gsc_urls`, `ym_urls`, `analytics_data` (при отсутствии — приоритизация без трафиковых данных, пометка `data_gap: true`).

---

## ██ КОНТРАКТ ВЫХОДНЫХ ДАННЫХ (JSON)

Наследует **Шаблон 1: Список с приоритетами**. Роль G валидирует.

```json
{
  "project_id": "string",
  "step_id": "04.2",
  "prompt_version": "string",
  "timestamp": "ISO 8601",
  "items": [
    {
      "id": "string (ORP001, ORP002...)",
      "title": "string (URL страницы)",
      "priority": "critical|high|medium|low",
      "action_required": "link_to|redirect_to|deindex|delete|review",
      "engine_tag": "[Я]|[G]|[ОБА]",
      "estimated_impact": "1-10",
      "effort": "1-10",
      "detection_source": "sitemap_only|gsc_only|ym_only|analytics_only|multi_source",
      "page_type": "landing|category|article|product|service|other",
      "traffic_90d": "number|null",
      "clicks_90d": "number|null",
      "impressions_90d": "number|null",
      "avg_position": "number|null",
      "in_sitemap": "boolean",
      "indexed_google": "boolean|null",
      "indexed_yandex": "boolean|null",
      "suggested_parent": "string|null",
      "data_gap": "boolean",
      "rationale": "string"
    }
  ],
  "summary": {
    "total": "number",
    "critical": "number",
    "high": "number",
    "medium": "number",
    "low": "number",
    "total_lost_traffic_90d": "number|null",
    "total_lost_clicks_90d": "number|null",
    "action_breakdown": {
      "link_to": "number",
      "redirect_to": "number",
      "deindex": "number",
      "delete": "number",
      "review": "number"
    }
  },
  "qc_score": "0-100"
}
```

**Правила:**
- Каждый `items[].engine_tag` обязателен — `[Я]`, `[G]` или `[ОБА]` в зависимости от `indexed_google` / `indexed_yandex`.
- `action_required` определяется скриптом по правилам (см. секцию Claude Code Scripts).
- `suggested_parent` — URL-кандидат для перелинковки (ближайшая по тематике категория/страница), вычисляется из crawl_data.

---

## ██ ПРОМПТ ДЛЯ CLAUDE OPUS

> **Тип шага — Скрипт.** Основная логика выполняется скриптами. Opus используется только для финальной приоритизации и определения `suggested_parent` в случаях, когда rule-based логика недостаточна (неоднозначный page_type или отсутствие аналитических данных).

**Системный промпт (минимальный)**

```
Ты — Роль B: Инженер автоматизации.
Консультант: Роль A: SEO-стратег.

Задача: Приоритизировать сиротские страницы и определить рекомендуемое действие.

ПРАВИЛА:
1. Каждая страница получает action_required: link_to|redirect_to|deindex|delete|review
2. Каждая запись помечена engine_tag: [Я]|[G]|[ОБА]
3. suggested_parent — URL ближайшей релевантной страницы из структуры сайта
4. Учитывай niche_cluster при определении page_type
5. Страницы с трафиком > 0 — никогда delete, только link_to или redirect_to

ВЫВОД: Валидный JSON по контракту. Без markdown. Без комментариев. Весь текст — на РУССКОМ языке.
```

**Пользовательский промпт (шаблон)**

```
Приоритизируй сиротские страницы:

Проект: {{project.project_id}}
Домен: {{project.domain}}
Кластер: {{project.niche_cluster}}
Приоритет ПС: {{project.priority_engines}}

Сиротские страницы (обнаруженные скриптом):
{{orphan_pages_raw | json}}

Структура сайта (top-level):
{{site_structure_summary | json}}

{{#if rag_context}}
Контекст из RAG:
{{rag_context}}
{{/if}}
```

---

## ██ RAG-ОБОГАЩЕНИЕ

| Коллекция Qdrant | Запрос | Назначение |
| --- | --- | --- |
| `site_structure` | `project_id` + top-level URL | Текущая иерархия страниц для определения `suggested_parent` |
| `competitors` | `niche_cluster` + `geo` | Структуры конкурентов для бенчмарка (какие типы страниц обычно линкуются) |
| `personas` | `project_id` | Персоны из Шага 00 — для определения ценности страниц по интентам |

Лимит RAG-контекста: 1500 токенов (шаг преимущественно скриптовый, RAG — вспомогательный).

---

## ██ СКРИПТЫ CLAUDE CODE

### Пред-шаг 1: Краулер внутренних ссылок

```
# step_04_2_pre_crawl.py
# Назначение: Краулинг сайта, построение карты внутренних ссылок
# Вход: project.domain, project.priority_engines
# Выход: crawl_data.json (all_crawled_urls[], internal_links_map{})
# Действия:
# 1. Рекурсивный краулинг домена (макс. 10 000 URL, глубина ≤ 10)
# 2. Парсинг <a href> — только внутренние ссылки (same-domain)
# 3. Построение internal_links_map: {url: [список входящих URL]}
# 4. Фиксация HTTP-статусов (200, 301, 404, 410, 500)
# 5. Извлечение <title> и <h1> каждой страницы
# 6. Сохранение crawl_data.json
# Ограничения: robots.txt и crawl-delay, задержка 200мс, User-Agent: SEOBot/1.0
```

### Пред-шаг 2: Парсинг Sitemap.xml

```
# step_04_2_pre_sitemap.py
# Назначение: Извлечение всех URL из Sitemap.xml (включая sitemap index)
# Вход: project.domain
# Выход: sitemap_urls[]
# Действия:
# 1. Загрузить /sitemap.xml (и robots.txt для альтернативных путей)
# 2. Если sitemap index — рекурсивно загрузить все вложенные карты
# 3. Фильтрация: только HTML-страницы (исключить изображения, PDF, видео)
# 4. Нормализация URL (trailing slash, http→https, www→без www)
# 5. Выход: sitemap_urls.json
```

### Пред-шаг 3: Данные GSC + Яндекс.Вебмастер

```
# step_04_2_pre_search_consoles.py
# Назначение: Получение списка проиндексированных URL и метрик из GSC и ЯВ
# Вход: project.domain, API-ключи
# Выход: gsc_urls[], ym_urls[], analytics_data{}
# Действия:
# 1. GSC API: Search Analytics за 90 дней → URL + clicks + impressions + position
# 2. GSC API: URL Inspection → indexed/not indexed
# 3. Яндекс.Вебмастер API: проиндексированные URL
# 4. Яндекс.Вебмастер API: поисковые запросы → URL + клики + показы + позиция
# 5. GA4/Метрика API (при наличии): sessions по URL за 90 дней
# 6. Нормализация URL (аналогично sitemap)
# 7. Объединение → gsc_data.json, ym_data.json, analytics_data.json
# Fallback: если API недоступен — пометка data_gap: true, лог предупреждения
```

### Основной скрипт: Детектор сиротских страниц

```
# step_04_2_main_orphan_detector.py
# Назначение: Обнаружение сиротских страниц путём сопоставления источников
# Вход: crawl_data.json, sitemap_urls.json, gsc_data.json, ym_data.json, analytics_data.json
# Выход: orphan_pages_raw.json
# Действия:
#
# 1. НОРМАЛИЗАЦИЯ
#    - Приведение всех URL к единому формату (lowercase, no trailing slash, https)
#    - Исключение технических URL: /wp-admin, /feed, /amp, параметры UTM
#
# 2. ПОСТРОЕНИЕ МНОЖЕСТВ
#    - Set A: crawled_urls (все URL, найденные краулером)
#    - Set B: linked_urls (URL, имеющие ≥1 входящую внутреннюю ссылку)
#    - Set C: sitemap_urls
#    - Set D: gsc_urls (проиндексированные Google)
#    - Set E: ym_urls (проиндексированные Яндекс)
#
# 3. ОПРЕДЕЛЕНИЕ СИРОТ
#    - orphan = (C ∪ D ∪ E) - B
#    — т.е. страницы, которые известны поисковикам или есть в sitemap,
#      но не имеют входящих внутренних ссылок
#    - Дополнительно: crawled_but_unlinked = A - B (найдены краулером, но без входящих)
#
# 4. ОБОГАЩЕНИЕ ДАННЫМИ
#    - Для каждого сиротского URL:
#      a) traffic_90d — из GA4/Метрики
#      b) clicks_90d, impressions_90d, avg_position — из GSC/ЯВ
#      c) in_sitemap — есть ли в Set C
#      d) indexed_google — есть ли в Set D
#      e) indexed_yandex — есть ли в Set E
#      f) http_status — из crawl_data
#      g) page_type — rule-based: по URL-паттерну (/catalog/, /blog/, /product/)
#
# 5. RULE-BASED ПРИОРИТИЗАЦИЯ
#    priority = f(traffic, impressions, position, page_type, in_sitemap):
#    - critical: traffic_90d > 100 ИЛИ clicks_90d > 50 ИЛИ avg_position < 20
#    - high: traffic_90d > 10 ИЛИ impressions_90d > 500 ИЛИ in_sitemap=true + indexed
#    - medium: in_sitemap=true ИЛИ indexed (любой ПС) ИЛИ traffic_90d > 0
#    - low: остальные (не в sitemap, не проиндексированы, нет трафика)
#
# 6. RULE-BASED ACTION
#    action_required:
#    - link_to: priority critical|high + http_status 200 (стратегически ценная, нужна в структуре)
#    - redirect_to: http_status 301 ИЛИ есть каннибализация с другой страницей
#    - deindex: priority low + нет трафика + нет позиций + контент устарел
#    - delete: priority low + http_status 404/410 + нет индексации
#    - review: неоднозначные случаи → передача Opus для финальной приоритизации
#
# 7. ENGINE TAG
#    engine_tag:
#    - [Я]: indexed_yandex=true, indexed_google=false
#    - [G]: indexed_google=true, indexed_yandex=false
#    - [ОБА]: оба проиндексированы ИЛИ оба не проиндексированы (при priority_engines=both)
#    - При priority_engines=yandex → все [Я], при google → все [G]
#
# 8. SUGGESTED PARENT
#    - По URL-иерархии: /catalog/shoes/red-shoes → suggested_parent = /catalog/shoes/
#    - Если URL на верхнем уровне — поиск по совпадению <title>/<h1> с существующими страницами
#    - Если невозможно определить rule-based → suggested_parent = null → Opus доопределит
#
# 9. ВЫХОД: orphan_pages_raw.json — массив записей для items[]
```

### Пост-шаг: Валидатор

```
# step_04_2_post_validate.py
# Назначение: Валидация выхода по JSON-схеме и QC-критериям
# Проверки:
# 1. JSON Schema — структура соответствует Template 1
# 2. Все items имеют id (формат ORP###)
# 3. Все items имеют engine_tag ∈ {[Я], [G], [ОБА]}
# 4. Все items с traffic_90d > 0 имеют action_required ∈ {link_to, redirect_to} (НЕ delete)
# 5. summary.total = len(items)
# 6. summary.critical + high + medium + low = total
# 7. action_breakdown сумма = total
# 8. Нет дублей URL в items
# 9. При priority_engines=both — есть [Я] И [G] записи
# 10. qc_score рассчитан корректно
# При провале: вернуть список ошибок → retry
```

### Пост-шаг: NocoDB + Qdrant

```
# step_04_2_post_store.py
# Назначение: Сохранение результатов в NocoDB и Qdrant
# Действия:
# 1. NocoDB: INSERT в таблицу orphan_pages (project_id, step_id, timestamp, items JSON)
# 2. NocoDB: UPDATE таблицу step_results (step_id=04.2, status=done, qc_score)
# 3. Qdrant: upsert в коллекцию 'site_structure' — обновить метаданные orphan-статуса
# 4. Qdrant: upsert в коллекцию 'orphan_pages' — эмбеддинг (url + page_type + rationale)
#    для использования в Шаге 07 (Перелинковка)
# 5. Тег project_id, prompt_version
```

---

## ██ КРИТЕРИИ QC

| **Проверка** | **Порог** | **Вес** | **При провале** |
| --- | --- | --- | --- |
| JSON Schema валидна | Pass | 15 | Retry |
| Все URL нормализованы | 100% (no duplicates) | 10 | Retry |
| Engine теги | 100% items имеют engine_tag | 15 | Retry |
| Трафиковые страницы защищены | 0 delete/deindex при traffic > 0 | 20 | Retry |
| Summary согласован | total = сумма по priority | 10 | Retry |
| Action breakdown | сумма = total | 5 | Retry |
| Покрытие источников | ≥2 из 3 (sitemap, GSC, ЯВ) использованы | 10 | Ревью |
| Suggested parent | ≥60% items имеют suggested_parent ≠ null | 10 | Ревью |
| Соответствие кластеру | page_type логичен для niche_cluster | 5 | Ревью |

QC = сумма весов пройденных проверок. ≥80 → Шаг 07, 60-79 → ревью, <60 → retry.

---

## ██ СТРУКТУРА N8N-ВОРКФЛОУ

```
Нода 1:  Триггер (после завершения Шага 04.1 или ручной запуск)
Нода 2:  PROJECT.md из NocoDB + output Шага 04.1
Нода 3:  step_04_2_pre_sitemap.py → sitemap_urls.json
Нода 4:  step_04_2_pre_crawl.py → crawl_data.json
Нода 5:  step_04_2_pre_search_consoles.py → gsc_data.json, ym_data.json, analytics_data.json
Нода 6:  step_04_2_main_orphan_detector.py → orphan_pages_raw.json
Нода 7:  Фильтр: items с action_required = "review" → Нода 8, остальные → Нода 9
Нода 8:  [Условная] Qdrant RAG → Сборка промпта → Claude Opus API → Парсинг JSON
          (только для неоднозначных записей, ~5-15% от общего числа)
Нода 9:  Объединение результатов (скрипт + Opus) → финальный output JSON
Нода 10: step_04_2_post_validate.py → провал → Нода 6 (макс. 3 retry)
Нода 11: step_04_2_post_store.py → NocoDB + Qdrant
Нода 12: Grafana → метрики (total orphans, critical count, lost traffic)
Нода 13: Передача → Шаг 07 (Перелинковка) или стоп
```

**Обработка ошибок:**

```
• Краулер таймаут: увеличить delay до 500мс, уменьшить max_urls до 5000, retry
• Sitemap 404: проверить robots.txt, попробовать /sitemap_index.xml, /sitemap.xml.gz
• GSC/ЯВ API 429/500: бэкофф 10с→30с→90с, макс. 3
• Opus 429/500: бэкофф 10с→30с→90с, макс. 3
• Qdrant недоступен: пропустить RAG и post_store для Qdrant, лог предупреждения
• QC провал 3x: алерт Telegram с логом 3 попыток
• Невалидный JSON от Opus: удалить markdown-обёртку, повторный парсинг → retry
```

---

## ██ ПРИМЕР FEW-SHOT

```json
{
  "project_id": "dental_moscow_001",
  "step_id": "04.2",
  "prompt_version": "step04_2_v1.0",
  "timestamp": "2025-12-15T14:30:00Z",
  "items": [
    {
      "id": "ORP001",
      "title": "https://dental-moscow.ru/uslugi/otbelivanie-zoom",
      "priority": "critical",
      "action_required": "link_to",
      "engine_tag": "[ОБА]",
      "estimated_impact": 8,
      "effort": 2,
      "detection_source": "multi_source",
      "page_type": "service",
      "traffic_90d": 245,
      "clicks_90d": 89,
      "impressions_90d": 3200,
      "avg_position": 12.4,
      "in_sitemap": true,
      "indexed_google": true,
      "indexed_yandex": true,
      "suggested_parent": "https://dental-moscow.ru/uslugi/",
      "data_gap": false,
      "rationale": "Страница услуги с трафиком 245 сессий/90д, позиция 12.4 — нет ни одной внутренней ссылки. Добавление в навигацию /uslugi/ и перелинковка со смежных услуг поднимет позиции."
    },
    {
      "id": "ORP002",
      "title": "https://dental-moscow.ru/blog/kak-vybrat-zubnuyu-pastu",
      "priority": "high",
      "action_required": "link_to",
      "engine_tag": "[Я]",
      "estimated_impact": 6,
      "effort": 2,
      "detection_source": "sitemap_only",
      "page_type": "article",
      "traffic_90d": 34,
      "clicks_90d": 12,
      "impressions_90d": 890,
      "avg_position": 28.7,
      "in_sitemap": true,
      "indexed_google": false,
      "indexed_yandex": true,
      "suggested_parent": "https://dental-moscow.ru/blog/",
      "data_gap": false,
      "rationale": "Статья блога в sitemap, проиндексирована Яндексом, но не Google. Нет входящих внутренних ссылок — Google не обнаруживает. Добавить ссылку из /blog/ и связанных статей."
    },
    {
      "id": "ORP003",
      "title": "https://dental-moscow.ru/akcii/skidka-mart-2024",
      "priority": "low",
      "action_required": "deindex",
      "engine_tag": "[ОБА]",
      "estimated_impact": 2,
      "effort": 1,
      "detection_source": "gsc_only",
      "page_type": "landing",
      "traffic_90d": 0,
      "clicks_90d": 0,
      "impressions_90d": 15,
      "avg_position": 87.3,
      "in_sitemap": false,
      "indexed_google": true,
      "indexed_yandex": true,
      "suggested_parent": null,
      "data_gap": false,
      "rationale": "Истёкшая акция (март 2024), нет трафика, не в sitemap, позиция 87. Рекомендация: noindex + удалить из индекса через GSC/ЯВ."
    },
    {
      "id": "ORP004",
      "title": "https://dental-moscow.ru/vrachi/ivanov-petr",
      "priority": "medium",
      "action_required": "review",
      "engine_tag": "[G]",
      "estimated_impact": 5,
      "effort": 3,
      "detection_source": "gsc_only",
      "page_type": "other",
      "traffic_90d": 8,
      "clicks_90d": 3,
      "impressions_90d": 210,
      "avg_position": 34.1,
      "in_sitemap": false,
      "indexed_google": true,
      "indexed_yandex": false,
      "suggested_parent": "https://dental-moscow.ru/vrachi/",
      "data_gap": false,
      "rationale": "Страница врача проиндексирована Google, но не Яндексом, не в sitemap, нет внутренних ссылок. Требует ревью: проверить актуальность (врач ещё работает?). Если да — link_to + sitemap. Если нет — redirect на /vrachi/."
    }
  ],
  "summary": {
    "total": 4,
    "critical": 1,
    "high": 1,
    "medium": 1,
    "low": 1,
    "total_lost_traffic_90d": 287,
    "total_lost_clicks_90d": 104,
    "action_breakdown": {
      "link_to": 2,
      "redirect_to": 0,
      "deindex": 1,
      "delete": 0,
      "review": 1
    }
  },
  "qc_score": 88
}
```

---

## ██ СТРАТЕГИЯ ОТКАТА

```
Откат A: Краулер не завершился (таймаут/блокировка).
  → Уменьшить глубину до 5, max_urls до 3000.
  → Использовать только sitemap + GSC/ЯВ (без собственного краулинга).
  → Пометка в NocoDB: partial_crawl = true.

Откат B: GSC и ЯВ API недоступны.
  → Работать только с sitemap + crawl_data.
  → Все items получают data_gap: true.
  → Приоритизация только по in_sitemap + URL-паттерну (без трафика/позиций).
  → Пометка: analytics_unavailable = true.

Откат C: Opus не может приоритизировать review-записи (3x fail).
  → Все review → medium priority, action_required = "review".
  → Алерт: ручная приоритизация требуется.
  → Передать в Шаг 07 как есть — перелинковка решит по ходу.
```

---

## ██ АДАПТАЦИИ ПОД КЛАСТЕР

```
• commercial_local:
  — Приоритет [Я]: Яндекс.Карты, 2GIS-страницы, NAP-страницы
  — Сиротские филиалы/адреса → critical (потеря локальных позиций)
  — page_type: "branch"|"service_local" — дополнительные типы

• commercial_national:
  — Фокус на категориях и лендингах услуг
  — Сиротские landing-страницы с конверсионным трафиком → critical
  — Проверка: есть ли в breadcrumbs (если нет — сирота де-факто)

• ecom:
  — Товарные карточки без входящих из категорий → critical
  — Фильтровые URL: /catalog/?color=red — исключить из orphan-анализа (обрабатываются Шагом 15)
  — Учёт пагинации: /catalog/page/2 — не считать сиротой

• saas:
  — Документация, changelog, интеграции — часто сиротские
  — Приоритет: страницы с comparison/alternative в URL → high
  — Учёт мультиязычности: /en/feature/ → проверять отдельно от /ru/feature/

• info:
  — Старые статьи блога — основной источник сирот
  — Приоритизация по Content Decay: если статья теряет трафик И сирота → critical
  — Теги/авторы без ссылок → medium (архивные страницы)

• medical:
  — Страницы врачей, симптомов, услуг — YMYL, потеря E-E-A-T при сиротстве
  — Профили врачей без ссылок из услуг → critical
  — Страницы лицензий/сертификатов → high (E-E-A-T сигнал)

• legal:
  — Страницы практик/кейсов без ссылок → critical
  — Юридические статьи (консультации) → high
  — Профили адвокатов → аналогично medical

• education:
  — Программы/курсы без ссылок из каталога → critical
  — Сезонные страницы (приём 2024) → low/deindex после сезона
  — Отзывы студентов → medium
```

---

## ██ МЕТРИКИ УСПЕХА

```
Опережающий:
  — Количество обнаруженных сиротских страниц (total) > 0
  — Покрытие источников: использованы ≥2 из 3 (sitemap, GSC, ЯВ)
  — % items с suggested_parent ≥ 60%

Запаздывающий:
  — После Шага 07: все critical/high orphans получили ≥1 входящую ссылку
  — Через 30 дней: рост показов orphan-страниц на ≥15% (Grafana)
  — Через 60 дней: рост кликов orphan-страниц на ≥10%

Контроль:
  — Ревью после Шага 07: suggested_parent корректен в ≥80% случаев?
  — False positive rate: ≤5% (страницы ошибочно отмечены как сироты)
  — False negative rate: проверка выборки 20 URL — все реальные сироты обнаружены?
```

---

## ██ ИСТОРИЯ ИЗМЕНЕНИЙ

| **Версия** | **Дата** | **Изменения** |
| --- | --- | --- |
| v1.0 | 2025-12 | Базовая версия: краулер + sitemap + GSC/ЯВ, rule-based приоритизация, Template 1 |
