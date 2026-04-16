# STEP_11.md — Метатеги

> **Самодостаточный модуль.** Загружается: `CORE.md` + `PROJECT.md` + этот файл. ~4.5K токенов.
> **Версия:** v1.0 · **Последнее обновление:** 2026-04-16

---

## ██ КАРТОЧКА ШАГА

| Параметр | Значение |
|---|---|
| **ID** | 11 |
| **Название** | Метатеги (title, description, OG, H1, AEO answer-block) |
| **Блок** | 3: Контент и оптимизация (09–13) |
| **Основная роль** | **F:** Промпт-инженер |
| **Консультативные** | **A:** SEO-стратег · **E:** CRO и поведенческий аналитик (CTR-оптимизация) · **G:** Инженер контрактов данных |
| **Тип** | Промпт (post-step валидация через Claude Code) |
| **HitL** | **Условный** — обязателен при `niche_cluster ∈ {medical, legal}` (YMYL) и при `qc_score ∈ [60,79]`; иначе автоматически → Шаг 12 |
| **Приоритет ROI** | ВЫСОКИЙ — прямое влияние на CTR в SERP, брендовая идентичность в сниппетах, цитируемость в AI-ответах (Яндекс.Нейро / Google AI Mode) |
| **Зависит от** | Шаг 10 (LSI-анализ) v1.1 · `PROJECT.md` · Qdrant-коллекции `content_chunks` (антиканнибализация), `serp_snippets` (benchmark) |
| **Передаёт в** | Шаг 12 (SEO-текст) · Шаг 13 (Schema.org + OG) · Шаг 15 (Canonical) · Шаг 18 (Вебмастер + GSC submission) · Шаг 29 (AEO — косвенно через `aeo_elements.answer_block`) |
| **JSON-шаблон** | Template 3: Content Generation (расширен для мульти-страничного выхода) |
| **Режим Opus** | JSON-вывод: `temperature 0.15`, `top_p 0.9`, `max_tokens 8192` |
| **Concurrency-лимит** | ≤ 3 одновременных вызовов Opus на проект (Redis Lock, как в Шаге 10) |
| **SLA шага** | ≤ 8 мин на прогон |
| **Golden test** | Прогон на нише `dental_moscow_001` (эталон верифицирован) |

---

## ██ ЦЕЛЬ

Сгенерировать для каждой целевой страницы **title**, **meta description**, **H1**, **OpenGraph/Twitter Cards** и **AEO answer-block** — с учётом LSI-терминов и семантических дыр из Шага 10, с разными акцентами для Яндекса (КФ-термины) и Google (E-E-A-T, Knowledge Graph), с 2–3 A/B-вариантами для последующего CTR-тестирования. Без каннибализации и keyword stuffing.

Шаг отвечает на вопрос: **«Какой сниппет в SERP и какой H1 на странице дают максимальный CTR и семантическое соответствие интенту пользователя?»**

---

## ██ КОНТРАКТ ВХОДНЫХ ДАННЫХ (JSON)

Вход — выход Шага 10 v1.1 + обогащение из Шагов 00/01/09 через `run_id`. **Роль G** верифицирует.

```json
{
  "run_id": "UUID v4 (сквозной от шага 00)",
  "project_id": "string",
  "prev_step_id": "10",
  "prompt_version": "string",
  "timestamp": "ISO 8601",
  "rag_unavailable": "boolean (наследуется)",
  "pages": [
    {
      "page_url": "string",
      "page_cluster": "string",
      "page_cluster_slug": "string",
      "page_type": "landing|category|article|faq|product",
      "target_queries": [
        {
          "query": "string",
          "engine": "yandex|google|both",
          "intent": "informational|commercial|transactional|navigational",
          "frequency": "number",
          "is_primary": "boolean"
        }
      ],
      "current_content": {
        "h1": "string (текущий H1, если есть)",
        "current_title": "string|null",
        "current_description": "string|null",
        "usp": "string|null (УТП бренда для страницы)"
      },
      "priority_terms_for_meta": ["string (≤5, из Шага 10 topic_coverage[])"],
      "lsi_items_meta_relevant": [
        {
          "lsi_term": "string",
          "term_normalized": "string",
          "engine_tag": "[Я]|[G]|[ОБА]",
          "priority": "critical|high|medium|low",
          "placement_hint": "h1|meta|intro|body|faq"
        }
      ],
      "brand_name": "string",
      "brand_phone": "string|null",
      "geo_modifier": "string|null (город/район для commercial_local)"
    }
  ],
  "project_context": {
    "niche_cluster": "string",
    "geo": "string",
    "language": "string",
    "priority_engines": "yandex|google|both",
    "domain": "string",
    "domain_age_years": "number"
  },
  "engine_split_from_step_10": {
    "yandex_lexical_notes": "string",
    "google_semantic_notes": "string"
  }
}
```

**Обязательные поля:** `run_id`, `pages[]` (≥1), у каждой — `target_queries[]` (≥1, минимум один `is_primary: true`), `priority_terms_for_meta[]` (наследуется из Шага 10), `current_content.h1`, `page_cluster_slug`.

---

## ██ КОНТРАКТ ВЫХОДНЫХ ДАННЫХ (JSON)

Наследует **Template 3: Content Generation**, расширен для мульти-страничного выхода, A/B-вариантов и AEO-блоков. **Роль G** валидирует.

