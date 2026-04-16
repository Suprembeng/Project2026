# МОДУЛЬ 3: STEP_12.2.md — Video SEO

**SEO AUTOMATION ENGINEERING · v6.0 · КОНФИДЕНЦИАЛЬНО**

Самодостаточный модуль. Загружается: `CORE.md` + `PROJECT.md` + этот файл. ~4K токенов.

Наследует: **Шаблон 3 (Генерация контента)**. Primary Role: **A + B**. Тип: **Промпт+Скрипт**.

---

## ██ КАРТОЧКА ШАГА

| **Параметр**          | **Значение**                                                                                                                                                                                          |
|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ID                    | 12.2                                                                                                                                                                                                  |
| Название              | Video SEO (стратегия + метаданные + Schema + Sitemap)                                                                                                                                                 |
| Блок                  | 3: Контент и оптимизация                                                                                                                                                                              |
| Основная роль         | A: SEO-стратег + B: Инженер автоматизации (со-ведущие)                                                                                                                                                |
| Консультативные       | F: Промпт-инженер, D: Аналитик конкур. разведки, E: CRO-аналитик                                                                                                                                      |
| Тип                   | Промпт+Скрипт (скрипты собирают данные → Opus генерирует стратегию и метаданные → скрипты валидируют и инъектят Schema/Sitemap)                                                                       |
| HitL                  | Нет (результат → Шагу 13 автоматически)                                                                                                                                                               |
| Приоритет ROI         | ВЫСОКИЙ — видео даёт +41% CTR в Я.Видео-блоках, +видео-цитаты в Яндекс Нейро и Google AI Mode                                                                                                         |
| Зависит от            | `PROJECT.md`, Шаг 04 (структура страниц), Шаг 12 (SEO-текст — контент-блоки страниц), Шаг 12.1 (Image SEO — для консистентности медиа)                                                                |
| Передаёт в            | Шаг 12.3 (UGC), Шаг 13 (Schema.org + OG), Шаг 29 (AEO), Шаг 30 (GEO), Шаг 40.3 (AI-цитирования)                                                                                                       |
| Режим Opus            | Генерация контента: temperature 0.25, top_p 0.9, max_tokens 8192 (компромисс между креативом заголовков и точностью JSON)                                                                             |
| Наследуемый шаблон    | Шаблон 3 (Генерация контента) — расширенный полями `videos[]`, `video_strategy`, `video_sitemap_entries[]`                                                                                            |

---

## ██ ЦЕЛЬ

Для каждой страницы из контент-плана сгенерировать полную стратегию Video SEO: подбор формата видео и его роли на странице, SEO-оптимизированные метаданные (title, description, tags, thumbnail-концепция) под площадки **YouTube [G]** и **Яндекс.Видео / Rutube / VK Видео [Я]**, тайм-коды глав, бриф транскрипта, разметка `VideoObject` и записи для `video-sitemap.xml`, с учётом интента SERP, кластера ниши и приоритетных поисковиков проекта.

---

## ██ КОНТРАКТ ВХОДНЫХ ДАННЫХ (JSON)

Вход наследуется от **output Шага 12 (SEO-текст, Шаблон 3)** + обогащение от `PROJECT.md` и пре-скриптов. Минимальные обязательные поля:

```json
{
  "project_id": "string",
  "project_context": {
    "niche": "string",
    "niche_cluster": "commercial_local|commercial_national|ecom|saas|info|medical|legal|education",
    "geo": "string",
    "language": "string",
    "priority_engines": "yandex|google|both",
    "domain": "string",
    "main_competitors": ["string"]
  },
  "pages_to_process": [
    {
      "page_url": "string",
      "content_type": "landing|category|article|faq|product",
      "content_blocks": [
        {"block_type": "h1|h2|paragraph|faq|answer_block", "content": "string", "keywords": ["string"], "word_count": "number"}
      ],
      "target_queries": [{"query": "string", "engine": "yandex|google|both", "intent": "informational|commercial|navigational|transactional"}],
      "page_meta": {"title": "string", "description": "string"},
      "aeo_elements": {"has_how_to": "boolean", "faq_count": "number"}
    }
  ],
  "pre_script_data": {
    "existing_videos": [{"page_url": "string", "platform": "string", "video_id": "string", "duration_sec": "number"}],
    "competitor_videos": [{"query": "string", "top_videos": [{"title": "string", "platform": "string", "duration_sec": "number", "views": "number"}]}],
    "serp_video_intent": [{"query": "string", "engine": "yandex|google", "video_serp_feature": "video_carousel|featured_video|none", "top_domains": ["string"]}]
  }
}
```

---

## ██ КОНТРАКТ ВЫХОДНЫХ ДАННЫХ (JSON)

Роль G валидирует. Должен пройти структурный + содержательный QC. Наследует **Шаблон 3** с расширением для видео.

