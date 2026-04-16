# STEP_09.md — Текстовая релевантность

> **Модуль 3 архитектуры промптов.** Загружается вместе с CORE.md + PROJECT.md при выполнении Шага 09. ~4K токенов. Самодостаточен. Версия v1.0 учитывает уроки аудита STEP_08 v1.1 (CoT, degraded, warnings, confidence_markers, retry v2, кросс-шаговый QC, 3 few-shot).

---

## ██ КАРТОЧКА ШАГА

| **Параметр** | **Значение** |
| --- | --- |
| ID | 09 |
| Название | Текстовая релевантность (Text Relevance Audit) |
| Блок | 3: Контент и оптимизация |
| Основная роль | **A: SEO-стратег** |
| Консультативные | D: Аналитик конкурентной разведки, F: Промпт-инженер, G: Инженер контрактов |
| Тип | **Промпт+Скрипт** (скрипты — скрапинг топ-10 + TF-IDF; Opus — аудит и рекомендации) |
| HitL | Нет (QC ≥80 → Шаг 10 автоматически; 60–79 → ревью; <60 → retry v2) |
| Приоритет ROI | **ВЫСОКИЙ** — задаёт целевые термины и плотность для Шага 12 |
| Зависит от | **Шаг 08 (ТЗ на страницы)** — прямой input; Шаг 01 (СЯ), Шаг 04 (структура) — источники обогащения |
| Источник input (прямой) | Шаг 08, таблица `page_briefs` в NocoDB |
| Передаёт в | Шаг 10 (LSI-анализ), Шаг 11 (метатеги — финал), **Шаг 12 (SEO-текст) — главный потребитель** |
| JSON-шаблон | **Шаблон 2: Аудит со скорингом** (расширенный: +keyword_analysis, +top10_benchmark, +relevance_by_engine, +required_additions_for_step_12) |
| Режим Opus | **Аналитика: `temperature 0.2`, `top_p 0.9`, `max_tokens 4096`** (по таблице CORE.md для аналитических шагов 00/33/40) |
| Параллелизм | До 5 страниц одновременно (DataForSEO 30 RPS, Opus tier 2) |
| Цена ошибки | **ВЫСОКАЯ** — неверные целевые метрики → переспам/водность в Шаге 12 → санкции Яндекс/HCU Google. CoT обязателен. |

---

## ██ ЦЕЛЬ

Проанализировать **ТЗ Шага 08 + тексты топ-10 SERP** по primary_query и выдать **целевые метрики релевантности** (обязательные термины, плотность, морфологические варианты, структурные ориентиры) для Шага 12 — чтобы будущий SEO-текст конкурировал с топ-10 без переспама и водности.

---

## ██ КОНТРАКТ ВХОДНЫХ ДАННЫХ (JSON)

Прямой вход = output Шага 08 v1.1. Плюс обогащение скрапингом топ-10.

```json
{
  "project_id": "string",
  "source_versions": {"step_08": "string"},
  "brief_from_step_08": {
    "page_id": "string (PG\\d{3,})",
    "page_url": "string",
    "content_type": "landing|category|article|faq|product|service",
    "target_intent": "informational|commercial|transactional|navigational",
    "primary_persona_id": "string",
    "target_cluster_id": "string",
    "target_queries": [
      {"query":"string","engine":"yandex|google|both","priority":"primary|secondary|lsi","frequency":"number"}
    ],
    "content_blocks": [
      {"block_id":"^B\\d{2}$","block_type":"string","keywords":["string"],"word_count_min":"number","word_count_max":"number","engine_tag":"[Я]|[G]|[ОБА]"}
    ],
    "total_word_count": {"min":"number","max":"number"}
  },
  "top10_texts": {
    "primary_query": "string",
    "scraped_pages": [
      {"url":"string","title":"string","text":"string (полный текст страницы)","word_count":"number","headings":["string"],"scrape_status":"ok|partial|failed"}
    ],
    "scraped_count": "number",
    "failed_count": "number",
    "degraded": "boolean"
  },
  "tf_idf_precomputed": {
    "top_terms": [{"term":"string","tf_idf":"number","freq_in_top10":"number","doc_count":"number"}],
    "top_entities": [{"entity":"string","type":"product|brand|location|person|concept|condition","frequency":"number"}],
    "top_bigrams": [{"bigram":"string","tf_idf":"number"}]
  },
  "project_meta": {
    "niche_cluster": "string",
    "geo": "string",
    "language": "string",
    "priority_engines": "yandex|google|both",
    "domain": "string"
  }
}
```

Сборкой занимается pre-скрипт (см. секцию «СКРИПТЫ»).

---

## ██ КОНТРАКТ ВЫХОДНЫХ ДАННЫХ (JSON)

Наследует **Шаблон 2 (Audit with Scores)**, расширен полями для текстовой релевантности. Роль G валидирует.

