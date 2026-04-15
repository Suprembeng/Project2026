# STEP_04.1.md — Каннибализация

> **SEO AUTOMATION ENGINEERING · МОДУЛЬ 3 · STEP_04.1**
> Самодостаточный модуль. Загружается: CORE.md + PROJECT.md + этот файл. ~4K токенов.

---

## ██ КАРТОЧКА ШАГА

| **Параметр**          | **Значение**                                                                 |
|-----------------------|------------------------------------------------------------------------------|
| ID                    | 04.1                                                                         |
| Название              | Каннибализация                                                               |
| Блок                  | 2: Структура и контент                                                       |
| Основная роль         | A: SEO-стратег                                                               |
| Консультативные       | B: Инженер автоматизации, D: Аналитик конкурентной разведки                  |
| Тип                   | Промпт+Скрипт                                                               |
| HitL                  | Нет (результат → Шаг 05 автоматически)                                       |
| Приоритет ROI         | ВЫСОКИЙ — каннибализация снижает видимость и рассеивает ссылочный вес        |
| Зависит от            | Шаг 04 (Структура страниц), Шаг 01 (Сбор СЯ), Шаг 02 (Структура сайта)    |
| Передаёт в            | Шаг 05 (Коммерч. факторы), Шаг 07 (Перелинковка), Шаг 08 (ТЗ на страницы), Шаг 15 (Canonical) |
| JSON-шаблон           | Template 1: List with Priorities                                             |
| Режим Opus            | JSON-вывод: temperature 0.15, top_p 0.9, max_tokens 8192                    |

---

## ██ ЦЕЛЬ

Обнаружить все случаи каннибализации ключевых слов (когда ≥2 страниц сайта конкурируют за один запрос/кластер), оценить severity каждого случая и сформировать приоритизированный список действий (merge, redirect, canonical, re-target, noindex) с engine-специфичными рекомендациями для Яндекса и Google.

---

## ██ КОНТРАКТ ВХОДНЫХ ДАННЫХ (JSON)

Вход агрегируется из output-контрактов Шагов 04, 01 и 02. Минимальные обязательные поля:

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
  "project_phase": "setup│growth│maintenance│recovery",

  "page_type_structures": [
    {
      "page_type": "string",
      "page_type_label": "string",
      "target_intents": ["string"],
      "mapped_persona_ids": ["string"],
      "content_blocks": [
        {
          "block_id": "string",
          "block_type": "string",
          "target_cluster_ids": ["string"],
          "keywords_placement": "string│null"
        }
      ]
    }
  ],

  "site_architecture": {
    "pages": [
      {
        "url_pattern": "string",
        "page_type": "string",
        "assigned_clusters": [
          {
            "cluster_id": "string",
            "queries": [
              { "query": "string", "engine": "yandex│google│both", "intent": "string", "volume": "number" }
            ]
          }
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
        "queries": [
          { "query": "string", "volume": "number", "engine": "yandex│google│both" }
        ]
      }
    ]
  },

  "existing_pages": [
    {
      "url": "string",
      "title": "string│null",
      "h1": "string│null",
      "meta_description": "string│null",
      "word_count": "number│null",
      "indexed": "boolean",
      "last_modified": "string│null"
    }
  ]
}
```

---

## ██ КОНТРАКТ ВЫХОДНЫХ ДАННЫХ (JSON)

Наследует **Template 1: List with Priorities**. Роль G валидирует. Должен пройти структурный QC.

```json
{
  "project_id": "string",
  "step_id": "04.1",
  "prompt_version": "string (regex: ^step04_1_v\\d+\\.\\d+$)",
  "timestamp": "ISO 8601",
  "source_data_versions": {
    "step_04": "string",
    "step_01": "string",
    "step_02": "string"
  },

  "items": [
    {
      "id": "string (CAN_001...)",
      "title": "string (краткое описание случая каннибализации)",
      "priority": "critical│high│medium│low",
      "action_required": "merge│redirect_301│canonical│re_target│noindex│split│rewrite",
      "engine_tag": "[Я]│[G]│[ОБА]",
      "estimated_impact": "number 1-10",
      "effort": "number 1-10",

      "cannibalization_type": "keyword│cluster│intent│mixed",
      "conflicting_pages": [
        {
          "url": "string",
          "page_type": "string",
          "assigned_clusters": ["string"],
          "current_position": "number│null",
          "current_traffic": "number│null",
          "is_primary": "boolean"
        }
      ],
      "overlapping_queries": [
        {
          "query": "string",
          "volume": "number",
          "engine": "yandex│google│both",
          "intent": "string"
        }
      ],
      "cosine_similarity": "number 0.0-1.0 (между конфликтующими страницами)",
      "resolution": {
        "winner_url": "string (страница-победитель)",
        "loser_urls": ["string"],
        "action_details": "string (конкретные шаги: что слить, куда редиректить, какой canonical поставить)",
        "redirect_map": [
          { "from": "string", "to": "string", "type": "301│canonical│noindex" }
        ],
        "clusters_to_reassign": [
          { "cluster_id": "string", "from_url": "string", "to_url": "string" }
        ]
      },
      "confidence": "[✓]│[◆]│[◇]│[⚠]",
      "detection_source": "embedding│serp│cluster_overlap│title_overlap│url_pattern"
    }
  ],

  "summary": {
    "total": "number",
    "critical": "number",
    "high": "number",
    "medium": "number",
    "low": "number",
    "total_affected_pages": "number",
    "total_affected_queries": "number",
    "estimated_traffic_at_risk": "number│null"
  },

  "prevention_rules": [
    {
      "rule": "string",
      "applies_to_page_types": ["string"],
      "engine_tag": "[Я]│[G]│[ОБА]"
    }
  ],

  "qc_score": "number 0-100"
}
```

---

## ██ ПРОМПТ ДЛЯ CLAUDE OPUS

### Системный промпт

```
Ты — Роль A: Senior SEO-стратег (10+ лет, Яндекс + Google).
Консультативные: Роль B (Инженер автоматизации), Роль D (Конкурентная разведка).