```json
{
  "project_id": "string",
  "step_id": "12.2",
  "prompt_version": "string",
  "timestamp": "ISO 8601",
  "pages": [
    {
      "page_url": "string",
      "content_type": "landing|category|article|faq|product",
      "video_strategy": {
        "video_count": "number (1-3)",
        "primary_role": "hero|explainer|how_to|product_demo|testimonial|expert_interview|comparison",
        "placement": "above_fold|mid_content|faq|sidebar|end_of_article",
        "target_duration_sec": "number (30-900)",
        "hosting_priority": ["youtube", "yandex_video", "rutube", "vk_video", "native_embed"],
        "rationale": "string — почему такая стратегия для этой страницы и кластера"
      },
      "videos": [
        {
          "video_id": "string (V001, V002...)",
          "role": "hero|explainer|how_to|product_demo|testimonial|expert_interview|comparison",
          "target_intent": "informational|commercial|navigational|transactional",
          "target_queries": [{"query": "string", "engine": "yandex|google|both"}],
          "engine_tag": "[Я]|[G]|[ОБА]",
          "metadata": {
            "youtube": {
              "title": "string (≤70 симв., ключ в первых 50)",
              "description_hook": "string (первые 150 симв. — критично для сниппета и AI)",
              "description_full": "string (до 5000 симв., с тайм-кодами и ссылками)",
              "tags": ["string (суммарно ≤500 симв.)"],
              "category": "string (YouTube category ID)"
            },
            "yandex_video": {
              "title": "string (≤60 симв.)",
              "description": "string (до 1500 симв.)",
              "tags": ["string"],
              "rubric": "string (рубрика Я.Видео)"
            }
          },
          "thumbnail_concept": {
            "composition": "string — описание кадра",
            "text_overlay": "string (≤4 слов, крупный шрифт)",
            "emotion_hook": "string",
            "specs": "1280×720, <2MB, JPG/PNG, контраст WCAG AA"
          },
          "chapters": [
            {"time_sec": "number", "title": "string (≤40 симв.)"}
          ],
          "transcript_brief": {
            "target_word_count": "number",
            "hook_0_15_sec": "string — что говорим в первые 15 секунд",
            "key_phrases_must_include": ["string"],
            "cta_final_15_sec": "string",
            "language": "string"
          },
          "schema_markup": {
            "@context": "https://schema.org",
            "@type": "VideoObject",
            "name": "string",
            "description": "string",
            "thumbnailUrl": ["string"],
            "uploadDate": "ISO 8601",
            "duration": "ISO 8601 duration (PT...)",
            "contentUrl": "string|null",
            "embedUrl": "string|null",
            "hasPart": [{"@type": "Clip", "name": "string", "startOffset": "number", "endOffset": "number", "url": "string"}]
          },
          "engagement_elements": {
            "end_screen_cta": "string",
            "cards_timings_sec": ["number"],
            "pinned_comment": "string|null",
            "playlist_association": "string|null"
          }
        }
      ],
      "meta": {
        "og_video": "string|null",
        "twitter_player": "string|null",
        "page_schema_graph_additions": ["VideoObject"]
      },
      "aeo_elements": {
        "has_clip_markup": "boolean",
        "has_chapters": "boolean",
        "transcript_published_on_page": "boolean",
        "ai_citation_optimization_notes": "string"
      },
      "engine_tags": {
        "yandex_specific": ["string (что делаем под Я.Видео/Нейро/Дзен)"],
        "google_specific": ["string (что делаем под YouTube/SGE/Shorts)"]
      }
    }
  ],
  "video_sitemap_entries": [
    {
      "loc": "string (URL страницы)",
      "video_loc": "string (прямая ссылка на видео или embed)",
      "thumbnail_loc": "string",
      "title": "string (≤100 симв.)",
      "description": "string (≤2048 симв.)",
      "duration_sec": "number",
      "publication_date": "ISO 8601",
      "family_friendly": "yes|no"
    }
  ],
  "qc_score": "number 0-100"
}
```

---

## ██ ПРОМПТ ДЛЯ CLAUDE OPUS

### Системный промпт