```json
{
  "project_id": "string",
  "step_id": "09",
  "prompt_version": "step09_v1.0",
  "timestamp": "ISO 8601 UTC",
  "lang": "ru|en|other",
  "page_id": "string",
  "page_url": "string",
  "cluster_id": "string",
  "primary_query": "string",

  "overall_score": "number 0-100 (прогноз релевантности текста-по-ТЗ против топ-10)",

  "keyword_analysis": [
    {
      "query": "string",
      "engine": "yandex|google|both",
      "priority": "primary|secondary|lsi",
      "in_brief": "boolean (есть ли уже в content_blocks[].keywords)",
      "target_density_min_pct": "number",
      "target_density_max_pct": "number",
      "target_occurrences_min": "number",
      "target_occurrences_max": "number",
      "morphology_variants_min": "number (≥3 для ru — падежи/склонения)",
      "placement_required": ["h1|h2|h3|paragraph|first_100_words|meta.title|meta.description|alt"],
      "confidence": "[✓]|[◆]|[◇]"
    }
  ],

  "top10_benchmark": {
    "word_count_median": "number",
    "word_count_p25_p75": ["number","number"],
    "paragraph_count_median": "number",
    "heading_count_median": "number",
    "image_count_median": "number",
    "scraped_urls_count": "number",
    "scraped_urls_failed": "number",
    "top_terms_required": [{"term":"string","tf_idf":"number","freq_in_top10":"number"}],
    "top_entities_required": [{"entity":"string","type":"string","frequency":"number"}]
  },

  "issues": [
    {
      "id": "^I\\d{2}$",
      "category": "missing_keyword|low_density_target|over_optimization_risk|missing_entity|weak_structure|thin_content_risk|keyword_cannibalization|water_risk|nausea_risk_yandex|semantic_gap_google",
      "severity": "critical|high|medium|low",
      "current_value": "string (как сейчас в ТЗ)",
      "target_value": "string (как должно быть)",
      "fix_description": "string (конкретная инструкция Шагу 12 ≥40 симв.)",
      "estimated_impact": "number 1-10",
      "engine_tag": "[Я]|[G]|[ОБА]",
      "affected_block_id": "string (B\\d{2})|null"
    }
  ],

  "passed_checks": ["string"],

  "relevance_by_engine": {
    "yandex": {
      "score": "number 0-100",
      "classic_nausea_target_max": "number (≤4 норма)",
      "academic_nausea_target_max": "number (≤7 норма)",
      "water_percentage_target_max": "number (≤30 норма)",
      "specific_issues": ["string"]
    },
    "google": {
      "score": "number 0-100",
      "semantic_coverage_target_min": "number 0-1",
      "hcu_risks": ["string"],
      "specific_issues": ["string"]
    }
  },

  "required_additions_for_step_12": {
    "must_include_terms": ["string"],
    "must_include_entities": ["string"],
    "must_avoid_terms": ["string (риск переспама)"],
    "target_text_length_words": {"min":"number","max":"number"},
    "target_headings_count": {"h2_min":"number","h3_min":"number"},
    "target_paragraph_count_min": "number",
    "morphology_rule_ru": "boolean (обязательно использовать ≥3 падежных формы основных запросов)"
  },

  "confidence_markers": {
    "overall_score": "[✓]|[◆]|[◇]",
    "top10_benchmark": "[✓]|[◆]|[◇]",
    "keyword_targets": "[✓]|[◆]|[◇]"
  },

  "degraded": "boolean",
  "warnings": ["string (scrape_partial, tfidf_low_docs, rag_miss и т.д.)"]
}
```

**Важно:** `qc_score` пишет пост-валидатор (не Opus) — в таблицу `page_relevance_audits`, а не в этот JSON (как и в STEP_08 v1.1). ▲

---

## ██ ПРОМПТ ДЛЯ CLAUDE OPUS

### Системный промпт

