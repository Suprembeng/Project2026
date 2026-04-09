# МОДУЛЬ 3: STEP_02.md — Структура сайта

Самодостаточный модуль. Загружается: CORE.md + PROJECT.md + этот файл. ~4K токенов.

---

# ██ КАРТОЧКА ШАГА

| **Параметр**          | **Значение**                                                                 |
|-----------------------|------------------------------------------------------------------------------|
| ID                    | 02                                                                           |
| Название              | Структура сайта (дерево URL, архитектура разделов)                            |
| Блок                  | 1: Подготовительный                                                          |
| Основная роль         | A: SEO-стратег                                                               |
| Консультативные       | B: Инженер автоматизации, E: CRO-аналитик, G: Инженер контрактов данных      |
| Тип                   | Промпт                                                                       |
| HitL                  | ⛔ ДА (ошибка = переделка всего сайта, необратимо)                            |
| Приоритет ROI         | КРИТИЧЕСКИЙ — фундамент навигации, перелинковки, индексации                   |
| Зависит от            | Шаг 01 (Сбор СЯ: кластеры + seed_queries + personas)                        |
| Передаёт в            | Шаг 04 (Структура страниц), Шаг 07 (Перелинковка), Шаг 14 (Robots+Sitemap), Шаг 08 (ТЗ на страницы) |
| Режим Opus            | Аналитика/Стратегия: temperature 0.2, top_p 0.9, max_tokens 8192            |

---

# ██ ЦЕЛЬ

Спроектировать полную архитектуру сайта: иерархию разделов, дерево URL, маппинг кластеров СЯ на страницы, и правила навигации — с учётом кластера ниши, поисковых движков [Я]/[G]/[ОБА] и поведенческих паттернов целевых персон. Результат — утверждённый человеком (HitL) стратегический документ, на основе которого строятся все последующие шаги.

---

# ██ КОНТРАКТ ВХОДНЫХ ДАННЫХ (JSON)

Вход из output contract Шага 01. Минимальные обязательные поля:

```json
{
  "project_id": "string",
  "step_id": "01",
  "niche": "string",
  "niche_cluster": "string",
  "geo": "string",
  "language": "string",
  "priority_engines": "yandex|google|both",
  "domain": "string",
  "cms": "string",
  "cms_spa": "boolean",
  "clusters": [
    {
      "cluster_id": "string (CL001...)",
      "cluster_name": "string",
      "intent": "informational|commercial|transactional|navigational",
      "hunt_level": "number 1-5",
      "page_type_suggestion": "landing|category|article|faq|product",
      "suggested_slug": "string|null",
      "persona_ids": ["string"],
      "engine_tag": "[Я]|[G]|[ОБА]",
      "priority": "critical|high|medium|low",
      "priority_score": "number 0-100",
      "queries": [
        {
          "query": "string",
          "volume_broad": "number|null",
          "engine": "yandex|google|both"
        }
      ]
    }
  ],
  "personas": [
    {
      "persona_id": "string",
      "persona_name": "string",
      "hunt_ladder": "object",
      "search_behavior": "object",
      "content_preferences": "object"
    }
  ],
  "data_quality": "full|partial|minimal",
  "existing_pages": ["string|null"]
}
```

---

# ██ КОНТРАКТ ВЫХОДНЫХ ДАННЫХ (JSON)

Наследует **Шаблон 5: Стратегия**. Роль G валидирует. Должен пройти структурный QC.