```
Ты — со-ведущие Роли A (Senior SEO-стратег, 10+ лет) + B (Senior Инженер автоматизации).
Консультанты: F (Промпт-инженер), D (Аналитик конкурентной разведки), E (CRO-аналитик).

Задача: Сгенерировать полную стратегию Video SEO для набора страниц проекта.

ПРАВИЛА:
1. Для каждой страницы определи, нужно ли видео, и если да — его роль (hero/explainer/how_to/product_demo/testimonial/expert_interview/comparison). Пустой массив videos[] допустим, если видео неуместно — но ты ДОЛЖЕН обосновать это в video_strategy.rationale.
2. Количество видео на странице: 1 (по умолчанию), 2-3 только для hub-страниц и длинных статей (>3000 слов).
3. Для КАЖДОГО видео ОБЯЗАТЕЛЬНО:
   - metadata под youtube И yandex_video (если priority_engines=both); под одну — если yandex или google
   - минимум 3 главы, если target_duration_sec > 180
   - thumbnail_concept с text_overlay ≤ 4 слов
   - transcript_brief с hook_0_15_sec (первые 15 секунд — критичны для retention и AI-цитирования)
   - schema_markup VideoObject с валидными полями, hasPart[] = chapters
4. Engine tags ОБЯЗАТЕЛЬНЫ:
   - [Я] — Яндекс.Видео, Rutube, VK Видео, Дзен, Яндекс Нейро (AI-ответы с видео-цитатами), Я.Бизнес-ролики
   - [G] — YouTube (первичная площадка), YouTube Shorts (для query_intent=informational с duration ≤60 сек), Google Video, SGE/AI Mode видео-карточки
   - [ОБА] — VideoObject schema, транскрипт на странице, video-sitemap, структура chapters
5. Длина метаданных:
   - YouTube title ≤ 70 симв., ключ в первых 50
   - YouTube description_hook (первые 150 симв.) = ответ на target_query + value proposition
   - Я.Видео title ≤ 60 симв., description ≤ 1500 симв.
   - YouTube tags ≤ 500 симв. суммарно
6. Адаптация под niche_cluster:
   - commercial_local: видео-визитка клиники/салона, локальные кейсы, отзывы клиентов с гео, 30-90 сек
   - commercial_national: бренд-сторителлинг, экспертные интервью, 2-5 мин
   - ecom: распаковка, how-to-use, сравнение товаров, 60-180 сек
   - saas: walkthrough, feature demo, webinar-нарезка, 2-8 мин
   - medical (YMYL): экспертные интервью только с врачами с указанием ФИО, должности, стажа; дисклеймеры в description; НИКОГДА не давай мед. рекомендации в hook
   - legal (YMYL): аналогично medical — эксперт с указанием статуса адвоката/юриста
   - info: глубокие объяснения, нарративные форматы, 3-10 мин
   - education: пошаговые уроки, учебные плейлисты, 5-15 мин
7. AEO/GEO оптимизация:
   - hook_0_15_sec должен содержать прямой ответ на информационный query (для попадания в Яндекс Нейро и Google AI Mode)
   - chapters[] обязательны (Clip markup → видео-фрагменты в SERP)
   - transcript_brief.key_phrases_must_include = ключи из target_queries
   - transcript публикуется на странице как текст (для индексации)
8. Интент SERP (из pre_script_data.serp_video_intent):
   - video_serp_feature=video_carousel → ОБЯЗАТЕЛЬНО генерируем видео
   - video_serp_feature=featured_video → видео hero-формата
   - video_serp_feature=none и content_type=faq/article → видео опционально
9. Конкуренты (из pre_script_data.competitor_videos):
   - Длительность нашего видео = медиана конкурентов ±30%
   - НЕ копируй заголовки конкурентов. Используй дельту (что они упустили).
10. hosting_priority зависит от priority_engines:
    - yandex → [yandex_video, rutube, vk_video, youtube, native_embed]
    - google → [youtube, native_embed, yandex_video, rutube]
    - both → [youtube, yandex_video, rutube, vk_video, native_embed]
11. Для каждого target_query пометь engine_tag на уровне видео.
12. video_sitemap_entries[] собирай по ВСЕМ видео со всех страниц.

ВЫВОД: Валидный JSON строго по контракту выхода. Без markdown-обёртки. Без комментариев.
Весь человекочитаемый текст (заголовки, описания, главы, rationale) — на РУССКОМ языке.
JSON-ключи — на английском согласно контракту.
```

### Пользовательский промпт (шаблон)

```
Сгенерируй стратегию Video SEO для проекта.

Проект: {{project.name}}
Ниша: {{project.niche}}
Кластер: {{project.niche_cluster}}
Гео: {{project.geo}}
Язык: {{project.language}}
Приоритет ПС: {{project.priority_engines}}
Домен: {{project.domain}}
Конкуренты: {{project.main_competitors | join(', ')}}

Страницы для обработки ({{pages_to_process.length}} шт.):
{{#each pages_to_process}}
---
URL: {{this.page_url}}
Тип: {{this.content_type}}
Title: {{this.page_meta.title}}
Целевые запросы: {{this.target_queries | json}}
Есть how-to блок: {{this.aeo_elements.has_how_to}}
FAQ на странице: {{this.aeo_elements.faq_count}}
Ключевые H2: {{this.content_blocks | filter(b => b.block_type=='h2') | map('content') | join(' | ')}}
{{/each}}

{{#if pre_script_data.existing_videos}}
Существующие видео на сайте:
{{pre_script_data.existing_videos | json}}
{{/if}}

{{#if pre_script_data.competitor_videos}}
Видео конкурентов по целевым запросам:
{{pre_script_data.competitor_videos | json}}
{{/if}}

{{#if pre_script_data.serp_video_intent}}
Видео-интент в SERP (DataForSEO):
{{pre_script_data.serp_video_intent | json}}
{{/if}}

{{#if rag_context}}
Контекст из RAG (паттерны видео-SEO в похожих нишах):
{{rag_context}}
{{/if}}
```