Задача: Проанализировать результаты скрипта детекции каннибализации и сформировать приоритизированный список действий по каждому случаю.

[ПРОТОКОЛ РАССУЖДЕНИЯ — CoT]
Перед ответом выполни внутренний анализ (НЕ выводи его в JSON):
[АНАЛИЗ] Сколько конфликтов обнаружено? Какие типы каннибализации?
[РОЛИ] A: SEO-стратегия разрешения. B: техническая реализация. D: что показывает SERP?
[КОНТРАКТ] Вход: список конфликтов от скрипта. Выход: Template 1 + resolution.
[БЕНЧМАРК] Как конкуренты решают аналогичные конфликты?
[РЕШЕНИЕ] Конкретное действие для каждого конфликта.
[РИСКИ] Потеря трафика при неправильном merge/redirect.

ПРАВИЛА:
1. Для КАЖДОГО конфликта из pre-step скрипта определи:
   - cannibalization_type: keyword (один запрос), cluster (кластер), intent (пересечение интентов), mixed
   - priority: critical (>1000 vol, позиции проседают), high (>500 vol), medium (>100 vol), low (<100 vol)
   - action_required: merge│redirect_301│canonical│re_target│noindex│split│rewrite
2. Для каждого конфликта выбери winner_url — страницу, которая остаётся:
   - Критерии выбора (в порядке приоритета):
     a) Больше трафика / лучшие позиции
     b) Больше внешних ссылок
     c) Выше в архитектуре (меньше depth)
     d) Более релевантный page_type для интента
     e) При прочих равных — старая страница (домен-авторитет)
3. Для winner_url — укажи какие кластеры/запросы сохранить.
   Для loser_urls — укажи действие (redirect, canonical, noindex, rewrite).
4. engine_tag ОБЯЗАТЕЛЕН на каждом item:
   [Я] — если каннибализация видна только в Яндексе (разные URL в выдаче по одному запросу)
   [G] — если только в Google
   [ОБА] — если в обоих
5. При priority_engines="both": ≥20% [Я] и ≥20% [G].
6. confidence маркировка:
   [✓] — подтверждено данными SERP (конфликтующие URL видны в топ-20)
   [◆] — высокое cosine similarity (>0.85) + cluster overlap
   [◇] — потенциальная каннибализация по title/H1 overlap
   [⚠] — ложноположительное, но рекомендуется мониторинг
