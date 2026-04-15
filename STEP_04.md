# STEP_04.md — Структура Страниц

> **SEO AUTOMATION ENGINEERING · МОДУЛЬ 3 · STEP_04**
> Самодостаточный модуль. Загружается: CORE.md + PROJECT.md + этот файл. ~4K токенов.

---

## ██ КАРТОЧКА ШАГА

| **Параметр**          | **Значение**                                                                 |
|-----------------------|------------------------------------------------------------------------------|
| ID                    | 04                                                                           |
| Название              | Структура страниц                                                            |
| Блок                  | 2: Структура и контент                                                       |
| Основная роль         | A: SEO-стратег + E: CRO и поведенческий аналитик                            |
| Консультативные       | F: Промпт-инженер, D: Аналитик конкурентной разведки                         |
| Тип                   | Промпт                                                                       |
| HitL                  | Нет (результат → Шаг 04.1 автоматически)                                    |
| Приоритет ROI         | ВЫСОКИЙ — скелет каждой страницы; ошибка = переделка контента и ТЗ           |
| Зависит от            | Шаг 00 (Анализ ЦА), Шаг 01 (Сбор СЯ), Шаг 02 (Структура сайта)            |
| Передаёт в            | Шаг 04.1 (Каннибализация), Шаг 05 (Коммерч. факторы), Шаг 07 (Перелинковка), Шаг 08 (ТЗ на страницы) |
| JSON-шаблон           | Template 5: Strategy                                                         |
| Режим Opus            | JSON-вывод: temperature 0.15, top_p 0.9, max_tokens 8192                    |

---

## ██ ЦЕЛЬ

Для каждого типа страницы, определённого в архитектуре сайта (Шаг 02), сформировать детальную структуру контентных блоков, CRO-элементов и SEO-зон с учётом интентов целевой аудитории (Шаг 00) и семантического ядра (Шаг 01), чтобы обеспечить единый стандарт качества страниц и фундамент для ТЗ копирайтерам/разработчикам (Шаг 08).

---

## ██ КОНТРАКТ ВХОДНЫХ ДАННЫХ (JSON)

Вход агрегируется из output-контрактов Шагов 00, 01 и 02. Минимальные обязательные поля:

```json
{
  "project_id": "string",
  "niche": "string",
  "niche_cluster": "string (commercial_local│commercial_national│ecom│saas│info│medical│legal│education)",
  "geo": "string",
  "language": "string",
  "priority_engines": "yandex│google│both",
  "domain": "string",
  "cms": "string",

  "personas": [
    {
      "persona_id": "string",
      "persona_name": "string",
      "jtbd": {
        "core_job": "string",
        "functional_jobs": ["string"],
        "triggers": ["string"],
        "barriers": ["string"]
      },
      "hunt_ladder": {
        "level_1_unaware": { "sample_queries": [{"query":"string","engine":"string","intent":"string"}] },
        "level_2_problem_aware": { "sample_queries": [] },
        "level_3_solution_aware": { "sample_queries": [] },
        "level_4_product_aware": { "sample_queries": [] },
        "level_5_most_aware": { "sample_queries": [] }
      },
      "search_behavior": { "primary_engine":"string", "device":"string" },
      "content_preferences": { "preferred_formats":["string"], "trust_signals":["string"] },
      "priority_score": "number 1-100"
    }
  ],

  "site_architecture": {
    "strategy_name": "string",
    "pages": [
      {
        "url_pattern": "string",
        "page_type": "main│category│subcategory│product│service│article│blog│faq│landing│contact│about│geo_landing",
        "assigned_clusters": [
          { "cluster_id": "string", "queries": [{"query":"string","engine":"string","intent":"string","volume":"number"}] }
        ],
        "depth": "number",
        "parent_url": "string│null"
      }
    ]
  },

  "semantic_core": {
    "total_queries": "number",
    "clusters": [
      {
        "cluster_id": "string",
        "cluster_name": "string",
        "primary_intent": "informational│navigational│commercial│transactional",
        "queries": [{"query":"string","volume":"number","engine":"string"}]
      }
    ]
  }
}
```

---

## ██ КОНТРАКТ ВЫХОДНЫХ ДАННЫХ (JSON)

Наследует **Template 5: Strategy**. Роль G валидирует. Должен пройти структурный QC.