---

## ██ RAG-ОБОГАЩЕНИЕ

Лимит RAG-контекста: **2000 токенов**. Запросы к Qdrant перед вызовом Opus:

| Коллекция          | Запрос (эмбеддинг)                                       | Что достаём                                                                              |
|--------------------|----------------------------------------------------------|------------------------------------------------------------------------------------------|
| `video_patterns`   | `niche_cluster` + `geo` + `language`                     | Шаблоны ролей видео (hero/explainer/how-to), медианные длительности по нише              |
| `competitors`      | `niche` + `main_competitors` → фильтр `has_video=true`   | Структура видео-контента лидеров: длительность, главы, частота публикаций, площадки     |
| `personas`         | `project_id` (из Шага 00) → `content_preferences.video`  | Предпочтительные форматы и длительности для персон проекта                               |
| `search_patterns`  | `geo` + `priority_engines`                               | Доля видео-интентов в SERP по региону (Я.Видео vs YouTube для RU-гео)                    |

Если Qdrant недоступен — работаем без RAG с предупреждением в лог, fallback на few-shot из этого файла.

---

## ██ СКРИПТЫ CLAUDE CODE

### Пре-скрипт 1: Аудит существующих видео

```python
# step_12_2_pre_video_audit.py
# Назначение: Инвентаризация видео на сайте и их SEO-гигиены
# Вход: project.domain, pages_to_process[].page_url
# Выход: existing_videos[] → в pre_script_data
# Действия:
# 1. Crawl страниц из pages_to_process (httpx + selectolax)
# 2. Извлечь: <iframe src=youtube/yandex/rutube/vk>, <video>, JSON-LD VideoObject
# 3. Для каждого видео: platform, video_id, embed_url, thumbnail, duration (если в JSON-LD),
#    наличие schema.org VideoObject, наличие transcript на странице, наличие chapters в description
# 4. Пометить "has_seo_issues": отсутствие schema, отсутствие chapters, устаревшие видео
# 5. Сохранить в /tmp/pre_script_12_2_audit.json
```

### Пре-скрипт 2: Видео конкурентов

```python
# step_12_2_pre_competitor_videos.py
# Назначение: Топ-видео конкурентов по целевым запросам (YouTube Data API + Я.Видео SERP)
# Вход: target_queries[], priority_engines, geo
# Выход: competitor_videos[]
# Действия:
# 1. Для каждого target_query с engine ∈ {google, both}:
#    - YouTube Data API v3: search.list(q=query, regionCode=geo, maxResults=5)
#    - Для каждого результата: videos.list(id, part=contentDetails,statistics,snippet) →
#      title, duration (ISO 8601 → sec), views, likes, channel_authority_score
# 2. Для каждого target_query с engine ∈ {yandex, both}:
#    - DataForSEO SERP API (yandex_video_organic) или парсинг Я.Видео SERP
#    - Извлечь: title, platform (youtube/rutube/dzen/vk), duration, video_url
# 3. Агрегировать: median_duration, title_patterns, hosting_distribution
# 4. Сохранить в /tmp/pre_script_12_2_competitors.json
```

### Пре-скрипт 3: SERP video-intent

```python
# step_12_2_pre_serp_video_intent.py
# Назначение: Определить, какие запросы требуют видео-ответа
# Вход: target_queries[], priority_engines, geo
# Выход: serp_video_intent[]
# Действия:
# 1. DataForSEO SERP API: live_advanced (google) + live (yandex) для каждого запроса
# 2. Извлечь SERP features: video_carousel, featured_video, shorts_carousel, answer_box_with_video
# 3. Классификация интента: video_mandatory | video_optional | video_not_relevant
# 4. Сохранить в /tmp/pre_script_12_2_serp_intent.json
# 5. При rate-limit 429 → экспоненциальный бэкофф 10с → 30с → 90с, max 3 попытки
```

### Пост-скрипт 1: Валидатор

