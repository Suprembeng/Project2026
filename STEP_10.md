# STEP_10.md — LSI-анализ

> **Самодостаточный модуль.** Загружается: `CORE.md` + `PROJECT.md` + этот файл. ~4K токенов.
> **Версия:** v1.0 · **Последнее обновление:** 2026-04

---

## ██ КАРТОЧКА ШАГА

| Параметр | Значение |
|---|---|
| **ID** | 10 |
| **Название** | LSI-анализ (семантическое обогащение контента) |
| **Блок** | 3: Контент и оптимизация (09–13) |
| **Основная роль** | **A:** SEO-стратег |
| **Консультативные** | **D:** Аналитик конкурентной разведки · **F:** Промпт-инженер · **G:** Инженер контрактов данных |
| **Тип** | Промпт + Скрипт |
| **HitL** | Нет (результат → Шаги 11, 12 автоматически) |
| **Приоритет ROI** | ВЫСОКИЙ — семантическая полнота напрямую влияет на Topical Authority и ранжирование в Яндекс.Нейро / Google AI Mode |
| **Зависит от** | Шаг 09 (Текстовая релевантность) · `PROJECT.md` · Qdrant-коллекции `competitors`, `content_chunks` |
| **Передаёт в** | Шаг 11 (Метатеги) · Шаг 12 (SEO-текст) · Шаг 08 (обновление ТЗ) · Шаг 29 (AEO-оптимизация) |
| **JSON-шаблон** | Template 1: Список с приоритетами |
| **Режим Opus** | JSON-вывод: `temperature 0.15`, `top_p 0.9`, `max_tokens 8192` |
| **Golden test** | Прогон на нише `dental_moscow_001` (эталон верифицирован) |

---

## ██ ЦЕЛЬ

Сформировать приоритизированный список **LSI-терминов** (латентно-семантических индексов) и **тематических сущностей** для каждой целевой страницы/кластера, которые необходимо внедрить в контент, чтобы увеличить тематическую релевантность (Topical Authority), закрыть семантические дыры относительно топ-10 SERP и усилить цитируемость в AI-ответах (Яндекс.Нейро, Google AI Mode, SGE).

Шаг отвечает на вопрос: **«Каких смысловых блоков и сущностей не хватает странице, чтобы ранжироваться на уровне топ-3?»**

---

## ██ КОНТРАКТ ВХОДНЫХ ДАННЫХ (JSON)

Вход — выход Шага 09 (Текстовая релевантность) + параметры проекта. **Роль G** верифицирует схему.

```json
{
  "project_id": "string",
  "prev_step_id": "09",
  "prompt_version": "string",
  "timestamp": "ISO 8601",
  "pages": [
    {
      "page_url": "string",
      "page_cluster": "string",
      "page_type": "landing|category|article|faq|product",
      "target_queries": [
        {
          "query": "string",
          "engine": "yandex|google|both",
          "intent": "informational|commercial|transactional|navigational",
          "frequency": "number",
          "current_position": "number|null"
        }
      ],
      "current_content": {
        "h1": "string",
        "text_sample": "string (первые 3000 символов для контекста)",
        "word_count": "number",
        "existing_terms": ["string"]
      },
      "relevance_score": "number 0-100",
      "relevance_gaps": ["string (из Шага 09)"]
    }
  ],
  "project_context": {
    "niche_cluster": "string",
    "geo": "string",
    "language": "string",
    "priority_engines": "yandex|google|both",
    "main_competitors": ["string"]
  },
  "competitor_content_signals": {
    "source": "pre_script_10_competitor_scrape",
    "enabled": "boolean"
  }
}
```

**Обязательные поля:** `pages[]` (≥1), у каждой страницы — `target_queries[]` (≥1), `current_content.h1`, `page_cluster`.

---

## ██ КОНТРАКТ ВЫХОДНЫХ ДАННЫХ (JSON)

Наследует **Template 1: Список с приоритетами**, расширен LSI-специфичными полями. **Роль G** валидирует.

