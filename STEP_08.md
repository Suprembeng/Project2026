# STEP_08.md — ТЗ на страницы

> **Модуль 3 архитектуры промптов.** Загружается вместе с CORE.md + PROJECT.md при выполнении Шага 08. ~4K токенов. Самодостаточен — содержит всё необходимое для генерации детального технического задания на страницу.

---

## ██ КАРТОЧКА ШАГА

| **Параметр** | **Значение** |
| --- | --- |
| ID | 08 |
| Название | ТЗ на страницы (Content Brief) |
| Блок | 2: Структура и контент |
| Основная роль | **F: Промпт-инженер** |
| Консультативные | A: SEO-стратег, E: CRO-аналитик, G: Инженер контрактов данных |
| Тип | Промпт (с pre-агрегатором входов и post-валидатором) |
| HitL | Нет (после QC ≥80 → Шаг 09 автоматически; QC 60–79 → ревью батча) |
| Приоритет ROI | **ВЫСОКИЙ** — ТЗ задаёт каркас всех контентных шагов 09–13 |
| Зависит от | Шаг 00 (персоны), Шаг 01 (СЯ), Шаг 04 (структура страниц), Шаг 05 (КФ), Шаг 06 (E-E-A-T), Шаг 07 (перелинковка) |
| Источник input (прямой) | **Шаг 07 (Перелинковка)** + агрегация из 00/01/04/05/06 через NocoDB |
| Передаёт в | Шаг 09 (текст. релевантность), 10 (LSI), 11 (метатеги), 12 (SEO-текст), 12.3 (UGC), 13 (Schema.org) |
| JSON-шаблон | **Шаблон 3: Генерация контента** (расширенный) |
| Режим Opus | Креативный контент: `temperature 0.4`, `top_p 0.95`, `max_tokens 8192` |
| Параллелизм | До 5 страниц одновременно (rate limit Opus + DataForSEO) |

---

## ██ ЦЕЛЬ

Сформировать **детальное техническое задание на одну страницу** с планом контент-блоков, мета-данными, AEO/GEO-элементами, требованиями E-E-A-T и КФ — так, чтобы шаги 09–13 могли генерировать финальный контент без повторной интерпретации стратегии. ТЗ ≠ финальный текст; ТЗ = каркас + правила + KPI блока.

---

## ██ КОНТРАКТ ВХОДНЫХ ДАННЫХ (JSON)

Агрегируется pre-скриптом из NocoDB (выходы шагов 00, 01, 04, 05, 06, 07). Один вызов = одна страница.

```json
{
  "project_id": "string",
  "page_input": {
    "page_id": "string (PG001...)",
    "planned_url": "string",
    "page_title_plan": "string",
    "content_type_hint": "landing│category│article│faq│product│service",
    "cluster_id": "string (из Шага 01)",
    "primary_query": "string",
    "secondary_queries": [{"query": "string", "engine": "yandex│google│both", "frequency": "number"}],
    "primary_persona_id": "string (из Шага 00, P001...)",
    "secondary_personas": ["string"]
  },
  "site_context": {
    "site_structure_node": "object (узел из Шага 04)",
    "internal_links_in":  [{"source_url": "string", "anchor": "string"}],
    "internal_links_out": [{"target_url": "string", "anchor_hint": "string"}],
    "commercial_factors_required": ["string (из Шага 05)"],
    "eeat_requirements":           ["string (из Шага 06)"]
  },
  "serp_snapshot": {
    "top10_formats":      [{"url": "string", "format": "string", "word_count": "number"}],
    "dominant_intent":    "informational│commercial│transactional│navigational",
    "featured_snippet":   "object│null",
    "ai_overview_present": "boolean",
    "people_also_ask":    ["string"]
  },
  "project_meta": {
    "niche_cluster":    "string",
    "geo":              "string",
    "language":         "string",
    "priority_engines": "yandex│google│both",
    "domain":           "string"
  }
}
```

---

## ██ КОНТРАКТ ВЫХОДНЫХ ДАННЫХ (JSON)

Наследует **Шаблон 3 (Генерация контента)**, расширен под специфику ТЗ. Роль G валидирует. Должен пройти структурный + содержательный + SEO + интентный QC.