7. prevention_rules: после анализа всех конфликтов — 3-7 правил предотвращения каннибализации для будущего контента.
8. Учитывай niche_cluster:
   - ecom: фильтр-страницы vs категории — частая проблема, решение через canonical/noindex фильтров
   - info: длинные статьи с пересечением тем — решение через внутреннюю перелинковку + re-target
   - commercial_local: гео-страницы — отдельная страница на район vs общая городская
   - saas: /features/ vs /solutions/ vs /use-cases/ — re-target по интентам
   - medical: симптомы vs лечение — разные интенты, не всегда каннибализация
9. НЕ считай каннибализацией:
   - Навигационные дубли (например, /about/ и /o-nas/ с 301 редиректом)
   - Категория + подкатегория (иерархическая вложенность, разные уровни интента)
   - Главная страница + сервисная страница по брендовому запросу
10. Если конфликтов > 30 — разбей на 2 вызова: critical+high в первом, medium+low во втором.
11. prompt_version: хардкод из системного промпта, не генерируй сам.

ВЫВОД: Валидный JSON по контракту Template 1. Без markdown-обёртки. Без комментариев. Без ```json. Только JSON. Весь текст — на РУССКОМ языке.
```

### Пользовательский промпт (Шаблон)

```
Проанализируй каннибализацию и сформируй приоритизированный список действий:

Проект: {{project.name}}
Ниша: {{project.niche}}
Кластер: {{project.niche_cluster}}
Гео: {{project.geo}}
Приоритет ПС: {{project.priority_engines}}
Домен: {{project.domain}}
CMS: {{project.cms}}
Фаза: {{project.project_phase}}

=== РЕЗУЛЬТАТЫ СКРИПТА ДЕТЕКЦИИ ===
Всего конфликтов: {{pre_step.total_conflicts}}
Критических (cosine >0.90): {{pre_step.critical_count}}
Высоких (cosine 0.85-0.90): {{pre_step.high_count}}
Средних (cosine 0.75-0.85): {{pre_step.medium_count}}

Конфликты:
{{#each pre_step.conflicts}}
---
conflict_id: {{id}}
type: {{type}}
cosine_similarity: {{cosine_similarity}}
pages:
{{#each pages}}
  - url: {{url}} | page_type: {{page_type}} | clusters: [{{assigned_clusters | join(', ')}}] | position_ya: {{position_yandex}} | position_g: {{position_google}} | traffic: {{monthly_traffic}}
{{/each}}
overlapping_queries:
{{#each overlapping_queries}}
  - "{{query}}" | vol: {{volume}} | engine: {{engine}} | intent: {{intent}}
{{/each}}
{{/each}}

=== АРХИТЕКТУРА (контекст) ===
{{#each site_architecture.pages | take(20)}}
- {{url_pattern}} ({{page_type}}, depth={{depth}})
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
| `page_structures`       | project_id + step_id=04                         | Структуры страниц из Шага 04 для контекста                          | 600           |
| `competitors`           | niche + geo + "каннибализация"                  | Как конкуренты решают конфликты, какие паттерны URL используют       | 600           |
| `cannibalization_cases` | niche_cluster                                   | Исторические кейсы каннибализации из похожих проектов                | 400           |
| `personas`              | project_id                                      | Интенты персон для правильного re-target                            | 400           |

Общий лимит RAG-контекста: **2000 токенов**.

Порядок инъекции: page_structures → competitors → cannibalization_cases → personas. Если суммарно > 2000, обрезать personas.

---

## ██ СКРИПТЫ CLAUDE CODE

> Тип шага: **Промпт+Скрипт** → обе секции развёрнуты: скрипты обнаруживают конфликты, Opus анализирует и приоритизирует.

### Пред-шаг 1: Сбор текущих данных страниц

```
# step_04_1_pre_crawl_pages.py
# Назначение: Получить актуальные title, H1, мета-описание, контент для всех страниц сайта
# Вход: site_architecture.pages[].url_pattern, project.domain
# Выход: page_content_map.json [{url, title, h1, meta_description, text_content, word_count}]
# Действия:
# 1. Для каждой страницы из site_architecture: GET запрос
#    - JS-рендеринг при CMS SPA (cms_spa=true): Playwright fallback
#    - Rate limit: 2 RPS к своему домену
#    - Timeout: 10с per URL, общий timeout: 600с
# 2. Извлечь: <title>, <h1>, <meta name="description">, основной текстовый контент (без nav/footer)
# 3. Для существующих existing_pages[] — merge данных (не перезатирать existing)
# 4. Кэш: Redis/файл, TTL 24 часа, ключ sha256(url)
# 5. Ошибки: пропуск с логом, не блокировать весь скрипт
```

### Пред-шаг 2: Embedding-анализ + Cosine Similarity

```
# step_04_1_pre_embedding_analysis.py
# Назначение: Обнаружение каннибализации через embedding similarity
# Вход: page_content_map.json, semantic_core.clusters, site_architecture.pages
# Выход: cannibalization_conflicts.json
# Действия:
# 1. Для каждой страницы: создать текстовый профиль
#    profile = f"{title} {h1} {' '.join(assigned_queries[:20])} {text_content[:500]}"
# 2. Получить эмбеддинг каждого профиля (OpenAI text-embedding-3-small)
#    - Батчинг: до 100 профилей за вызов
#    - Retry: 3× с бэкоффом 2/4/8с при 429/500
#    - Fallback: sentence-transformers локально при 3× fail
# 3. Матрица cosine similarity NxN (N = количество страниц)
#    - Оптимизация: skip пары с одинаковым parent_url и depth_diff > 1 (иерархия)
# 4. Обнаружение конфликтов (пороги):
#    - cosine >= 0.90 → critical (почти дубликат)
#    - cosine >= 0.85 → high (сильное пересечение)
#    - cosine >= 0.75 → medium (потенциальное пересечение)
#    - cosine < 0.75 → нет конфликта (skip)
# 5. Для каждого конфликта определить overlapping_queries:
#    - Пересечение assigned_clusters между страницами
#    - Пересечение конкретных запросов внутри кластеров
# 6. Обнаружение по title/H1 overlap:
#    - Jaccard similarity названий > 0.6 → добавить как conflict (detection_source: title_overlap)
# 7. Обнаружение по URL-паттерну:
#    - /uslugi/chistka-zubov/ vs /uslugi/professionalnaya-chistka/ → flag
#    - Levenshtein distance URL-path < 0.3 нормализованный → conflict
# 8. SERP-проверка (опциональная, через DataForSEO):
#    - Для топ-20 запросов по volume: проверить есть ли 2+ URL домена в одной выдаче
#    - Если оба URL в топ-20 → detection_source: serp, confidence: [✓]
#    - Если yandex и google показывают разные URL → engine_tag определяется раздельно
# 9. Дедупликация конфликтов: если пара A-B и B-C конфликтуют → объединить в группу A-B-C
# 10. Выход: массив конфликтов с метаданными для передачи в промпт Opus
```

### Пред-шаг 3: Сбор позиций и трафика (опциональный)

```
# step_04_1_pre_positions.py
# Назначение: Получить текущие позиции и трафик для конфликтующих страниц
# Вход: cannibalization_conflicts.json, project.domain
# Выход: enriched_conflicts.json (конфликты с позициями и трафиком)
# Действия:
# 1. Для каждого overlapping_query из конфликтов:
#    - DataForSEO Rank Tracker или Топвизор API: позиция по yandex + google
#    - GA4/Метрика API: трафик per URL за 30 дней
# 2. Определить winner/loser по данным:
#    - Кто выше в SERP?
#    - Кто получает больше трафика?
# 3. Если API недоступен → graceful degradation: пометить position=null, traffic=null
# 4. Обогатить cannibalization_conflicts.json позициями и трафиком
```

### Пост-шаг: Валидатор

```
# step_04_1_post_validate.py
# Назначение: Валидация выхода по JSON-схеме и бизнес-правилам
# Проверки:
# 1. JSON Schema (Template 1 + расширения cannibalization)
# 2. Каждый item имеет уникальный id (regex: ^CAN_\d{3}$)
# 3. conflicting_pages.length >= 2 для каждого item
# 4. Ровно 1 is_primary=true в conflicting_pages
# 5. winner_url входит в conflicting_pages и имеет is_primary=true
# 6. loser_urls — все оставшиеся URL из conflicting_pages
# 7. engine_tag валиден: [Я]│[G]│[ОБА]
# 8. При priority_engines="both": баланс ≥15% [Я] и ≥15% [G]
# 9. overlapping_queries не пусты для каждого item
# 10. cosine_similarity ∈ [0.0, 1.0]
# 11. confidence ∈ {[✓], [◆], [◇], [⚠]}
# 12. action_required ∈ {merge, redirect_301, canonical, re_target, noindex, split, rewrite}
# 13. summary.total == items.length
# 14. summary.critical + high + medium + low == total
# 15. prevention_rules >= 3 и <= 7
# 16. prompt_version соответствует regex ^step04_1_v\d+\.\d+$
# 17. timestamp — валидный ISO 8601
# 18. Нет дублирующихся пар URL в разных items (один конфликт = один item)
# 19. redirect_map: from ≠ to, нет циклических редиректов
# 20. clusters_to_reassign: cluster_id существует в semantic_core
```

### Пост-шаг: Qdrant

```
# step_04_1_post_qdrant.py
# Назначение: Сохранение результатов каннибализации в Qdrant
# Действия:
# 1. Для каждого item: эмбеддинг(title + overlapping_queries + action_required)
# 2. Upsert в коллекцию 'cannibalization_cases'
# 3. Deterministic ID: md5(project_id + step_id + conflict_pages_sorted)
# 4. Payload: project_id, step_id, niche_cluster, priority, action_required, timestamp
# 5. Retry: 3× при connection error, fallback в NocoDB-очередь
```

---

## ██ КРИТЕРИИ QC

| **Проверка**                                      | **Порог**                            | **Вес** | **При провале** |
|--------------------------------------------------|--------------------------------------|---------|-----------------|
| JSON Schema Template 1                            | Pass/Fail                            | 15      | Retry           |
| Все конфликты из pre-step покрыты                 | 100% critical+high, ≥80% medium      | 20      | Retry           |
| Уникальность item.id                              | 100%                                 | 5       | Retry           |
| Ровно 1 is_primary per item                       | 100%                                 | 10      | Retry           |
| winner_url = is_primary URL                       | 100%                                 | 10      | Retry           |
| Engine tag на каждом item                         | 100%                                 | 10      | Retry           |
| Баланс engine_tag при both                        | ≥15% [Я] и ≥15% [G]                | 5       | Ревью           |
| Нет циклических redirect_map                      | 0 циклов                            | 5       | Retry           |
| overlapping_queries не пусты                      | 100% items                           | 5       | Retry           |
| confidence маркировка                             | 100% items                           | 5       | Ревью           |
| prevention_rules                                  | 3-7 правил                           | 5       | Ревью           |
| summary корректна                                 | total == items.length, суммы верны   | 5       | Retry           |

**QC = сумма весов. ≥80 → Шаг 05, 60-79 → ревью, <60 → retry.**

---

## ██ СТРУКТУРА N8N-ВОРКФЛОУ

```
Нода 1:   Триггер (после завершения Шага 04 или ручной запуск)
Нода 1a:  Idempotency Lock (NocoDB: step_04_1_lock:{project_id})
Нода 2:   Загрузка PROJECT.md из NocoDB
Нода 2a:  Dependency Gate (step_04 done?)
Нода 3│4: [PARALLEL] Загрузка output Шага 04 + Шага 01 из NocoDB
Нода 4a:  Merge + Cross-validate cluster_ids
Нода 5:   step_04_1_pre_crawl_pages.py (краулинг страниц)
Нода 6:   step_04_1_pre_embedding_analysis.py (cosine similarity)
Нода 7:   step_04_1_pre_positions.py (позиции и трафик, опционально)
Нода 8:   Qdrant RAG (page_structures + competitors + cannibalization_cases + personas)
Нода 8a:  Token Budget Check (<6K → обрезать кластеры)
Нода 9:   Сборка промпта (system + user + RAG + CoT)
Нода 10:  Claude Opus API (temp 0.15, top_p 0.9, max_tokens 8192)
          — retryOnFail: true, maxTries: 3, waitBetweenTries: exponential 10/30/90с