```json
{
  "project_id": "string",
  "step_id": "04",
  "prompt_version": "string",
  "timestamp": "ISO 8601",

  "strategy_name": "Структура страниц",

  "goals": [
    {
      "goal": "string",
      "kpi": "string",
      "target_value": "string",
      "deadline": "string"
    }
  ],

  "page_type_structures": [
    {
      "page_type": "main│category│subcategory│product│service│article│blog│faq│landing│contact│about│geo_landing",
      "page_type_label": "string (человекочитаемое название на русском)",
      "target_intents": ["informational│navigational│commercial│transactional"],
      "target_hunt_levels": ["number 1-5"],
      "mapped_persona_ids": ["string"],

      "content_blocks": [
        {
          "block_id": "string (B001...)",
          "block_type": "h1│breadcrumbs│intro│usp│main_content│h2_section│product_card│price_table│cta│faq│reviews│trust_signals│related│geo_block│author_block│schema_zone│internal_links│footer_seo",
          "position": "number (порядок на странице)",
          "required": "boolean",
          "description": "string",
          "seo_function": "string (какую SEO-задачу решает блок)",
          "cro_function": "string│null (какую CRO-задачу решает блок)",
          "engine_tag": "[Я]│[G]│[ОБА]",
          "min_word_count": "number│null",
          "max_word_count": "number│null",
          "keywords_placement": "string│null (рекомендации по размещению ключей)",
          "schema_type": "string│null (если блок предполагает Schema-разметку)",
          "notes": "string│null"
        }
      ],

      "required_schema_types": ["string (FAQPage│Product│LocalBusiness│Article│HowTo│BreadcrumbList...)"],

      "cro_elements": [
        {
          "element": "string",
          "placement": "string",
          "trigger_persona": "string│null",
          "engine_tag": "[Я]│[G]│[ОБА]"
        }
      ],

      "internal_linking_zones": [
        {
          "zone": "string (breadcrumbs│sidebar│related│footer│contextual)",
          "target_page_types": ["string"],
          "anchor_strategy": "string",
          "engine_tag": "[Я]│[G]│[ОБА]"
        }
      ],

      "mobile_adaptations": ["string"],

      "engine_specific_notes": {
        "yandex": ["string"],
        "google": ["string"]
      }
    }
  ],

  "tactics": [
    {
      "tactic_id": "string",
      "description": "string",
      "priority": "critical│high│medium│low",
      "engine_tag": "[Я]│[G]│[ОБА]",
      "resources_needed": "string",
      "expected_result": "string",
      "timeline": "string"
    }
  ],

  "risks": [
    {
      "risk": "string",
      "probability": "high│medium│low",
      "mitigation": "string"
    }
  ],

  "qc_score": "number 0-100"
}
```

---

## ██ ПРОМПТ ДЛЯ CLAUDE OPUS

### Системный промпт

```
Ты — дуальная роль:
• Роль A: Senior SEO-стратег (10+ лет, Яндекс + Google). Принимаешь финальное решение по структуре.
• Роль E: CRO и поведенческий аналитик. Консультируешь по конверсионным элементам.

Задача: Для каждого типа страницы из архитектуры сайта создать детальную структуру контентных блоков, CRO-элементов и SEO-зон.

ПРАВИЛА:
1. Проанализируй ВСЕ типы страниц из site_architecture. Для каждого создай отдельный page_type_structure.
2. Каждый content_block ОБЯЗАТЕЛЬНО содержит:
   - block_type, position (порядок сверху вниз), required (true/false)
   - seo_function (какую SEO-задачу решает)
   - cro_function (какую CRO-задачу решает, null если нет)
   - engine_tag: [Я]│[G]│[ОБА]
3. Привязывай блоки к persona_id и hunt_level: кто и на каком уровне осведомлённости видит этот блок.
4. Учитывай niche_cluster:
   - commercial_local: блок гео-привязки, карта, NAP, отзывы, Яндекс.Бизнес-виджет
   - ecom: карточка товара, фильтры, цена, сравнение, отзывы, наличие
   - medical: блок врача (E-E-A-T), лицензии, YMYL-дисклеймер
   - saas: демо/триал CTA, интеграции, сравнительная таблица
   - info: автор (E-E-A-T), дата обновления, оглавление, depth
   - legal: статус адвоката, кейсы, срочный CTA
5. CRO-элементы (Роль E):
   - Каждый тип страницы — минимум 2 CTA (основной + вторичный)
   - Trust signals: для каждого типа перечислить (отзывы, сертификаты, гарантии и т.д.)
   - Барьеры из jtbd.barriers персон → блоки, снимающие эти барьеры
6. Schema.org: для каждого типа страницы указать required_schema_types.
7. Internal linking zones: breadcrumbs, sidebar, related, contextual — для каждого типа.
8. Mobile: для каждого типа указать адаптации (accordion FAQ, sticky CTA, скрытие тяжёлых блоков).
9. Engine-specific:
   [Я]: ПФ-элементы (время на странице, интерактив), КФ-блоки (цена, доставка, гарантия), Яндекс.Нейро-оптимизация
   [G]: E-E-A-T блоки (автор, экспертиза, источники), CWV-оптимизация (lazy load, LCP), AEO-блоки (answer box)
10. Тактики (tactics): конкретные действия по внедрению структур. Каждая с engine_tag.
11. Риски: не менее 3 рисков с митигацией.
12. goals: 2-4 цели с измеримыми KPI.

ВЫВОД: Валидный JSON по контракту Template 5. Без markdown. Без комментариев. Весь текст — на РУССКОМ языке.
```