```json
{
  "project_id": "string",
  "step_id": "08",
  "prompt_version": "string (step08_v1.0)",
  "timestamp": "ISO 8601",
  "page_url": "string",
  "page_id": "string",
  "content_type": "landing│category│article│faq│product│service",
  "target_intent": "informational│commercial│transactional│navigational",
  "primary_persona_id": "string",
  "secondary_personas": ["string"],
  "target_cluster_id": "string",
  "target_queries": [
    {"query": "string", "engine": "yandex│google│both", "priority": "primary│secondary│lsi", "frequency": "number"}
  ],
  "content_blocks": [
    {
      "block_id": "B01",
      "block_type": "h1│h2│h3│paragraph│faq│answer_block│cta│table│list│quote",
      "content": "string (тема/план блока, НЕ финальный текст — финал в Шаге 12)",
      "keywords": ["string"],
      "word_count_min": "number",
      "word_count_max": "number",
      "engine_tag": "[Я]│[G]│[ОБА]",
      "rationale": "string (почему этот блок здесь, со ссылкой на персону/SERP)"
    }
  ],
  "meta": {
    "title": "string (≤60 символов)",
    "description": "string (120–160 символов)",
    "og_title": "string",
    "og_description": "string",
    "canonical": "string│null"
  },
  "schema_markup": {
    "primary_type": "Article│Product│FAQPage│HowTo│LocalBusiness│Organization│Service│Breadcrumb│null",
    "required_fields": ["string"],
    "yandex_extra": ["string (Recipe, Question, Offer и т.п. — если применимо)"]
  },
  "aeo_elements": {
    "answer_block": "string (40–60 слов, прямой ответ на primary_query)│null",
    "faq_count": "number (≥3 для AEO-готовности)",
    "has_how_to": "boolean",
    "has_table": "boolean",
    "snippet_target": "paragraph│list│table│none"
  },
  "geo_elements": {
    "brand_mentions_min": "number",
    "expert_quotes_required": "boolean",
    "stats_required": "boolean",
    "citation_anchor_phrases": ["string"]
  },
  "internal_linking": {
    "links_in_planned":  [{"source_url": "string", "anchor": "string"}],
    "links_out_planned": [{"target_url": "string", "anchor": "string", "placement_block_id": "string"}]
  },
  "eeat_signals": {
    "author_required": "boolean",
    "author_credentials": "string│null",
    "publish_date": "boolean",
    "update_date": "boolean",
    "sources_min": "number",
    "certificates_required": "boolean (true для YMYL)"
  },
  "commercial_factors": {
    "nap_required": "boolean",
    "price_block": "boolean",
    "cta_blocks_min": "number",
    "trust_signals": ["string"]
  },
  "engine_tags": {
    "yandex_specific": ["string"],
    "google_specific": ["string"]
  },
  "serp_intent_match": {
    "top10_dominant_format": "string",
    "matches_intent": "boolean",
    "deviation_rationale": "string│null"
  },
  "image_requirements": {
    "count_min": "number",
    "alt_pattern": "string (шаблон: {keyword} + {context})",
    "schema_image": "boolean"
  },
  "total_word_count": {"min": "number", "max": "number"},
  "qc_score": "number 0–100"
}
```

---

## ██ ПРОМПТ ДЛЯ CLAUDE OPUS

### Системный промпт

```
Ты — Роль F: Senior Промпт-инженер с консультативной поддержкой Роли A
(SEO-стратег, право вето) и Роли G (контракты данных, валидация).

Задача: Сформировать детальное ТЗ на ОДНУ страницу — каркас, по которому
шаги 09–13 сгенерируют финальный контент, метатеги, схему и текст.

ПРАВИЛА:
1. Один вызов = ОДНА страница. Не объединяй несколько в массив.
2. content_blocks — это ПЛАН (темы, KPI блока), а НЕ финальный текст. Финал — Шаг 12.
3. Title строго ≤60 символов. Description строго в диапазоне 120–160 символов.
4. Минимум 3 FAQ для AEO-готовности (кроме чистых product-страниц).
5. answer_block: 40–60 слов, прямой ответ на primary_query, для Featured Snippet
   и Яндекс Нейро / Google AI Overview.
6. Каждый запрос и каждый content_block обязан иметь engine tag:
   yandex / google / both для запросов; [Я] / [G] / [ОБА] для блоков.
7. Структура страницы должна совпадать с dominant_format топ-10 SERP.
   Если отклоняешься — заполни serp_intent_match.deviation_rationale.
8. Учитывай niche_cluster:
   - commercial_local: NAP-блок, карта, отзывы, гео-якоря, CTA звонок
   - ecom: фильтры/цены/сравнения/Schema Product, отзывы товаров
   - medical/legal: YMYL — author_required=true, certificates_required=true,
     sources_min ≥ 3, expert_quotes_required=true
   - saas: сравнительные таблицы, интеграции, API, англ. термины
   - info: глубина, авторская экспертиза, обновление дат
9. internal_linking: используй ТОЛЬКО URL из site_context.internal_links_*.
   Не выдумывай новые URL.
10. По persona: учитывай query_style и content_preferences целевой персоны.
11. [Я]/[G] разметка обязательна при priority_engines = both.
12. Schema.org: обязательно подбери primary_type под content_type.
    Для Яндекса добавь yandex_extra если применимо (Recipe, Question, Offer).
13. NO галлюцинации: если данных недостаточно для блока — пиши null/0,
    а не выдумывай.

ВЫВОД: Валидный JSON по контракту OUTPUT. Без markdown-обёртки.
Без пояснений до/после. Весь текст — на РУССКОМ языке.
JSON-ключи — на английском (по контракту).
```