```json
{
  "project_id": "string",
  "step_id": "02",
  "prompt_version": "string",
  "timestamp": "ISO 8601",
  "strategy_name": "Архитектура сайта",
  "goals": [
    {
      "goal": "string",
      "kpi": "string",
      "target_value": "string",
      "deadline": "string"
    }
  ],
  "site_tree": [
    {
      "node_id": "string (N001...)",
      "level": "number 0-4",
      "parent_id": "string|null",
      "page_type": "main|section|category|subcategory|landing|article|faq|product|listing|hub|service",
      "title_ru": "string",
      "url_slug": "string",
      "full_url": "string",
      "cluster_ids": ["string"],
      "primary_intent": "informational|commercial|transactional|navigational",
      "hunt_level_range": "string (e.g. 3-5)",
      "persona_ids": ["string"],
      "engine_tag": "[Я]|[G]|[ОБА]",
      "priority": "critical|high|medium|low",
      "seo_notes": "string",
      "children_count": "number",
      "estimated_pages": "number"
    }
  ],
  "url_rules": {
    "pattern": "string",
    "max_depth": "number",
    "trailing_slash": "boolean",
    "transliteration": "string (правило транслитерации)",
    "forbidden_chars": ["string"],
    "examples": ["string"],
    "engine_notes": {
      "yandex": "string",
      "google": "string"
    }
  },
  "navigation": {
    "main_menu": [
      {
        "label": "string",
        "url": "string",
        "children": ["object|null"]
      }
    ],
    "breadcrumbs_pattern": "string",
    "footer_links": ["string"],
    "sidebar_strategy": "string"
  },
  "cluster_to_page_map": [
    {
      "cluster_id": "string",
      "cluster_name": "string",
      "node_id": "string",
      "full_url": "string",
      "mapping_type": "primary|secondary|merged"
    }
  ],
  "orphan_clusters": [
    {
      "cluster_id": "string",
      "reason": "string"
    }
  ],
  "cannibalization_risks": [
    {
      "cluster_ids": ["string"],
      "risk_description": "string",
      "resolution": "string"
    }
  ],
  "tactics": [
    {
      "tactic_id": "string",
      "description": "string",
      "priority": "critical|high|medium|low",
      "engine_tag": "[Я]|[G]|[ОБА]",
      "resources_needed": "string",
      "expected_result": "string",
      "timeline": "string"
    }
  ],
  "risks": [
    {
      "risk": "string",
      "probability": "high|medium|low",
      "mitigation": "string"
    }
  ],
  "summary": {
    "total_nodes": "number",
    "max_depth": "number",
    "total_clusters_mapped": "number",
    "orphan_clusters_count": "number",
    "pages_by_type": {
      "section": "number",
      "category": "number",
      "landing": "number",
      "article": "number",
      "faq": "number",
      "product": "number",
      "other": "number"
    },
    "cannibalization_risks_count": "number"
  },
  "qc_score": "number 0-100"
}
```

---

# ██ ПРОМПТ ДЛЯ CLAUDE OPUS

## Системный промпт

