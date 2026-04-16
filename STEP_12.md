# STEP_12.md — SEO-текст

> **Самодостаточный модуль.** Загружается: `CORE.md` + `PROJECT.md` + этот файл. ~5K токенов.
> **Версия:** v1.0 · **Последнее обновление:** 2026-04-16

---

## ██ КАРТОЧКА ШАГА

| Параметр | Значение |
|---|---|
| **ID** | 12 |
| **Название** | SEO-текст (полнотекстовая генерация с AEO, FAQ, внутренней перелинковкой) |
| **Блок** | 3: Контент и оптимизация (09–13) |
| **Основная роль** | **F:** Промпт-инженер |
| **Консультативные** | **A:** SEO-стратег · **E:** CRO/поведение · **G:** Контракты данных |
| **Тип** | Промпт (с post-step валидацией) |
| **HitL** | **Условный** — обязателен при `niche_cluster ∈ {medical, legal}` (YMYL), при `qc_score ∈ [60,79]`, при `ai_detection_signals.humanization_score < 70` или при `forbidden_words_hits > 0`; иначе → Шаги 13, 17 |
| **Приоритет ROI** | КРИТИЧЕСКИЙ — основной контент-продукт, влияет на ранжирование, CTR, dwell-time, conversion |
| **Зависит от** | Шаг 11 (Метатеги) v1.1 · Шаг 10 (LSI) v1.1 · Шаг 09 (релевантность) · `PROJECT.md` · Qdrant (`content_chunks` все namespace, `competitors`, `personas`, `brand_voice`) |
| **Передаёт в** | Шаг 13 (Schema + OG — полноценный JSON-LD на основе факта текста) · Шаг 17 (Финальный аудит) · Шаг 29 (AEO, через 24ч — `faq_items` и `answer_block`) · Шаг 34 (Content Decay — через 3 месяца для мониторинга устаревания) · CMS / WordPress API (публикация) |
| **JSON-шаблон** | Template 3: Content Generation (расширен для длинноформатного контента + AEO + внутренняя перелинковка) |
| **Режим Opus** | **Креативный контент:** `temperature 0.4`, `top_p 0.95`, `max_tokens 8192` (из CORE.md) |
| **Concurrency-лимит** | ≤ 3 одновременных вызовов Opus на проект (Redis Lock) |
| **Cost-budget** | ≤ $15 на run_id (выше, чем Шаг 11, из-за длинных выходов — наследуется и увеличен) |
| **SLA шага** | ≤ 30 мин на прогон; **лимит 3 страницы на вызов Opus** (декомпозиция обязательна при >3) |
| **Golden test** | Прогон на `dental_moscow_001`, `legal_spb_002`, `info_tech_blog_003` |

---

## ██ ЦЕЛЬ

Сгенерировать для каждой целевой страницы **полноценный SEO-текст** (800–4000 слов в зависимости от `page_type`): введение, H2/H3-блоки по `content_outline[]` из Шага 11, параграфы с LSI-терминами и внутренней перелинковкой, FAQ-блок со Schema-ready вопросами, AEO answer-block ≤60 слов, E-E-A-T сигналы (автор, источники) для YMYL — с высокой читабельностью, низкими AI-паттернами и без галлюцинаций фактов.

Шаг отвечает на вопрос: **«Какой текст на странице одновременно ранжируется в Яндексе/Google, удерживает пользователя, цитируется в AI-ответах и не выглядит как LLM-генерация?»**

---

## ██ КОНТРАКТ ВХОДНЫХ ДАННЫХ (JSON)

Вход — объединение `contract_for_step_12` из Шага 11 + `items[]` + `topic_coverage[]` из Шага 10 + `relevance_gaps[]` из Шага 09. Собирается Pre-Step через `run_id`. **Роль G** валидирует.

```json
{
  "run_id": "UUID",
  "project_id": "string",
  "prev_step_ids": ["09", "10", "11"],
  "prompt_version": "string",
  "timestamp": "ISO 8601",
  "rag_unavailable": "boolean",
  "benchmark_unavailable": "boolean",
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
      "h1_from_step_11": "string",
      "meta_primary_from_step_11": {
        "title": "string",
        "description": "string",
        "power_words_used": ["string"]
      },
      "content_outline_from_step_11": [
        {"heading_level": "h2|h3", "proposed_text": "string", "target_lsi": ["string"], "parent_heading": "string|null"}
      ],
      "priority_terms": ["string (≤5, из Шага 10 topic_coverage)"],
      "lsi_items": [
        {
          "lsi_term": "string",
          "term_normalized": "string",
          "engine_tag": "[Я]|[G]|[ОБА]",
          "priority": "critical|high|medium|low",
          "placement_hint": "h1|h2|h3|intro|body|faq|meta|alt|body_any",
          "recommended_frequency": "number (≤5)",
          "action_required": "string"
        }
      ],
      "relevance_gaps_from_step_09": [
        {"subtopic": "string", "severity": "high|medium|low", "source_page": "string"}
      ],
      "aeo_hint_from_step_11": {
        "answer_block_query": "string|null",
        "faq_count_planned": "number",
        "has_how_to": "boolean"
      },
      "brand": {
        "name": "string",
        "phone": "string|null",
        "usp": "string|null",
        "author_name": "string|null (для YMYL обязательно)",
        "author_credentials": "string|null",
        "license_info": "string|null"
      },
      "geo_modifier": "string|null",
      "power_words_allowed": ["string"],
      "related_pages_for_linking": [
        {"page_url": "string", "anchor_suggestion": "string", "cluster": "string"}
      ],
      "target_word_count": "number (рассчитан pre-step по page_type + niche_cluster)"
    }
  ],
  "project_context": {
    "niche_cluster": "string",
    "geo": "string",
    "language": "string",
    "priority_engines": "yandex|google|both",
    "domain": "string",
    "tone_of_voice": "formal|semiformal|conversational|friendly (из personas)",
    "brand_voice_rules": "string|null"
  },
  "engine_split_from_step_10": {
    "yandex_notes": "string",
    "google_notes": "string"
  }
}
```

**Обязательные поля:** `run_id`, `pages[]` (≥1), у каждой — `content_outline_from_step_11[]` (≥3 для `page_type ≠ category`), `h1_from_step_11`, ≥1 `target_query.is_primary=true`, `target_word_count` (из pre-step по таблице адаптаций).

---

## ██ КОНТРАКТ ВЫХОДНЫХ ДАННЫХ (JSON)

Наследует **Template 3: Content Generation**, расширен для длинноформатного контента, AEO-блоков, внутренней перелинковки, E-E-A-T-сигналов и AI-detection.