### Пользовательский промпт (шаблон)

```
Сформируй ТЗ на страницу:

=== СТРАНИЦА ===
URL: {{page_input.planned_url}}
Заголовок (план): {{page_input.page_title_plan}}
Тип контента (подсказка): {{page_input.content_type_hint}}
Кластер: {{page_input.cluster_id}}
Основной запрос: {{page_input.primary_query}}
Дополнительные запросы:
{{page_input.secondary_queries | json}}

=== ПЕРСОНЫ ===
Основная: {{page_input.primary_persona_id}}
Вторичные: {{page_input.secondary_personas | join(', ')}}

=== САЙТ ===
Узел структуры: {{site_context.site_structure_node | json}}
Входящие ссылки: {{site_context.internal_links_in | json}}
Исходящие ссылки: {{site_context.internal_links_out | json}}
Требуемые КФ: {{site_context.commercial_factors_required | join(', ')}}
Требования E-E-A-T: {{site_context.eeat_requirements | join(', ')}}

=== SERP-СНИМОК ===
Доминирующий формат топ-10: {{serp_snapshot.top10_formats | json}}
Доминирующий интент: {{serp_snapshot.dominant_intent}}
Featured Snippet: {{serp_snapshot.featured_snippet | json}}
AI Overview: {{serp_snapshot.ai_overview_present}}
PAA: {{serp_snapshot.people_also_ask | join(' / ')}}

=== ПРОЕКТ ===
Кластер ниши: {{project_meta.niche_cluster}}
Гео: {{project_meta.geo}}
Язык: {{project_meta.language}}
Приоритет ПС: {{project_meta.priority_engines}}
Домен: {{project_meta.domain}}

{{#if rag_context}}
=== КОНТЕКСТ ИЗ RAG (≤2000 токенов) ===
{{rag_context}}
{{/if}}

Верни ТЗ как валидный JSON по контракту OUTPUT.
```

---

## ██ RAG-ОБОГАЩЕНИЕ

| **Коллекция Qdrant** | **Запрос** | **Что инжектится** | **Лимит** |
| --- | --- | --- | --- |
| `personas` | `project_id + primary_persona_id` | JTBD, hunt_ladder, content_preferences, search_behavior | 800 ток. |
| `competitors` | `niche + geo + content_type_hint` | Структуры топ-3 страниц конкурентов по этому интенту | 600 ток. |
| `briefs_existing` | `project_id` | Уже выпущенные ТЗ на похожие страницы (анти-каннибализация) | 400 ток. |
| `aeo_patterns` | `niche_cluster + content_type` | Шаблоны answer_block / FAQ под нишу | 200 ток. |

**Жёсткий лимит RAG-контекста: 2000 токенов** (умещаемся в бюджет ~9–10K на вызов).

Если коллекция недоступна → продолжаем без неё, лог `WARNING: rag_partial`. Только `briefs_existing` критична — без неё пропуск семантического QC.

---

## ██ СКРИПТЫ CLAUDE CODE

### Pre-step: Агрегатор входов