```json
{
  "project_id": "string",
  "step_id": "10",
  "prompt_version": "step10_v1.0",
  "timestamp": "ISO 8601",
  "items": [
    {
      "id": "LSI-{page_cluster}-{NNN}",
      "page_url": "string",
      "page_cluster": "string",
      "title": "string (краткое описание термина/сущности)",
      "lsi_term": "string (сам LSI-термин)",
      "term_type": "co_occurrence|entity|synonym|attribute|intent_modifier|ontology_relation",
      "semantic_relevance": "number 0-1 (cosine similarity к ядру кластера)",
      "co_occurrence_score": "number 0-1 (частота совместного появления с целевыми запросами в топ-10)",
      "competitor_usage_pct": "number 0-100 (% топ-10, использующих термин)",
      "current_in_content": "boolean (есть ли уже в тексте страницы)",
      "recommended_frequency": "number (рекомендуемое число вхождений)",
      "placement_hint": "h1|h2|h3|intro|body|faq|meta|alt|any",
      "priority": "critical|high|medium|low",
      "action_required": "string (конкретная инструкция для Шага 12)",
      "engine_tag": "[Я]|[G]|[ОБА]",
      "estimated_impact": "number 1-10",
      "effort": "number 1-10",
      "source": "competitor_top10|serp_snippet|wikipedia_entity|ai_synthesis|yandex_wordstat_related|google_related_searches",
      "rationale": "string (обоснование — почему критично)"
    }
  ],
  "topic_coverage": [
    {
      "page_url": "string",
      "coverage_score_before": "number 0-100",
      "coverage_score_after_projected": "number 0-100",
      "missing_subtopics": ["string"],
      "dominant_entities": ["string"]
    }
  ],
  "summary": {
    "total_items": 0,
    "critical": 0,
    "high": 0,
    "medium": 0,
    "low": 0,
    "yandex_specific": 0,
    "google_specific": 0,
    "both": 0,
    "pages_processed": 0
  },
  "engine_split": {
    "yandex_lexical_notes": "string (особенности Яндекс-семантики для ниши: морфология, словоформы, синонимы)",
    "google_semantic_notes": "string (особенности Google: BERT-контекст, сущности, Knowledge Graph)"
  },
  "qc_score": "number 0-100"
}
```

**Инварианты контракта:**
- Каждый `item.engine_tag ∈ {[Я], [G], [ОБА]}` — без исключений.
- При `priority_engines == "both"` в `items[]` обязательно присутствуют записи со всеми тремя тегами.
- `priority == "critical"` ⟹ `estimated_impact ≥ 8` и `competitor_usage_pct ≥ 60`.
- Если `current_in_content == true` и `recommended_frequency` совпадает с фактической — термин не попадает в `items[]`.

---

## ██ ПРОМПТ ДЛЯ CLAUDE OPUS

### Системный промпт