### Пользовательский промпт (Шаблон)

```
Создай детальную структуру страниц:

Проект: {{project.name}}
Ниша: {{project.niche}}
Кластер: {{project.niche_cluster}}
Гео: {{project.geo}}
Язык: {{project.language}}
Приоритет ПС: {{project.priority_engines}}
Домен: {{project.domain}}
CMS: {{project.cms}}

=== ПЕРСОНЫ (из Шага 00) ===
{{#each personas}}
- {{persona_id}}: {{persona_name}} | core_job: {{jtbd.core_job}} | triggers: {{triggers | join(', ')}} | barriers: {{barriers | join(', ')}} | device: {{search_behavior.device}} | trust: {{content_preferences.trust_signals | join(', ')}}
{{/each}}

=== АРХИТЕКТУРА САЙТА (из Шага 02) ===
Типы страниц:
{{#each site_architecture.pages | groupBy('page_type')}}
Тип: {{@key}} | Кол-во URL: {{this.length}}
  Пример URL: {{this[0].url_pattern}}
  Интенты: {{this[0].assigned_clusters | map('primary_intent') | unique | join(', ')}}
{{/each}}

=== СЕМАНТИЧЕСКОЕ ЯДРО (из Шага 01) ===
Всего запросов: {{semantic_core.total_queries}}
Кластеры (топ-10 по объёму):
{{#each semantic_core.clusters | sortBy('total_volume') | take(10)}}
- {{cluster_id}}: {{cluster_name}} | intent: {{primary_intent}} | запросов: {{queries.length}}
{{/each}}

{{#if rag_context}}
=== КОНТЕКСТ ИЗ RAG ===
{{rag_context}}
{{/if}}
```

---

## ██ RAG-ОБОГАЩЕНИЕ

| Коллекция Qdrant       | Запрос                                          | Назначение                                                          | Лимит токенов |
|-------------------------|------------------------------------------------|----------------------------------------------------------------------|---------------|
| `competitors`           | niche + geo + "структура страниц"               | Блоки и CRO-элементы конкурентов                                    | 800           |
| `personas`              | project_id                                      | Полные портреты ЦА для маппинга блоков                              | 600           |
| `page_templates`        | niche_cluster + page_type                       | Эталонные шаблоны страниц из похожих проектов                       | 600           |

Общий лимит RAG-контекста: **2000 токенов**.

Порядок инъекции: competitors → personas → page_templates. Если суммарно > 2000, обрезать page_templates.

---

## ██ СКРИПТЫ CLAUDE CODE

> Тип шага: **Промпт** → секция CLAUDE CODE SCRIPTS содержит только pre-step сбор данных конкурентов и post-step валидатор.

### Пред-шаг: Парсинг структуры страниц конкурентов

```
# step_04_pre_competitor_pages.py
# Назначение: Извлечение структуры блоков со страниц конкурентов
# Вход: project.main_competitors, site_architecture.pages (типы страниц)
# Выход: competitor_page_structures.json
# Действия:
# 1. Для каждого конкурента: по 1 URL на каждый тип страницы
# 2. Извлечь: H1, H2-секции, наличие FAQ, отзывов, CTA, Schema, breadcrumbs
# 3. Определить паттерны блоков: какие блоки встречаются у 2+ конкурентов
# 4. Сохранить в Qdrant коллекцию 'competitors' с тегом step_04
```

### Пред-шаг: Анализ SERP-фич по интентам