```python
# step_08_pre_aggregate_inputs.py
# Назначение: Собрать в один JSON всё, что нужно для ТЗ одной страницы.
# Вход:  project_id, page_id (из очереди N8N)
# Выход: Полный INPUT по контракту Шага 08
# Действия:
# 1. NocoDB: получить узел структуры из Шага 04 (по page_id)
# 2. NocoDB: подтянуть кластер из Шага 01 (primary + secondary queries)
# 3. NocoDB: получить персону из Шага 00 + вторичные
# 4. NocoDB: входящие/исходящие ссылки из Шага 07
# 5. NocoDB: КФ из Шага 05, E-E-A-T требования из Шага 06
# 6. Сборка корректного JSON по контракту INPUT
# 7. Передача в Pre-step #2 для обогащения SERP
```

### Pre-step: SERP-снимок

```python
# step_08_pre_serp_snapshot.py
# Назначение: Получить топ-10 SERP по primary_query — основа интентного QC.
# Вход:  primary_query, geo, language, priority_engines
# Выход: serp_snapshot (вписывается в INPUT)
# Действия:
# 1. DataForSEO: SERP топ-10 (Яндекс и/или Google по priority_engines)
# 2. Для каждого URL: format-классификатор (article/category/landing/...)
# 3. Определить dominant_intent голосованием по топ-10
# 4. Извлечь Featured Snippet, AI Overview presence, People Also Ask
# 5. Кэш в NocoDB на 7 дней (по primary_query + geo)
# Обработка ошибок: 429 → бэкофф 10/30/90с; полный fail → INPUT без serp_snapshot,
#                   флаг degraded=true → штраф QC -10
```

### Post-step: Валидатор

```python
# step_08_post_validate.py
# Назначение: Структурный + содержательный + SEO + интентный QC ТЗ.
# Проверки:
# 1. JSON Schema (ajv) по контракту OUTPUT
# 2. Title: 1 ≤ len ≤ 60 символов
# 3. Description: 120 ≤ len ≤ 160 символов
# 4. answer_block: если не null → 40 ≤ слов ≤ 60
# 5. faq_count ≥ 3 (кроме content_type='product')
# 6. Каждый target_query имеет engine tag из enum
# 7. Каждый content_block имеет engine_tag из [Я]/[G]/[ОБА]
# 8. При priority_engines='both' — должны присутствовать И [Я], И [G] блоки
# 9. internal_linking.links_out: target_url ∈ site_context.internal_links_out
#     (без выдуманных URL)
# 10. serp_intent_match.matches_intent=true ИЛИ непустой deviation_rationale
# 11. Для YMYL (medical/legal): eeat_signals.author_required=true,
#     certificates_required=true, sources_min ≥ 3
# 12. content_blocks: ровно один блок типа h1
# 13. total_word_count.min ≥ медианы топ-10 × 0.7
```

### Post-step: Семантический QC (анти-каннибализация)

```python
# step_08_post_semantic_qc.py
# Назначение: Не плодим каннибализирующие страницы.
# Действия:
# 1. Эмбеддинг (page_title_plan + primary_query + answer_block)
# 2. Cosine search в Qdrant 'briefs_existing' по project_id
# 3. Если max_similarity ≥ 0.85 → флаг CANNIBALIZATION → ревью человеком
# 4. Иначе → upsert в 'briefs_existing' с тегом project_id + page_id
```

### Post-step: Сохранение в NocoDB

```python
# step_08_post_save.py
# Назначение: Передать ТЗ в потребителей (Шаги 09–13).
# Действия:
# 1. INSERT в таблицу 'page_briefs' с prompt_version и qc_score
# 2. Триггер Шага 09 (если qc_score ≥ 80) или ревью (60–79) или retry (<60)
# 3. Метрика в Grafana: brief_qc_score, brief_generation_time, retry_count
```

---

## ██ КРИТЕРИИ QC

| **Проверка** | **Порог** | **Вес** | **При провале** |
| --- | --- | --- | --- |
| JSON Schema валиден | Pass | 15 | Retry |
| Title ≤60 символов | Pass | 10 | Retry |
| Description 120–160 символов | Pass | 10 | Retry |
| answer_block 40–60 слов (если не null) | Pass | 10 | Retry |
| FAQ ≥ 3 (кроме product) | Pass | 10 | Retry |
| Engine tags на всех запросах и блоках | 100% | 10 | Retry |
| [Я]+[G] оба присутствуют при both | Pass | 5 | Retry |
| internal_linking без выдуманных URL | 100% | 10 | Retry |
| serp_intent_match совпадает или обоснован | Pass | 5 | Ревью |
| YMYL: E-E-A-T поля заполнены | Pass | 5 | Retry (для med/legal) |
| Семантика: <0.85 cosine с существующими | Pass | 5 | Ревью (каннибализация) |
| total_word_count.min ≥ 0.7 × медианы топ-10 | Pass | 5 | Ревью |