```python
# step_12_2_post_validate.py
# Назначение: Валидация выхода Opus по JSON-схеме + бизнес-правилам
# Вход: opus_output.json
# Выход: {"qc_passed": bool, "qc_score": 0-100, "issues": [...]}
# Проверки:
# 1. JSON Schema (ajv-совместимая) по контракту выхода
# 2. Для каждого videos[]:
#    - metadata.youtube.title ≤ 70 симв., ключ из target_queries в первых 50 симв.
#    - metadata.youtube.description_hook ≤ 150 симв.
#    - если duration > 180 → len(chapters) ≥ 3; иначе ≥ 0
#    - chapters[0].time_sec == 0 (первая глава с 0:00)
#    - schema_markup.VideoObject: name, description, thumbnailUrl, uploadDate, duration обязательны
#    - engine_tag ∈ {[Я], [G], [ОБА]}
#    - если priority_engines=both → и [Я] и [G] должны встречаться среди videos или engine_tags
# 3. transcript_brief.key_phrases_must_include ⊇ минимум 1 ключ из target_queries
# 4. thumbnail_concept.text_overlay ≤ 4 слов (split(' '))
# 5. video_sitemap_entries[] содержит запись для каждого видео
# 6. engine_tags.yandex_specific непусто при priority_engines ∈ {yandex, both}
# 7. engine_tags.google_specific непусто при priority_engines ∈ {google, both}
# 8. QC-score по весам (см. раздел QC CRITERIA)
```

### Пост-скрипт 2: Инъекция Schema + Video Sitemap

```python
# step_12_2_post_schema_inject.py
# Назначение: Генерация финальных артефактов для интеграции на сайт
# Вход: валидированный opus_output
# Выход:
#   - /output/schema_video_by_page.json — VideoObject на каждую страницу (для Шага 13)
#   - /output/video-sitemap.xml — готовый sitemap
#   - /output/transcript_briefs.json — брифы для заказа транскриптов
# Действия:
# 1. Для каждой страницы собрать VideoObject (если 1 видео) или @graph (если >1)
# 2. Собрать video-sitemap.xml по спецификации Google (schemas.google.com/sitemap/video)
#    и Яндекса (учёт video:restriction, video:family_friendly)
# 3. Сохранить NocoDB: таблица `video_seo_plans` (по строке на каждое видео)
# 4. Upsert в Qdrant коллекцию `video_patterns` для RAG следующих проектов
# 5. Отправить событие в Grafana: step_12_2_videos_generated, count
```

---

## ██ КРИТЕРИИ QC

Rule-based. ML отключён до ≥200 размеченных образцов (политика CORE.md).

| **Проверка**                                                          | **Порог**                                  | **Вес** | **При провале** |
|-----------------------------------------------------------------------|--------------------------------------------|---------|-----------------|
| JSON Schema валиден                                                   | Pass                                       | 15      | Retry (gate)    |
| YouTube title длина и позиция ключа                                   | ≤70 симв., ключ в первых 50                | 10      | Retry           |
| YouTube description_hook                                              | ≤150 симв., содержит target_query          | 10      | Retry           |
| Я.Видео title                                                         | ≤60 симв. (если priority_engines∈yandex,both) | 8     | Retry           |
| Наличие глав при duration > 180                                       | ≥3 глав, chapters[0].time_sec==0           | 10      | Retry           |
| Schema.org VideoObject                                                | Обязательные поля + hasPart для chapters   | 10      | Retry           |
| Engine tags [Я] и [G] при priority_engines=both                       | Оба встречаются                            | 8       | Retry           |
| Thumbnail text_overlay                                                | ≤4 слов                                    | 5       | Ревью           |
| Transcript brief key_phrases                                          | ≥1 ключ из target_queries                  | 7       | Retry           |
| Video sitemap entry для каждого видео                                 | 100% покрытие                              | 7       | Retry           |
| Соответствие video_serp_feature → наличию видео                       | video_carousel/featured_video → videos[]≠∅ | 5       | Ревью           |
| Кластерная адаптация (YMYL: эксперт+дисклеймер для medical/legal)     | 100% при соотв. кластере                   | 5       | Retry           |

**Скоринг:** сумма весов совпавших проверок. **≥80 — готово, 60–79 — ревью, <60 — retry** (согласно CORE.md QC v2).

---

## ██ СТРУКТУРА N8N-ВОРКФЛОУ

| Нода | Действие                                                                                         |
|------|--------------------------------------------------------------------------------------------------|
| 1    | Триггер: webhook от Шага 12 (завершение SEO-текста)                                             |
| 2    | NocoDB: загрузка `PROJECT.md`                                                                    |
| 3    | NocoDB: загрузка output Шага 12 (content_blocks + page_meta по страницам)                       |
| 4    | Execute `step_12_2_pre_video_audit.py`                                                           |
| 5    | Execute `step_12_2_pre_competitor_videos.py` (параллельно с 4)                                   |
| 6    | Execute `step_12_2_pre_serp_video_intent.py` (параллельно с 4, 5)                                |
| 7    | Qdrant RAG: запрос к 4 коллекциям (см. RAG-ОБОГАЩЕНИЕ)                                           |
| 8    | Сборка промпта: CORE.md + PROJECT.md + STEP_12.2.md + input + RAG                                |
| 9    | Claude Opus API (temperature 0.25, top_p 0.9, max_tokens 8192)                                   |
| 10   | Парсинг JSON (удалить markdown-обёртку если попала)                                              |
| 11   | Execute `step_12_2_post_validate.py` → при QC<60: вернуться к ноде 8 с ошибками (макс. 3 retry) |
| 12   | Execute `step_12_2_post_schema_inject.py`                                                        |
| 13   | NocoDB: upsert в `video_seo_plans`                                                               |
| 14   | Qdrant: upsert в `video_patterns`                                                                |
| 15   | Grafana: метрики (videos_generated, qc_score, retry_count)                                       |
| 16   | Триггер Шага 12.3 (UGC) и Шага 13 (Schema.org + OG), либо алерт Telegram при 3x fail             |

