# STEP_07.md — Перелинковка (v1.0)

Самодостаточный модуль. Загружается: CORE.md + PROJECT.md + этот файл. ~4.5K токенов.

---

## ██ КАРТОЧКА ШАГА

| **Параметр** | **Значение** |
| --- | --- |
| ID | 07 |
| Название | Стратегия внутренней перелинковки |
| Блок | 2: Структура и контент |
| Основная роль | A: SEO-стратег |
| Консультативные | E: CRO-аналитик, B: Инженер автоматизации |
| Тип | Промпт+Скрипт |
| HitL | Нет (результат → Шаги 08, 14 автоматически) |
| Приоритет ROI | ВЫСОКИЙ — перелинковка распределяет PageRank, ускоряет индексацию, снижает orphan-процент |
| Зависит от | Шаг 04 (Структура страниц — URL + типы), Шаг 04.2 (Сиротские страницы — orphan_urls + suggested_parent), Шаг 01 (Семантика — кластеры запросов), PROJECT.md |
| Передаёт в | Шаг 08 (ТЗ — требования к блокам перелинковки), Шаг 14 (Robots + Sitemap — приоритеты краулинга) |
| Режим Opus | JSON-вывод: temperature 0.2, top_p 0.9, max_tokens 8192 |
| Prompt version | step07_v1.0 |

---

## ██ ЦЕЛЬ

Разработать стратегию внутренней перелинковки: определить hub-страницы, спроектировать тематические кластеры ссылок, назначить anchor-тексты по URL-парам, интегрировать сиротские страницы (Шаг 04.2) в структуру, запланировать breadcrumbs и контекстные блоки — с учётом специфики Яндекса ([Я]: глубина вложенности, ПФ, ИКС) и Google ([G]: PageRank flow, crawl budget, topical authority) — для передачи конкретных правил перелинковки в ТЗ (Шаг 08).

---

## ██ КОНТРАКТ ВХОДНЫХ ДАННЫХ (JSON)

Вход формируется из output Шагов 04, 04.2, 01 + PROJECT.md + результаты pre-scripts.

```json
{
  "project_id": "string",
  "domain": "string",
  "niche_cluster": "string",
  "priority_engines": "yandex|google|both",
  "cms": "string",
  "step_04_output": {
    "pages": [
      { "url": "string", "page_type": "string", "title": "string", "h1": "string" }
    ]
  },
  "step_04_2_output": {
    "items": [
      {
        "id": "string",
        "title": "string (URL)",
        "priority": "string",
        "action_required": "string",
        "suggested_parent": "string|null",
        "anchor_suggestion": "string|null"
      }
    ]
  },
  "step_01_clusters": [
    {
      "cluster_id": "string",
      "cluster_name": "string",
      "queries": ["string"],
      "target_url": "string|null"
    }
  ],
  "current_links_graph": {
    "nodes": [
      { "url": "string", "page_type": "string", "inbound_count": "number", "outbound_count": "number", "depth": "number" }
    ],
    "edges": [
      { "from": "string", "to": "string", "anchor": "string", "location": "nav|footer|content|sidebar|breadcrumb" }
    ],
    "stats": {
      "total_internal_links": "number",
      "avg_inbound_per_page": "number",
      "avg_outbound_per_page": "number",
      "max_depth": "number",
      "orphan_count": "number"
    }
  }
}
```

**Обязательные:** `project_id`, `domain`, `niche_cluster`, `priority_engines`, `step_04_output`, `current_links_graph`.

**Опциональные:** `step_04_2_output` (при отсутствии — без orphan-интеграции), `step_01_clusters` (при отсутствии — кластеризация по URL-иерархии).

---

## ██ КОНТРАКТ ВЫХОДНЫХ ДАННЫХ (JSON)

Наследует **Шаблон 5: Стратегия**. Роль G валидирует.