```
Ты — Роль A: Senior SEO-стратег и информационный архитектор с 10+ летним опытом проектирования структур сайтов для Яндекса и Google.

Задача: Спроектировать полную архитектуру сайта на основе кластеров семантического ядра из Шага 01.

ПРАВИЛА:

1. ГЛУБИНА ВЛОЖЕННОСТИ:
   - Максимум 3 клика от главной до конечной страницы (уровни 0-3, исключительно 0-4 для каталогов ecom).
   - [Я] Яндекс штрафует глубокие структуры — держи ≤3.
   - [G] Google допускает до 4, но предпочитает flat architecture.

2. URL-ПАТТЕРНЫ:
   - Транслитерация: ГОСТ 7.79-2000 (Система Б) для ru, latin slug для en.
   - [Я] Яндекс хорошо читает кириллические URL, но транслит предпочтительнее для совместимости.
   - [G] Google предпочитает короткие latin slugs.
   - Без дублирования ключей в URL (domain.ru/stomatologiya/detskaya-stomatologiya — плохо).
   - Trailing slash: единый стандарт, указать в url_rules.
   - Максимальная длина URL: 80 символов (без домена).

3. МАППИНГ КЛАСТЕРОВ:
   - Каждый кластер из Шага 01 ОБЯЗАТЕЛЬНО привязан к ноде site_tree (mapping_type: primary).
   - Если 2+ кластера лучше объединить на одной странице — mapping_type: merged, указать причину.
   - Если кластер не подходит ни к одной странице — orphan_clusters с reason.
   - ЗАПРЕЩЕНО: один кластер → несколько страниц (каннибализация). Исключение: regional landing pages с разным гео.

4. ТИПЫ СТРАНИЦ по niche_cluster:
   - commercial_local: главная + услуги + районы/филиалы + блог + отзывы + контакты
   - commercial_national: главная + каталог (категории/подкатегории) + услуги + блог + о компании
   - ecom: главная + каталог (категории/подкатегории/фильтры) + карточки товаров + бренды + блог
   - saas: главная + продукт (фичи/pricing/integrations) + use cases + blog + docs + comparisons
   - info: главная + хабы (topical authority) + статьи + FAQ + about
   - medical: главная + услуги + врачи + заболевания (info hub) + отзывы + лицензии
   - legal: главная + услуги + кейсы + статьи (info hub) + консультация + о компании
   - education: главная + курсы (каталог) + программы + блог + FAQ + отзывы

5. НАВИГАЦИЯ:
   - Главное меню: ≤7 пунктов первого уровня.
   - [Я] Яндекс: меню с текстовыми ссылками (не JS-only).
   - [G] Google: breadcrumbs обязательны, Schema BreadcrumbList.
   - Каждая страница доступна из ≥2 путей (breadcrumbs + меню + перелинковка).

6. SPA-АДАПТАЦИЯ:
   - Если cms_spa=true: каждый раздел ДОЛЖЕН иметь статический URL, SSR/prerender обязателен.
   - Пометить в seo_notes ноды, требующие SSR.

7. КАННИБАЛИЗАЦИЯ:
   - Проверь все кластеры с intent=commercial на пересечение по ключевым запросам.
   - Если 2 кластера имеют ≥30% общих запросов — объединить или разнести по интентам.
   - Каждый риск каннибализации — в cannibalization_risks с resolution.

8. ПЕРСОНЫ:
   - Каждая персона из Шага 00 должна иметь ≥3 нод в site_tree, покрывающих её customer journey (awareness → decision).
   - hunt_level_range в ноде = диапазон уровней Ханта кластеров, привязанных к ноде.

9. ПРИОРИТИЗАЦИЯ:
   - critical: страницы с кластерами priority=critical из Шага 01 (создаются первыми).
   - high: страницы с кластерами priority=high.
   - medium/low: остальные.

10. ТАКТИКИ:
    - ≥3 тактики по созданию структуры.
    - Каждая с engine_tag, timeline и expected_result.
    - Первая тактика — всегда «Создание приоритетных страниц» (critical nodes).

11. РИСКИ:
    - ≥2 риска (например: слишком глубокая вложенность, каннибализация, SPA без SSR).
    - Каждый с mitigation.

ВЫВОД: Валидный JSON по контракту. Без markdown. Без комментариев. Весь текст — на РУССКОМ языке.
```

## Пользовательский промпт (Шаблон)

```
Спроектируй архитектуру сайта:

Проект: {{project.project_id}}
Ниша: {{project.niche}}
Кластер: {{project.niche_cluster}}
Гео: {{project.geo}}
Язык: {{project.language}}
Приоритет ПС: {{project.priority_engines}}
Домен: {{project.domain}}
CMS: {{project.cms}}
SPA: {{project.cms_spa}}
Data quality: {{step_01.data_quality}}

Кластеры из Шага 01 ({{clusters_count}} кластеров):
{{clusters_json}}

Персоны из Шага 00 ({{personas_count}} персон):
{{personas_summary_json}}

{{#if existing_pages}}
Существующие страницы сайта:
{{existing_pages_json}}
{{/if}}

{{#if rag_context}}
Контекст из RAG (структуры конкурентов):
{{rag_context}}
{{/if}}
```

---

# ██ RAG-ОБОГАЩЕНИЕ

| Коллекция Qdrant      | Запрос                                      | Назначение                                      |
|-----------------------|---------------------------------------------|--------------------------------------------------|
| competitors           | niche + geo + "site structure"              | Структуры сайтов конкурентов (разделы, глубина)  |
| clusters              | project_id + step_id=01                     | Кластеры текущего проекта для маппинга           |
| site_structures       | niche_cluster + "architecture"              | Шаблоны структур для аналогичных кластеров       |

Лимит RAG-контекста: 2000 токенов.

---

# ██ СКРИПТЫ CLAUDE CODE

## Пред-шаг: Парсинг структур конкурентов

```python
# step_02_pre_competitor_structure.py
# Назначение: Извлечение структуры URL конкурентов → паттерны архитектуры
# Вход: project.main_competitors (из PROJECT.md)
# Выход: competitor_structures.json
# Действия:
# 1. Каждый конкурент: Screaming Frog API / sitemap.xml → дерево URL
# 2. Извлечь: глубину, количество уровней, типы страниц, паттерны URL
# 3. Агрегировать: общие паттерны среди ≥2 конкурентов
# 4. Сохранить в Qdrant 'competitors' с тегом 'site_structure'
```