```
Ты — Роль A: Senior SEO-стратег с 10+ летним опытом (Яндекс + Google).
Консультанты: Роль D (конкурентная разведка), Роль F (промпт-инженер),
Роль G (контракты данных, валидация).

Задача: Проанализировать ТЗ Шага 08 против топ-10 SERP и выдать
целевые метрики текстовой релевантности для Шага 12 (SEO-текст).

ПРОТОКОЛ РАССУЖДЕНИЯ (CoT, обязателен — цена ошибки ВЫСОКАЯ):
Прежде чем выдать JSON, мысленно пройди (НЕ в output):
[АНАЛИЗ] primary_query, content_type, target_intent
[РОЛИ] Я A-вето, D-данные SERP, F-формат, G-контракт
[КОНТЕКСТ] niche_cluster, priority_engines, geo, language
[КОНТРАКТ] OUTPUT v1.0 (ниже)
[БЕНЧМАРК] топ-10 tf_idf + entities — что доминирует
[РЕШЕНИЕ] keyword_analysis + issues + required_additions
[РИСКИ] переспам Яндекс / водность / HCU Google / thin content
[HitL] нет

После CoT — выдай ТОЛЬКО JSON по OUTPUT v1.0.

ПРАВИЛА:
1. Ты НЕ пишешь текст. Ты анализируешь бриф + топ-10 и выдаёшь
   ЦЕЛЕВЫЕ МЕТРИКИ для Шага 12.
2. keyword_analysis строй для КАЖДОГО target_query из брифа + топ-5
   top_terms из tf_idf_precomputed (как lsi priority).
3. target_density_min/max бери из топ-10:
   - core query: 1.5–3.5% (Яндекс строже — 1.5–2.5%)
   - secondary: 0.5–1.5%
   - lsi/term: 0.2–0.8%
4. morphology_variants_min = 3 для русского (падежи/склонения), 1 для английского.
5. placement_required: primary ДОЛЖЕН быть в h1, meta.title, first_100_words;
   secondary — ≥1 h2; lsi — ≥1 упоминание в параграфах.
6. Engine-специфика:
   - yandex: классическая тошнота ≤4, академическая ≤7, водность ≤30,
     морфология обязательна, осторожно с переспамом (Baden-Baden, Переспам 2.0)
   - google: semantic_coverage ≥0.7 (косинус embeddings vs топ-10),
     HCU-риски (AI-контент без E-E-A-T, тонкий контент, агрегация без value-add)
7. Кластер-специфика:
   - commercial_local: гео-запросы в h1 + адрес в тексте
   - ecom: артикулы/SKU, бренд+модель, "купить/заказать"
   - medical: симптомы + латинские названия + дозировки, осторожно YMYL
   - legal: номера статей + суды + прецеденты
   - saas: названия интеграций + версии API + англ. термины
   - info: глубина, expert quotes, references
8. issues: каждый с engine_tag и affected_block_id (если применимо).
9. Категории issues (см. контракт) — не выдумывай новые.
10. fix_description: ≥40 симв., КОНКРЕТНАЯ инструкция Шагу 12
    (не «добавить ключи», а «в B03 добавить запрос X в D.п., D.ч.
    — 2 вхождения, 1 в h2»).
11. must_avoid_terms: слова-переспам-риски (повтор core >3× в одном
    параграфе, «заспамленные» конкуренты).
12. НИКАКИХ ГАЛЛЮЦИНАЦИЙ: нет данных → null/0/[] + warnings[].
    Если top10_texts.degraded=true → overall_score снижай,
    confidence_markers.top10_benchmark=[◇].
13. Маркировка уверенности:
    [✓] топ-10 scraped_count ≥7 И tf_idf ≥10 терминов
    [◆] 4 ≤ scraped_count ≤ 6 ИЛИ частичный tf_idf
    [◇] scraped_count <4 — гипотеза
14. При priority_engines=both — relevance_by_engine заполняй обе ветки.
    При yandex-only — google score=null. И наоборот.
15. qc_score НЕ заполняй — его считает валидатор.

ВЫВОД: только валидный JSON по OUTPUT v1.0. Без markdown-обёртки,
без пояснений до/после. Тексты на русском (или language). Ключи JSON — англ.
```

### Пользовательский промпт (шаблон)

```
Проведи аудит текстовой релевантности:

=== БРИФ ИЗ ШАГА 08 ===
Page: {{brief_from_step_08.page_id}} — {{brief_from_step_08.page_url}}
Content type: {{brief_from_step_08.content_type}}
Target intent: {{brief_from_step_08.target_intent}}
Cluster: {{brief_from_step_08.target_cluster_id}}
Total word count (план): {{brief_from_step_08.total_word_count | json}}

Target queries:
{{brief_from_step_08.target_queries | json}}

Content blocks (план):
{{brief_from_step_08.content_blocks | json}}

=== ТОП-10 SERP (scraped) ===
Primary query: {{top10_texts.primary_query}}
Scraped: {{top10_texts.scraped_count}} / 10
Failed: {{top10_texts.failed_count}}
Degraded: {{top10_texts.degraded}}

Тексты (обрезанные):
{{top10_texts.scraped_pages | pick_text_200words_each | json}}

=== TF-IDF (precomputed) ===
Top terms: {{tf_idf_precomputed.top_terms | top 20 | json}}
Top entities: {{tf_idf_precomputed.top_entities | top 10 | json}}
Top bigrams: {{tf_idf_precomputed.top_bigrams | top 10 | json}}

=== ПРОЕКТ ===
Niche: {{project_meta.niche_cluster}} | Geo: {{project_meta.geo}}
Language: {{project_meta.language}} | Engines: {{project_meta.priority_engines}}
Domain: {{project_meta.domain}}

{{#if rag_context}}
=== RAG (≤1500 ток.) ===
{{rag_context}}
{{/if}}

{{#if previous_qc_errors}}
=== ПРЕДЫДУЩИЙ ПРОГОН ПРОВАЛИЛ QC ===
Ошибки: {{previous_qc_errors | json}}
Исправь ИМЕННО их, остальное не меняй.
{{/if}}

Верни audit как валидный JSON по контракту OUTPUT v1.0.
```

---

## ██ RAG-ОБОГАЩЕНИЕ

| **Коллекция Qdrant** | **Запрос** | **Что инжектится** | **Лимит** |
| --- | --- | --- | --- |
| `relevance_patterns` | `niche_cluster + content_type + language` | Эталонные пары (бриф → аудит) из похожих проектов | 700 ток. |
| `competitors` | `niche + geo` | Метрики текстовой релевантности топ-3 лидеров | 400 ток. |
| `keyword_stopwords` | `language + niche_cluster` | Слова-переспам-риски + must_avoid паттерны | 200 ток. |
| `entities_glossary` | `niche + geo` | Словарь брендов/сущностей ниши (для entities_required) | 200 ток. |