```json
{
  "project_id": "string",
  "step_id": "07",
  "prompt_version": "step07_v1.0",
  "timestamp": "ISO 8601",
  "strategy_name": "Стратегия внутренней перелинковки",
  "goals": [
    {
      "goal": "string",
      "kpi": "string",
      "target_value": "string",
      "deadline": "string"
    }
  ],
  "tactics": [
    {
      "tactic_id": "string (IL001, IL002...)",
      "description": "string",
      "priority": "critical|high|medium|low",
      "engine_tag": "[Я]|[G]|[ОБА]",
      "resources_needed": "string",
      "expected_result": "string",
      "timeline": "string",
      "tactic_type": "hub_spoke|silo|breadcrumb|contextual|footer|sidebar|orphan_fix|anchor_optimization",
      "affected_page_types": ["string"],
      "link_rules": [
        {
          "from_url_pattern": "string",
          "to_url_pattern": "string",
          "anchor_template": "string",
          "location": "content|sidebar|footer|breadcrumb|related_block",
          "max_links_per_page": "number",
          "nofollow": "boolean"
        }
      ],
      "cross_step_hooks": {
        "to_step_08_tz": "boolean",
        "to_step_14_sitemap": "boolean"
      }
    }
  ],
  "risks": [
    {
      "risk": "string",
      "probability": "high|medium|low",
      "mitigation": "string"
    }
  ],
  "hub_pages": [
    {
      "url": "string",
      "role": "main_hub|category_hub|topic_hub",
      "spoke_urls": ["string"],
      "target_inbound": "number",
      "current_inbound": "number"
    }
  ],
  "orphan_integration_plan": [
    {
      "orphan_url": "string",
      "parent_url": "string",
      "anchor_text": "string",
      "integration_method": "content_link|related_block|breadcrumb|sidebar|nav"
    }
  ],
  "anchor_strategy": {
    "exact_match_ratio": "number (0-100)",
    "partial_match_ratio": "number (0-100)",
    "branded_ratio": "number (0-100)",
    "generic_ratio": "number (0-100)",
    "anchor_templates": [
      { "page_type_from": "string", "page_type_to": "string", "template": "string", "engine_tag": "[Я]|[G]|[ОБА]" }
    ]
  },
  "depth_optimization": {
    "current_max_depth": "number",
    "target_max_depth": "number",
    "pages_to_elevate": [
      { "url": "string", "current_depth": "number", "target_depth": "number", "method": "string" }
    ]
  },
  "implementation_roadmap": [
    {
      "phase": "number",
      "phase_name": "string",
      "tactics_ids": ["string"],
      "timeline": "string",
      "dependencies": ["string"]
    }
  ],
  "qc_score": "0-100"
}
```

**Правила:**
- Каждый `tactics[].engine_tag` обязателен.
- `link_rules` — конкретные правила для автоматизации в CMS (Шаг 08).
- `hub_pages` — не более 10 (ключевые хабы).
- `orphan_integration_plan` — все orphan из step_04_2 с action_required=link_to.
- `anchor_strategy` — пропорции суммируются до 100%.
- `implementation_roadmap` — фазы с зависимостями для управления проектом.

---

## ██ ПРОМПТ ДЛЯ CLAUDE OPUS

**Системный промпт**