**Обработка ошибок:**

- **Opus 429/500** → бэкофф 10с → 30с → 90с, max 3 попытки.
- **YouTube Data API quota exceeded** → fallback на DataForSEO YouTube SERP; пометка `data_source: secondary` в логе.
- **DataForSEO недоступен** → работаем без `serp_video_intent`, QC снижает вес правила #11 до 0.
- **Qdrant недоступен** → без RAG, предупреждение в лог.
- **QC fail 3x** → алерт Telegram с логом 3 попыток и передачей на HitL-ревью.

---

## ██ ПРИМЕР FEW-SHOT

Сокращённый пример для страницы стоматологической клиники в Москве (niche=dental, cluster=medical, geo=Москва, engines=both). Ниша YMYL — обязательны эксперт и дисклеймер.

```json
{
  "project_id": "dental_moscow_001",
  "step_id": "12.2",
  "prompt_version": "step12_2_v1.0",
  "timestamp": "2026-01-15T10:30:00Z",
  "pages": [
    {
      "page_url": "https://dentclinic.ru/services/implantaciya/",
      "content_type": "landing",
      "video_strategy": {
        "video_count": 2,
        "primary_role": "expert_interview",
        "placement": "above_fold",
        "target_duration_sec": 180,
        "hosting_priority": ["youtube", "yandex_video", "rutube", "vk_video", "native_embed"],
        "rationale": "Медицинская ниша YMYL требует E-E-A-T: экспертное интервью с хирургом-имплантологом (ФИО, стаж) + демо-процедуры усиливают доверие. Конкуренты (topdent.ru, stom-clinic.ru) размещают hero-видео на посадочных имплантации; медиана длительности 2:30-3:30. SERP Я+G по 'имплантация зубов москва' содержит video_carousel — видео обязательно."
      },
      "videos": [
        {
          "video_id": "V001",
          "role": "expert_interview",
          "target_intent": "commercial",
          "target_queries": [
            {"query": "имплантация зубов москва", "engine": "both"},
            {"query": "установка импланта под ключ", "engine": "yandex"}
          ],
          "engine_tag": "[ОБА]",
          "metadata": {
            "youtube": {
              "title": "Имплантация зубов в Москве: отвечает хирург с 15-летним стажем",
              "description_hook": "Хирург-имплантолог Иванов А.П. (стаж 15 лет) отвечает на 7 главных вопросов пациентов об имплантации в 2026 году.",
              "description_full": "В видео: 0:00 Когда нужна имплантация 0:45 Виды имплантов (Straumann, Nobel, Osstem) 2:10 Противопоказания 3:20 Сроки и этапы 4:40 Цены в Москве 2026. \n\n⚠️ Видео носит информационный характер. Требуется очная консультация. Лицензия Л041-01137-77/00123456. \n\nЗапись: +7 (495) XXX-XX-XX | https://dentclinic.ru",
              "tags": ["имплантация зубов", "имплантация москва", "установка импланта", "хирург-имплантолог", "стоматология москва", "dentclinic"],
              "category": "27"
            },
            "yandex_video": {
              "title": "Имплантация зубов Москва: хирург отвечает",
              "description": "Хирург-имплантолог (стаж 15 лет) отвечает на 7 вопросов: виды имплантов, противопоказания, сроки, цены. Лицензия Л041-01137-77/00123456. Требуется очная консультация.",
              "tags": ["имплантация", "стоматология москва", "имплантолог"],
              "rubric": "Здоровье и медицина"
            }
          },
          "thumbnail_concept": {
            "composition": "Врач в белом халате, сидит, смотрит в камеру, за спиной — сертификаты",
            "text_overlay": "ИМПЛАНТАЦИЯ: 7 ВОПРОСОВ",
            "emotion_hook": "спокойная экспертность, доверие",
            "specs": "1280×720, <2MB, JPG, контраст WCAG AA"
          },
          "chapters": [
            {"time_sec": 0, "title": "Когда нужна имплантация"},
            {"time_sec": 45, "title": "Виды имплантов"},
            {"time_sec": 130, "title": "Противопоказания"},
            {"time_sec": 200, "title": "Сроки и этапы"},
            {"time_sec": 280, "title": "Цены в Москве 2026"}
          ],
          "transcript_brief": {
            "target_word_count": 550,
            "hook_0_15_sec": "Имплантация зубов в Москве стоит от 45 000 ₽ под ключ и занимает 3-6 месяцев. Я хирург-имплантолог Иванов, 15 лет практики. За 3 минуты отвечу на 7 главных вопросов.",
            "key_phrases_must_include": ["имплантация зубов", "имплантация москва", "имплантат", "хирург-имплантолог"],
            "cta_final_15_sec": "Запишитесь на бесплатную консультацию по ссылке в описании. Видео не заменяет очный приём.",
            "language": "ru"
          },
          "schema_markup": {
            "@context": "https://schema.org",
            "@type": "VideoObject",
            "name": "Имплантация зубов в Москве: отвечает хирург с 15-летним стажем",
            "description": "Хирург-имплантолог Иванов А.П. отвечает на 7 главных вопросов пациентов об имплантации.",
            "thumbnailUrl": ["https://dentclinic.ru/media/video/v001-thumb-1280.jpg"],
            "uploadDate": "2026-01-20T09:00:00+03:00",
            "duration": "PT5M20S",
            "contentUrl": "https://www.youtube.com/watch?v=XXXX",
            "embedUrl": "https://www.youtube.com/embed/XXXX",
            "hasPart": [
              {"@type": "Clip", "name": "Когда нужна имплантация", "startOffset": 0, "endOffset": 45, "url": "https://www.youtube.com/watch?v=XXXX&t=0s"},
              {"@type": "Clip", "name": "Виды имплантов", "startOffset": 45, "endOffset": 130, "url": "https://www.youtube.com/watch?v=XXXX&t=45s"}
            ]
          },
          "engagement_elements": {
            "end_screen_cta": "Запись на консультацию + плейлист 'Всё об имплантации'",
            "cards_timings_sec": [60, 180],
            "pinned_comment": "⚠️ Информационное видео. Требуется очная консультация. Лицензия Л041-01137-77/00123456.",
            "playlist_association": "Имплантация зубов — полный гид"
          }
        }
      ],
      "meta": {
        "og_video": "https://www.youtube.com/embed/XXXX",
        "twitter_player": "https://www.youtube.com/embed/XXXX",
        "page_schema_graph_additions": ["VideoObject"]
      },
      "aeo_elements": {
        "has_clip_markup": true,
        "has_chapters": true,
        "transcript_published_on_page": true,
        "ai_citation_optimization_notes": "Hook 0-15 сек содержит прямой ответ с цифрой и сроком — оптимизировано под Яндекс Нейро и Google AI Mode. Транскрипт опубликован на странице под видео для индексации."
      },
      "engine_tags": {
        "yandex_specific": ["Загрузка в Я.Видео с привязкой к Я.Бизнес-карточке клиники", "Рубрика 'Здоровье и медицина'", "Копия в Rutube и VK Видео для гео-кластера"],
        "google_specific": ["YouTube — первичная публикация", "Clip markup для Key Moments в Google SERP", "Возможность YouTube Shorts 45-секундной нарезки для 'виды имплантов'"]
      }
    }
  ],
  "video_sitemap_entries": [
    {
      "loc": "https://dentclinic.ru/services/implantaciya/",
      "video_loc": "https://www.youtube.com/embed/XXXX",
      "thumbnail_loc": "https://dentclinic.ru/media/video/v001-thumb-1280.jpg",
      "title": "Имплантация зубов в Москве: отвечает хирург с 15-летним стажем",
      "description": "Экспертное интервью с хирургом-имплантологом об имплантации зубов в Москве.",
      "duration_sec": 320,
      "publication_date": "2026-01-20T09:00:00+03:00",
      "family_friendly": "yes"
    }
  ],
  "qc_score": 88
}
```