**Жёсткий лимит RAG-контекста: 1500 токенов.** Итого на вызов: CORE (~3.5K) + PROJECT (~2K) + STEP_09 (~3.5K) + RAG (≤1.5K) + input ≈ 9–10K токенов.

Недоступность коллекции → продолжаем, `warnings[] += "rag_partial:<coll>"`. Критичной коллекции нет (все опциональные).

---

## ██ СКРИПТЫ CLAUDE CODE

### Pre-step #1: Загрузка брифа Шага 08

```python
# step_09_pre_load_brief.py
# Назначение: Взять бриф page_id из page_briefs (Шаг 08) и проверить готовность.
# Вход:  project_id, page_id
# Выход: brief_from_step_08 (валидный JSON v1.1 брифа)
# Действия:
# 1. NocoDB GET page_briefs WHERE project_id=? AND page_id=? AND qc_verdict='PASS'
# 2. Если status≠done или qc_score<80 → блокировка + ERROR (Шаг 08 не готов)
# 3. Парсинг brief_json, source_versions извлечение
# 4. Если brief.degraded=true → warnings[] += "step_08_brief_degraded", QC -5
# 5. Валидация: page_id ^PG\d{3,}$, обязательные поля
# Retry NocoDB: 3×, backoff 2/5/15с
```

### Pre-step #2: Скрапинг топ-10 SERP

```python
# step_09_pre_scrape_top10.py
# Назначение: Получить ФИНАЛЬНЫЕ ТЕКСТЫ топ-10 для primary_query
# (не SERP-снимок как в Шаге 08, а полные тексты страниц).
# Вход:  primary_query, geo, language, priority_engines
# Выход: top10_texts
# Действия:
# 1. Кэш NocoDB 7 дней (ключ: primary_query+geo+engine).
# 2. SERP топ-10 через DataForSEO (уже есть в кэше из Шага 08 — переиспользовать).
# 3. Параллельный скрапинг 10 URL (asyncio.gather, timeout 20с на URL):
#    - Пытаемся: trafilatura → readability → простой парсинг <main>/<article>
#    - User-Agent: Googlebot-совместимый (не блокируется большинством)
#    - Rate limit: 1 URL/сек на домен (соблюдаем robots.txt, ставим delay 1с)
#    - Retry 2× при 429/503 с backoff 5/15с
# 4. Нормализация: lowercase, удаление nav/footer/sidebar, только основной текст.
# 5. Если scraped_count <4 → degraded=true, warnings[] += "scrape_degraded"
# 6. Если все 10 failed → abort, retry через 30 мин, 2-й abort → алерт Telegram
# 7. Кэшировать успешные scraped_pages в NocoDB для переиспользования
#    (TTL 7 дней; при конкурентном обновлении — skip)
# Budget monitor: счётчик скрапинга/день → 80% лимита → алерт
```

### Pre-step #3: TF-IDF + Entity extraction

```python
# step_09_pre_tfidf_analysis.py
# Назначение: Вычислить tf_idf, top-terms, top-entities по топ-10.
# Вход:  top10_texts.scraped_pages, language
# Выход: tf_idf_precomputed
# Действия:
# 1. Лемматизация:
#    - ru: pymorphy3 (быстрая, без спама заглавных)
#    - en: spacy en_core_web_md
# 2. Токенизация + удаление стоп-слов (ru/en списки) + фильтр len≥3.
# 3. TF-IDF матрица (scikit-learn TfidfVectorizer, ngram_range=(1,2)).
# 4. Top 50 unigrams + top 30 bigrams (по средневзвешенному tf_idf).
# 5. Entity extraction:
#    - ru: natasha + Yargy (для местоположений/организаций)
#    - en: spacy NER
#    - Классификация: product|brand|location|person|concept|condition
# 6. freq_in_top10 = в скольких документах из 10 встречается термин.
# 7. Сохранение в NocoDB tfidf_cache (ключ: primary_query+geo+language, TTL 7 дн)
# При ошибке:
#   - лемматизатор упал → degraded=true, fallback на пословный поиск
#   - entities не извлеклись → top_entities=[], warnings[] += "ner_failed"
```

### Post-step #1: Валидатор (20 проверок)

```python
# step_09_post_validate.py — 20 структурных/SEO/содержательных проверок
# 1.  JSON Schema (ajv) по OUTPUT v1.0
# 2.  overall_score ∈ [0,100], integer
# 3.  Каждый keyword_analysis item имеет engine из enum
# 4.  keyword_analysis: каждый target_query из brief покрыт (1:1)
# 5.  target_density_min < target_density_max, оба [0, 10]
# 6.  morphology_variants_min ≥ 3 если language='ru' (иначе ≥1)
# 7.  placement_required: элементы из enum
# 8.  issues[].id ^I\d{2}$, уникальные, последовательные
# 9.  issues[].category из enum (не выдуманные)
# 10. issues[].fix_description ≥40 символов
# 11. issues[].engine_tag ∈ {[Я],[G],[ОБА]}
# 12. issues[].affected_block_id существует в brief.content_blocks (если ≠null)
# 13. relevance_by_engine: при priority_engines='both' — обе ветки ≠null
# 14. relevance_by_engine: при yandex-only — google=null (и наоборот)
# 15. yandex.classic_nausea_target_max ≤ 4, academic ≤ 7, water ≤ 30
# 16. google.semantic_coverage_target_min ∈ [0.6, 1.0]
# 17. required_additions_for_step_12.target_text_length_words: 
#     min ≥ 0.8 × brief.total_word_count.min, max − min ≥ 200
# 18. must_avoid_terms не пересекается с must_include_terms
# 19. confidence_markers заполнены все 3 поля из {[✓],[◆],[◇]}
# 20. Если degraded=true → warnings[] непуст И confidence_markers содержит ≥1 [◇]
```