```json
{
  "run_id": "UUID",
  "project_id": "string",
  "step_id": "11",
  "prompt_version": "step11_v1.0",
  "timestamp": "ISO 8601",
  "rag_unavailable": "boolean",
  "pages": [
    {
      "page_url": "string (строго ∈ input.pages[].page_url)",
      "page_cluster": "string",
      "page_type": "landing|category|article|faq|product",
      "content_blocks": [
        {
          "block_type": "h1",
          "content": "string (≤70 симв., уникален, не = title)",
          "keywords": ["string (использованные LSI)"],
          "word_count": "number"
        }
      ],
      "meta_primary": {
        "title": "string (45-60 симв. идеал, макс 70)",
        "title_length": "number",
        "description": "string (120-160 симв. идеал, 110-165 допустимо)",
        "description_length": "number",
        "og_title": "string (≤90 симв.)",
        "og_description": "string (≤200 симв.)",
        "og_image_alt": "string",
        "twitter_title": "string",
        "twitter_description": "string",
        "canonical_hint": "string|null (передаётся в Шаг 15)",
        "primary_query_used": "string (какой target_query инкорпорирован)",
        "primary_query_position": "number (позиция 1-N в title)",
        "power_words_used": ["string (слова-триггеры для CTR)"],
        "rationale": "string (≤200 симв., обоснование)"
      },
      "meta_variants": [
        {
          "variant_id": "B|C",
          "title": "string",
          "description": "string",
          "angle": "price|urgency|brand|expertise|social_proof|benefit",
          "rationale": "string"
        }
      ],
      "primary_variant": "A",
      "aeo_elements": {
        "answer_block": "string|null (≤60 слов, для Яндекс.Нейро и Google AI Mode)",
        "faq_count": "number (количество запланированных FAQ в Шаге 12)",
        "has_how_to": "boolean",
        "structured_answer_query": "string|null (на какой target_query отвечает answer_block)"
      },
      "schema_markup_hint": {
        "primary_type": "Article|Product|LocalBusiness|FAQPage|HowTo|Service|null",
        "required_props": ["string (подсказки для Шага 13)"]
      },
      "engine_tags": {
        "yandex_specific": ["string (КФ-термины, morph-варианты)"],
        "google_specific": ["string (E-E-A-T сигналы, entities)"],
        "both_used": ["string"]
      },
      "cannibalization_check": {
        "title_cosine_max": "number 0-1 (макс. сходство с другими title проекта)",
        "h1_cosine_max": "number 0-1",
        "warning": "boolean (true если cosine > 0.85)",
        "conflicting_pages": ["string"]
      },
      "ctr_signals": {
        "has_number": "boolean",
        "has_year": "boolean",
        "has_geo": "boolean",
        "has_brand": "boolean",
        "has_benefit": "boolean",
        "predicted_ctr_tier": "high|medium|low"
      },
      "qc_score_page": "number 0-100"
    }
  ],
  "summary": {
    "total_pages": "number",
    "avg_title_length": "number",
    "avg_description_length": "number",
    "pages_with_cannibalization_warn": "number",
    "yandex_optimized": "number (страниц с [Я]-акцентом)",
    "google_optimized": "number",
    "both_optimized": "number",
    "pages_with_aeo_block": "number"
  },
  "engine_split": {
    "yandex_notes": "string (общие паттерны для [Я] в прогоне)",
    "google_notes": "string (общие паттерны для [G])"
  },
  "qc_score": "number 0-100",
  "qc_score_per_page": [{"page_url": "string", "score": "number 0-100"}]
}
```

**Инварианты контракта (синхронизированы с промптом и post_validate):**

- `meta_primary.title_length` ∈ [45, 70]; `description_length` ∈ [110, 165].
- `meta_primary.title ≠ current_content.h1` (разные) И `meta_primary.title ≠ content_blocks[h1].content`.
- `primary_query` из `is_primary: true` target_query — присутствует в title в первых 30 символах.
- `meta_variants[].length ≥ 1` (минимум 1 доп. вариант + primary = 2 A/B-варианта).
- `cannibalization_check.warning == true` ⟹ `qc_score_page ≤ 75`.
- При `niche_cluster == "commercial_local"` — обязательно `ctr_signals.has_geo == true`.
- При `priority_engines == "yandex"` — `engine_tags.google_specific` пустой; `yandex_specific` непустой.
- При `priority_engines == "google"` — зеркально.
- `aeo_elements.answer_block` если заполнен — ≤ 60 слов (≈350 символов).
- `page_url` в `pages[]` ⊆ `input.pages[].page_url`.

---

## ██ АНТИГАЛЛЮЦИНАЦИОННЫЙ ПРОТОКОЛ

Наследуется из STEP_10 v1.1, адаптирован для контент-генерации:

1. **Использование LSI-терминов**: термины в `keywords[]` и `engine_tags[]` брать ТОЛЬКО из `input.lsi_items_meta_relevant[]` и `input.priority_terms_for_meta[]`. Не выдумывать новые.
2. **Запрет на непроверяемые утверждения**: в title/description запрещены фразы «№1», «лучший», «единственный», «гарантия 100%» без явного подтверждения в `current_content.usp`.
3. **Запрет на факты о бренде без источника**: если в `input` нет `brand_phone`, `domain_age_years > X` — не писать «работаем 20 лет» / «звоните сейчас +7…».
4. **Цены**: упоминание цены в title/description допустимо ТОЛЬКО если в `lsi_items_meta_relevant[]` есть термин с `term_type=intent_modifier` и `lsi_term` содержит «цена»/«от X руб.»; при этом указывать «от» без конкретной суммы.
5. **Географическое привязывание**: `geo_modifier` из input — единственный источник гео в title. Не выдумывать районы/метро.
6. **Контрольная проверка**: post_validate сверяет, что все слова в `meta_primary.power_words_used[]` реально присутствуют в `title + description`.

---

## ██ ПРОМПТ ДЛЯ CLAUDE OPUS

### Системный промпт