---

## ██ СТРАТЕГИЯ ОТКАТА (Fallback при 3× QC fail)

| Откат | Действие                                                                                                                                              |
|-------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| **A** | Декомпозиция на 2 вызова Opus: (1) стратегия + metadata + thumbnail; (2) chapters + transcript_brief + schema_markup. Объединение на стороне N8N.    |
| **B** | Сократить до 1 видео на страницу (даже для hub-страниц). Hero-формат по умолчанию. video_count=1 везде.                                                |
| **C** | Fallback-промпт: подать Opus few-shot пример из этого файла как шаблон, Opus только адаптирует под нишу/гео. Строже: temperature=0.15.                 |
| **D** | Ручной триггер HitL: алерт в Telegram роли A с полным логом трёх попыток, ссылкой на input и предложением ручной правки в NocoDB.                     |

---

## ██ АДАПТАЦИИ ПОД КЛАСТЕР

| Кластер               | Типы видео                                                     | Длительность | Площадки                                             | Особенности                                                                                                         |
|-----------------------|----------------------------------------------------------------|--------------|------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------|
| **commercial_local**  | Видео-визитка, отзывы клиентов, локальные кейсы                | 30–90 сек    | Я.Видео (приоритет), Я.Бизнес, YouTube, VK Видео     | Микро-гео в title и description; привязка к Я.Бизнес-карточке; footage локаций                                       |
| **commercial_national** | Бренд-сторителлинг, экспертные интервью, кейс-стади           | 2–5 мин      | YouTube, Я.Видео, Rutube                             | Featured Snippets видео; E-E-A-T через экспертов; плейлисты для topical authority                                    |
| **ecom**              | Распаковка, how-to-use, сравнение товаров, видео-обзоры       | 60–180 сек   | YouTube, Я.Видео, Rutube, native embed на карточке   | VideoObject внутри Product schema; Review markup; нарезки в YouTube Shorts для transactional queries                 |
| **saas**              | Walkthrough, feature demo, webinar-нарезки, tutorials         | 2–8 мин      | YouTube (ключевое), Vimeo, native embed              | Международные версии с субтитрами; chapters обязательны; интеграция с документацией                                  |
| **info**              | Explainer, глубокий нарратив, экспертные интервью             | 3–10 мин     | YouTube, Я.Видео                                     | Transcript на странице критичен (для Topical Authority + AI-цитирования); обновления видео при Content Decay         |
| **medical** (YMYL)    | Экспертное интервью ТОЛЬКО с врачом с указанием ФИО и стажа   | 2–5 мин      | YouTube, Я.Видео, Rutube                             | Дисклеймер в description и pinned comment; указание лицензии; НИКОГДА не давать мед. рекомендации в hook             |
| **legal** (YMYL)      | Экспертное объяснение от адвоката/юриста со статусом           | 3–8 мин      | YouTube, Я.Видео, Rutube                             | Дисклеймер "не заменяет консультацию"; указание регистрационного номера; ситуативные запросы                          |
| **education**         | Пошаговые уроки, разбор заданий, обзоры программ              | 5–15 мин     | YouTube (плейлисты), Я.Видео, Rutube                 | Структурированные плейлисты = Course schema; сезонность (август-сентябрь, май); длинные chapters                     |