## Пред-шаг: Текущие страницы сайта

```python
# step_02_pre_existing_pages.py
# Назначение: Сбор текущих страниц сайта из sitemap/crawl
# Вход: project.domain
# Выход: existing_pages.json
# Действия:
# 1. Fetch sitemap.xml → список URL
# 2. Fallback: GSC API → indexed pages
# 3. Каждая страница: URL, title, status code, canonical
# 4. Передать в input contract как existing_pages[]
```

## Пост-шаг: Валидатор

```python
# step_02_post_validate.py
# Назначение: Валидация по JSON-схеме + SEO-правила
# Проверки:
# 1. JSON Schema — все обязательные поля
# 2. Все кластеры из Шага 01 имеют маппинг (cluster_to_page_map + orphan_clusters = 100%)
# 3. Максимальная глубина site_tree ≤ 4
# 4. URL-slugs уникальны
# 5. URL-slugs соответствуют url_rules.pattern
# 6. Главное меню ≤ 7 пунктов
# 7. Каждая персона покрыта ≥ 3 нодами
# 8. Нет дублей node_id
# 9. parent_id ссылается на существующий node_id (или null для корня)
# 10. engine_tag присутствует на всех нодах
# 11. cannibalization_risks не пуст при наличии merged кластеров
# 12. tactics ≥ 3
# 13. risks ≥ 2
# 14. Каждый URL ≤ 80 символов (без домена)
```

## Пост-шаг: Qdrant

```python
# step_02_post_qdrant.py
# Назначение: Сохранение site_tree в Qdrant для RAG следующих шагов
# Действия:
# 1. Эмбеддинг (title_ru + url_slug + seo_notes)
# 2. Upsert в 'site_structures'
# 3. Тег project_id + step_id=02
```

---

# ██ КРИТЕРИИ QC

| **Проверка**                             | **Порог**                    | **Вес** | **При провале** |
|------------------------------------------|------------------------------|---------|-----------------|
| JSON Schema валидна                      | Pass/Fail                    | 10      | Retry           |
| 100% кластеров имеют маппинг            | orphan ≤ 5%                  | 15      | Retry           |
| Глубина site_tree ≤ 4                    | max_depth ≤ 4                | 10      | Retry           |
| URL-slugs уникальны                     | 100%                         | 10      | Retry           |
| URL длина ≤ 80 символов                  | 100%                         | 5       | Retry           |
| Меню ≤ 7 пунктов                        | ≤ 7                          | 5       | Ревью           |
| Персоны покрыты ≥3 нод                  | все персоны                  | 10      | Ревью           |
| engine_tag на всех нодах                | 100%                         | 8       | Retry           |
| parent_id корректны                     | 100%                         | 7       | Retry           |
| Тактики ≥ 3                            | ≥ 3                          | 5       | Retry           |
| Риски ≥ 2                              | ≥ 2                          | 5       | Retry           |
| cannibalization_risks при merged         | не пуст если merged > 0     | 5       | Ревью           |
| Нет дублей node_id                      | 100%                         | 5       | Retry           |

QC = сумма весов. ≥80 → HitL (утверждение), 60-79 → ревью, <60 → retry.

**ВАЖНО:** После прохождения QC шаг НЕ завершается автоматически. Результат отправляется на утверждение человеку (HitL). Только после «Утверждено» → передача в Шаги 04, 07, 08, 14.

---

# ██ СТРУКТУРА N8N-ВОРКФЛОУ

```
Нода 1:  Триггер (из Шага 01 или ручной запуск)
Нода 2:  NocoDB: загрузка result Шага 01 (clusters + personas)
Нода 2b: NocoDB: загрузка PROJECT.md параметров
Нода 3a: step_02_pre_competitor_structure.py (параллельно)
Нода 3b: step_02_pre_existing_pages.py (параллельно)
Нода 4:  Qdrant RAG: структуры конкурентов + шаблоны
Нода 5:  Merge: объединение данных (clusters + personas + competitors + existing + RAG)
Нода 6:  Build Prompt: сборка system + user prompt
Нода 7:  Claude Opus API (temp 0.2, top_p 0.9, max_tokens 8192)
Нода 8:  Parse JSON: извлечение и парсинг ответа
Нода 9:  step_02_post_validate.py → провал → Нода 9r (макс. 3 retry)
Нода 9r: Prepare Retry: switch(retry_count) → стратегия
Нода 10: step_02_post_qdrant.py: сохранение site_tree в Qdrant
Нода 11: NocoDB: сохранение результата + prompt_version
Нода 12: Grafana: push метрик (duration, qc_score, node_count)
Нода 13: HitL Gate: Telegram/Email → «Утвердить» / «Запросить изменения»
Нода 14: IF утверждено → Шаг 04, 07, 08, 14 | ELSE → Нода 6 с комментариями
```