```
# step_04_pre_serp_features.py
# Назначение: Определить какие SERP-фичи показываются по ключевым кластерам
# Вход: semantic_core.clusters (топ-20 по объёму)
# Выход: serp_features_map.json
# Действия:
# 1. DataForSEO SERP API: по 1 запросу из каждого кластера
# 2. Извлечь: featured_snippet, faq, local_pack, reviews, knowledge_panel
# 3. Маппинг: SERP-фича → рекомендуемый блок на странице
#    (featured_snippet → answer_block, faq → faq_section, local_pack → geo_block)
# 4. Результат → RAG-контекст для промпта
```

### Пост-шаг: Валидатор

```
# step_04_post_validate.py
# Назначение: Валидация выхода по JSON-схеме и бизнес-правилам
# Проверки:
# 1. JSON Schema (Template 5 + расширение page_type_structures)
# 2. page_type_structures >= количество уникальных page_type в site_architecture
# 3. Каждый page_type_structure содержит >= 5 content_blocks
# 4. Каждый content_block имеет engine_tag
# 5. Каждый page_type имеет >= 1 cta в content_blocks (block_type == "cta")
# 6. Каждый page_type имеет >= 1 элемент в required_schema_types
# 7. Каждый page_type имеет >= 1 internal_linking_zone
# 8. mobile_adaptations не пусты для каждого page_type
# 9. tactics >= 5
# 10. risks >= 3
# 11. goals >= 2
# 12. Все engine_tag валидны: [Я]│[G]│[ОБА]
# 13. Нет дублирующихся page_type
# 14. При priority_engines == "both": есть хотя бы 1 [Я] и 1 [G] в engine_specific_notes каждого типа
```

### Пост-шаг: Qdrant

```
# step_04_post_qdrant.py
# Назначение: Сохранение структур страниц в Qdrant для RAG последующих шагов
# Действия:
# 1. Эмбеддинг каждого page_type_structure (page_type + content_blocks описания)
# 2. Upsert в коллекцию 'page_structures'
# 3. Метаданные: project_id, step_id, page_type, timestamp
# 4. Используется в Шагах 05, 07, 08
```

---

## ██ КРИТЕРИИ QC

| **Проверка**                                      | **Порог**                            | **Вес** | **При провале** |
|--------------------------------------------------|--------------------------------------|---------|-----------------|
| Покрытие типов страниц                            | 100% типов из site_architecture      | 20      | Retry           |
| Количество блоков на тип                          | ≥5 на каждый тип                     | 15      | Retry           |
| Теги engine                                       | 100% блоков размечены                | 15      | Retry           |
| CTA-блоки                                         | ≥1 на каждый тип                     | 10      | Retry           |
| Schema-типы                                       | ≥1 на каждый тип                     | 10      | Retry           |
| Маппинг персон                                    | ≥1 persona_id на каждый тип          | 10      | Ревью           |
| CRO-элементы (Роль E)                             | ≥2 на каждый тип                     | 5       | Ревью           |
| Internal linking zones                            | ≥1 на каждый тип                     | 5       | Retry           |
| Mobile-адаптации                                  | Не пусты                             | 5       | Ревью           |
| Соответствие niche_cluster                        | Кластерные блоки присутствуют        | 5       | Ревью           |

**QC = сумма весов. ≥80 → Шаг 04.1, 60-79 → ревью, <60 → retry.**

---

## ██ СТРУКТУРА N8N-ВОРКФЛОУ

```
Нода 1:  Триггер (после завершения Шага 02 или ручной запуск)
Нода 2:  Загрузка PROJECT.md из NocoDB
Нода 3:  Загрузка output Шага 00 (персоны) из NocoDB
Нода 4:  Загрузка output Шага 01 (семантическое ядро) из NocoDB
Нода 5:  Загрузка output Шага 02 (архитектура сайта) из NocoDB
Нода 6:  step_04_pre_competitor_pages.py
Нода 7:  step_04_pre_serp_features.py
Нода 8:  Qdrant RAG (competitors + personas + page_templates)
Нода 9:  Сборка промпта (system + user + RAG-контекст)
Нода 10: Claude Opus API (temp 0.15, top_p 0.9, max_tokens 8192)
Нода 11: Парсинг JSON (удаление markdown-обёртки если есть)
Нода 12: step_04_post_validate.py → провал → Нода 9 (макс. 3 retry)
Нода 13: step_04_post_qdrant.py
Нода 14: Сохранение в NocoDB (step_results, prompt_version)
Нода 15: Метрики → Grafana
Нода 16: Запуск Шага 04.1 или стоп при QC fail 3x
```

### Обработка ошибок