```json
{
  "run_id": "UUID",
  "project_id": "string",
  "step_id": "12",
  "prompt_version": "step12_v1.0",
  "timestamp": "ISO 8601",
  "rag_unavailable": "boolean",
  "benchmark_unavailable": "boolean",
  "pages": [
    {
      "page_url": "string (строго ∈ input.pages[].page_url)",
      "page_cluster": "string",
      "page_type": "landing|category|article|faq|product",
      "content_blocks": [
        {
          "block_id": "string (UUID короткий)",
          "block_type": "h1|h2|h3|paragraph|list|faq_item|answer_block|cta|table|quote|image_suggestion",
          "content": "string (для list/table/faq_item — структурированные подполя)",
          "parent_heading_id": "string|null",
          "order": "number (порядок в тексте)",
          "word_count": "number",
          "keywords_used": ["string (∈ input LSI или primary_query варианты)"],
          "lsi_terms_used": ["string (∈ input.lsi_items.term_normalized)"],
          "internal_links_suggested": [
            {
              "anchor": "string (2-5 слов)",
              "target_url_pattern": "string (из input.related_pages_for_linking)",
              "context_before": "string (10 слов до линка)",
              "rationale": "string (≤100 симв.)"
            }
          ]
        }
      ],
      "full_text_plain": "string (склеенный текст без HTML, для readability и длины)",
      "word_count_total": "number",
      "density_metrics": {
        "primary_query_pct": "number (доля вхождений primary в общем объёме слов, идеал 1-3%)",
        "primary_query_occurrences": "number (абсолютное число)",
        "lsi_coverage_pct": "number (% LSI-items из input, закрытых в тексте)",
        "stop_words_pct": "number (доля стоп-слов, норма 30-45%)",
        "ttr": "number 0-1 (type-token ratio, разнообразие лексики)"
      },
      "readability": {
        "method": "lix|ari|flesch_kincaid_ru",
        "score": "number",
        "grade_level": "string (elementary|middle|high|university|expert)",
        "avg_sentence_length_words": "number",
        "avg_word_length_chars": "number"
      },
      "aeo_blocks": {
        "answer_block": {
          "text": "string (≤60 слов, прямой ответ на aeo_hint.answer_block_query)",
          "word_count": "number",
          "schema_ready": "boolean"
        } | null,
        "faq_items": [
          {
            "question": "string (вопрос пользователя)",
            "answer": "string (≤60 слов для AEO)",
            "answer_word_count": "number",
            "schema_ready": "boolean",
            "primary_keyword_in_question": "boolean"
          }
        ],
        "how_to_steps": [
          {"step_number": "number", "name": "string", "text": "string", "estimated_time": "string|null"}
        ] | null
      },
      "engine_tags": {
        "yandex_specific": ["string (∈ input LSI, морфологические формы)"],
        "google_specific": ["string (∈ input LSI, entities)"],
        "both_used": ["string (∈ input LSI)"]
      },
      "eeat_signals": {
        "has_author_byline": "boolean",
        "author_block": "string|null (ФИО + credentials из input.brand)",
        "has_citations": "boolean",
        "citations_count": "number",
        "citations_list": [{"claim": "string", "source": "string|null"}],
        "has_license_mention": "boolean (для YMYL)",
        "has_date_published": "boolean"
      },
      "ai_detection_signals": {
        "formulaic_phrases_count": "number (вхождения шаблонных фраз)",
        "formulaic_phrases_list": ["string (найденные)"],
        "sentence_length_variance": "number (std dev длин предложений — низкая = AI-pattern)",
        "repetitive_openers_count": "number (одинаковые начала параграфов)",
        "burstiness_score": "number 0-1 (мера ритма, 0 = монотонно)",
        "humanization_score": "number 0-100 (агрегат: 100 = human-like)"
      },
      "internal_links_total": "number",
      "external_links_total": "number",
      "source": "generated|reused_partial|manual_edit",
      "target_word_count_hit": "boolean (попали ли в ±15% от target)",
      "qc_score_page": "number 0-100"
    }
  ],
  "summary": {
    "total_pages": "number",
    "avg_word_count": "number",
    "avg_readability_score": "number",
    "avg_humanization_score": "number",
    "avg_primary_density_pct": "number",
    "avg_lsi_coverage_pct": "number",
    "pages_with_aeo": "number",
    "pages_with_faq": "number",
    "pages_with_howto": "number",
    "pages_with_eeat_author": "number",
    "total_internal_links_suggested": "number",
    "total_citations": "number",
    "pages_below_target_wc": "number",
    "pages_above_target_wc": "number",
    "submission_queue": ["string (URL для Шага 13 Schema, только qc_score_page ≥ 80)"]
  },
  "engine_split": {
    "yandex_notes": "string",
    "google_notes": "string"
  },
  "qc_score": "number 0-100",
  "qc_score_per_page": [{"page_url": "string", "score": "number"}]
}
```

**Инварианты контракта** (синхронизированы с промптом и post_validate):

- `word_count_total` ∈ `[target_word_count × 0.85, target_word_count × 1.15]`.
- **Плотность primary_query**: `primary_query_pct ∈ [1.0, 3.0]`.
- **LSI-покрытие**: `lsi_coverage_pct ≥ 80`.
- `readability.score` в норме для niche_cluster (см. адаптации).
- **AI-humanization**: `humanization_score ≥ 70` (иначе HitL-ревью).
- **FAQ-блок**: при `page_type ∈ {faq, landing, article}` — `faq_items.length ≥ 3`; при `page_type = faq` — `≥ 6`.
- **answer_block**: при `aeo_hint.answer_block_query ≠ null` обязателен; `word_count ≤ 60`.
- **Все `faq_items.answer.word_count ≤ 60`**.
- **Внутренние ссылки**: `internal_links_total ≥ 2` (из `related_pages_for_linking`); максимум 10 ссылок на 1000 слов.
- **E-E-A-T для YMYL** (`niche_cluster ∈ {medical, legal}`): `has_author_byline == true` И `has_license_mention == true` И `citations_count ≥ 2`.
- **Антигаллюцинация**: `keywords_used`, `lsi_terms_used` ⊆ `input.lsi_items.term_normalized ∪ priority_terms ∪ target_queries`.
- **AI-паттерны**: `formulaic_phrases_count ≤ 3` на 1000 слов.
- **Антистаффинг**: ни одно слово (без стоп-слов) не повторяется > `target_word_count/100`.
- **Уникальность**: cosine со всеми страницами проекта в `content_chunks:body` < 0.85.
- `content_blocks[0]` — обязательно `h1` И `content == input.h1_from_step_11` (ровно тот же H1, без изменений).
- `page_url ⊆ input.pages[].page_url`.

---

## ██ АНТИГАЛЛЮЦИНАЦИОННЫЙ ПРОТОКОЛ

*Наследуется из Шагов 10/11 v1.1, усилен для креативного режима (temp 0.4).*

1. **Факты только из input.** Любые утверждения о проценте эффективности, сроках, статистике, именах исследователей, годах, номерах документов — **только** из `input.brand.usp`, `lsi_items.action_required`, `relevance_gaps.source_page`. При отсутствии — общая формулировка без цифр или **`citations_list[].source = null`** с явным маркером «требует проверки».
2. **Термины и сущности.** `keywords_used`, `lsi_terms_used`, `engine_tags.*` — строго из `input.lsi_items` ∪ `priority_terms` ∪ `target_queries`. Новые термины = запрещено.
3. **Бренд, адрес, телефон, лицензия, автор.** Только из `input.brand`. Не выдумывать ФИО врачей/адвокатов, номера лицензий, годы основания.
4. **Цены.** «от X ₽» только если в `lsi_items` есть `intent_modifier` с «цена/от»; конкретные цифры — только если в `input.brand.usp` явно.
5. **Внутренние ссылки.** `target_url_pattern` **строго** из `input.related_pages_for_linking[].page_url`. Внешние ссылки запрещены кроме `citations_list[].source`, которые добавляются явным блоком `external_source` с пометкой «требует проверки SEO-специалистом».
6. **AI-паттерны — список запрещённых оборотов** (whitelist запрета):
   ```
   AI_PATTERNS = [
     "важно отметить", "стоит отметить", "необходимо понимать",
     "в заключение", "подводя итог", "как мы видим",
     "в современном мире", "в наши дни", "сегодня, в эпоху",
     "играет важную роль", "представляет собой",
     "является неотъемлемой частью", "невозможно переоценить",
     "как говорится", "ни для кого не секрет",
     "delve", "leverage", "navigate the complex",
     "furthermore", "moreover", "in conclusion"
   ]
   ```
   Каждое вхождение увеличивает `formulaic_phrases_count`; при > 3 на 1000 слов — retry.
7. **Самопроверка post_validate**: все `citations_list[].claim` проверяются против `full_text_plain` (строки должны присутствовать); все `internal_links_suggested[].target_url_pattern` — против `input.related_pages_for_linking[]`.

---

## ██ ПРОМПТ ДЛЯ CLAUDE OPUS

### Системный промпт