## Обработка ошибок

- Opus 429/500: бэкофф 10с→30с→90с, макс. 3
- Qdrant недоступен: без RAG, лог предупреждения
- QC провал 3x: алерт Telegram, сохранение partial result
- HitL «Запросить изменения»: комментарии → инъекция в промпт → повторный вызов Opus

---

# ██ ПРИМЕР FEW-SHOT

```json
{
  "project_id": "dental_moscow_001",
  "step_id": "02",
  "prompt_version": "step02_v1.0",
  "timestamp": "2025-12-15T10:30:00Z",
  "strategy_name": "Архитектура сайта",
  "goals": [
    {
      "goal": "Создать плоскую структуру с глубиной ≤3 клика",
      "kpi": "max_depth",
      "target_value": "3",
      "deadline": "2 недели"
    },
    {
      "goal": "100% кластеров СЯ привязаны к страницам",
      "kpi": "cluster_mapping_rate",
      "target_value": "100%",
      "deadline": "2 недели"
    }
  ],
  "site_tree": [
    {
      "node_id": "N001",
      "level": 0,
      "parent_id": null,
      "page_type": "main",
      "title_ru": "Главная — Стоматология «Улыбка» в Москве",
      "url_slug": "",
      "full_url": "/",
      "cluster_ids": ["CL003"],
      "primary_intent": "navigational",
      "hunt_level_range": "5",
      "persona_ids": ["P001", "P002", "P003"],
      "engine_tag": "[ОБА]",
      "priority": "critical",
      "seo_notes": "Главная: бренд + гео + основная услуга. [Я] ИКС-сигнал. [G] Sitelinks.",
      "children_count": 5,
      "estimated_pages": 1
    },
    {
      "node_id": "N002",
      "level": 1,
      "parent_id": "N001",
      "page_type": "section",
      "title_ru": "Услуги стоматологии",
      "url_slug": "uslugi",
      "full_url": "/uslugi/",
      "cluster_ids": [],
      "primary_intent": "commercial",
      "hunt_level_range": "3-4",
      "persona_ids": ["P001", "P002"],
      "engine_tag": "[ОБА]",
      "priority": "critical",
      "seo_notes": "Hub-страница услуг. Перелинковка на все подуслуги. [Я] КФ: прайс-лист.",
      "children_count": 6,
      "estimated_pages": 1
    },
    {
      "node_id": "N003",
      "level": 2,
      "parent_id": "N002",
      "page_type": "landing",
      "title_ru": "Детская стоматология в Москве",
      "url_slug": "detskaya",
      "full_url": "/uslugi/detskaya/",
      "cluster_ids": ["CL001"],
      "primary_intent": "commercial",
      "hunt_level_range": "4",
      "persona_ids": ["P001"],
      "engine_tag": "[ОБА]",
      "priority": "critical",
      "seo_notes": "Landing: CL001 (priority 88). Quick win — позиция 15. [Я] Гео-привязка в title. [G] FAQ Schema.",
      "children_count": 0,
      "estimated_pages": 1
    },
    {
      "node_id": "N010",
      "level": 1,
      "parent_id": "N001",
      "page_type": "hub",
      "title_ru": "Полезные статьи о стоматологии",
      "url_slug": "blog",
      "full_url": "/blog/",
      "cluster_ids": [],
      "primary_intent": "informational",
      "hunt_level_range": "1-3",
      "persona_ids": ["P001", "P002", "P003"],
      "engine_tag": "[ОБА]",
      "priority": "high",
      "seo_notes": "Контент-хаб для Topical Authority. [G] Discover-eligible. [Я] ПФ: время на сайте.",
      "children_count": 8,
      "estimated_pages": 1
    },
    {
      "node_id": "N011",
      "level": 2,
      "parent_id": "N010",
      "page_type": "article",
      "title_ru": "Когда вести ребёнка к стоматологу",
      "url_slug": "kogda-vesti-rebyonka",
      "full_url": "/blog/kogda-vesti-rebyonka/",
      "cluster_ids": ["CL002"],
      "primary_intent": "informational",
      "hunt_level_range": "1",
      "persona_ids": ["P001"],
      "engine_tag": "[ОБА]",
      "priority": "medium",
      "seo_notes": "Awareness-контент, hunt_level 1. ai_search_relevant=true. [G] AEO: Featured Snippet target.",
      "children_count": 0,
      "estimated_pages": 1
    }
  ],
  "url_rules": {
    "pattern": "/{section}/{subsection}/",
    "max_depth": 3,
    "trailing_slash": true,
    "transliteration": "ГОСТ 7.79-2000 Система Б (ш→sh, щ→shch, ё→yo, ъ→, ь→)",
    "forbidden_chars": ["_", ".", "CAPS", "кириллица"],
    "examples": [
      "/uslugi/detskaya/",
      "/blog/kogda-vesti-rebyonka/",
      "/otzyvy/"
    ],
    "engine_notes": {
      "yandex": "Яндекс индексирует кириллические URL, но транслит предпочтительнее для шеринга и QR-кодов.",
      "google": "Google предпочитает короткие ASCII slugs. Избегать параметров (?page=) в пользу ЧПУ."
    }
  },
  "navigation": {
    "main_menu": [
      { "label": "Услуги", "url": "/uslugi/", "children": [
        { "label": "Детская стоматология", "url": "/uslugi/detskaya/" },
        { "label": "Имплантация", "url": "/uslugi/implantaciya/" }
      ]},
      { "label": "Цены", "url": "/ceny/", "children": null },
      { "label": "Врачи", "url": "/vrachi/", "children": null },
      { "label": "Блог", "url": "/blog/", "children": null },
      { "label": "Отзывы", "url": "/otzyvy/", "children": null },
      { "label": "Контакты", "url": "/kontakty/", "children": null }
    ],
    "breadcrumbs_pattern": "Главная > {section} > {page}",
    "footer_links": ["/uslugi/", "/ceny/", "/vrachi/", "/blog/", "/otzyvy/", "/kontakty/", "/politika-konfidencialnosti/"],
    "sidebar_strategy": "Блог: категории + популярные статьи. Услуги: смежные услуги + CTA запись."
  },
  "cluster_to_page_map": [
    { "cluster_id": "CL001", "cluster_name": "детская стоматология цены", "node_id": "N003", "full_url": "/uslugi/detskaya/", "mapping_type": "primary" },
    { "cluster_id": "CL002", "cluster_name": "когда вести ребёнка к стоматологу", "node_id": "N011", "full_url": "/blog/kogda-vesti-rebyonka/", "mapping_type": "primary" },
    { "cluster_id": "CL003", "cluster_name": "стоматология официальный сайт", "node_id": "N001", "full_url": "/", "mapping_type": "primary" }
  ],
  "orphan_clusters": [],
  "cannibalization_risks": [
    {
      "cluster_ids": ["CL001", "CL007"],
      "risk_description": "Кластеры 'детская стоматология цены' и 'лечение зубов детям стоимость' пересекаются на 35% по запросам",
      "resolution": "Объединить на странице /uslugi/detskaya/ с разделением по H2: 'Цены' и 'Виды лечения'"
    }
  ],
  "tactics": [
    {
      "tactic_id": "T001",
      "description": "Создать приоритетные страницы (critical nodes): главная, услуги, детская стоматология, цены",
      "priority": "critical",
      "engine_tag": "[ОБА]",
      "resources_needed": "Контент-менеджер, SEO-специалист",
      "expected_result": "Базовая структура с основными landing pages, готовая к индексации",
      "timeline": "Неделя 1-2"
    },
    {
      "tactic_id": "T002",
      "description": "Настроить навигацию, breadcrumbs, Schema BreadcrumbList",
      "priority": "high",
      "engine_tag": "[G]",
      "resources_needed": "Разработчик",
      "expected_result": "Rich results breadcrumbs в Google, улучшение CTR на 5-10%",
      "timeline": "Неделя 2-3"
    },
    {
      "tactic_id": "T003",
      "description": "Развернуть контент-хаб (блог) с категориями по hunt_level",
      "priority": "high",
      "engine_tag": "[ОБА]",
      "resources_needed": "Контент-менеджер, копирайтер",
      "expected_result": "Topical Authority по нише, покрытие awareness-запросов (hunt 1-2)",
      "timeline": "Неделя 3-6"
    }
  ],
  "risks": [
    {
      "risk": "Каннибализация между landing pages услуг и статьями блога по одинаковым запросам",
      "probability": "medium",
      "mitigation": "Разнесение по интентам: landing = commercial, статья = informational. Canonical при пересечении."
    },
    {
      "risk": "Слишком глубокая вложенность при масштабировании каталога",
      "probability": "low",
      "mitigation": "Максимум 3 уровня. При росте — flat categories с фильтрами вместо подкатегорий."
    }
  ],
  "summary": {
    "total_nodes": 18,
    "max_depth": 2,
    "total_clusters_mapped": 15,
    "orphan_clusters_count": 0,
    "pages_by_type": {
      "section": 2,
      "category": 0,
      "landing": 6,
      "article": 8,
      "faq": 1,
      "product": 0,
      "other": 1
    },
    "cannibalization_risks_count": 1
  },
  "qc_score": 92
}
```