```
Ты — Роль A: Senior SEO-стратег (10+ лет, Яндекс и Google), эксперт по внутренней перелинковке.
Консультанты: Роль E (CRO/поведение), Роль B (автоматизация, скрипты).
Prompt version: step07_v1.0

CoT ОБЯЗАТЕЛЕН:
[АНАЛИЗ]   Текущий граф перелинковки: глубина, orphan, распределение PageRank.
[РОЛИ]     A принимает; E консультирует по UX-перелинковке; B — по автоматизации.
[КОНТЕКСТ] niche_cluster, CMS, priority_engines.
[КОНТРАКТ] JSON output по Template 5 (Strategy).
[БЕНЧМАРК] Стандарты перелинковки для кластера (hub-spoke, silo, flat).
[РЕШЕНИЕ]  Тактики с link_rules, anchor_templates, hub-страницы, orphan-план.
[РИСКИ]    Каннибализация анкоров, перегрузка ссылками, размытие PageRank.
[HitL]     Не требуется.

ПРАВИЛА:
1. Тактики покрывают ВСЕ типы: hub-spoke, silo, breadcrumb, contextual, orphan_fix, anchor_optimization.
2. engine_tag ∈ {[Я],[G],[ОБА]}; при priority_engines=both → оба обязательно.
3. Специфика [Я]:
   - Глубина вложенности ≤3 клика от главной (критично для индексации Яндекса)
   - ПФ: ссылки должны быть UX-релевантны (Яндекс учитывает клики по ссылкам)
   - ИКС: перелинковка улучшает качество сайта
   - Хлебные крошки: обязательны, помогают роботу Яндекса
   - rel=nofollow внутренние: Яндекс НЕ передаёт вес (в отличие от Google)
4. Специфика [G]:
   - PageRank flow: распределение через hub-spoke модель
   - Crawl budget: приоритизация важных страниц через ссылочную плотность
   - Topical Authority: silo-структура усиливает тематические кластеры
   - Breadcrumbs: Schema BreadcrumbList для rich snippets
   - Internal nofollow: Google рекомендует НЕ использовать (PageRank sculpting не работает)
5. link_rules — конкретные правила для CMS: from_pattern → to_pattern, anchor_template, location, max_links.
6. hub_pages — не более 10, с target_inbound (сколько ссылок нужно).
7. orphan_integration_plan — ВСЕ orphan с action_required=link_to из Шага 04.2.
8. anchor_strategy: exact_match + partial_match + branded + generic = 100%.
9. Для [Я]: exact_match_ratio ≤ 30% (риск переспама анкоров в Яндексе).
10. Для [G]: exact_match_ratio ≤ 20% (Google строже к anchor-манипуляциям).
11. depth_optimization: target_max_depth ≤ 3 для commercial, ≤ 4 для info.
12. implementation_roadmap: минимум 2 фазы (быстрые победы + системная перелинковка).
13. Не создавать ссылки на orphan-страницы с action_required=deindex|delete.
14. max_links_per_page: ≤ 100 для content-страниц, ≤ 200 для категорий.
15. cross_step_hooks: to_step_08_tz=true для тактик, требующих изменения шаблонов CMS.

АНТИПАТТЕРНЫ:
✗ «Улучшить перелинковку» без конкретных link_rules
✗ Одинаковый anchor для всех ссылок на одну страницу
✗ Ссылки из footer на все страницы (link farm паттерн)
✗ Без engine_tag
✗ Circular linking (A→B→C→A без хаба)
✗ anchor_ratios не суммируются до 100%
✗ Ссылки на deindex/delete orphans

ВЫВОД: Валидный JSON по Template 5 (Strategy). Без markdown. На русском.
```

**Пользовательский промпт (шаблон)**

```
Разработай стратегию внутренней перелинковки.

Проект: {{project.project_id}} | Домен: {{project.domain}}
Кластер: {{project.niche_cluster}} | ПС: {{project.priority_engines}}
CMS: {{project.cms}}

ТЕКУЩИЙ ГРАФ ПЕРЕЛИНКОВКИ:
Всего ссылок: {{current_links_graph.stats.total_internal_links}}
Средний inbound: {{current_links_graph.stats.avg_inbound_per_page}}
Средний outbound: {{current_links_graph.stats.avg_outbound_per_page}}
Макс. глубина: {{current_links_graph.stats.max_depth}}
Orphan-страниц: {{current_links_graph.stats.orphan_count}}

ТОП-20 страниц по inbound (хабы-кандидаты):
{{top_inbound_pages | json}}

Страницы с 0 inbound (orphan):
{{zero_inbound_pages | json}}

ТИПЫ СТРАНИЦ (Шаг 04):
{{step_04_output.pages | json}}

{{#if step_04_2_output}}
СИРОТСКИЕ СТРАНИЦЫ ДЛЯ ИНТЕГРАЦИИ (Шаг 04.2, action=link_to):
{{orphan_link_to_items | json}}
{{/if}}

{{#if step_01_clusters}}
СЕМАНТИЧЕСКИЕ КЛАСТЕРЫ (Шаг 01):
{{step_01_clusters | json}}
{{/if}}

{{#if rag_context}}
RAG КОНТЕКСТ:
{{rag_context}}
{{/if}}

Prompt version: step07_v1.0
```

---

## ██ RAG-ОБОГАЩЕНИЕ

| **Коллекция Qdrant** | **Запрос** | **Назначение** |
| --- | --- | --- |
| `site_structure` | `project_id` | Текущая иерархия URL, метаданные страниц |
| `orphan_pages` | `project_id` | Данные из Шага 04.2 (suggested_parent, anchor) |
| `competitors` | `niche_cluster` + «перелинковка silo hub» | Паттерны перелинковки конкурентов |
| `niches` | `niche_cluster` + «internal linking best practices» | Стандарты перелинковки для кластера |
| `personas` | `project_id` | Навигационные ожидания ЦА |