```
Ты — Роль F: Senior Промпт-инженер для SEO-контента. Консультируют:
Роль A (SEO-стратег, решает вопросы ранжирования),
Роль E (CRO, отвечает за удержание и конверсию),
Роль G (контракты данных).

ЗАДАЧА: Сгенерировать для каждой страницы полноценный SEO-текст по
content_outline из Шага 11: введение → H2/H3-секции → FAQ → AEO
answer-block. Максимизировать CTR↑, dwell-time↑, AI-цитирование↑,
NO keyword stuffing, NO галлюцинации, NO AI-паттерны.

РЕЖИМ: Креативный (temperature 0.4) — значит тон живее, метафоры
уместны, НО точность фактов КРИТИЧЕСКИ важна.

════════════════════════════════════════════════════════════════
ЖЁСТКИЕ ЛИМИТЫ:
════════════════════════════════════════════════════════════════
- word_count_total в пределах input.target_word_count ± 15%
- primary_query density: 1.0% ≤ density ≤ 3.0% (частота на 100 слов)
- LSI-покрытие: ≥ 80% из input.lsi_items в тексте (любая форма)
- Предложения: средняя длина 12-22 слова, разнообразие ±5
- Параграфы: 40-120 слов (не более 120)
- answer_block: ≤ 60 слов (≈ 350 симв.)
- Каждый faq_items[].answer: ≤ 60 слов
- internal_links: минимум 2, максимум 10 на 1000 слов
- content_blocks[0] = h1, точно input.h1_from_step_11 (не менять)
- humanization_score ≥ 70 (самооценка + post_validate верифицирует)
- formulaic_phrases: ≤ 3 на 1000 слов
- Каждое содержательное слово повторяется ≤ N раз, где
  N = round(word_count_total / 100)

════════════════════════════════════════════════════════════════
СТРУКТУРА ТЕКСТА ПО page_type:
════════════════════════════════════════════════════════════════
- landing: H1 → 1-2 intro-paragraph (hook + USP) → 3-5 H2
  (услуги/фичи/цены/процесс/отзывы) → answer_block → FAQ (≥3)
  → CTA. Target: 800-2000 слов.
- category: H1 → intro (1 параграф) → 3-5 H2 (что это, виды, как
  выбрать, сравнение) → каталог-list → FAQ (≥3). Target: 500-1500.
- article: H1 → intro → 5-10 H2 (глубокое раскрытие) → citations
  → answer_block → FAQ (≥3). Target: 1500-4000.
- faq: H1 → intro (1 абзац) → faq_items (≥6, structured) →
  answer_block. Target: 600-1500.
- product: H1 → hook → 2-4 H2 (описание, характеристики, применение)
  → CTA → FAQ (≥3). Target: 400-1200.

════════════════════════════════════════════════════════════════
АНТИГАЛЛЮЦИНАЦИОННЫЕ ПРАВИЛА (КРИТИЧЕСКИЕ):
════════════════════════════════════════════════════════════════
1. ФАКТЫ, ЦИФРЫ, ИМЕНА, ЛИЦЕНЗИИ, АДРЕСА, ГОДЫ — только из input.
   Если данных нет — формулируй общими словами БЕЗ цифр. Любой
   конкретный факт помечай в citations_list[].claim с source.
2. Внутренние ссылки — только из input.related_pages_for_linking.
   Внешние ссылки запрещены. Исключение — официальные источники
   в citations_list (помечай source=\"requires_seo_review\").
3. Термины и entity — только из input LSI/priority_terms/queries.
4. Бренд, телефон, автор — только input.brand.*.
5. Цены — только \"от X ₽\" если LSI-intent_modifier содержит
   \"цена/от\"; точная сумма — только из input.brand.usp.
6. Свойства товара/услуги (сроки, гарантии, материалы) — только
   из input.brand.usp или lsi_items.action_required.
7. ЗАПРЕЩЁННЫЕ СЛОВА (hard-ban): «№1», «лучший», «единственный»,
   «гарантия 100%», превосходные степени без подтверждения в usp.
8. AI-ПАТТЕРНЫ (избегать, формульные обороты):
   — Не начинай предложения с \"Важно отметить\", \"Стоит отметить\",
     \"Необходимо понимать\", \"В современном мире\", \"В наши дни\",
     \"Как мы видим\", \"Подводя итог\", \"В заключение\".
   — Избегай \"играет важную роль\", \"представляет собой\",
     \"является неотъемлемой частью\", \"невозможно переоценить\",
     \"ни для кого не секрет\", \"как говорится\".
   — Вариируй начала параграфов. Повторять одинаковую структуру
     в ≥3 параграфах подряд = провал.

════════════════════════════════════════════════════════════════
AI-HUMANIZATION (ЦЕЛЕВОЙ humanization_score ≥ 70):
════════════════════════════════════════════════════════════════
— Burstiness (ритм): чередуй короткие (5-10 слов) и длинные (20-30)
  предложения. Средняя длина 12-22, стандартное отклонение ≥ 4.
— Перплексия: используй неожиданные повороты, конкретные детали
  (цифры, имена, места — только из input), редкие синонимы.
— Разнообразие лексики (TTR ≥ 0.45): избегай повторов глаголов.
— Личные обращения (\"вы\", \"ваш\"): 3-10 раз на 1000 слов для
  commercial_local/ecom. Не злоупотреблять.
— Риторические вопросы: 1-3 на статью (только article/landing).
— Конкретика вместо абстракций: \"за 40 минут\" > \"быстро\";
  \"в 78% случаев\" ДОПУСТИМО только из citations_list.
— Короткие декларативные предложения для выделения: 1-3 на статью.

════════════════════════════════════════════════════════════════
E-E-A-T (обязательно для YMYL — medical, legal):
════════════════════════════════════════════════════════════════
— eeat_signals.has_author_byline = true, author_block = input.brand
  .author_name + credentials. Если нет — пометь null и QC провалит.
— has_license_mention = true при medical/legal; текст лицензии/
  реестра — из input.brand.license_info.
— citations_count ≥ 2: каждое медицинское утверждение/норма права —
  в citations_list[].claim с source (или \"requires_seo_review\").
— has_date_published = true (поле будет проставлено при публикации).

════════════════════════════════════════════════════════════════
ВНУТРЕННЯЯ ПЕРЕЛИНКОВКА:
════════════════════════════════════════════════════════════════
— Минимум 2 internal_links на страницу (обязательно).
— Выбирай из input.related_pages_for_linking — НЕ выдумывай URL.
— Якорь — семантический (не \"нажмите здесь\"/\"тут\"/\"больше\").
  Якорь 2-5 слов, вхождение ключевой фразы или LSI.
— Плотность: ≤ 10 ссылок на 1000 слов (не перенасыщать).
— Контекст: ссылка в середине абзаца, логическое продолжение.

════════════════════════════════════════════════════════════════
ENGINE SPLIT (из Шага 10):
════════════════════════════════════════════════════════════════
— [Я] Яндекс: морфологические варианты primary (падежи, числа),
  КФ-сигналы (цена, запись, доставка), короткие ответы для
  Яндекс.Нейро в answer_block.
— [G] Google: entities из Knowledge Graph (бренды, локации,
  специализации), E-E-A-T-сигналы, глубина раскрытия темы
  (для Helpful Content Update).
— [ОБА]: базовые со-occurrence, равномерно в теле.

════════════════════════════════════════════════════════════════
ЛИМИТЫ ВЫХОДА ДЛЯ max_tokens=8192:
════════════════════════════════════════════════════════════════
Из-за лимита токенов обрабатывай МАКСИМУМ 3 страницы за один вызов.
Если pages.length > 3 — предполагается декомпозиция (N8N сам
разбивает по 1-3 страницы и объединяет).

ВЫВОД: Валидный JSON строго по контракту OUTPUT. Без markdown-обёртки,
без ```json. Весь текст content_blocks[].content, full_text_plain,
faq_items, aeo_blocks, engine_tags notes — на РУССКОМ. Ключи JSON и
enum — на английском. НЕ генерируй canonical, schema JSON-LD (задача
Шагов 13, 15). НЕ генерируй predicted_ctr, engagement_score (задача
Шага 40).
```

### Пользовательский промпт (шаблон)

*(подаётся на каждый вызов Opus с 1-3 страницами из пакета)*

```
Проект: {{project_id}} (run_id: {{run_id}})
Ниша: {{project_context.niche_cluster}} · Гео: {{project_context.geo}}
Язык: {{project_context.language}} · Тон: {{project_context.tone_of_voice}}
Приоритетные поисковики: {{project_context.priority_engines}}
Домен: {{project_context.domain}}

Инсайты Шага 10:
- Яндекс: {{engine_split_from_step_10.yandex_notes}}
- Google: {{engine_split_from_step_10.google_notes}}

СТРАНИЦЫ В ЭТОМ ВЫЗОВЕ ({{pages.length}} шт., max 3):