```
• Opus 429/500: бэкофф 10с→30с→90с, макс. 3
• Qdrant недоступен: промпт без RAG-контекста, лог предупреждения
• Нет данных Шага 00/01/02: СТОП, алерт — зависимости не выполнены
• QC провал 3x: алерт Telegram с логом 3 попыток
• Невалидный JSON: удалить markdown-обёртку, повторный парсинг → retry
• Слишком большой вход (>6K токенов): обрезать semantic_core до топ-10 кластеров
```

---

## ██ ПРИМЕР FEW-SHOT

```json
{
  "project_id": "dental_moscow_001",
  "step_id": "04",
  "prompt_version": "step04_v1.0",
  "timestamp": "2025-12-15T10:30:00Z",

  "strategy_name": "Структура страниц",

  "goals": [
    {
      "goal": "Обеспечить единый стандарт контентных блоков для всех типов страниц",
      "kpi": "Доля страниц, соответствующих структуре",
      "target_value": "100%",
      "deadline": "До запуска контент-генерации (Шаг 08)"
    },
    {
      "goal": "Включить CRO-элементы, снижающие барьеры ЦА",
      "kpi": "Конверсия целевых страниц",
      "target_value": "+15% к baseline через 3 мес.",
      "deadline": "3 месяца после внедрения"
    }
  ],

  "page_type_structures": [
    {
      "page_type": "service",
      "page_type_label": "Страница услуги",
      "target_intents": ["commercial", "transactional"],
      "target_hunt_levels": [3, 4, 5],
      "mapped_persona_ids": ["P001", "P002"],

      "content_blocks": [
        {
          "block_id": "B001",
          "block_type": "breadcrumbs",
          "position": 1,
          "required": true,
          "description": "Хлебные крошки: Главная → Услуги → [Название услуги]",
          "seo_function": "Навигация, передача ссылочного веса, BreadcrumbList Schema",
          "cro_function": null,
          "engine_tag": "[ОБА]",
          "min_word_count": null,
          "max_word_count": null,
          "keywords_placement": null,
          "schema_type": "BreadcrumbList",
          "notes": null
        },
        {
          "block_id": "B002",
          "block_type": "h1",
          "position": 2,
          "required": true,
          "description": "Заголовок H1 с основным ключом и гео: '[Услуга] в [Город]'",
          "seo_function": "Основной ранжирующий сигнал, релевантность запросу",
          "cro_function": "Первый контакт — подтверждение что пользователь на нужной странице",
          "engine_tag": "[ОБА]",
          "min_word_count": 3,
          "max_word_count": 12,
          "keywords_placement": "Основной ключ в начале, гео в конце",
          "schema_type": null,
          "notes": "[Я] ПФ: точное вхождение запроса повышает CTR из выдачи"
        },
        {
          "block_id": "B003",
          "block_type": "intro",
          "position": 3,
          "required": true,
          "description": "Вводный абзац: УТП + решение core_job персоны + основной CTA",
          "seo_function": "Попадание в сниппет, [G] featured snippet, [Я] быстрый ответ",
          "cro_function": "Снятие барьера 'Страх ребёнка' — акцент на безболезненность",
          "engine_tag": "[ОБА]",
          "min_word_count": 40,
          "max_word_count": 80,
          "keywords_placement": "Основной + 1 LSI в первом предложении",
          "schema_type": null,
          "notes": null
        },
        {
          "block_id": "B004",
          "block_type": "usp",
          "position": 4,
          "required": true,
          "description": "Блок преимуществ: 4-6 иконок с кратким описанием (безболезненно, рядом, гарантия, опыт)",
          "seo_function": "[Я] КФ: коммерческие факторы повышают ранжирование",
          "cro_function": "Снятие барьеров: 'Страх ребёнка' → 'Безболезненное лечение', 'Цена' → 'Прозрачные цены'",
          "engine_tag": "[Я]",
          "min_word_count": 30,
          "max_word_count": 120,
          "keywords_placement": "LSI-ключи в описаниях преимуществ",
          "schema_type": null,
          "notes": "Дизайн: иконки + короткий текст, 2-3 в ряд на мобильных"
        },
        {
          "block_id": "B005",
          "block_type": "main_content",
          "position": 5,
          "required": true,
          "description": "Основной контент: описание услуги, этапы, показания, противопоказания",
          "seo_function": "Текстовая релевантность, плотность ключей, LSI, длина контента",
          "cro_function": "Образование клиента — подготовка к принятию решения (Hunt 3→4)",
          "engine_tag": "[ОБА]",
          "min_word_count": 300,
          "max_word_count": 800,
          "keywords_placement": "Основной ключ в H2, LSI в тексте, естественное распределение",
          "schema_type": "MedicalProcedure",
          "notes": "[G] E-E-A-T: ссылки на исследования, экспертный тон"
        },
        {
          "block_id": "B006",
          "block_type": "price_table",
          "position": 6,
          "required": true,
          "description": "Таблица цен: услуга, цена от, что включено",
          "seo_function": "[Я] КФ: наличие цен — критический коммерческий фактор",
          "cro_function": "Снятие барьера 'Цена' — прозрачность, диапазон 'от'",
          "engine_tag": "[Я]",
          "min_word_count": null,
          "max_word_count": null,
          "keywords_placement": "Название услуги в строках таблицы",
          "schema_type": "Offer",
          "notes": "Дизайн: адаптивная таблица, на мобильных — карточки"
        },
        {
          "block_id": "B007",
          "block_type": "author_block",
          "position": 7,
          "required": true,
          "description": "Блок врача: фото, ФИО, специализация, стаж, сертификаты",
          "seo_function": "[G] E-E-A-T: экспертиза, авторство, YMYL-доверие",
          "cro_function": "Trust signal — персонификация, снятие барьера доверия",
          "engine_tag": "[G]",
          "min_word_count": 30,
          "max_word_count": 100,
          "keywords_placement": "Специализация врача содержит ключ услуги",
          "schema_type": "Physician",
          "notes": "medical кластер: обязательно лицензия и сертификаты"
        },
        {
          "block_id": "B008",
          "block_type": "cta",
          "position": 8,
          "required": true,
          "description": "Основной CTA: 'Записаться на приём' с формой (имя + телефон)",
          "seo_function": "[Я] ПФ: интерактив повышает время на странице",
          "cro_function": "Основная конверсия — запись на приём",
          "engine_tag": "[ОБА]",
          "min_word_count": null,
          "max_word_count": null,
          "keywords_placement": null,
          "schema_type": null,
          "notes": "Mobile: sticky-кнопка внизу экрана"
        },
        {
          "block_id": "B009",
          "block_type": "reviews",
          "position": 9,
          "required": true,
          "description": "Блок отзывов: 3-5 отзывов с именем, датой, рейтингом",
          "seo_function": "[Я] КФ + [G] E-E-A-T: социальное доказательство, UGC",
          "cro_function": "Trust signal — снятие барьера доверия, подтверждение quality",
          "engine_tag": "[ОБА]",
          "min_word_count": null,
          "max_word_count": null,
          "keywords_placement": null,
          "schema_type": "Review",
          "notes": "Агрегировать из Яндекс.Карт, Google Maps, 2GIS"
        },
        {
          "block_id": "B010",
          "block_type": "faq",
          "position": 10,
          "required": true,
          "description": "FAQ: 5-8 вопросов по услуге, барьерам, ценам, подготовке",
          "seo_function": "FAQ Schema → rich snippet, [Я] Нейро → быстрый ответ, длинный хвост",
          "cro_function": "Снятие оставшихся барьеров, ответы на возражения",
          "engine_tag": "[ОБА]",
          "min_word_count": 200,
          "max_word_count": 500,
          "keywords_placement": "Вопросы = long-tail запросы из семантического ядра",
          "schema_type": "FAQPage",
          "notes": "[G] AEO: формулировать ответы для AI-сниппетов (40-60 слов)"
        },
        {
          "block_id": "B011",
          "block_type": "geo_block",
          "position": 11,
          "required": true,
          "description": "Гео-блок: адрес, карта (Яндекс.Карты + Google Maps), как добраться, метро",
          "seo_function": "[Я] геозависимость + [G] Local SEO: NAP consistency",
          "cro_function": "Снятие барьера 'далеко' — визуализация близости",
          "engine_tag": "[ОБА]",
          "min_word_count": 20,
          "max_word_count": 80,
          "keywords_placement": "Гео-ключи: район, метро, улица",
          "schema_type": "LocalBusiness",
          "notes": "commercial_local: КРИТИЧЕСКИЙ блок. [Я] 2GIS-виджет"
        },
        {
          "block_id": "B012",
          "block_type": "internal_links",
          "position": 12,
          "required": true,
          "description": "Связанные услуги: 3-5 ссылок на смежные услуги из той же категории",
          "seo_function": "Перелинковка: передача веса, снижение bounce rate",
          "cro_function": "Кросс-продажа, увеличение глубины сессии",
          "engine_tag": "[ОБА]",
          "min_word_count": null,
          "max_word_count": null,
          "keywords_placement": "Анкоры = ключевые слова целевых страниц",
          "schema_type": null,
          "notes": null
        }
      ],

      "required_schema_types": ["BreadcrumbList", "MedicalProcedure", "FAQPage", "LocalBusiness", "Physician", "Offer", "Review"],

      "cro_elements": [
        {
          "element": "Sticky CTA-кнопка 'Записаться' на мобильных",
          "placement": "Фиксированная внизу экрана",
          "trigger_persona": "P001",
          "engine_tag": "[ОБА]"
        },
        {
          "element": "Всплывающее окно 'Бесплатная консультация' при 60% скролла",
          "placement": "Попап после основного контента",
          "trigger_persona": "P002",
          "engine_tag": "[ОБА]"
        },
        {
          "element": "Виджет 'Рассчитать стоимость' (калькулятор)",
          "placement": "После блока цен",
          "trigger_persona": "P001",
          "engine_tag": "[Я]"
        }
      ],

      "internal_linking_zones": [
        {
          "zone": "breadcrumbs",
          "target_page_types": ["main", "category"],
          "anchor_strategy": "Название раздела",
          "engine_tag": "[ОБА]"
        },
        {
          "zone": "contextual",
          "target_page_types": ["article", "faq"],
          "anchor_strategy": "Информационный ключ из смежного кластера",
          "engine_tag": "[ОБА]"
        },
        {
          "zone": "related",
          "target_page_types": ["service"],
          "anchor_strategy": "Коммерческий ключ целевой страницы",
          "engine_tag": "[ОБА]"
        }
      ],

      "mobile_adaptations": [
        "FAQ в accordion (свернуть/развернуть)",
        "Таблица цен → горизонтальный скролл или карточки",
        "Sticky CTA-кнопка внизу экрана",
        "Карта — lazy load, не загружать до скролла",
        "Блок врача — горизонтальный свайп при нескольких врачах"
      ],

      "engine_specific_notes": {
        "yandex": [
          "КФ: обязательны блоки цен, отзывов, контактов",
          "ПФ: интерактивные элементы (калькулятор, фильтры) для увеличения времени на странице",
          "Яндекс.Нейро: структурированные ответы в FAQ для попадания в быстрые ответы",
          "Турбо-страницы: при наличии — дублировать ключевые блоки"
        ],
        "google": [
          "E-E-A-T: блок врача с сертификатами обязателен (YMYL)",
          "CWV: LCP < 2.5с — lazy load карт и изображений",
          "AEO: ответы в FAQ 40-60 слов для featured snippet",
          "Structured data: все 7 Schema-типов для rich results"
        ]
      }
    }
  ],

  "tactics": [
    {
      "tactic_id": "T001",
      "description": "Создать wireframe-шаблоны в Figma для каждого типа страницы с расположением блоков",
      "priority": "critical",
      "engine_tag": "[ОБА]",
      "resources_needed": "Дизайнер, 2-3 дня",
      "expected_result": "Визуальный стандарт для всех типов страниц",
      "timeline": "Неделя 1"
    },
    {
      "tactic_id": "T002",
      "description": "Внедрить Schema.org разметку для всех required_schema_types через Шаг 13",
      "priority": "critical",
      "engine_tag": "[G]",
      "resources_needed": "Разработчик, JSON-LD шаблоны",
      "expected_result": "Rich snippets в Google, повышение CTR на 15-30%",
      "timeline": "Неделя 2"
    },
    {
      "tactic_id": "T003",
      "description": "Настроить КФ-блоки (цены, доставка, гарантии) на всех коммерческих страницах",
      "priority": "critical",
      "engine_tag": "[Я]",
      "resources_needed": "Контент-менеджер, актуальные данные по ценам",
      "expected_result": "Повышение позиций в Яндексе по коммерческим запросам",
      "timeline": "Неделя 1-2"
    },
    {
      "tactic_id": "T004",
      "description": "Реализовать mobile-first адаптации: sticky CTA, accordion FAQ, lazy load",
      "priority": "high",
      "engine_tag": "[ОБА]",
      "resources_needed": "Frontend-разработчик, 3-4 дня",
      "expected_result": "CWV в зелёной зоне, снижение bounce rate на мобильных",
      "timeline": "Неделя 2-3"
    },
    {
      "tactic_id": "T005",
      "description": "Добавить E-E-A-T блоки (автор, лицензии, источники) на YMYL-страницы",
      "priority": "high",
      "engine_tag": "[G]",
      "resources_needed": "Данные о врачах/экспертах, сканы лицензий",
      "expected_result": "Усиление E-E-A-T сигналов для Google",
      "timeline": "Неделя 2"
    }
  ],

  "risks": [
    {
      "risk": "Шаблонность структуры — все страницы одного типа идентичны, что снижает уникальность",
      "probability": "medium",
      "mitigation": "Вариативность в content_blocks через адаптацию под конкретный кластер запросов"
    },
    {
      "risk": "Перегруженность страницы блоками — негативное влияние на CWV (LCP, CLS)",
      "probability": "high",
      "mitigation": "Lazy load для тяжёлых блоков (карта, отзывы), critical CSS для above-the-fold"
    },
    {
      "risk": "Несоответствие блоков реальным возможностям CMS (Bitrix/Tilda ограничения)",
      "probability": "medium",
      "mitigation": "Проверка с разработчиком на этапе wireframe, fallback-блоки для ограниченных CMS"
    }
  ],

  "qc_score": 88
}
```