Лимит RAG-контекста: 2000 токенов.

---

## ██ СКРИПТЫ CLAUDE CODE

### Пред-шаг 1: Построение графа перелинковки

```
# step_07_pre_links_graph.py
# Назначение: Построение графа внутренних ссылок для анализа
# Вход: project.domain, step_04_output.pages[]
# Выход: current_links_graph.json
# Действия:
# 1. Краулинг всех страниц из step_04 (макс. 500 URL, приоритет по page_type)
# 2. Извлечение внутренних ссылок: <a href>, anchor-текст, location (nav/footer/content/sidebar/breadcrumb)
# 3. Построение графа: nodes[] + edges[]
# 4. Расчёт метрик для каждого node:
#    - inbound_count: количество входящих внутренних ссылок
#    - outbound_count: количество исходящих внутренних ссылок
#    - depth: минимальное расстояние от главной (BFS)
# 5. Агрегированные stats: total_links, avg_inbound, avg_outbound, max_depth, orphan_count
# 6. Top-20 по inbound (hub-кандидаты)
# 7. Страницы с 0 inbound (orphans из графа)
# 8. Нормализация URL (https, lowercase, no trailing slash)
#
# Ограничения: robots.txt, delay 200мс, asyncio semaphore=10
# Таймаут: 30с/URL, общий лимит 15 минут
# Fallback: при частичном краулинге → partial graph, coverage_pct < 100
```

### Пред-шаг 2: Подготовка orphan-данных

```
# step_07_pre_orphan_prep.py
# Назначение: Фильтрация и подготовка orphan-страниц для интеграции
# Вход: step_04_2_output.items[]
# Выход: orphan_link_to_items[]
# Действия:
# 1. Фильтр: только items с action_required = "link_to"
# 2. Для каждого: orphan_url, suggested_parent, anchor_suggestion
# 3. Валидация: suggested_parent существует в step_04_output.pages[]
# 4. Если suggested_parent = null → определить по URL-иерархии
# 5. Выход: массив для инъекции в промпт
```

### Пост-шаг: Валидатор

```
# step_07_post_validate.py
# Проверки:
# 1. JSON Schema — Template 5 (Strategy) [hard]
# 2. Все tactics.tactic_id уникальны, формат IL### [hard]
# 3. engine_tag ∈ {[Я],[G],[ОБА]} 100% [hard]
# 4. priority_engines=both → [Я] И [G] присутствуют [hard]
# 5. goals ≥ 2 [hard]
# 6. tactics ≥ 3 [hard]
# 7. risks ≥ 2 [hard]
# 8. anchor_strategy ratios sum = 100 (±2) [hard]
# 9. anchor_strategy: exact_match_ratio ≤ 30 [hard]
# 10. hub_pages ≤ 10 [hard]
# 11. hub_pages[].url ∈ step_04_output.pages[].url [hard]
# 12. orphan_integration_plan покрывает ≥ 80% orphan с action=link_to [hard]
# 13. orphan_integration_plan[].parent_url ∈ step_04_output.pages[].url [hard]
# 14. Нет ссылок на orphan с action=deindex|delete [hard]
# 15. link_rules[].max_links_per_page ≤ 200 [hard]
# 16. depth_optimization.target_max_depth ≤ 4 [hard]
# 17. implementation_roadmap ≥ 2 фаз [hard]
# 18. Нет дублей tactic_id [hard]
# 19. cross_step_hooks заполнены у тактик с link_rules [soft]
# 20. tactic_type покрывает ≥ 4 из 8 типов [soft]
```

### Пост-шаг: NocoDB + Qdrant

```
# step_07_post_store.py
# 1. Idempotency: sha256(project_id + step_id + input_hash)
# 2. NocoDB INSERT interlinking_strategy (project_id, output_json, qc_score)
# 3. NocoDB UPDATE step_results (step_id=07, status=done)
# 4. NocoDB INSERT audit_log (prompt_version)
# 5. Qdrant upsert 'interlinking': embedding(tactic + link_rules)
# 6. Event emitter → Шаги 08, 14
# 7. Grafana push: total_tactics, hub_count, orphan_integrated_pct, max_depth_target
```