```
Ты — Роль F: Senior Промпт-инженер, специализирующийся на SEO-метатегах.
Консультируют: Роль A (SEO-стратег), Роль E (CRO/CTR-аналитик),
Роль G (контракты данных).

ЗАДАЧА: Сгенерировать для каждой целевой страницы title, description,
H1, OpenGraph/Twitter Cards и AEO answer-block. Максимизировать CTR
в SERP и семантическое соответствие интенту. Сделать 2-3 A/B-варианта
title+description для последующего тестирования.

════════════════════════════════════════════════════════════════
ЖЁСТКИЕ ЛИМИТЫ (нарушение = QC fail):
════════════════════════════════════════════════════════════════
- title: 45-60 символов идеал, 70 символов — абсолютный максимум
- description: 120-160 символов идеал, 110-165 — допустимо
- H1 ≠ title (по словам), ≠ current_content.h1 (если качество низкое)
  ИЛИ H1 = current_content.h1 при его высоком качестве (переиспользуй).
- og_title ≤ 90, og_description ≤ 200
- aeo_elements.answer_block ≤ 60 слов (≈350 симв.) при наличии
- primary_query (is_primary: true) — в первых 30 символах title

════════════════════════════════════════════════════════════════
АНТИГАЛЛЮЦИНАЦИОННЫЕ ПРАВИЛА:
════════════════════════════════════════════════════════════════
1. В `keywords[]` и `engine_tags` используй ТОЛЬКО термины из
   input.lsi_items_meta_relevant[] и input.priority_terms_for_meta[].
2. Запрещены непроверяемые утверждения: «№1», «лучший»,
   «единственный», «гарантия 100%», «20 лет опыта» без явного
   подтверждения в input.current_content.usp или project_context.
3. Цены: «от X руб.» только если LSI-термин содержит «цена/от»;
   не придумывать конкретные суммы.
4. Гео-модификатор: только из input.pages[].geo_modifier. Не
   выдумывать районы, станции метро, ориентиры.
5. Имя бренда: только из input.pages[].brand_name.
6. Поле `power_words_used[]` должно содержать только слова,
   физически присутствующие в сгенерированных title + description.

════════════════════════════════════════════════════════════════
СПЕЦИФИКА ПО ПОИСКОВИКУ:
════════════════════════════════════════════════════════════════
- [Я] ЯНДЕКС: Коммерческие факторы в title/desc (цена, наличие,
  запись, доставка, гарантия). Морфологическая гибкость: основной
  запрос может быть в форме, совпадающей с частотной в Wordstat.
  Для Яндекс.Нейро — answer_block ≤60 слов даёт прямой ответ
  на primary_query. Микрогеография (метро, район) в [commercial_local].
  CTR-триггеры: «Быстро», «Без боли», «Сегодня», «Рядом», конкретные цифры.
- [G] GOOGLE: Бренд в title (обычно в конце через «—» или «|»).
  E-E-A-T сигналы: упоминание лицензии/сертификата/автора для YMYL.
  Entities из Knowledge Graph предпочтительны (бренды, локации,
  медицинские термины). CTR-триггеры: год («2026»), цифры,
  value proposition. Избегать clickbait.
- [ОБА]: Базовый запрос + основное предложение.

════════════════════════════════════════════════════════════════
ФОРМУЛЫ А/B-ВАРИАНТОВ (primary + 2 дополнительных):
════════════════════════════════════════════════════════════════
Primary — безопасный сбалансированный вариант.
Variant B (angle ∈ {price, urgency}) — прицел на transactional.
Variant C (angle ∈ {expertise, social_proof, brand}) — прицел на
доверие/бренд. Если priority_engines=yandex — variant B обычно
«price+urgency»; если google — variant C обычно «expertise».

════════════════════════════════════════════════════════════════
ПРАВИЛА ФОРМИРОВАНИЯ:
════════════════════════════════════════════════════════════════
1. Не дублируй primary_query в title и description дословно —
   используй синонимы/LSI из input.
2. Не используй эмодзи в title/description (риск фильтрации Яндекса).
   Допустимо только в og_*.
3. Запрещены CAPS-слова (кроме зарегистрированных аббревиатур MBA, IT
   и названий брендов ICON, MacBook и т.п.).
4. Разделители в title: « — » (em-dash) или « | » (pipe); не «/».
5. Title: структура по приоритету:
   [commercial_local, ecom]: {Primary Query} {Modifier|USP} — {Brand, City}
   [info]: {Primary Query}: {Specification} — {Brand|Year}
   [saas]: {Brand} | {Primary Query} — {USP}
   [medical, legal]: {Primary Query} в {City} — {Brand} ({License ID})
6. Description: структура 3 части через точку или запятую:
   часть 1: что делаем (primary query + LSI)
   часть 2: ключевое преимущество (USP или КФ)
   часть 3: CTA (звоните, запишитесь, узнайте, рассчитайте)
7. H1: уникален от title, выражает ту же суть, но
   длиннее/описательнее. ≥25 символов. Содержит primary_query
   в «человеческой» форме.
8. schema_markup_hint.primary_type выбирать по page_type:
   landing → Service/LocalBusiness; article → Article; product → Product;
   faq → FAQPage; category → CollectionPage; howto → HowTo.

════════════════════════════════════════════════════════════════
АНТИКАННИБАЛИЗАЦИЯ:
════════════════════════════════════════════════════════════════
Если в input передан Qdrant-контекст existing_titles[] — не делай
новый title с cosine-подобием >0.85 к любому существующему.
При невозможности избежать сходства — добавь дифференциатор
(гео, аудитория, формат: «для детей», «в Москве», «онлайн»)
и помечай cannibalization_check.warning=true с указанием
conflicting_pages[].

АДАПТАЦИЯ ПОД niche_cluster — см. секцию адаптаций модуля.

ВЫВОД: Валидный JSON строго по контракту OUTPUT. Без markdown-обёртки,
без комментариев, без префикса ```json. Весь текст в полях title,
description, h1, og_*, twitter_*, rationale, answer_block, notes — 
на РУССКОМ языке. Ключи JSON и enum-значения — на английском.
```

### Пользовательский промпт (шаблон с переменными)

```
Сгенерируй метатеги для проекта {{project_id}} (run_id: {{run_id}}).