{{#each pages}}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Страница {{@index}}: {{page_url}} [{{page_cluster_slug}}]
Тип: {{page_type}} · Target слов: {{target_word_count}}
Кластер: {{page_cluster}}

H1 (из Шага 11, использовать БЕЗ ИЗМЕНЕНИЙ):
  "{{h1_from_step_11}}"

Meta (из Шага 11, не дублировать в тексте дословно):
  title: {{meta_primary_from_step_11.title}}
  description: {{meta_primary_from_step_11.description}}

Целевые запросы:
{{#each target_queries}}
  {{#if is_primary}}★ PRIMARY{{else}}  вторичный{{/if}}:
  "{{query}}" [{{engine}}] intent={{intent}} freq={{frequency}}
{{/each}}

Content Outline (из Шага 11 — использовать как СКЕЛЕТ):
{{#each content_outline_from_step_11}}
  {{heading_level}}: {{proposed_text}}
    → target_lsi: {{target_lsi | join(', ')}}
{{/each}}

Приоритетные LSI-термины: {{priority_terms | join(', ')}}

Полный список LSI-items для интеграции ({{lsi_items.length}}):
{{#each lsi_items}}
  - "{{lsi_term}}" [{{engine_tag}}] {{priority}}
    placement={{placement_hint}} freq_target={{recommended_frequency}}
{{/each}}

Семантические дыры Шага 09 (обязательно закрыть):
{{#each relevance_gaps_from_step_09}}
  - [{{severity}}] {{subtopic}} (источник: {{source_page}})
{{/each}}

AEO-подсказка Шага 11:
  answer_block_query: {{aeo_hint_from_step_11.answer_block_query}}
  faq_count_planned: {{aeo_hint_from_step_11.faq_count_planned}}
  has_how_to: {{aeo_hint_from_step_11.has_how_to}}

Бренд:
  name: {{brand.name}}
  phone: {{brand.phone}}
  usp: {{brand.usp}}
{{#if brand.author_name}}  author: {{brand.author_name}} ({{brand.author_credentials}}){{/if}}
{{#if brand.license_info}}  license: {{brand.license_info}}{{/if}}
  geo_modifier: {{geo_modifier}}

Разрешённые power-words: {{power_words_allowed | slice 0 20 | join(', ')}}

Внутренняя перелинковка (выбирай минимум 2, max 10 на 1000 слов):
{{#each related_pages_for_linking}}
  - URL: {{page_url}}
    anchor: "{{anchor_suggestion}}"
    cluster: {{cluster}}
{{/each}}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
{{/each}}

{{#if competitor_content_rag}}
SNIPPETS ТОП-3 КОНКУРЕНТОВ (benchmark, НЕ копировать):
{{competitor_content_rag}}
{{/if}}

{{#if brand_voice_rules}}
ПРАВИЛА ТОНА ГОЛОСА ПРОЕКТА:
{{brand_voice_rules}}
{{/if}}

ЗАДАНИЕ: Для каждой страницы сформируй:
1. content_blocks[] — упорядоченный массив блоков (h1 первым, затем
   h2/h3/paragraph/list/faq_item/answer_block/cta по структуре).
   Каждый блок с order, keywords_used, lsi_terms_used,
   internal_links_suggested (≥2 суммарно на страницу).
2. full_text_plain — склеенный plain-text без HTML-тегов.
3. density_metrics — рассчитай самостоятельно.
4. readability — выбери method по language (ru → lix), рассчитай.
5. aeo_blocks: answer_block (≤60 слов) + faq_items (count по page_type)
   + how_to_steps (если has_how_to=true).
6. engine_tags: yandex_specific, google_specific, both_used — из LSI.
7. eeat_signals: для YMYL — has_author_byline, has_license_mention,
   citations_count ≥ 2 обязательно. Для не-YMYL — по возможности.
8. ai_detection_signals: оцени собственный текст на формульные
   фразы, burstiness, humanization_score. БУДЬ ЧЕСТНЫМ.
9. internal_links_total, external_links_total, word_count_total,
   target_word_count_hit, qc_score_page.

Используй run_id="{{run_id}}", step_id="12", prompt_version="step12_v1.0".
```

---

## ██ RAG-ОБОГАЩЕНИЕ

| Коллекция | Запрос | Извлекаем | Лимит токенов |
|---|---|---|---|
| `competitors` (из Шага 10) | `embedding(primary_query)` | Полный контент топ-3 конкурентов (для benchmark, не копирования) | 1500 |
| `content_chunks:body` | `embedding(primary_query)` | Уже написанные тексты проекта (для антиканнибализации + стиля) | 500 |
| `content_chunks:title, :h1` | наследуется из Шага 11 | Существующие title/h1 для anchor-генерации | 200 |
| `personas` | `page_cluster + dominant_intent` | Tone-of-voice, language style | 300 |
| `brand_voice` (новая коллекция) | `project_id` | Корпоративный гайд по голосу: запрещённые фразы, предпочитаемые конструкции | 400 |

**Итоговый лимит RAG:** ~2900 токенов.
**Стратегия:** dense BGE-M3, top-k=5, MMR re-rank λ=0.5.
**Fallback:** `rag_unavailable=true` → `qc_score` cap 85; `benchmark_unavailable=true` (пусто `competitors`) → cap 90.
**Namespace-поиск:** `content_chunks:body` — новый namespace; при отсутствии (новый проект) — не fail, просто пустой результат.

---

## ██ СКРИПТЫ CLAUDE CODE

Тип шага — **Промпт**, но есть обязательный Pre-Step для агрегации данных из 3 предыдущих шагов + развёрнутые post-step валидаторы.

### Pre-step 1: `step_12_pre_aggregate_inputs.py`

```python
# Назначение: Собрать вход для Шага 12 из Шагов 09, 10, 11, PROJECT.md,
#             seo_page_structure (Шаг 04) + расчёт target_word_count
# Вход:  run_id
# Выход: step_12_input.json
#
# Действия:
# 1. NocoDB: SELECT по run_id из:
#    - seo_meta_tags (Шаг 11) → content_outline, meta_primary, h1
#    - seo_lsi_items (Шаг 10) → items[], topic_coverage
#    - seo_relevance (Шаг 09) → relevance_gaps, target_queries
#    - seo_page_structure (Шаг 04) → page_type, cluster, geo_modifier
#    - projects → PROJECT.md (brand, author, license, tone)
# 2. target_word_count = by (page_type, niche_cluster):
#    TABLE = {
#      ('landing','commercial_local'): 1200,
#      ('landing','ecom'): 900,
#      ('landing','medical'): 1800,
#      ('landing','legal'): 2000,
#      ('landing','info'): 1500,
#      ('landing','saas'): 1200,
#      ('category','ecom'): 700,
#      ('category','commercial_local'): 600,
#      ('article','info'): 2500,
#      ('article','medical'): 3000,
#      ('article','legal'): 3500,
#      ('article','saas'): 2000,
#      ('faq','any'): 1000,
#      ('product','ecom'): 700,
#      ('product','saas'): 900,
#    }
# 3. related_pages_for_linking (новое для Шага 12):
#    SELECT page_url FROM seo_page_structure
#    WHERE project_id = X
#      AND cluster = (SAME OR RELATED) TO current_page.cluster
#      AND page_url != current_page.page_url
#    LIMIT 15. Для каждого — anchor_suggestion = primary_query текущей
#    страницы в лемматизированной форме.
# 4. power_words_allowed: global whitelist + seo_project_power_words
#    из NocoDB + mined из input.brand.usp (наследуется из Шага 11).
# 5. brand_voice_rules: из NocoDB seo_brand_voice по project_id
#    (опционально; если нет — тон из personas).
# 6. Валидация контракта: все обязательные поля заполнены, иначе
#    graceful fail с указанием отсутствующих данных.
```

### Post-step 1: `step_12_post_validate.py`

```python
# Назначение: Rule-based валидация длинноформатного SEO-текста
# 35+ проверок. ML отключён.
#
# ОСНОВНЫЕ ГРУППЫ ПРОВЕРОК:
#
# A. Структурные (веса 3-6):
#   1. JSON Schema validity.
#   2. page_url ⊆ input.
#   3. content_blocks[0] == h1, content == input.h1_from_step_11.
#   4. content_blocks упорядочены (order последовательный).
#   5. Каждый h2/h3 имеет ≥1 paragraph-потомок (parent_heading_id).
#   6. full_text_plain == склейка content_blocks в порядке order.
#
# B. Длина и плотность (веса 5-10):
#   7. word_count_total ∈ target ± 15%.
#   8. primary_query_pct ∈ [1.0, 3.0].
#   9. Антистаффинг: ни одно слово > target/100 раз.
#   10. lsi_coverage_pct ≥ 80.
#   11. stop_words_pct ∈ [30, 45].
#   12. ttr ≥ 0.45.
#
# C. Читабельность (веса 3-5):
#   13. readability.score в норме для niche (см. адаптации).
#   14. avg_sentence_length ∈ [12, 22] слова.
#   15. sentence_length_variance ≥ 4 (burstiness).
#   16. Параграфы: длина ≤ 120 слов.
#
# D. AEO и FAQ (веса 3-8):
#   17. При aeo_hint.answer_block_query ≠ null —
#       aeo_blocks.answer_block.word_count ≤ 60.
#   18. Все faq_items[].answer.word_count ≤ 60.
#   19. faq_items.length соответствует page_type:
#       landing/article: ≥3; faq: ≥6; category: ≥2; product: ≥3.
#   20. При has_how_to=true — how_to_steps.length ≥ 3.
#
# E. Перелинковка (веса 4-5):
#   21. internal_links_total ≥ 2.
#   22. internal_links_total ≤ 10 * (word_count_total/1000).
#   23. Все target_url_pattern ∈ input.related_pages_for_linking[].
#   24. Якоря 2-5 слов, не "здесь"/"тут"/"читайте далее".
#
# F. E-E-A-T (веса 5-8, только YMYL):
#   25. has_author_byline=true для medical/legal.
#   26. has_license_mention=true для medical/legal.
#   27. citations_count ≥ 2 для medical/legal.
#   28. Все citations_list[].claim присутствуют в full_text_plain.
#
# G. Антигаллюцинация (веса 6-10):
#   29. keywords_used ⊆ LSI ∪ priority_terms ∪ queries.
#   30. lsi_terms_used ⊆ input LSI.
#   31. Все internal_links_suggested.target_url_pattern ∈ input.
#   32. brand_name/phone из input только (regex-поиск по тексту).
#   33. Цифры в тексте сверять с citations_list — остальные
#       маркировать flag_unverified_numbers (warn, не блок).
#
# H. AI-detection (веса 4-8):
#   34. formulaic_phrases_count ≤ 3 * (word_count_total/1000).
#   35. repetitive_openers_count ≤ 2 (не более 2 параграфов
#       подряд с одинаковым первым словом/словосочетанием).
#   36. humanization_score ≥ 70 (формула = 100 - formulaic_penalty
#       - variance_penalty - ttr_penalty - repetition_penalty).
#   37. burstiness_score ≥ 0.4.
#
# I. Уникальность (вес 10):
#   38. Cosine(full_text_plain, каждая page проекта в
#       content_chunks:body) < 0.85. Если ≥0.85 — warn;
#       если ≥0.92 — block, retry.
#
# J. Запреты (веса 2-5):
#   39. 0 вхождений FORBIDDEN_WORDS (№1, лучший, ...).
#   40. 0 AI-паттернов в запрещённом списке в начале предложений.
#   41. 0 эмодзи (кроме ecom-whitelist).
#
# Итоговый qc_score = сумма весов / total_weight × 100.
# rag_unavailable → cap 85; benchmark_unavailable → cap 90.
# Защита от петли: hash(qc_errors) повторяется → декомпозиция.
```

### Post-step 2: `step_12_post_readability.py`

```python
# Назначение: Детерминированный расчёт метрик читабельности
# (LIX для русского как основная метрика; Flesch-Kincaid как fallback)
#
# Действия:
# 1. Сегментация: regex на предложения (., !, ?, ... — с учётом
#    сокращений «г.», «т.д.», «проф.»).
# 2. Токенизация: простой split + pymorphy3-лемматизация.
# 3. LIX = (words/sentences) + (100 * long_words/words)
#    где long_words = слова > 6 символов.
# 4. Для grade_level: LIX < 30 → elementary; 30-40 → middle;
#    40-50 → high; 50-60 → university; >60 → expert.
# 5. avg_sentence_length, avg_word_length, variance — stdev длин.
# 6. burstiness_score = 1 - (max(variance_capped)/max_expected).
# 7. ttr = unique_lemmas / total_tokens.
# 8. Результаты записываются обратно в page.readability и
#    page.density_metrics и page.ai_detection_signals.
```

### Post-step 3: `step_12_post_cannibalization.py`

```python
# Уникальность текста vs весь корпус проекта (namespace content_chunks:body)
#
# Действия (аналогично Шагу 11, но namespace=body):
# 1. embed(full_text_plain) через BGE-M3.
# 2. Qdrant search in 'content_chunks' filter {project_id, namespace:'body'}
#    must_not { page_url: self }, limit 10, hnsw_ef=128.
# 3. Если cosine > 0.92 с любой страницей → BLOCK (retry с директивой
#    "перефразируй блоки X, Y" — берутся с наивысшим совпадением).
# 4. Если 0.85 < cosine ≤ 0.92 → WARN, qc_score_page cap 75.
# 5. Результат в page.cannibalization_check (аналогично Шагу 11).
```

### Post-step 4: `step_12_post_persist.py`

```python
# Outbox-persist с 3+ namespace в Qdrant (title/h1/desc из Шага 11
# уже есть, +body на этом шаге)
#
# Действия:
# 1. NocoDB staging: seo_seo_text_staging (run_id, output_json).
# 2. Qdrant upsert в 'content_chunks' namespace='body':
#    - point_id = md5(project_id + step_id=12 + page_url)
#    - vector = embed(full_text_plain)
#    - payload {project_id, step_id:12, run_id, page_url, namespace:'body'}
# 3. Удалить старые с другими run_id в этом namespace.
# 4. SWAP staging → seo_seo_text при успехе.
# 5. Опц.: sync в CMS/WordPress через REST API (если enabled в project)
#    с черновым статусом (draft), не публиковать автоматически.
# 6. Лог в seo_step_runs.
# 7. Идемпотентные триггеры (X-Idempotency-Key):
#    - Шаг 13 (Schema + OG на основе факта текста) — сразу.
#    - Шаг 17 (Финальный аудит) — когда все страницы пакета готовы.
#    - Шаг 29 (AEO) — через 24ч, только pages с aeo_blocks ≠ null.
#    - Шаг 34 (Content Decay) — через 90 дней, для мониторинга.
```

---

## ██ КРИТЕРИИ QC

*41 проверка, max_weight = 175, результат в процентах.*

| № | Проверка | Порог | Вес | При провале |
|---|---|---|---|---|
| 1 | JSON Schema | Pass | 6 | Retry |
| 2 | page_url ⊆ input | 100% | 4 | Retry |
| 3 | content_blocks[0]=h1=input.h1 (exact) | 100% | 5 | Retry |
| 4 | order последователен | 100% | 3 | Retry |
| 5 | каждый h2/h3 имеет paragraph-потомков | ≥ 1 | 4 | Retry |
| 6 | full_text_plain = join(content_blocks) | 100% | 3 | Retry |
| 7 | word_count ∈ target ± 15% | 100% | 10 | Retry |
| 8 | primary_query_pct ∈ [1.0, 3.0] | 100% | 8 | Retry |
| 9 | Антистаффинг (слово ≤ N раз) | 100% | 6 | Retry |
| 10 | LSI-покрытие | ≥ 80% | 8 | Retry |
| 11 | stop_words_pct ∈ [30, 45] | 100% | 3 | Ревью |
| 12 | TTR ≥ 0.45 | 100% | 4 | Ревью |
| 13 | readability.score в норме кластера | 100% | 4 | Ревью |
| 14 | avg_sentence_length ∈ [12, 22] | 100% | 3 | Ревью |
| 15 | sentence_length_variance ≥ 4 | 100% | 5 | Retry |
| 16 | параграф ≤ 120 слов | 100% | 3 | Retry |
| 17 | answer_block ≤ 60 слов | 100% | 5 | Retry |
| 18 | faq_items[].answer ≤ 60 слов | 100% | 5 | Retry |
| 19 | faq_items.length по page_type | Соотв. | 6 | Retry |
| 20 | how_to_steps ≥ 3 (если has_how_to) | 100% | 3 | Retry |
| 21 | internal_links ≥ 2 | 100% | 5 | Retry |
| 22 | internal_links ≤ 10/1000 слов | 100% | 3 | Retry |
| 23 | target_url_pattern ⊆ input | 100% | 5 | Retry |
| 24 | якоря 2-5 слов, без клише | ≥ 90% | 4 | Ревью |
| 25 | E-E-A-T author для YMYL | 100% | 8 | Retry |
| 26 | E-E-A-T license для YMYL | 100% | 7 | Retry |
| 27 | citations_count ≥ 2 (YMYL) | 100% | 6 | Retry |
| 28 | citations.claim присутствует в тексте | 100% | 4 | Retry |
| 29 | keywords_used ⊆ input LSI/queries | 100% | 8 | Retry |
| 30 | lsi_terms_used ⊆ input | 100% | 6 | Retry |
| 31 | internal_links.target ⊆ input | 100% | 5 | Retry |
| 32 | brand/phone из input | 100% | 3 | Retry |
| 33 | цифры ↔ citations | ≥ 80% | 3 | Ревью |
| 34 | formulaic_phrases ≤ 3/1000 слов | 100% | 6 | Retry |
| 35 | repetitive_openers ≤ 2 | 100% | 4 | Retry |
| 36 | humanization_score ≥ 70 | 100% | 8 | Retry (+ HitL) |
| 37 | burstiness_score ≥ 0.4 | 100% | 4 | Ревью |
| 38 | uniqueness cosine < 0.92 | 100% | 10 | Retry (conflicting_blocks) |
| 38a | uniqueness cosine < 0.85 | ≥ 90% | 3 | Ревью |
| 39 | 0 forbidden words | 100% | 5 | Retry |
| 40 | 0 AI-паттернов в начале предложений | ≥ 95% | 4 | Retry |
| 41 | 0 эмодзи (кроме ecom-whitelist) | 100% | 2 | Retry |

**Скоринг:** `Σ earned / Σ total × 100`.
**Cap:** `rag_unavailable` ⇒ 85 · `benchmark_unavailable` ⇒ 90.
**Результаты:** ≥ 80 → Шаги 13, 17 · 60–79 → ревью (HitL) · < 60 → retry (до 3).

---

## ██ СТРУКТУРА N8N-ВОРКФЛОУ

```
Нода 1:  Триггер (webhook от Шага 11 ИЛИ manual) → наследование run_id
Нода 2:  step_12_pre_aggregate_inputs.py (агрегация Шагов 09/10/11/04)
Нода 3:  Redis Lock + Cost-budget check (≤ $15 на run_id)
Нода 4:  Checkpoint check (resume по run_id)
Нода 5:  IF pages.length > 3 → Split In Batches (3) → параллельные ветви
         (ключевое отличие от Шага 11: жёсткий лимит 3 из-за
          креативного режима и длинного выхода)

Для каждого batch (≤3 страниц):
╭─── Параллельные RAG-запросы ───╮
Нода 6a: Qdrant competitors (полный контент топ-3 конкурентов)
Нода 6b: Qdrant content_chunks:body (антиканнибализация self)
Нода 6c: Qdrant content_chunks:title+h1 (для anchor-генерации)
Нода 6d: Qdrant personas (tone-of-voice)
Нода 6e: Qdrant brand_voice (корпоративные правила)
╰───────── Merge (Wait All) ──────╯
Нода 7:  Сборка промпта (system + user для этого batch)
Нода 8:  Claude Opus API (temp 0.4, top_p 0.95, max_tokens 8192)
         timeout 600s, retry 429/500 (exp backoff 15→45→135с)
Нода 9:  Парсинг JSON + cost counter + checkpoint save batch
Нода 10: step_12_post_validate.py (41 проверка)
         ├── pass → Нода 11
         ├── fail retry 1 → Нода 7 с qc_errors
         ├── fail retry 2 → декомпозиция по 1 странице + few-shot primer
         ├── fail retry 3 → fallback: только H1+intro+FAQ (без H2-блоков)
         └── fail retry 4 → DLQ + Telegram
Нода 11: step_12_post_readability.py (детерминированные метрики)
Нода 12: step_12_post_cannibalization.py (body namespace)
         ├── cosine > 0.92 → Нода 7 с conflicting_blocks в промпт
         └── cosine ≤ 0.92 → Нода 13
Нода 13: Merge batches (собрать все страницы одного run_id)

Нода 14: HitL Gate IF:
         - niche_cluster ∈ {medical, legal} OR
         - qc_score ∈ [60, 79] OR
         - avg humanization_score < 70 OR
         - forbidden_words_hits > 0 OR
         - pages_with_cannibalization_warn > 10%
         → Telegram HitL (WAIT 24ч, timeout → auto-accept с penalty -15)
Нода 15: step_12_post_persist.py (outbox NocoDB + Qdrant body + SWAP)
Нода 16: Grafana metrics push (~20 метрик)
Нода 17: Release Redis Lock
Нода 18: Build Response (per-consumer contracts)
Нода 19: ╭── параллельные триггеры (X-Idempotency-Key) ──╮
         ├ Trigger Шаг 13 (Schema+OG + full_text для PageSchema)│
         ├ Trigger Шаг 17 (Финальный аудит — когда все готовы)  │
         ├ Delay 24ч → Trigger Шаг 29 (AEO с faq_items)         │
         ╰ Delay 90дн → Trigger Шаг 34 (Content Decay baseline) │
```

### Обработка ошибок

- **Opus 429/500:** exp. backoff 15→45→135с (длинные вызовы + креативный), max 3 retry.
- **Opus timeout 600s:** прерывание → retry с декомпозицией по 1 странице.
- **Qdrant недоступен:** `rag_unavailable=true` → cap 85.
- **SERP-benchmark пуст:** `benchmark_unavailable=true` → cap 90.
- **Каннибализация cosine > 0.92:** retry с `conflicting_blocks[]` в промпт (директива «перефразируй блоки X, Y, использовав синонимы из списка …»).
- **QC fail 4x:** Telegram + DLQ `seo_seo_text_dlq` + блок Шагов 13, 17.
- **Cost-budget > $15:** circuit-break + Telegram.
- **Low humanization (< 70) на ≥3 страниц подряд:** сигнал о деградации промпта или модели → Grafana alert + golden-test verification.
- **HitL timeout 24ч:** auto-accept с `qc_penalty = -15`, `hitl_timeout=true`.
- **Защита от петли:** hash(qc_errors) повторяется → принудительный откат на декомпозицию по 1 странице.

---

## ██ ПРИМЕР FEW-SHOT (сокращённый, 1 страница)

**Ниша:** `dental_moscow_001` (детская стоматология, Москва, `commercial_local`, `both`).
**Страница:** `/uslugi/lechenie-kariesa-u-detey/` (`page_type = landing`, `target_word_count = 1200`).

```json
{
  "run_id": "8f4a1b2c-...",
  "project_id": "dental_moscow_001",
  "step_id": "12",
  "prompt_version": "step12_v1.0",
  "timestamp": "2026-04-16T14:22:10Z",
  "rag_unavailable": false,
  "benchmark_unavailable": false,
  "pages": [
    {
      "page_url": "/uslugi/lechenie-kariesa-u-detey/",
      "page_cluster": "detskaya_stomatologiya_karies",
      "page_type": "landing",
      "content_blocks": [
        {
          "block_id": "b001",
          "block_type": "h1",
          "content": "Лечение кариеса у детей в Москве — без боли и сверления",
          "parent_heading_id": null,
          "order": 1,
          "word_count": 9,
          "keywords_used": ["лечение кариеса у детей", "Москва", "без сверления"],
          "lsi_terms_used": [],
          "internal_links_suggested": []
        },
        {
          "block_id": "b002",
          "block_type": "paragraph",
          "content": "Ребёнок боится стоматолога? Мы понимаем. В клинике «Денталь» на Соколе детей лечат без бормашины — с помощью серебрения, ICON-инфильтрации и безопасной детской анестезии. Приём ведёт Анна Петрова, детский стоматолог с 12-летним опытом. Первая консультация — бесплатно.",
          "parent_heading_id": null,
          "order": 2,
          "word_count": 38,
          "keywords_used": ["лечение кариеса у детей"],
          "lsi_terms_used": ["серебрение молочных зубов", "ICON технология", "анестезия для детей"],
          "internal_links_suggested": [
            {"anchor": "детский стоматолог", "target_url_pattern": "/vrachi/detskiy-stomatolog/", "context_before": "Приём ведёт Анна Петрова,", "rationale": "Карточка врача — якорь релевантен primary_query кластера"}
          ]
        },
        {
          "block_id": "b003",
          "block_type": "h2",
          "content": "Как мы лечим кариес молочных зубов без бормашины",
          "parent_heading_id": null,
          "order": 3,
          "word_count": 8,
          "keywords_used": ["лечение кариеса молочных зубов"],
          "lsi_terms_used": ["без бормашины"],
          "internal_links_suggested": []
        },
        {
          "block_id": "b004",
          "block_type": "paragraph",
          "content": "Выбор метода зависит от возраста ребёнка и глубины кариеса. Для детей 2–4 лет с поверхностным кариесом мы применяем серебрение: на зуб наносится раствор с ионами серебра, который останавливает разрушение эмали. Процедура длится 5–10 минут, не требует сверления и хорошо переносится даже самыми маленькими пациентами.",
          "parent_heading_id": "b003",
          "order": 4,
          "word_count": 47,
          "keywords_used": [],
          "lsi_terms_used": ["серебрение молочных зубов", "ионы серебра", "без сверления"],
          "internal_links_suggested": [
            {"anchor": "подробнее о серебрении", "target_url_pattern": "/uslugi/serebrenie-molochnyh-zubov/", "context_before": "раствор с ионами серебра, который останавливает разрушение эмали.", "rationale": "Глубокое раскрытие метода — логическое продолжение темы"}
          ]
        },
        {
          "block_id": "b013",
          "block_type": "answer_block",
          "content": "Лечение кариеса у детей в Москве проводится без сверления: для молочных зубов применяется серебрение (возраст 2–4 года) или ICON-инфильтрация; при глубоком кариесе используется щадящая анестезия или седация закисью азота. Стоимость от 2500 ₽ за зуб, запись онлайн за 1 минуту.",
          "parent_heading_id": null,
          "order": 13,
          "word_count": 44,
          "keywords_used": ["лечение кариеса у детей", "Москва"],
          "lsi_terms_used": ["серебрение", "ICON-инфильтрация", "седация закисью азота"],
          "internal_links_suggested": []
        }
      ],
      "full_text_plain": "Лечение кариеса у детей в Москве — без боли и сверления. Ребёнок боится стоматолога? Мы понимаем. ...",
      "word_count_total": 1183,
      "density_metrics": {
        "primary_query_pct": 1.6,
        "primary_query_occurrences": 19,
        "lsi_coverage_pct": 87,
        "stop_words_pct": 37.2,
        "ttr": 0.52
      },
      "readability": {
        "method": "lix",
        "score": 41.3,
        "grade_level": "high",
        "avg_sentence_length_words": 15.4,
        "avg_word_length_chars": 5.8
      },
      "aeo_blocks": {
        "answer_block": {
          "text": "Лечение кариеса у детей в Москве проводится без сверления: для молочных зубов применяется серебрение (возраст 2–4 года) или ICON-инфильтрация; при глубоком кариесе используется щадящая анестезия или седация закисью азота. Стоимость от 2500 ₽ за зуб, запись онлайн за 1 минуту.",
          "word_count": 44,
          "schema_ready": true
        },
        "faq_items": [
          {"question": "С какого возраста можно лечить кариес у ребёнка без сверления?", "answer": "Серебрение применяется с 2 лет, ICON-инфильтрация — с 3 лет при поверхностном кариесе. Решение принимает стоматолог после осмотра.", "answer_word_count": 22, "schema_ready": true, "primary_keyword_in_question": true},
          {"question": "Больно ли ребёнку во время процедуры?", "answer": "Нет. Серебрение и ICON проходят без бормашины и без боли. При необходимости анестезии используется безопасный препарат в детской дозировке.", "answer_word_count": 23, "schema_ready": true, "primary_keyword_in_question": false},
          {"question": "Сколько стоит лечение кариеса у ребёнка в вашей клинике?", "answer": "От 2500 ₽ за серебрение одного зуба, от 4500 ₽ за ICON, от 6000 ₽ за лечение глубокого кариеса с анестезией. Первая консультация бесплатно.", "answer_word_count": 25, "schema_ready": true, "primary_keyword_in_question": true}
        ],
        "how_to_steps": null
      },
      "engine_tags": {
        "yandex_specific": ["от 2500 ₽", "запись онлайн", "м. Сокол", "бесплатная консультация"],
        "google_specific": ["ICON-инфильтрация", "12-летний опыт", "лицензия"],
        "both_used": ["серебрение молочных зубов", "без сверления", "детский стоматолог", "анестезия для детей"]
      },
      "eeat_signals": {
        "has_author_byline": true,
        "author_block": "Анна Петрова, детский стоматолог, 12 лет практики",
        "has_citations": true,
        "citations_count": 2,
        "citations_list": [
          {"claim": "Серебрение рекомендовано Европейской академией детской стоматологии", "source": "requires_seo_review"},
          {"claim": "ICON-инфильтрация одобрена FDA с 2009 года", "source": "requires_seo_review"}
        ],
        "has_license_mention": true,
        "has_date_published": false
      },
      "ai_detection_signals": {
        "formulaic_phrases_count": 1,
        "formulaic_phrases_list": ["является"],
        "sentence_length_variance": 5.2,
        "repetitive_openers_count": 0,
        "burstiness_score": 0.58,
        "humanization_score": 83
      },
      "internal_links_total": 4,
      "external_links_total": 0,
      "source": "generated",
      "target_word_count_hit": true,
      "qc_score_page": 88
    }
  ],
  "summary": {
    "total_pages": 1,
    "avg_word_count": 1183,
    "avg_readability_score": 41.3,
    "avg_humanization_score": 83,
    "avg_primary_density_pct": 1.6,
    "avg_lsi_coverage_pct": 87,
    "pages_with_aeo": 1,
    "pages_with_faq": 1,
    "pages_with_howto": 0,
    "pages_with_eeat_author": 1,
    "total_internal_links_suggested": 4,
    "total_citations": 2,
    "pages_below_target_wc": 0,
    "pages_above_target_wc": 0,
    "submission_queue": ["/uslugi/lechenie-kariesa-u-detey/"]
  },
  "engine_split": {
    "yandex_notes": "Использованы морфологические формы primary (лечение/лечим/лечат); КФ-сигналы (цена от 2500 ₽, запись, м. Сокол). Answer_block ≤60 слов для Яндекс.Нейро.",
    "google_notes": "E-E-A-T: автор с credentials + упоминание лицензии. Entity ICON, FDA. 2 citations для YMYL-достоверности. Helpful Content — глубина раскрытия методов."
  },
  "qc_score": 88,
  "qc_score_per_page": [{"page_url": "/uslugi/lechenie-kariesa-u-detey/", "score": 88}]
}
```

---

## ██ СТРАТЕГИЯ ОТКАТА (Retry Protocol v2)

- **Retry 1** — `qc_errors[]` в промпт, та же стратегия.
- **Retry 2 (Откат A, ДЕКОМПОЗИЦИЯ ПО 1 СТРАНИЦЕ)** — каждая страница отдельным вызовом Opus. Это **стандартный режим** при больших прогонах, не только fallback.
- **Retry 3 (Откат B)** — упрощение: только H1 + intro paragraph + FAQ-блок (≥3). Без content_outline H2-блоков. `fallback_mode=true`.
- **Retry 4 (Откат C)** — генерация только `answer_block + faq_items`, тело дополняет редактор. Помечать `source=manual_edit_needed`.
- **Retry 5 (Откат D / HitL)** — Telegram-алерт с draft-ссылкой; SEO-копирайтер дописывает вручную.

**Защита от бесконечной петли:** `qc_errors_hash` повторен между retry N и N+1 → принудительный переход к следующему откату.

**Специальный откат при низкой humanization:** если `humanization_score < 70` на ≥ 3 страницах подряд — блокировка Шагов 13, 17 до ручной верификации; возможно проблема в модели/промпте.

---

## ██ АДАПТАЦИИ ПОД КЛАСТЕР

| Кластер | Target WC (landing / article) | Тон | Обязательные блоки | Readability (LIX) | Спец. требования |
|---|---|---|---|---|---|
| **commercial_local** | 800-1200 / — | Semiformal, живой | intro, H2: услуги/цены/процесс/отзывы, FAQ≥3, CTA | 35-45 | КФ в каждом H2 (цена, запись, гео); answer_block обязателен |
| **commercial_national** | 1000-1500 / — | Semiformal | intro, H2: USP/преимущества/сравнение, FAQ≥3 | 40-50 | Бренд в intro и conclusion |
| **ecom** | 600-900 / — | Transactional | H2: характеристики, размерная сетка, применение, FAQ≥3 | 30-40 | Schema Product-поля в description-блоках |
| **saas** | 1200-2000 / 2500-4000 | Conversational, expertise | H2: use-cases, integrations, pricing, comparison, FAQ≥4 | 40-55 | Code-примеры допустимы (block_type: code); EN-термины OK |
| **info** | — / 2500-4000 | Informational, deep | H2≥5 (глубокое раскрытие), citations≥3, FAQ≥3, answer_block **обязателен** | 40-55 | E-E-A-T: автор желателен; дата обновления обязательна |
| **medical (YMYL)** | 1500-2500 / 3000-4500 | Formal, expertise | **E-E-A-T обязателен** (author, license, citations≥3), FAQ≥4, disclaimer block | 50-65 | Обязательно: дата, автор-врач с credentials, лицензия, МКБ-коды, НЕТ медицинских советов без «обратитесь к врачу» |
| **legal (YMYL)** | 1800-2500 / 3500-5000 | Formal | **E-E-A-T обязателен** (адвокат с № реестра, citations≥3, статьи закона), FAQ≥4 | 55-70 | Ссылки на НПА, номера статей, обязательный disclaimer |
| **education** | 1000-1500 / 2000-3500 | Conversational, friendly | H2: программа, преподаватели, формат, сертификация, FAQ≥3 | 35-50 | Структурированные списки; фото-suggestions |

**Важно для YMYL:** при невозможности сгенерировать корректный текст без медицинских/юридических советов — fail-open с HitL и передача в редактор-эксперта.

---

## ██ МЕТРИКИ УСПЕХА

**Опережающие (сразу после Шага 12):**
- `qc_score ≥ 80` для ≥ 90% страниц.
- `avg humanization_score ≥ 75`.
- 0 страниц с `forbidden_words_hits > 0`.
- 0 страниц с критической каннибализацией (cosine > 0.92).
- `avg lsi_coverage_pct ≥ 85`.
- `avg primary_density_pct ∈ [1.2, 2.2]`.
- 100% страниц с `target_word_count_hit = true`.
- `avg internal_links_total ≥ 3`.
- YMYL: 100% страниц с `eeat_signals.has_author_byline = true`.

**Запаздывающие (30–60 дней после публикации):**
- Средняя позиция страниц ↑ на ≥ 5 пунктов в выдаче.
- Dwell-time ≥ 90 секунд на article, ≥ 45 на landing/category.
- Bounce-rate < 60% (commercial_local < 50%).
- Появление FAQ-rich-snippets в Google / FAQ-turbo в Яндексе ≥ 30% страниц.
- AI-citation (Шаг 40.3) по primary_query ≥ 35% страниц с `aeo_blocks`.
- Organic impressions в GSC/Вебмастер ≥ +25% за 60 дней.
- Conversion-rate (для commercial) ≥ +10%.

**Контрольные:**
- Спот-чек 10% страниц вручную: текст звучит «человечно», без AI-паттернов.
- Сравнение с golden set: отклонение `humanization_score` от эталона > 15 → тревога о деградации модели.
- A/B-эксперимент (Шаг 40): CTR новых текстов vs baseline через 30 дней.
- Шаг 34 (Content Decay): через 90 дней — отсутствие резкого падения (> 20%) органики как индикатор качества базового текста.

**Бюджет токенов:** 8–15K вход + 6–8K выход **на батч из 1–3 страниц**. Итого: ~3 вызовов на пакет из 10 страниц.
**Cost-budget:** ≤ $15 на `run_id` (при 10 страницах — средняя стоимость $8-12).

---

## ██ RUNTIME-КОНТРАКТЫ И НАБЛЮДАЕМОСТЬ

- **`run_id`** сквозной (Шаги 00 → 40.3).
- **Grafana-метрики (20):** `seo_pages_generated`, `avg_word_count`, `avg_qc_score`, `avg_humanization_score`, `avg_readability_score`, `avg_lsi_coverage_pct`, `avg_primary_density_pct`, `retry_count`, `duration_ms`, `rag_unavailable`, `benchmark_unavailable`, `fallback_mode`, `cannibalization_blocks`, `cannibalization_warns`, `hitl_required`, `hitl_timeout`, `forbidden_words_hits`, `total_internal_links_suggested`, `total_citations`, `opus_cost_usd`.
- **Алерты:**
  - QC fail 4x → Telegram.
  - SLA > 30 мин → Telegram.
  - `humanization_score < 70` на ≥ 3 страницах подряд → Slack (возможная деградация модели).
  - `cost-budget > $15` → Telegram + circuit-break.
  - YMYL без E-E-A-T → Telegram + блок Шагов 13, 17.
- **DLQ:** `seo_seo_text_dlq` с полным `opus_raw_response` (до 100KB).
- **Checkpoints:** `prompt_response.json` + `validation_report.json` в `seo_step_checkpoints` (TTL 7 дней) — per batch для resume.
- **A/B-трекинг:** Шаг 40 читает `content_blocks` и `aeo_blocks`, 30-дневный эксперимент (публикация vs старая версия через feature flag в CMS).

---

## ██ ИСТОРИЯ ИЗМЕНЕНИЙ

| Версия | Дата | Изменения |
|---|---|---|
| v1.0 | 2026-04-16 | Первая редакция. Template 3 расширен для длинноформатного контента + AEO + E-E-A-T + внутренняя перелинковка + AI-detection. **Креативный режим Opus** (temp 0.4, top_p 0.95, max_tokens 8192) — усиленный антигалл-протокол с whitelist power-words и FORBIDDEN_WORDS. **Жёсткая декомпозиция** — max 3 страницы на вызов (не 15 как в Шаге 11) из-за длинного выхода. **41 QC-проверка** (структура, длина, плотность, читабельность, AEO, перелинковка, E-E-A-T, антигалл, AI-detection, уникальность, запреты). **AI-humanization QC** — formulaic_phrases (запрет 22 оборотов), burstiness, TTR, repetitive_openers, humanization_score ≥ 70. **E-E-A-T для YMYL** — author_byline + license + citations ≥ 2 обязательны. **4-й namespace Qdrant** (`content_chunks:body`) для уникальности. **Условный HitL** для YMYL, низкой humanization, forbidden_words. **Cost-budget $15/run_id** (больше, чем Шаг 11 из-за длинных выходов). **Адаптации под 8 кластеров** с target_word_count, тоном, обязательными блоками, readability-нормой (LIX). **Retry Protocol v2** с декомпозицией по 1 странице как стандарт. **Few-shot** на dental_moscow_001 с полным JSON-выходом. **Интеграция** с Шагами 13 (Schema+OG), 17 (Финальный аудит), 29 (AEO, async 24ч), 34 (Content Decay, async 90дн). |

---

**Контрольный чек-лист соответствия CORE.md + STEP_00-эталону + v1.1-стандарту (Шагов 10, 11):**

- [x] Основная роль F + консультативные A/E/G (из карты 63 шагов)
- [x] Тип «Промпт»: Pre-step агрегация + post-step валидация (4 скрипта)
- [x] **Opus mode Креативный** (0.4/0.95/8192) — Шаг 12 явно в списке из CORE.md
- [x] `[Я]/[G]/[ОБА]` обязателен + согласованность с `priority_engines`
- [x] Русский язык в текстовых полях; английские ключи/enum
- [x] QC rule-based (41 проверка, ML отключён)
- [x] Retry Protocol v2 + защита от петли (унаследовано из Шага 11)
- [x] JSON-контракт = Template 3 + расширения для длинноформатного контента
- [x] Input contract = выход Шага 11 v1.1 (`contract_for_step_12`) + агрегация 09, 10
- [x] Output → Шаги 13, 17, 29, 34 с per-consumer контрактами
- [x] Few-shot реалистичен (детская стоматология, Москва)
- [x] **Антигаллюцинационный протокол усилен** для креативного режима (FORBIDDEN_WORDS + AI_PATTERNS)
- [x] Опережающие + запаздывающие + контрольные метрики
- [x] **Условный HitL** для YMYL, humanization < 70, forbidden_words > 0, qc 60-79
- [x] Наблюдаемость: run_id, checkpoints, DLQ, 20 Grafana-метрик
- [x] Лимит токенов: ≤ 3 страницы/вызов (декомпозиция — стандарт)
- [x] Адаптации под 8 кластеров с target_word_count, тоном, readability
- [x] **E-E-A-T для YMYL** обязательный (author, license, citations ≥ 2)
- [x] **AI-humanization QC** — новый класс проверок (formulaic, burstiness, TTR, humanization_score)
- [x] **Внутренняя перелинковка** с антигалл-ограничением (только из `related_pages_for_linking`)
- [x] **4-й namespace Qdrant** (`content_chunks:body`) для антиканнибализации тела
- [x] Cost-budget $15/run_id с circuit-breaker (больше Шага 11)