**Сумма весов = 100.** ≥80 → передача в Шаг 09; 60–79 → ревью батча; <60 → retry по протоколу v2 (см. CORE.md).

---

## ██ СТРУКТУРА N8N-ВОРКФЛОУ

| **Нода** | **Назначение** |
| --- | --- |
| 1 | Триггер: новая запись в очереди `pages_for_brief` (создаётся после Шага 07) |
| 2 | Загрузка CORE.md + PROJECT.md + STEP_08.md |
| 3 | `step_08_pre_aggregate_inputs.py` |
| 4 | `step_08_pre_serp_snapshot.py` (с retry/бэкофф) |
| 5 | Qdrant RAG (4 коллекции, лимит 2000 ток.) |
| 6 | Сборка финального промпта (system + user + RAG) |
| 7 | Claude Opus API (temp 0.4, top_p 0.95, max_tokens 8192) |
| 8 | Парсинг JSON (удаление markdown-обёртки → повторный парсинг при ошибке) |
| 9 | `step_08_post_validate.py` → провал → возврат на Ноду 6 (макс. 3 retry) |
| 10 | `step_08_post_semantic_qc.py` |
| 11 | `step_08_post_save.py` → NocoDB + триггер Шага 09 |
| 12 | Grafana: метрики qc_score, retry_count, latency |
| 13 | Цикл: следующая страница из очереди ИЛИ стоп |

**Параллелизм:** до 5 одновременных вызовов (rate limit Opus tier 2 + DataForSEO 30 RPS).

**Обработка ошибок:** Opus 429/500 → бэкофф 10→30→90с (макс. 3); Qdrant down → без RAG + лог; QC fail 3× → Telegram-алерт; невалидный JSON → удалить markdown → повторный парсинг → retry.

---

## ██ ПРИМЕР FEW-SHOT

Сокращённый, для grounding модели. Ниша — детская стоматология в Москве (продолжение примера из STEP_00).