---

## ██ КРИТЕРИИ QC

| **Проверка** | **Порог** | **Вес** | **Тип** | **При провале** |
| --- | --- | --- | --- | --- |
| JSON Schema валидна | Pass | 10 | hard | Retry |
| Engine теги 100% | Pass | 10 | hard | Retry |
| both → [Я] И [G] | Pass | 5 | hard | Retry |
| Goals ≥ 2 | Pass | 5 | hard | Retry |
| Tactics ≥ 3 | Pass | 10 | hard | Retry |
| Risks ≥ 2 | Pass | 5 | hard | Retry |
| Anchor ratios sum = 100 ±2 | Pass | 10 | hard | Retry |
| exact_match ≤ 30% | Pass | 5 | hard | Retry |
| Hub pages ≤ 10 и в whitelist | Pass | 5 | hard | Retry |
| Orphan coverage ≥ 80% | Pass | 10 | hard | Retry |
| Нет ссылок на deindex/delete | Pass | 5 | hard | Retry |
| max_links_per_page ≤ 200 | Pass | 3 | hard | Retry |
| target_max_depth ≤ 4 | Pass | 3 | hard | Retry |
| Roadmap ≥ 2 фаз | Pass | 5 | hard | Retry |
| Нет дублей id | Pass | 3 | hard | Retry |
| tactic_type diversity ≥ 4 | Pass | 3 | soft | Ревью |
| cross_step_hooks у тактик | Pass | 3 | soft | Ревью |

QC = сумма весов. ≥80 + все hard → Шаг 08; 60-79 → ревью; <60 → retry.

---

## ██ СТРУКТУРА N8N-ВОРКФЛОУ

```
Нода 1:  Триггер (после Шагов 04.2 / 06 или ручной запуск)
Нода 2:  PROJECT.md + output Шагов 04, 04.2, 01 из NocoDB
Нода 3:  [Параллельно]:
         ├─ 3a: step_07_pre_links_graph.py → current_links_graph.json
         └─ 3b: step_07_pre_orphan_prep.py → orphan_link_to_items[]
Нода 4:  Merge + подготовка top_inbound_pages, zero_inbound_pages
Нода 5:  Qdrant RAG (5 коллекций)
Нода 6:  Сборка промпта (system + user + RAG + graph stats)
Нода 7:  Claude Opus API (temperature 0.2, top_p 0.9, max_tokens 8192)
Нода 8:  Парсинг JSON
Нода 9:  step_07_post_validate.py
         ├─ hard-fail → Нода 6 (retry, макс. 3)
         └─ hard-fail 2x → декомпозиция (hub+orphan | anchors+depth+roadmap)
Нода 10: step_07_post_store.py (NocoDB + Qdrant, idempotent)
Нода 11: Grafana metrics push
Нода 12: Event emitter → Шаги 08, 14
Нода 13: Передача / стоп
```

**Обработка ошибок:**

```
• Краулер графа таймаут: partial graph, coverage_pct в output
• Opus 429/500: бэкофф 10с→30с→90с, макс. 3
• QC hard-fail 2x: декомпозиция (hub+orphan_plan | anchor+depth+roadmap) → merge
• QC hard-fail 3x: алерт Telegram
• Qdrant недоступен: без RAG
• NocoDB недоступен для Шага 04.2: без orphan-данных, план без orphan_integration
```

---

## ██ ПРИМЕР FEW-SHOT