Нода 11:  Парсинг JSON (удаление markdown-обёртки если есть)
Нода 12:  step_04_1_post_validate.py
          ├── QC ≥80 → Нода 13
          ├── QC 60-79 → Нода 12a: HitL Telegram → Нода 13 или retry
          └── QC <60 → Retry Protocol v2:
              → Retry 1: Нода 9r1 — инъекция ошибок QC
              → Retry 2: Нода 9r2 — декомпозиция (critical+high / medium+low)
              → Retry 3: Нода 9r3 — fallback-промпт с few-shot
              → Нода 16a: Rollback + Telegram-алерт
Нода 13:  step_04_1_post_qdrant.py
Нода 14:  NocoDB Save (prompt_hash, prompt_version, source_data_versions, qc_score)
Нода 15:  Grafana Metrics
Нода 16:  Trigger Шаг 05 или стоп при fail
```

### Обработка ошибок

```
• Opus 429/500: бэкофф 10с→30с→90с, макс. 3
• Qdrant недоступен: промпт без RAG-контекста, лог предупреждения
• DataForSEO 402/403: graceful degradation — конфликты без позиций, Opus анализирует только по cosine
• Embedding API fail: fallback на sentence-transformers локально
• QC провал 3x: алерт Telegram с логом 3 попыток
• Невалидный JSON: strip markdown, повторный парсинг → retry
• Нет данных Шага 04: СТОП, алерт — зависимости не выполнены
• > 50 конфликтов: автодекомпозиция на 2 вызова Opus (по priority)
```

---

## ██ ПРИМЕР FEW-SHOT

```json
{
  "project_id": "dental_moscow_001",
  "step_id": "04.1",
  "prompt_version": "step04_1_v3.0",
  "timestamp": "2025-12-16T14:00:00Z",
  "source_data_versions": {
    "step_04": "step04_v3.0",
    "step_01": "step01_v2.0",
    "step_02": "step02_v2.0"
  },

  "items": [
    {
      "id": "CAN_001",
      "title": "Каннибализация: /uslugi/chistka-zubov/ и /blog/professionalnaya-chistka-zubov/ конкурируют за кластер 'чистка зубов'",
      "priority": "critical",
      "action_required": "re_target",
      "engine_tag": "[ОБА]",
      "estimated_impact": 8,
      "effort": 3,
      "cannibalization_type": "cluster",
      "conflicting_pages": [
        {
          "url": "/uslugi/chistka-zubov/",
          "page_type": "service",
          "assigned_clusters": ["cluster_chistka_main"],
          "current_position": 12,
          "current_traffic": 340,
          "is_primary": true
        },
        {
          "url": "/blog/professionalnaya-chistka-zubov/",
          "page_type": "article",
          "assigned_clusters": ["cluster_chistka_main", "cluster_chistka_info"],
          "current_position": 18,
          "current_traffic": 120,
          "is_primary": false
        }
      ],
      "overlapping_queries": [
        { "query": "чистка зубов москва", "volume": 2400, "engine": "both", "intent": "commercial" },
        { "query": "профессиональная чистка зубов", "volume": 1800, "engine": "both", "intent": "commercial" },
        { "query": "чистка зубов цена", "volume": 1200, "engine": "yandex", "intent": "transactional" }
      ],
      "cosine_similarity": 0.91,
      "resolution": {
        "winner_url": "/uslugi/chistka-zubov/",
        "loser_urls": ["/blog/professionalnaya-chistka-zubov/"],
        "action_details": "Сервисная страница остаётся основной по коммерческим запросам (чистка зубов москва, цена). Блог-статью перенацелить на информационный кластер: 'как проходит чистка зубов', 'больно ли делать чистку', 'виды чистки'. Убрать из статьи коммерческие запросы и CTA записи. Добавить canonical на сервисную при пересечении.",
        "redirect_map": [],
        "clusters_to_reassign": [
          { "cluster_id": "cluster_chistka_main", "from_url": "/blog/professionalnaya-chistka-zubov/", "to_url": "/uslugi/chistka-zubov/" }
        ]
      },
      "confidence": "[✓]",
      "detection_source": "embedding"
    },
    {
      "id": "CAN_002",
      "title": "Каннибализация: /uslugi/implantaciya/ и /uslugi/implant-zubov/ — дубль URL",
      "priority": "critical",
      "action_required": "redirect_301",
      "engine_tag": "[Я]",
      "estimated_impact": 9,
      "effort": 1,
      "cannibalization_type": "keyword",
      "conflicting_pages": [
        {
          "url": "/uslugi/implantaciya/",
          "page_type": "service",
          "assigned_clusters": ["cluster_implant"],
          "current_position": 8,
          "current_traffic": 560,
          "is_primary": true
        },
        {
          "url": "/uslugi/implant-zubov/",
          "page_type": "service",
          "assigned_clusters": ["cluster_implant"],
          "current_position": 24,
          "current_traffic": 45,
          "is_primary": false
        }
      ],
      "overlapping_queries": [
        { "query": "имплантация зубов москва", "volume": 3200, "engine": "yandex", "intent": "commercial" },
        { "query": "имплант зуба цена", "volume": 2100, "engine": "both", "intent": "transactional" }
      ],
      "cosine_similarity": 0.96,
      "resolution": {
        "winner_url": "/uslugi/implantaciya/",
        "loser_urls": ["/uslugi/implant-zubov/"],
        "action_details": "Полный 301 редирект с /uslugi/implant-zubov/ на /uslugi/implantaciya/. Уникальный контент со страницы-донора (если есть) перенести на победителя. Обновить sitemap.xml.",
        "redirect_map": [
          { "from": "/uslugi/implant-zubov/", "to": "/uslugi/implantaciya/", "type": "301" }
        ],
        "clusters_to_reassign": []
      },
      "confidence": "[✓]",
      "detection_source": "serp"
    },
    {
      "id": "CAN_003",
      "title": "Потенциальная каннибализация: /uslugi/viniry/ и /blog/viniry-ili-koronki/ по запросу 'виниры'",
      "priority": "medium",
      "action_required": "canonical",
      "engine_tag": "[G]",
      "estimated_impact": 5,
      "effort": 2,
      "cannibalization_type": "intent",
      "conflicting_pages": [
        {
          "url": "/uslugi/viniry/",
          "page_type": "service",
          "assigned_clusters": ["cluster_viniry"],
          "current_position": 6,
          "current_traffic": 280,
          "is_primary": true
        },
        {
          "url": "/blog/viniry-ili-koronki/",
          "page_type": "article",
          "assigned_clusters": ["cluster_viniry_info"],
          "current_position": null,
          "current_traffic": 90,
          "is_primary": false
        }
      ],
      "overlapping_queries": [
        { "query": "виниры", "volume": 4500, "engine": "google", "intent": "mixed" }
      ],
      "cosine_similarity": 0.78,
      "resolution": {
        "winner_url": "/uslugi/viniry/",
        "loser_urls": ["/blog/viniry-ili-koronki/"],
        "action_details": "Установить canonical с блог-статьи на сервисную по коммерческому запросу 'виниры'. Блог-статью оставить для информационного трафика ('виниры или коронки что лучше'). Google в SERP показывает оба URL — canonical поможет консолидировать.",
        "redirect_map": [
          { "from": "/blog/viniry-ili-koronki/", "to": "/uslugi/viniry/", "type": "canonical" }
        ],
        "clusters_to_reassign": []
      },
      "confidence": "[◆]",
      "detection_source": "embedding"
    }
  ],

  "summary": {
    "total": 3,
    "critical": 2,
    "high": 0,
    "medium": 1,
    "low": 0,
    "total_affected_pages": 6,
    "total_affected_queries": 6,
    "estimated_traffic_at_risk": 1435
  },

  "prevention_rules": [
    {
      "rule": "Один коммерческий кластер = одна сервисная страница. Блог-статьи нацеливать только на информационные подкластеры.",
      "applies_to_page_types": ["service", "article"],
      "engine_tag": "[ОБА]"
    },
    {
      "rule": "Перед созданием новой страницы — проверка cosine similarity с существующими (порог 0.80). При превышении — не создавать, а расширять существующую.",
      "applies_to_page_types": ["service", "article", "landing"],
      "engine_tag": "[ОБА]"
    },
    {
      "rule": "Гео-страницы: уникальный контент ≥40% от общего объёма. При меньшем — canonical на основную гео-страницу.",
      "applies_to_page_types": ["geo_landing"],
      "engine_tag": "[Я]"
    },
    {
      "rule": "[G] При пересечении интентов (commercial + informational) — canonical с информационной на коммерческую.",
      "applies_to_page_types": ["article", "blog", "faq"],
      "engine_tag": "[G]"
    },
    {
      "rule": "[Я] Фильтр-страницы ecom: canonical на родительскую категорию при <5 товаров в фильтре.",
      "applies_to_page_types": ["category", "subcategory"],
      "engine_tag": "[Я]"
    }
  ],

  "qc_score": 91
}
```

---

## ██ СТРАТЕГИЯ ОТКАТА

```
Откат A: Декомпозиция на 2 вызова Opus:
  (1) Конфликты с priority critical + high (самые важные)
  (2) Конфликты с priority medium + low
  Объединение в post-step скрипте.