---

# ██ СТРАТЕГИЯ ОТКАТА

| Откат | Описание |
|-------|----------|
| **Откат A** | Декомпозиция на 2 вызова: (1) site_tree + url_rules, (2) cluster_to_page_map + navigation + tactics. Объединить Code-нодой. |
| **Откат B** | Сократить до top-20 кластеров по priority_score. Пометка в NocoDB для ручного расширения оставшихся. |
| **Откат C** | Few-shot как жёсткий шаблон. Opus адаптирует page_type и URL-паттерны под niche_cluster. |

---

# ██ АДАПТАЦИИ ПОД КЛАСТЕР

| Кластер             | Адаптация структуры                                                                                  |
|---------------------|-------------------------------------------------------------------------------------------------------|
| commercial_local    | Разделы по районам/филиалам. Страницы «услуга + район». URL: /uslugi/detskaya/metro-taganskaya/. [Я] Гео-привязка обязательна. |
| commercial_national | Каталог с категориями/подкатегориями. Landing pages по сегментам ЦА. Programmatic potential.          |
| ecom                | Каталог: категории → подкатегории → карточки. Фильтры как отдельные URL (не JS-only). Пагинация rel=prev/next. [G] Product Schema. |
| saas                | /product/, /pricing/, /integrations/, /vs/ (сравнения), /use-cases/. Мультиязычность: hreflang.       |
| info                | Topical Authority hubs: /hub/topic/ → /hub/topic/article/. Pillar page + cluster articles.            |
| medical             | /uslugi/, /zabolevaniya/ (info hub), /vrachi/ (E-E-A-T). YMYL: лицензии в footer на каждой странице.  |
| legal               | /uslugi/, /keysy/, /stati/ (info hub). Срочность: /srochnaya-konsultaciya/ в меню.                    |
| education           | /kursy/ (каталог), /programmy/, /otzyvy/. Сезонность: landing по наборам.                             |

---

# ██ МЕТРИКИ УСПЕХА

| Тип          | Метрика                                                                                   |
|--------------|-------------------------------------------------------------------------------------------|
| Опережающий  | 100% кластеров Шага 01 имеют маппинг? URL-slugs соответствуют правилам?                   |
| Запаздывающий | После Шага 04 — все страницы имеют уникальные title/H1 без пересечений?                   |
| Контроль     | После HitL — человек подтвердил структуру? Количество итераций «Запросить изменения» ≤ 2? |

---

# ██ ИСТОРИЯ ИЗМЕНЕНИЙ

| Версия | Дата      | Изменения                                                                                  |
|--------|-----------|--------------------------------------------------------------------------------------------|
| v1.0   | 2025-01   | Базовая структура: site_tree, URL-правила, маппинг кластеров                               |
| v2.0   | 2025-06   | Добавлены: navigation, cannibalization_risks, persona coverage, engine_tag на нодах         |
| v3.0   | 2025-12   | SPA-адаптация, RAG-обогащение (конкуренты), QC расширен до 13 проверок, адаптации по кластерам, тактики |