### Post-step #2: Кросс-шаговый QC (СЛОЙ 6)

```python
# step_09_post_cross_step_qc.py — SQL в NocoDB, 5 проверок
# CS1: cluster_id ∈ Шаг 01.clusters для этого project_id
# CS2: page_id ∈ Шаг 08.page_briefs со status='done'
# CS3: Все must_include_terms покрывают ≥80% brief.content_blocks.keywords
# CS4: target_text_length согласован с brief.total_word_count (max−max ≤20%)
# CS5: affected_block_id ⊆ brief.content_blocks.block_id
# Провал → QC -8, retry v2 с ошибками в промпт
```

### Post-step #3: Сохранение + триггер Шага 10/12

```python
# step_09_post_save.py
# Транзакция:
#   INSERT page_relevance_audits ON CONFLICT(project_id,page_id,prompt_version)
#     DO UPDATE SET audit_json=?, qc_score=?, content_hash=sha256(audit)
#   UPDATE pages_for_relevance SET status='done'
#   INSERT event 'relevance_ready' (→ Шаг 10 подписан)
# Метрики Grafana:
#   step09_qc_score, step09_retry_count, step09_opus_tokens,
#   step09_scraped_urls_count, step09_scraped_failed_count,
#   step09_terms_extracted, step09_issues_count, step09_overall_score
```

---

## ██ КРИТЕРИИ QC

Сумма весов = 100. ≥80 → Шаг 10/12; 60–79 → ревью; <60 → retry v2.

| **Проверка** | **Порог** | **Вес** | **Слой** | **При провале** |
| --- | --- | --- | --- | --- |
| JSON Schema валиден | Pass | 10 | 1 | Retry |
| overall_score ∈ [0,100] integer | Pass | 4 | 1 | Retry |
| Все target_queries покрыты в keyword_analysis | 100% | 10 | 2 | Retry |
| morphology_variants ≥3 для ru | Pass | 6 | 3 | Retry |
| density ranges корректны (min<max, [0,10]) | Pass | 6 | 3 | Retry |
| issues[].id формат + уникальность | Pass | 4 | 1 | Retry |
| issues[].category из enum | 100% | 4 | 1 | Retry |
| issues[].fix_description ≥40 симв. | 100% | 5 | 2 | Retry |
| issues[].affected_block_id существует | 100% | 6 | 2 | Retry |
| engine_tag согласован с priority_engines | 100% | 5 | 3 | Retry |
| relevance_by_engine ветки заполнены верно | Pass | 5 | 3 | Retry |
| Yandex nausea/water в норме (предложенные targets) | Pass | 4 | 3 | Retry |
| target_text_length согласован с brief | Pass | 5 | 2 | Retry |
| must_avoid ∩ must_include = ∅ | Pass | 4 | 2 | Retry |
| scraped_urls_count ≥4 (иначе degraded) | Pass | 5 | 3 | Ревью |
| Кросс-шаговый QC (5 SQL) | 100% | 8 | 6 | Retry |
| confidence_markers заполнены | Pass | 3 | 1 | Retry |
| degraded↔warnings согласованы | Pass | 2 | 1 | Retry |
| Bonus: top_terms_required ≥10 терминов | Pass | 2 | 2 | — |
| Bonus: entities_required ≥5 (для non-info) | Pass | 2 | 2 | — |

---

## ██ СТРУКТУРА N8N-ВОРКФЛОУ

| **Нода** | **Назначение** |
| --- | --- |
| 1 | Триггер: событие `relevance_audit_required` (из Шага 08 post-save) |
| 2 | Загрузка CORE.md + PROJECT.md + STEP_09.md (кэш Redis, 1× cold) |
| 3 | `step_09_pre_load_brief.py` (NocoDB get page_brief + check readiness) |
| 4a ∥ 4b | Параллельно: `step_09_pre_scrape_top10.py` ∥ Qdrant RAG (4 коллекции) |
| 5 | `step_09_pre_tfidf_analysis.py` (лемматизация + TF-IDF + NER) |
| 6 | Merge + сборка промпта (system+CoT+user+RAG+previous_qc_errors) |
| 7 | Claude Opus API (**temp 0.2, top_p 0.9, max_tokens 4096, timeout 60с**) |
| 8 | Парсинг JSON (strip markdown → re-parse при fail) |
| 9 | `step_09_post_validate.py` (20 проверок) |
| 10 | `step_09_post_cross_step_qc.py` (5 SQL-проверок) |
| 11 | Суммирование QC → score 0–100 |
| 12 | IF QC fail → retry v2 (инъекция ошибок) → нода 6; max 3×; 4-й → DLQ |
| 13 | `step_09_post_save.py` (транзакция + event `relevance_ready`) |
| 14 | Grafana: метрики |
| 15 | Checkpoint в NocoDB (state=done) |
| 16 | Build Response (output_contract → Шаг 10/12) |