```json
{
  "project_id": "dental_moscow_001",
  "step_id": "08",
  "prompt_version": "step08_v1.0",
  "timestamp": "2026-04-16T12:00:00Z",
  "page_url": "/uslugi/detskaya-stomatologiya/",
  "page_id": "PG014",
  "content_type": "category",
  "target_intent": "commercial",
  "primary_persona_id": "P001",
  "secondary_personas": ["P003"],
  "target_cluster_id": "CL_KIDS_DENT_MSK",
  "target_queries": [
    {"query": "детская стоматология москва", "engine": "both", "priority": "primary", "frequency": 8400},
    {"query": "лечение зубов у ребёнка", "engine": "both", "priority": "secondary", "frequency": 3200},
    {"query": "детский стоматолог рядом", "engine": "yandex", "priority": "secondary", "frequency": 1900}
  ],
  "content_blocks": [
    {
      "block_id": "B01", "block_type": "h1",
      "content": "H1 с гео-якорем и УТП без боли. Пример формулировки для копирайтера.",
      "keywords": ["детская стоматология", "москва"],
      "word_count_min": 4, "word_count_max": 9,
      "engine_tag": "[ОБА]",
      "rationale": "Топ-10 показывает: H1 с гео + benefit конвертит лучше чистого ключа"
    },
    {
      "block_id": "B02", "block_type": "answer_block",
      "content": "Прямой ответ для Яндекс Нейро / AI Overview: что такое детская стоматология, для какого возраста, ключевое УТП клиники.",
      "keywords": ["детская стоматология"],
      "word_count_min": 40, "word_count_max": 60,
      "engine_tag": "[ОБА]",
      "rationale": "AEO-блок для Featured Snippet и AI Overview (присутствует в SERP)"
    },
    {
      "block_id": "B03", "block_type": "h2",
      "content": "Услуги: лечение кариеса, пломбирование, удаление, ортодонтия. Каждая — кратко с ценой и ссылкой.",
      "keywords": ["лечение кариеса у детей", "детская ортодонтия"],
      "word_count_min": 200, "word_count_max": 350,
      "engine_tag": "[ОБА]",
      "rationale": "Раскрытие основного интента — что лечат"
    },
    {
      "block_id": "B04", "block_type": "h2",
      "content": "Врачи: блок с фото, ФИО, стаж, сертификаты — обязательно для YMYL.",
      "keywords": ["детский стоматолог москва"],
      "word_count_min": 150, "word_count_max": 250,
      "engine_tag": "[ОБА]",
      "rationale": "E-E-A-T для YMYL + commercial_local; PAA содержит запрос про врача"
    },
    {
      "block_id": "B05", "block_type": "h2",
      "content": "NAP-блок + карта 2GIS/Яндекс.Карты + время работы. Критично для commercial_local.",
      "keywords": ["детская стоматология рядом", "детская стоматология москва адрес"],
      "word_count_min": 80, "word_count_max": 120,
      "engine_tag": "[Я]",
      "rationale": "Геозависимость + Яндекс.Бизнес сигналы"
    },
    {
      "block_id": "B06", "block_type": "faq",
      "content": "FAQ ≥5 вопросов из People Also Ask: с какого возраста водить, больно ли, цена, наркоз, как подготовить ребёнка.",
      "keywords": ["детская стоматология возраст", "детская стоматология цена"],
      "word_count_min": 250, "word_count_max": 400,
      "engine_tag": "[ОБА]",
      "rationale": "AEO + FAQPage Schema; PAA вопросы из SERP-снимка"
    },
    {
      "block_id": "B07", "block_type": "cta",
      "content": "Кнопка записи онлайн + телефон. Дублировать после H1 и в конце.",
      "keywords": ["записаться к детскому стоматологу"],
      "word_count_min": 10, "word_count_max": 20,
      "engine_tag": "[ОБА]",
      "rationale": "КФ из Шага 05: онлайн-запись обязательна"
    }
  ],
  "meta": {
    "title": "Детская стоматология в Москве — лечение зубов у детей",
    "description": "Детская стоматология в Москве: лечение без боли, опытные врачи, лицензии. Запись онлайн, удобный график, цены от 1500 ₽. Звоните!",
    "og_title": "Детская стоматология в Москве — лечение без боли",
    "og_description": "Опытные детские стоматологи. Лечение, ортодонтия, профилактика. Запись онлайн.",
    "canonical": "https://example.ru/uslugi/detskaya-stomatologiya/"
  },
  "schema_markup": {
    "primary_type": "Service",
    "required_fields": ["name", "provider", "areaServed", "offers", "review"],
    "yandex_extra": ["Question (для FAQ)", "MedicalOrganization"]
  },
  "aeo_elements": {
    "answer_block": "Детская стоматология — это профильное направление лечения зубов у детей от 1 года до 18 лет. В клинике работают сертифицированные детские врачи, используется седация и психологическая адаптация. Приём, лечение кариеса, ортодонтия, профилактика — без боли и стресса для ребёнка.",
    "faq_count": 5,
    "has_how_to": false,
    "has_table": true,
    "snippet_target": "paragraph"
  },
  "geo_elements": {
    "brand_mentions_min": 4,
    "expert_quotes_required": true,
    "stats_required": true,
    "citation_anchor_phrases": ["по данным клиники", "врачи отмечают", "согласно протоколу"]
  },
  "internal_linking": {
    "links_in_planned": [
      {"source_url": "/", "anchor": "детская стоматология"},
      {"source_url": "/uslugi/", "anchor": "стоматология для детей"}
    ],
    "links_out_planned": [
      {"target_url": "/uslugi/detskaya-stomatologiya/lechenie-kariesa/", "anchor": "лечение кариеса у детей", "placement_block_id": "B03"},
      {"target_url": "/vrachi/?spec=detskij", "anchor": "детские стоматологи клиники", "placement_block_id": "B04"}
    ]
  },
  "eeat_signals": {
    "author_required": true,
    "author_credentials": "детский стоматолог, стаж от 5 лет, сертификат",
    "publish_date": true,
    "update_date": true,
    "sources_min": 3,
    "certificates_required": true
  },
  "commercial_factors": {
    "nap_required": true,
    "price_block": true,
    "cta_blocks_min": 2,
    "trust_signals": ["лицензия", "отзывы", "сертификаты врачей", "фото клиники"]
  },
  "engine_tags": {
    "yandex_specific": ["Яндекс.Бизнес виджет", "блок отзывов с Яндекс.Карт", "микроразметка для Нейро"],
    "google_specific": ["GBP-разметка", "FAQPage Schema", "оптимизация под AI Overview"]
  },
  "serp_intent_match": {
    "top10_dominant_format": "category_page_with_services_and_doctors",
    "matches_intent": true,
    "deviation_rationale": null
  },
  "image_requirements": {
    "count_min": 6,
    "alt_pattern": "{услуга} в {клиника} — {benefit}",
    "schema_image": true
  },
  "total_word_count": {"min": 1200, "max": 1800},
  "qc_score": 92
}
```