---

## ██ СТРАТЕГИЯ ОТКАТА

```
Откат A: Декомпозиция на 2 вызова Opus:
  (1) Генерация page_type_structures для коммерческих типов (service, product, category)
  (2) Генерация для информационных типов (article, blog, faq) + tactics/risks
  Объединение результатов в post-step скрипте.

Откат B: Сократить до 3 основных типов страниц (main, service/product, article).
  Остальные типы — пометка в NocoDB для ручного расширения.

Откат C: Few-shot как жёсткий шаблон.
  Opus адаптирует блоки под конкретную нишу/гео, не генерируя структуру с нуля.
```

---

## ██ АДАПТАЦИИ ПОД КЛАСТЕР

| **Кластер**            | **Обязательные блоки**                                             | **Специфика**                                           |
|------------------------|--------------------------------------------------------------------|---------------------------------------------------------|
| commercial_local       | geo_block, NAP, карта, отзывы, цены, часы работы                    | Яндекс.Бизнес виджет, 2GIS, микро-гео (район, метро)   |
| commercial_national    | usp, сравнение, trust_signals, масштабные CTA                      | Featured Snippets, SERP-фичи, конкурентные USP          |
| ecom                   | product_card, price, наличие, доставка, фильтры, reviews, related  | Schema Product, каталожная навигация, фильтры-страницы   |
| saas                   | demo_cta, features_table, integrations, comparison, pricing_tiers  | Мультиязычность, English-блоки, API-документация         |
| info                   | author_block, date_updated, toc, sources, depth_content            | E-E-A-T автора, Topical Authority, длинный контент       |
| medical                | physician_block, license, ymyl_disclaimer, symptoms, treatment     | YMYL: каждый блок с экспертной атрибуцией                |
| legal                  | lawyer_block, case_results, urgency_cta, consultation_form         | Статус адвоката, срочность, конфиденциальность           |
| education              | program_card, schedule, enrollment_cta, student_reviews, faq       | Сезонность, длинный цикл решения, Student Journey        |