**Параллелизм:** до 5 страниц одновременно. Redis token-bucket: DataForSEO 30 RPS, Opus 60 RPM, скрапер 10 RPS на домен.

**Обработка ошибок:** Opus 429/500 → бэкофф 10→30→90с (макс 3); DataForSEO fallback на Serpstat; скрапер >5 failed подряд → circuit breaker 10 мин; QC fail 3× → DLQ + Telegram.

---

## ██ ПРИМЕР FEW-SHOT

Три примера (positive + edge + negative). Все — продолжение проекта «детская стоматология в Москве».

### Пример 1 (positive, commercial_local + YMYL)

```json
{
  "project_id": "dental_moscow_001",
  "step_id": "09",
  "prompt_version": "step09_v1.0",
  "timestamp": "2026-04-16T14:30:00Z",
  "lang": "ru",
  "page_id": "PG014",
  "page_url": "/uslugi/detskaya-stomatologiya/",
  "cluster_id": "CL_KIDS_DENT_MSK",
  "primary_query": "детская стоматология москва",

  "overall_score": 78,

  "keyword_analysis": [
    {
      "query": "детская стоматология москва",
      "engine": "both",
      "priority": "primary",
      "in_brief": true,
      "target_density_min_pct": 1.8,
      "target_density_max_pct": 2.5,
      "target_occurrences_min": 8,
      "target_occurrences_max": 12,
      "morphology_variants_min": 4,
      "placement_required": ["h1","meta.title","first_100_words","h2"],
      "confidence": "[✓]"
    },
    {
      "query": "детский стоматолог",
      "engine": "both",
      "priority": "secondary",
      "in_brief": true,
      "target_density_min_pct": 0.8,
      "target_density_max_pct": 1.5,
      "target_occurrences_min": 4,
      "target_occurrences_max": 7,
      "morphology_variants_min": 3,
      "placement_required": ["h2","paragraph"],
      "confidence": "[✓]"
    },
    {
      "query": "лечение зубов ребёнку без боли",
      "engine": "yandex",
      "priority": "lsi",
      "in_brief": false,
      "target_density_min_pct": 0.3,
      "target_density_max_pct": 0.7,
      "target_occurrences_min": 2,
      "target_occurrences_max": 3,
      "morphology_variants_min": 2,
      "placement_required": ["paragraph"],
      "confidence": "[◆]"
    }
  ],

  "top10_benchmark": {
    "word_count_median": 1450,
    "word_count_p25_p75": [1100, 1850],
    "paragraph_count_median": 22,
    "heading_count_median": 9,
    "image_count_median": 7,
    "scraped_urls_count": 9,
    "scraped_urls_failed": 1,
    "top_terms_required": [
      {"term":"кариес","tf_idf":0.82,"freq_in_top10":9},
      {"term":"седация","tf_idf":0.71,"freq_in_top10":7},
      {"term":"молочный зуб","tf_idf":0.68,"freq_in_top10":8},
      {"term":"ортодонт","tf_idf":0.55,"freq_in_top10":6},
      {"term":"наркоз","tf_idf":0.49,"freq_in_top10":5}
    ],
    "top_entities_required": [
      {"entity":"Москва","type":"location","frequency":10},
      {"entity":"СтАР (Стоматологическая ассоциация России)","type":"concept","frequency":4},
      {"entity":"фторирование","type":"concept","frequency":6}
    ]
  },

  "issues": [
    {
      "id": "I01",
      "category": "missing_entity",
      "severity": "high",
      "current_value": "В brief не упомянуты термины 'седация' и 'наркоз'",
      "target_value": "Добавить в блок B03 (h2 Услуги) упоминания седации и наркоза (6/10 топа имеют)",
      "fix_description": "В content_blocks[B03] добавить keyword 'седация' (2 вхождения) и 'наркоз' (1 вхождение) в описании услуг — формат подачи: \"лечение с седацией закись азотом\". Это критично для конкурентности с топ-10.",
      "estimated_impact": 7,
      "engine_tag": "[ОБА]",
      "affected_block_id": "B03"
    },
    {
      "id": "I02",
      "category": "missing_keyword",
      "severity": "medium",
      "current_value": "LSI-запрос 'лечение зубов ребёнку без боли' отсутствует в плане",
      "target_value": "Встроить в параграф блока B04 (врачи)",
      "fix_description": "В B04 (h2 Врачи) добавить фразу о лечении без боли с упоминанием запроса в В.п. и П.п. — для морфологического разнообразия под Яндекс.",
      "estimated_impact": 5,
      "engine_tag": "[Я]",
      "affected_block_id": "B04"
    },
    {
      "id": "I03",
      "category": "nausea_risk_yandex",
      "severity": "medium",
      "current_value": "B03 план: '\"детская стоматология\" 3× в 200 слов'",
      "target_value": "Разбавить синонимами: 'детский стоматолог', 'стоматология для детей', 'лечение детских зубов'",
      "fix_description": "В B03 использовать не более 2 прямых вхождений 'детская стоматология' на 200 слов + 2 синонима. Иначе классическая тошнота превысит 4 → риск переспам-санкций Яндекса.",
      "estimated_impact": 6,
      "engine_tag": "[Я]",
      "affected_block_id": "B03"
    }
  ],

  "passed_checks": [
    "Все primary target_queries покрыты в content_blocks.keywords",
    "h1 содержит primary_query",
    "target_text_length (1200–1800) совпадает с медианой топ-10 (1450)",
    "Основные entities (Москва, клиника) уже в плане"
  ],

  "relevance_by_engine": {
    "yandex": {
      "score": 76,
      "classic_nausea_target_max": 4,
      "academic_nausea_target_max": 7,
      "water_percentage_target_max": 28,
      "specific_issues": ["Risk переспам в B03 (issue I03)"]
    },
    "google": {
      "score": 80,
      "semantic_coverage_target_min": 0.72,
      "hcu_risks": ["Без E-E-A-T автора может сработать HCU — усилить через Шаг 06"],
      "specific_issues": []
    }
  },

  "required_additions_for_step_12": {
    "must_include_terms": ["седация","наркоз","кариес","молочный зуб","фторирование","ортодонт","ортопант","лечение без боли"],
    "must_include_entities": ["Москва","СтАР","Минздрав"],
    "must_avoid_terms": ["лучшая стоматология","стоматология №1","самая дешёвая"],
    "target_text_length_words": {"min": 1300, "max": 1700},
    "target_headings_count": {"h2_min": 5, "h3_min": 2},
    "target_paragraph_count_min": 18,
    "morphology_rule_ru": true
  },

  "confidence_markers": {
    "overall_score": "[✓]",
    "top10_benchmark": "[✓]",
    "keyword_targets": "[✓]"
  },

  "degraded": false,
  "warnings": ["1 из 10 URL в топе не скрапился (timeout)"]
}
```