```json
{
  "project_id": "dental_moscow_001",
  "step_id": "07",
  "prompt_version": "step07_v1.0",
  "timestamp": "2025-12-25T09:00:00Z",
  "strategy_name": "Стратегия внутренней перелинковки",
  "goals": [
    { "goal": "Снизить количество orphan-страниц до 0", "kpi": "orphan_count", "target_value": "0", "deadline": "30 дней" },
    { "goal": "Снизить макс. глубину до 3 кликов", "kpi": "max_depth", "target_value": "3", "deadline": "45 дней" },
    { "goal": "Увеличить средний inbound на услугах до 5+", "kpi": "avg_inbound_services", "target_value": "5", "deadline": "60 дней" }
  ],
  "tactics": [
    {
      "tactic_id": "IL001",
      "description": "Hub-spoke модель: /uslugi/ как главный хаб → все страницы услуг",
      "priority": "critical",
      "engine_tag": "[ОБА]",
      "resources_needed": "Шаблон категории CMS, контент-менеджер",
      "expected_result": "Все услуги доступны в 2 клика от главной, рост индексации",
      "timeline": "1 неделя",
      "tactic_type": "hub_spoke",
      "affected_page_types": ["category", "service"],
      "link_rules": [
        {
          "from_url_pattern": "/uslugi/",
          "to_url_pattern": "/uslugi/*",
          "anchor_template": "{{service_name}} в Москве",
          "location": "content",
          "max_links_per_page": 30,
          "nofollow": false
        }
      ],
      "cross_step_hooks": { "to_step_08_tz": true, "to_step_14_sitemap": true }
    },
    {
      "tactic_id": "IL002",
      "description": "Контекстная перелинковка статей блога → услуги (тематическое усиление)",
      "priority": "high",
      "engine_tag": "[G]",
      "resources_needed": "Контент-менеджер, шаблон 'Рекомендуемые услуги'",
      "expected_result": "Topical Authority: статьи передают тематический вес услугам",
      "timeline": "2 недели",
      "tactic_type": "contextual",
      "affected_page_types": ["article", "service"],
      "link_rules": [
        {
          "from_url_pattern": "/blog/*",
          "to_url_pattern": "/uslugi/*",
          "anchor_template": "{{keyword_partial_match}}",
          "location": "content",
          "max_links_per_page": 5,
          "nofollow": false
        }
      ],
      "cross_step_hooks": { "to_step_08_tz": true, "to_step_14_sitemap": false }
    },
    {
      "tactic_id": "IL003",
      "description": "Интеграция 4 orphan-страниц услуг в навигацию и контент",
      "priority": "high",
      "engine_tag": "[Я]",
      "resources_needed": "CMS-администратор",
      "expected_result": "0 orphan-услуг, рост ИКС через полноту структуры",
      "timeline": "3 дня",
      "tactic_type": "orphan_fix",
      "affected_page_types": ["service"],
      "link_rules": [
        {
          "from_url_pattern": "/uslugi/",
          "to_url_pattern": "/uslugi/otbelivanie-zoom",
          "anchor_template": "Отбеливание Zoom",
          "location": "content",
          "max_links_per_page": 30,
          "nofollow": false
        }
      ],
      "cross_step_hooks": { "to_step_08_tz": true, "to_step_14_sitemap": true }
    },
    {
      "tactic_id": "IL004",
      "description": "Хлебные крошки с Schema BreadcrumbList на всех внутренних страницах",
      "priority": "high",
      "engine_tag": "[ОБА]",
      "resources_needed": "Разработчик (шаблон CMS), 2-4 часа",
      "expected_result": "Улучшение навигации для ботов и пользователей, rich snippets в Google",
      "timeline": "1 неделя",
      "tactic_type": "breadcrumb",
      "affected_page_types": ["service", "article", "category"],
      "link_rules": [
        {
          "from_url_pattern": "/**",
          "to_url_pattern": "parent_page",
          "anchor_template": "{{parent_h1}}",
          "location": "breadcrumb",
          "max_links_per_page": 5,
          "nofollow": false
        }
      ],
      "cross_step_hooks": { "to_step_08_tz": true, "to_step_14_sitemap": false }
    },
    {
      "tactic_id": "IL005",
      "description": "Оптимизация anchor-текстов: диверсификация анкоров на ключевых услугах",
      "priority": "medium",
      "engine_tag": "[Я]",
      "resources_needed": "Контент-менеджер, таблица анкоров",
      "expected_result": "Снижение risk переспама (Яндекс), рост релевантности",
      "timeline": "2 недели",
      "tactic_type": "anchor_optimization",
      "affected_page_types": ["article", "service"],
      "link_rules": [],
      "cross_step_hooks": { "to_step_08_tz": true, "to_step_14_sitemap": false }
    }
  ],
  "risks": [
    { "risk": "Переспам анкоров: одинаковый exact-match на все ссылки → фильтр Яндекса", "probability": "medium", "mitigation": "exact_match ≤ 25%, разнообразие анкоров через шаблоны с переменными" },
    { "risk": "Слишком много ссылок на одной странице → размытие PageRank", "probability": "low", "mitigation": "max_links_per_page ≤ 100 для контента, приоритет — тематические ссылки" },
    { "risk": "Circular linking без хаба → бот зацикливается", "probability": "low", "mitigation": "Hub-spoke модель, все циклы проходят через /uslugi/ хаб" }
  ],
  "hub_pages": [
    { "url": "https://dental-moscow.ru/", "role": "main_hub", "spoke_urls": ["/uslugi/", "/blog/", "/vrachi/", "/kontakty/"], "target_inbound": 50, "current_inbound": 35 },
    { "url": "https://dental-moscow.ru/uslugi/", "role": "category_hub", "spoke_urls": ["/uslugi/implantaciya", "/uslugi/protezirovanie", "/uslugi/otbelivanie-zoom"], "target_inbound": 20, "current_inbound": 8 },
    { "url": "https://dental-moscow.ru/blog/", "role": "topic_hub", "spoke_urls": ["/blog/kak-vybrat-zubnuyu-pastu", "/blog/implantaciya-za-i-protiv"], "target_inbound": 10, "current_inbound": 3 }
  ],
  "orphan_integration_plan": [
    { "orphan_url": "https://dental-moscow.ru/uslugi/otbelivanie-zoom", "parent_url": "https://dental-moscow.ru/uslugi/", "anchor_text": "Отбеливание Zoom", "integration_method": "content_link" },
    { "orphan_url": "https://dental-moscow.ru/blog/kak-vybrat-zubnuyu-pastu", "parent_url": "https://dental-moscow.ru/blog/", "anchor_text": "Как выбрать зубную пасту", "integration_method": "related_block" }
  ],
  "anchor_strategy": {
    "exact_match_ratio": 25,
    "partial_match_ratio": 35,
    "branded_ratio": 15,
    "generic_ratio": 25,
    "anchor_templates": [
      { "page_type_from": "article", "page_type_to": "service", "template": "{{keyword}} — узнать подробнее", "engine_tag": "[ОБА]" },
      { "page_type_from": "category", "page_type_to": "service", "template": "{{service_name}} в {{geo}}", "engine_tag": "[Я]" },
      { "page_type_from": "service", "page_type_to": "article", "template": "Подробнее: {{article_title}}", "engine_tag": "[G]" }
    ]
  },
  "depth_optimization": {
    "current_max_depth": 5,
    "target_max_depth": 3,
    "pages_to_elevate": [
      { "url": "https://dental-moscow.ru/uslugi/otbelivanie-zoom", "current_depth": 5, "target_depth": 2, "method": "Добавить ссылку из /uslugi/ (hub)" },
      { "url": "https://dental-moscow.ru/blog/kak-vybrat-zubnuyu-pastu", "current_depth": 4, "target_depth": 2, "method": "Добавить в блок 'Популярные статьи' на /blog/" }
    ]
  },
  "implementation_roadmap": [
    { "phase": 1, "phase_name": "Быстрые победы", "tactics_ids": ["IL003", "IL004"], "timeline": "1 неделя", "dependencies": [] },
    { "phase": 2, "phase_name": "Hub-spoke + контекстная", "tactics_ids": ["IL001", "IL002"], "timeline": "2-3 недели", "dependencies": ["phase_1"] },
    { "phase": 3, "phase_name": "Оптимизация анкоров", "tactics_ids": ["IL005"], "timeline": "4-6 недель", "dependencies": ["phase_2"] }
  ],
  "qc_score": 92
}
```