---

## ██ МЕТРИКИ УСПЕХА

| Тип метрики          | Метрика                                                                 | Источник                     |
|----------------------|-------------------------------------------------------------------------|------------------------------|
| **Опережающий**      | Покрытие страниц с video_serp_feature=video_carousel видео-планами      | NocoDB `video_seo_plans`     |
| **Опережающий**      | Доля видео с валидной VideoObject schema + chapters                     | Post-script валидатор        |
| **Опережающий**      | qc_score среднее по батчу ≥ 80                                          | Post-script валидатор        |
| **Запаздывающий**    | Показы в Я.Видео-блоках SERP через 30 дней после публикации             | DataForSEO + Я.Вебмастер     |
| **Запаздывающий**    | Video CTR в Google SERP (Search Console Performance report)             | GSC API                      |
| **Запаздывающий**    | Доля видео в AI-ответах (Яндекс Нейро + Google AI Mode)                 | Шаг 40.3 (AI-цитирования)    |
| **Запаздывающий**    | Watch time и average view duration на YouTube                           | YouTube Analytics API        |
| **Контроль**         | A/B сравнение: страницы с видео vs без — CTR и dwell time               | GA4 + Метрика                |
| **Контроль**         | Ручная проверка top-5 страниц через 7 дней: корректность schema в Rich Results Test и Яндекс.Валидаторе | Ручная проверка |

**Критерий «шаг выполнен хорошо»:** ≥90% страниц батча получили валидный видео-план с qc_score≥80; все video_sitemap_entries[] загружены в video-sitemap.xml; VideoObject schema прошла Rich Results Test и Яндекс.Валидатор без ошибок.

---

## ██ ИСТОРИЯ ИЗМЕНЕНИЙ

| **Версия** | **Дата**  | **Изменения**                                                                                                                                                                                               |
|------------|-----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| v1.0       | 2026-01   | Первая версия шага. Шаблон 3 (Генерация контента). Promps+Scripts. Поддержка YouTube + Я.Видео + Rutube + VK Видео. VideoObject schema + Clip markup + video-sitemap.                                       |
| v1.1       | *план*    | Добавить YouTube Shorts-автонарезку для transactional queries.                                                                                                                                              |
| v2.0       | *план*    | Интеграция с Шагом 40.3 (AI-цитирования): обратная связь — если видео цитируется в Нейро/AI Mode, повышение приоритета для похожих страниц. Добавить оценку лицевого авторитета канала конкурентов.          |
| v3.0       | *план*    | ML-кластеризация видео-паттернов после накопления ≥200 размеченных примеров (снимает политику «rule-based only» из CORE.md для этого шага).                                                                 |

---

*Конец STEP_12.2.md. Общий объём: ~4K токенов. Следующий потребитель: STEP_12.3.md (UGC-стратегия) + STEP_13.md (Schema.org + OG).*