---

## ██ МЕТРИКИ УСПЕХА

| **Тип метрики**     | **Метрика**                                                           | **Как измерять**                                          |
|---------------------|----------------------------------------------------------------------|----------------------------------------------------------|
| Опережающая         | Покрытие: все типы страниц из архитектуры имеют структуру             | COUNT(page_type_structures) / COUNT(unique page_types)    |
| Опережающая         | Полнота: каждый тип имеет ≥5 блоков, CTA, Schema, mobile-адаптации   | QC-валидатор                                              |
| Запаздывающая       | После Шага 08: ТЗ копирайтерам полностью основаны на структурах      | Ревью Шага 08: 100% блоков из Step 04 отражены в ТЗ      |
| Запаздывающая       | Через 3 мес.: страницы с внедрённой структурой показывают рост CTR   | GSC/Метрика: CTR delta для страниц со структурой vs без   |
| Контрольная         | Ревью после Шага 05: КФ-блоки из Step 04 совпадают с аудитом Step 05 | Кросс-шаговый QC: пересечение блоков                      |

---

## ██ ИСТОРИЯ ИЗМЕНЕНИЙ

| **Версия** | **Дата**   | **Изменения**                                                                      |
|------------|------------|-------------------------------------------------------------------------------------|
| v1.0       | 2025-01    | Базовая структура: типы страниц + блоки                                             |
| v2.0       | 2025-06    | Добавлены CRO-элементы (Роль E), engine_tag, Schema-типы, mobile-адаптации          |
| v3.0       | 2025-12    | Маппинг персон и hunt_levels, RAG-обогащение, internal_linking_zones, SERP-фичи, niche_cluster адаптации, few-shot, fallback-стратегии |