```
Ты — Роль A: Senior SEO-стратег с 10+ летним опытом семантического SEO
для Яндекса и Google. Консультируют: Роль D (конкурентная разведка),
Роль F (промпт-инженер), Роль G (контракты данных).

ЗАДАЧА: Провести LSI-анализ целевых страниц и выдать приоритизированный
список семантических терминов и сущностей для обогащения контента.

ЧТО ТАКОЕ LSI В ЭТОМ ШАГЕ:
1. co_occurrence — термины, совместно встречающиеся с целевыми запросами в топ-10
2. entity — именованные сущности (бренды, локации, персоны, продукты)
3. synonym — синонимы и морфологические варианты целевых запросов
4. attribute — атрибуты/свойства (материал, цена, срок, технология)
5. intent_modifier — модификаторы интента (купить, отзывы, цена, сравнение)
6. ontology_relation — родовидовые связи, часть-целое, причина-следствие

ПРАВИЛА:
1. Анализируй КАЖДУЮ страницу из input.pages отдельно — минимум 8, максимум
   25 LSI-терминов на страницу.
2. Каждый термин ОБЯЗАТЕЛЬНО помечен engine_tag: [Я] | [G] | [ОБА].
3. Приоритизация:
   - critical: competitor_usage_pct ≥ 60, current_in_content == false,
     semantic_relevance ≥ 0.75
   - high: competitor_usage_pct ≥ 40 ИЛИ semantic_relevance ≥ 0.85
   - medium: competitor_usage_pct 20-40
   - low: усиливающие термины, < 20% конкурентов
4. Специфика поисковиков:
   - [Я] Яндекс: учитывай морфологию и словоформы, спектр синонимов,
     Яндекс.Нейро цитирует фактологические блоки с прямыми ответами,
     ранжирование по запрос-документ CatBoost, важны КФ-термины
     (цена, отзывы, наличие, доставка, гарантия) для commercial_*.
   - [G] Google: учитывай BERT-контекст, сущности Knowledge Graph,
     E-E-A-T-сигналы (автор, источник), Helpful Content Update —
     value-driven синонимы, не keyword stuffing.
   - [ОБА]: базовые co-occurrence термины, не зависящие от поисковика.
5. Адаптация под niche_cluster:
   - commercial_local: геомодификаторы, NAP-сущности, "рядом со мной"
   - ecom: атрибуты товара, цена, бренд, размер, отзывы
   - medical (YMYL): симптомы, МКБ-коды, специализации врачей, лицензии
   - saas: интеграции, альтернативы, use-case, ROI, pricing-tier
   - info: подтемы, вопросы-якоря, FAQ-сущности
   - legal (YMYL): статьи закона, прецеденты, процедуры
   - education: уровни (начальный/продвинутый), форматы, сертификации
6. Для каждого LSI-термина укажи placement_hint — где разместить (h1|h2|h3|
   intro|body|faq|meta|alt|any).
7. Формула priority_score внутри блока:
   priority_score = 0.35 * semantic_relevance + 0.35 * (competitor_usage_pct/100)
   + 0.20 * (estimated_impact/10) + 0.10 * intent_match
8. АНТИПАТТЕРНЫ (строго запрещено):
   - Keyword stuffing: не предлагай recommended_frequency > 5 для словоформы.
   - Общие термины без привязки к нише.
   - Термины без engine_tag.
   - Дубли синонимов с разной морфологией (сгруппируй в один item).
   - Игнорирование relevance_gaps из Шага 09.

ВЫВОД: Валидный JSON строго по контракту OUTPUT. Без markdown-обёртки,
без комментариев, без префикса ```json. Весь текст — на РУССКОМ языке.
Ключи JSON — на английском согласно контракту.
```

### Пользовательский промпт (шаблон с переменными)

```
Проведи LSI-анализ для проекта {{project_id}}.

КОНТЕКСТ ПРОЕКТА:
- Ниша: {{project_context.niche_cluster}}
- Гео: {{project_context.geo}}
- Язык: {{project_context.language}}
- Приоритетные поисковики: {{project_context.priority_engines}}
- Основные конкуренты: {{project_context.main_competitors | join(', ')}}

СТРАНИЦЫ ДЛЯ АНАЛИЗА ({{pages.length}} шт.):