КОНТЕКСТ ПРОЕКТА:
- Ниша: {{project_context.niche_cluster}}
- Гео: {{project_context.geo}}
- Язык: {{project_context.language}}
- Приоритетные поисковики: {{project_context.priority_engines}}
- Домен: {{project_context.domain}} (возраст: {{project_context.domain_age_years}} лет)

ИНСАЙТЫ ИЗ ШАГА 10 (LSI-анализ):
- Яндекс: {{engine_split_from_step_10.yandex_lexical_notes}}
- Google: {{engine_split_from_step_10.google_semantic_notes}}

СТРАНИЦЫ ({{pages.length}} шт.):

{{#each pages}}
─────────────────────────────────────
Страница #{{@index}}: {{page_url}} [{{page_cluster_slug}}]
Кластер: {{page_cluster}} · Тип: {{page_type}}
Бренд: {{brand_name}}{{#if brand_phone}} · Тел: {{brand_phone}}{{/if}}
Гео-модификатор: {{geo_modifier | default('—')}}
УТП: {{current_content.usp | default('не задано')}}

Текущие (для сверки на дубли):
  H1: {{current_content.h1}}
  Title: {{current_content.current_title | default('—')}}
  Description: {{current_content.current_description | default('—')}}

Целевые запросы:
{{#each target_queries}}
  {{#if is_primary}}★ PRIMARY{{else}}  вторичный{{/if}}: "{{query}}" [{{engine}}] · intent={{intent}} · freq={{frequency}}
{{/each}}

Приоритетные LSI-термины для мета (из шага 10):
{{priority_terms_for_meta | join(', ')}}

LSI-items с placement_hint ∈ {meta, h1, intro}:
{{#each lsi_items_meta_relevant}}
  - "{{lsi_term}}" [{{engine_tag}}] priority={{priority}} placement={{placement_hint}}
{{/each}}
─────────────────────────────────────
{{/each}}

{{#if existing_titles_rag}}
СУЩЕСТВУЮЩИЕ TITLE НА ПРОЕКТЕ (для антиканнибализации):
{{existing_titles_rag}}
{{/if}}

{{#if competitor_snippets_rag}}
SNIPPETS ТОП-3 КОНКУРЕНТОВ (benchmark, не копировать):
{{competitor_snippets_rag}}
{{/if}}

ЗАДАНИЕ: Для каждой страницы сформируй meta_primary, 2 варианта
meta_variants, H1, aeo_elements (при intent=informational),
schema_markup_hint, engine_tags, cannibalization_check (признаки).
Заполни summary, engine_split, qc_score, qc_score_per_page[].
Используй run_id={{run_id}}, step_id="11", prompt_version="step11_v1.0".
```

---

## ██ RAG-ОБОГАЩЕНИЕ

| Коллекция | Запрос | Извлекаем | Лимит токенов |
|---|---|---|---|
| `content_chunks` | embedding(primary_query) для каждой страницы | Существующие title/H1 проекта → `existing_titles_rag` для антиканнибализации | 500 |
| `serp_snippets` | `target_queries[]` (кешированные из Шага 10) | Топ-3 title/description конкурентов как benchmark (не для копирования) | 400 |
| `personas` | `page_cluster + target_intent` | Language style гайд (формальный/разговорный) для tone-of-voice | 200 |

**Итоговый лимит RAG:** 1100 токенов.
**Стратегия:** dense search (BGE-M3), top-k=5 на коллекцию, фильтр `project_id`.
**Fallback:** если Qdrant недоступен — `rag_unavailable=true`, Opus работает без benchmark-сниппетов, `qc_score` cap 85 (как в Шаге 10).

---

## ██ СКРИПТЫ CLAUDE CODE

Тип шага — **Промпт**, поэтому pre-scripts отсутствуют. Реализованы только post-step валидаторы.

### Post-step 1: `step_11_post_validate.py`

```python
# Назначение: Rule-based валидация метатегов
# Вход:  output от Opus
# Выход: validation_report.json + qc_score + qc_errors[]
#
# Проверки:
# 1.  JSON Schema (ajv): все обязательные поля, типы
# 2.  page_url ⊆ input.pages[].page_url
# 3.  Длина title: 45 ≤ len ≤ 70 (строго), title_length совпадает
# 4.  Длина description: 110 ≤ len ≤ 165
# 5.  H1 ≠ meta_primary.title (case-insensitive Jaccard < 0.6)
# 6.  primary_query в первых 30 символах title (подстрочное совпадение
#     по леммам через pymorphy3)
# 7.  priority_terms_for_meta покрыты ≥ 80%: минимум 80% терминов
#     из input фигурируют в (title + description + H1) в любой форме
# 8.  Антистаффинг: ни один термин не повторяется >2 раз в (title+desc)
# 9.  meta_variants.length ≥ 1 (то есть ≥2 варианта всего)
# 10. cannibalization_check.warning ⇒ qc_score_page ≤ 75
# 11. ctr_signals согласованность:
#     - has_number ↔ /\d+/ в title
#     - has_year ↔ /20\d{2}/ в title
#     - has_geo ↔ geo_modifier в title/desc
#     - has_brand ↔ brand_name в title/desc
# 12. engine_tags ↔ priority_engines (как в Шаге 10):
#     - yandex → google_specific пустой
#     - google → yandex_specific пустой
#     - both → оба непустые
# 13. Антигаллюцинация:
#     - все слова power_words_used[] реально в (title + description)
#     - все термины в engine_tags.* ∈ input.lsi_items_meta_relevant[]
#       ∪ input.priority_terms_for_meta[]
# 14. aeo_elements.answer_block — при заполнении ≤60 слов
# 15. schema_markup_hint.primary_type — валидный enum schema.org
# 16. Запрет эмодзи в title/description (regex Unicode emoji classes)
# 17. Запрет CAPS-слов (len>3, все заглавные) кроме whitelist
#     (ICON, USA, ЦАО, ЮАО, MBA, PhD, IT и т.п.)
# 18. Разделители: title содержит «—» (em-dash) или «|» — структурно
# 19. summary-согласованность:
#     - sum(yandex_optimized, google_optimized, both_optimized) = total_pages
#     - avg_title_length = mean(pages[].meta_primary.title_length)
# 20. При niche_cluster=commercial_local: ctr_signals.has_geo=true
#
# Антигаллюцинационная сверка (обязательная):
#   для каждого термина в power_words_used и engine_tags.*:
#     assert term lemma ∈ (input.lsi_items_meta_relevant ∪ priority_terms_for_meta)
#
# При провале: qc_errors[] с путями JSONPath → retry до 3 раз.
# Защита от петли: hash(qc_errors) одинаковый 2 раза → декомпозиция.
```

### Post-step 2: `step_11_post_cannibalization.py`

```python
# Назначение: Семантическая антиканнибализация title/H1 против всего корпуса проекта
# Вход:  pages[] с title и h1, Qdrant collection 'content_chunks'
# Выход: cannibalization_report[] (блокирует при critical)
#
# Действия:
# 1. Для каждой новой пары (title, h1):
#    - vec_title = embed(title)  через BGE-M3
#    - vec_h1 = embed(h1)
# 2. Qdrant HNSW search top-10 в 'content_chunks':
#    filter: project_id=X AND page_url != current AND step_id in (11, legacy)
# 3. title_cosine_max = max из 10; h1_cosine_max аналогично.
# 4. Если cosine > 0.92 — БЛОКИРОВАТЬ (критическая каннибализация):
#    cannibalization_check.warning=true, qc_score_page cap 60, retry.
# 5. Если cosine ∈ (0.85, 0.92] — WARN (мягкая каннибализация):
#    warning=true, qc_score_page cap 75. conflicting_pages[] заполнен.
# 6. Результат мержится в output.pages[].cannibalization_check.
#
# ВАЖНО: собственную страницу исключать (must_not page_url).
```

### Post-step 3: `step_11_post_persist.py`

```python
# Назначение: Персистентность + подготовка для шагов 12, 13, 15, 18
# Действия:
# 1. Staging insert в NocoDB seo_meta_tags_staging (run_id, project_id, step_id=11)
# 2. Qdrant upsert 'content_chunks':
#    point_id = md5(project_id + step_id + page_url)
#    vector = embed(title + " " + description)
#    payload: {project_id, page_url, step_id:11, run_id, title, description, h1}
#    удалить старые точки WHERE project_id=X AND step_id=11 AND run_id != current
# 3. SWAP: insert в seo_meta_tags при успехе обоих
# 4. Лог в seo_step_runs
# 5. Подготовка триггеров:
#    - trigger_12: output_contract
#    - trigger_13: передача schema_markup_hint → Шаг 13 (Schema.org)
#    - trigger_15: передача canonical_hint → Шаг 15 (Canonical)
#    - trigger_18: список страниц для индексации в Яндекс.Вебмастер/GSC
```

---

## ██ КРИТЕРИИ QC

| № | Проверка | Порог | Вес | При провале |
|---|---|---|---|---|
| 1 | JSON Schema валидна | Pass | 8 | Retry |
| 2 | `page_url ⊆ input` | 100% | 5 | Retry |
| 3 | `title_length` ∈ [45,70] | 100% стр | 10 | Retry |
| 4 | `description_length` ∈ [110,165] | 100% стр | 10 | Retry |
| 5 | `H1 ≠ title` (Jaccard < 0.6) | 100% стр | 8 | Retry |
| 6 | Primary query в первых 30 симв. title | 100% стр | 8 | Retry |
| 7 | Покрытие `priority_terms_for_meta` | ≥ 80% | 8 | Ревью |
| 8 | Антистаффинг: термин ≤ 2 повторов | 100% | 5 | Retry |
| 9 | `meta_variants.length ≥ 1` | 100% стр | 5 | Retry |
| 10 | `engine_tags ↔ priority_engines` | 100% | 5 | Retry |
| 11 | Антигаллюцинация: `power_words_used ⊆ (title∪desc)` | 100% | 8 | Retry |
| 12 | Антигаллюцинация: `engine_tags.* ⊆ input LSI-terms` | 100% | 8 | Retry |
| 13 | Каннибализация title: `cosine_max ≤ 0.92` | 100% стр | 10 | Retry |
| 14 | Каннибализация warn: `cosine_max ≤ 0.85` | 100% стр | 3 | Ревью |
| 15 | `aeo.answer_block` ≤ 60 слов (если есть) | 100% | 3 | Retry |
| 16 | Запрет эмодзи в title/description | 100% | 3 | Retry |
| 17 | Запрет некорректных CAPS-слов | 100% | 2 | Retry |
| 18 | `ctr_signals` согласованы с содержимым | 100% | 3 | Retry |
| 19 | `commercial_local` ⇒ `has_geo=true` | 100% | 5 | Retry |
| 20 | `summary`-согласованность | 100% | 2 | Retry |
| — | Соответствие `niche_cluster` | ≥ 70% | — | Ревью |

**Скоринг:** сумма весов пройденных / 119 → ×100.
`rag_unavailable == true` ⇒ cap 85.
**Результаты:** ≥ 80 → Шаги 12, 13, 15, 18 · 60–79 → ревью (условный HitL) · < 60 → retry (до 3).

---

## ██ СТРУКТУРА N8N-ВОРКФЛОУ

```
Нода 1:  Триггер (webhook от Шага 10 ИЛИ manual) → наследование run_id
Нода 2:  Загрузка output Шага 10 из NocoDB (по run_id, step_id=10)
Нода 3:  Загрузка данных страниц из NocoDB (seo_page_analysis, шаги 09/04)
Нода 4:  Загрузка PROJECT.md
Нода 5:  Redis Lock acquire: opus_concurrent_calls ≤ 3
Нода 6:  Checkpoint check (если prompt_response по run_id есть — skip к Ноде 9)
╭─── Параллельная ветвь (pre-RAG) ───╮
Нода 7a: Qdrant RAG «content_chunks» (existing_titles)
Нода 7b: Qdrant RAG «serp_snippets» (competitor benchmark)
Нода 7c: Qdrant RAG «personas» (tone-of-voice)
╰──────── Merge (Wait All) ──────────╯
Нода 8:  Сборка промпта (Template + merge RAG)
Нода 9:  Claude Opus API (temp 0.15, top_p 0.9, max_tokens 8192)
         └── retry 429/500: exp. backoff 10→30→90с
Нода 10: Парсинг JSON (regex-strip markdown → json.loads → retry)
Нода 11: step_11_post_validate.py
         ├── pass → Нода 12
         ├── fail retry 1 → Нода 8 с qc_errors[] в промпт
         ├── fail retry 2 → Декомпозиция по страницам (Split → N вызовов)
         ├── fail retry 3 → Fallback: только title+description, без вариантов/AEO
         └── fail retry 4 → Telegram алерт + DLQ + блок следующих шагов
Нода 12: step_11_post_cannibalization.py (семантический QC)
         ├── cosine > 0.92 → возврат на Нода 8 с инструкцией «добавить
         │    дифференциатор» (retry)
         └── cosine ≤ 0.92 → Нода 13
Нода 13: HitL Gate: IF niche_cluster ∈ {medical, legal} OR qc_score ∈ [60,79]
         → Telegram approval → WAIT до 24ч
Нода 14: step_11_post_persist.py (NocoDB staging + Qdrant + SWAP)
Нода 15: Grafana metrics push:
         meta_pages_count, avg_qc_score, retry_count, duration_ms,
         cannibalization_blocks, hitl_required, ctr_tier_distribution
Нода 16: Release Redis Lock
Нода 17: ╭── параллельные триггеры ──╮
         ├ Шаг 12 (SEO-текст)         │
         ├ Шаг 13 (Schema.org + OG)   │
         ├ Шаг 15 (Canonical)         │
         ├ Шаг 18 (GSC/Вебмастер)     │
         ╰ async Шаг 29 (AEO citation)│
```

### Обработка ошибок

- **Opus 429/500:** exp. backoff (10→30→90с), 3 попытки; circuit-break на 5 мин после 5 подряд 5xx.
- **Qdrant недоступен:** `rag_unavailable=true` → Opus без benchmark → `qc_score` cap 85.
- **Каннибализация cosine > 0.92:** принудительный retry с директивой в промпт добавить дифференциатор.
- **QC fail 4x:** Telegram алерт + DLQ (`seo_meta_dlq`) + блок Шагов 12, 13, 15.
- **Invalid JSON:** regex-strip → retry parse → fail → retry Opus.
- **Timeout ноды 9 (300s):** прерывание → retry с декомпозицией.
- **Checkpoint resume:** при рестарте чтение `prompt_response.json` из NocoDB по `run_id`.
- **Защита от retry-петли:** хеш qc_errors повторяется → принудительная декомпозиция.

---

## ██ ПРИМЕР FEW-SHOT (сокращённый, 1 страница)

**Ниша:** `dental_moscow_001` (детская стоматология, Москва, `commercial_local`, `both`).
**Страница:** `/uslugi/lechenie-kariesa-u-detey/`.
**Primary query:** `лечение кариеса у детей` [`both`, commercial, freq=4800].

```json
{
  "run_id": "8f4a1b2c-...",
  "project_id": "dental_moscow_001",
  "step_id": "11",
  "prompt_version": "step11_v1.0",
  "timestamp": "2026-04-16T12:18:44Z",
  "rag_unavailable": false,
  "pages": [
    {
      "page_url": "/uslugi/lechenie-kariesa-u-detey/",
      "page_cluster": "detskaya_stomatologiya_karies",
      "page_type": "landing",
      "content_blocks": [
        {
          "block_type": "h1",
          "content": "Лечение кариеса у детей в Москве — без боли и сверления",
          "keywords": ["лечение кариеса у детей", "без сверления", "Москва"],
          "word_count": 9
        }
      ],
      "meta_primary": {
        "title": "Лечение кариеса у детей в Москве — от 2500 ₽ | Денталь",
        "title_length": 57,
        "description": "Лечим кариес у ребёнка без боли: серебрение, ICON, анестезия. Опытные детские стоматологи. Цена от 2500 ₽, запись онлайн. м. Сокол.",
        "description_length": 139,
        "og_title": "Детский стоматолог в Москве: лечение кариеса без боли — Денталь",
        "og_description": "Серебрение молочных зубов, технология ICON, безопасная анестезия. Опыт 12 лет, запись онлайн за 1 минуту.",
        "og_image_alt": "Детский стоматолог лечит кариес у улыбающегося ребёнка",
        "twitter_title": "Лечение кариеса у детей — Денталь Москва",
        "twitter_description": "Без боли, с серебрением и ICON. От 2500 ₽. Запись онлайн.",
        "canonical_hint": "https://dental-example.ru/uslugi/lechenie-kariesa-u-detey/",
        "primary_query_used": "лечение кариеса у детей",
        "primary_query_position": 1,
        "power_words_used": ["без боли", "от", "онлайн"],
        "rationale": "Primary в начале, КФ (цена+гео+бренд) — для Яндекса. Power-words «без боли» — эмоциональный триггер."
      },
      "meta_variants": [
        {
          "variant_id": "B",
          "title": "Детский стоматолог Москва — кариес за 1 визит | Денталь",
          "description": "Вылечим кариес у ребёнка за один приём: серебрение, ICON, седация. Цена от 2500 ₽. м. Сокол, запись онлайн.",
          "angle": "urgency",
          "rationale": "«За 1 визит» — триггер срочности для занятых родителей."
        },
        {
          "variant_id": "C",
          "title": "Лечение кариеса у детей — Денталь, 12 лет опыта в Москве",
          "description": "Детские стоматологи с опытом 12 лет. Серебрение, ICON, анестезия без страха. Лицензия. Запись онлайн, м. Сокол.",
          "angle": "expertise",
          "rationale": "Акцент на E-E-A-T: опыт + лицензия — для Google."
        }
      ],
      "primary_variant": "A",
      "aeo_elements": {
        "answer_block": "Лечение кариеса у детей в Москве проводится без сверления методами серебрения молочных зубов (для детей 2–4 лет) и технологии ICON (инфильтрация кариеса без бормашины). При необходимости используется безопасная детская анестезия или седация. Средняя стоимость от 2500 ₽ за зуб.",
        "faq_count": 6,
        "has_how_to": false,
        "structured_answer_query": "лечение кариеса у детей"
      },
      "schema_markup_hint": {
        "primary_type": "Service",
        "required_props": ["provider (LocalBusiness)", "areaServed (Moscow)", "serviceType", "offers.priceRange"]
      },
      "engine_tags": {
        "yandex_specific": ["цена от 2500 ₽", "запись онлайн", "м. Сокол"],
        "google_specific": ["лицензия", "12 лет опыта", "ICON"],
        "both_used": ["лечение кариеса у детей", "без боли", "серебрение"]
      },
      "cannibalization_check": {
        "title_cosine_max": 0.72,
        "h1_cosine_max": 0.68,
        "warning": false,
        "conflicting_pages": []
      },
      "ctr_signals": {
        "has_number": true,
        "has_year": false,
        "has_geo": true,
        "has_brand": true,
        "has_benefit": true,
        "predicted_ctr_tier": "high"
      },
      "qc_score_page": 91
    }
  ],
  "summary": {
    "total_pages": 1,
    "avg_title_length": 57,
    "avg_description_length": 139,
    "pages_with_cannibalization_warn": 0,
    "yandex_optimized": 0,
    "google_optimized": 0,
    "both_optimized": 1,
    "pages_with_aeo_block": 1
  },
  "engine_split": {
    "yandex_notes": "КФ-акцент: цена + м. Сокол + «запись онлайн» в description. Морфология primary_query сохранена в частотной форме Wordstat. Answer_block ≤60 слов — заточен под Яндекс.Нейро.",
    "google_notes": "E-E-A-T через variant C (12 лет опыта, лицензия). Entity «ICON» распознаваем Knowledge Graph. Schema Service с LocalBusiness provider — для Google Rich Results."
  },
  "qc_score": 91,
  "qc_score_per_page": [{"page_url": "/uslugi/lechenie-kariesa-u-detey/", "score": 91}]
}
```

---

## ██ СТРАТЕГИЯ ОТКАТА (Retry Protocol v2)

- **Retry 1** — `qc_errors[]` в промпт, та же стратегия.
- **Retry 2 (Откат A)** — декомпозиция по страницам: N параллельных вызовов Opus по 1 странице; merge и повторная валидация.
- **Retry 3 (Откат B)** — упрощение контракта: только `meta_primary` (без `meta_variants` и `aeo_elements`). Помечать `fallback_mode=true` в NocoDB.
- **Retry 4 (Откат C)** — только `title` + `description` (без H1, OG, Twitter, schema_hint). Остальные поля генерирует Шаг 13 и Шаг 12 по дефолтам.
- **Retry 5 (Откат D / HitL)** — Telegram-алерт с draft-ссылкой; SEO-специалист правит вручную в NocoDB → ручной триггер Шагов 12, 13.

**Защита от бесконечной петли:** если `qc_errors` между retry N и N+1 идентичны по хешу → принудительный переход на следующий откат.

---

## ██ АДАПТАЦИИ ПОД КЛАСТЕР

| Кластер | Структура title | Акценты description | Обязательные ctr_signals | AEO answer_block |
|---|---|---|---|---|
| **commercial_local** | `{Query} в {City}/{District} — от {Price} \| {Brand}` | КФ (цена, запись, м. X, часы), USP, CTA «позвонить/записаться» | `has_geo=true`, `has_number`, `has_brand` | Опционально (если intent=info в target_queries) |
| **commercial_national** | `{Query} — {USP} \| {Brand}` | Экспертиза, гарантия, доставка по РФ, year | `has_number`, `has_brand` | Рекомендовано |
| **ecom** | `{Product/Query} — купить в {Brand}, доставка от {City}` | Цена, акция, доставка, рейтинг, количество отзывов | `has_number` (цена/рейтинг), `has_brand` | Опционально |
| **saas** | `{Brand} \| {Query} — {USP}` | ROI, интеграции, free trial, длительность trial | `has_brand`, `has_benefit` | Рекомендовано (сравнения, how-to) |
| **info** | `{Query}: {Specification} — {Brand}/{Year}` | Автор, экспертиза, полнота, год обновления | `has_year`, `has_benefit` | **Обязательно** (AEO — основной ROI) |
| **medical (YMYL)** | `{Query} в {City} — {Brand} (лицензия № {N})` | Врачи с ФИО, опыт лет, лицензия, запись, м. X | `has_geo`, `has_brand`, лицензия в desc | **Обязательно** + `has_how_to=false` |
| **legal (YMYL)** | `{Query} в {City} — {Brand}, адвокат {ФИО}` | Адвокатский статус (№ реестра), опыт дел, бесплатная консультация | `has_geo`, `has_brand` | **Обязательно** |
| **education** | `{Query} — {Format}, {Duration} \| {Brand}` | Сертификация, длительность, уровень, стоимость, рассрочка | `has_brand`, `has_benefit` | Рекомендовано |

**Особые правила:**
- YMYL — **обязательно** указывать лицензию/адвокатский номер в description (если доступно в `current_content.usp`).
- `saas` — допустимы английские термины в title (API, SaaS, CRM), но не более 1 английского слова.
- `ecom` — title может содержать модель/артикул (`iPhone 15 Pro 256GB`).

---

## ██ МЕТРИКИ УСПЕХА

**Опережающие (сразу после Шага 11):**
- `qc_score ≥ 80` для ≥ 90% страниц (по `qc_score_per_page[]`).
- 0 страниц с критической каннибализацией (cosine > 0.92).
- ≤ 10% страниц с мягкой каннибализацией (0.85 < cosine ≤ 0.92).
- `avg_title_length` ∈ [50, 60]; `avg_description_length` ∈ [125, 155].
- ≥ 80% страниц имеют минимум 1 `meta_variant` для A/B.
- Покрытие `priority_terms_for_meta` ≥ 80% на страницу.
- `pages_with_aeo_block / total_pages ≥ 70%` для `info` и YMYL кластеров.

**Запаздывающие (после публикации + переиндексация 14–30 дней):**
- Рост CTR в GSC / Вебмастере по страницам ≥ +15% относительно baseline.
- Рост impressions по `priority_terms_for_meta` ≥ +20% за 30 дней.
- Снижение bounce-rate на страницах ≥ −5 пп (связь: title соответствует контенту).
- Появление в AI-ответах (Шаг 40.3) по primary_query на ≥ 30% страниц с `aeo_elements.answer_block`.
- Rich Results появление для `schema_markup_hint.primary_type ∈ {Service, FAQPage, Product}`.

**Контрольные:**
- Спот-чек 10% страниц вручную: title/description звучат «человечно», не keyword-stuffing.
- Сравнение с golden set: отклонение `predicted_ctr_tier` от факта > 20% → ревизия CTR-сигналов.
- A/B-эксперимент (через Шаг 40): primary vs variant B/C через 30 дней — победитель становится новым primary.

**Бюджет токенов:** 3–5K вход + 2–4K выход (меньше, чем Шаг 10 — нет numeric signals). Предел — 15 страниц на вызов. >15 → декомпозиция.

---

## ██ RUNTIME-КОНТРАКТЫ И НАБЛЮДАЕМОСТЬ

- **`run_id`** наследуется из Шага 10 (одна UUID на весь прогон 00 → 40.3).
- **Grafana-метрики шага:** `meta_pages_count`, `avg_qc_score`, `retry_count`, `duration_ms`, `rag_unavailable`, `fallback_mode`, `cannibalization_blocks`, `cannibalization_warns`, `hitl_required`, `ctr_tier_high_pct`, `aeo_block_coverage_pct`.
- **Алерты:** QC fail 4x → Telegram; SLA > 8 мин → Telegram; каннибализация cosine > 0.92 на ≥ 3 страницах → Slack (sign гипер-каннибализации).
- **DLQ:** топик `seo_meta_dlq` (NocoDB) с `opus_raw_text` для ручного анализа.
- **Checkpoints:** `prompt_response.json` + `validation_report.json` в таблице `seo_step_checkpoints` с TTL 7 дней.
- **A/B-трекинг:** Шаг 40 читает `primary_variant` и `meta_variants[]`, запускает 14-дневный эксперимент по ротации в CMS.

---

## ██ ИСТОРИЯ ИЗМЕНЕНИЙ

| Версия | Дата | Изменения |
|---|---|---|
| v1.0 | 2026-04-16 | Первая редакция. Template 3 расширен до мульти-страничного выхода + `meta_variants[]` (A/B) + `aeo_elements` + `schema_markup_hint` + `cannibalization_check` + `ctr_signals` + `engine_tags` с [Я]/[G] split. **Антигаллюцинационный протокол** (наследован из Шага 10 v1.1): power_words и engine_tags только из `lsi_items_meta_relevant` ∪ `priority_terms_for_meta`. **Условный HitL** для YMYL и qc 60-79. **Redis Lock** ≤3 Opus-вызова на проект. **Checkpoint resume** по `run_id`. **20 QC-проверок** (Jaccard H1/title, power_words subset, каннибализация cosine > 0.92 блок, > 0.85 warn). **8 адаптаций** под niche_cluster. **Few-shot** на `dental_moscow_001` с 3 вариантами meta + AEO-блок + schema hint. **Интеграция** с Шагами 12, 13, 15, 18, 29. |

---

**Контрольный чек-лист соответствия CORE.md + STEP_00-эталону + STEP_10 v1.1:**

- [x] Основная роль F + консультативные A/E/G (из карты 63 шагов)
- [x] Тип «Промпт»: pre-scripts отсутствуют, только post-step валидация
- [x] Opus mode 0.15/0.9/8192 (Шаг 11 НЕ в списке «креативных» — только 12, 35)
- [x] `[Я]/[G]/[ОБА]` обязателен + согласованность с `priority_engines`
- [x] Русский язык в полях; английские ключи/enum
- [x] QC rule-based (20 проверок, ML отключён)
- [x] Retry Protocol v2 + защита от петли (унаследовано из Шага 10)
- [x] JSON-контракт = Template 3 + мульти-страничное расширение + A/B + AEO
- [x] Input contract = output Шага 10 v1.1 (с наследованием `run_id`, `priority_terms_for_meta`, `engine_split_from_step_10`, `lsi_items_meta_relevant`)
- [x] Output → Шаги 12, 13, 15, 18, 29 с явными полями для каждого потребителя
- [x] Few-shot реалистичен для русскоязычного рынка (детская стоматология, Москва)
- [x] Антипаттерны + антигаллюцинационный блок (наследован из Шага 10 v1.1)
- [x] Опережающие + запаздывающие + контрольные метрики
- [x] Условный HitL для YMYL (наследован из Шага 10 v1.1)
- [x] Наблюдаемость: run_id, checkpoints, DLQ, Grafana
- [x] Лимиты токенов чётко обозначены
- [x] Адаптация под 8 niche_cluster с различными структурами title
- [x] Антиканнибализация через Qdrant cosine (блок > 0.92, warn > 0.85)
- [x] Интеграция с A/B-трекингом Шага 40 через `primary_variant` + `meta_variants[]`