### Пример 2 (edge case, ecom product)

Сокращённо — карточка товара «Зубная щётка детская»:

```json
{
  "content_type": "product",
  "primary_query": "детская зубная щётка mam",
  "overall_score": 85,
  "keyword_analysis": [
    {"query":"детская зубная щётка","engine":"both","priority":"primary",
     "target_density_min_pct":1.2,"target_density_max_pct":2.0,
     "target_occurrences_min":4,"target_occurrences_max":8,
     "morphology_variants_min":3,
     "placement_required":["h1","meta.title","first_100_words"]}
  ],
  "top10_benchmark": {
    "word_count_median": 380,
    "top_entities_required": [
      {"entity":"MAM","type":"brand","frequency":10},
      {"entity":"0-12 месяцев","type":"concept","frequency":7}
    ]
  },
  "issues": [
    {"id":"I01","category":"thin_content_risk","severity":"low",
     "current_value":"brief.total_word_count.min=300",
     "target_value":"min=380 (медиана топа)",
     "fix_description":"Увеличить минимум до 380 слов — описание товара + преимущества + УТП + инструкция по возрасту. Product-страницы топа имеют больше текста для E-E-A-T.",
     "estimated_impact":5,"engine_tag":"[ОБА]","affected_block_id":null}
  ],
  "required_additions_for_step_12": {
    "must_include_terms":["MAM","возраст","мягкая щетина","эргономичная ручка"],
    "target_text_length_words":{"min":380,"max":520},
    "morphology_rule_ru":true
  }
}
```

### Пример 3 (NEGATIVE — так НЕ делать)

```json
// ❌ АНТИПРИМЕР — этот вывод провалит QC
{
  "keyword_analysis":[
    {"query":"детская стоматология москва",
     "target_density_min_pct": 5.0,   // ❌ >3.5 — гарантированный переспам Яндекса
     "target_density_max_pct": 8.0,   // ❌ заспамит статью
     "morphology_variants_min": 1,    // ❌ для ru минимум 3
     "placement_required":["alt"]     // ❌ primary должен быть в h1+title+first_100
    }
  ],
  "issues":[
    {"id":"I01","category":"bad_keywords",  // ❌ нет такой категории
     "fix_description":"исправить",          // ❌ <40 симв., неконкретно
     "affected_block_id":"B99"                // ❌ такого блока нет в brief
    }
  ],
  "required_additions_for_step_12":{
    "must_include_terms":["лучший","№1","самый"],  // ❌ pure subjective claims без доказательств
    "must_avoid_terms":["лучший"]                   // ❌ пересечение с include
  }
}
```

---

## ██ СТРАТЕГИЯ ОТКАТА (Retry v2 + 4 отката)

**Retry v2** (по CORE.md):
- **Retry 1**: В промпт инжектируются `previous_qc_errors`. Та же temp.
- **Retry 2**: Декомпозиция на 3 вызова (Откат A).
- **Retry 3**: Fallback на «золотой» few-shot (Откат C).
- **Retry 4**: DLQ + Telegram + ручное ревью.