---

## ██ СТРАТЕГИЯ ОТКАТА

**Откат A (после 1-го fail):** Декомпозиция на 3 вызова: (1) `meta + answer_block + faq`, (2) `content_blocks + internal_linking`, (3) `eeat + commercial + schema`. Объединение Code-нодой.

**Откат B (после 2-го fail):** Сократить ТЗ до минимально жизнеспособного: только H1, answer_block, 3 FAQ, meta, schema_type. Помечаем `degraded=true`, флаг ручного расширения в NocoDB.

**Откат C (после 3-го fail):** Few-shot из этого STEP_08 как «золотой» шаблон, Opus адаптирует под текущую страницу. Жёсткий промпт «строго по структуре эталона». Если и это fail → Telegram-алерт.

**Откат D (semantic QC fail — каннибализация):** Не retry, а ревью человеком. Решение: объединить страницы / переориентировать новую на иной интент / отменить.

---

## ██ АДАПТАЦИИ ПОД КЛАСТЕР

| **Кластер** | **Что меняется в ТЗ** |
| --- | --- |
| **commercial_local** | Обязательны NAP-блок, карта (Яндекс.Карты + 2GIS для [Я], GBP для [G]), отзывы, гео-якоря в H1/H2, CTA «позвонить» дублируется |
| **commercial_national** | Усиление E-E-A-T, расширенные сравнительные таблицы, ссылочные блоки для линкбилдинга, Featured Snippet target = table |
| **ecom** | Schema Product обязательна, блоки фильтров, цена/наличие/отзывы товара, минимум 3 фото с alt по шаблону |
| **saas** | Сравнительные таблицы «X vs Y», блок интеграций/API, англ. термины разрешены, схема SoftwareApplication, AEO-фокус |
| **info** | Глубина 1500+ слов, авторская экспертиза в шапке, дата обновления критична, sources_min ≥5 |
| **medical** | YMYL: жёсткие E-E-A-T (`author_required=true`, `certificates_required=true`, `sources_min ≥3`), дисклеймеры, MedicalOrganization Schema |
| **legal** | YMYL: статус адвоката, ссылки на статьи закона, форма консультации, expert_quotes обязательны |
| **education** | Сезонные блоки (приёмная кампания), длинный CTA-funnel, Course Schema |

---

## ██ МЕТРИКИ И КРИТЕРИИ УСПЕХА

**Опережающие (на этапе генерации ТЗ):**
- Средний `qc_score` по батчу ≥ 85
- Доля retry ≤ 15%
- Доля каннибализаций (semantic QC fail) ≤ 5%
- Среднее время генерации одной ТЗ ≤ 45 секунд

**Запаздывающие (после Шагов 09–13):**
- Шаг 12 принимает ТЗ без переделки в ≥90% случаев
- Финальный текст по ТЗ проходит интентный QC (Шаг 41) с первого раза в ≥80% случаев
- Через 30 дней после публикации: ≥60% страниц в топ-30 по primary_query

**Контрольные:**
- Ручное ревью 5% случайных ТЗ редактором — соответствие персоне и интенту
- A/B: ТЗ с answer_block vs без — рост попадания в Featured Snippet / AI Overview

---

## ██ ИСТОРИЯ ИЗМЕНЕНИЙ

| **Версия** | **Дата** | **Изменения** |
| --- | --- | --- |
| v1.0 | 2026-04 | Базовая версия. Шаблон 3 расширен полями для AEO/GEO/КФ/E-E-A-T/internal_linking/SERP intent match. Pre-агрегатор + SERP-снимок + post-валидатор + semantic QC. Адаптации под 8 кластеров. |

---

> **Конец STEP_08.md.** Загружается с CORE.md (~3–4K) + PROJECT.md (~2K) + этот файл (~4K) = ~10K токенов на вызов Opus.