{{#each pages}}
─────────────────────────────────────
Страница #{{@index}}: {{page_url}}
Кластер: {{page_cluster}} · Тип: {{page_type}}
H1: {{current_content.h1}}
Объём текста: {{current_content.word_count}} слов
Relevance score (из Шага 09): {{relevance_score}}/100
Семантические дыры (из Шага 09): {{relevance_gaps | join('; ')}}

Целевые запросы:
{{#each target_queries}}
  - "{{query}}" [{{engine}}] · intent={{intent}} · freq={{frequency}} · pos={{current_position}}
{{/each}}

Существующие термины в тексте:
{{current_content.existing_terms | join(', ')}}

Фрагмент текущего контента:
{{current_content.text_sample}}
─────────────────────────────────────
{{/each}}

{{#if competitor_content_signals.enabled}}
КОНКУРЕНТНЫЕ СИГНАЛЫ (из pre-script 10, Qdrant коллекция competitors):
{{rag_competitor_context}}
{{/if}}

{{#if rag_ontology_context}}
ОНТОЛОГИЯ НИШИ (из Qdrant коллекции niches):
{{rag_ontology_context}}
{{/if}}

ЗАДАНИЕ: Для каждой страницы сформируй items[] с LSI-терминами по контракту,
заполни topic_coverage[], summary, engine_split и qc_score.
```

---

## ██ RAG-ОБОГАЩЕНИЕ

Запрашиваемые Qdrant-коллекции:

| Коллекция | Запрос | Что извлекаем | Лимит |
|---|---|---|---|
| `competitors` | `niche + geo + page_cluster` | Топ-5 чанков контента конкурентов по кластеру, частотные термины | 800 токенов |
| `content_chunks` | `target_queries[] embedding` | Собственные страницы того же кластера (для избежания каннибализации) | 400 токенов |
| `niches` | `niche_cluster` | Онтология ниши: сущности, отношения, типовые LSI-паттерны | 500 токенов |
| `serp_snippets` | `target_queries[]` | Snippet + PAA + Related searches из DataForSEO SERP | 300 токенов |

**Итоговый лимит RAG-контекста:** 2000 токенов.
**Инъекция:** переменные `rag_competitor_context`, `rag_ontology_context` в user-промпте.
**Стратегия извлечения:** hybrid search (dense BGE-M3 + BM25), top-k=8 на коллекцию, re-ranking по MMR для разнообразия.

---

## ██ СКРИПТЫ CLAUDE CODE

### Pre-step 1: Парсер контента конкурентов

```python
# step_10_pre_competitor_scrape.py
# Назначение: Сбор контента топ-10 SERP по целевым запросам для LSI-извлечения
# Вход:  pages[].target_queries[] из Шага 09
# Выход: competitor_content_signals.json → Qdrant collection 'competitors'
#
# Действия:
# 1. Для каждого уникального target_query → DataForSEO SERP API (Яндекс + Google)
# 2. Для топ-10 URL каждой выдачи: Playwright-краулинг, извлечение основного
#    текста (trafilatura/readability), очистка от boilerplate
# 3. Чанкинг по 512 токенов с оверлапом 64
# 4. Эмбеддинг через BGE-M3 (многоязычный) → upsert в Qdrant с метадатой
#    {project_id, query, engine, competitor_domain, page_url}
# 5. Параллельно: TF-IDF и RAKE экстракция ключевых фраз на корпусе
#    топ-10 → файл lsi_candidates_raw.json
# 6. Кеш SERP на 7 дней в NocoDB (экономия кредитов DataForSEO)
#
# Обработка ошибок:
# - Cloudflare/CAPTCHA → пометить URL как skipped, продолжить
# - Robots.txt disallow → пропустить
# - < 3 успешных URL на запрос → лог warning, запрос помечен как low_confidence
```

### Pre-step 2: Извлечение расширений (wordstat + related searches)

```python
# step_10_pre_semantic_expansion.py
# Назначение: Сбор синонимов, словоформ и related-запросов из нативных источников
# Вход:  pages[].target_queries[]
# Выход: semantic_expansion.json
#
# Действия:
# 1. [Я] Яндекс.Wordstat API (через Директ API или парсер):
#    - "Похожие запросы" (правая колонка Wordstat)
#    - Морфологические варианты через pymorphy3
# 2. [G] Google:
#    - Related Searches (DataForSEO keywords_data)
#    - People Also Ask (PAA) вопросы
#    - Autocomplete suggestions (prefix + A..Я, A..Z)
# 3. Фильтрация:
#    - Минимум 10 показов/мес для [Я]
#    - Минимум 10 impressions для [G]
# 4. Дедупликация нормализованных форм (лемматизация spaCy ru_core_news_lg)
# 5. Маркировка источника: yandex_wordstat_related | google_related_searches | paa
```

### Pre-step 3: Расчёт co-occurrence и semantic relevance

```python
# step_10_pre_cooccurrence_calc.py
# Назначение: Подготовить численные сигналы для Opus (чтобы LLM не галлюцинировал цифры)
# Вход:  competitor_content_signals, semantic_expansion
# Выход: lsi_signals.json — передаётся в prompt как rag_competitor_context
#
# Действия:
# 1. Для каждого кандидата-термина из lsi_candidates_raw.json:
#    - competitor_usage_pct = (число URL топ-10 с термином) / (всего URL) * 100
#    - co_occurrence_score = PMI(term, target_query) нормализованный
#    - semantic_relevance = cosine(embedding(term), centroid(target_queries))
# 2. Кластеризация синонимов по cosine > 0.90 → группировка в один item
# 3. Отсев шума:
#    - Частотность < 5% в корпусе ИЛИ semantic_relevance < 0.35 → drop
#    - Стоп-слова ниши (обновляемый список в NocoDB)
# 4. Выход: упорядоченный список до 50 кандидатов на страницу
#    (Opus финально отберёт 8-25 и расставит приоритеты)
```

### Post-step 1: Валидатор JSON-контракта

```python
# step_10_post_validate.py
# Назначение: Проверка выхода Opus на соответствие OUTPUT CONTRACT + бизнес-правила
# Вход:  response.json от Opus
# Выход: validation_report.json (pass|review|retry) + qc_score
#
# Проверки (rule-based, без ML):
# 1. JSON Schema (ajv): все обязательные поля, типы корректны
# 2. items.length: на каждую входную page — от 8 до 25 items
# 3. engine_tag: 100% items имеют [Я]|[G]|[ОБА]
# 4. При priority_engines == "both": есть записи со всеми тремя тегами
# 5. Consistency: если current_in_content == true и recommended_frequency
#    == фактической в existing_terms → ошибка (не должно быть в items)
# 6. Priority-инварианты:
#    - critical ⟹ estimated_impact ≥ 8 И competitor_usage_pct ≥ 60
#    - сумма critical+high на страницу ≤ 60% от items (не всё критично)
# 7. Антистаффинг: recommended_frequency ≤ 5 для любой словоформы
# 8. summary.* численно сходится с подсчётом по items[]
# 9. topic_coverage есть для каждой input.pages
# 10. engine_split: yandex_lexical_notes и google_semantic_notes непустые
#     при priority_engines == "both"
#
# При провале: формируется qc_errors[] → вставляется в prompt на retry
```

### Post-step 2: Семантический QC (каннибализация)

```python
# step_10_post_semantic_qc.py
# Назначение: Убедиться, что LSI-рекомендации не создают каннибализацию
# Вход:  items[], existing pages from Qdrant collection 'content_chunks'
# Выход: cannibalization_warnings[]
#
# Действия:
# 1. Для каждого item с priority ∈ {critical, high}:
#    - Составить hypothetical enriched_text = current_content + lsi_term * N
#    - Эмбеддинг → cosine со всеми остальными страницами проекта
#    - Если cosine > 0.85 с другой страницей → warning (возможная каннибализация)
# 2. Warnings записываются в NocoDB, НЕ блокируют выдачу (Шаг 04.1
#    уже отвечает за каннибализацию, здесь только сигнал)
```

### Post-step 3: Сохранение в NocoDB и Qdrant

```python
# step_10_post_persist.py
# Назначение: Персистентность результатов + обновление RAG
# Действия:
# 1. NocoDB:
#    - таблица seo_lsi_items: items[] с project_id, step_id, prompt_version
#    - таблица seo_topic_coverage: topic_coverage[]
#    - лог запуска в seo_step_runs
# 2. Qdrant:
#    - upsert в коллекцию 'lsi_recommendations': эмбеддинг lsi_term + метадата
#    - тег project_id, page_cluster — для последующего использования в Шагах 11, 12
# 3. Передача в N8N state-машину: trigger_next = ["step_11", "step_12"]
```

---

## ██ КРИТЕРИИ QC

| Проверка | Порог | Вес | При провале |
|---|---|---|---|
| JSON Schema валидна | Pass | 15 | Retry |
| items на страницу | 8 ≤ N ≤ 25 | 15 | Retry |
| Все items имеют engine_tag | 100% | 10 | Retry |
| При `both`: есть [Я], [G], [ОБА] | Каждый ≥1 | 10 | Retry |
| Инвариант critical (impact ≥ 8 & competitor_pct ≥ 60) | 100% critical | 10 | Retry |
| Антистаффинг: `recommended_frequency ≤ 5` | 100% | 10 | Retry |
| `topic_coverage` для каждой входной страницы | 100% | 5 | Retry |
| `engine_split` заполнен (при `both`) | Оба поля ≠ "" | 5 | Ревью |
| Каннибализационные warnings | ≤ 2 на проект | 10 | Ревью |
| Соответствие niche_cluster | Не менее 70% items релевантны кластеру | 10 | Ревью |

**Скоринг:** сумма весов по пройденным проверкам из 100.
**Результаты:** ≥ 80 → передача в Шаги 11, 12 · 60–79 → ревью · < 60 → retry (до 3 раз).

---

## ██ СТРУКТУРА N8N-ВОРКФЛОУ

```
Нода 1:  Триггер (по завершению Шага 09 ИЛИ ручной запуск)
Нода 2:  Загрузка output Шага 09 из NocoDB (таблица seo_relevance)
Нода 3:  Загрузка PROJECT.md
Нода 4:  step_10_pre_competitor_scrape.py (Code-нода)
         └─ параллельно для всех уникальных queries
Нода 5:  step_10_pre_semantic_expansion.py (Code-нода)
Нода 6:  step_10_pre_cooccurrence_calc.py (Code-нода)
         └─ выход: lsi_signals.json
Нода 7:  Qdrant RAG-запрос (коллекции competitors, niches, content_chunks, serp_snippets)
Нода 8:  Сборка промпта (Template нода + merge lsi_signals + rag_context)
Нода 9:  Claude Opus API (temp 0.15, top_p 0.9, max_tokens 8192)
Нода 10: Парсинг JSON (с удалением markdown-обёртки на случай некорректного вывода)
Нода 11: step_10_post_validate.py
         └─ провал → Нода 8 с qc_errors[] в промпт (макс. 3 цикла)
Нода 12: step_10_post_semantic_qc.py
Нода 13: step_10_post_persist.py (NocoDB + Qdrant upsert)
Нода 14: Grafana metrics push (lsi_terms_count, avg_qc_score, retry_count)
Нода 15: Trigger Шагов 11, 12 · либо стоп при qc_score < 60 после 3 retry
```

### Обработка ошибок

- **Opus 429/500:** exponential backoff 10с → 30с → 90с, макс. 3 попытки.
- **DataForSEO квота исчерпана:** fallback на кешированный SERP (NocoDB, TTL 7 дн.).
- **Qdrant недоступен:** работа без RAG-контекста, qc_score ограничен 85.
- **Cloudflare блок конкурентов:** skip URL, warning в лог, продолжение.
- **QC fail 3x:** алерт в Telegram с 3 последними логами Opus + блокировка Шагов 11, 12.
- **Невалидный JSON от Opus:** regex-удаление ```json…```, повторный parse; при неудаче — retry.

---

## ██ ПРИМЕР FEW-SHOT (сокращённый)

**Ниша:** `dental_moscow_001` (детская стоматология, Москва, commercial_local, both engines)
**Входная страница:** `/uslugi/lechenie-kariesa-u-detey/`

```json
{
  "project_id": "dental_moscow_001",
  "step_id": "10",
  "prompt_version": "step10_v1.0",
  "timestamp": "2026-04-16T10:42:11Z",
  "items": [
    {
      "id": "LSI-detskaya-stomatologiya-001",
      "page_url": "/uslugi/lechenie-kariesa-u-detey/",
      "page_cluster": "detskaya_stomatologiya_karies",
      "title": "Серебрение молочных зубов",
      "lsi_term": "серебрение молочных зубов",
      "term_type": "co_occurrence",
      "semantic_relevance": 0.82,
      "co_occurrence_score": 0.74,
      "competitor_usage_pct": 70,
      "current_in_content": false,
      "recommended_frequency": 3,
      "placement_hint": "h2",
      "priority": "critical",
      "action_required": "Добавить H2-блок «Серебрение молочных зубов» с описанием метода, показаний и ограничений (≥150 слов)",
      "engine_tag": "[ОБА]",
      "estimated_impact": 9,
      "effort": 4,
      "source": "competitor_top10",
      "rationale": "7 из 10 конкурентов топ-10 Яндекса и Google раскрывают метод как альтернативу сверлению у малышей 2-4 лет. Отсутствие блока = критическая семантическая дыра для запросов 'лечение кариеса без сверления ребёнку'."
    },
    {
      "id": "LSI-detskaya-stomatologiya-002",
      "page_url": "/uslugi/lechenie-kariesa-u-detey/",
      "page_cluster": "detskaya_stomatologiya_karies",
      "title": "ICON (айкон) технология",
      "lsi_term": "ICON технология лечения кариеса",
      "term_type": "entity",
      "semantic_relevance": 0.79,
      "co_occurrence_score": 0.62,
      "competitor_usage_pct": 50,
      "current_in_content": false,
      "recommended_frequency": 2,
      "placement_hint": "body",
      "priority": "high",
      "action_required": "Упомянуть ICON в блоке современных методов, сослаться на DMG (производитель)",
      "engine_tag": "[G]",
      "estimated_impact": 7,
      "effort": 3,
      "source": "wikipedia_entity",
      "rationale": "Google хорошо распознаёт ICON как entity в Knowledge Graph, усиливает E-E-A-T. Яндекс на запрос 'icon лечение кариеса' не показывает фичу — для [Я] приоритет ниже."
    },
    {
      "id": "LSI-detskaya-stomatologiya-003",
      "page_url": "/uslugi/lechenie-kariesa-u-detey/",
      "page_cluster": "detskaya_stomatologiya_karies",
      "title": "Цена и наличие",
      "lsi_term": "цена лечения кариеса у ребёнка",
      "term_type": "intent_modifier",
      "semantic_relevance": 0.68,
      "co_occurrence_score": 0.81,
      "competitor_usage_pct": 90,
      "current_in_content": false,
      "recommended_frequency": 2,
      "placement_hint": "h2",
      "priority": "critical",
      "action_required": "Добавить блок-таблицу с ценами (от-до) — критический КФ для Яндекса",
      "engine_tag": "[Я]",
      "estimated_impact": 10,
      "effort": 2,
      "source": "yandex_wordstat_related",
      "rationale": "Коммерческий фактор Яндекса: 9/10 топа имеют явный ценовой блок. Без цены на commercial_local в Яндексе — потеря ПФ и КФ."
    }
  ],
  "topic_coverage": [
    {
      "page_url": "/uslugi/lechenie-kariesa-u-detey/",
      "coverage_score_before": 54,
      "coverage_score_after_projected": 87,
      "missing_subtopics": ["серебрение", "ICON", "озонотерапия", "седация закисью азота", "цена"],
      "dominant_entities": ["кариес", "молочные зубы", "пломба", "анестезия"]
    }
  ],
  "summary": {
    "total_items": 14,
    "critical": 4,
    "high": 5,
    "medium": 3,
    "low": 2,
    "yandex_specific": 5,
    "google_specific": 3,
    "both": 6,
    "pages_processed": 1
  },
  "engine_split": {
    "yandex_lexical_notes": "Для коммерческого детского-стомат. кластера в Яндексе критичны КФ-сущности: цена, адрес, телефон, запись онлайн. Морфология: 'ребёнку/ребенку/детям' — нужны все варианты в H2 и alt. Яндекс.Нейро цитирует FAQ-блоки с прямым ответом ≤60 слов.",
    "google_semantic_notes": "Google опирается на E-E-A-T: обязательна авторская подпись детского стоматолога с сертификатом. Knowledge Graph распознаёт ICON, DMG, МКБ-код K02. Helpful Content Update — избегать повторов маркетинговых клише, value-first контент."
  },
  "qc_score": 89
}
```

---

## ██ СТРАТЕГИЯ ОТКАТА (Fallback при 3x QC fail)

**Откат A — Декомпозиция по страницам:**
Разбить input.pages на отдельные вызовы Opus (по одной странице за вызов). Уменьшает сложность и context-load, часто решает проблемы с невалидным JSON на больших инпутах.

**Откат B — Декомпозиция по этапам:**
Вызов 1: выдать только `items[]` с базовыми полями (lsi_term, term_type, priority, engine_tag).
Вызов 2: обогатить items числовыми полями (semantic_relevance, co_occurrence_score) из pre-script-сигналов детерминированно (без LLM).
Вызов 3: сгенерировать `topic_coverage` и `engine_split` отдельным вызовом.

**Откат C — Сокращение до top-8 на страницу:**
Жёсткий лимит 8 items/страница, только priority ∈ {critical, high}. Пометка `fallback_mode=true` в NocoDB для ручного расширения.

**Откат D — Few-shot primer:**
Подать Opus пример из golden test set (nicha + result) как образец формата. Уменьшает галлюцинации структуры.

**Откат E — Ручная ревью:**
После 3-х откатов — алерт в Telegram с ссылкой на черновик и qc_errors[]. SEO-специалист правит вручную в NocoDB, Шаги 11, 12 запускаются по ручному триггеру.

---

## ██ АДАПТАЦИИ ПОД КЛАСТЕР

| Кластер | Фокус LSI | Приоритет engine | Специфика |
|---|---|---|---|
| **commercial_local** | Геомодификаторы, NAP-сущности, «рядом», «в [район]», КФ-термины (цена, часы работы, запись) | [Я] доминирует | Яндекс.Бизнес-атрибуты, 2GIS-рубрики, микрогеография станций метро |
| **commercial_national** | Бренд + продукт + атрибут, сравнения, «обзор», «рейтинг» | [ОБА] паритет | Sitelinks-сущности, Featured Snippets для «vs»-запросов |
| **ecom** | Атрибуты товара (размер, цвет, материал, бренд), «отзывы», «купить», ценовые модификаторы | [ОБА] | Schema Product entities, фильтровые LSI-ветви |
| **saas** | Use-case термины, интеграции, «альтернатива Х», ROI, pricing-tier, компаративы | [G] доминирует | English-термины допустимы, Entity SEO через G2/Capterra |
| **info** | Подтемы из PAA, Wikipedia-entities, вопросы-якоря, семантика FAQ | [ОБА] | AEO-приоритет, Featured Snippets, Яндекс.Нейро citation |
| **medical** (YMYL) | МКБ-коды, симптомы, названия процедур, специализации врачей, лицензионные термины | [ОБА] + HitL-ревью | Медицинская терминология обязательна, E-E-A-T-сущности (автор с ФИО врача, лицензия) |
| **legal** (YMYL) | Статьи законов, прецеденты, процедуры, сроки, инстанции | [Я]+[G] | Ссылки на источники права, entity: суды, органы |
| **education** | Уровни (beginner/intermediate/advanced), форматы, сертификации, предметные области | [ОБА] | Сезонные модификаторы, student-journey термины |

---

## ██ МЕТРИКИ УСПЕХА

**Опережающие (сразу после Шага 10):**
- `qc_score ≥ 80` для ≥ 90% страниц прогона.
- Средняя `topic_coverage_after_projected − coverage_score_before ≥ +25` пунктов.
- ≥ 1 `critical`-item на каждую целевую страницу.
- `cannibalization_warnings ≤ 2` на проект.

**Запаздывающие (после внедрения в Шагах 11, 12 + переиндексация через 14–30 дней):**
- Рост text-relevance score (Шаг 09) на ≥ 15 пунктов после применения items.
- Движение позиций в top-10 по ≥ 40% целевых запросов страницы.
- Появление цитат в Яндекс.Нейро / Google AI Mode по запросам со страницы (метрика Шага 40.3 AI-citation-tracking).
- Рост impressions в GSC/Вебмастере на ≥ 20% за 30 дней после публикации.

**Контрольные (quality gate):**
- Ручной спот-чек 10% items на предмет релевантности кластеру.
- Сравнение с golden test set — отклонение по priority-распределению > 20% → алерт.
- A/B-ревью: 5% items проходят через HitL случайной выборкой; accuracy > 85% = промпт стабилен.

**Бюджет токенов:** средний вызов Opus на шаге — 6–9K входных + 4–7K выходных токенов (зависит от числа страниц). Предел — 10 страниц за вызов, больше → декомпозиция.

---

## ██ ИСТОРИЯ ИЗМЕНЕНИЙ

| Версия | Дата | Изменения |
|---|---|---|
| v1.0 | 2026-04 | Первая редакция. Шаблон Template 1 + LSI-расширения. Pre-scripts: competitor scrape, semantic expansion, co-occurrence. Post: validate, semantic QC, persist. Адаптации под 8 кластеров, engine_split для [Я]/[G]. Few-shot на кейсе `dental_moscow_001`. |

---

**Контрольный чек-лист соответствия CORE.md + STEP_00-эталону:**
- [x] Основная роль A + консультативные D/F/G обозначены
- [x] Opus mode зафиксирован (JSON-вывод: 0.15/0.9/8192)
- [x] Engine_tag [Я]/[G]/[ОБА] обязателен в каждом item
- [x] System prompt явно требует: русский язык + JSON без markdown
- [x] QC rule-based only (ML отключён)
- [x] Retry-протокол v2 интегрирован (Retry 1: ошибки в промпт → Retry 2: декомпозиция → Retry 3: fallback + few-shot → алерт)
- [x] JSON-контракт наследует Template 1 и расширен LSI-полями
- [x] Input contract = output Шага 09
- [x] Output передаётся в Шаги 11, 12, 08, 29
- [x] Few-shot реалистичен для русскоязычного рынка (Москва, стоматология)
- [x] Антипаттерны прописаны в системном промпте
- [x] Метрики опережающие + запаздывающие + контрольные
- [x] Все разделы на русском; JSON-ключи на английском