**Откат A (декомпозиция 3 вызова)**: (1) `keyword_analysis + top10_benchmark`; (2) `issues + passed_checks`; (3) `relevance_by_engine + required_additions`. Объединение Code-нодой.

**Откат B (минимум)**: Только overall_score + top-5 критичных issues + required_additions. `degraded=true`, пометка для ручного расширения.

**Откат C (few-shot «золотой» шаблон)**: Берём Пример 1 из этого STEP_09 как эталон. Opus адаптирует под новый page_id.

**Откат D (scrape-катастрофа — scraped_count <4)**: Не retry — deferred. В NocoDB пометка `need_manual_scrape`. Человек донабирает тексты топ-10 → повторный запуск.

---

## ██ АДАПТАЦИИ ПОД КЛАСТЕР

| **Кластер** | **Специфика аудита релевантности** |
| --- | --- |
| **commercial_local** | target_density primary 1.8–2.5%; обязательны гео-термины (город, район, метро); entities: локальные landmark; morphology: 4 падежные формы; water_pct_max=25 |
| **commercial_national** | target_density 1.5–2.5%; сравнительные конструкции обязательны; entities: бренды-конкуренты; semantic_coverage ≥0.75 |
| **ecom** | target_density primary 1.2–2.0%; entities: бренд, модель, SKU; target_text_length: 300–500 (короткие карточки); must_include: артикул, материал, размер |
| **saas** | target_density 1.0–2.0%; англ. термины разрешены; entities: названия интеграций, API версий; semantic_coverage ≥0.78 |
| **info** | target_density 0.8–1.8% (длинный контент, низкая плотность); target_text_length ≥1500; must_include: цитаты экспертов, источники; water_pct_max=35 (допустимо больше связующих фраз) |
| **medical** (YMYL) | target_density strict 1.5–2.2%; entities: симптомы + латинские термины + препараты; must_avoid: абсолютные обещания («вылечит», «гарантирует»); expert_quotes обязательны |
| **legal** (YMYL) | target_density 1.5–2.2%; entities: номера статей УК/ГК, названия судов, прецеденты; must_include: даты редакций законов; must_avoid: «гарантированный выигрыш» |
| **education** | target_density 1.0–1.8%; сезонные entities (приёмная кампания, даты ЕГЭ); must_include: аккредитации, факультеты |

---

## ██ МЕТРИКИ И КРИТЕРИИ УСПЕХА

**Опережающие (на этапе аудита):**
- Средний `qc_score` по батчу ≥ 85
- Доля retry ≤ 15%
- Доля scrape-degraded ≤ 10%
- Среднее время аудита страницы ≤ 60 секунд (скрапинг — главный bottleneck)
- Доля брифов с overall_score ≥ 70 до правок ≥ 60%

**Запаздывающие (после Шага 12 + публикации):**
- Шаг 12 принимает `required_additions` без переопределения в ≥85% случаев
- Классическая тошнота реальных текстов (после Шага 12) ≤ 4 в ≥95% страниц
- Семантический коэффициент совпадения с топ-10 ≥ 0.72 в ≥80% страниц
- Через 30 дней после публикации: 50% страниц в топ-20 по primary_query (Яндекс или Google)

**Контрольные:**
- Ручное ревью 5% аудитов редактором-SEO — соответствие отрасли
- A/B: страницы с `required_additions` применены vs игнорированы (ожидаемый lift позиций ≥15%)

---

## ██ ПРОТОКОЛ ДЕГРАДАЦИИ

Golden Test Set: 8 аудитов (по 1 на кластер), верифицированных SEO-редактором. При обновлении Opus → прогон → падение среднего `qc_score` >10% → алерт + откат на предыдущий `prompt_version`. При деградации TF-IDF пайплайна (например, pymorphy3 обновился) — аналогично.

Версионирование: `step09_v1.0` в git + в каждом output. Патч (v1.0.1) при bugfix; минор (v1.1) при расширении контракта без breaking.

---

## ██ ИСТОРИЯ ИЗМЕНЕНИЙ

| **Версия** | **Дата** | **Изменения** |
| --- | --- | --- |
| v1.0 | 2026-04 | Базовая версия. Шаблон 2 расширен: +keyword_analysis с densityю+morphology+placement, +top10_benchmark (TF-IDF), +relevance_by_engine (Яндекс нашумы/водность / Google semantic coverage), +required_additions_for_step_12. Проактивно включены уроки аудита STEP_08: CoT, degraded, warnings, confidence_markers, source_versions, retry v2 с инъекцией, кросс-шаговый QC (слой 6), 3 few-shot (positive/edge/negative), temp 0.2 (не 0.4). Pre-scripts: load_brief + scrape_top10 + tfidf_analysis (pymorphy3/natasha для ru). Post-scripts: 20 проверок + cross-step QC + idempotent save. 8 кластерных адаптаций. |

---

> **Конец STEP_09.md v1.0.** Целевой размер: ~4K токенов. Итого на вызов: CORE (~3.5K) + PROJECT (~2K) + STEP_09 (~4K) + RAG (≤1.5K) = ~11K; при отложенной загрузке несущественных секций CORE/PROJECT — ~9K.