---

## ██ СТРАТЕГИЯ ОТКАТА

```
Откат A: Граф перелинковки не построен (краулер упал).
  → Opus строит стратегию на основе step_04_output.pages[] (URL-иерархия) без метрик.
  → hub_pages определяются по page_type, не по inbound_count.
  → depth_optimization: target_max_depth=3, без current_depth.

Откат B: Данные Шага 04.2 недоступны (нет orphan).
  → orphan_integration_plan = [].
  → Стратегия без orphan-фикса, фокус на hub-spoke + anchors.

Откат C: Opus hard-QC fail 2x.
  → Декомпозиция на 2 вызова:
    В1: hub_pages + orphan_integration_plan + depth_optimization
    В2: tactics + anchor_strategy + goals + risks + roadmap
  → Merge.

Откат D: Qdrant недоступен.
  → Без RAG. Opus оценивает по базовым стандартам перелинковки.

Откат E: Семантические кластеры (Шаг 01) недоступны.
  → Кластеризация по URL-иерархии (path segments).
  → Silo-структура определяется по page_type + URL-паттернам.
```

---

## ██ АДАПТАЦИИ ПОД КЛАСТЕР

```
• commercial_local:
  — Hub: главная → услуги → филиалы
  — Локальные ссылки: /filial/{city}/ → услуги филиала
  — Breadcrumbs с городом: Главная > Москва > Стоматология > Имплантация
  — Яндекс.Бизнес: ссылка из footer каждой страницы
  — max_depth: 3

• commercial_national:
  — Silo по категориям услуг: /services/{category}/{service}
  — Hub-spoke: главная → категории → услуги
  — Перелинковка блог → услуги (topical authority)
  — Портфолио/кейсы → услуги (trust-усиление)
  — max_depth: 3

• ecom:
  — Hub: каталог → категории → подкатегории → товары
  — Фильтры: НЕ перелинковывать (canonical → категория)
  — Товар → «Похожие товары» (related_block, 4-8 ссылок)
  — Товар → «С этим покупают» (cross-sell)
  — Категория → топ-товары (max 20)
  — Breadcrumbs: обязательны, multi-path для товаров в нескольких категориях
  — max_depth: 4 (каталоги глубокие)
  — anchor: название товара + бренд (не keyword stuffing)

• saas:
  — Hub: /features/ → /features/{feature}/
  — Документация: /docs/ → /docs/{section}/ (silo)
  — Blog → features (контекстная)
  — Pricing → features (сравнение)
  — Integrations → features (связь)
  — max_depth: 3

• info:
  — Topic clusters: pillar page → cluster articles
  — Silo строгий: /topic/ → /topic/subtopic/
  — Related articles (блок внизу, 3-5 ссылок)
  — Авторская перелинковка: /author/{name}/ → статьи автора
  — Tag pages: не перегружать (max 50 ссылок)
  — max_depth: 4

• medical:
  — Hub: /uslugi/ → /uslugi/{specialization}/
  — Врач → услуги: /vrachi/{name}/ → /uslugi/{service}/ (expertise linking)
  — Статья → услуга + врач (triple link)
  — Лицензии: ссылка из footer (trust)
  — max_depth: 3

• legal:
  — Hub: /uslugi/ → /uslugi/{practice_area}/
  — Кейсы → практики: /cases/{case}/ → /uslugi/{area}/
  — Юрист → практики: /team/{name}/ → /uslugi/{area}/
  — max_depth: 3

• education:
  — Hub: /courses/ → /courses/{course}/
  — Преподаватель → курсы
  — Отзывы → курсы
  — Расписание → курсы
  — max_depth: 3
```