Откат B: Сократить до топ-10 конфликтов по cosine_similarity.
  Остальные — пометка в NocoDB для ручного разбора.

Откат C: Few-shot как жёсткий шаблон.
  Opus адаптирует шаблон конфликта под конкретные URL/запросы.
  Менее точные action_details, но структура валидна.
```

---

## ██ АДАПТАЦИИ ПОД КЛАСТЕР

| **Кластер**            | **Типичные паттерны каннибализации**                                  | **Специфика решения**                                    |
|------------------------|----------------------------------------------------------------------|----------------------------------------------------------|
| commercial_local       | Гео-страницы (/moscow/ vs /msk/), услуга+район vs услуга+город      | Canonical на город, гео-подстраницы только при >40% уникальности |
| commercial_national    | Категория vs лендинг, бренд+модель vs модель                        | Re-target по интентам, сохранение обоих при разных интентах |
| ecom                   | Фильтры vs категории, товар vs обзор товара                         | Noindex фильтров с <5 товаров, canonical на категорию     |
| saas                   | /features/ vs /solutions/ vs /use-cases/                             | Re-target: features=navigational, solutions=commercial, use-cases=informational |
| info                   | Длинные статьи с пересечением тем, обновлённая vs старая версия      | Merge контента в одну статью, 301 со старой               |
| medical                | Симптом vs диагноз vs лечение — разные интенты, ложная каннибализация | Не считать каннибализацией при разных hunt_levels          |
| legal                  | Услуга vs статья-консультация по той же теме                        | Re-target: услуга=transactional, статья=informational      |
| education              | Программа vs курс vs отзывы о курсе                                | Иерархическая вложенность, canonical при пересечении       |

---

## ██ МЕТРИКИ УСПЕХА

| **Тип метрики**     | **Метрика**                                                           | **Как измерять**                                          |
|---------------------|----------------------------------------------------------------------|----------------------------------------------------------|
| Опережающая         | Все critical+high конфликты имеют resolution с winner_url             | QC-валидатор: 100% coverage                               |
| Опережающая         | prevention_rules покрывают все обнаруженные паттерны                  | Ревью: каждый тип конфликта → ≥1 правило                 |
| Запаздывающая       | Через 2 нед. после внедрения: количество URL домена в SERP по одному запросу ≤1 | DataForSEO/Топвизор: мониторинг SERP presence             |
| Запаздывающая       | Через 1 мес.: рост средней позиции winner_url на ≥3 позиции          | GSC / Яндекс.Вебмастер: position delta                    |
| Контрольная         | Ревью после Шага 07: redirect_map из Step 04.1 учтён в перелинковке  | Кросс-шаговый QC: пересечение данных                      |
| Контрольная         | Ревью после Шага 15: canonical из Step 04.1 внедрён                  | Кросс-шаговый QC: canonical_map пересечение               |

---

## ██ ИСТОРИЯ ИЗМЕНЕНИЙ

| **Версия** | **Дата**   | **Изменения**                                                                      |
|------------|------------|-------------------------------------------------------------------------------------|
| v1.0       | 2025-01    | Базовое обнаружение дублей по title/URL                                             |
| v2.0       | 2025-06    | Embedding cosine similarity, SERP-верификация, engine_tag, Template 1               |
| v3.0       | 2025-12    | CoT, маркировка уверенности, prevention_rules, niche_cluster адаптации, Retry Protocol v2, cross-step refs, detection_source, resolution с redirect_map и clusters_to_reassign, расширенный QC (20 проверок) |