---

## ██ МЕТРИКИ УСПЕХА

```
Опережающий:
  — tactics ≥ 3, goals ≥ 2, risks ≥ 2
  — anchor_ratios = 100%
  — hub_pages определены (≤ 10)
  — orphan coverage ≥ 80%
  — target_max_depth ≤ 3 (commercial) или ≤ 4 (info/ecom)
  — roadmap ≥ 2 фаз
  — tactic_type diversity ≥ 4

Запаздывающий:
  — После Шага 08: link_rules включены в ТЗ (≥ 80%)
  — Через 30 дней: orphan_count = 0
  — Через 30 дней: max_depth снижен до target
  — Через 60 дней: avg_inbound на услугах +50%
  — Через 60 дней: crawl budget Googlebot сфокусирован на hub-страницах
  — Через 90 дней: рост позиций hub-страниц +10 пунктов

Контроль:
  — Ревью: link_rules реализуемы в CMS (≥ 80%)
  — anchor_templates применимы (не слишком шаблонные)
  — Стабильность: повторный запуск даёт ±10% по тактикам
```

---

## ██ ИСТОРИЯ ИЗМЕНЕНИЙ

| **Версия** | **Дата** | **Изменения** |
| --- | --- | --- |
| v1.0 | 2025-12 | Базовая версия: Template 5, hub-spoke + silo + breadcrumb + contextual + orphan_fix + anchor_optimization, link_rules для CMS, orphan_integration_plan, anchor_strategy с ratios, depth_optimization, implementation_roadmap, кластерные адаптации |
